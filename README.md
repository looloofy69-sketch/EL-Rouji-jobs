# EL-Rouji — Application and Jobs Documentation

**Documentation baseline:** 24 October 2026  
**Core application source:** [looloofy69-sketch/EL-Rouji](https://github.com/looloofy69-sketch/EL-Rouji)  
**Hosted application:** [EL-Rouji](https://el-rouji-70120b.v2.appdeploy.ai/)

This repository is the documentation and planning workspace for EL-Rouji's application operations and future job-processing capabilities.

> **Repository status:** This repository was empty when documentation work began. It currently documents the existing EL-Rouji application; it does **not** contain an executable app, a scheduler, a queue, or worker implementation. The application source of truth is the core repository linked above.

## What EL-Rouji is

EL-Rouji is a multilingual AI-assistant interface oriented toward practical everyday and professional work. The current application provides Assistant, Research, Writing, Study, Coding, and Planning modes; English, French, and Arabic UI languages; browser-local conversation history; optional account-based history sync; built-in task prompts; a local calculator; research when configured; and image/text-file analysis.

The current supported attachment formats are PNG, JPEG, WebP, TXT, MD, CSV, and JSON. PDF and Office document analysis are not supported by the current upload flow.

## Documentation map

- [Product overview and capability status](docs/PRODUCT_OVERVIEW.md)
- [System architecture and data flows](docs/ARCHITECTURE.md)
- [Backend API reference](docs/API_REFERENCE.md)
- [User guide](docs/USER_GUIDE.md)
- [Developer setup and build boundary](docs/DEVELOPMENT_SETUP.md)
- [Security, privacy, and trust boundaries](docs/SECURITY_PRIVACY.md)
- [Operations, QA, and release runbook](docs/OPERATIONS_QA_RELEASE.md)
- [Future jobs and scheduler specification](docs/JOBS_DESIGN_SPEC.md)
- [ADR-001: Candidate AppDeploy-native job runtime (proposed, not approved)](docs/ADR-001-APPDEPLOY-JOB-RUNTIME.md)
- [Change log](CHANGELOG.md)

## Repository boundaries

| Repository / service | Responsibility | Current status |
|---|---|---|
| [EL-Rouji](https://github.com/looloofy69-sketch/EL-Rouji) | Frontend, backend API handlers, build configuration, CI, and primary application docs | Source imported; CI build/typecheck established; hosted product QA and security review still have outstanding items |
| EL-Rouji-jobs (this repository) | App documentation, job-system design, and later job-runner implementation if approved | Documentation-only at this baseline; no runnable jobs code |
| AppDeploy | Hosted frontend/backend runtime, platform client SDK, secret configuration, and deployment workflow | Existing hosted runtime; release behavior must be verified independently from GitHub CI |

## Current verified engineering facts

- The core app uses React, TypeScript, Vite, Tailwind CSS 3, PostCSS, and Autoprefixer.
- The backend uses AppDeploy SDK services and exposes health, chat, research, image/document analysis, and authenticated conversation-sync routes.
- Research web search uses the server-side `TAVILY_API_KEY` secret when configured.
- GitHub Actions in the core repository installs from the lockfile, prints a full dependency-audit summary, enforces a production dependency audit gate, runs TypeScript checking, and builds with a CI-only AppDeploy client shim.
- The CI shim is **not** a functional replacement for hosted authentication or image processing. Never deploy a bundle built with that shim.
- No release tag has been created in the core repository. Hosted QA, account-isolation checks, security/privacy review, and full-tree development-tool audit disposition remain open.
- Scheduled jobs, recurring workflows, voice, and general third-party integrations are not current app capabilities.
- AppDeploy provides native cron and database primitives, but the jobs runtime remains unimplemented; see the proposed runtime ADR before making an implementation decision.

See [core source and readiness notes](https://github.com/looloofy69-sketch/EL-Rouji/tree/main) before making implementation claims. Documentation is not evidence that an item has passed a live test.
