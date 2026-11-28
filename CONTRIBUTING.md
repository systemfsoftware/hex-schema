# Contributing

## Requirements

- Linux, x86_64 or aarch64, with glibc: `flake.nix` declares only those two systems and
  `pnpm-workspace.yaml`'s `supportedArchitectures` pins linux/glibc, so on macOS or Windows use a Linux VM or WSL
- Nix with flakes
- Node.js 24
- pnpm (the version pinned in `package.json`'s `packageManager`)

The dev shell (`nix develop`) puts every tool on PATH.

## Dependencies

`@systemfsoftware` packages come from npm like any other dependency: manifests name caret ranges and `pnpm-lock.yaml` holds the exact versions. To take newer ones, run `pnpm update '@systemfsoftware/*'`. `flake.nix` pins core at a full commit (input `core`) only for its nixpkgs.

## Setup and workflow

```bash
nix develop   # then run the rest inside it
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
