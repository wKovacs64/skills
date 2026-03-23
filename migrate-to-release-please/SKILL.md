---
name: migrate-to-release-please
description: Migrate a repository from semantic-release to release-please. Use when the user wants to switch from semantic-release (auto-releases on push) to release-please (Release PR workflow), or mentions "release-please migration".
user-invocable: true
---

# Migrate from semantic-release to release-please

This skill migrates a Node.js repository from semantic-release to release-please.

## Workflow Change

**Before (semantic-release):** Push to main → automatic version bump + release
**After (release-please):** Push to main → Release PR created/updated → Merge PR when ready → GitHub release created

## Migration Steps

### 1. Gather Information

First, read these files to understand the current setup:

- `package.json` - get current version and identify semantic-release dependencies
- The CI workflow file (usually `.github/workflows/ci.yml` or similar) - find the release job
- `release.config.js` or `release.config.cjs` - if exists, note it for deletion

### 2. Create release-please Configuration Files

Create `.release-please-manifest.json` with the current version from package.json:

```json
{
  ".": "X.Y.Z"
}
```

Create `release-please-config.json`:

```json
{
  "release-type": "node",
  "include-component-in-tag": false,
  "packages": {
    ".": {}
  }
}
```

The `include-component-in-tag: false` option gives clean tags like `v2.0.0` instead of `package-name-v2.0.0`.

### 3. Update CI Workflow

#### Skip CI on release-please branches

Update the workflow triggers to skip CI on release-please branches (they only contain version bumps and changelog updates - the actual code already passed CI):

```yaml
on:
  push:
    branches-ignore:
      - 'release-please--**'
  workflow_dispatch:
```

If there are also `pull_request` triggers, add `branches-ignore` to those as well.

#### Replace the release job

Replace the semantic-release job with release-please. The new job should:

- Keep the same `needs` dependencies (run after tests pass)
- Keep the condition `if: github.ref == 'refs/heads/main'`
- Add permissions for `contents: write`, `issues: write`, and `pull-requests: write`
- Use `googleapis/release-please-action@v4` (uses `GITHUB_TOKEN` by default)

Example replacement:

```yaml
  release:
    name: 🚀 Release
    runs-on: ubuntu-latest
    needs: [list-existing-needs-here]
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: write
      issues: write
      pull-requests: write
    steps:
      - name: 🆕 Release Please
        uses: googleapis/release-please-action@v4
```

### 4. Remove semantic-release Dependencies

Remove these from `package.json` devDependencies:

- `semantic-release`
- `@semantic-release/git`
- `@semantic-release/changelog` (if present)
- `@semantic-release/npm` (if present)
- Any other `@semantic-release/*` packages

### 5. Delete semantic-release Config

Delete the semantic-release config file if it exists:

- `release.config.js`
- `release.config.cjs`
- `.releaserc`
- `.releaserc.json`
- `.releaserc.yml`

### 6. Delete CHANGELOG.md (if exists)

Check for `CHANGELOG.md` in the root directory only (not via glob, which can get flooded with node_modules results). Delete it if it exists - release-please will create and manage its own.

### 7. Add CHANGELOG.md to .prettierignore

If a `.prettierignore` file exists, add `CHANGELOG.md` to it. This prevents prettier from reformatting the release-please generated changelog.

### 8. Update RELEASING.md (if exists)

If there's a `RELEASING.md` file, update it to describe the new workflow:

```markdown
# Releasing

This package is released using [release-please](https://github.com/googleapis/release-please).

### Workflow:

1. Create feature branches and open PRs to `main` using
   [conventional commit](https://www.conventionalcommits.org/) messages (`feat:`, `fix:`, etc.)

2. release-please automatically creates/updates a Release PR with the changelog and version bump

3. When ready to release, merge the Release PR

4. release-please creates the GitHub release with the new tag
```

### 9. Update Lockfile

Run the package manager's install command to update the lockfile:

- `pnpm install` for pnpm
- `npm install` for npm
- `yarn install` for yarn

### 10. Format Files

Run the project's formatter if configured (check for format script in package.json).

## Post-Migration Notes

Inform the user of these manual steps after pushing:

1. If the repo has a `dev` branch workflow, they can now work directly on `main`
2. The custom `GH_TOKEN` secret can be removed if not used elsewhere (release-please uses built-in `GITHUB_TOKEN`)
3. First push to `main` will create a Release PR that they merge when ready to release

## Monorepo Support

For monorepos, the config files need adjustment:

`.release-please-manifest.json`:
```json
{
  "packages/foo": "1.0.0",
  "packages/bar": "2.0.0"
}
```

`release-please-config.json`:
```json
{
  "packages": {
    "packages/foo": {
      "release-type": "node"
    },
    "packages/bar": {
      "release-type": "node"
    }
  }
}
```
