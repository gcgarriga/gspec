# Contributing to gspec

Thanks for contributing.

## What to update together

If you change the workflow, keep these files aligned:

- `SKILL.md` — source of truth for the skill behavior
- `README.md` — user-facing explanation
- `examples/quick.md` — compact example for the fast path
- `examples/greenfield.md` — example of the workflow from scratch
- `examples/brownfield.md` — example of the workflow in an existing codebase
- `references/explore-guide.md` — exploration support material

## How to test locally

1. Edit the files in this repo.
2. Start a **new** Copilot CLI session so the skill reloads.
3. Validate these paths:
   - Brownfield: `gspec explore`
   - Greenfield: `gspec`
   - Quick path: `gspec quick`
4. Check that the generated markdown still matches the documented format.

## Contribution bar

- Keep it simple.
- Prefer clarity over cleverness.
- Keep the README and examples in sync with the skill.
- Avoid adding process for process's sake.
