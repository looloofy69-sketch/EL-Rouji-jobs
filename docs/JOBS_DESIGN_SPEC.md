# EL-Rouji Jobs — Design Specification

**Status:** proposal / requirements baseline only  
**Implementation status:** not implemented in this repository as of 24 October 2026  
**Related application:** [EL-Rouji core](https://github.com/looloofy69-sketch/EL-Rouji)

## 1. Purpose

This document defines what must be decided and verified before a persistent jobs/scheduler subsystem is added to EL-Rouji. It is not an implementation guide for an existing worker. No queue, scheduler, job API, worker process, recurring task engine, or job database was present when this repository was inspected.

The first delivery should solve a clearly scoped operational requirement; it should not silently imply that existing prompt starters or the Planning mode execute future tasks.

## 2. Terminology

- **Job:** one bounded unit of work with an input, owner/system scope, lifecycle state, execution attempt history, and result.
- **Queue:** durable source from which workers claim ready jobs.
- **Worker:** process/service that claims and executes jobs.
- **Scheduler:** component that converts a one-time or recurring schedule into eligible job executions.
- **Run/attempt:** one execution attempt of a job.
- **Idempotency key:** identifier used to avoid duplicate side effects when the same logical request is retried.

These are proposed terms; they do not describe current deployed objects.

## 3. Proposed lifecycle

```text
Request accepted
      │ validate authorization, payload, quota
      ▼
    QUEUED ──────► RUNNING ──────► SUCCEEDED
      │               │
      │               ├──────────► RETRY_WAIT ───► RUNNING
      │               │
      │               └──────────► FAILED
      ├──────────────────────────► CANCELLED
      └──────────────────────────► EXPIRED
```

The final state set and transitions need to be agreed before an implementation. State changes must be durable and auditable.

## 4. Requirements before implementation

### Identity and authorization
- Every user-owned job must have a trusted server-derived owner/tenant ID; never trust a client-supplied owner identifier.
- Authorization must be checked on create, read, cancel, retry, edit, and delete operations.
- A user must never list or inspect another user's job payloads, outputs, or logs.
- Define separately whether system-owned maintenance jobs exist and which operator identity can manage them.

### Persistence and delivery
- Select and document the durable queue/storage service supported by the deployment platform.
- Define behavior for duplicate delivery, worker restart, partial completion, and interrupted network calls.
- Use idempotency or deduplication keys where repeated execution could create side effects.
- Do not promise exactly-once execution unless the implementation can actually guarantee it. Prefer durable at-least-once delivery plus idempotent handlers when suitable.
- Make job state transitions atomic where needed.

### Schedule semantics
- Before recurring jobs are supported, specify accepted time zones, daylight-saving transitions, missed-run behavior, schedule edits, overlap policy, maximum recurrence frequency, and cancellation.
- Choose whether a schedule produces a new immutable run record each time.
- Include a global kill switch and per-owner disable/cancel behavior.
- Never silently run a job based on an ambiguous local time.

### Worker reliability
- Define lease/visibility timeout and recovery of abandoned RUNNING jobs.
- Retry only classified transient errors; use bounded attempts, exponential backoff, and jitter.
- Define terminal failure and operator replay semantics.
- Bound execution duration, concurrency, output size, memory, and external API calls.
- Jobs that call external systems must have clear permission and cancellation boundaries.

### Security and privacy
- Do not store provider secrets in job payloads, logs, or client-visible outputs.
- Reference secrets through a server-side secret store and restrict worker access.
- Treat job prompts, downloaded pages, and file inputs as untrusted data.
- Define payload/result retention, deletion, exports, and log redaction.
- Add rate limits, per-owner quotas, abuse protection, and cost ceilings before external/AI job execution is exposed.

### Observability and operations
- Capture job ID, attempt ID, timestamps, state transitions, duration, and sanitized error category.
- Do not log full prompts, private files, credentials, or full user outputs by default.
- Provide actionable metrics for queue depth, age of oldest job, failure/retry rates, execution duration, and dead-letter count.
- Document pause, drain, replay, cancellation, rollback, and incident procedures.
- Set an explicit operator owner before production use.

## 5. Minimal proposed job record

An initial schema could include:

| Field | Purpose |
|---|---|
| `id` | Opaque server-generated job ID |
| `ownerId` | Server-verified owner or system identity |
| `type` | Allow-listed handler identifier, not arbitrary executable code |
| `status` | State from the agreed lifecycle |
| `input` | Validated, size-bounded input with secret references excluded |
| `createdAt`, `updatedAt` | Server-set timestamps |
| `scheduledFor` | Optional UTC instant for a one-time scheduled run |
| `scheduleId` | Optional reference to a recurring schedule |
| `attemptCount` | Number of recorded execution attempts |
| `idempotencyKey` | Optional deduplication key |
| `resultSummary` | Bounded, sanitized result metadata |
| `errorCode` | Stable error classification; no secrets |
| `expiresAt` | Retention/deadline boundary where applicable |

This table is a proposal, not a current database schema. Prefer normalized run/attempt records if auditability is required.

## 6. Proposed API surface (not implemented)

An eventual API could include:
- `POST /api/jobs` — validate and enqueue a job.
- `GET /api/jobs` — list jobs visible to the authenticated owner.
- `GET /api/jobs/:id` — retrieve one authorized job.
- `POST /api/jobs/:id/cancel` — request cancellation.
- `POST /api/jobs/:id/retry` — operator- or policy-authorized retry.
- A separate admin-only interface for scheduler control and dead-letter inspection.

Do not implement these paths until authentication model, persistence, quotas, schema, status semantics, and error contracts are agreed. Public job creation must not allow arbitrary code execution.

## 7. Suggested delivery gates

1. **Requirements:** define the use cases, job types, owner model, limits, and retention.
2. **Architecture decision:** select scheduler, durable storage/queue, worker runtime, secret boundary, and failure behavior.
3. **Single-shot job MVP:** one allow-listed job type, authenticated submission, durable state, idempotency, bounded retries, status API, and audit logging.
4. **Hardening:** cross-account authorization tests, abuse/rate/cost limits, retention/deletion, failure simulation, and operational runbook.
5. **Recurring schedules:** add only after time-zone, DST, missed-run, overlap, edit, and cancellation semantics are tested.
6. **Production readiness:** run load/failure tests, set monitoring/alerts and ownership, test rollback, and document actual guarantees.

## 8. Acceptance criteria for the first runnable job

- A job is durably recorded before the API claims it was accepted.
- Unauthorized users cannot see, cancel, retry, or mutate the job.
- Duplicate requests do not produce duplicate side effects when the specified idempotency contract says they should not.
- Worker restart and transient provider failure have deterministic, bounded behavior.
- Payload and output sizes, concurrency, duration, and cost are bounded.
- Secrets and sensitive content are absent from ordinary logs.
- Automated tests cover all state transitions and authorization rules.
- The deployment and rollback procedure is documented and exercised.

## 9. Non-goals for the initial subsystem

Do not add unrestricted shell execution, arbitrary user-provided scripts, unlimited recurring loops, or autonomous external side effects in the first iteration. Any execution with external consequences requires explicit authorization, guardrails, audit records, cancellation behavior, and a security review.
