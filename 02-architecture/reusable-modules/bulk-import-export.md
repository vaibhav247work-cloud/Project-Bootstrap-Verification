# Bulk Data Import and Export Module Standard

## Purpose

This standard defines the architecture for importing and exporting large data sets (CSV, Excel XLSX, JSON lines) in SaaS and ERP applications. It ensures operations do not exhaust server memory (JVM OOM), lock database tables, or time out HTTP reverse proxies.

---

## 1. Asynchronous Execution Lifecycle

Any import or export involving more than 1,000 records **MUST** run asynchronously through a background job:

```text
1. User requests Import/Export
         ↓
2. Backend creates a `batch_jobs` record (status = 'PENDING')
         ↓
3. Backend returns immediately with `job_id` and HTTP 202 Accepted
         ↓
4. Background Worker picks up job:
   ├─ For Export: Streams database records -> writes to file -> uploads to S3/GCS
   └─ For Import: Reads file chunk-by-chunk -> validates -> writes in batches
         ↓
5. Worker updates `batch_jobs` (status = 'COMPLETED' or 'FAILED')
         ↓
6. User downloads file via presigned URL / receives completion notification
```

---

## 2. Memory-Safe Streaming Architecture

1. **MUST NOT** load entire datasets into memory:
   - For database exports, use **Keyset / Cursor Pagination** or JDBC streaming (`setFetchSize(1000)`).
   - Never perform `SELECT * FROM table` or load all entities into a `List<Entity>`.
2. **Streaming File Writing:**
   - Write output rows directly to a temporary disk stream or streaming compression pipe (`ZipOutputStream`).
   - For Excel exports, use memory-efficient streaming libraries (e.g., Apache POI `SXSSFWorkbook` with row flushing window).
3. **Upload to Object Store:**
   - Once file generation completes, upload to object storage under the tenant's isolated path: `s3://.../tenants/{id}/exports/{job_id}.zip`.
   - Temporary export download links must expire automatically (default: 24–48 hours).

---

## 3. Bulk Import Rules and Two-Phase Commit

1. **Phase 1 — Validation & Dry-Run:**
   - Read and parse file in chunks (e.g., 500 rows per batch).
   - Validate headers, data types, required fields, foreign key references, and business constraints.
   - Collect row-by-row validation errors:
     ```json
     {
       "row": 42,
       "field": "email",
       "error": "INVALID_FORMAT",
       "value": "invalid-email"
     }
     ```
2. **Phase 2 — Chunked Database Ingestion:**
   - If validation passes (or user chooses "Skip invalid rows"), commit data in bounded transaction chunks (e.g., 200–500 rows per transaction).
   - Never attempt to insert 50,000 records in a single database transaction; it creates lock escalation, buffer pool exhaustion, and replication lag.
3. **Tenant & Identity Attribution:**
   - All imported records must have `tenant_id` and `created_by_user_id` stamped with the authenticated user who initiated the import.
   - Write a summary audit event when the batch completes.

---

## 4. Batch Job Tracking Schema

```sql
CREATE TABLE batch_jobs (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    tenant_id BIGINT UNSIGNED NOT NULL,
    user_id BIGINT UNSIGNED NOT NULL,
    job_type VARCHAR(32) NOT NULL,            -- 'IMPORT', 'EXPORT'
    entity_type VARCHAR(64) NOT NULL,         -- 'EMPLOYEE', 'BOOKING', 'TRANSACTION'
    status VARCHAR(32) NOT NULL,              -- 'PENDING', 'PROCESSING', 'COMPLETED', 'FAILED', 'CANCELLED'
    total_rows INT UNSIGNED NOT NULL DEFAULT 0,
    processed_rows INT UNSIGNED NOT NULL DEFAULT 0,
    failed_rows INT UNSIGNED NOT NULL DEFAULT 0,
    file_id BIGINT UNSIGNED NULL,             -- references stored_files for download
    error_summary_url VARCHAR(512) NULL,     -- presigned link to error CSV if validation failed
    started_at DATETIME NULL,
    completed_at DATETIME NULL,
    created_at DATETIME NOT NULL,
    INDEX idx_tenant_user_job (tenant_id, user_id, status)
);
```
