# gspec

**Minimal spec-driven development for GitHub Copilot CLI.**

Think before you code. Explore → Specify → Plan → then let Copilot build it.

---

## What is gspec?

gspec is a [Copilot CLI](https://docs.github.com/copilot/concepts/agents/about-copilot-cli) skill that gives you a structured workflow for going from an idea (or existing codebase) to a clear implementation plan — all through conversation.

Inspired by [spec-kit](https://github.com/github/spec-kit) but stripped to the essentials: **no external CLI, no templates, no scripts, no dependencies.** Just a skill file that teaches your AI agent to understand the codebase, define requirements, and plan before writing code. Then Copilot's native `/plan` mode takes over for implementation.

Works with **any tech stack**, **any project type**, **greenfield or brownfield**.

---

## Installation

### Option 1: Clone (recommended)

```bash
git clone https://github.com/gcgarriga/gspec.git ~/.copilot/skills/gspec-skill
```

### Option 2: Manual copy

Copy the `gspec-skill/` directory into your Copilot CLI skills folder:

```
~/.copilot/skills/gspec-skill/
```

### Verify

Start a new Copilot CLI session and type `gspec`. The skill will be automatically discovered.

> **Requires:** [GitHub Copilot CLI](https://docs.github.com/copilot/concepts/agents/about-copilot-cli) with an active Copilot subscription.

---

## The 3 Phases

| # | Phase | Command | Output | What it does |
|---|-------|---------|--------|-------------|
| 1 | **Explore** | `gspec explore` | `.gspec/context.md` | Scans the codebase (brownfield) or captures project intent (greenfield) |
| 2 | **Specify** | `gspec specify` | `.gspec/spec.md` | Defines **what** to build — requirements, scope boundaries |
| 3 | **Plan** | `gspec plan` | `.gspec/plan.md` | Decides **how** to build it — architecture, tech stack *(optional for small features)* |

Then hand off to Copilot's native `/plan` mode for task generation and implementation. The `.gspec/` artifacts persist across sessions — reference them anytime with `@.gspec/context.md`.

---

## Quick Start

### Quick job (skip the ceremony)

```
> gspec quick — add a /health endpoint that returns service status
```

Produces a single combined document instead of 3 separate files. Use for small features.

### Start a new project

```
> gspec — I want to build a CLI tool for managing dotfiles
```

The agent walks you through all 3 phases, pausing after each for your input. Then switch to `/plan` mode to implement.

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
> @.gspec/context.md @.gspec/plan.md Implement the feature following these patterns.
```

Copilot generates tasks, tracks them, and implements them — using the `.gspec/` artifacts as context.

### Plan multiple features (brownfield)

```
> gspec specify wishlist       # Spec one feature
> gspec specify auth-system    # Spec another
```

Features get their own subdirectory under `.gspec/features/`.

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
- **Generates `.github/copilot-instructions.md`** from the discovered patterns — so every future Copilot session (CLI and VS Code) automatically follows your codebase conventions

The output is written as a **cold-reader briefing** — any future AI session can read `context.md` and immediately understand the project.

**Greenfield** — The agent suggests using `/research` for deep domain research, then asks about your project goals, users, and constraints.

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
> @.gspec/context.md @.gspec/plan.md Implement the wishlist feature following these patterns.
```

- Copilot generates tasks, tracks them, and implements them
- Reference `.gspec/` artifacts with `@` mentions for context
- The patterns section in `context.md` ensures code matches the codebase

---

## Copilot-Native Integrations

gspec is built specifically for Copilot CLI, not just running on it:

| Feature | How gspec uses it |
|---------|------------------|
| **`.github/copilot-instructions.md`** | Explore phase generates this from discovered coding patterns — every future Copilot session auto-loads your conventions |
| **`/research`** | Greenfield explore suggests deep domain research via Copilot's multi-source research |
| **`@` file mentions** | Handoff prompts use `@.gspec/context.md` to pass artifacts as context |
| **Plan mode** | Handoff suggests `Shift+Tab` to enter plan mode for task generation |
| **`gh` CLI** | Optionally creates GitHub Issues from the spec for project tracking |
| **Skills system** | User-level skill — auto-discovered in every session, every project |

---

## Skill Contents

```
gspec-skill/
├── SKILL.md                        # Core skill definition (all phase instructions)
├── README.md                       # This file
├── references/
│   └── explore-guide.md            # Detailed codebase exploration patterns
├── examples/
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
| **Phases** | 3 (explore → plan) + Copilot native | 8+ (constitution, clarify, analyze, checklist...) |
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
Copilot CLI already has excellent native task tracking (`/plan` mode) and implementation capabilities. gspec focuses on what Copilot doesn't do natively: persistent codebase understanding, structured requirements, and researched tech stack planning.

**Q: What if I start in one session and continue in another?**
Just say `gspec` or `gspec status`. The agent reads `.gspec/` and picks up where you left off. That's the whole point — artifacts persist.

**Q: Can I use this with other AI agents (Claude Code, Cursor, etc.)?**
The SKILL.md format is Copilot CLI-specific, but the methodology and artifact structure work anywhere. You could adapt the prompts for other agents.

---

## License

MIT
