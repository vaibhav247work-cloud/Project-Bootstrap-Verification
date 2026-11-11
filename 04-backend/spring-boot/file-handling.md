# File Upload and Download Standard

1. **MUST** store file metadata in MySQL and file bytes in approved object/file storage, not normal transactional table BLOB columns, unless a documented small-file exception exists.
2. **MUST** validate authorization, size, declared and detected content type, extension policy and safe filename before accepting upload.
3. **MUST** generate server-side storage keys/names; never use client filenames/paths as storage paths.
4. **MUST** scan untrusted uploads for malware when product risk requires it and quarantine until safe.
5. **MUST** use private storage and authorize every download; use short-lived signed URLs only after server-side permission/scope check.
6. **MUST** prevent content-sniffing/download risks with safe response headers and never execute uploaded content from the application origin.
7. **MUST** define lifecycle, retention, deletion, orphan cleanup and audit for sensitive uploads/downloads.
8. **MUST** process large uploads/downloads asynchronously/streaming with quotas and rate limits.
