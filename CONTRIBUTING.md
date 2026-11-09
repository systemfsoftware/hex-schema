# Contributing

## Requirements

- Linux, x86_64 or aarch64, with glibc: `flake.nix` declares only those two systems and
  `pnpm-workspace.yaml`'s `supportedArchitectures` pins linux/glibc, so on macOS or Windows use a Linux VM or WSL
- Nix with flakes
- GNU coreutils: the setup command uses `cp -L --no-preserve=mode`
- Node.js 24
- pnpm (the version pinned in `package.json`'s `packageManager`)

The dev shell (`nix develop`) puts every tool on PATH, `bin/dprint` included; on a bare
shell that has nix the wrapper scripts resolve what they need themselves.

## Dependencies on core

Core packages are not installed from a registry. `flake.nix` pins core at a full commit (input `core`), and
`nix build .#sfs-deps` links the core tarballs this lockfile names. `pnpm-workspace.yaml` maps every core package
(and its transitive core closure) to `file:.sfs-deps/<name>-<version>.tgz` through `overrides`.

To take newer core packages:

1. Bump the `core` input.
2. Rebuild `.sfs-deps` (the tarballs' names carry their versions, so this is what tells you the
   new versions).
3. Point `pnpm-workspace.yaml`'s overrides at those new `<name>-<version>.tgz` names.
4. Run `pnpm install`.

Other external dependencies come from the configured npm registry.

## Setup and workflow

```bash
mkdir -p .sfs-deps && cp -L --no-preserve=mode "$(nix build --no-link --print-out-paths .#sfs-deps)"/*.tgz .sfs-deps/
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
