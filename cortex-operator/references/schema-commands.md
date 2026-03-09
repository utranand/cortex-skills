# Schema Extraction & Database Commands

> Detailed reference for extracting database schemas from SQL files and managing database directories.
> Schema data is stored under a database-level directory within the project, enabling multi-database isolation.

## extract

**Syntax:**
```bash
./bin/cortex extract <sql_file> [--project <name>] [--database <name>]
```

Parses SQL dump files (CREATE TABLE statements) and generates schema documentation for each table. Stores results under `.cortex/projects/<project>/<database>/schema/`.

**Prerequisites:** Active project must be set (or use `--project` flag).

**Flags:**
- `--database <name>` — Names the logical database directory. If omitted: prompts interactively in TTY mode, or generates a timestamped name in non-interactive/CI mode.
- `--project <name>` — Target project. Defaults to active project.

**Output Structure:**
```
.cortex/projects/<project>/
├── shadow-index.json               # Project-scoped index (schema entries added)
├── shadow-index.md                 # Project index -> lists databases
└── <database>/                     # Database-level directory
    ├── shadow-index.md             # Database index -> links to schema
    └── schema/
        ├── shadow-index.md         # Schema index -> lists tables/views/functions
        └── public/
            └── tables/
                ├── users.md
                ├── transactions.md
                └── payment_methods.md
```

**Logical ID format:** `<project>:<database>:schema:<schema>:<type>:<name>`

**Examples:**

```bash
# Extract with interactive database name prompt
./bin/cortex project use payment-system
./bin/cortex extract database-dump.sql

# Extract with explicit database name
./bin/cortex extract database-dump.sql --database payment-db

# Extract with full explicit context
./bin/cortex extract schema.sql --project auth-service --database auth-db

# Multi-database project
./bin/cortex extract users-schema.sql    --project saas-platform --database users-db
./bin/cortex extract products-schema.sql --project saas-platform --database catalog-db
```

**Post-extraction checklist:**
```bash
./bin/cortex validate       # verify index integrity
./bin/cortex index          # regenerate .md index files
./bin/cortex generate-map   # update MAP.md
```

---

## database list

**Syntax:**
```bash
./bin/cortex database list [--project <name>]
```

Lists all database directories in the active project. A directory is recognized as a database if it contains a `schema/` subdirectory.

**Example output:**
```
Databases in project "payment-system":
  - auth-db
  - prod-payment-db
  - reporting-db
```

---

## database rename

**Syntax:**
```bash
./bin/cortex database rename <old_name> <new_name> [--project <name>]
```

Renames a database folder and atomically updates all logical IDs and file path mappings in the project-scoped shadow-index. Prompts for confirmation in interactive mode.

**Safety features:**
- Pre-validates `shadow-index.json` is parseable before touching filesystem
- Creates timestamped backup of `shadow-index.json` before modification
- Validates old name exists and new name is not already in use

**What it updates:**
- Logical IDs: `<project>:<old>:schema:*` -> `<project>:<new>:schema:*`
- File paths: `/<project>/<old>/` -> `/<project>/<new>/`
- Regenerates all hierarchical shadow-index.md files

**Example:**
```bash
./bin/cortex project use payment-system
./bin/cortex database rename payment-db prod-payment-db
```

---

## Common Workflows

### Extract Schema for New Project
```bash
./bin/cortex project create my-project
./bin/cortex project use my-project
./bin/cortex extract dump.sql --database main-db
./bin/cortex validate
./bin/cortex generate-map
```

### Multi-Database Extraction
```bash
./bin/cortex project use saas-platform
./bin/cortex extract users.sql --database users-db
./bin/cortex extract catalog.sql --database catalog-db
./bin/cortex extract orders.sql --database orders-db
./bin/cortex validate
```

### Cleanup Legacy Schemas
```bash
./bin/cortex doctor --cleanup-schemas    # migrate global -> project-scoped
./bin/cortex validate
```
