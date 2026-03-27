# renovate-config

Org-wide hardened [Renovate](https://docs.renovatebot.com/) preset for `midnightntwrk`.

## Presets

### `default.json` (all repos)

Supply-chain hardened defaults applied to every repo in the org:

- **7-day cooldown** (`minimumReleaseAge`) on all third-party dependencies — blocks the majority of supply chain attacks
- **`internalChecksFilter: strict`** — enforces the cooldown (without this it's cosmetic only)
- **`constraintsFiltering: strict`** — blocks upgrades incompatible with declared runtime versions
- **OSV vulnerability scanning** enabled; security patches bypass cooldown and schedule
- **GitHub Actions pinned to SHA digests** (via `config:best-practices`)
- **Major updates** require dashboard approval
- **Weekly schedule** (Monday before 7am UK) with rate limits (2 PRs/hour, 5 concurrent)
- **`@midnight-ntwrk/*`** packages grouped and trusted (no cooldown)
- **Cargo `rangeStrategy: replace`** — preserves semver ranges in `Cargo.toml` (lock file handles pinning)

### `earthfile.json` (Earthfile repos only)

Adds a custom regex manager to track version `ARG`/`ENV` variables annotated with `# renovate:` comments in Earthfiles.

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

### Repo with GitHub Packages npm registry (e.g. midnight-wallet, midnight-faucet)

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>midnightntwrk/renovate-config"],
  "nix": { "enabled": true },
  "rebaseWhen": "conflicted",
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
