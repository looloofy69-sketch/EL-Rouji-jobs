# Product Overview

**Baseline date:** 24 October 2026  
**Primary source:** [EL-Rouji repository](https://github.com/looloofy69-sketch/EL-Rouji)

## 1. Product purpose

EL-Rouji is a general-purpose AI assistant interface designed to work with Arabic, Algerian Darija, French, and English. Its UI language selector currently offers English, French, and Arabic; when users write in Algerian Darija, the assistant prompt allows a Darija response when appropriate.

The product combines ordinary chat with task-specific modes, built-in prompt starters, research with source links, and selected image/text-file analysis.

## 2. Current capabilities

| Capability | Current behavior | Important qualification |
|---|---|---|
| Assistant chat | Sends the recent conversation context to the hosted AI generation service | Server currently accepts up to 16 messages per request |
| Research | Queries Tavily and synthesizes returned source excerpts, or reads a supplied public HTTP(S) page | Search requires a valid server-side `TAVILY_API_KEY`; configured does not mean verified |
| Writing | Helps draft, revise, summarize, and translate through mode instructions and built-in prompts | It is an AI prompt workflow, not a separate document editor |
| Study | Uses step-by-step teaching instructions and can prepare flashcards | Output quality depends on the model and supplied context |
| Coding | Provides code explanation, debugging, and refactoring prompts | No claim of a sandboxed code execution environment |
| Planning | Helps break goals into steps and milestones | No persistent task scheduler is included |
| Built-in tool hub | Includes writing studio, translator, summarizer, study cards, code helper, web research, file/image reader, and plan builder prompts | Most are prompt-based workflows, not independent microservices |
| Calculator | Evaluates arithmetic with Math.js in the browser | Requires security QA for untrusted expressions and resource limits |
| Local conversation history | Saves conversation data in browser-local storage | Specific to that browser/device; not equivalent to account cloud sync |
| Optional account sync | Authenticated API reads/writes/deletes account-scoped conversation records | Live account isolation and deletion behavior still require hosted testing |
| Image analysis | Accepts PNG, JPEG, or WebP image data | Size and request constraints apply |
| Text-file analysis | Accepts TXT, MD, CSV, and JSON content | Maximum file content is 100,000 characters |
| Language/RTL layout | English, French, and Arabic UI with RTL support for Arabic | Desktop/mobile and mixed-script QA remains a release requirement |

## 3. Explicitly unsupported or not yet implemented

The following must not be presented as existing features:

- Scheduled jobs, recurring execution, a job queue, worker processes, and daily automated loops.
- Voice input/output.
- PDF, DOCX, XLSX, or general Office-document analysis.
- CRM, payment, or other third-party integrations not described in the source.
- General-purpose code execution or a secure execution sandbox.
- Formal privacy certification, a published SLA, 24/7 on-call operations, automatic backups, or complete observability.
- A completed independent application security audit.

The separate `EL-Rouji-jobs` repository may be used to specify future job-processing work, but such a specification is not evidence of implementation.

## 4. Users and primary journeys

1. **Ask a question:** choose a mode, type a message, receive a response, and continue the conversation.
2. **Use a task starter:** select a built-in tool, review/edit its prepared prompt, and explicitly send it.
3. **Research a topic:** submit a focused question or public page URL, review the answer and source links, then verify important claims at the linked sources.
4. **Analyze a file:** select a supported image/text format, ask a question, and review the result or an actionable error.
5. **Manage history:** use local history, export/copy/delete a conversation, or sign in and test account-based sync where configured.

## 5. Current delivery status

The source repository has a committed lockfile, TypeScript/Vite CI checks, and a production dependency-audit gate. The core README reports that the latest recorded production-only audit passed with zero findings in CI run #67, while the full dependency-tree audit reported three moderate development/build-tool findings. That record is historical evidence from the noted run, not a guarantee about a future install.

The product is **not documented as release-ready**: hosted core-flow QA, live research verification, account isolation, data deletion/retention decisions, security review, and triage of full-tree audit findings remain open. No release tag is asserted.

## 6. Sources of truth

- [Core README](https://github.com/looloofy69-sketch/EL-Rouji/blob/main/README.md)
- [Architecture notes](https://github.com/looloofy69-sketch/EL-Rouji/blob/main/docs/ARCHITECTURE.md)
- [Next-mile readiness](https://github.com/looloofy69-sketch/EL-Rouji/blob/main/docs/NEXT_MILE_READINESS.md)
- [QA test plan](https://github.com/looloofy69-sketch/EL-Rouji/blob/main/docs/QA_TEST_PLAN.md)
