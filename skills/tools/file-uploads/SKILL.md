---
id: file-uploads
name: File Uploads
description: Design and troubleshoot secure, reliable file upload flows across frontend, API, storage, and background processing. Use when implementing uploads, fixing failed transfers, or hardening validation and storage controls.
category: Tools
requires: []
examples:
  - Implement presigned upload flow with server-side validation and malware scanning hooks.
  - Debug intermittent large-file upload failures caused by timeout or multipart misconfiguration.
---

# File Uploads

Build upload pipelines that are secure, observable, and resilient under real network and file-size conditions.

## When to Use

- You are implementing new upload capabilities or migrating storage providers.
- Uploads fail, stall, or produce corrupted/inaccessible objects.
- You need standardized controls for file type, size, and post-upload processing.

## Workflow

1. Define upload model (direct-to-storage, proxy-through-API, or hybrid).
2. Specify constraints: max size, allowed MIME/types, retention, and ownership.
3. Implement secure handoff:
   - Authenticated upload tokens/presigned URLs
   - Server-side validation of metadata and content type
4. Add integrity checks (checksum/hash) and idempotent retry handling.
5. Integrate post-upload pipeline (virus scan, thumbnail/transcode, indexing).
6. Monitor failures by phase: auth, transfer, validation, processing.
7. Validate with realistic file sizes, poor networks, and concurrent uploads.

## Upload Patterns

**Presigned URL flow.** Client requests a short-lived URL from your API; API validates auth and quota, generates a signed URL for direct PUT to storage (S3, GCS, Azure Blob). Client uploads directly to storage, avoiding proxy bandwidth. After upload, client notifies API to trigger post-processing. Use for large files and high throughput. Ensure URL expiry is tight (5ΓÇô15 minutes) and scope to a single object key.

**Multipart upload with resume.** For files above a threshold (e.g. 100MB), use multipart: initiate upload, upload parts in parallel, complete when all parts are received. Track part ETags and support resuming failed parts without restarting. Implement cleanup of orphaned multipart uploads via lifecycle rules or scheduled jobs to avoid storage bloat.

**Chunked upload for large files.** Split files into fixed-size chunks (e.g. 5MB). Upload chunks sequentially or in parallel; reassemble server-side using chunk order and total count. Support resumable uploads by allowing clients to query which chunks are already received and retry only missing ones.

**Server-side content-type verification with magic bytes.** Never trust `Content-Type` from the client. Read the first few bytes of the file and match against known magic-byte signatures (e.g. PDF: `%PDF`, JPEG: `FF D8 FF`, PNG: `89 50 4E 47`). Reject mismatches. Combine with extension and MIME allowlists for defense in depth.

## Common Pitfalls

- **Trusting client MIME type.** Attackers can send `Content-Type: image/png` with executable content. Always validate with magic bytes or a dedicated library.
- **No size limits on public endpoints.** Unbounded uploads enable DoS via storage exhaustion or memory pressure. Enforce max size at API gateway, application, and storage layer.
- **Missing virus scanning.** Uploaded files can contain malware. Integrate ClamAV, VirusTotal API, or cloud-native scanning before making objects accessible.
- **Broken multipart cleanup on failure.** Failed or abandoned multipart uploads leave orphaned parts. Configure S3 lifecycle rules or run periodic cleanup jobs.
- **CORS misconfiguration blocking browser uploads.** Ensure `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods` (PUT, POST), and `Access-Control-Allow-Headers` are correctly set for presigned or direct-to-storage flows.

## Output Format

Return:

- **Upload architecture and trust boundaries:** Diagram or description of client ΓåÆ API ΓåÆ storage flow; where validation occurs; which components hold credentials.
- **Validation and security controls:** Size limits, MIME allowlist, magic-byte verification, auth model, post-upload scanning.
- **Failure mode analysis with instrumentation gaps:** For each phase (auth, transfer, validation, processing), list observed failures and missing metrics/logs.
- **Hardening actions with rollout order:** Prioritized list (e.g. add magic-byte check, then virus scan, then multipart cleanup) with dependencies noted.
- **Concrete implementation notes:** Presigned URL expiry, multipart part size, chunk size, cleanup schedule, CORS config snippet.

## Constraints

- Never trust client-reported MIME type alone.
- Do not expose permanent write credentials to clients.
- Avoid unbounded file-size acceptance in public endpoints.
- Ensure uploaded objects are private by default unless explicitly public.