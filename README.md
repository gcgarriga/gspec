# gspec

**Minimal spec-driven development for GitHub Copilot CLI.**

Think before you code. Explore → Specify → Plan → then let Copilot build it.

**Quick links:** [Installation](#installation) · [Quick Start](#quick-start) · [Examples](#examples) · [When to use gspec](#when-to-use-gspec) · [FAQ](#faq)

---

## What is gspec?

gspec is a [Copilot CLI](https://docs.github.com/copilot/concepts/agents/about-copilot-cli) skill for going from an idea (or an unfamiliar codebase) to a clear implementation plan — all through conversation.

Inspired by [spec-kit](https://github.com/github/spec-kit) but stripped to the essentials: **no external CLI, no templates, no scripts, no dependencies.** Just a skill file that teaches your AI agent to understand the codebase, define requirements, and plan before writing code.

gspec writes persistent artifacts to `.gspec/`, so future Copilot sessions can resume with context instead of starting from scratch. After that, Copilot's native **plan mode** takes over for implementation.

Works with **any tech stack**, **any project type**, **greenfield or brownfield**.

---

## Why gspec?

- **It gives Copilot durable memory for a project.** `context.md`, `spec.md`, and `plan.md` survive across sessions.
- **It is especially useful in brownfield work.** Explore captures architecture, patterns, strengths, and debt before implementation starts.
- **It improves requirement quality.** Specify challenges vague asks and turns them into scoped, decision-ready requirements.
- **It keeps planning lightweight.** Plan recommends concrete approaches and libraries without replacing Copilot's native task workflow.
- **It is Copilot-native.** It can suggest `/research`, generate `copilot-instructions.md`, and hand off cleanly with `@` file mentions.

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
| 1 | **Explore** | `gspec explore` | `.gspec/context.md` | Scans the codebase (brownfield) or captures project intent (greenfield) |
| 2 | **Specify** | `gspec specify` | `.gspec/spec.md` | Defines **what** to build — requirements, scope boundaries |
| 3 | **Plan** | `gspec plan` | `.gspec/plan.md` | Decides **how** to build it — architecture, tech stack *(optional for small features)* |

For named features, spec and plan artifacts can also live under `.gspec/features/<name>/`.

Then hand off to Copilot's native **planning/task workflow** in plan mode. The `.gspec/` artifacts persist across sessions — reference them anytime with `@.gspec/context.md`.

---

## Quick Start

### Quick job (skip the ceremony)

```
> gspec quick — add a /health endpoint that returns service status
```

Produces a single combined document instead of 3 separate files. Use for small features.
The output is a single `.gspec/spec.md` with three sections: `Context`, `Requirements`, and `Approach`.
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
> @.gspec/context.md @.gspec/spec.md @.gspec/plan.md
> Implement the feature following these patterns and requirements.
```

Copilot can then generate tasks and implement using the `.gspec/` artifacts as context.

### Plan multiple features (brownfield)

```
> gspec specify wishlist       # Spec one feature
> gspec plan wishlist          # Plan that same feature later
> gspec specify auth-system    # Spec another
> gspec status                 # Resume and see what already exists
```

Features get their own subdirectory under `.gspec/features/`. Re-running a phase with the same feature name continues that feature's artifacts.

---

## Examples

If you want to see the workflow before trying it:

- [`examples/quick.md`](examples/quick.md) — small brownfield change with the compact `gspec quick` flow
- [`examples/greenfield.md`](examples/greenfield.md) — starting a new CLI project from scratch
- [`examples/brownfield.md`](examples/brownfield.md) — planning a wishlist feature in an existing API

These examples show both the conversational prompts and the resulting `.gspec/` artifacts.

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

## Project Artifacts

gspec saves all artifacts in `.gspec/` at your project root:

```
.gspec/
├── context.md              # Codebase understanding or project intent
├── spec.md                 # Requirements and scope
├── plan.md                 # Technical implementation plan
└── features/               # Multi-feature support (optional)
    └── wishlist/
        ├── spec.md
        └── plan.md
```

> **Tip:** Add `.gspec/` to your `.gitignore` if you don't want to commit spec artifacts, or commit them if you want the spec to travel with the code.

---

## How It Works

### Explore (Phase 1)

**Brownfield** — The agent automatically:
- Reads your README and documentation
- Reads the entry point and traces a request end-to-end
- Reads tests to understand domain boundaries
- Extracts coding patterns and principles as **rules to follow** (not just observations)
- Assesses strengths and technical debt
- **Offers to generate or update `.github/copilot-instructions.md`** from the discovered patterns — so every future Copilot session (CLI and VS Code) can automatically follow your codebase conventions

The output is written as a **cold-reader briefing** — any future AI session can read `context.md` and immediately understand the project.

**Greenfield** — The agent suggests using `/research` for deep domain research, then asks about your project goals, users, constraints, and prior art.

### Specify (Phase 2)

The agent doesn't just take dictation — it **actively challenges** your requirements:
- Identifies gaps ("You mentioned creating items, but what about editing or deleting?")
- Challenges assumptions ("You said real-time — do you need WebSockets, or is polling OK?")
- Surfaces implicit requirements ("This will need authentication — is that in scope?")
- Clarifies priorities ("If you shipped half of this, which half matters most?")

Output uses plain-language requirements, not user story ceremony. Optionally creates a **GitHub Issue** from the spec for project tracking.

### Plan (Phase 3) — *optional*

**Skip this for small features** where implementation is obvious from the spec.

For complex features, the agent **uses web search** to research current options, then **recommends tech stack options** ranked by fit:
- 2-3 options with tradeoffs, ecosystem maturity, and DX considerations
- Specific library recommendations for common concerns (auth, validation, testing, ORM)
- For brownfield: suggestions that complement existing patterns

### Implementation → Copilot Native

After gspec, switch to plan mode (`Shift+Tab`) and reference the artifacts:

```
> @.gspec/context.md @.gspec/spec.md @.gspec/plan.md
> Implement the wishlist feature following these patterns and requirements.
```

- Copilot can generate tasks and implementation steps from these artifacts
- Reference `.gspec/` artifacts with `@` mentions for context
- The patterns section in `context.md` ensures code matches the codebase

---

## Copilot-Native Integrations

gspec is built specifically for Copilot CLI, not just running on it:

| Feature | How gspec uses it |
|---------|------------------|
| **`.github/copilot-instructions.md`** | Explore phase offers to generate or update this from discovered coding patterns so future Copilot sessions can auto-load your conventions |
| **`/research`** | Greenfield explore suggests deep domain research via Copilot's multi-source research |
| **`@` file mentions** | Handoff prompts use `@.gspec/context.md`, `@.gspec/spec.md`, and `@.gspec/plan.md` to pass persistent context |
| **Plan mode** | Handoff suggests `Shift+Tab` to enter plan mode for task generation |
| **`gh` CLI** | Optionally creates GitHub Issues from the spec for project tracking |
| **Skills system** | User-level skill — auto-discovered in every session, every project |

---

## Skill Contents

```
gspec-skill/
├── SKILL.md                        # Core skill definition (all phase instructions)
├── README.md                       # This file
├── CONTRIBUTING.md                 # Contributor workflow and local testing guide
├── references/
│   └── explore-guide.md            # Detailed codebase exploration patterns
├── examples/
│   ├── quick.md                    # Compact example for gspec quick
│   ├── greenfield.md               # Full example: CLI tool from scratch
│   └── brownfield.md               # Full example: feature on existing API
└── .claude-plugin/
    └── plugin.json                 # Plugin metadata
```

---

## Design Philosophy

| Principle | What it means |
|-----------|--------------|
| **Spec before code** | Think about *what* and *why* before *how*. Reduces rework. |
| **Simple until proven otherwise** | Start with the simplest design. Add complexity only when a requirement demands it. |
| **Lightweight** | No external tools, no installs, no config. One skill file. |
| **Conversational** | The agent asks questions, challenges assumptions, and suggests what you might be missing. |
| **Incremental** | Run one phase or all three. Resume across sessions. Update artifacts anytime. |
| **Opinionated** | Recommends tech stacks and libraries — doesn't just ask what you want. |
| **Universal** | Works with any language, framework, or project type. |

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
Just say `gspec` or `gspec status`. The agent reads `.gspec/` and picks up where you left off. That's the whole point — artifacts persist.

**Q: Can I use this with other AI agents (Claude Code, Cursor, etc.)?**
The SKILL.md format is Copilot CLI-specific, but the methodology and artifact structure work anywhere. You could adapt the prompts for other agents.

---

## Contributing

If you want to improve gspec:

1. Edit the files in this repo locally.
2. Start a **new** Copilot CLI session so the updated skill is reloaded.
3. Test at least these flows:
   - `gspec explore` in a brownfield repo
   - `gspec quick` for a small feature
   - `gspec` end-to-end in an empty directory
4. Keep `SKILL.md`, `README.md`, `examples/`, and `references/` consistent with each other.

If you change the process, update the examples too — they are part of the product.

---

## License

MIT
