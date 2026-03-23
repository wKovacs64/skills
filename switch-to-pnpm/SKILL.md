---
name: switch-to-pnpm
description: Convert a project from npm to pnpm package management. Use when switching package managers, setting up pnpm, or when user mentions migrating from npm to pnpm.
---

# Migrate from npm to pnpm

Convert a project from npm to pnpm for package management.

## Steps

1. **Analyze project structure**:
   - Check if monorepo (multiple package.json files, workspaces config)
   - Identify npm-specific files: package-lock.json, .npmrc
   - Check for npm scripts that need updating
   - Look for CI/CD configs referencing npm

2. **Clean existing npm artifacts**:
   ```bash
   rm -rf node_modules
   rm -f package-lock.json
   ```
   - For monorepos, also remove node_modules from all workspace packages

3. **Configure pnpm workspace** (monorepos only):
   - Create `pnpm-workspace.yaml`:
     ```yaml
     packages:
       - 'packages/*'
       - 'apps/*'
     ```

4. **Add packageManager field to package.json before dependencies**:
   ```json
   "packageManager": "pnpm@10.27.0"
   ```
   - Use `pnpm info pnpm` to identify the latest version and use that version in the `packageManager` field.
   - Insert right before the first dependencies section (peerDependencies, dependencies, or devDependencies - whichever comes first)
   - This tells tools (including pnpm/action-setup) which version to use

5. **Install dependencies with pnpm**:
   ```bash
   pnpm install
   ```
   - This creates pnpm-lock.yaml
   - Review any peer dependency warnings
   - **IMPORTANT**: pnpm may warn about ignored build scripts and suggest running `pnpm approve-builds`. Never run this command - ignore the warning.

6. **Append pnpm-lock.yaml to .prettierignore** (if using Prettier):
   - Append `pnpm-lock.yaml` to `.prettierignore` next to other package-related entries (package-lock.json, package.json)

7. **Move prisma to dependencies** (if using Prisma):
   - Move `prisma` from `devDependencies` to `dependencies` in package.json
   - This works with npm but not pnpm due to how pnpm resolves dependencies

8. **Handle MSW** (if using Mock Service Worker):
   - Remove the `msw` section from package.json (e.g., `"msw": { "workerDirectory": "public" }`)
   - Add a postinstall script in approximate alphabetical order within scripts:
     ```json
     "scripts": {
       "postinstall": "msw init ./public --no-save || true",
       ...
     }
     ```

9. **Update package.json scripts**:
   - Replace `npm run` with `pnpm` (shorthand) in scripts that call other scripts
   - For installed packages, use `pnpm exec`:
     - `npx prisma generate` ΓåÆ `pnpm exec prisma generate`
   - For one-off packages not in project, use `pnpm dlx`:
     - `npx create-react-app` ΓåÆ `pnpm dlx create-react-app`

10. **Update playwright.config.ts** (if using Playwright):
   - Update `webServer.command` to use `pnpm` instead of `npm run`
   - Example:
     ```ts
     webServer: {
       command: process.env.CI
         ? `cross-env PORT=${PORT} pnpm start`
         : `cross-env PORT=${PORT} pnpm dev`,
     },
     ```

11. **Update CI/CD configurations**:
   - GitHub Actions: Update workflow files in `.github/workflows/`
     ```yaml
     - name: ≡ƒôª Install pnpm
       uses: pnpm/action-setup@v4

     - name: ΓÄö Setup node
       uses: actions/setup-node@v4
       with:
         cache: pnpm
         node-version-file: '.nvmrc'

     - name: ≡ƒôÑ Install deps
       run: pnpm install

     - name: ≡ƒö¼ Generate Prisma client
       run: pnpm exec prisma generate

     - name: ≡ƒöÄ Type check
       run: pnpm typecheck
     ```
   - Replace `npm run X` with `pnpm X`
   - Update cache from `npm` to `pnpm`
   - Note: `pnpm install` automatically uses `--frozen-lockfile` in CI environments

