# shared-workflows

Public, organization-wide GitHub Actions workflows for Plato. The only workflow
is secret scanning.

## Secrets Scanning

`.github/workflows/secrets-scanning.yml` runs two independent scanners against
the full history reachable from the checked-out `HEAD`:

| Job          | Purpose                                                               |
| ------------ | --------------------------------------------------------------------- |
| `gitleaks`   | Gitleaks with its default rules only                                  |
| `trufflehog` | TruffleHog with standard detector verification                        |
| `secrets`    | Aggregator; passes only if both scanners succeeded (stable status check) |

Design:

- **Self-contained.** No private repositories, no secrets, no
  `pull_request_target`, no inputs, no code from the scanned repository is
  executed, and nothing is installed from package registries. Only
  `contents: read` is requested; checkout does not persist credentials.
- **Pinned.** `actions/checkout` is pinned to a commit SHA. Gitleaks and
  TruffleHog are pinned release archives whose SHA-256 is checked before
  extraction.
- **Centrally controlled rules.** Repository-provided `.gitleaks.toml`,
  `.gitleaksignore` and `gitleaks:allow` comments are not honored.
- **Strict.** Any TruffleHog result (`verified`, `unknown` or `unverified`)
  fails the job, as does any Gitleaks finding or scanner error.
- **No secret-bearing output.** Gitleaks output is redacted. TruffleHog raw
  output stays in the runner temp directory; logs show only the finding count
  and detector names. No artifacts are uploaded.

## Usage

Both public and private repositories may call this workflow. Pin it to a
commit SHA:

```yaml
jobs:
  scan:
    permissions:
      contents: read
    uses: plato-corp/shared-workflows/.github/workflows/secrets-scanning.yml@<commit-sha>
```

The resulting required status check is `scan / secrets`.

## Organization required workflow

An organization ruleset can require this workflow by selecting this repository
(`plato-corp/shared-workflows`) as the source and
`.github/workflows/secrets-scanning.yml` as the path. Because public
repositories cannot use a workflow sourced from a private repository, the
organization must select this public repository rather than a private one.
This README does not imply that any such ruleset is currently configured.

## License

[MIT](LICENSE)
