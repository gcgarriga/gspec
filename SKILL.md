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
Phase 1: Explore   →  AGENTS.md + .github/copilot-instructions.md  (auto-loaded rules)
                   →  .gspec/brief.md                       (project knowledge, attach with @)
Phase 2: Specify   →  .gspec/spec.md                        (define what to build)
Phase 3: Plan      →  .gspec/plan.md                        (decide how to build it)
→ Hand off to Copilot's plan mode for tasks and implementation
```

Each phase builds on the previous. Instruction files (`AGENTS.md`, `.github/copilot-instructions.md`) live in standard locations for auto-loading. Spec and plan artifacts are stored in `.gspec/` at the project root. All outputs **persist across sessions** — any future Copilot session can read them.

For projects with multiple features, use `.gspec/features/<feature-name>/` for per-feature artifacts (spec.md, plan.md), while `brief.md` stays at the `.gspec/` root since it describes the whole project.

**Brief persistence:** `brief.md` is project-wide — it describes the codebase, not a feature. Maintain it on the default branch (`main`/`master`) so new branches start with the current version. Existing feature branches need an explicit merge or rebase to pick up later updates. Feature-specific files (`spec.md`, `plan.md`) naturally live on their feature branches.

## Design Philosophy

These principles govern how gspec thinks — and how it guides future implementation:

1. **Simple until proven otherwise** — Always recommend the simplest design that meets the requirements. Add complexity only when a concrete requirement demands it.
2. **Right-size everything** — A 50-line script doesn't need an architecture diagram. A distributed system does. Scale the depth of each phase to the project's actual complexity.
3. **Fewer moving parts** — Prefer stdlib over libraries. Prefer one library over three. Every dependency is a liability.
4. **Match the codebase** — For brownfield: match existing patterns exactly. Don't introduce new paradigms. For greenfield: establish patterns early and follow them consistently.
5. **Artifacts should be useful, not thorough** — Every line in brief.md, spec.md, and plan.md should earn its place. If it doesn't help someone build the right thing, cut it.

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

Feature names must be a **single slug** (lowercase, hyphens for spaces): `wishlist`, `auth-system`, `price-alerts`. If the user provides a multi-word name without hyphens (e.g., `gspec specify add auth system`), ask them to clarify which part is the feature name. Don't guess.

If no feature name is provided, use the root artifacts (`.gspec/spec.md`, `.gspec/plan.md`).

**Fast-track:** If the user says "gspec quick" with a description, produce a single combined spec.md containing:

```markdown
# [Feature Name]

## Context
[2-3 sentences: what the project is, relevant existing patterns if brownfield]

## Requirements
[What to build — plain language]

