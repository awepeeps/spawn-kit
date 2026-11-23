# Repository guidance: Awepeeps Spawn Kit

This file applies to **every file in this repository**. It is a public Paper plugin, so keep all
content source/generic. [README.md](README.md) is the user-facing guide; the repo-local skill
[spawnkit-spawn-layout-change](.agents/skills/spawnkit-spawn-layout-change/SKILL.md) covers the
build-change workflow.

## Purpose and layout

Two independent, separately toggled parts live in `src/main/java/com/awepeeps/spawnkit`:

- **`spawn-zone`** — the protected 3x3x`core-depth` core plus its `clear-height` arrival column,
  spawn control, and the destructible house footprint.
- **`spawn-kit`** — the test-only house, starter chest and BlueMap marker, gated by `enabled` and
  an inclusive `starts-on`/`expires-on` window in `timezone`.

Config is `src/main/resources/config.yml`; command/permission metadata is `plugin.yml`. Tests are
`src/test/java/com/awepeeps/spawnkit` (in-memory `SyntheticWorld` plus a fake-world
`WorldBlockReader`).

## Invariants to preserve

- Nothing is ever placed in the arrival space; the house is ordinary destructible world and is
  never protected, repaired or rebuilt.
- The per-world issuance record (`issued-kits.yml`) is written before any block changes; a
  fresh-world reset deletes it only as an explicit owner-authorized step — never on your own.
- Layout version 3 upgrades are additive and never overwrite an existing house or chest.
- Prefer built-in Paper/Bukkit events and APIs over custom machinery; BlueMap is a softdepend.
- MCSManager owns the container, itzg Paper image and runtime; this plugin does not manage them.

## Build and test

Requires JDK 25; `just install` installs Temurin 25 and the pinned tools via mise.
Run from the repo root:

```sh
just install
just check        # format, workflow lint, one clean test+build
just test         # tests only
just reproducible # two clean builds, cmp, and the SHA-256 (main and release PRs)
```

Jar: `build/libs/awepeeps-spawn-kit-1.3.3.jar`. `just reproducible` rebuilds twice
and fails on any byte difference, then prints the SHA-256; it runs on `main` and on
release pull requests (a PR that changes `CHANGELOG.md`). Releases pin the exact
merge commit plus jar hash in the private deployment repo, never a moving branch.
Update `CHANGELOG.md` for user-visible changes.

## Privacy

Public repo: never commit host paths, server/world names, coordinates, seeds, instance IDs, logs,
config dumps, backups, secrets, or personal author data.

## Git hooks

Hooks are installed once per checkout with `just hooks` and run through Lefthook, loading the
shared jobs from `awepeeps/.github` at the `hooks-v2` tag. Agents never bypass them (C7.11):
no `LEFTHOOK=0`, no `git commit --no-verify`, and no `lefthook-local.yml`.
