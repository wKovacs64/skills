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
   - Replace with new subpath import pattern (e.g., `#/app/`)
   - Handle all import types: default, named, type-only, dynamic, re-exports
   - **IMPORTANT: Use relative `./` only for same directory or subdirectories**:
     - Use `./foo` or `./bar/baz` (same dir or deeper) Γ£ô
     - Use `#/app/foo` instead of `../foo` (parent dir) Γ£ô
     - IDEs use `#/` when not relative to current dir, `./` when it is

5. **Remove unnecessary packages** (if present):
   - `npm uninstall vite-tsconfig-paths tsconfig-paths`
   - Remove `tsconfigPaths()` from vite.config.ts if present
   - Vite 5+ handles Node.js subpath imports natively

6. **Verify the conversion**:
   - Run typecheck
   - Run build
   - Run tests if available

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
