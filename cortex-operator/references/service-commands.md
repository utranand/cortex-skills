# Service Management Commands

> Detailed reference for creating and managing services within projects.
> Services are sub-units representing microservices, API versions, or feature areas.
> Each service has its own namespace for logic, features, and ingested content.

## new service

**Syntax:**
```bash
./bin/cortex new service <name>
```

Create a new service in the active project.

**Prerequisites:** Active project must be set.

**What it creates:**
```
.cortex/projects/<project>/<service>/
├── logic/          # Business logic artifacts
├── features/       # Feature specifications
├── ingested/       # Reverse-engineered code
└── README.md       # Service documentation
```

**Example:**
```bash
./bin/cortex project use payment-system
./bin/cortex new service checkout-api
./bin/cortex new service admin-api
./bin/cortex new service reporting-api
```

---

## service use

**Syntax:**
```bash
./bin/cortex service use <name>
```

Switch the active service context within the current project.

**Example:**
```bash
./bin/cortex service use admin-api

./bin/cortex service current
# Active Context:
#   Project: payment-system
#   Service: admin-api
#   Path: .cortex/projects/payment-system/admin-api/
```

---

## service list

**Syntax:**
```bash
./bin/cortex service list
```

List all services in the active project.

**Prerequisites:** Active project must be set.

**Example output:**
```
Available Services in 'payment-system':
  * checkout-api     <- Active
    admin-api
    reporting-api
```

---

## service current

**Syntax:**
```bash
./bin/cortex service current
```

Display the currently active project and service context.

**Example output:**
```
Active Context:
  Project: payment-system
  Service: checkout-api
  Path: .cortex/projects/payment-system/checkout-api/
```

---

## service clear

**Syntax:**
```bash
./bin/cortex service clear
```

Clear the active service selection. Project context remains.

**Example:**
```bash
./bin/cortex service clear
# Active service cleared

./bin/cortex service current
# Active Context:
#   Project: payment-system
#   Service: none
```

---

## project remove-service

**Syntax:**
```bash
./bin/cortex project remove-service <name> [--project <project>]
```

**DESTRUCTIVE** — Remove a service and all its data (logic, features, ingested content).

- Deletes service directory
- Removes all service-related index entries
- Auto-updates `default_service` to next available service

**Always confirm with the user before running this command.**

---

## Common Workflows

### Set Up Services for a Project
```bash
./bin/cortex project use my-project
./bin/cortex new service api-gateway
./bin/cortex new service auth-service
./bin/cortex new service data-pipeline
./bin/cortex service use api-gateway     # set initial active service
```

### Switch Service Context
```bash
./bin/cortex service list                # see what's available
./bin/cortex service use <name>          # switch
./bin/cortex service current             # verify
```

### Ingest Code Into a Service
```bash
./bin/cortex project use my-project
./bin/cortex service use api-gateway
./bin/cortex ingest ./src/api --reverse-engineer
./bin/cortex validate
```
