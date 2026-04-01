# Example: Quick Feature

A compact example of using `gspec quick` for a small feature where a full 3-phase workflow would be overkill.

---

**User:** gspec quick — add a `/health` endpoint that returns service status and app version

**Agent:**

```text
📋 gspec status:
No existing .gspec/spec.md found.
.gspec/brief.md exists — using it for Context.
```

✅ Wrote `.gspec/spec.md`

```markdown
# Quick Spec: Health Endpoint

## Context
- Existing Express API with route -> validation -> service structure
- This is a small brownfield change following established patterns
- No new infrastructure or architecture decisions are needed

## Requirements
- Add `GET /health`
- Return HTTP 200 with:
  - service status (`ok`)
  - application version
- Keep the response lightweight and safe for public exposure
- Add or update tests following the existing API test pattern

## Approach
- Add the route in the existing health/system route area if present; otherwise create a small route following current conventions
- Reuse existing config/version source rather than hardcoding the version
- Keep the implementation simple and avoid introducing new abstractions
```

The project rules are already auto-loaded from `.github/copilot-instructions.md`, so now switch to Copilot plan mode and use the quick spec as the extra context:

```text
@.gspec/spec.md
Implement this change following the existing project patterns already loaded for this repo.
```
