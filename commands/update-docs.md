---
name: update-docs
description: Create and update project documentation to match current code
---

Create or update project documentation so it accurately reflects the current codebase.

## Documentation philosophy

There are two layers:

1. **`README.md` (root)** — The entry point. Concise. Gets someone from zero to running in minutes.
2. **`docs/` directory** — The deep dives. Every detail, every option, every edge case lives here.

The root README links to docs. It never tries to be comprehensive itself.

## Docs structure

Scale the `docs/` layout to match the project's complexity:

**Small/medium projects** — flat files in `docs/`:

```
docs/
├── architecture.md      # always exists — system overview, modules, data flow
├── <module>.md           # one per module/service
└── ...
```

**Multi-service projects** — subdirectories per service, plus top-level overview docs:

```
docs/
├── architecture.md      # always exists — high-level system overview, how services connect
├── deployment.md        # how to deploy, infrastructure, environments, CI/CD
├── <service>/
│   ├── architecture.md  # service-level internals, data model, design decisions
│   ├── <reference>.md   # commands, endpoints, etc.
│   └── ...
└── ...
```

Rules for choosing the layout:
- **3+ services or independently deployable units** → use subdirectories
- **`docs/architecture.md`** always exists at the top level, even with subdirectories.
- **`docs/deployment.md`** should exist for any project that runs in production.
- **Per-service `architecture.md`** covers that service's internals.
- When migrating from flat to subdirectories, update all README links.

## Style rules

- **Concise over complete.** If it's getting long, move detail to `docs/`.
- **No fluff.** No "Welcome to...", no badges, no "Getting Started" preamble that says nothing.
- **Commands are copy-pasteable.** Every shell block should work if pasted verbatim.
- **Show, don't describe.** A code example beats a paragraph of explanation.
- **No aspirational content.** Only document what the code actually does right now.
- **Plain language.** No marketing speak, no "simply", no "just", no "easily".
- **Preserve existing tone.** When updating, match the style that's already there.

## Workflow

### Step 1: Discover project structure

Before anything else, understand the project:
1. Read the existing `README.md` and any files in `docs/`
2. List top-level directories to identify modules/services
3. Look at `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Makefile`, `docker-compose.yml` etc. to understand the tech stack and entry points

### Step 2: Identify what changed

Run `git diff --name-only HEAD` to see uncommitted changes. If there are no uncommitted changes, compare against the last few commits with `git diff --name-only HEAD~3..HEAD`.

If the user specified a scope (e.g., a module or service name), focus only on that area.

If there are no changes at all, do a full audit — compare all existing docs against the current code.

### Step 3: Map changes to documentation

For each changed source file, determine:
- Does it affect the root README? (e.g., new commands, changed setup steps, new modules)
- Does it affect `docs/architecture.md`? (e.g., new services, changed data model, new storage layer, changed communication patterns)
- Does it affect `docs/deployment.md`? (e.g., new env vars, changed infra, new CI steps)
- Which module-specific doc does it affect?

If a doc file doesn't exist yet but should, create it.

### Step 4: Read the source files and existing docs

For each affected area:
1. Read the changed source files to understand what's new or different
2. Read the existing documentation
3. Compare what the docs say vs what the code actually does

### Step 5: Update the root README.md

The root README must stay concise. It should contain **only** these sections, in this order:

1. **Title + one-liner** — what this project is, one sentence
2. **Setup** — prerequisites, install commands, everything to go from fresh clone to running
3. **Usage** — all the commands to start servers, run the tool, etc.
4. **Project structure** — brief directory overview (2-4 lines, not a tree dump)
5. **Docs** — links to detailed docs in `docs/` with one-line descriptions
6. **License**

If the README is growing beyond ~100-150 lines, something needs to move to `docs/`.

### Step 6: Update detailed docs in `docs/`

**`docs/architecture.md`** — Must always exist. Starts with the big picture, then drills down:
1. System overview — what modules/services exist, how they communicate
2. Data flow — write path and read path
3. Core design — key architectural patterns
4. Entity/data model — entities, relationships
5. Storage — schema, serialization formats, indexes
6. Other invariants — ID generation, hashing, security boundaries

**Module/service docs** — Full reference for each module.

### Step 7: Cross-linking

- README links to every `docs/` file in its Docs section
- Use anchor links for specific sections
- After any rename or restructure, grep for broken links

### Step 8: Summary

After updating, provide a brief summary of what changed in each doc file and why.
