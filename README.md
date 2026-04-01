<div align="center">
  <h1>gspec</h1>
  <p><strong>Minimal spec-driven development for GitHub Copilot CLI.</strong></p>
  <p>Explore → Specify → Plan → then let Copilot build it.</p>
  <p>
    <img alt="GitHub Copilot CLI skill" src="https://img.shields.io/badge/GitHub%20Copilot%20CLI-skill-0969da">
    <img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-2ea043">
  </p>
  <p>
    <a href="#installation">Installation</a> ·
    <a href="#quick-start">Quick Start</a> ·
    <a href="#examples">Examples</a> ·
    <a href="#when-to-use-gspec">When to use</a> ·
    <a href="#faq">FAQ</a>
  </p>
</div>

---

## What is gspec?

gspec is a [Copilot CLI](https://docs.github.com/copilot/concepts/agents/about-copilot-cli) skill for going from an idea (or an unfamiliar codebase) to a clear implementation plan — through conversation.

Inspired by [spec-kit](https://github.com/github/spec-kit) but stripped to the essentials: **no external CLI, no templates, no scripts, no dependencies.** Just a skill file that teaches your AI agent to understand the codebase, define requirements, and plan before writing code.

gspec writes two kinds of project artifacts: auto-loaded instruction files (`AGENTS.md`, `.github/copilot-instructions.md`) and `.gspec/` documents you can reference with `@` when you need deeper context. After that, Copilot's native **plan mode** takes over for implementation.

Works with **any tech stack**, **any project type**, **greenfield or brownfield**.

---

## Why gspec?

| What gspec adds | Why it matters |
|---|---|
| **Auto-loaded rules + clean handoff** | Explore generates `AGENTS.md` + `.github/copilot-instructions.md` — Copilot loads them automatically in every session. Then you can attach `@.gspec/brief.md`, `@.gspec/spec.md`, or `@.gspec/plan.md` only when deeper context is needed. |
| **Deep project reference** | `.gspec/brief.md` captures project understanding — architecture and debt for brownfield, intent and constraints for greenfield — as a cold-reader briefing you can attach with `@` when a session needs more context. |
| **Brownfield understanding first** | Explore captures architecture, patterns, strengths, and **categorized technical debt** (bugs, legacy patterns, architectural concerns rated by severity) before implementation starts. |
| **Better requirement shaping** | Specify challenges vague asks, separates *what* from *how*, and indexes requirements (R1, R2…) for traceability across plan and test scenarios. |
| **Lightweight planning** | Plan asks for your design ideas first, pushes back on flaws, then recommends concrete approaches — without replacing Copilot's native task workflow. |

---

## Installation

### Option 1: Clone (recommended)

```bash
git clone https://github.com/gcgarriga/gspec.git ~/.copilot/skills/gspec-skill
```

### Option 2: Manual copy

Copy this repository into your Copilot CLI skills folder at:

```
~/.copilot/skills/gspec-skill/
```

### Verify

Start a new Copilot CLI session and type `gspec`. The skill will be automatically discovered.

### Update

```bash
cd ~/.copilot/skills/gspec-skill && git pull
```

> **Requires:** [GitHub Copilot CLI](https://docs.github.com/copilot/concepts/agents/about-copilot-cli) with an active Copilot subscription.

---

## The 3 Phases

| # | Phase | Command | Output | What it does |
|---|-------|---------|--------|-------------|
| 1 | **Explore** | `gspec explore` | `AGENTS.md` + `.github/copilot-instructions.md` + `.gspec/brief.md` | Scans the codebase (brownfield) or captures project intent (greenfield) |
| 2 | **Specify** | `gspec specify` | `.gspec/spec.md` | Defines **what** to build — requirements, scope boundaries |
| 3 | **Plan** | `gspec plan` | `.gspec/plan.md` | Decides **how** to build it — architecture, tech stack *(optional for small features)* |

You can run any phase independently, or use `gspec quick` for a single combined spec. The plan phase is optional for small features.

For named features, spec and plan artifacts can also live under `.gspec/features/<name>/`.

Then hand off to Copilot's native **planning/task workflow** in plan mode. The `.gspec/` artifacts persist across sessions — reference them anytime with `@.gspec/brief.md`. The instruction files (`AGENTS.md` and `.github/copilot-instructions.md`) are auto-loaded by Copilot without needing `@` mentions.

The first time `.gspec/` is created, gspec asks which artifacts to **track in git**. `.gspec/brief.md`, `spec.md`/`plan.md`, and feature directories can be tracked or ignored independently. `AGENTS.md` and `.github/copilot-instructions.md` are recommended to stay tracked on the default branch.

---

## Quick Start

### Fast track: single combined file

```
> gspec quick — add a /health endpoint that returns service status
```

Produces a single combined document instead of 3 separate files. Use for small features, especially on projects that have already been explored.
The output is a single `.gspec/spec.md` with three sections: `Context`, `Requirements`, and `Approach`. If `.gspec/brief.md` exists, gspec uses it to ground the `Context` section.
gspec still checks the current `.gspec/` state first, and may ask whether to update or start fresh if a spec already exists.
See `examples/quick.md` for a full example.

### Start a new project

```
> gspec — I want to build a CLI tool for managing dotfiles
```

The agent walks you through all 3 phases, pausing after each for your input. Then switch to plan mode to implement.

### Explore an existing codebase

```
> gspec explore
```

Reads the code, traces request flows, identifies architecture patterns, and saves a summary.

### Jump to a specific phase

```
> gspec specify          # Define requirements
> gspec plan             # Create implementation plan
> gspec status           # See where you left off
```

### After gspec → use Copilot natively

Switch to plan mode (`Shift+Tab` to cycle modes) and give it context:

```
> @.gspec/brief.md @.gspec/spec.md @.gspec/plan.md
> Implement the feature following these patterns and requirements.
```

Copilot can then generate tasks and implement using the `.gspec/` artifacts as context. The instruction files (`AGENTS.md`, `.github/copilot-instructions.md`) are loaded automatically — no `@` mention needed.
For named features, reference the files under `.gspec/features/<name>/` instead of the root paths.

### Plan multiple features (brownfield)

```
> gspec specify wishlist       # Spec one feature
> gspec plan wishlist          # Plan that same feature later
> gspec specify auth-system    # Spec another
> gspec status                 # Resume and see what already exists
```

Features get their own subdirectory under `.gspec/features/`. For example, `gspec specify wishlist` writes `.gspec/features/wishlist/spec.md`, and `gspec plan wishlist` writes `.gspec/features/wishlist/plan.md`. Re-running a phase with the same feature name continues that feature's artifacts.

---

## Examples

If you want to see the workflow before trying it:

- [`examples/quick.md`](examples/quick.md) — small brownfield change with the compact `gspec quick` flow
- [`examples/greenfield.md`](examples/greenfield.md) — starting a new CLI project from scratch
- [`examples/brownfield.md`](examples/brownfield.md) — planning a wishlist feature in an existing API

These examples show both the conversational prompts and the resulting instruction files plus `.gspec/` artifacts.

---

## When to use gspec

### Great fit

- You are adding a non-trivial feature to an unfamiliar codebase
- The request is underspecified and you want the agent to challenge assumptions
- You want requirements and implementation thinking to persist across sessions
- You want Copilot to follow existing project patterns before it starts coding

### Probably overkill

- A one-line fix or obviously local change
- A tiny change that does not need a spec or a plan
- You are already in the middle of implementation and only need debugging help

`gspec quick` is the middle ground when a full 3-phase workflow would be too much.

---

## How It Works

### Explore (Phase 1)

**Brownfield** — The agent automatically:
- Reads your README and documentation
- Reads the entry point and traces a request end-to-end
- Reads tests to understand domain boundaries
- Extracts coding patterns and principles as **rules to follow** (not just observations)
- Assesses strengths and **categorized technical debt** — bugs, legacy patterns, architectural concerns (severity-rated: 🔴 blocking / 🟡 costly / 🟢 tolerable), and missing infrastructure
- **Generates `AGENTS.md` and `.github/copilot-instructions.md`** from the discovered coding patterns — so every future Copilot session (CLI and VS Code) automatically follows your codebase conventions without needing `@` mentions
- **Generates `.gspec/brief.md`** as a deep project reference — architecture, patterns, and technical debt captured as a cold-reader briefing that can be attached with `@` when sessions need richer context

**Greenfield** — The agent:
- Suggests using `/research` for domain research
- Asks about your project goals, users, and constraints
- Captures prior art and project intent in `.gspec/brief.md`
- Generates the same instruction files so future sessions inherit the project rules

### Specify (Phase 2)

The agent doesn't just take dictation — it **actively challenges** your requirements:
- Identifies gaps ("You mentioned creating items, but what about editing or deleting?")
- Challenges assumptions ("You said real-time — do you need WebSockets, or is polling OK?")
- Surfaces implicit requirements ("This will need authentication — is that in scope?")
- Clarifies priorities ("If you shipped half of this, which half matters most?")
- Separates what from how ("You're specifying Redis — is caching the requirement, or Redis specifically?")

Requirements are **indexed** (R1, R2, R3…) so the Plan and test scenarios can reference them — keeping everything traceable without extra tooling.

Output uses plain-language requirements, not user story ceremony. It can also optionally create a **GitHub Issue** from the spec for project tracking when the repo has a GitHub remote and `gh` is available/authenticated.

### Plan (Phase 3) — *optional*

**Skip this for small features** where implementation is obvious from the spec.

For complex features, the agent **asks for your design ideas first** — you can braindump architecture thoughts, tech preferences, or patterns you'd like to use. Then it **pushes back on design flaws** (over-engineering, contradictions with the spec, unnecessary complexity) before making its own recommendations.

After the design is settled, it **uses web search** to research current options, then **recommends tech stack options** ranked by fit:
- 2-3 options with tradeoffs, ecosystem maturity, and DX considerations
- Specific library recommendations for common concerns (auth, validation, testing, ORM)
- For brownfield: suggestions that complement existing patterns

Finally, it outlines **key test scenarios** mapped to requirement indexes (R1, R2…) — a lightweight verification frame, not a full test suite.

---

## Copilot-Native Integrations

gspec is built specifically for Copilot CLI:

- `AGENTS.md` and `.github/copilot-instructions.md` auto-load project rules
- `@.gspec/brief.md`, `@.gspec/spec.md`, and `@.gspec/plan.md` pass deeper context on demand
- `/research` helps with greenfield domain discovery
- Plan mode handles implementation after gspec has done the thinking
- `gh` can optionally turn a spec into a GitHub Issue

---

## Design Philosophy

- **Spec before code** — Think about *what* and *why* before *how*.
- **Simple until proven otherwise** — Start with the simplest design that meets the need.
- **Lightweight** — No external tools, no installs, no config.
- **Conversational** — The agent asks questions, challenges assumptions, and fills obvious gaps.
- **Incremental** — Run one phase or all three, then resume later if needed.
- **Lightly opinionated** — Opinionated about process, but flexible on solutions.
- **Universal** — Works with any language, framework, or project type.

---

## Comparison with spec-kit

| | gspec | spec-kit |
|---|---|---|
| **Install** | Copy a skill folder | `uv tool install` + `specify init` |
| **Dependencies** | None (just Copilot CLI) | Python 3.11+, uv, git |
| **Setup per project** | None | `specify init --ai copilot` |
| **Templates** | None (freeform markdown) | Structured templates with placeholders |
| **Git automation** | None | Branch creation, PR workflows |
| **Scripts** | None | Shell/PowerShell scripts for setup |
| **Phases** | 3 (explore → specify → plan) + Copilot native | 8+ (constitution, clarify, analyze, checklist...) |
| **Multi-agent** | Copilot CLI only | 20+ agents supported |
| **Best for** | Individual devs who want lightweight structure | Teams wanting full governance and process |

---

## FAQ

**Q: Do I need to run all phases?**
No. Run any phase independently, or use `gspec quick` for a combined single-file output. The plan phase is optional for small features.

**Q: Can I edit the artifacts manually?**
Yes. They're plain markdown. Edit them and the agent will use your changes.

**Q: Does it work with VS Code Copilot?**
gspec is built for Copilot CLI. For VS Code, consider [spec-kit](https://github.com/github/spec-kit) which supports `.github/agents/` and `.github/prompts/`.

**Q: Why not use gspec for tasks and implementation too?**
Copilot CLI already has excellent native planning/task and implementation capabilities in plan mode. gspec focuses on what Copilot doesn't do natively: persistent codebase understanding, structured requirements, and researched tech stack planning.

**Q: What if I start in one session and continue in another?**
Just say `gspec` or `gspec status`. The agent reads `.gspec/` and picks up where you left off. If the codebase has changed since `brief.md` was last written, gspec detects the drift and offers to update only the affected sections — no need to re-explore from scratch.

**Q: Does `.gspec/` get committed to git?**
Your choice. The first time gspec creates `.gspec/`, it asks whether the artifacts should be tracked by git (so they can be shared with teammates and visible in PRs) or kept local by adding `.gspec/` to `.gitignore`. gspec never runs `git commit` for you — you always control what actually gets committed to the repo.

**Q: Can I use this with other AI agents (Claude Code, Cursor, etc.)?**
The SKILL.md format is Copilot CLI-specific, but the methodology and artifact structure work anywhere. You could adapt the prompts for other agents.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## License

MIT
