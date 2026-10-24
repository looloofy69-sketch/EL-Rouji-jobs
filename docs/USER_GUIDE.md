# User Guide

This guide describes the current EL-Rouji interface documented by the source repository. Exact labels may vary as the UI changes.

## 1. Start a conversation

1. Open the [hosted EL-Rouji application](https://el-rouji-70120b.v2.appdeploy.ai/).
2. Choose a language from the interface selector: English, French, or Arabic.
3. Pick a mode if the task benefits from one, or stay in Assistant mode.
4. Enter your message and send it. Use Shift+Enter for a new line where supported.
5. Review the response and continue the conversation as needed.

For important, current, legal, medical, financial, or otherwise high-impact questions, verify information using appropriate authoritative sources or qualified professionals.

## 2. Choose a mode

| Mode | Recommended use |
|---|---|
| Assistant | General questions and everyday/professional help |
| Research | Current or source-dependent questions; inspect the returned source links |
| Writing | Drafting, editing, summarizing, rewriting, and translation |
| Study | Step-by-step explanations and learning support |
| Coding | Code explanations, debugging, and refactoring advice |
| Planning | Breaking a goal into ordered actions and milestones |

Modes guide the AI's response. They do not guarantee factual accuracy or provide additional permissions by themselves.

## 3. Use built-in tools

The Built-in Tools hub contains prompt-based task starters for writing, translation, summarization, flashcards, code help, research, file/image reading, and planning. A tool may prepare a prompt in the composer; review and edit it before sending. The current AppDeploy QA scenario explicitly expects the translator to prefill a prompt without automatically submitting it.

The calculator evaluates supported mathematical expressions locally in the browser. Check the result before using it for consequential decisions.

## 4. Research and source links

Research mode requires working server-side search configuration for general queries. The backend expects `TAVILY_API_KEY` in AppDeploy's secure server settings. Users should not enter that secret into chat or browser settings.

A successful response may include numbered source references and source cards. Open the links and check whether each source actually supports the claim. If research is unavailable, retry later or provide a public page URL if the UI supports that path. A green connection indicator or configured key does not prove live research is operational.

## 5. Attachments

The current supported formats are:

- Images: PNG, JPEG/JPG, WebP
- Text: TXT, Markdown (MD), CSV, JSON

For text files, the backend accepts up to 100,000 characters. Images and questions have separate size limits. A file can be rejected if its extension, MIME type, content, or size is invalid.

PDF, DOCX, XLSX, and other Office-document formats are not supported by the documented current upload flow. Do not rename a PDF to `.txt` and expect the content to be parsed correctly.

## 6. Conversation history

### Local history

The app stores local conversation history in browser storage on the current device/browser. This is convenient but should not be treated as account-wide storage. Anyone who can use the same browser profile may be able to access local history.

Use the app's export/copy/delete/clear-history controls as appropriate. Before clearing, check the confirmation prompt. Clearing browser history should not be assumed to delete separate cloud-synced copies.

### Optional account sync

The interface offers sign-in and cloud sync. When using it:
- Sign in only through the expected hosted flow.
- Verify the interface indicates whether sync is active.
- After syncing, reload and confirm the test conversation remains available.
- Sign out before handing a shared device to another person.
- Do not store highly sensitive information until retention, ownership isolation, and deletion behavior have been verified.

These are operational recommendations, not a claim that every account flow has passed live QA.

## 7. Errors and troubleshooting

- **Assistant unavailable:** retry once, check the hosted status, and avoid repeatedly submitting a sensitive or expensive request.
- **Research not configured:** the backend may be missing a valid server-side search key. This is an operator configuration issue, not something to fix by pasting a key into chat.
- **Attachment rejected:** check the supported list, the actual file type, and size.
- **History not syncing:** check sign-in status and try local history independently. Do not use another person's account to debug.
- **Arabic layout looks wrong:** record device/browser width, selected language, and a minimal reproduction with synthetic text for QA.

## 8. Safe use

Treat AI responses as assistance rather than a guarantee. Do not submit passwords, API keys, unnecessary personal information, or private work material unless the data-handling arrangements are appropriate and understood. Web pages and uploaded files can contain misleading content; EL-Rouji's backend is designed to treat their contents as untrusted data, but users should still evaluate the result critically.
