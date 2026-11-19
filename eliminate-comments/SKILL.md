---
name: eliminate-comments
description: Treat code comments as smells and eliminate them, scoped to a PR, branch, time box, path, or the whole repo.
disable-model-invocation: true
---

# Eliminate comments

Every comment is surface the agent must keep true on every later change, and most are not worth it: they restate the code, go stale, leave session notes that outlive the session, or serve as a **fig leaf**: an explanation that licenses a workaround instead of fixing the design. Treat each comment as a **smell**: find what it compensates for, fix that, then delete the comment. The target is zero comments in scope; a survivor must justify itself.

## Steps

1. **Scope.** Resolve the user's argument:
   - PR: `gh pr checkout <n>`; lines added in `gh pr diff <n>`.
   - Branch: lines added since `git merge-base origin/<default branch> HEAD`.
   - Time box: lines added since `git rev-list -1 --before="<time box>" HEAD`; if that prints nothing, the whole repo.
   - Path: every line under it.
   - None: the whole repo.

   For the diff scopes, comments on those lines plus comments attached to code the diff changes. Source files only; skip generated and vendored ones. Stop if the scope holds no comments.
2. **Inventory.** List every comment in scope (file, line range, text) in a ledger file made with `mktemp`, splitting a comment that carries several claims into one row per claim. Record the comment-line count and the command that produced it. Suppressions, comments that silence a checker (`eslint-disable`, `@ts-expect-error`, `# noqa`), are in scope. Out of scope: license headers, shebangs, and comments a program reads as input: tool configuration (`/// <reference>`, `// @vitest-environment`, `//go:build`), codegen markers, doctests, docstrings rendered as CLI help or API schema, JSDoc types in JS files.
3. **Context.** Read the repo's standards (`AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, coding-standards docs, lint config), its ADR titles (bodies on demand), and its glossary. Note any that require or model the smells below, or that no comment standard exists. Run the formatter, typecheck, lint, and tests once for a baseline.
4. **Diagnose and apply.** Give each row its smell and disposition. Two tests, in order. First, write in the row the specific bug or wrong decision losing this would cause: the change a future editor would make and what then breaks. No named bug, Delete. Second, can a reader divine it from names, types, or tests? If yes, Delete; if no, move it first. Apply all Delete rows and run the checks; then apply the remaining rows one at a time, rerunning affected tests after each. Mark each row done or deferred, with a reason.

   **Risk gate:** apply a change when a test exercises the code it touches, adding one if absent. Defer, with the proposed design, only a change to a published package's API or to behavior no test can pin. A deferred row keeps its comment.

   Past ~100 rows, partition by directory. Keep for yourself every Duplicate row and any row that needs a file outside one partition. Dispatch one subagent per partition with this file's path, its ledger slice, and the step 3 notes: it runs step 4 only, on its slice, without running checks, and returns its rows, rows it could not finish inside its partition, and any ADR, glossary, or doc text. Write those shared files yourself.
5. **Verify.** Run the checks against the baseline and fix what the changes broke, or revert that change and defer its row. Rerun the step 2 inventory: every comment in scope must match a Keep or deferred row, including any on code you added; diagnose the rest as new rows.
6. **Report.** Comment lines in scope before and after; row counts per disposition; ADRs and docs written; deferred rows with their proposed design; dead code found; the standards, docs, or lint rules from step 3 that teach the smells, with the edit you'd make (where none exists, propose the standard "comments say why, never what"); directories now at zero comments as candidates for a lint ban or ratchet. Offer to file the deferred rows as issues.

Done when every ledger row is done or deferred with a reason, the step 5 inventory matches the ledger, and the checks match the baseline.

## Smells

Each smell names its usual disposition.

- **Echo** → Delete: restates what the code does, or what a test name already says. Includes section banners and `// Arrange / Act / Assert`.
- **Trivia** → Delete: true and not derivable from the code, but has no named bug.
- **Stale** → Delete: contradicts the current code, names things that no longer exist, or cites a superseded decision. Often sits beside dead code.
- **Session note** → Delete: written for the agent's own session or a future one, or narrates change history ("now uses", "previously", "moved from", "per review"). Git holds history.
- **Commented-out code** → Delete: git holds it.
- **TODO** (FIXME, HACK) → Delete when it cites an issue in this repo's tracker; otherwise defer, and delete it once filed. An upstream issue does not count: the local issue links it, and the workaround beside it is a Library quirk.
- **Duplicate** → Encode: the same explanation copied to several sites, usually because the code it explains is copied too.
- **Library quirk** → Encode: explains surprising third-party behavior.
- **Constraint** → Encode: an ordering, race, or invariant rule the code must uphold.
- **Fig leaf** → Redesign: the comment beside a workaround, including a suppression. It marks where the code should have been designed differently, not explained.
- **Decision** → Record: a durable cross-cutting contract or trade-off: a persistence format, defaults tuned by hand, a platform limit that forced the design.
- **Term or reference** → Record: defines a domain term, or catalogs something a doc already owns (key bindings, config options).

## Dispositions

Every disposition but Keep ends with the comment deleted.

- **Delete**: nobody needs it, or the code, a name, a test, or git already says it.
- **Encode**: move the knowledge into code. A name on a function, variable, predicate, or option (`keepOpenOnSubpage`); a type that makes the wrong state unrepresentable; a test named for the constraint, which you break once to watch it go red; an adapter module wrapping a library, with function names that state its quirks (`removeAllShapesDespiteReadonly`); one shared helper replacing a Duplicate; a lint rule replacing a remembered convention.
- **Redesign**: change the code so the workaround no longer exists: a typed wrapper instead of a cast, a `ResizeObserver` instead of waiting frames, a variant prop instead of class overrides, a fixed type error instead of a suppression.
- **Record**: move the knowledge to where the repo keeps it. An ADR is the last resort: reach for it only after Encode and Redesign fail, and only for a decision the code must obey, not a description of how the code works; the latter belongs in code. A Fig leaf goes to Redesign even when it reads as a deliberate deviation. The ADR must pass "When to offer an ADR" in the `domain-modeling` skill's `ADR-FORMAT.md`, and its rows name the criteria met and why code cannot carry it. Write it without asking, in that file's format, one ADR per decision however many rows feed it. Source every claim from the comment, the code, or git history, and list unknown alternatives as open questions. A Term goes to the glossary in the same skill's format, a reference to the doc that owns it. Expect few ADRs: most "why" comments are library quirks and belong in code.
- **Keep**: one line saying why, never what, when no name, type, test, redesign, or doc can carry it. The row holds its named bug and why each other disposition fails. Also keep doc comments on a published package's public API that tell consumers something the signature does not, and comments a repo standard requires (report that standard).
