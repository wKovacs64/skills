# matt-skills-gpt54-with-opus46-examples-according-to-gpt54

Framework-neutral adaptations of Matt Pocock's agent skills for more traditional web app codebases, using the `gpt54` version as the base and adding more concrete web-testing examples.

## What changed

- Keeps the original process shape: grilling, PRDs, vertical slices, TDD, and architecture review
- Reinterprets "deep modules" as caller-facing boundaries that hide workflow complexity
- Pushes agents toward user-visible behavior, end-to-end slices, and boundary-level tests
- Avoids assuming a full architectural rewrite is needed before the skills become useful
- Adds concrete testing guidance for `Vitest`, `Playwright`, and `MSW`
- De-emphasizes React component and hook unit tests as default targets

## Included skills

- `grill-me`
- `write-a-prd`
- `prd-to-issues`
- `tdd`
- `improve-codebase-architecture`

## Notes

- These are not verbatim copies; they are edited variants intended for web-app teams
- Wording is kept relatively short so the files still behave like skill prompts, not long docs
- `tdd/deep-modules.md` exists because Matt's original `tdd` skill references a companion `deep-modules.md`; this repo adapts that file into a "useful boundaries" framing
- Support docs are included for `tdd` and `improve-codebase-architecture`, adapted to the same boundary-oriented framing
- `tdd/tests.md` and `tdd/mocking.md` add concrete API/browser examples and stricter mocking guidance
- Browser behavior is tested in real browsers with `Playwright`, not `jsdom`-style simulated environments
- HTTP boundaries prefer `MSW` over mocking `fetch`

## Likely fit

These versions are a good fit for server-rendered or hybrid web apps where useful boundaries often look like:

- route or request handlers
- actions / server functions
- service objects
- end-to-end browser workflows exercised in `Playwright`
- third-party integration adapters

## Source inspiration

- Matt Pocock's skills repo: `https://github.com/mattpocock/skills`
- Original blog post: `https://www.aihero.dev/5-agent-skills-i-use-every-day`
