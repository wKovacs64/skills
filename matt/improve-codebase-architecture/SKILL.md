---
name: improve-codebase-architecture
description: Explore a codebase to find opportunities for architectural improvement, focusing on clearer feature boundaries, better testability, and less scattered workflow logic.
---

# Improve Codebase Architecture

Explore a codebase like an AI would, surface architectural friction, discover opportunities for improving testability, and propose refactors that create clearer feature boundaries or deepen scattered workflow logic as GitHub issue RFCs.

A **useful boundary** in a web app has a small interface hiding workflow complexity. This can be a deep module in the classic sense, but it can also be a feature-level service, action helper, or orchestration layer that lets you test behavior at the boundary instead of through internal seams.

## Process

### 1. Explore the codebase

Use the Agent tool with subagent_type=Explore to navigate the codebase naturally. Do NOT follow rigid heuristics - explore organically and note where you experience friction:

- Where does understanding one concept require bouncing between many small files?
- Where is one user-visible workflow scattered across many tiny files, hooks, helpers, and adapters such that the calling code must reassemble the real behavior?
- Where have pure functions been extracted just for testability, but the real bugs hide in how they're called?
- Where do tightly-coupled modules create integration risk in the seams between them?
- Which parts of the codebase are untested, or hard to test?

The friction you encounter IS the signal.

### 2. Present candidates

Present a numbered list of boundary-improvement opportunities. For each candidate, show:

- **Cluster**: Which modules/concepts are involved
- **Why they're coupled**: Shared types, call patterns, co-ownership of a concept
- **Dependency category**: See [REFERENCE.md](REFERENCE.md) for the four categories
- **Test impact**: What existing low-level or seam-heavy tests could be replaced by tests at the caller-facing boundary

Do NOT propose interfaces yet. Ask the user: "Which of these would you like to explore?"

### 3. User picks a candidate

### 4. Frame the problem space

Before spawning sub-agents, write a user-facing explanation of the problem space for the chosen candidate:

- The constraints any new interface would need to satisfy
- The dependencies it would need to rely on
- A rough illustrative code sketch to make the constraints concrete - this is not a proposal, just a way to ground the constraints

Show this to the user, then immediately proceed to Step 5. The user reads and thinks about the problem while the sub-agents work in parallel.

### 5. Design multiple interfaces

Spawn 3+ sub-agents in parallel using the Agent tool. Each must produce a **radically different** interface for the improved boundary.

Prompt each sub-agent with a separate technical brief (file paths, coupling details, dependency category, what's being hidden). This brief is independent of the user-facing explanation in Step 4. Give each agent a different design constraint:

- Agent 1: "Minimize the interface - aim for 1-3 entry points max"
- Agent 2: "Maximize flexibility - support many use cases and extension"
- Agent 3: "Optimize for the most common caller - make the default case trivial"
- Agent 4 (if applicable): "Design around the ports & adapters pattern for cross-boundary dependencies"

Each sub-agent outputs:

1. Interface signature (types, methods, params)
2. Usage example showing how callers use it
3. What complexity it hides internally
4. Dependency strategy (how deps are handled - see [REFERENCE.md](REFERENCE.md))
5. Trade-offs

Present designs sequentially, then compare them in prose.

After comparing, give your own recommendation: which design you think is strongest and why. If elements from different designs would combine well, propose a hybrid. Be opinionated - the user wants a strong read, not just a menu.

### 6. User picks an interface (or accepts recommendation)

### 7. Create GitHub issue

Create a refactor RFC as a GitHub issue using `gh issue create`. Use the template in [REFERENCE.md](REFERENCE.md). Do NOT ask the user to review before creating - just create it and share the URL.