## Approach
[How to build it — tech choices, key decisions]
```

Use this for small features that don't need 3 separate files. On brownfield projects, it works best after Phase 1 because the instruction files are already auto-loaded and `brief.md` can ground the Context section. If `brief.md` exists, read it for Context — and run the staleness check as usual, since `gspec quick` depends on accurate project context.

`gspec quick` follows the same naming rules as other phases:
- `gspec quick — add a health endpoint` → writes `.gspec/spec.md` (root)
- `gspec quick health-check — add a health endpoint` → writes `.gspec/features/health-check/spec.md`

If a `spec.md` already exists at the target path, apply the same "update or start fresh" rule — ask before overwriting.

---

## On Entry: Always Check Status First

**Before running any phase**, check the `.gspec/` directory to understand current state:

1. Check if `.gspec/` exists and check for `AGENTS.md` (repo root) and `.github/copilot-instructions.md`
2. Check which artifacts exist: `brief.md`, `spec.md`, `plan.md`
3. Check for feature subdirectories: `.gspec/features/*/`
4. **Check brief.md freshness** (see staleness detection below)
5. Report status to the user:

```
📋 gspec status:
✅ brief.md — last updated [date]
✅ AGENTS.md — synced with brief.md
✅ copilot-instructions.md — synced with AGENTS.md
✅ spec.md — [feature name from header]
⬜ plan.md — not yet created

Suggested next step: gspec plan
```

If the user asked for a specific phase, proceed with it. If they just said "gspec", suggest the next logical phase based on what exists.

If artifacts already exist and the user re-runs a phase, ask whether they want to **update** the existing artifact or **start fresh**. For multi-feature projects, ask which feature they're working on unless it is already explicit in the command.

### Staleness detection for brief.md

Every time you read `brief.md` (on entry, before specify, before plan), check whether it's still current:

1. **Read the `Date:` field** from the brief.md header
2. **Compare against recent git activity** (skip this step if the project is not a git repo — fall back to comparing the `Date:` field against today's date and warn if it's more than 2 weeks old):
   ```bash
   # What changed since brief.md was last updated?
   git log --oneline --since="YYYY-MM-DD" -- . ':!.gspec' | wc -l
   git log --stat --since="YYYY-MM-DD" -- . ':!.gspec' | head -60
   ```
3. **Assess impact** — look at *what* changed, not just *how much*:
   - Changes to the data model (schema files, migrations, ORM models) → Data Layer section is stale
   - New or removed routes/handlers → Architecture and Key Directories sections are stale
   - Dependency changes (package.json, go.mod, requirements.txt) → Tech Stack section is stale
   - New test files or changed test patterns → Testing conventions may be stale
   - The debt section goes stale fastest — new TODOs, resolved issues, and pattern changes accumulate silently

4. **Report and offer targeted refresh** instead of a full re-explore:

```
⚠️ brief.md may be stale (last updated 2025-03-12, 47 commits since):
  - prisma/schema.prisma changed — Data Layer section may be outdated
  - 2 new route files added in src/routes/ — Architecture section may be incomplete
  - package.json gained 3 new dependencies — Tech Stack section may be outdated

Options:
  1. Targeted refresh — I'll re-read the changed files and update only the affected sections
  2. Full re-explore — re-run gspec explore from scratch
  3. Skip — proceed with current brief.md as-is
```

**Targeted refresh flow:**
- Re-read only the files that changed since the last explore date
- Update only the affected sections of brief.md, preserving sections that are still accurate
- Update the `Date:` field to today
- Present a diff summary of what changed in brief.md so the user can verify
- If the Rules were updated during a targeted refresh, also update `AGENTS.md` and `.github/copilot-instructions.md` to stay in sync

**When to skip the staleness check:**
- The user explicitly said `gspec explore` — they already want a fresh explore, no need to check
- brief.md was updated today — it's current
- Fewer than 5 commits since the last update and none touch key structural files (schema, entry point, dependencies) — not worth flagging

### First-time setup: ask about git tracking

When creating the `.gspec/` directory for the first time (i.e., it does not already exist), ask the user which artifacts to track in git:

> Which artifacts should be tracked in git?
>
> - **`AGENTS.md`** — project instructions for AI agents. **Recommended** to track on the default branch.
> - **`.github/copilot-instructions.md`** — auto-loaded by Copilot. **Recommended** to track on the default branch.
> - **`.gspec/brief.md`** — deep project reference. Recommended to track on the default branch.
> - **`spec.md` / `plan.md`** — can be project-wide or feature-specific. Track them if you want specs visible in PRs.
> - **Feature directories** (`.gspec/features/`) — feature-specific specs and plans.
>
> Note: gspec will not run `git add` or `git commit` for you.

Configure `.gitignore` based on the user's choices. Examples:

- **Track everything** — do nothing; `.gspec/` remains unignored.
- **Track only `brief.md`** — add to `.gitignore` (order matters — the exception must come after the wildcard):
  ```
  .gspec/*
  !.gspec/brief.md
  ```
- **Track nothing from .gspec/** — add `.gspec/` to `.gitignore`.
- **Keep instruction files local-only** — add either path explicitly if you do not want to track it:
  ```
  AGENTS.md
  .github/copilot-instructions.md
  ```

Only ask once — if `.gspec/` already exists and tracking is already configured, skip this question. However, if `.gitignore` contains a coarse `.gspec/` rule (ignoring everything) and the user re-runs gspec, offer to replace it with a granular configuration (e.g., tracking only `brief.md`).

---

## Phase 1: Explore

**Goal:** Build a deep, shared understanding of the project — and set up the codebase for AI agents.

**Primary output:** `AGENTS.md` (repo root) + `.github/copilot-instructions.md` — auto-loaded instruction files that make every AI session better immediately.

**Always generated:** `.gspec/brief.md` — deep project reference for spec/plan phases. Attach with `@.gspec/brief.md` when the agent needs architectural understanding.

**Branch guidance:** Instruction files and `brief.md` are project-wide — they should be committed to the default branch (`main`/`master`) so every feature branch inherits them.

### Brownfield (existing codebase detected)

Detection heuristic:
- If `package.json`, `Cargo.toml`, `go.mod`, `requirements.txt`, `*.csproj`, `Gemfile`, `pom.xml`, or similar exist → brownfield
- If directory has significant source files (>5 non-config files) → brownfield
- Otherwise → greenfield

**Monorepo detection:** If you find `turbo.json`, `nx.json`, `lerna.json`, `pnpm-workspace.yaml`, or multiple independent dependency files in subdirectories (`packages/`, `apps/`, `services/`), ask the user which package to focus on before exploring. Don't try to cover every package at once — it produces a shallow brief.md.

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

#### Step 4: Produce the outputs

After exploring, produce three files from the same exploration:

**A. Instruction files (primary — auto-loaded by agents)**

Write `AGENTS.md` at the repo root and `.github/copilot-instructions.md`. These contain the prescriptive rules from Step 3 — the stuff agents need in every session. Keep both files tight (<500 words). They should cover:

1. **Commands** — how to build, test, lint, run
2. **Rules** — the prescriptive coding patterns from Step 3 (error handling, naming, architecture patterns, testing conventions)
3. **Boundaries** — what agents must not touch, using three tiers:
   - ✅ **Always:** things agents should do without asking (run tests, follow naming conventions)
   - ⚠️ **Ask first:** changes that need human approval (schema changes, new dependencies, config changes)
   - 🚫 **Never:** hard limits (commit secrets, modify vendor/, delete tests)
4. **Canonical examples** — point to real files as templates for new code when the codebase has them. For greenfield projects with no implementation yet, omit this section.

Format for `AGENTS.md`:
```markdown
# AGENTS.md

<!-- Generated by gspec explore — edit freely, this is your file -->

## Commands

| Task    | Command          |
|---------|------------------|
| Dev     | `npm run dev`    |
| Test    | `npm test`       |
| Build   | `npm run build`  |
| Lint    | `npm run lint`   |

## Rules

[Extract the prescriptive rules from Step 3 — written as imperatives]

## Boundaries

- ✅ **Always:** Run tests before committing. Follow naming conventions. Use the service layer for business logic.
- ⚠️ **Ask first:** Adding new dependencies. Changing database schema. Modifying CI config.
- 🚫 **Never:** Commit secrets or .env files. Modify files in vendor/. Remove failing tests.

## Canonical Examples

Follow these files as templates for new code:
- **Route handler:** `src/routes/products.ts`
- **Service:** `src/services/product.ts`
- **Test:** `src/__tests__/routes/products.test.ts`
```

For greenfield projects with no implementation yet, omit the `Canonical Examples` section until there are real files to point at.

For `.github/copilot-instructions.md`, use the same content but with Copilot-appropriate framing:
```markdown
# Copilot Instructions

<!-- Generated by gspec explore — edit freely, this is your file -->

## Coding Conventions

[Same rules as AGENTS.md]

## Testing Conventions

[Same testing rules as AGENTS.md]

## Boundaries

[Same boundaries as AGENTS.md]
```

If either file already exists, show the user the diff and ask before updating.

**B. Brief (always generated — attach with `@` for deep context)**

Write `.gspec/brief.md`. This is the deep project reference — architecture, data model, debt. It does NOT duplicate the rules in instruction files. Cover these:

1. **What this project does** — 2-3 sentences
2. **Tech stack** — table format (language, framework, database, etc.)
3. **Architecture** — how code is organized, request flow, key directories
4. **Data layer** — database, ORM, key models and relationships
5. **Strengths** — what's well-designed, patterns worth preserving
6. **Technical debt and known issues** — structured assessment (see Debt format below)
7. **Coding rules reference** — a single line: "See `AGENTS.md` and `.github/copilot-instructions.md` for coding conventions and boundaries."

##### Debt and Issues Format

Don't just list debt as flat bullets. Categorize and prioritize:

```markdown
## Technical Debt and Known Issues

### Bugs and broken behavior
[Failing tests, known runtime errors, error-prone code paths. Only include issues you can **verify** — a failing test, an obvious logic error, a documented known issue. Don't speculate.]

### Legacy patterns
[Patterns that work but don't match current best practices or the project's own stated conventions. Flag when the codebase has two competing approaches (e.g., some routes use middleware auth, others check auth inline) — the inconsistency itself is the issue.]

### Architectural concerns
[Structural issues that affect maintainability or extensibility. Classify severity:]
- 🔴 **Blocking** — Would prevent a reasonable feature from being built without refactoring first (e.g., circular dependencies, hardcoded assumptions that break when extended)
- 🟡 **Costly** — Won't block work but will make it significantly harder or more error-prone (e.g., 600-line god objects, missing abstractions that force duplication)
- 🟢 **Tolerable** — Worth noting for future cleanup but safe to work around (e.g., inconsistent naming, missing rate limiting)

### Missing infrastructure
[Standard concerns not yet addressed: logging, monitoring, rate limiting, input sanitization, etc. Only flag what's actually missing — don't list everything a production app "should" have if it's a prototype.]
```

**How to discover these:**
- **Read test results** if easily runnable — failing tests are the most reliable bug signal
- **Search for TODOs, FIXMEs, HACKs, XXXs** — developers leave breadcrumbs
- **Check issue tracker** if accessible (GitHub Issues via `gh issue list`) for known bugs
- **Look for inconsistencies** — when 8 routes follow a pattern and 2 don't, that's likely debt, not intentional
- **Check git blame on complex files** — rapid churn often signals a problem area

The goal is not to audit the codebase exhaustively — it's to surface issues that would **surprise or trip up** someone building a new feature.

#### Step 5: Confirm and commit

After writing all three files, present a summary:

```
✅ gspec explore complete:

Instruction files (auto-loaded in Copilot):
  📄 AGENTS.md — [N] rules, [N] boundaries
  📄 .github/copilot-instructions.md — synced with AGENTS.md

Project reference (attach with @.gspec/brief.md):
  📄 .gspec/brief.md — architecture, data model, [N] debt items

In Copilot, these instruction files are loaded automatically. For other agents,
attach or adapt them as needed. No @ mention needed for Copilot sessions.
```

Ask if anything is missing or incorrect. For brownfield, double-check: are the rules written as imperatives? Are boundaries using the three-tier format? Is the debt section categorized and severity-rated?

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

Capture answers in `.gspec/brief.md` as project intent.

After gathering user input, **enrich with web research:**
- Search for existing projects in the same space — summarize what exists and where this project would differentiate
- Search for common architectural patterns for this type of project
- Surface any domain-specific gotchas (e.g., "photo management apps need EXIF parsing", "real-time chat needs connection state management")
- Include a "Prior Art & Inspiration" section in `brief.md` with links and brief notes on what to learn from existing solutions

For greenfield, the instruction files capture initial decisions (chosen stack, planned conventions), and brief.md captures project intent, constraints, and prior art. Both will be updated as the codebase grows.

### brief.md Format

Write structured markdown with clear sections:

**Write as if briefing a new developer (or AI agent) who needs to understand the project for architectural decisions — not for routine coding (that's what the instruction files handle).**

**Brownfield default format:**

```markdown
# Project Brief

**Type:** brownfield
**Date:** YYYY-MM-DD

## What this is
[2-3 sentences]

## Stack

| Layer       | Choice                          |
|-------------|---------------------------------|
| Language    | [e.g., TypeScript 5.3]          |
| Framework   | [e.g., Express.js 4.18]         |
| Database    | [e.g., PostgreSQL via Prisma]   |
| ...         | ...                             |

## Architecture
[Request flow diagram + directory map]

## Data Model
[Key entities and relationships. Reference schema file.]

## Strengths
[What's well-designed, patterns worth preserving]

## Technical Debt and Known Issues
[Categorized and severity-rated — see Debt format]

## Coding Rules
See `AGENTS.md` and `.github/copilot-instructions.md` for coding conventions and boundaries.
```

**Greenfield default format:**

```markdown
# Project Brief

**Type:** greenfield
**Date:** YYYY-MM-DD

## Project Intent
[2-3 sentences on what this is, who it serves, and why it should exist]

## Domain
[Problem space and what makes it tricky]

## Prior Art & Inspiration
- **[Project/tool]** — [what to learn from it]
- **[Project/tool]** — [what to borrow or avoid]

## Target Users
[Who this is for]

## Key Constraints
[Hard constraints — tech, compliance, team skills, integration points]

## Coding Rules
See `AGENTS.md` and `.github/copilot-instructions.md` for coding conventions and boundaries.
```

After writing, present a brief summary to the user and ask if anything is missing or wrong. For brownfield, double-check: are the **patterns and principles** written as actionable rules, not just observations? Is the **debt section** categorized and prioritized, not just a flat list?

---

## Phase 2: Specify

**Goal:** Define **what** to build and **why**. Not how.

**Prerequisite:** `.gspec/brief.md` should exist. For brownfield, **strongly recommend running Phase 1 first** — specs written without codebase understanding tend to ignore existing patterns and constraints. If the user skips it, warn once and proceed. For greenfield, ask the user to describe the project context verbally.

**Output:** `.gspec/spec.md` (or `.gspec/features/<name>/spec.md` for multi-feature projects)

### Process

1. **Read** `.gspec/brief.md` for project context
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
   - **Separate what from how:** When the user mixes design decisions into requirements ("I want a Redis cache"), separate the need from the solution. Capture the requirement ("Data must be cached to avoid repeated expensive lookups") and move the solution idea to a "Design Notes" section for the Plan phase.
   - **Push back on premature design choices:** "You're specifying Redis — is caching the requirement, or is Redis specifically? Let's nail down the need first, then pick the solution in the Plan phase."
4. **Write** `.gspec/spec.md`

### spec.md Format

Write freeform markdown. Keep it practical — no ceremony.

```markdown
# Specification: [Feature/Project Name]

## Overview
[2-3 sentence summary of what this is and why it matters]

## Requirements
[Index each requirement so Plan and test scenarios can reference them by ID.]

R1. [First requirement — plain language, not user stories]
R2. [Second requirement]
R3. ...

## Design Notes
[Optional — solution ideas the user mentioned during Specify that were separated from requirements. These feed into the Plan phase. Delete if empty.]

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

1. **Read** `.gspec/brief.md` and `.gspec/spec.md`
2. **Ask for the user's design ideas first** — before recommending anything:
   - "Do you have thoughts on how you'd approach this? Architecture ideas, tech preferences, patterns you'd like to use?"
   - Let them braindump freely — accept rough, mixed, incomplete input
   - Review any "Design Notes" captured in `spec.md` during the Specify phase
   - **Push back on design flaws:**
     - **Over-engineering:** "You suggested microservices, but the spec has 3 endpoints and one data model. A monolith with clean module boundaries would be simpler — what's driving the microservices choice?"
     - **Contradictions with spec:** "Your spec says 'must work offline' (R3) but this architecture requires a live API — which takes priority?"
     - **Unnecessary complexity:** "Do you need a message queue here, or would a simple background job cover it?"
   - Acknowledge good ideas explicitly — adopt them with rationale
   - If the user has no design ideas, that's fine — proceed with agent recommendations
3. **Recommend a tech stack:**
   - Analyze the requirements from `spec.md` and context from `brief.md`

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
4. **Create the implementation plan** covering:
   - **Architecture overview** — High-level design, component diagram in text
   - **Tech stack** — Specific technologies and versions with rationale
   - **Data model** — Key entities and relationships (if applicable)
   - **API / Interface design** — Key endpoints, interfaces, or contracts (if applicable)
   - **Key implementation decisions** — Patterns, libraries, approaches chosen and why
   - **Dependencies & prerequisites** — What needs to exist before implementation
   - **Existing debt to address** — Cross-reference `brief.md`'s debt section. Call out any 🔴 Blocking or 🟡 Costly items that directly affect this feature. If a debt item must be fixed before or during implementation, say so explicitly. Don't repeat the full debt list — only the items that matter for *this* feature.
   - **Risks & mitigations** — What could go wrong, how to handle it

   **Right-size the plan:** Not every section is needed. Skip sections that don't apply. A CLI tool might only need Tech Stack + Implementation Approach. A full-stack app needs most sections. Ask: "Would removing this section lose important information?" If no, cut it.

5. **Simplicity check** — Before writing, ask yourself:
   - Is this the simplest architecture that meets ALL the requirements?
   - Am I adding layers/abstractions that aren't justified by a concrete requirement?
   - Could I remove a dependency and use stdlib instead?
   - Am I pattern-matching from "best practices" or from actual project needs?
   If the plan feels heavy, simplify it. Challenge your own recommendations.
6. **Outline key test scenarios** — Map critical test cases to requirement indexes:
   - Focus on scenarios that verify the spec is met, not exhaustive test plans
   - Each scenario references one or more Rx: "Test: R2, R5 — user cannot access another user's data after session expires"
   - Include happy paths, key error paths, and edge cases from the spec
   - This is a lightweight verification frame, not a full test suite — keep it to the scenarios that matter most
7. **Write** `.gspec/plan.md`

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

## Existing Debt to Address
[Only debt from brief.md that directly affects this feature — skip if none]

## Risks
[What could go wrong and how to mitigate]

## Test Scenarios
[Key scenarios that verify spec requirements — reference Rx identifiers. Skip if obvious from spec.]
- R1, R3: [scenario description]
- R2: [scenario description]
- R4, R5: [edge case scenario]
```

After writing, **run the Self-Review Quality Gate** (see below) for the plan, then present the summary and ask the user to confirm or refine.

---

## After gspec: Handoff to Copilot

Once the last needed artifact is complete, actively help the user transition:

1. **Summarize what was produced** — list the `.gspec/` files and what each contains
2. **Give the user a ready-to-use prompt** they can copy. Example:

   > "Implement the feature described in .gspec/plan.md. Reference .gspec/spec.md for requirements. Attach @.gspec/brief.md if you need architectural context."

   For named features, use the paths under `.gspec/features/<name>/` in the handoff prompt.

3. **Suggest switching to plan mode** (`Shift+Tab`) — Copilot's native mode for task generation and implementation
4. **Remind the user** that instruction files are auto-loaded — agents already know the coding rules. For deeper context, attach `@.gspec/brief.md`.

The value of gspec is that these artifacts are a **persistent briefing document** for any AI agent. No more re-explaining the codebase or requirements every session.

---

## Self-Review: Quality Gate

**After generating any artifact, review it against these checks BEFORE presenting to the user.** Fix issues silently, then present the improved version.

### Brief (brownfield)
1. Architecture describes actual flow, not just folder names
2. No section is just a list of names — each has enough explanation to be actionable
3. Debt is categorized and severity-rated — not a flat bullet list
4. Every debt item is verifiable — based on something found, not speculation
5. Rules are NOT duplicated in brief.md — they reference instruction files

### Brief (greenfield)
1. Project intent is specific enough to build from — not just "a web app" but what it does, for whom, and why
2. Constraints are concrete — "must run on AWS Lambda" not "should be scalable"
3. Prior art section exists and names at least 2-3 existing solutions with what to learn from each
4. Target users are identified — even if it's "me, on my own machines"

### Instruction files (AGENTS.md / copilot-instructions.md)
1. Rules are written as imperatives ("Do X"), not observations ("X is used")
2. Commands are exact and complete — a new session can build/test from these alone
3. Boundaries use three-tier format (✅ Always, ⚠️ Ask first, 🚫 Never)
4. Canonical examples point to real files that exist in the codebase, or the section is omitted for greenfield projects with no implementation yet
5. Total length is under 500 words — tight enough to not pollute context
6. AGENTS.md and copilot-instructions.md are in sync

### Spec
1. Every requirement has an index (R1, R2...) — no unnumbered requirements
2. No false open questions — if it's a technical fact, it's a requirement, not a question
3. Error paths covered for every integration point
4. No vague "automatically" without specifying the mechanism
5. State management is explicit if the system has loops or accumulating data
6. Design ideas are in "Design Notes", not mixed into requirements

### Plan
1. Every Rx from spec.md is addressed — no orphaned requirements
2. Simplest architecture that meets all requirements — nothing speculative
3. Environment setup commands included if dependencies are listed
4. No vague verbs — "parse JSON into dict" not "handle the response"
5. Complexity distributed evenly — no step does >3 things while others do 1
6. If user provided design ideas, they are either adopted with rationale or rejected with explanation
7. Test scenarios reference valid Rx identifiers from the spec

### Cross-artifact consistency
1. Before writing any artifact, re-read upstream artifacts and flag conflicts with the user
2. Plan does not contradict constraints or scope boundaries in spec
3. If re-running a phase, warn about stale downstream artifacts ("spec.md was updated — plan.md may need updating too")

---

## General Rules

1. **Check `.gspec/` status on entry** — report what exists, suggest next step
2. **On first `.gspec/` creation** — ask which artifacts to track in git (see "First-time setup" above). Recommend tracking `AGENTS.md`, `copilot-instructions.md`, and `brief.md` on the default branch.
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
