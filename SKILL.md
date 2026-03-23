---
name: gspec
description: This skill should be used when the user asks to "spec this project", "explore this codebase", "create a spec", "gspec", "define requirements", "create an implementation plan", "what does this codebase do", "help me plan this feature", "start a new project", "understand this project", "gspec status", "where did I leave off", "resume gspec", or needs a structured workflow for understanding a codebase, defining requirements, and planning implementation — whether starting from scratch (greenfield) or working with existing code (brownfield).
---

# gspec — Minimal Spec-Driven Development

A lightweight workflow for going from codebase or idea to a clear implementation plan.

## When to Use This Skill

- Starting a new project and want structure before coding
- Joining an existing codebase and need to understand it before building
- Planning a feature and want requirements → plan flow
- User says "gspec" or any of the trigger phrases above

## Core Workflow

```
Phase 1: Explore   →  .gspec/context.md    (understand where you are)
Phase 2: Specify   →  .gspec/spec.md       (define what to build)
Phase 3: Plan      →  .gspec/plan.md       (decide how to build it)
→ Hand off to Copilot's plan mode for tasks and implementation
```

Each phase builds on the previous. Artifacts are stored in `.gspec/` at the project root and **persist across sessions** — any future Copilot session can read them.

For projects with multiple features, use `.gspec/features/<feature-name>/` for per-feature artifacts (spec.md, plan.md), while `context.md` stays at the `.gspec/` root since it describes the whole project.

**Context persistence:** `context.md` is project-wide — it describes the codebase, not a feature. Maintain it on the default branch (`main`/`master`) so new branches start with the current version. Existing feature branches need an explicit merge or rebase to pick up later updates. Feature-specific files (`spec.md`, `plan.md`) naturally live on their feature branches.

## Design Philosophy

These principles govern how gspec thinks — and how it guides future implementation:

1. **Simple until proven otherwise** — Always recommend the simplest design that meets the requirements. Add complexity only when a concrete requirement demands it.
2. **Right-size everything** — A 50-line script doesn't need an architecture diagram. A distributed system does. Scale the depth of each phase to the project's actual complexity.
3. **Fewer moving parts** — Prefer stdlib over libraries. Prefer one library over three. Every dependency is a liability.
4. **Match the codebase** — For brownfield: match existing patterns exactly. Don't introduce new paradigms. For greenfield: establish patterns early and follow them consistently.
5. **Artifacts should be useful, not thorough** — Every line in context.md, spec.md, and plan.md should earn its place. If it doesn't help someone build the right thing, cut it.

## How to Drive the Workflow

The user controls which phase to run. Listen for:
- **"gspec explore"** or **"explore this codebase"** → Phase 1
- **"gspec specify"** or **"define requirements for..."** → Phase 2
- **"gspec plan"** or **"create an implementation plan"** → Phase 3
- **"gspec status"** or **"gspec"** alone → Show current state and suggest next step
- **"gspec"** with a description → Run all 3 phases sequentially, pausing after each

If the user says "gspec" with a description of what they want to build, run all phases sequentially, pausing after each to confirm before proceeding.

If the user includes a trailing feature name after a phase command, treat it as the active feature and use the feature subdirectory:

- `gspec specify wishlist` → write `.gspec/features/wishlist/spec.md`
- `gspec plan wishlist` → write `.gspec/features/wishlist/plan.md`

If no feature name is provided, use the root artifacts (`.gspec/spec.md`, `.gspec/plan.md`).

**Fast-track:** If the user says "gspec quick" with a description, produce a single combined `.gspec/spec.md` containing:

```markdown
# [Feature Name]

## Context
[2-3 sentences: what the project is, relevant existing patterns if brownfield]

## Requirements
[What to build — plain language]

## Approach
[How to build it — tech choices, key decisions]
```

Use this for small features that don't need 3 separate files.

---

## On Entry: Always Check Status First

**Before running any phase**, check the `.gspec/` directory to understand current state:

1. Check if `.gspec/` exists
2. Check which artifacts exist: `context.md`, `spec.md`, `plan.md`
3. Check for feature subdirectories: `.gspec/features/*/`
4. Report status to the user:

