# Reference

## Dependency Categories

When assessing a candidate boundary, classify its dependencies:

### 1. In-process

Pure computation, in-memory state, no I/O. Usually safe to merge behind one clearer boundary and test directly.

### 2. Local-substitutable

Dependencies with realistic local stand-ins, such as a test database, in-memory queue, or fake filesystem. Improve the boundary if the substitute is good enough to verify real behavior.

### 3. Remote but owned (Ports & Adapters)

Services you own across a network boundary. Define a port at the boundary. The workflow logic owns behavior; transport stays injectable. Tests usually use `MSW` or a local adapter so the client and workflow still run for real. Production uses the real transport.

Recommendation shape: "Define a shared interface, implement a production adapter and a test strategy based on `MSW` or a local adapter, then test the workflow through the boundary instead of through the transport."

### 4. True external (Mock)

Third-party services you do not control. Mock or fake them at the boundary. For HTTP integrations, prefer `MSW` over mocking `fetch`. Keep the external dependency behind an injected interface so boundary tests remain focused on your behavior.

## Testing Strategy

The core principle: **replace, don't layer.**

- Old low-level tests on scattered helpers may become waste once stronger boundary tests exist
- Write tests at the improved boundary, not at every internal seam
- Tests should assert on observable outcomes through the public interface
- Tests should survive internal refactors and wiring changes

## Issue Template

<issue-template>

## Problem

Describe the architectural friction:

- Which modules or workflows are scattered and tightly coupled
- What integration risk exists in the seams between them
- Why this makes the codebase harder to navigate, change, or test

## Proposed Interface

Describe the chosen boundary design:

- Interface signature (types, methods, params)
- Usage example showing how callers use it
- What workflow complexity it hides internally

## Dependency Strategy

Which category applies and how dependencies are handled:

- **In-process**: merged directly
- **Local-substitutable**: tested with a realistic stand-in
- **Ports & adapters**: port definition, production adapter, test adapter
- **Mock**: mock or fake at the external boundary, preferably at the HTTP layer for networked systems

## Testing Strategy

- **New boundary tests to write**: the key behaviors to verify at the interface
- **Old tests to delete**: low-level tests made redundant by the new boundary
- **Test environment needs**: any adapters, fixtures, or stand-ins required

## Implementation Recommendations

Durable architectural guidance that is not coupled to current file paths:

- What the boundary should own
- What it should hide
- What it should expose
- How callers should migrate to it

</issue-template>
