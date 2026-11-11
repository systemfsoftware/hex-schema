# Contributing

## Requirements

- Linux, x86_64 or aarch64, with glibc: `flake.nix` declares only those two systems and
  `pnpm-workspace.yaml`'s `supportedArchitectures` pins linux/glibc, so on macOS or Windows use a Linux VM or WSL
- Nix with flakes
- Node.js 24
- pnpm (the version pinned in `package.json`'s `packageManager`)

The dev shell (`nix develop`) puts every tool on PATH, `bin/dprint` included; on a bare
shell that has nix the wrapper scripts resolve what they need themselves.

## Dependencies on core

https://nix.systemfsoftware.com is the package registry for `@systemfsoftware` packages: a producer pushes its package tarballs there from `main`, a consumer substitutes them through its flake inputs, and nothing is published to npm.

`flake.nix` pins core at a full commit (input `core`), and `pnpm-release-management.lib.mkProducerTarballs` lays core's pushed tarballs into `.deps/` when the dev shell starts. `pnpm-workspace.yaml` maps every core package to `file:.deps/<name>.tgz` through `overrides`.

To take newer core packages:

1. Bump the `core` input, then run `nix flake update core`.
2. Run `nix eval --raw .#producer-tarballs.overrides` and update `pnpm-workspace.yaml`'s `overrides` if core added or dropped a package.
3. Enter `nix develop` so `.deps/` holds the new tarballs.
4. Run `pnpm update '@systemfsoftware/*'` to record the new versions and integrities.

Other external dependencies come from the configured npm registry.

## Setup and workflow

```bash
nix develop   # the shell hook copies core tarballs into .deps/; then run the rest inside it
pnpm install
pnpm format
pnpm check:local
```

`pnpm check:local` reproduces the required CI static lane and tests: dprint, build, lint, typecheck, API reports,
attw and the tests. Run it before pushing. Its `test:types` step runs only for packages that define a `test:types`
script, and none do yet, so adding type tests means adding that script. It does not check commit messages; commitlint
enforces those in the merge queue.

## Pull requests

Open pull requests against `main` in systemfsoftware/hex-schema. Commit messages follow Conventional Commits
(`commitlint.config.ts`), and a change to a
publishable package's packed output advances its version at least a patch (there is no changeset
tooling; the bump is made directly in the package's `package.json`).
