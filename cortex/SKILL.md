---
name: cortex
description: >
  Use this skill when navigating, reading, or exploring the Cortex cognitive file system —
  including shadow-index files, logical ID resolution, schemas, logic fragments, personas,
  SOPs, services, or any .cortex/ artifacts. Trigger proactively whenever the user mentions
  "shadow-index", ".cortex/", logical IDs (colon-separated like "project:schema:table"),
  cortex projects, cortex services, or asks to read, find, look up, or explore anything
  managed by the Cortex CLI framework. Also trigger when the user wants to understand what
  schemas, logic, or services are available in a project.
version: 1.0.0
cortex:
  logical-id: skill:cortex
  depends-on: []
  provides:
    - shadow-index navigation
    - logical ID resolution
    - vault awareness
    - project context orientation
---

# Cortex Navigation Foundation

The Cortex cognitive file system gives every artifact a stable logical ID registered in a shadow-index. This skill is the navigation layer — it teaches the core rule of **resolve before reading** and how to orient within the `.cortex/` structure. Every other Cortex-aware skill builds on these patterns.

## The Core Rule: Always Resolve Before Reading

Never access a `.cortex/` file by guessing its path. The shadow-index is the single source of truth. The workflow is always:

```
1. Determine the logical ID for what you need
2. Look it up in the appropriate shadow-index
3. Get the physical path from the "files" entry
4. Read that path
```

This makes the system refactor-safe. Files can move; their logical IDs never change.

## Quick Navigation Pattern

### Step 1: Orient with MAP.md

```
Read: .cortex/MAP.md
```

MAP.md is a generated human-readable view of everything registered in the shadow-index. Use it when you don't know what exists yet. It is not authoritative (shadow-index.json is), but it's the fastest way to see what's available.

### Step 2: Check active context

```
Read: .cortex/settings.json
Fields: default_project, default_service, operational_mode
```

The `default_project` scopes all project-specific lookups. `operational_mode` tells you what mutations are permitted.

### Step 3: Pick the right index

| What you're looking for | Which index to read |
|:------------------------|:--------------------|
| Personas, SOPs, constitution, system files | `.cortex/shadow-index.json` (global) |
| Schemas, logic, features, services, ingested code | `.cortex/projects/<project>/shadow-index.json` (project-scoped) |
| Cross-repo references | `remotes` field in global index → path to remote `.cortex/` |

### Step 4: Resolve the logical ID

Open the relevant shadow-index and find your ID under `"files"`:

```json
{
  "files": {
    "payment-system:checkout-api:logic:fraud-detection": ".cortex/projects/payment-system/services/checkout-api/logic/fraud-detection.md"
  }
}
```

Or resolve via CLI:
```bash
./bin/cortex resolve <logical-id>
```

### Step 5: Read the artifact

Read the resolved path. Every entry in the shadow-index points to a file that physically exists — the index guarantees it.

## Logical ID Patterns at a Glance

| Pattern | Example | What it is |
|:--------|:--------|:-----------|
| `core:{type}:{name}` | `core:persona:developer` | System artifact (persona, SOP, constitution) |
| `root:{type}:{name}` | `root:general:settings` | Root-level file (MAP.md, settings.json) |
| `{project}:{service}:{type}:{entity}` | `payment-system:checkout-api:logic:fraud-detection` | Service-level artifact |
| `{project}:{db}:schema:{schema}:{type}:{name}` | `payment-system:payment-db:schema:public:table:users` | Database schema artifact |
| `{project}:{service}:collected:{name}` | `auth:api:collected:openapi-spec` | Imported external doc |
| `{project}:{service}:feature:{name}` | `auth:api:feature:oauth2-login` | Feature specification |
| `skill:{skill-name}` | `skill:cortex` | Claude skill definition |

Reading a logical ID left-to-right reveals scope → domain → type → entity. You can often predict an ID before confirming it in the index.

## Vault Stability Rules

Before any write, check which vault you're targeting:

| Vault | Path | Allowed Operations |
|:------|:-----|:-------------------|
| IMMUTABLE | `.cortex/core/`, `bin/` | Read only. Never write. |
| PROTECTED | `.cortex/projects/`, `.cortex/logic/`, `.cortex/memory/` | Write via CLI commands + SOPs only |
| APPEND-ONLY | `.cortex/history/` | Append via `./bin/cortex log` only. Never delete. |
| USER-OWNED | `.cortex/custom/` | Full access in Developer Mode |
| VOLATILE | `.cortex/output/` | Freely read and regenerated |

Read operations are always safe in any vault.

## Operational Modes

Read `operational_mode` from `.cortex/settings.json`:

- **user** — Read-only across all `.cortex/`. Observe and report, no mutations.
- **developer** — Full access within constitutional limits (SOPs must be followed for PROTECTED vaults).

When in User Mode, you can still read any artifact and run read-only CLI commands.

## Bootstrap Pattern for Any New Session

When you start working in a Cortex project, orient before acting:

```
1. Read: .cortex/settings.json             → active project + mode
2. Read: .cortex/MAP.md                    → what exists
3. Read: .cortex/shadow-index.json         → resolve global IDs
4. Read: .cortex/projects/<project>/shadow-index.json  → resolve project IDs
5. Read: .cortex/core/constitution.md      → governance rules
```

Or run `./bin/cortex bootstrap` (or `./bin/cortex ai`) to get all of this in one shot as structured output.

## Key CLI Commands

```bash
./bin/cortex bootstrap              # Full session context (constitution + project + summaries)
./bin/cortex ai                     # Same as bootstrap
./bin/cortex resolve <id>           # Logical ID → physical path
./bin/cortex validate               # Verify shadow-index integrity (run after any mutation)
./bin/cortex generate-map           # Regenerate MAP.md from shadow-index
./bin/cortex index                  # Rebuild shadow-index.md navigation files
./bin/cortex project current        # Show active project
./bin/cortex project list           # All registered projects
./bin/cortex project use <name>     # Switch active project
./bin/cortex service use <name>     # Switch active service
./bin/cortex persona <name>         # Load persona context (developer, architect, dba, analyst, pm)
```

## For Skills Building on This Foundation

If you're a domain skill (schema-reader, logic-navigator, feature-finder, etc.) that builds on this foundation:

1. Resolve logical IDs before any file read — never hardcode paths
2. Read `settings.json` for active project before project-scoped operations
3. Declare dependency in your SKILL.md frontmatter:
   ```yaml
   cortex:
     logical-id: skill:<your-skill>
     depends-on:
       - skill:cortex
   ```
4. Include a `## Cross-Skill References` section pointing to this skill
5. For the full skill registry and cross-reference map, load:
   `skill:cortex:ref:ecosystem` → `.cortex/skills/cortex/references/skills-ecosystem.md`

## Cross-Skill References

This is the foundation skill. It has no dependencies. All other Cortex-aware skills depend on it.

## Related Cortex Artifacts

| What | Logical ID | When to Read |
|:-----|:-----------|:-------------|
| Supreme governance rules | `core:general:constitution` | Before any mutation; when in doubt about what's allowed |
| AI agent operating guide | `core:general:AI_AGENT_GUIDE` | For full agent operating protocol |
| Human-readable index | `root:general:map` | For orientation when you don't know what exists |
| Active settings | `root:general:settings` | Always — defines project scope and operational mode |
| Skills ecosystem map | `skill:cortex:ref:ecosystem` | When building on this skill or discovering other skills |
