# Bridge Federation Commands

> Detailed reference for linking multiple Cortex instances together.
> Bridge federation enables cross-project visibility for microservices architectures,
> shared libraries, and multi-repository environments.

## bridge add

**Syntax:**
```bash
./bin/cortex bridge add <alias> <path>
# OR (auto-aliasing — derives alias from path)
./bin/cortex bridge add <path>
```

Link a remote `.cortex` directory as a named alias for federated access.

**Examples:**
```bash
# Explicit alias
./bin/cortex bridge add shared-lib ~/projects/common-utils/.cortex

# Auto-alias (derives "auth-service" from path)
./bin/cortex bridge add ~/projects/auth-service/.cortex

# Multiple bridges
./bin/cortex bridge add data-pipeline ~/services/etl-pipeline/.cortex
./bin/cortex bridge add notification-service ~/services/notifications/.cortex
```

**Stored in:** `.cortex/bridges.json`

---

## bridge list

**Syntax:**
```bash
./bin/cortex bridge list
```

Display all configured federated bridges.

**Example output:**
```
Federated Bridges:
  - shared-lib -> ~/projects/common-utils/.cortex
  - auth-service -> ~/services/auth/.cortex (microservice)
  - data-pipeline -> ~/services/etl/.cortex (data-source)
```

---

## bridge show

**Syntax:**
```bash
./bin/cortex bridge show <alias>
```

Display detailed information about a specific bridge.

**Example output:**
```
Bridge: shared-lib
Path: ~/projects/common-utils/.cortex
Status: OK Valid
Relationship: consumes-api
```

---

## bridge update

**Syntax:**
```bash
./bin/cortex bridge update <alias> <new_path>
```

Update the path for an existing bridge (e.g., when the linked project moves).

**Example:**
```bash
./bin/cortex bridge update shared-lib ~/new-location/common-utils/.cortex
```

---

## bridge remove

**Syntax:**
```bash
./bin/cortex bridge remove <alias>
```

**DESTRUCTIVE** — Remove a federated bridge link. Confirm with user first.

---

## bridge validate

**Syntax:**
```bash
./bin/cortex bridge validate
```

Validate all configured bridges for broken links or invalid paths. Run this regularly and after any bridge modifications.

**Example output:**
```
Bridge Validation:
  OK shared-lib -> ~/projects/common-utils/.cortex
  FAIL old-service -> ~/services/deprecated/.cortex
    Issue: Path not found
  OK auth-service -> ~/services/auth/.cortex
```

---

## bridge rel

**Syntax:**
```bash
./bin/cortex bridge rel <alias> <type>
```

Define the relationship type between your project and the linked bridge.

**Relationship Types:**

| Type | Meaning |
|:-----|:--------|
| `consumes-api` | Uses API from linked project |
| `provides-api` | Exposes API to linked project |
| `shared-lib` | Shared code library |
| `data-source` | Data provider |
| `depends-on` | Generic dependency |

**Examples:**
```bash
./bin/cortex bridge rel auth-service consumes-api
./bin/cortex bridge rel common-utils shared-lib
./bin/cortex bridge rel analytics-db data-source
```

---

## Common Workflows

### Microservices Architecture Setup
```bash
cd ~/projects/main-app
./bin/cortex bridge add auth-service ~/services/auth/.cortex
./bin/cortex bridge rel auth-service consumes-api

./bin/cortex bridge add payment-service ~/services/payment/.cortex
./bin/cortex bridge rel payment-service consumes-api

./bin/cortex bridge validate
./bin/cortex generate-map --federated
```

### Shared Library Integration
```bash
./bin/cortex bridge add ui-components ~/libs/react-components/.cortex
./bin/cortex bridge rel ui-components shared-lib

./bin/cortex bridge add api-client ~/libs/api-client/.cortex
./bin/cortex bridge rel api-client shared-lib

./bin/cortex bridge validate
```

### Fix Broken Bridge
```bash
./bin/cortex bridge validate              # identify broken bridges
./bin/cortex bridge show <alias>          # inspect details
./bin/cortex bridge update <alias> <new-path>  # fix path
./bin/cortex bridge validate              # verify fix
```
