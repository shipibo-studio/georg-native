# GeorgNative

**On-device Apple intelligence for JavaScript desktop apps.** GeorgNative is a small Swift library with a C interface that lets apps built on JavaScript runtimes call macOS frameworks they cannot reach on their own: Apple Intelligence, text recognition, embeddings, speech transcription and system memory pressure. Everything runs on the Mac, offline, with no account and no telemetry.

It is built first for [tinyjs](https://tinyjs.app) (txiki.js + the system WebView) and for [Georg](https://dasgeorg.cc), a macOS app to orchestrate coding agents, but the interface is plain C with JSON in and JSON out, so any runtime with a foreign function interface can use it.

> [!NOTE]
> **Status: design stage.** There is no code yet. The architecture and the API are drafts, and this is the best moment to shape them. See [How to help](#how-to-help).

## Why

JavaScript desktop runtimes that render in the system WebView are small and fast, but they stop at the edge of what JavaScript can call. The most useful on-device features of macOS live in Swift-only frameworks:

| Capability | Apple framework | Example use |
| --- | --- | --- |
| Text generation, summaries, titles, classification | FoundationModels (Apple Intelligence) | Title a conversation, summarize a long log |
| Text recognition in images and documents | Vision | Turn a screenshot or a scanned PDF into Markdown |
| Text from PDFs | PDFKit | Read the text layer of a PDF before falling back to OCR |
| Sentence embeddings, language detection | NaturalLanguage | Semantic search over local notes |
| Speech to text | Speech (SpeechAnalyzer) | Dictation in a text field |
| Memory pressure | Dispatch | Release caches before the system starts swapping |

GeorgNative wraps them behind one small, stable, documented interface, so an app only needs to load one library.

## Principles

- **On-device only.** No network access, no accounts, no telemetry. Data never leaves the Mac.
- **Small and boring interface.** A handful of C functions that exchange JSON. No Swift or Objective-C knowledge needed to use it.
- **Capability detection first.** Every feature reports whether it is available on this Mac and why not (OS version, hardware, Apple Intelligence turned off, model still downloading), so apps can degrade gracefully.
- **Untrusted input, always.** Paths, sizes and durations are validated; long work has time limits; external documents are parsed in an isolated helper. See [`SECURITY.md`](SECURITY.md).
- **Model output is data, not instructions.** Generation is guided and has no tool calling with side effects.

## Planned capabilities

| Module | First version | Requires |
| --- | --- | --- |
| `system` | Version, capability report, memory pressure events | macOS 26 |
| `intelligence` | Guided generation with a JSON schema, summaries, titles | macOS 26 on a Mac with Apple Intelligence enabled |
| `vision` | Text recognition in images; documents (tables, lists) to Markdown | macOS 26 |
| `pdf` | Text layer of a PDF, with OCR fallback per page | macOS 26 |
| `language` | Contextual embeddings, language detection | macOS 26 |
| `speech` | Transcription of an audio file; live dictation later | macOS 26 |

macOS 26 is the floor because FoundationModels, `RecognizeDocumentsRequest` and `SpeechAnalyzer` arrived there. Earlier versions are out of scope for now.

## How it will look

The full proposal is in [`docs/api.md`](docs/api.md). In short, an app submits a JSON request and receives a JSON result:

```json
{ "id": "42", "method": "vision.recognizeText", "params": { "path": "/Users/me/Desktop/shot.png", "languages": ["es", "en"] } }
```

```json
{ "id": "42", "ok": true, "result": { "text": "…", "confidence": 0.97 } }
```

From tinyjs, through its FFI, that becomes a short wrapper. How results come back to JavaScript without blocking (callbacks, polling or a notification file descriptor) is one of the [open design questions](docs/api.md#open-questions).

## Documentation

- [`docs/architecture.md`](docs/architecture.md): how the library is built, threading, memory ownership, errors and versioning.
- [`docs/api.md`](docs/api.md): the proposed C interface and every request, with open questions.
- [`docs/roadmap.md`](docs/roadmap.md): milestones from the first release to later capabilities.
- [`SECURITY.md`](SECURITY.md): threat model and how to report a vulnerability.

## How to help

This project is open to contributions from the start, and the design stage is where help matters most.

- **Review the API.** Read [`docs/api.md`](docs/api.md) and comment on the issues labeled [`design`](https://github.com/shipibo-studio/georg-native/labels/design).
- **Share your runtime.** If you build on Bun, Deno, Node (koffi), Electron or Tauri and would use this, tell us in [Discussions](https://github.com/shipibo-studio/georg-native/discussions) what you need from the FFI side.
- **Know the Apple frameworks?** FoundationModels, Vision and SpeechAnalyzer are new; experience with their edge cases is very welcome.
- **Pick an issue** labeled [`help wanted`](https://github.com/shipibo-studio/georg-native/labels/help%20wanted) or [`good first issue`](https://github.com/shipibo-studio/georg-native/labels/good%20first%20issue).

Read [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening a pull request, and follow the [Code of Conduct](CODE_OF_CONDUCT.md).

## Contact

| For | Where |
| --- | --- |
| Questions, ideas, how you would use it | [Discussions](https://github.com/shipibo-studio/georg-native/discussions) |
| Bugs and feature proposals | [Issues](https://github.com/shipibo-studio/georg-native/issues/new/choose) |
| Security vulnerabilities (never in public) | [Private advisory](https://github.com/shipibo-studio/georg-native/security/advisories/new) or **security@dasgeorg.cc** |
| Privacy | **privacy@dasgeorg.cc** (the library collects no data; ask anything about how it handles yours) |
| Code of conduct reports | **oi@dasgeorg.cc** |
| Anything else | **support@dasgeorg.cc** |

## Related projects

- [tinyjs](https://tinyjs.app): desktop apps in about 6 MB with a JavaScript backend and the system WebView.
- [georg-mail-oss](https://github.com/shipibo-studio/georg-mail-oss): a self-hosted mail client for your own domain on Cloudflare Workers, also part of the georg family by Shipibo Studio.

## License

[MIT](LICENSE) © 2026 Shipibo Studio. Apple, Apple Intelligence and macOS are trademarks of Apple Inc. This project is not affiliated with or endorsed by Apple. Apps that use Apple Intelligence through this library must follow Apple's own terms for those features.
