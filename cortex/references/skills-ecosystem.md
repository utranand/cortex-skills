# Cortex Skills Ecosystem

> Quick reference for all Claude skills registered in this Cortex instance and their relationships.
> Load this file when you need to discover available skills or declare cross-skill dependencies.

## Current Skills Registry

| Logical ID | Path | Version | Provides |
|:-----------|:-----|:--------|:---------|
| `skill:cortex` | `.cortex/skills/cortex/SKILL.md` | 1.0.0 | Shadow-index navigation, logical ID resolution, vault awareness, project context orientation |

## Dependency Graph

```
┌─────────────────────────────────────────────────┐
│             Domain Skills (planned)             │
│  schema-reader  logic-navigator  feature-finder │  ← Built on skill:cortex
└──────────────────────┬──────────────────────────┘
                       │ depends-on
┌──────────────────────▼──────────────────────────┐
│           skill:cortex  (v1.0.0)                │  ← Foundation: all Cortex skills build here
│  Shadow-index nav · Logical ID resolution       │
│  Vault awareness · Project context orientation  │
└──────────────────────┬──────────────────────────┘
                       │ reads
┌──────────────────────▼──────────────────────────┐
│              Cortex File System                 │
│  .cortex/shadow-index.json                      │
│  .cortex/projects/*/shadow-index.json           │
│  .cortex/core/ · .cortex/projects/              │
└─────────────────────────────────────────────────┘
```

## Planned Domain Skills

These skills are documented in the architecture but not yet created:

| Logical ID | Builds On | Will Provide |
|:-----------|:----------|:-------------|
| `skill:schema-reader` | `skill:cortex` | Read and interpret database schema artifacts |
| `skill:logic-navigator` | `skill:cortex` | Read and reason over business logic fragments |
| `skill:feature-finder` | `skill:cortex` | Locate and read feature blueprint specifications |
| `skill:persona-loader` | `skill:cortex` | Load persona context for the active role |
| `skill:federation-walker` | `skill:cortex` | Traverse bridge-linked Cortex instances |

## Cross-Skill Reference Protocol

When your skill depends on `skill:cortex`, declare it in frontmatter:

```yaml
cortex:
  logical-id: skill:<your-skill>
  depends-on:
    - skill:cortex
  provides:
    - <what your skill enables>
```

And in the body of your SKILL.md:

```markdown
## Cross-Skill References

This skill builds on:
- `skill:cortex` — Shadow-index navigation and logical ID resolution
  Path: `.cortex/skills/cortex/SKILL.md`
```

Always reference other skills by logical ID, not path. Resolve via shadow-index when you need the physical path.

## Adding a New Skill

1. Create the skill directory and files:
   ```
   .cortex/skills/<skill-name>/
   ├── SKILL.md
   └── references/
       └── skills-ecosystem.md  (symlink or copy, always include)
   ```

2. Register in `.cortex/shadow-index.json`:
   ```json
   "skill:<name>": ".cortex/skills/<name>/SKILL.md",
   "skill:<name>:ref:ecosystem": ".cortex/skills/<name>/references/skills-ecosystem.md"
   ```

3. Validate: `./bin/cortex validate`

4. Add a row to the **Current Skills Registry** table above.

5. Update the **Dependency Graph** if the skill introduces a new tier.
