---
name: convert-to-subpath-imports
description: Convert TypeScript path aliases (tsconfig.json paths like ~/*) to native Node.js subpath imports (package.json imports with # prefix). Use when migrating path aliases, setting up subpath imports, or when user mentions converting ~/imports to #imports.
---

# Convert TypeScript Path Aliases to Native Subpath Imports

Convert a project from TypeScript path aliases (tsconfig.json) to native Node.js subpath imports (package.json with `#` prefix mapping to project root).

## Steps

1. **Read tsconfig.json** to identify existing path aliases in `compilerOptions.paths`

2. **Update package.json** with an `imports` field:
   - Use `#/*` to map to project root: `"#/*": "./*"`
   - Place near top of package.json (after name/private/type)
   - This allows importing from anywhere: `#/app/...`, `#/prisma/...`, etc.

3. **Update tsconfig.json paths** to match:
   - Change `"~/*": ["./app/*"]` to `"#/*": ["./*"]`
   - Keep `baseUrl` if present
   - Note: `paths` is still required because TypeScript's bundler moduleResolution doesn't resolve wildcard subpath imports from package.json without file extensions

4. **Convert all imports** in source files:
   - Find all files using old alias pattern (e.g., `~/`)
     - Include documentation and examples (e.g., multiple occurrences in @app/assets/svg-icons/README.md - one in code snippet and one in documentation above it)
   - Replace with new subpath import pattern (e.g., `#/app/`)
   - Handle all import types: default, named, type-only, dynamic, re-exports
   - **Check for imports that bypassed the alias** (e.g., `from 'app/foo'` without `~/`)
     - These won't be caught by the bulk `~/` replacement
     - Search for `from 'app/` or `from "app/` and convert to `#/app/`
   - **Convert `../` imports** to use `#/` (e.g., `from '../_app.foo/route'` ΓåÆ `#/app/routes/_app.foo/route`)
   - **Dynamic imports in client/browser code**:
     - Do NOT convert `../` paths inside `import()` calls that run in the browser
     - Browser runtime doesn't support Node.js subpath imports
     - Example: keep `import('../mocks/browser')` as-is in client entry files

5. **Fix imports that should use `./` instead of `#/`** (REQUIRED - do not skip):

   The rule: Use `./` for same-directory imports and for root-level files importing subdirectories. Use `#/` only for cross-directory imports.

   **a) Root-level app files** (files directly in `app/` like `root.tsx`, `entry.client.tsx`):
   - These should use `./` for app subdirectories, not `#/app/`
   - Check: `grep -l "from '#/app/" app/*.{ts,tsx} 2>/dev/null`
   - Fix any matches: `#/app/core/utils` ΓåÆ `./core/utils`

   **b) Same-directory imports** (files importing from their own directory):
   - A file should never import from its own directory path using `#/`
   - Use this script to find violations:
     ```bash
     find app -name '*.ts' -o -name '*.tsx' | while read file; do
       dir=$(dirname "$file" | sed 's|^\./||')
       # Convert dir path to import pattern (e.g., app/core -> #/app/core/)
       pattern="from ['\"]#/$dir/"
       if grep -qE "$pattern" "$file" 2>/dev/null; then
         echo "$file imports from own directory using #/"
         grep -E "$pattern" "$file"
       fi
     done
     ```
   - Fix any matches: `#/app/<same-dir>/foo` ΓåÆ `./foo`

6. **Remove unnecessary packages** (if present):
   - `npm uninstall vite-tsconfig-paths tsconfig-paths`
   - Remove `tsconfigPaths()` from vite.config.ts if present
   - Vite 5+ handles Node.js subpath imports natively

7. **Verify the conversion**:
   - Run typecheck
   - Run lint
   - Run build
   - Run tests if available

8. **Format the code**:
   - Run `npm run format` to format the code

## Example

Before (tsconfig.json):
```json
"paths": { "~/*": ["./app/*"] }
```

After (package.json):
```json
"imports": { "#/*": "./*" }
```

After (tsconfig.json):
```json
"paths": { "#/*": ["./*"] }
```

Before (source):
```typescript
import { db } from '~/core/db.server';
```

After (source, cross-directory):
```typescript
// In app/routes/notes.tsx importing from app/core/
import { db } from '#/app/core/db.server';  // Use #/ not ../
```

After (source, same directory):
```typescript
// In app/auth/auth.server.ts importing from app/auth/types.ts
import type { User } from './types';  // Use ./ for same dir
```

## Why Both Configs?

- **package.json `imports`**: Runtime/build resolution (Node.js, Vite)
- **tsconfig.json `paths`**: TypeScript type checking and IDE support (TS bundler moduleResolution doesn't fully support wildcard subpath imports)