```
📋 gspec status:
✅ context.md — last updated [date]
✅ spec.md — [feature name from header]
⬜ plan.md — not yet created

Suggested next step: gspec plan
```

If the user asked for a specific phase, proceed with it. If they just said "gspec", suggest the next logical phase based on what exists.

If artifacts already exist and the user re-runs a phase, ask whether they want to **update** the existing artifact or **start fresh**. For multi-feature projects, ask which feature they're working on unless it is already explicit in the command.

### First-time setup: ask about git tracking

When creating the `.gspec/` directory for the first time (i.e., it does not already exist), ask the user which artifacts to track in git:

> Which `.gspec/` artifacts should be tracked in git?
>
> - **`context.md`** — describes the whole project. Recommended to track on the default branch so every branch inherits it.
> - **`spec.md` / `plan.md`** — can be project-wide or feature-specific. Track them if you want specs visible in PRs and shared with teammates.
> - **Feature directories** (`.gspec/features/`) — feature-specific specs and plans. Track them if you want feature artifacts visible in PRs.
>
> Note: gspec will not run `git add` or `git commit` for you.

Configure `.gitignore` based on the user's choices. Examples:

- **Track everything** — do nothing; `.gspec/` remains unignored.
- **Track only `context.md`** — add to `.gitignore` (order matters — the exception must come after the wildcard):
  ```
  .gspec/*
  !.gspec/context.md
  ```
- **Track nothing** — add `.gspec/` to `.gitignore`.

Only ask once — if `.gspec/` already exists and tracking is already configured, skip this question. However, if `.gitignore` contains a coarse `.gspec/` rule (ignoring everything) and the user re-runs gspec, offer to replace it with a granular configuration (e.g., tracking only `context.md`).

---

## Phase 1: Explore

**Goal:** Build a deep, shared understanding of the project context.

**Output:** `.gspec/context.md`

**Branch guidance:** `context.md` is project-wide — it should be committed to the default branch (`main`/`master`) so every feature branch inherits it. When updating `context.md` from a feature branch, remind the user to merge or cherry-pick the update back to the default branch.

### Brownfield (existing codebase detected)

Detection heuristic:
- If `package.json`, `Cargo.toml`, `go.mod`, `requirements.txt`, `*.csproj`, `Gemfile`, `pom.xml`, or similar exist → brownfield
- If directory has significant source files (>5 non-config files) → brownfield
- Otherwise → greenfield

**Do not just detect file existence.** Actually read and understand the codebase. Use the explore reference guide (`references/explore-guide.md`) for detailed patterns.

#### Step 1: Read existing documentation first
- Read `README.md` — this is the fastest way to understand the project
- Check for `CONTRIBUTING.md`, `docs/`, `ADR/`, architectural decision records
- Read any existing specs, design docs, or wiki links mentioned in docs

#### Step 2: Understand the architecture by reading code
- **Read the main entry point** (e.g., `src/index.ts`, `main.py`, `cmd/main.go`) to understand the application bootstrap and request flow
- **Trace one request end-to-end** — from route/handler → service/business logic → data layer. This reveals the actual architecture better than any diagram.
- **Read 2-3 test files** — tests reveal domain boundaries, expected behaviors, and how the code is meant to be used
- **Check the data model** — database schema, ORM models, or type definitions that define the domain

#### Step 3: Extract patterns and coding principles

Analyze how the codebase works so new code can match it exactly. Cover these in one pass:

**How code is structured:**
- Error handling pattern (custom errors, middleware, Result types)
- Auth/authz approach
- Architectural patterns (service layer, repository, event-driven)
- Naming conventions (casing, file naming, variable naming)
- Config management (env vars, config files, feature flags)
- Dependency injection style
- Linter/formatter configs — read them if they exist (`.eslintrc`, `.prettierrc`, `ruff.toml`, `.editorconfig`)

**How code is tested:**
- Test naming and structure (arrange-act-assert, given-when-then)
- Mocking strategy (real DB vs mocks/fakes)
- Test helpers, factories, fixtures — where they live
- Coverage target and current state

**How code performs:**
- Caching layers, pagination patterns, async patterns
- Database query patterns (indexes, eager/lazy loading)

**How code is secured:**
- Input validation approach (boundary vs deep)
- Secrets management
- Rate limiting, CORS, CSRF

