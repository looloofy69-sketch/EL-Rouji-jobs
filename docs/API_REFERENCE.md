# Backend API Reference

This reference is based on `backend/index.ts` in the core [EL-Rouji source repository](https://github.com/looloofy69-sketch/EL-Rouji/blob/main/backend/index.ts). Routes and validation can change; update this document with code changes.

**Base path:** the deployed application API base URL plus the paths shown below.  
**Content type:** JSON request/response bodies unless otherwise stated.

## Conventions

- Languages: `en`, `fr`, `ar`. Unknown language values default to English.
- Conversation roles: `user`, `assistant`.
- The backend's error helper returns a JSON error response; exact envelope/headers are controlled by AppDeploy and should be confirmed in live integration tests.
- The absence of explicit `requireAuth()` in a handler below means this source file does not show route-level auth middleware. It does not by itself prove the route is reachable without any platform-level protection.

## 1. Health

### `GET /api/health`

Reports basic service identity and whether the research secret name is configured.

Example response:

```json
{
  "ok": true,
  "name": "EL-Rouji",
  "service": "core-assistant",
  "capabilities": {
    "webResearch": false
  }
}
```

`webResearch` is derived from whether the secure secret list contains `TAVILY_API_KEY`. It is **not** an end-to-end test of the key, provider access, or search results.

## 2. Chat

### `POST /api/chat`

Sends recent messages to AI generation.

Request:

```json
{
  "language": "en",
  "mode": "assistant",
  "messages": [
    { "role": "user", "content": "Explain recursion with an example." }
  ]
}
```

Accepted modes: `assistant`, `research`, `writing`, `study`, `coding`, `planning`.

Limits enforced in source:
- At least one message is required.
- Only the last 16 submitted messages are processed.
- Each message must contain 1–6,000 characters.
- Combined processed content may not exceed 22,000 characters.
- The last processed message must be from the user.
- Message bodies must be JSON objects with a valid role and string content.

Success response:

```json
{
  "reply": "Recursion is ...",
  "assistant": "EL-Rouji"
}
```

Typical errors include malformed input (400), oversized requests (413), provider rate/usage limit (429), or temporary AI failure (502). The source does not attach `requireAuth()` to this route; verify perimeter controls and abuse/rate limits.

## 3. Research

### `POST /api/research`

Performs web search and synthesis, or summarizes a supplied public HTTP(S) URL.

Request:

```json
{
  "query": "What changed in the latest release?",
  "language": "en"
}
```

Validation and behavior:
- Query must contain 1–400 characters.
- If the query parses as an allowed public HTTP(S) URL, the backend scrapes the page and summarizes extracted text.
- Otherwise it reads `TAVILY_API_KEY` from AppDeploy server-side secrets and calls Tavily.
- Search requests ask for at most six results.
- Retrieved excerpts and page text are treated as untrusted content.
- Returned sources include title, URL, domain, snippet, and optional publication date.

Success response shape:

```json
{
  "reply": "A source-grounded synthesis ...",
  "sources": [
    {
      "title": "Source title",
      "url": "https://example.com/article",
      "domain": "example.com",
      "snippet": "Relevant returned excerpt"
    }
  ]
}
```

Likely status classes: 400 invalid query, 503 missing research configuration, 429 provider rate limit, and 502 upstream scrape/search/synthesis failure. The URL filter rejects common localhost and private-network address ranges; it is not a substitute for a full SSRF/security review. Verify redirect and DNS behavior at the hosted boundary.

## 4. Image analysis

### `POST /api/analyze-image`

Analyzes a user-provided image.

Request:

```json
{
  "filename": "photo.png",
  "mimeType": "image/png",
  "imageData": "BASE64_ENCODED_IMAGE_BYTES",
  "question": "What text is visible?",
  "language": "en"
}
```

- Accepted MIME values: `image/png`, `image/jpeg`, `image/webp`.
- `imageData` is base64 text; the source rejects an empty value, malformed base64 characters, and values over 8,500,000 characters.
- Question length must not exceed 4,000 characters.
- Success returns `{ "reply": "...", "analyzed": true }`.
- Typical errors: 400 invalid type/body, 413 invalid/oversized image payload, 429 usage limit, 502 analysis failure.

The current client may use the AppDeploy-provided image helper. The CI shim is not a valid runtime test for image resizing.

## 5. Text-document analysis

### `POST /api/analyze-document`

Analyzes supported text-file content supplied in the request.

Request:

```json
{
  "filename": "notes.md",
  "content": "# Notes\nMain point: ...",
  "question": "Summarize the key points.",
  "language": "en"
}
```

- Supported filename extensions: `.txt`, `.md`, `.csv`, `.json`.
- Content must be non-empty and at most 100,000 characters.
- Question length must not exceed 4,000 characters.
- Success returns `{ "reply": "...", "analyzed": true }`.
- Typical errors: 400 unsupported format/body, 413 empty or oversized content, 429 usage limit, 502 analysis failure.

PDF, DOCX, XLSX, and binary Office formats are not accepted by this route.

## 6. Account conversation listing

### `GET /api/conversations`

**Authentication:** the source applies `requireAuth()`.

Lists up to 40 valid conversation records for the authenticated user, sorted newest-first.

Success response:

```json
{
  "conversations": [
    {
      "id": "conversation_123",
      "title": "Example",
      "updatedAt": 1792755107550,
      "messages": []
    }
  ]
}
```

Malformed stored records are discarded during parsing. Exact AppDeploy storage behavior and auth error envelopes must be tested on the hosted service.

## 7. Create/update a synced conversation

### `PUT /api/conversations/:id`

**Authentication:** the source applies `requireAuth()`.

The path ID must match the record's `id`.

Request shape:

```json
{
  "id": "conversation_123",
  "title": "Example",
  "updatedAt": 1792755107550,
  "messages": [
    {
      "id": "message_1",
      "role": "user",
      "content": "Hello",
      "createdAt": 1792755000000
    },
    {
      "id": "message_2",
      "role": "assistant",
      "content": "Hello!",
      "createdAt": 1792755001000,
      "sources": []
    }
  ]
}
```

Source-level limits:
- Conversation and message IDs must match the safe identifier pattern `[A-Za-z0-9_-]{1,80}`.
- At most 160 messages per record.
- Message content must be non-empty and at most 6,000 characters.
- At most eight source links are retained per message.
- Title is trimmed and capped at 120 characters.
- Serialized record may not exceed 230,000 characters.
- Invalid structure returns 400; message-count/record-size limits return 413.

Success response:

```json
{
  "saved": true,
  "id": "conversation_123",
  "updatedAt": 1792755107550
}
```

## 8. Delete a synced conversation

### `DELETE /api/conversations/:id`

**Authentication:** the source applies `requireAuth()`.

Deletes the matching record from the authenticated user's storage prefix after validating the ID.

Success response:

```json
{ "deleted": true }
```

A false value means the storage operation did not report the record as deleted. The backend response must be tested alongside the client's user-facing behavior. Do not infer a complete retention policy from this endpoint alone.

## 9. API test checklist

- Validate missing, wrong-type, empty, and oversized request fields.
- Verify source links reject unsafe schemes and private-network destinations.
- Verify only the account owner can read, update, or delete a conversation.
- Confirm all public-facing routes have appropriate rate/cost controls at the hosted edge.
- Verify no stack traces, secrets, or sensitive provider payloads are returned to clients.
- Verify the deployed route behavior against the exact source commit before a release.
