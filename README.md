# renovate-config

Org-wide hardened [Renovate](https://docs.renovatebot.com/) preset for `midnightntwrk`.

## Presets

### `default.json` (all repos)

Supply-chain hardened defaults applied to every repo in the org:

- **14-day cooldown** (`minimumReleaseAge`) on all third-party dependencies — blocks the majority of supply chain attacks
- **`internalChecksFilter: strict`** — enforces the cooldown (without this it's cosmetic only)
- **`constraintsFiltering: strict`** — blocks upgrades incompatible with declared runtime versions
- **OSV vulnerability scanning** enabled; security patches bypass cooldown and schedule
- **GitHub Actions pinned to SHA digests** (via `config:best-practices`)
- **Major updates** require dashboard approval
- **Weekly schedule** (Monday before 7am UK) with rate limits (2 PRs/hour, 5 concurrent)
- **`@midnight-ntwrk/*`** packages grouped and trusted (no cooldown)
- **Cargo `rangeStrategy: replace`** — preserves semver ranges in `Cargo.toml` (lock file handles pinning)

### `earthfile.json` (Earthfile repos only)

- **`ARG`/`ENV` version pins** annotated with a `# renovate:` comment, via a custom regex manager
- **Base images** — Earthly's `FROM` syntax is Dockerfile-compatible, so the native `dockerfile` manager tracks the tag *and* the digest. Earthly-only forms (`FROM +target`, `COPY +target/artifact`) are simply not matched

Renovate cannot recompute a `sha256` literal sitting next to a version `ARG`, so
a repo that hash-pins its downloads will get a PR that fails at `sha256sum -c`
until the hashes are refreshed. That is the repo's job, not the preset's.

## Usage

### Standard repo (TypeScript, JS, etc.)

Replace the contents of `renovate.json` with:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>midnightntwrk/renovate-config"]
}
```

### Rust + Earthfile repo (e.g. midnight-node)

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>midnightntwrk/renovate-config",
    "github>midnightntwrk/renovate-config:earthfile"
  ],
  "git-submodules": { "enabled": true }
}
```

### Repo with GitHub Packages npm registry

If the repo pulls `@midnight-ntwrk` packages from GitHub Packages, add a `hostRules` entry to point Renovate at the right registry:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>midnightntwrk/renovate-config"],
  "hostRules": [
    {
      "hostType": "npm",
      "matchHost": "https://npm.pkg.github.com/",
      "npmrc": "@midnight-ntwrk:registry=https://npm.pkg.github.com/"
    }
  ]
}
```

## Related

- [Renovate minimumReleaseAge docs](https://docs.renovatebot.com/key-concepts/minimum-release-age/)
- [Renovate configuration options](https://docs.renovatebot.com/configuration-options/)