Write these as **rules to follow**, not observations. Example: "All routes validate input with Zod schemas before calling services" — not "Zod is used for validation."

#### Step 4: Produce the summary
Cover these in `.gspec/context.md`:

1. **What this project does** — 2-3 sentences
2. **Tech stack** — Languages, frameworks, major libraries, versions
3. **Architecture** — How code is organized. Describe the actual flow, not just folder names.
4. **Key directories and entry points**
5. **Data layer** — Database, ORM, key models and relationships
6. **How to build, test, and run** — Commands for dev, test, build, deploy
7. **Patterns and principles** — The prescriptive rules from Step 3. This is the most important section — it tells the agent exactly how to write code that fits this codebase.
8. **Strengths and debt** — What's well-designed, where the complexity lives

#### Step 5: Generate `.github/copilot-instructions.md`

After writing `context.md`, **offer to generate or update `.github/copilot-instructions.md`** from the patterns and principles section. This is a Copilot-native feature — instructions in this file are automatically loaded in every Copilot session (CLI and VS Code), so the codebase conventions are always active without needing `@` mentions.

If the file already exists, show the user the diff and ask before updating. If it doesn't exist, create it with:

```markdown
# Copilot Instructions

<!-- Generated by gspec explore — edit freely, this is your file -->

## Coding Conventions

[Extract the prescriptive rules from .gspec/context.md Patterns and principles section]

## Testing Conventions

[Extract the testing rules from .gspec/context.md]
```

Keep it short — only the rules that Copilot should follow when writing code. Don't duplicate the full context.md.

### Greenfield (empty or near-empty directory)

**Suggest the user run `/research` first** if the domain is unfamiliar or complex. Copilot's built-in `/research` command does deep multi-source research (web + GitHub) and is more thorough than basic web search. Example prompt for the user:

> `/research What are the best practices, common patterns, and pitfalls for building [type of app] in [domain]?`

If the user skips `/research`, use web search as a fallback. Look for:
- Similar projects and existing solutions (prior art) — what works, what doesn't?
- Common architectural patterns for this type of application
- Domain-specific standards, regulations, or best practices
- Potential pitfalls that others have encountered

Then ask the user:
1. What kind of project is this? (web app, CLI tool, library, API, mobile app, etc.)
2. What domain/problem does it address?
3. Who are the target users?
4. Any known constraints? (must use specific tech, compliance, team skills, timeline)
5. Is this a prototype/MVP or production-grade from the start?
6. Any existing systems this needs to integrate with?

Capture answers in `.gspec/context.md` as project intent.

After gathering user input, **enrich with web research:**
- Search for existing projects in the same space — summarize what exists and where this project would differentiate
- Search for common architectural patterns for this type of project
- Surface any domain-specific gotchas (e.g., "photo management apps need EXIF parsing", "real-time chat needs connection state management")
- Include a "Prior Art & Inspiration" section in `context.md` with links and brief notes on what to learn from existing solutions

### context.md Format

Write freeform markdown. Start with a header and type indicator:

**Write as if briefing a new developer (or AI agent) who has never seen this codebase.** Be explicit — no jargon shortcuts, no "see X file for details." The whole point of context.md is that a future session can read it cold and immediately understand the project and how to write code that fits.

```markdown
# Project Context

**Type:** brownfield | greenfield
**Date:** YYYY-MM-DD

[rest of content organized by the categories above]
```

After writing, present a brief summary to the user and ask if anything is missing or wrong. For brownfield, double-check: are the **patterns and principles** written as actionable rules, not just observations?

---

## Phase 2: Specify

**Goal:** Define **what** to build and **why**. Not how.

**Prerequisite:** `.gspec/context.md` should exist. If not, run Phase 1 first or ask the user to describe the project context.

**Output:** `.gspec/spec.md` (or `.gspec/features/<name>/spec.md` for multi-feature projects)

### Process

1. **Read** `.gspec/context.md` for project context
2. **Ask the user** to describe what they want to build. Encourage them to focus on:
   - What the user/system should be able to do
   - Why this matters (the problem being solved)
   - Who the users are
   - What success looks like
