# ADR-001: Candidate runtime for the first EL-Rouji job runner

**Status:** Proposed; not approved, implemented, or deployed  
**Date:** 24 October 2026  
**Decision owner:** EL-Rouji project owner  
**Scope:** First bounded job-processing MVP

## Context

The core EL-Rouji application is hosted on AppDeploy. A read-only inspection of the connected AppDeploy SDK reference confirmed support for native scheduled handlers (cron.json) and an application database (db). The current core deployment reports no configured cron entries. This repository still contains no executable job runner, scheduler, job API, or worker.

The existence of platform primitives does not itself establish that a generic queue will scale or is the right design. AppDeploy documents strict database request/item limits and states that db.list filters are applied after reads; unbounded scans of a growing job table would be unsafe and potentially expensive.

## Candidate decision

For a low-volume, allow-listed MVP, prefer an AppDeploy-native worker deployed as part of the core application, with this repository retaining the job contract, tests, operations runbook, and future integration tests.

- Durable job state: AppDeploy database APIs.
- Periodic dispatcher: a bounded, top-level cron handler exported from backend/index.ts.
- Cron configuration: project-root cron.json, reviewed as a complete desired-state file before every deployment.
- API ownership: core app routes. Every user-owned job operation must derive its owner from requireAuth() and scope all reads and writes to that identity.
- Execution: named, server-defined handlers only. Never execute arbitrary user code or let clients select provider credentials.
- Delivery promise: durable at-least-once processing with idempotent handlers and attempt receipts; do not claim exactly-once execution.
- Higher volume or tighter latency requirements: revisit a dedicated managed queue rather than forcing database scans beyond their intended limits.

This candidate is not authorization to add a production queue or deploy cron changes.

## Verified platform constraints

The current AppDeploy SDK reference documents:

- cron.json entries contain a unique name, a five-field cron expression, a top-level exported handler name, and optional IANA timezone/static payload.
- Minimum cron interval is five minutes; no more than six cron entries are supported per app.
- Cron handlers receive invocation metadata and must return { statusCode: 200 }; returning undefined fails the invocation.
- Listing cron.json is desired-state management: listed entries are upserted and omitted existing entries are deleted. An empty array deletes all cron entries.
- Database calls have a 1 MiB serialized-input cap. db.add/get/update/delete accept at most 500 array elements per call; each serialized record is limited to 256 KiB. Per-app minute and daily quotas apply.
- db.list filters run after reads, so filters do not reduce read cost. Reads must be bounded and pagination/checkpoints explicit.
- Conditional patch/transaction APIs exist; transaction limits and unknown outcomes must be handled using the current platform contract.

The connected SDK reference is the source of truth for implementation. These constraints must be rechecked against the deployed SDK contract before coding; no job runtime has been added yet.

## Proposed lifecycle

QUEUED -> RUNNING -> SUCCEEDED

Recovery/terminal paths: RETRY_WAIT -> RUNNING, FAILED, CANCELLED, EXPIRED.

A worker must atomically claim an eligible job using conditional state changes, increment a bounded attempt counter, store a sanitized attempt record, and checkpoint completion. Retries are allowed only for classified transient failures. The system must reconcile unknown write outcomes by request/attempt ID rather than blindly repeating side effects.

## Decisions required before implementation

1. Exact first use case and allow-listed job type. A generic job runner with no real handler is not useful.
2. Whether the first jobs are system-owned only or user-created; if user-created, define the owner-facing API/UI and per-user quotas.
3. Initial maximum queue depth, concurrency, execution duration, result size, and provider-spend ceiling.
4. Retention and deletion rules for job inputs, outputs, and attempt metadata.
5. How to bound due-job selection efficiently with AppDeploy's post-read db.list filtering. A date-bucket/index strategy must be tested with realistic volume before calling the queue production-ready.
6. Retry/backoff, cancellation-race, missed-schedule, timezone/DST, and overlapping-run semantics.
7. Live deployment tests, operational owner, alerting, pause/drain/replay procedure, and rollback evidence.

## Exit criteria for the first implementation PR

- A single named job type has a complete contract and an observable, bounded side effect.
- Server-authenticated owner scope is enforced for every user-owned API operation.
- Tests cover atomic claim, duplicate delivery, unknown write outcome, retry exhaustion, cancellation races, and cross-owner access denial.
- Queue reads are demonstrably bounded; no route or cron drains an unbounded table.
- The cron handler is disabled until its payload, schedule, and operations owner are reviewed.
- GitHub CI and a staging AppDeploy test pass. The implementation PR does not include a production deployment.

## Reference boundary

See JOBS_DESIGN_SPEC.md for lifecycle, API, security, reliability, and release requirements. Use the connected AppDeploy SDK reference for actual method signatures. This document records a candidate architecture, not approval to implement or launch it.
