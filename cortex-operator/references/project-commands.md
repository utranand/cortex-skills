# Project Management Commands

> Detailed reference for all project lifecycle commands.
> Projects are top-level organizational units — each maintains its own namespace, schemas, services, and shadow-index.

## project create

**Syntax:**
```bash
./bin/cortex project create <name>
```

Scaffolds a new project workspace with directory structure and project-scoped index.

**What it creates:**
```
.cortex/projects/<name>/
├── README.md
├── shadow-index.json         # Project-scoped index
└── shadow-index.md           # Human-readable index
```

**Example:**
```bash
./bin/cortex project create e-commerce
./bin/cortex project create auth-service
```

**Follow-up:** After creating, switch to it with `project use` and optionally create services.

---

## project use

**Syntax:**
```bash
./bin/cortex project use <name>
```

Switch the active project context. All subsequent scoped operations (extract, ingest, service creation) use this project.

**Important:** Switching projects does NOT change the active service. Services are project-specific, so you may need to set a new service context afterward.

**Example:**
```bash
./bin/cortex project use payment-system
./bin/cortex project current    # verify switch
```

---

## project list

**Syntax:**
```bash
./bin/cortex project list
```

Display all projects in `.cortex/projects/` with the active project marked.

**Example output:**
```
Available Projects:
  * payment-system     <- Active
    auth-service
    analytics-platform
```

---

## project current

**Syntax:**
```bash
./bin/cortex project current
```

Show the currently active project and path.

**Example output:**
```
Active Context:
  Project: payment-system
  Path: .cortex/projects/payment-system/
```

---

## project remove

**Syntax:**
```bash
./bin/cortex project remove <name>
```

**DESTRUCTIVE** — Completely removes a project and all its data (services, schemas, logic, ingested content). Cannot be undone.

**Behavior:**
- Deletes project directory: `.cortex/projects/<name>/`
- Removes project-scoped index entries
- Auto-updates `default_project` to next available project
- Clears `default_service` (services are project-specific)

**Always confirm with the user before running this command.**

---

## project remove-schema

**Syntax:**
```bash
./bin/cortex project remove-schema <name> [--project <project>]
```

Remove a specific schema and its index entries from a project.

**Example:**
```bash
./bin/cortex project remove-schema old_users_table
./bin/cortex project remove-schema legacy_schema --project auth-service
```

---

## project remove-service

**Syntax:**
```bash
./bin/cortex project remove-service <name> [--project <project>]
```

**DESTRUCTIVE** — Remove a service and all its index entries from a project.

- Deletes service directory
- Removes all service-related index entries
- Auto-updates `default_service` to next available service

---

## project reverse-engineer

**Syntax:**
```bash
./bin/cortex project reverse-engineer [--path <directory>]
```

Extract business logic patterns from source code into structured Decision Matrix fragments. Requires active project AND service context.

**Prerequisites:**
```bash
./bin/cortex project use <project>
./bin/cortex service use <service>
```

**Example:**
```bash
./bin/cortex project reverse-engineer
./bin/cortex project reverse-engineer --path ./src/services/payment
```

**Output:** Logic fragments stored in `.cortex/projects/<project>/<service>/logic/`

---

## Common Workflows

### New Project Setup
```bash
./bin/cortex project create my-project
./bin/cortex project use my-project
./bin/cortex new service api-v1
./bin/cortex service use api-v1
```

### Verify Project State
```bash
./bin/cortex project current
./bin/cortex service list
./bin/cortex validate
```