3. **Actively challenge and probe** — Don't just accept the first description. Push for completeness:
   - **Identify gaps:** "You mentioned users can create X, but what about editing or deleting them?"
   - **Challenge assumptions:** "You said 'real-time updates' — do you mean WebSocket push, or is polling acceptable?"
   - **Probe edge cases:** "What happens when [unlikely but possible scenario]?"
   - **Question scale:** "How many [entities] do you expect? Tens, thousands, millions?"
   - **Surface implicit requirements:** "You'll need authentication for this — should I include that in scope, or is it already handled?"
   - **Suggest what's missing:** Based on the domain, proactively suggest requirements the user likely hasn't thought of. E.g., for an e-commerce feature: "Should we handle inventory conflicts when two users try to buy the last item?"
   - **Clarify priorities:** "If you had to ship half of this, which half matters most?"
4. **Write** `.gspec/spec.md`

### spec.md Format

Write freeform markdown. Keep it practical — no ceremony.

```markdown
# Specification: [Feature/Project Name]

## Overview
[2-3 sentence summary of what this is and why it matters]

## Requirements
[What the system must do — plain language, organized by area. Not user stories.]
[Example: "Users can add products to their wishlist" — not "As a user, I want to add products..."]

## Constraints
[Performance, security, accessibility, scale — only what actually matters for this project]

## Out of Scope
[What this does NOT include]

## Open Questions
[Anything unresolved — delete this section if empty]
```

After writing, **run the Self-Review Quality Gate** (see below) for the spec, then present the summary and ask the user to confirm or refine.

**Offer to create a GitHub Issue** from the spec only if all of these are true:
- the project is a git repo
- it has a GitHub remote
- `gh` is installed and authenticated

Use `gh issue create` with the spec content as the body. If any prerequisite is missing, skip the offer or explain why it is unavailable. Only offer, don't push — the user may prefer to keep specs local.

---

## Phase 3: Plan

**Goal:** Define **how** to build it — architecture, tech stack, key decisions.

**Prerequisite:** `.gspec/spec.md` should exist. If not, ask the user for requirements first.

**Output:** `.gspec/plan.md` (or `.gspec/features/<name>/plan.md`)

**Skip this phase** if the feature is small enough that implementation is obvious from the spec (e.g., add a CRUD endpoint to an existing API following established patterns). Go straight to the handoff — tell the user to switch to Copilot's plan mode and use `@.gspec/spec.md` as context. The plan exists to make decisions — if there are no decisions to make, skip it.

### Process

1. **Read** `.gspec/context.md` and `.gspec/spec.md`
2. **Recommend a tech stack** before asking for preferences:
   - Analyze the requirements from `spec.md` and context from `context.md`

   **For brownfield:** The tech stack is mostly decided. Focus on:
   - What new libraries are needed for this feature? (e.g., email service, job queue)
   - Do any existing libraries need upgrading?
   - Recommend specific libraries for new concerns — verify current versions via web search
   - Flag if the existing stack has a significant limitation for this feature

   **For greenfield:** Research and recommend the full stack:
   - **Use web search** to research current best options — latest versions, comparisons, known pitfalls
   - **Proactively suggest 2-3 tech stack options** ranked by fit, with brief rationale for each
   - For each option, explain: why it fits, tradeoffs, ecosystem maturity, developer experience
   - Recommend specific libraries for common concerns: auth, validation, testing, ORM, logging, error handling
   - Favor well-supported, production-proven tools over trendy ones
   - **Cite your research** when it influences a recommendation

   **Ask the user** which option they prefer, or if they have a different stack in mind
3. **Create the implementation plan** covering:
   - **Architecture overview** — High-level design, component diagram in text
   - **Tech stack** — Specific technologies and versions with rationale
   - **Data model** — Key entities and relationships (if applicable)
   - **API / Interface design** — Key endpoints, interfaces, or contracts (if applicable)
   - **Key implementation decisions** — Patterns, libraries, approaches chosen and why
   - **Dependencies & prerequisites** — What needs to exist before implementation
   - **Risks & mitigations** — What could go wrong, how to handle it

   **Right-size the plan:** Not every section is needed. Skip sections that don't apply. A CLI tool might only need Tech Stack + Implementation Approach. A full-stack app needs most sections. Ask: "Would removing this section lose important information?" If no, cut it.

