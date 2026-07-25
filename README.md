# renovate-config

The shared [Renovate](https://docs.renovatebot.com/) configuration for
`coo-quack` repositories. One file, `default.json`, extended by every repo that
Renovate manages.

## Using it

A repository's own `renovate.json` should be this, plus anything genuinely
specific to it:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>coo-quack/renovate-config"]
}
```

## What it sets

- **`config:recommended`** — Renovate's own baseline.
- **`helpers:pinGitHubActionDigests`** — action refs are pinned to commit
  digests and kept current, so a moved tag cannot change what CI runs.
- **`rangeStrategy: "pin"`** — the same idea for everything else. Manifests
  record exact versions rather than ranges, so what resolves is what is
  written down, and Renovate raises it rather than a silent re-resolution.
  Note the consequence for a package published with runtime dependencies: an
  exact version in a published manifest forces itself on consumers. Only
  `calc-mcp` is in that position today; if that becomes a problem, the
  carve-out belongs in its own `renovate.json`, not here.
- **Automerge on green**, with a one-day minimum release age. Required status
  checks are the gate: an update that breaks a repository turns them red and
  the merge stops there.
- **Dependency Dashboard**, and OSV vulnerability alerts.
- **Lock file maintenance**, automerged.
- **devDependencies grouped** into one pull request.

## Changing it

A change here reaches every repository in the org on Renovate's next run, which
is why the repo takes pull requests rather than pushes. Validate before opening
one:

```bash
npx --package renovate renovate-config-validator default.json
```

## Why it exists

The same configuration used to live as a copy in each repository, and the
copies had drifted: one had digest pinning for actions and a migrated key that
the others did not, and five were still emitting `Config migration necessary`
from the validator. Repository-specific rules — a version exclusion, say — stay
in the repository they belong to; everything else is here.
