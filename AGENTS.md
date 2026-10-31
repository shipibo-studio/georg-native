# AGENTS.md

Instructions for AI coding agents working in this repository.

## Project

**GeorgNative** is an open source (MIT) Swift library with a C interface that gives JavaScript desktop runtimes (tinyjs first, any runtime with an FFI) access to on-device macOS frameworks: FoundationModels (Apple Intelligence), Vision, PDFKit, NaturalLanguage, Speech and memory pressure. JSON in, JSON out.

- **Status: design stage.** There is no code yet. `docs/architecture.md` and `docs/api.md` are proposals; do not implement until the user asks.
- **Public repository.** Everything committed here is public: no secrets, no private paths, no personal data, no internal notes from other repositories.

## Related repositories

Expected local layout: sibling directories under the same parent folder.

| Repository | Local path | Relationship |
| --- | --- | --- |
| georg-app (private) | `../georg-app` | **Georg**, the first consumer. Its plan calls this layer GeorgNative (Stage 0) |
| georg-use (private) | `../georg-use` | tinyjs menu bar app and tinyjs testbed; a possible second consumer |
| georg-ds (private) | `../georg-ds` | Design system of the family. Not used here (no UI) |

Never copy content from the private repositories into this one.

## Standards

- **English only** for docs, comments, commit messages and identifiers. See [`CONTRIBUTING.md`](CONTRIBUTING.md).
- **Clean Code**, **Conventional Commits** and a `CHANGELOG.md` entry for every user-visible change.
- **Stable C interface:** any change to `gn_*` functions or existing request and result fields needs a design issue first and a matching update to `docs/api.md`.

## Security rules

- **No network access** in the library and no telemetry, ever.
- **Every input is untrusted:** validate JSON size and shape, resolve paths with `realpath`, accept only regular files, enforce deadlines and size limits.
- **External documents are parsed in the isolated helper**, never in the host process.
- **Model output is data:** no tool calling with side effects.
- **Never log user content**, and keep it out of error messages.
- Do not commit secrets, certificates, signing identities or `.env` files.

## Testing

- Unit and integration tests for every change; a feature or fix without tests is not done. Bug fixes start with a test that reproduces the bug.
- Integration tests for capabilities that are unavailable on the machine are skipped with a clear message, never silently passed.
