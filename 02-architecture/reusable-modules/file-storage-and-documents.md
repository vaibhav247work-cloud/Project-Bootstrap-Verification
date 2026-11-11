# File Storage and Document Management Module Standard

## Purpose

This standard defines the architecture and security controls for file uploads, document storage, and asset delivery. It abstracts underlying cloud storage (AWS S3, Google Cloud Storage, Azure Blob, MinIO) and enforces tenant directory isolation, content-type verification, and secure access boundaries.

---

## 1. Storage Architecture & Direct-to-Storage Uploads

To protect backend server memory and bandwidth, file uploads exceeding 5 MB should use **Client Direct-to-Storage Uploads via Presigned URLs**.

```text
1. Client requests upload ticket -> Backend API validates auth, quota, file type & size
2. Backend generates presigned S3/GCS PUT URL (valid for 5–15 minutes)
3. Client uploads file directly to object store via presigned URL
4. Client notifies Backend -> Backend verifies upload completion, checks size & metadata
5. (Optional) Asynchronous virus scanner & thumbnail generator runs via event notification
```

For small files (< 5 MB), streaming multipart upload through backend endpoints is permitted, provided the server streams directly to storage without loading the entire payload into RAM.

---

## 2. Tenant Path Isolation & Bucket Strategy

1. **MUST** enforce strict tenant key isolation within object storage:
   ```text
   s3://company-product-storage/tenants/{tenant_id}/{module}/{year}/{month}/{unique_file_id}.ext
   ```
2. **MUST NOT** store customer files in a shared unpartitioned root path.
3. **MUST NOT** make storage buckets publicly readable. All download access is mediated through:
   - **Time-limited presigned GET URLs** (default expiration: 15–60 minutes), OR
   - **Authenticated streaming proxy endpoints** for highly confidential records requiring continuous server-side authorization checks.

---

## 3. Upload Validation and Security Gates

1. **File Extension vs. MIME Magic Bytes:**
   - **MUST NOT** trust the client-supplied `Content-Type` header or filename extension alone.
   - Verify magic bytes (file signature) at the backend or via an asynchronous ingest lambda/worker (e.g., using Apache Tika or file-type detection).
2. **Executable & Script File Prohibition:**
   - Deny `.exe`, `.sh`, `.bat`, `.cmd`, `.html`, `.js`, `.php`, `.py`, `.svg` (unless SVG is strictly sanitized against embedded script tags).
3. **Malware & Virus Scanning:**
   - Files marked as `UNVERIFIED` on upload.
   - Quarantine or scan files via an asynchronous scanner (e.g., ClamAV, AWS GuardDuty Malware Protection).
   - Only transition status to `AVAILABLE` after clean scan results.
4. **Filename Sanitization:**
   - Store files on disk/S3 using UUIDs/ULIDs (e.g., `01JXYZ...pdf`). Never use user-provided raw filenames as object storage keys.
   - Store original human filename in the database table for display and `Content-Disposition` download headers.

---

## 4. Metadata Schema

```sql
CREATE TABLE stored_files (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    tenant_id BIGINT UNSIGNED NOT NULL,
    storage_provider VARCHAR(32) NOT NULL,    -- 'AWS_S3', 'GCS', 'MINIO'
    bucket_name VARCHAR(128) NOT NULL,
    storage_key VARCHAR(512) NOT NULL,        -- 'tenants/101/invoices/2026/01J...pdf'
    original_filename VARCHAR(255) NOT NULL,
    content_type VARCHAR(128) NOT NULL,
    file_size_bytes BIGINT UNSIGNED NOT NULL,
    checksum_sha256 CHAR(64) NOT NULL,
    status VARCHAR(32) NOT NULL,              -- 'PENDING', 'SCANNING', 'AVAILABLE', 'QUARANTINED', 'DELETED'
    created_by_user_id BIGINT UNSIGNED NOT NULL,
    created_at DATETIME NOT NULL,
    deleted_at DATETIME NULL,
    INDEX idx_tenant_status (tenant_id, status),
    UNIQUE KEY uq_storage_key (storage_key)
);
```

---

## 5. Retention and Lifecycle Policies

1. Implement automated lifecycle rules on buckets:
   - Incomplete multipart uploads aborted after 7 days.
   - Temporary export files deleted automatically after 24–48 hours.
   - Soft-deleted files moved to glacier/cold storage or permanently purged after the contractual retention window.