4. **Simplicity check** — Before writing, ask yourself:
   - Is this the simplest architecture that meets ALL the requirements?
   - Am I adding layers/abstractions that aren't justified by a concrete requirement?
   - Could I remove a dependency and use stdlib instead?
   - Am I pattern-matching from "best practices" or from actual project needs?
   If the plan feels heavy, simplify it. Challenge your own recommendations.
5. **Write** `.gspec/plan.md`

### plan.md Format

Write freeform markdown. Include:

```markdown
# Implementation Plan: [Feature/Project Name]

## Architecture
[High-level design and component relationships]

## Tech Stack
[Technologies chosen with brief rationale]

## Data Model
[Key entities and relationships — skip if not applicable]

## Key Interfaces
[APIs, contracts, module boundaries — skip if not applicable]

## Implementation Approach
[How to build this, key patterns and decisions]

## Dependencies & Prerequisites
[What needs to be in place before starting]

## Risks
[What could go wrong and how to mitigate]
```

After writing, **run the Self-Review Quality Gate** (see below) for the plan, then present the summary and ask the user to confirm or refine.

---

## After gspec: Handoff to Copilot

Once the last needed artifact is complete, actively help the user transition:

1. **Summarize what was produced** — list the `.gspec/` files and what each contains
2. **Give the user a ready-to-use prompt** they can copy. Example:

   > "Implement the feature described in .gspec/plan.md, following the coding patterns in .gspec/context.md. Reference .gspec/spec.md for requirements."

   For named features, use the paths under `.gspec/features/<name>/` in the handoff prompt.

3. **Suggest switching to plan mode** (`Shift+Tab`) — Copilot's native mode for task generation and implementation
4. **Remind the user** that `.gspec/` artifacts persist — in any future session on this project, they can say `@.gspec/context.md` to give the agent full codebase context instantly

The value of gspec is that these artifacts are a **persistent briefing document** for any AI agent. No more re-explaining the codebase or requirements every session.

---

## Self-Review: Quality Gate

**After generating any artifact, review it against these checks BEFORE presenting to the user.** Fix issues silently, then present the improved version.

### Context (brownfield)
1. Patterns are written as rules ("Do X"), not observations ("X is used")
2. Architecture describes actual flow, not just folder names
3. Build/test/run commands are complete — a new dev could get running from this alone
4. No section is just a list of names — each has enough explanation to be actionable

### Spec
1. No false open questions — if it's a technical fact, it's a requirement, not a question
2. Error paths covered for every integration point
3. No vague "automatically" without specifying the mechanism
4. State management is explicit if the system has loops or accumulating data

### Plan
1. Simplest architecture that meets all requirements — nothing speculative
2. Environment setup commands included if dependencies are listed
3. No vague verbs — "parse JSON into dict" not "handle the response"
4. Complexity distributed evenly — no step does >3 things while others do 1

---

## General Rules

1. **Check `.gspec/` status on entry** — report what exists, suggest next step
2. **On first `.gspec/` creation** — ask which artifacts to track in git (see "First-time setup" above). Recommend tracking `context.md` on the default branch at minimum.
3. **Read existing artifacts** before writing new ones — phases build on each other
4. **Be challenging** — probe for gaps, don't just accept requirements at face value
5. **Keep artifacts concise** — no boilerplate. Every line should earn its place.
6. **Respect scope** — if the user only wants one phase, do that phase
7. **Update, don't overwrite** — if an artifact exists, ask whether to update or start fresh
8. **Multi-feature support** — use `.gspec/features/<name>/` subdirectories when needed
9. **Use web search proactively** — see the Web Research section

---

## Web Research

**Use web search proactively — don't rely solely on training data.** Key moments to search:

| Phase | Search for |
|-------|-----------|
| **Explore (greenfield)** | Prior art, similar projects, common patterns and pitfalls in the domain |
| **Explore (brownfield)** | Docs for unfamiliar frameworks/libraries found in the codebase |
| **Specify** | Domain-specific requirements, industry standards, competing products |
| **Plan** | Current best practices, latest stable versions, library comparisons, known issues |

Always verify library versions are current. Cite findings when they influence recommendations.
