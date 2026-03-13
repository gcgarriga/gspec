# Codebase Exploration Guide

Reference patterns for Phase 1 (Explore) of the gspec workflow. The goal is to **understand** the codebase — not just identify what files exist.

## Exploration Strategy

**Always follow this order:**

### 1. Read Documentation First (fastest signal)

```bash
# Read README — the single best source of project understanding
cat README.md 2>/dev/null

# Check for other docs
ls docs/ 2>/dev/null
ls ADR/ decisions/ 2>/dev/null
cat CONTRIBUTING.md 2>/dev/null | head -60
cat ARCHITECTURE.md 2>/dev/null
```

### 2. Understand the Shape (structure + scale)

```bash
# Directory structure — 2 levels, exclude noise
find . -maxdepth 2 -type d -not -path './.git/*' -not -path './node_modules/*' -not -path './.next/*' -not -path './target/*' -not -path './dist/*' -not -path './build/*' -not -path './.gspec/*'

# Scale — how big is this project?
find . -type f -not -path './.git/*' -not -path './node_modules/*' -not -path './target/*' | wc -l

# File types — what languages are in play?
find . -type f -not -path './.git/*' -not -path './node_modules/*' -not -path './target/*' | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -15

# Recent activity — what's being worked on?
git log --oneline -15 2>/dev/null
```

### 3. Read the Entry Point (understand the bootstrap)

Find and read the main entry point. This reveals how the application starts, what it initializes, and how requests flow in.

Common entry points by ecosystem:
- **Node/TS:** `src/index.ts`, `src/app.ts`, `src/server.ts`, `src/main.ts`, or `scripts.start` in `package.json`
- **Python:** `main.py`, `app.py`, `src/__main__.py`, `manage.py` (Django), `wsgi.py`
- **Go:** `cmd/*/main.go`, `main.go`
- **Rust:** `src/main.rs`, `src/lib.rs`
- **Java:** `src/main/java/**/Application.java`, class with `@SpringBootApplication`
- **.NET:** `Program.cs`, `Startup.cs`

**Read the entry point file completely.** Understand what middleware, routes, services, or modules are registered.

### 4. Trace One Request End-to-End

Pick a simple endpoint or feature and trace its path through the codebase:
1. **Route/Handler** — how is the request received?
2. **Validation** — how is input validated?
3. **Business logic** — where does the core logic live?
4. **Data access** — how does it talk to the database or external services?
5. **Response** — how is the response constructed?

This reveals the actual architecture better than any folder name.

### 5. Read Tests for Domain Understanding

Tests are often the best documentation of expected behavior:

```bash
# Find test files
find . -type f -name '*test*' -o -name '*spec*' | head -20

# Read 2-3 test files to understand:
# - What the domain concepts are
# - How the code is meant to be used
# - What edge cases matter
```

### 6. Understand the Data Model

```bash
# Database schemas / migrations
find . -type d -name 'migrations' -o -name 'migrate' 2>/dev/null
find . -name 'schema.prisma' -o -name 'schema.sql' -o -name '*.entity.ts' 2>/dev/null

# ORM models
find . -name 'models' -type d 2>/dev/null
find . -name '*.model.*' -o -name '*.entity.*' 2>/dev/null | head -10
```

Read the data model files. The entities and their relationships define the domain.

## Language & Framework Detection

Use these as **starting signals**, not conclusions. Always verify by reading the actual code.

| File | Indicates |
|------|-----------|
| `package.json` | Node.js / JavaScript / TypeScript |
| `tsconfig.json` | TypeScript |
| `Cargo.toml` | Rust |
| `go.mod` | Go |
| `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile` | Python |
| `Gemfile` | Ruby |
| `pom.xml`, `build.gradle` | Java |
| `*.csproj`, `*.sln` | .NET / C# |
| `composer.json` | PHP |
| `mix.exs` | Elixir |
| `Package.swift` | Swift |
| `pubspec.yaml` | Dart / Flutter |

### Framework Detection (within ecosystems)

**JavaScript/TypeScript:**
- `next.config.*` → Next.js
- `vite.config.*` → Vite
- `angular.json` → Angular
- `svelte.config.*` → SvelteKit
- `nuxt.config.*` → Nuxt
- `astro.config.*` → Astro
- Check `package.json` dependencies for the actual libraries

**Python:**
- `manage.py` → Django
- FastAPI/Flask — check imports in `main.py` or `app.py`

**Check dependency files** for the actual libraries in use. Read the dependency list — it tells you what problems the project has already solved (auth, validation, logging, etc.).

## Architecture Patterns

**Monolith indicators:**
- Single project root with `src/` or `app/`
- One build configuration, one deploy target
- Shared database models across features

**Microservices indicators:**
- Multiple service directories (`services/`, `apps/`)
- Docker Compose with multiple services
- Multiple independent dependency files
- API gateway or service mesh configuration

**Monorepo indicators:**
- `turbo.json`, `nx.json`, `lerna.json`, `pnpm-workspace.yaml`
- `packages/` or `apps/` with independent package files
- Workspace configuration in root package file

## What to Assess (Beyond Detection)

When exploring, actively form opinions about:

- **Code quality** — Is it consistent? Well-structured? Or spaghetti?
- **Test coverage** — Are there tests? Are they meaningful or just smoke tests?
- **Documentation** — Is the README useful? Are there inline comments where needed?
- **Technical debt** — Are there TODOs? Deprecated patterns? Copy-paste duplication?
- **Complexity hotspots** — Which files/modules are unusually large or complex?
- **Security posture** — How is auth handled? Are secrets in env vars or hardcoded?

These assessments go into the "Strengths and debt" section of `context.md`.
