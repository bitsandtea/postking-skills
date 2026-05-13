# Installing `postking-cli`

## Install

Global install (preferred):

```
npm install -g postking-cli
```

## Local install

If global install fails with permission errors (common in sandboxed agent terminals), fall back to a local install and invoke via `npx`:

```
npm install postking-cli
npx pking --version
```

Use `npx pking <command>` for all subsequent calls when on a local install.

## Upgrade

If `pking --version` reports a version older than `1.0.0`, the install predates the stable API. Reinstall using the global install command above (append `@latest` to ensure the newest release is fetched) before continuing any PostKing operation.
