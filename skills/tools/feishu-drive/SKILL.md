---
id: feishu-drive
name: Feishu Drive
description: Integrate and operate Feishu Drive workflows for upload, permissioning, and document lifecycle automation with production-safe controls. Use when building or fixing Feishu Drive sync and automation pipelines.
category: Tools
requires: []
examples:
  - Automate Feishu Drive file uploads with folder routing and permission assignment by team.
  - Diagnose Feishu Drive API failures caused by token scope, rate limits, or file metadata mismatches.
---

# Feishu Drive

Run Feishu Drive automations with reliable API behavior, explicit ownership controls, and observable sync outcomes.

## When to Use

- You are implementing Feishu Drive-based document workflows or sync jobs.
- API calls succeed inconsistently due to auth, scope, or quota constraints.
- You need deployment and operational guardrails for enterprise file handling.

## Workflow

1. Confirm workflow scope: upload, sync, permission updates, or archival.
2. Validate API credentials, token refresh behavior, and required scopes.
3. Define deterministic folder mapping and naming conventions.
4. Implement permission model:
   - Owner/editor/viewer rules by role or group
   - Least-privilege defaults
5. Add retry/backoff for transient API failures and rate limits.
6. Capture audit logs for file ID, actor, action, and result status.
7. Run end-to-end validation with representative files and permission checks.

## Feishu API Patterns

**Tenant access token vs user access token.** Tenant access tokens are app-level and apply to app-authorized resources. User access tokens are user-level and apply to resources the user can access. Use tenant tokens for server-side automation (e.g., batch uploads to a shared folder). Use user tokens when acting on behalf of a specific user. Ensure the correct token type for the operation; mixing them causes failures.

**Folder tree traversal.** Use the Drive API to list folders recursively. Start from root or a known folder token. Handle pagination: Feishu uses `page_token` and `page_size`. Build the tree in memory or cache it for repeated access. Respect rate limits during traversal.

**File metadata operations.** Use `drive.v1.files` and `drive.v1.metas` for file operations. Metadata includes: file_token, name, parent_token, type, size, owner. For updates, use the correct endpoint and version. Check file type before operations (doc vs sheet vs file).

**Permission inheritance model.** Folders inherit permissions from parents. New files inherit from the folder. Explicit permission grants on a file override inheritance for that file. Document the inheritance chain: who can access a file depends on folder hierarchy and explicit grants. Test permission changes on a copy before applying to production folders.

## Common Pitfalls

**Token expiry causing silent failures.** Feishu tokens expire (typically 2 hours). Implement proactive refresh before expiry. Use the refresh token or re-authenticate. Log token refresh events. If calls fail with auth errors, check expiry first.

**Incorrect scope for file operations.** Some operations require specific scopes (e.g., `drive:drive:readonly` vs `drive:drive`). Request the minimum scope needed. Verify scope before deployment. Scope mismatches often cause 403 or partial failures.

**Rate limit handling without exponential backoff.** Feishu returns 429 when rate limited. Use exponential backoff: 1s, 2s, 4s, etc., with a max retry count. Do not retry immediately with the same delay. Respect `Retry-After` header if present.

**Folder permission inheritance surprises.** Changing folder permissions affects all descendants. Revoking access on a parent revokes access to children. Test permission changes in a sandbox before production. Document the inheritance model for operators.

**Large file upload timeout.** Large files may timeout on single-request uploads. Use chunked upload or multipart upload when available. Set appropriate timeouts and retry logic. Consider file size limits and split large files if needed.

## Output Format

Return:

- **Integration architecture.** Diagram or description of components: sync job, scheduler, Feishu API client, auth flow. Data flow and failure points.

- **Auth model.** Token type (tenant vs user), refresh flow, scope list, and storage (env vars, secrets manager). Verification steps.

- **API reliability findings.** Rate limits observed, retry strategy (backoff, max retries), and error handling for 4xx/5xx.

- **Permission mapping.** Owner/editor/viewer rules by role or group. Inheritance model. Governance risks (e.g., broad write access).

- **Folder mapping.** Deterministic folder structure, naming conventions, and conflict resolution.

- **Validation results.** End-to-end test outcomes: upload, permission check, metadata verification. Pass/fail status.

- **Next operational steps.** Audit logging setup, monitoring, alerting, and incident response.

## Constraints

- Do not hardcode access tokens in code or CI configs.
- Avoid broad shared-folder write access without ownership controls.
- Respect API rate limits and implement bounded retries.
- Keep auditability for every file mutation operation.