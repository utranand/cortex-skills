---
name: cortex-operator
description: >
  Use this skill to execute Cortex CLI commands for managing projects, services, schemas,
  databases, and bridge federation. Trigger proactively when the user wants to create, switch,
  list, or remove projects or services, extract database schemas from SQL files, manage bridge
  links between Cortex instances, or run any `./bin/cortex` operation. Also trigger when the
  user mentions cortex commands, project setup, schema extraction, service creation, bridge
  federation, or asks to "set up", "initialize", "extract", "ingest", or "link" anything
  in the Cortex ecosystem — even if they don't explicitly say "cortex".
version: 1.0.0
cortex:
  logical-id: skill:cortex-operator
  depends-on:
    - skill:cortex
  provides:
    - cortex CLI command execution
    - project lifecycle management
    - service lifecycle management
    - schema extraction orchestration
    - bridge federation management
---

# Cortex Operator

This skill executes Cortex CLI commands on behalf of the user. Where `skill:cortex` teaches
navigation (resolve before reading), this skill handles **mutations** — creating projects,
extracting schemas, linking bridges, managing services. It knows the correct command syntax,
required prerequisites, argument patterns, and safety constraints for each operation.

## Before Every Operation

Establish context first. Many commands depend on an active project or service.

```bash
# Always check current context before running project/service-scoped commands
./bin/cortex project current
./bin/cortex service current
```

Read `.cortex/settings.json` for `default_project` and `default_service` if you need
to verify context without running CLI commands.

## Command Decision Guide

Match the user's intent to the right command category, then load the relevant reference
file for detailed syntax and examples.

| User Intent | Category | Reference File |
|:------------|:---------|:---------------|
| Create, switch, list, or remove a project | Project | `references/project-commands.md` |
| Create, switch, list, or clear a service | Service | `references/service-commands.md` |
| Extract schema from SQL, manage databases | Schema | `references/schema-commands.md` |
| Link repos, validate bridges, set relationships | Bridge | `references/bridge-commands.md` |

When you identify the category, read the corresponding reference file for the full
command syntax, flags, prerequisites, and examples. The reference files are the
authoritative source for command details.

## Execution Rules

### 1. Context Before Action

Most commands are project-scoped. Before running any project-scoped command, verify
the active project is correct:

```bash
./bin/cortex project current
```

If the user hasn't specified a project and none is active, ask which project to use
before proceeding.

### 2. Destructive Commands Need Confirmation

These commands are irreversible — always confirm with the user before executing:

| Command | Why It's Destructive |
|:--------|:--------------------|
| `project remove <name>` | Deletes all project data permanently |
| `project remove-schema <name>` | Deletes schema artifacts |
| `project remove-service <name>` | Deletes service and all its data |
| `bridge remove <alias>` | Removes federation link |
| `doctor --rebuild` | Reconstructs entire shadow-index from scratch |
| `doctor --prune` | Removes entries with missing files |

For these commands, state what will happen and ask for explicit confirmation before executing.

### 3. Multi-Step Operations

Some tasks require a sequence of commands. Execute them in order and verify each step:

**Setting up a new project with services:**
```bash
./bin/cortex project create <name>
./bin/cortex project use <name>
./bin/cortex new service <service-name>
./bin/cortex service use <service-name>
```

**Extracting schema into a project:**
```bash
./bin/cortex project use <project>          # ensure correct context
./bin/cortex extract <sql-file> --database <db-name>
./bin/cortex validate                        # verify index integrity after mutation
```

**Linking and configuring a bridge:**
```bash
./bin/cortex bridge add <alias> <path>
./bin/cortex bridge rel <alias> <relationship-type>
./bin/cortex bridge validate
```

### 4. Always Validate After Mutations

After any command that modifies the shadow-index (create, extract, remove, rename),
run validation:

```bash
./bin/cortex validate
```

If validation fails, run diagnostics:

```bash
./bin/cortex doctor
```

### 5. Regenerate Indexes When Needed

After structural changes (new projects, services, schema extractions), regenerate
the human-readable indexes:

```bash
./bin/cortex index                    # rebuild shadow-index.md files
./bin/cortex generate-map             # rebuild MAP.md
```

## Quick Command Reference

### Project Commands (most common)

```bash
./bin/cortex project create <name>           # scaffold new project
./bin/cortex project use <name>              # switch active project
./bin/cortex project list                    # list all projects
./bin/cortex project current                 # show active project context
./bin/cortex project remove <name>           # DELETE project (destructive!)
```

### Service Commands

```bash
./bin/cortex new service <name>              # create service in active project
./bin/cortex service use <name>              # switch active service
./bin/cortex service list                    # list services in active project
./bin/cortex service current                 # show active service context
./bin/cortex service clear                   # clear active service selection
```

### Schema Commands

```bash
./bin/cortex extract <sql-file>                          # extract (prompts for db name)
./bin/cortex extract <sql-file> --database <db-name>     # extract with explicit db
./bin/cortex extract <sql-file> --project <p> --database <db>  # full explicit
./bin/cortex database list                               # list databases in project
./bin/cortex database rename <old> <new>                 # rename database
```

### Bridge Commands

```bash
./bin/cortex bridge add <alias> <path>       # link external .cortex
./bin/cortex bridge add <path>               # link with auto-alias
./bin/cortex bridge list                     # list all bridges
./bin/cortex bridge show <alias>             # show bridge details
./bin/cortex bridge update <alias> <path>    # update bridge path
./bin/cortex bridge remove <alias>           # remove bridge (destructive!)
./bin/cortex bridge validate                 # validate all bridges
./bin/cortex bridge rel <alias> <type>       # set relationship type
```

### Maintenance (use when things go wrong)

```bash
./bin/cortex validate                        # verify shadow-index integrity
./bin/cortex doctor                          # diagnose issues
./bin/cortex doctor --rebuild                # rebuild index from scratch (destructive!)
./bin/cortex doctor --prune                  # remove stale entries (destructive!)
./bin/cortex index                           # regenerate .md index files
./bin/cortex generate-map                    # regenerate MAP.md
```

## Error Recovery

If a command fails:

1. **"No active project"** → Run `./bin/cortex project use <name>` first
2. **"Project not found"** → Run `./bin/cortex project list` to see available projects
3. **"Service not found"** → Run `./bin/cortex service list` to see available services
4. **"Shadow index invalid"** → Run `./bin/cortex doctor` to diagnose, then `./bin/cortex doctor --rebuild` if needed
5. **"Bridge path not found"** → Run `./bin/cortex bridge validate` to check all bridges

## Cross-Skill References

This skill builds on:
- `skill:cortex` — Shadow-index navigation and logical ID resolution
  Path: `.cortex/skills/cortex/SKILL.md`
  Load when: you need to resolve a logical ID or navigate the shadow-index before executing a command

## Related Cortex Artifacts

| What | Logical ID | When to Read |
|:-----|:-----------|:-------------|
| Foundation navigation | `skill:cortex` | Before any shadow-index lookup |
| Skills ecosystem map | `skill:cortex:ref:ecosystem` | When discovering available skills |
| Settings & active context | `root:general:settings` | Before project/service-scoped commands |
| System constitution | `core:general:constitution` | When unsure about governance rules |
| Full capabilities reference | — | `docs/CORTEX_CAPABILITIES.md` for commands not yet covered here |
