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
- **`constraintsFiltering: strict`** — honour each consumer's declared engine
  constraints (`engines.node`, `packageManager`) when choosing candidate
  versions. Renovate's default is `none`, which would propose pnpm 11 / Node 22+
  to a Node-20 repo regardless of its pin; `strict` makes the preset respect the
  cap centrally and repo-agnostically — no private repo name in the public
  preset. The notable beneficiary is `rc-hubspot` (Node-20 for HubSpot
  compatibility), which stays on pnpm 10.x until its pin lifts. Trade-off:
  `strict` skips an update when a package publishes no usable engine metadata,
  so a genuinely-compatible bump can be held back silently — acceptable for the
  fleet's mainstream tooling, which all declares clean `engines`.
- **`dependencyDashboardApproval: false`** — Renovate creates each update's
  branch/PR on its own, no per-update tick on the Dependency Dashboard. This is
  an explicit override: the Mend Renovate Cloud portal had approval switched on
  at the org level, which parked every update under "Pending Approval" and meant
  nothing was created until a human checked the box. Repo/preset config wins over
  Mend's inherited org config, so setting it `false` here re-enables hands-free
  creation fleet-wide. (Renovate's own default is already `false`; this guards
  against the portal default.)
- **`semanticCommits: enabled`** — conventional-commit PR titles, matching the
  fleet's commitlint + release-please setup.
- **Automerge (non-major)** — `minor`/`patch`/`pin`/`digest` updates merge
  themselves once CI is green, so routine bumps land without manual babysitting.
  Major updates stay manual for a human look. `lockFileMaintenance` is **not**
  automerged: CI builds each repo's `@kc/*` `file:` siblings from their own
  `main`, not the local branch, so a wrongly-regenerated lockfile can pass CI
  undetected — those land by hand. With no `schedule` set, Renovate creates
  PRs on any run (still held back by `minimumReleaseAge`), so updates flow
  continuously rather than batching into a weekly window.
- **Grouping** — the shared toolchain (biome, cspell, commitlint, markdownlint,
  remark, lefthook) lands as one `dev-tooling` PR per repo; the Corepack `pnpm`
  pin is surfaced on its own.
- **`nix: { enabled: false }`** — the nix manager works where `nix` is reliably
  present (self-hosted, local), but the **hosted Mend app installs `nix`
  dynamically via containerbase** and it's documented-flaky for actually writing
  `flake.lock` (failed `install-tool nix`, Hydra download failures, and reports
  of it doing nothing) — which matched our own run (detected the flake,
  `updates:[]`, no PR). So `flake.lock` updates stay with the dotfiles
  `flake-update` workflow, which runs `nix flake update` natively in Actions.
  Refs: renovate discussions #29706, #30620, #31807; PR #31921.

## Repo-specific carve-outs

Anything that names a private repo lives in **that repo's own `renovate.json`**,
never here (this preset is public). The notable case is
`RECIPE-marketing/rc-hubspot`, which is pinned to Node 20 for HubSpot
compatibility. With `constraintsFiltering: strict` (above), the engine cap is
now honoured centrally from rc-hubspot's own `engines.node` — pnpm stays on 10.x
and Node below 21 without the preset naming the repo. Any further rc-hubspot
specifics (beyond what its `engines` field already expresses) still belong in
its own config.
