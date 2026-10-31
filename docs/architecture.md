# Architecture

> [!NOTE]
> Draft. Nothing here is implemented yet. Each decision lists its reasons so it can be challenged; open points are tracked in [`api.md`](api.md#open-questions) and in issues labeled `design`.

## Shape

```
┌────────────────────────────┐        ┌──────────────────────────────────────────┐
│ Host app (JavaScript)      │  FFI   │ libgeorgnative.dylib (Swift)             │
│ tinyjs / txiki.js, Bun,    │ ─────► │  C interface (gn_*)                      │
│ Deno, Node (koffi), …      │  JSON  │   ├─ dispatcher (request id, deadlines)  │
│                            │ ◄───── │   ├─ modules: system, intelligence,      │
└────────────────────────────┘        │   │  vision, pdf, language, speech       │
                                      │   └─ isolated helper for documents ──┐   │
                                      └──────────────────────────────────────│───┘
                                                                             ▼
                                                       georg-native-helper (XPC or
                                                       child process, sandboxed)
```

- **One dynamic library**, `libgeorgnative.dylib`, built from a Swift Package. The host app copies it into its bundle (`Contents/Frameworks/`) and signs it with its own identity, so the library runs with the app's identity and permissions (microphone, speech recognition).
- **A C interface** exported with `@_cdecl`. C is the one ABI every FFI understands; Swift and Objective-C interfaces are not stable across compilers.
- **JSON in and out.** One request format for every capability keeps the C surface tiny and lets the API grow without new symbols. The cost (encoding) is negligible next to the work being done (model inference, OCR).

## Requests and results

- Each request carries an `id` chosen by the host, a `method` (`module.action`) and `params`.
- Work is **asynchronous**: the call returns at once and the result arrives later, so a long OCR or generation never blocks the host's event loop.
- Every request has a **deadline** (a default per method, which the host can lower) and can be **cancelled** by id.
- Results are **one JSON object per request**: `{ "id", "ok": true, "result" }` or `{ "id", "ok": false, "error": { "code", "message" } }`. Streaming methods (generation, dictation) send several `progress` messages before the final result.

## Threading

- Swift concurrency inside, one task per request, with a cap on concurrent heavy jobs (OCR, generation) to avoid saturating the Mac.
- The C functions are **thread-safe** and never call back into the host on a thread the host did not expect: results are queued and the host collects them (see the delivery options in [`api.md`](api.md#open-questions)).

## Memory ownership

- Strings passed in are **borrowed**: the library copies what it needs before the call returns.
- Strings returned are **owned by the caller** and released with `gn_free`. No other function frees memory.
- No pointers into Swift objects ever cross the boundary.

## Errors

Errors have a stable `code` from a closed set, so hosts can branch on them, and a human `message` that may change:

| Code | Meaning |
| --- | --- |
| `invalid_request` | Malformed JSON, unknown method or invalid parameter |
| `unavailable` | The capability is not available on this Mac (`reason` says why) |
| `permission_denied` | The user has not granted access (microphone, speech recognition) |
| `not_found` | A path does not exist or is not readable |
| `too_large` | Input over the documented limit |
| `timeout` | The deadline passed |
| `cancelled` | The host cancelled the request |
| `internal` | A bug; reported with no user data in the message |

## Capability detection

`system.capabilities` returns, for each method, `available: true|false` and, when false, a `reason` (`os_version`, `hardware`, `apple_intelligence_disabled`, `model_not_ready`, `permission`). Hosts call it at startup and when the app becomes active, because Apple Intelligence can be turned on or finish downloading while the app runs.

## Documents and the isolated helper

PDFs and images from outside the app are untrusted. Parsing them happens in a **separate helper process** with a restrictive sandbox (no network, read access only to the file it was given, no writes), a time limit and a memory limit. If the helper crashes, the request fails with `internal` and the host is unaffected.

## Versioning

- **Semantic versioning** for releases.
- `gn_abi_version()` returns an integer that changes only when the C interface changes incompatibly. Hosts check it at load time and refuse to run with a mismatch.
- New methods and new optional fields are minor releases. Removing or renaming them is a major release, announced in [`CHANGELOG.md`](../CHANGELOG.md).

## Building and distribution

- Swift Package Manager, Swift 6 language mode with strict concurrency.
- Releases publish a universal `.dylib` and the helper, with checksums, built by CI from a tag. Hosts sign them inside their own bundle; the project does not ship signed binaries that apps would trust blindly.
- No third-party dependencies in the shipped library.

## Testing

- Unit tests for request parsing, validation and error mapping (hostile input included: huge strings, invalid UTF-8, paths with `..` and symlinks).
- Integration tests per module, skipped with a clear message when the capability is unavailable on the test machine.
- A small C host and a tinyjs example that exercise the real interface end to end.
