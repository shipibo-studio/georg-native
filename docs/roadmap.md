# Roadmap

| Milestone | Scope | Status |
| --- | --- | --- |
| **Design** | Architecture and API agreed in [`architecture.md`](architecture.md) and [`api.md`](api.md); open questions closed | In progress |
| **0.1.0: foundations** | Swift Package, C interface, dispatcher, `system.capabilities`, `system.memoryPressure`, CI, a C host example | Planned |
| **0.2.0: text** | `vision.recognizeText`, `vision.document`, `pdf.text` with the isolated helper | Planned |
| **0.3.0: intelligence** | `intelligence.generate` (guided, streaming), `intelligence.summarize` | Planned |
| **0.4.0: language** | `language.detect`, `language.embed` | Planned |
| **0.5.0: speech** | `speech.transcribeFile` | Planned |
| **Later** | Live dictation, examples for Bun, Deno and Node, a typed JavaScript wrapper package | Ideas |

The first consumer is [Georg](https://dasgeorg.cc): titles and summaries of agent sessions, OCR of screenshots and PDFs into notes, semantic search over saved items, and memory pressure to suspend idle agents.

Want something on this list sooner, or something that is missing? Open a [feature request](https://github.com/shipibo-studio/georg-native/issues/new?template=feature_request.yml) or start a [discussion](https://github.com/shipibo-studio/georg-native/discussions).
