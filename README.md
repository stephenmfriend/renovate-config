# renovate-config

Shared [Renovate](https://docs.renovatebot.com) preset for the Acumatica/HubSpot
repo fleet. One canonical set of update rules so dependency versions stay
converged across every repo instead of re-diverging over time.

## Usage

This repo **must be public** — the fleet spans two GitHub orgs
(`stephenmfriend` and `RECIPE-marketing`), and the hosted Renovate app
authenticates per-org, so a private preset can't be read across the org
boundary. The preset contains only update *rules* — no secrets, no private repo
names — so public is safe.

Each consuming repo extends it from a one-line `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>stephenmfriend/renovate-config"]
}
```

## What `default.json` does

- **`config:best-practices`** — Renovate's recommended baseline. Pins
  devDependencies to exact versions and pins GitHub Action digests, so versions
  stop silently floating.
- **`minimumReleaseAge: 5 days`** — never propose a version younger than this.
  Avoids opening PRs for packages so fresh that pnpm 11's supply-chain policy
  would reject the install (the lefthook 2.1.9 lesson).
- **`semanticCommits: enabled`** — conventional-commit PR titles, matching the
  fleet's commitlint + release-please setup.
- **Grouping** — the shared toolchain (biome, cspell, commitlint, markdownlint,
  remark, lefthook) lands as one `dev-tooling` PR per repo; the Corepack `pnpm`
  pin is surfaced on its own.
- **`nix: { enabled: false }`** — flake input updates stay with the dotfiles
  `flake-update` workflow for now. Flip to `true` to let Renovate take over
  `flake.lock` and retire that workflow.

## Repo-specific carve-outs

Anything that names a private repo lives in **that repo's own `renovate.json`**,
never here (this preset is public). The notable case is
`RECIPE-marketing/rc-hubspot`, which is pinned to Node 20 for HubSpot
compatibility and therefore caps `pnpm` below 11 and Node below 21 in its own
config.
