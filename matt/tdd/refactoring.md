# Refactor Candidates

After a TDD cycle reaches GREEN, look for:

- **Duplication** -> extract shared logic
- **Overexposed workflows** -> hide more orchestration behind the boundary
- **Thin helper chains** -> combine them into a more useful interface
- **Long methods** -> break into private helpers while keeping tests on the public boundary
- **Feature envy** -> move logic closer to the boundary or model that owns the concept
- **Primitive obsession** -> introduce richer domain values where they simplify behavior
- **Existing code the new work exposes as awkward** -> improve it while the behavior is fresh in mind

## Boundary-Oriented Refactoring

Ask:

- Can callers know less?
- Can this workflow have one stable entry point?
- Can I delete low-level tests once a better boundary test exists?
- Can I remove helper churn without losing clarity?

Refactor toward simpler calling code, not just smaller functions.

## Rule

Never refactor while RED. Reach GREEN first, then deepen or simplify with tests protecting behavior.
