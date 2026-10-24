# Security, Privacy, and Trust Boundaries

**Status:** initial documentation based on source inspection; not a completed security audit.  
**Primary source:** [core backend](https://github.com/looloofy69-sketch/EL-Rouji/blob/main/backend/index.ts)

## 1. Data types and flows

| Data | Current documented handling | Questions still requiring verification |
|---|---|---|
| Chat messages | Sent to the hosted AI service; recent context is included | Provider retention, operational logs, and applicable contractual controls |
| Browser history | Stored locally under EL-Rouji browser-storage keys | Clear-history behavior, shared-device exposure, and backup/export behavior |
| Cloud conversation history | Optional authenticated account sync in AppDeploy storage | Cross-account isolation, deletion semantics, retention and backup lifecycle |
| Research query | Sent to the backend; general queries may be sent to Tavily; results go into AI synthesis | Provider terms, query retention, rate/cost protections |
| Public page supplied for research | Backend scrape/extracts text and passes it to AI as untrusted content | Redirect/DNS protections, edge controls, abuse/rate limiting |
| Uploaded image or text | Sent to an AI analysis endpoint | Provider processing/retention, request logs, actual upload limits |
| API credentials | Intended to remain in AppDeploy server-side secrets | Confirm secret handling, access policy, rotation and incident procedure |

Do not promise that submitted content is never retained by a provider unless that promise is backed by the actual configured provider terms and contract.

## 2. Existing source-level protections

The current backend includes:
- Authentication middleware on the three cloud conversation routes.
- User-scoped storage prefixes for conversation records.
- Validation of conversation and message IDs.
- Message, conversation, title, source-count, and serialized-record size limits.
- Validation of language and role fields.
- HTTP(S)-only validation for stored research source URLs and checks rejecting several common local/private IP patterns.
- File-type and payload-length checks for current image/document analysis routes.
- System instructions treating retrieved web page text, research excerpts, uploaded text, and image text as untrusted data.

These are source-level observations. They are not proof that all security controls are correct or sufficient in production.

## 3. Important open risks

### Public-facing AI routes

The current source applies `requireAuth()` to conversation operations. It does not show the same middleware on `/api/chat`, `/api/research`, `/api/analyze-image`, or `/api/analyze-document`. Confirm the actual hosted edge/auth boundary. Before broad public rollout, add or verify effective request limits, abuse protection, concurrency limits, cost controls, and monitoring appropriate to the service.

### Account authorization

Test using two synthetic accounts. Account B must not be able to list, read, overwrite, or delete Account A's conversation, even when the exact ID is known. Validate both direct API requests and the UI.

### Upload handling

Verify size/type limits at the UI, API, decoding, and upstream provider boundaries. Test malformed base64, wrong MIME values, empty text, very long filenames, and oversized content. Accept only declared supported formats.

### Research URL handling

The current URL helper rejects common localhost/private address patterns and only accepts HTTP(S). This is not a complete SSRF review. Verify redirect handling, DNS rebinding, IPv6/alternate encodings, and the actual platform's scrape behavior before exposing URL fetches to untrusted callers.

### Calculator

The calculator uses Math.js locally. Review the allowed function/name surface, computation limits, error handling, and resistance to excessive expressions. Do not describe it as security-reviewed until tests support that claim.

### Dependency/build chain

The core repository's latest documented full-tree audit reported three moderate development/build-tool findings in CI run #67; production-only audit in that run reported zero. This does not make the development/build chain audit-free. Track and explicitly remediate or accept the full-tree findings using the core repo's dependency audit notes.

### Logging and deletion

Use non-sensitive diagnostics. Never log keys, full message payloads, uploaded content, or account exports by default. Establish documented retention and deletion behavior for local data, cloud objects, platform logs, and providers before claiming full deletion.

## 4. Minimum security gate before broad rollout

- [ ] Verify access control and account isolation for conversation GET/PUT/DELETE flows.
- [ ] Verify auth/perimeter controls and abuse/rate/cost limits on all AI endpoints.
- [ ] Validate error responses do not leak secrets, internal stack traces, or sensitive request content.
- [ ] Test malformed/oversized requests and supported/unsupported file types.
- [ ] Review URL scraping against SSRF and redirect risks.
- [ ] Confirm secure secret storage, access control, and credential rotation.
- [ ] Document data retention, export, deletion, and provider handling.
- [ ] Review open dependency/build advisories and record their disposition.
- [ ] Record results against the exact Git commit and hosted AppDeploy version.

## 5. Incident handling

If a service credential is suspected exposed, revoke or rotate it first, replace it in secure server-side configuration, verify the service with a non-sensitive test, and document the incident without reproducing the credential. If user data is exposed, follow the owner's incident-response obligations and avoid putting private content in GitHub issues.

## 6. Claims to avoid

Do not claim the app is fully secure, privacy-certified, backed up, GDPR-compliant, continuously monitored, or protected by an SLA without current evidence and appropriate review.
