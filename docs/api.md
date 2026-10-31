# API (proposal)

> [!NOTE]
> Draft for discussion. Names, fields and limits can change before 0.1.0. Comment on the issues labeled `design`, or open a discussion.

## C interface

```c
// Version of the C interface. Changes only on incompatible changes.
int32_t gn_abi_version(void);

// Library version, as a semantic version string. Free with gn_free.
char *gn_version(void);

// Submit a request (UTF-8 JSON). Returns 0 when accepted, or a negative
// error code when the request cannot even be queued. The result arrives later.
int32_t gn_submit(const char *request_json);

// Cancel a pending request by the id the host gave it.
void gn_cancel(const char *request_id);

// Collect finished results and progress messages, as a JSON array.
// Returns NULL when there is nothing new. Free with gn_free.
char *gn_poll(int32_t max_messages);

// Release a string returned by the library.
void gn_free(char *pointer);
```

How the host learns that `gn_poll` has something new is an [open question](#open-questions).

## Request

```json
{
  "id": "host-chosen string, unique among pending requests",
  "method": "module.action",
  "params": { },
  "deadline_ms": 30000
}
```

## Result

```json
{ "id": "…", "ok": true, "result": { } }
{ "id": "…", "ok": false, "error": { "code": "unavailable", "message": "…", "reason": "apple_intelligence_disabled" } }
{ "id": "…", "progress": { } }
```

Error codes are listed in [`architecture.md`](architecture.md#errors).

## Methods

### `system`

| Method | Params | Result |
| --- | --- | --- |
| `system.capabilities` | — | `{ "<method>": { "available": bool, "reason"?: string } }` |
| `system.memoryPressure.subscribe` | — | Progress messages `{ "level": "normal" \| "warning" \| "critical" }` until cancelled |

### `intelligence` (FoundationModels)

| Method | Params | Result |
| --- | --- | --- |
| `intelligence.generate` | `instructions`, `prompt`, optional `schema` (JSON Schema subset for guided generation), `maxTokens`, `temperature` | `{ "text" }` or `{ "json" }` when a schema was given; with `stream: true`, progress messages carry partial text |
| `intelligence.summarize` | `text`, `style` (`title`, `oneLine`, `paragraph`) | `{ "text" }` |

No tool calling: the model cannot trigger actions. Output is untrusted data for the host.

### `vision`

| Method | Params | Result |
| --- | --- | --- |
| `vision.recognizeText` | `path` (image), `languages`, `level` (`fast`, `accurate`) | `{ "text", "lines": [{ "text", "box", "confidence" }] }` |
| `vision.document` | `path` (image) | `{ "markdown" }` with paragraphs, lists and tables |

### `pdf`

| Method | Params | Result |
| --- | --- | --- |
| `pdf.text` | `path`, `pages` (range), `ocrFallback` (bool) | `{ "pages": [{ "number", "markdown", "source": "textLayer" \| "ocr" }] }` |

### `language` (NaturalLanguage)

| Method | Params | Result |
| --- | --- | --- |
| `language.detect` | `text` | `{ "language", "confidence" }` |
| `language.embed` | `texts` (array), `language` | `{ "dimensions", "vectors": [[…]] }` |

### `speech`

| Method | Params | Result |
| --- | --- | --- |
| `speech.transcribeFile` | `path` (audio), `locale` | `{ "text", "segments": [{ "text", "start", "end" }] }` |
| `speech.dictate` *(later)* | `locale` | Progress messages with partial text until cancelled |

## Limits (initial values)

| Input | Limit |
| --- | --- |
| Request JSON | 1 MB |
| Text for `intelligence` and `language` | Bounded by the model context; larger input fails with `too_large`, never truncated silently |
| Image | 50 MB, 100 megapixels |
| PDF | 200 pages per request |
| Audio file | 2 hours |
| Concurrent heavy jobs | 2 (configurable) |

## Open questions

1. **Result delivery.** Polling with `gn_poll` works with every FFI, but the host needs to know when to poll. Options: (a) a file descriptor the host watches (a pipe written once per new message), (b) an optional C callback for runtimes that support them safely, (c) both. Which FFIs (txiki.js, Bun, Deno, koffi) handle each option well? Discuss in [#1](https://github.com/shipibo-studio/georg-native/issues/1).
2. **Schema subset for guided generation.** FoundationModels generates into Swift types; mapping a JSON Schema at runtime needs a defined subset (objects, strings, numbers, enums, arrays). Which subset is enough? Discuss in [#2](https://github.com/shipibo-studio/georg-native/issues/2).
3. **Helper process.** XPC service inside the host bundle, or a plain child process? XPC is the macOS way but ties the helper to the bundle layout. Discuss in [#3](https://github.com/shipibo-studio/georg-native/issues/3).
4. **Paths or bytes.** Accept only file paths (simple, no copies) or also base64 data for content the host already has in memory? Discuss in [#4](https://github.com/shipibo-studio/georg-native/issues/4).
5. **Model availability events.** Should capability changes (Apple Intelligence turned on, model downloaded) be pushed as progress messages instead of being polled? Discuss in [#5](https://github.com/shipibo-studio/georg-native/issues/5).
