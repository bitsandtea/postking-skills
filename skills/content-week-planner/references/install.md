# Installing `postking-cli`

## Install

Global install (preferred):

```
npm i -g postking-cli
```

## Ephemeral invocation (no global install)

If the global install fails with permission errors (common in sandboxed agent terminals), invoke via `npx` without installing:

```
npx postking-cli
npx pking --version
```

Use `npx pking <command>` for all subsequent calls when running ephemerally.

## Upgrade

If `pking --version` reports a version older than `1.0.0`, the install predates the stable API. Upgrade the `postking-cli` package to the latest version (`npm i -g postking-cli@latest`) before continuing any PostKing operation.
