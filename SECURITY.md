# Security policy

## Reporting a vulnerability

If you find a security vulnerability, **do not open a public issue** and do not describe it in a pull request.

Report it privately through GitHub: **[Report a vulnerability](https://github.com/shipibo-studio/georg-native/security/advisories/new)**.

Please include:

- A description of the vulnerability and the affected version or commit
- Steps to reproduce it (a minimal request JSON or input file helps)
- Its potential impact
- Suggested mitigations, if any

We acknowledge reports within **48 business hours** and reply with a remediation timeline. We credit reporters in the advisory unless they prefer otherwise.

We appreciate good-faith reports. Please do not access, modify or extract third-party data during your research, and do not disclose the finding publicly until a fix is available.

## Supported versions

| Version | Supported |
| --- | --- |
| Latest release | Yes |
| Older releases | No |

There are no releases yet.

## Threat model

GeorgNative runs inside the host app's process, with the app's permissions. It treats **everything it receives as untrusted**, because the host may pass along content from agents, web pages or downloaded files.

| Risk | How the library handles it |
| --- | --- |
| Malicious documents (PDFs, images) exploiting parsers | Parsed in an isolated helper process with a restrictive sandbox, time and memory limits |
| Path tricks (`..`, symlinks, special files) | Paths resolved with `realpath`, only regular files accepted, read-only access |
| Resource exhaustion (huge inputs, endless work) | Documented size limits, deadlines on every request, a cap on concurrent heavy jobs |
| Malformed or hostile JSON | Strict parsing with size limits; invalid requests fail with `invalid_request` |
| Prompt injection through generated text | No tool calling: the model cannot trigger actions. Output is returned as data, and hosts must treat it as untrusted |
| Data leaving the device | No network access in the library and no telemetry |
| User data in logs or errors | Never logged; error messages contain no user content |
| Tampered binaries | Built by CI from a tag with published checksums; each host signs the library inside its own bundle |

## Automated scanning

- **gitleaks** (secrets) and **semgrep** (static analysis) run on every pull request, on every push to `main`, and weekly.
- **GitHub secret scanning** with push protection, and **Dependabot** for security updates and SHA-pinned GitHub Actions.
