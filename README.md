# renovate-config

The shared [Renovate](https://docs.renovatebot.com/) configuration for
`coo-quack` repositories. One file, `default.json`, extended by every repo that
Renovate manages.

## Using it

This repository extends the preset too. It holds a CI workflow whose actions
are digest-pinned like everyone else's, and a policy repository that sits
outside its own policy is not one.

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
- **Override targets never take a major.** An entry in `pnpm-workspace.yaml`
  exists to lift a transitive package out of an advisory window while staying
  inside what its parent declared. A major takes it outside that. Renovate
  raised a `fast-uri` override from 3.1.4 to 4.1.1 and automerged it, while
  the `ajv` that pulls it declared `^3.0.1` — and its newest release still
  does. Majors here need dashboard approval instead.
- **Constraints are exempt from that pinning.** `engines`,
  `peerDependencies` and `required_version` state the range a package
  supports rather than a version it depends on, so they keep their range
  operator. Pinning them narrows what can install the package: the first
  attempt rewrote `"node": ">=22.22.3"` to `"node": "v26.5.0"` in a published
  manifest, and `"openclaw": ">=2026.6.0"` to a single release.
- **Automerge on green**, with a one-day minimum release age. Required status
  checks are the gate: an update that breaks a repository turns them red and
  the merge stops there.
- **Dependency Dashboard**, and OSV vulnerability alerts.
- **Lock file maintenance**, automerged.
- **devDependencies grouped** into one pull request.

## Changing it

A change here reaches every repository in the org on Renovate's next run, which
is why the repo takes pull requests rather than pushes. CI validates every one:

```bash
npx --package renovate renovate-config-validator --strict --no-global default.json
```

`--no-global` reads the file as a shared preset rather than as self-hosted
global configuration, which is what the other repos actually extend.
`--strict` fails on warnings and on a needed config migration, not only on
hard errors — migration warnings are what went unnoticed across five
repositories before this preset existed.

The validator checks shape, not judgement. It accepts a rule that is valid
and wrong: `rangeStrategy: "pin"` applied to `engines` produced
`"node": "v26.5.0"` in a published manifest and passed validation. Read what
Renovate's first run after a change actually proposes, in every repository,
before letting automerge have it.

## Why it exists

The same configuration used to live as a copy in each repository, and the
copies had drifted: one had digest pinning for actions and a migrated key that
the others did not, and five were still emitting `Config migration necessary`
from the validator. Repository-specific rules — a version exclusion, say — stay
in the repository they belong to; everything else is here.
