# System Architecture

**Baseline date:** 24 October 2026  
**Source repository:** [looloofy69-sketch/EL-Rouji](https://github.com/looloofy69-sketch/EL-Rouji)

## 1. Logical view

```text
Browser
  ├─ React + TypeScript UI (src/App.tsx)
  │    ├─ Language/mode state and chat composer
  │    ├─ Local conversation history (browser storage)
  │    ├─ Optional account sync controls
  │    ├─ Attachment selection and response/source rendering
  │    └─ Built-in task hub (src/components/BuiltInTools.tsx)
  ├─ AppDeploy client SDK
  │
  ▼
AppDeploy-hosted backend (backend/index.ts)
  ├─ /api/chat ─────────────► AppDeploy AI generation
  ├─ /api/research ─────────► Tavily search and/or public-page scrape + AI synthesis
  ├─ /api/analyze-image ────► AI image analysis
  ├─ /api/analyze-document ─► AI analysis of supplied text
  ├─ /api/conversations/* ──► authenticated AppDeploy storage
  └─ /api/health ───────────► capability/configuration status
```

This is a logical diagram derived from the source; it is not a deployment topology diagram or proof that every hosted integration was live-tested.

## 2. Source layout in the core repository

| Path | Responsibility |
|---|---|
| `src/main.tsx` | React entry point |
| `src/App.tsx` | Main chat UI, modes/languages, browser history, sync UI, research and attachment flows |
| `src/components/BuiltInTools.tsx` | Built-in task prompts and local calculator |
| `src/index.css` | Responsive styling and RTL layout rules |
| `backend/index.ts` | API router, AI integration, research, input validation, and cloud conversation storage |
| `package.json` / `package-lock.json` | Scripts and locked dependency graph |
| `vite.config.ts` | Vite config and CI-only AppDeploy client alias |
| `.github/workflows/ci.yml` | GitHub Actions install/audit/typecheck/build pipeline |
| `tests/tests.json` | AppDeploy-style QA scenarios; not an npm test suite |

## 3. Frontend and state

- The UI uses React 19, TypeScript, Vite, Lucide icons, React Markdown, and GitHub-Flavored Markdown support.
- UI language values are `en`, `fr`, and `ar`; assistant modes are `assistant`, `research`, `writing`, `study`, `coding`, and `planning`.
- Browser-local storage keys in the source are `elrouji.conversations.v1`, `elrouji.language.v1`, and `elrouji.active.v1`.
- Local history and account-synced history are separate mechanisms. Do not infer that local deletion automatically deletes a cloud record, or vice versa; verify the actual UI and API behavior.
- Attachments travel through the current browser-to-backend flow. Only PNG/JPEG/WebP and TXT/MD/CSV/JSON are documented as supported.

## 4. Backend behavior

The current backend is implemented with AppDeploy SDK imports for AI generation, routing, authentication, secrets, JSON responses, errors, and storage. It is not a standalone Express server.

The current route set is:

- `GET /api/health`
- `POST /api/chat`
- `POST /api/research`
- `POST /api/analyze-image`
- `POST /api/analyze-document`
- `GET /api/conversations`
- `PUT /api/conversations/:id`
- `DELETE /api/conversations/:id`

Conversation endpoints explicitly use `requireAuth()`. The other listed endpoints do not show that middleware in the current source. Confirm whether the platform adds any relevant edge protection, then assess exposure, abuse limits, quotas, and billing before public-scale use.

Conversation records are stored under a user-scoped path based on the authenticated user ID. IDs and payload fields are validated, and the backend caps selected message, source, and record sizes. These checks do not substitute for testing cross-account authorization and deletion semantics against the hosted runtime.

## 5. Research data flow

1. The client sends a research query and selected language.
2. If the query is a public HTTP(S) URL, the backend attempts to scrape and summarize that page.
3. Otherwise, the backend reads `TAVILY_API_KEY` from server-side secrets and requests up to six search results.
4. The backend passes selected result excerpts to the AI service as untrusted source material.
5. The response includes a synthesized reply and source metadata; the UI presents source cards.

A missing key leads to a configuration error. A present key does not prove the provider accepts it or that search succeeds. Research results and web-page content are untrusted input, not instructions.

## 6. Build and deployment boundary

The core repository runs CI outside AppDeploy by selecting `ci/appdeploy-client-shim.ts` when `EL_ROUJI_CI_BUILD=1`. That shim allows source bundling but does not implement hosted sign-in or real image resizing. **Never deploy a CI-shim bundle.**

AppDeploy's current validator rejected the tested Tailwind CSS 4 CSS entry. Tailwind CSS 3 is the documented compatible baseline at this snapshot. Deploy through the AppDeploy workflow and record both the Git commit and the AppDeploy version.

## 7. Jobs repository boundary

This repository (`EL-Rouji-jobs`) is documentation-only at this baseline. No worker, scheduler, queue, persistent job model, periodic runner, or job API was present when the repository was inspected. See [Jobs Design Specification](JOBS_DESIGN_SPEC.md) for future requirements.

## 8. Architecture questions still requiring verification

- Is each non-conversation API route adequately protected against abuse, unexpected cost, and excessive request concurrency?
- Are hosted authentication, account isolation, and cloud deletion correct end to end?
- Are file size limits and MIME/content validation consistent across UI and backend?
- Are provider errors, logs, retention, and data deletion adequate for the intended users?
- Is the deployed artifact exactly traceable to a reviewed source commit and build?
