---
name: switch-to-oxfmt
description: Use when migrating a project from prettier to oxfmt, or when the user wants to replace prettier with oxfmt as their code formatter.
user-invocable: true
---

# Switch from Prettier to Oxfmt

Migrate a JS/TS project from prettier to oxfmt. Assumes pnpm.

## Migration Steps

### 1. Gather Information

Read these files to understand the current setup:

- `package.json` — find prettier deps, scripts referencing prettier, and whether `tailwindcss` is a dependency
- Prettier config file — check for: `prettier.config.js`, `.cjs`, `.mjs`, `.prettierrc`, `.prettierrc.json`, `.prettierrc.yml`, `.prettierrc.yaml`, `.prettierrc.toml`, `.prettierrc.cjs`, `.prettierrc.mjs`
- `.prettierignore` — note entries for migration to oxfmt ignore patterns
- CI workflow files (`.github/workflows/*.yml`) — find jobs referencing prettier

### 2. Create Branch

Create and switch to a new branch for the migration:

```sh
git checkout -b chore/switch-to-oxfmt
```

### 3. Warn About Prettier Overrides

If the prettier config contains non-default formatting rules (e.g. `singleQuote`, `trailingComma`, `printWidth`, `semi`, `tabWidth`, etc.), warn the user that these won't carry over to oxfmt. List the specific rules that won't be preserved. oxfmt is opinionated with minimal configuration options.

### 4. Remove Prettier Packages

Remove prettier and all related packages from devDependencies:

```sh
pnpm remove prettier prettier-plugin-tailwindcss @foo/prettier-config
```

Adapt to whatever prettier-related packages are actually installed (plugins, shared configs, etc.).

### 5. Install Oxfmt

```sh
pnpm add -DE oxfmt
```

### 6. Delete Prettier Config Files

Delete whichever of these exist:

- `prettier.config.js` / `.cjs` / `.mjs`
- `.prettierrc` / `.prettierrc.json` / `.prettierrc.yml` / `.prettierrc.yaml` / `.prettierrc.toml` / `.prettierrc.cjs` / `.prettierrc.mjs`

### 7. Migrate `.prettierignore` to `.oxfmtrc.json`

Create `.oxfmtrc.json`. Migrate entries from `.prettierignore` into `ignorePatterns` (skip entries already covered by `.gitignore`). Also skip `package.json` — prettier can't format it well so projects ignore it, but oxfmt handles it fine. If `tailwindcss` is in dependencies or devDependencies, add `sortTailwindcss` with `clsx` and `cn` function detection.

With Tailwind:

```json
{
  "$schema": "./node_modules/oxfmt/configuration_schema.json",
  "sortTailwindcss": {
    "functions": ["clsx", "cn"]
  },
  "ignorePatterns": ["CHANGELOG.md", "pnpm-lock.yaml"]
}
```

Without Tailwind:

```json
{
  "$schema": "./node_modules/oxfmt/configuration_schema.json",
  "ignorePatterns": ["CHANGELOG.md", "pnpm-lock.yaml"]
}
```

Then delete `.prettierignore`.

### 8. Update package.json Scripts

Replace prettier commands in scripts:

- `prettier --cache --write .` → `oxfmt --write .`
- `prettier --cache --check .` → `oxfmt --check .`
- `prettier --write .` → `oxfmt --write .`
- `prettier --check .` → `oxfmt --check .`

Drop `--cache` — oxfmt doesn't support it.

### 9. Update CI Workflows

In GitHub Actions workflow files (`.github/workflows/*.yml`):

- Rename job IDs referencing prettier (e.g. `prettier:` → `format:`)
- Update job display names (e.g. `name: Prettier` → `name: Oxfmt`). If the existing workflow uses emojis in job names, use 🔤 for the Oxfmt job.
- Update `needs` arrays that reference the old job ID
- Update any `run` commands that invoke prettier

### 10. Set Up VS Code Settings

Create or update `.vscode/settings.json` with `oxc.oxc-vscode` as the default formatter. If the file already exists, merge these settings and remove any prettier formatter references.

```json
{
  "editor.defaultFormatter": "oxc.oxc-vscode",
  "[javascript]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[typescript]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[javascriptreact]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[typescriptreact]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[json]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[jsonc]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[css]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[markdown]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[yaml]": { "editor.defaultFormatter": "oxc.oxc-vscode" }
}
```

### 11. Update Lockfile

```sh
pnpm install
```

### 12. Commit

Commit all config migration changes:

```
chore: switch from prettier to oxfmt
```

### 13. Check for Remaining Prettier References

Search the codebase for any remaining references to "prettier", excluding the lockfile:

```sh
rg -i prettier --glob '!pnpm-lock.yaml' --glob '!yarn.lock' --glob '!package-lock.json'
```

If any results are found, inform the user of the remaining references so they can decide how to handle them (e.g. `prettier-ignore` comments in markdown, ESLint config references, etc.).

### 14. Inform User

Tell the user to:

1. Run `oxfmt --write .` to reformat the codebase
2. Commit the reformatting separately (e.g. `chore: reformat`)
3. Install the OXC VS Code extension (`oxc.oxc-vscode`) if not already installed