12. **Update Dockerfiles** (if present):
   - Install pnpm as its own RUN command right after NODE_ENV: `RUN npm install -g pnpm@10.27.0`
   - Do NOT append pnpm install to other RUN commands (e.g., system dependencies)
   - Copy pnpm-lock.yaml instead of package-lock.json
   - Use `pnpm install --frozen-lockfile` for reproducible builds
   - Use `pnpm install --frozen-lockfile --prod` for production-only deps
   - Copy package.json before running `pnpm exec prisma generate` (pnpm requires it)
   - Example:
     ```dockerfile
     FROM node:24-slim AS base
     ENV NODE_ENV="production"

     # install pnpm
     RUN npm install -g pnpm@10.27.0

     FROM base AS deps
     COPY package.json pnpm-lock.yaml .npmrc ./
     RUN pnpm install --frozen-lockfile

     FROM base AS prod-deps
     COPY package.json pnpm-lock.yaml .npmrc ./
     RUN pnpm install --frozen-lockfile --prod

     FROM base AS build
     COPY --from=deps /app/node_modules node_modules
     COPY package.json package.json
     COPY prisma prisma
     RUN pnpm exec prisma generate
     COPY . .
     RUN pnpm build
     ```

13. **Update shell scripts** (if present):
   - Check for scripts like `start.sh`, `entrypoint.sh`, etc.
   - Replace `npm run` with `pnpm`
   - Replace `npx` with `pnpm exec`
   - Example:
     ```bash
     # Before
     npx prisma migrate deploy
     npm run start

     # After
     pnpm exec prisma migrate deploy
     pnpm start
     ```

14. **Update renovate.json** (if using Renovate):
   - Add customManagers section right before packageRules to keep Dockerfile pnpm version in sync:
     ```json
     "customManagers": [
       {
         "customType": "regex",
         "managerFilePatterns": ["Dockerfile", "Dockerfile.*"],
         "matchStrings": ["npm install -g pnpm@(?<currentValue>[^\\s]+)"],
         "depNameTemplate": "pnpm",
         "datasourceTemplate": "npm"
       }
     ],
     "packageRules": [
     ```

15. **Update documentation**:
   - README.md: Replace npm commands with pnpm equivalents
   - CONTRIBUTING.md: Update setup instructions
   - Common replacements:
     - `npm install` ΓåÆ `pnpm install`
     - `npm run dev` ΓåÆ `pnpm dev`
     - `npm test` ΓåÆ `pnpm test`

16. **Verify the migration**:
    - Run `pnpm install` (should use lockfile)
    - Run build: `pnpm build`
    - Run tests: `pnpm test`
    - Run dev server: `pnpm dev`
    - Run linting: `pnpm lint`

## Command Reference

| npm | pnpm |
|-----|------|
| `npm install` | `pnpm install` |
| `npm install <pkg>` | `pnpm add <pkg>` |
| `npm install -D <pkg>` | `pnpm add -D <pkg>` |
| `npm install -g <pkg>` | `pnpm add -g <pkg>` |
| `npm uninstall <pkg>` | `pnpm remove <pkg>` |
| `npm run <script>` | `pnpm <script>` or `pnpm run <script>` |
| `npm ci` | `pnpm install --frozen-lockfile` |
| `npx <installed-pkg>` | `pnpm exec <pkg>` |
| `npx <one-off-pkg>` | `pnpm dlx <pkg>` |
| `npm update` | `pnpm update` |
| `npm audit` | `pnpm audit` |

## Monorepo Considerations

- pnpm uses `pnpm-workspace.yaml` for workspace config (not package.json workspaces)
- Run commands in specific workspace: `pnpm --filter <pkg-name> <cmd>`
- Run commands in all workspaces: `pnpm -r <cmd>`
- Add dep to workspace: `pnpm add <pkg> --filter <workspace>`
