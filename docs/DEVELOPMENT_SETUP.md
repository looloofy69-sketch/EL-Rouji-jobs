# Developer Setup and Build Notes

**Primary code repository:** [looloofy69-sketch/EL-Rouji](https://github.com/looloofy69-sketch/EL-Rouji)  
**This repository:** documentation-only at the baseline date.

## 1. Requirements

The core GitHub Actions workflow uses Node.js 22 and installs from the committed `package-lock.json` using `npm ci`. Use the same Node major version for the closest CI match.

The core application uses React, TypeScript, Vite, Tailwind CSS 3, PostCSS, Autoprefixer, React Markdown, Remark GFM, Math.js, and Lucide React. The production backend relies on AppDeploy-provided SDK modules.

## 2. Obtain source

```sh
git clone https://github.com/looloofy69-sketch/EL-Rouji.git
cd EL-Rouji
```

Install exact locked dependencies:

```sh
npm ci
```

Type-check:

```sh
npm run typecheck
```

The package scripts also define `dev`, `build`, and `preview`. Runtime behavior outside AppDeploy needs care because `@appdeploy/client` is supplied by the hosted platform rather than being a normal package dependency in the manifest.

## 3. Repository CI build

The core repository's documented CI build command is:

```sh
EL_ROUJI_CI_BUILD=1 npm run build
```

GitHub Actions currently performs:
1. Checkout.
2. Set up Node.js 22.
3. `npm ci`.
4. Full dependency-audit summary for triage.
5. Blocking production dependency audit: `npm audit --omit=dev --audit-level=low`.
6. TypeScript check.
7. Vite build with the CI-only AppDeploy client shim.

A green CI run proves dependency installation, type-checking, and bundle generation under that CI configuration. It does not prove hosted sign-in, image resizing, Tavily search, backend authorization, or UI quality.

## 4. Critical CI shim restriction

When `EL_ROUJI_CI_BUILD=1`, Vite aliases `@appdeploy/client` to `ci/appdeploy-client-shim.ts`. The shim exists to build outside AppDeploy. Its authentication methods are intentionally unavailable and its image helper is not the hosted implementation.

**Never deploy the output of a CI-shim build.** Use the AppDeploy deployment/source workflow so the platform supplies the real client SDK and backend runtime.

## 5. Configuration and secrets

Research reads `TAVILY_API_KEY` from AppDeploy's secure server-side secret store. Keep all provider credentials in server-side secret settings.

Do not:
- Commit credentials or `.env` files.
- Expose backend secrets in frontend variables, screenshots, test fixtures, issue reports, or logs.
- Treat the health endpoint's `webResearch` flag as an end-to-end test.
- use local browser storage to hold service credentials.

After configuring a key, verify a harmless research query against the hosted app without logging or revealing the key.

## 6. Tests

There is no `test` script in the current core `package.json`. The file `tests/tests.json` is an AppDeploy QA specification; it is not a standalone automated npm test suite. Use the source repo's [QA test plan](https://github.com/looloofy69-sketch/EL-Rouji/blob/main/docs/QA_TEST_PLAN.md) for hosted manual checks.

When behavior changes, add appropriate automated tests where the environment permits, update the QA specification, run CI, and perform hosted smoke tests for platform-dependent behavior.

## 7. CSS compatibility

Keep the current Tailwind CSS 3 baseline until AppDeploy supports a validated alternative. A Tailwind CSS 4 attempt using `@import "tailwindcss";` was rejected by the current AppDeploy template validator and rolled back.

## 8. Where to make changes

- Application UI or backend: core `EL-Rouji` repository.
- Documentation and future jobs design: this `EL-Rouji-jobs` repository.
- Do not copy source into this repository or implement a new worker implicitly. Establish a scoped implementation plan and explicit interface contract first.
