# Contributing to GeorgNative

Thanks for your interest. Contributions of every size are welcome: a comment on the API, a typo fix, a test, a new capability.

By taking part you agree to follow the [Code of Conduct](CODE_OF_CONDUCT.md).

## Ways to contribute

- **Design (now the most useful):** review [`docs/api.md`](docs/api.md) and [`docs/architecture.md`](docs/architecture.md), and answer the open questions on issues labeled `design`.
- **Questions and ideas:** use [Discussions](https://github.com/shipibo-studio/georg-native/discussions). Issues are for bugs and concrete proposals.
- **Bugs:** open a [bug report](https://github.com/shipibo-studio/georg-native/issues/new?template=bug_report.yml) with your macOS version, Mac model and host runtime.
- **Features:** open a [feature request](https://github.com/shipibo-studio/georg-native/issues/new?template=feature_request.yml) before writing code, so we can agree on the shape of the API first.
- **Security issues:** never in public. Follow [`SECURITY.md`](SECURITY.md).

## Development setup

Once there is code (see [`docs/roadmap.md`](docs/roadmap.md)):

- macOS 26 or later, Xcode 26 or later (Swift 6).
- A Mac with Apple Intelligence enabled to run the `intelligence` tests; they are skipped elsewhere with a clear message.
- `swift build` and `swift test` from the repository root.

## Pull requests

1. **Open or find an issue first** for anything beyond a small fix, and say you are working on it.
2. **Fork** the repository and create a branch from `main`: `feat/vision-document`, `fix/pdf-page-range`, `docs/api-errors`.
3. **Keep it focused:** one change per pull request.
4. **Add tests** for new behavior and for every bug fix (start with a test that reproduces the bug). Include hostile input: huge strings, invalid UTF-8, missing files, paths with `..` and symlinks.
5. **Update the docs** (`docs/api.md` for any change to requests or results) and add an entry under `[Unreleased]` in [`CHANGELOG.md`](CHANGELOG.md).
6. **Open the pull request** and fill in the template. CI must be green, and one maintainer review is required before merging.

## Standards

- **English** in code, comments, docs and commit messages.
- **[Conventional Commits](https://www.conventionalcommits.org/):** `feat(vision): recognize document tables`, `fix(pdf): honor the page range`, `docs: clarify memory ownership`.
- **Clean code:** intention-revealing names, small single-purpose functions, named constants, comments that explain why.
- **No new dependencies** in the shipped library without discussing it first.
- **No network access** in the library, ever, and no telemetry.
- **Security by default:** validate every input, set time limits, never log user content. See [`SECURITY.md`](SECURITY.md).
- **Stable interface:** changes to the C functions or to existing request and result fields need a design issue first.

## Releases

Maintainers tag `vX.Y.Z` on `main` after moving the `[Unreleased]` entries in `CHANGELOG.md` under the new version. CI builds the artifacts from the tag and publishes them with checksums.

## Licensing of contributions

By contributing, you agree that your contributions are licensed under the [MIT License](LICENSE) of this project.
