# GID Graph Building Guide (For AI)

This guide helps AI assistants understand how to build GID (Graph-Indexed Development) dependency graphs for software projects.

---

## What is a GID Graph?

A GID graph represents the dependency relationships in a software project. It consists of:
- **Nodes**: Components, features, files, or other entities
- **Edges**: Relationships between nodes (dependencies, implementations, etc.)

The graph is stored in `.gid/graph.yml` in YAML format.

---

## Node Types

| Type | Description | When to Use |
|------|-------------|-------------|
| `Feature` | User-perceivable functionality | Business requirements, user stories |
| `Component` | Technical module/service/class | Services, controllers, utilities |
| `Interface` | API endpoint or contract | REST endpoints, GraphQL queries |
| `Data` | Data model or schema | Database tables, DTOs |
| `File` | Source code file | Auto-extracted, or manual for key files |
| `Test` | Test case or suite | Test files, test scenarios |
| `Decision` | Architecture decision | ADRs, technical choices |

### Node Properties

```yaml
nodes:
  UserService:
    type: Component
    description: "Handles user CRUD operations"  # Required: clear description
    layer: application                            # Optional: architecture layer
    path: src/services/user.ts                   # Optional: file path
    status: active                               # Optional: active/deprecated/planned
    priority: core                               # Optional: core/important/nice-to-have
```

---

## Edge Relations

| Relation | Meaning | Example |
|----------|---------|---------|
| `implements` | Component realizes a Feature | UserService → UserRegistration |
| `depends_on` | Component uses another Component | UserService → Database |
| `calls` | Component invokes an Interface | Frontend → UserAPI |
| `reads` | Component reads from Data | ReportService → UserTable |
| `writes` | Component writes to Data | UserService → UserTable |
| `tested_by` | Component is tested by Test | UserService → UserServiceTest |
| `extends` | Component extends another | AdminService → UserService |
| `composes` | Component contains another | OrderService → PaymentHandler |

### Edge Properties

```yaml
edges:
  - from: UserService
    to: Database
    relation: depends_on
    description: "Queries user data"  # Optional: why this dependency exists
```

---

## Architecture Layers

Components can belong to architecture layers. Dependencies should flow in one direction:

```
interface → application → domain → infrastructure
    ↓            ↓           ↓           ↓
 (API)      (Services)   (Models)    (Database)
```

**Valid**: `interface → application` (Controller calls Service)
**Invalid**: `infrastructure → application` (Database calls Service)

```yaml
nodes:
  UserController:
    type: Component
    layer: interface

  UserService:
    type: Component
    layer: application

  User:
    type: Data
    layer: domain

  PostgresDB:
    type: Component
    layer: infrastructure
```

---

## Building a Graph: Step by Step

### Step 1: Identify Features

Start with user-facing functionality:

```yaml
nodes:
  UserRegistration:
    type: Feature
    description: "User can create a new account"
    priority: core

  UserLogin:
    type: Feature
    description: "User can authenticate with email/password"
    priority: core

  PasswordReset:
    type: Feature
    description: "User can reset forgotten password"
    priority: important
```

### Step 2: Identify Components

Map out the technical modules:

```yaml
nodes:
  # ... features above ...

  AuthService:
    type: Component
    description: "Handles authentication logic"
    layer: application
    path: src/services/auth.ts

  UserService:
    type: Component
    description: "Manages user data"
    layer: application
    path: src/services/user.ts

  EmailService:
    type: Component
    description: "Sends transactional emails"
    layer: infrastructure
    path: src/services/email.ts

  Database:
    type: Component
    description: "PostgreSQL database connection"
    layer: infrastructure
```

### Step 3: Define Relationships

Connect nodes with edges:

```yaml
edges:
  # Feature implementations
  - from: AuthService
    to: UserRegistration
    relation: implements

  - from: AuthService
    to: UserLogin
    relation: implements

  - from: AuthService
    to: PasswordReset
    relation: implements

  # Component dependencies
  - from: AuthService
    to: UserService
    relation: depends_on

  - from: AuthService
    to: EmailService
    relation: depends_on

  - from: UserService
    to: Database
    relation: depends_on

  - from: EmailService
    to: Database
    relation: depends_on
```

---

## Complete Example

```yaml
# .gid/graph.yml
nodes:
  # ═══════════════════════════════════════════════════════════════════
  # Features (User-perceivable functionality)
  # ═══════════════════════════════════════════════════════════════════
  UserRegistration:
    type: Feature
    description: "User can create a new account with email verification"
    priority: core
    status: active

  OrderPlacement:
    type: Feature
    description: "User can place orders for products"
    priority: core
    status: active

  PaymentProcessing:
    type: Feature
    description: "User can pay using credit card or PayPal"
    priority: core
    status: active

  # ═══════════════════════════════════════════════════════════════════
  # Components - Interface Layer (API)
  # ═══════════════════════════════════════════════════════════════════
  UserController:
    type: Component
    description: "REST API endpoints for user operations"
    layer: interface
    path: src/controllers/user.controller.ts

  OrderController:
    type: Component
    description: "REST API endpoints for order operations"
    layer: interface
    path: src/controllers/order.controller.ts

  # ═══════════════════════════════════════════════════════════════════
  # Components - Application Layer (Business Logic)
  # ═══════════════════════════════════════════════════════════════════
  UserService:
    type: Component
    description: "User business logic and validation"
    layer: application
    path: src/services/user.service.ts

  OrderService:
    type: Component
    description: "Order processing and validation"
    layer: application
    path: src/services/order.service.ts

  PaymentService:
    type: Component
    description: "Payment processing integration"
    layer: application
    path: src/services/payment.service.ts

  # ═══════════════════════════════════════════════════════════════════
  # Components - Domain Layer (Models)
  # ═══════════════════════════════════════════════════════════════════
  User:
    type: Data
    description: "User entity model"
    layer: domain
    path: src/models/user.model.ts

  Order:
    type: Data
    description: "Order entity model"
    layer: domain
    path: src/models/order.model.ts

  # ═══════════════════════════════════════════════════════════════════
  # Components - Infrastructure Layer
  # ═══════════════════════════════════════════════════════════════════
  Database:
    type: Component
    description: "PostgreSQL database connection"
    layer: infrastructure

  StripeClient:
    type: Component
    description: "Stripe payment API client"
    layer: infrastructure

  EmailService:
    type: Component
    description: "SendGrid email integration"
    layer: infrastructure

edges:
  # ═══════════════════════════════════════════════════════════════════
  # Feature Implementations
  # ═══════════════════════════════════════════════════════════════════
  - from: UserService
    to: UserRegistration
    relation: implements

  - from: OrderService
    to: OrderPlacement
    relation: implements

  - from: PaymentService
    to: PaymentProcessing
    relation: implements

  # ═══════════════════════════════════════════════════════════════════
  # Interface → Application (Controllers call Services)
  # ═══════════════════════════════════════════════════════════════════
  - from: UserController
    to: UserService
    relation: depends_on

  - from: OrderController
    to: OrderService
    relation: depends_on

  - from: OrderController
    to: PaymentService
    relation: depends_on

  # ═══════════════════════════════════════════════════════════════════
  # Application → Application (Service dependencies)
  # ═══════════════════════════════════════════════════════════════════
  - from: OrderService
    to: UserService
    relation: depends_on
    description: "Validates user before creating order"

  - from: OrderService
    to: PaymentService
    relation: depends_on
    description: "Processes payment for order"

  - from: UserService
    to: EmailService
    relation: depends_on
    description: "Sends verification emails"

  # ═══════════════════════════════════════════════════════════════════
  # Application → Infrastructure (Services use infra)
  # ═══════════════════════════════════════════════════════════════════
  - from: UserService
    to: Database
    relation: depends_on

  - from: OrderService
    to: Database
    relation: depends_on

  - from: PaymentService
    to: StripeClient
    relation: depends_on

  # ═══════════════════════════════════════════════════════════════════
  # Data relationships
  # ═══════════════════════════════════════════════════════════════════
  - from: UserService
    to: User
    relation: reads

  - from: UserService
    to: User
    relation: writes

  - from: OrderService
    to: Order
    relation: reads

  - from: OrderService
    to: Order
    relation: writes
```

---

## Best Practices

### DO:
1. **Start with Features** - They anchor the graph to business value
2. **Use clear descriptions** - Every node should have a meaningful description
3. **Respect layer boundaries** - Dependencies flow interface → application → domain → infrastructure
4. **Keep it focused** - Include components that matter for understanding the system
5. **Update incrementally** - Evolve the graph as the system changes

### DON'T:
1. **Don't include every file** - Focus on meaningful components, not every utility
2. **Don't create circular dependencies** - A → B → C → A is a design smell
3. **Don't skip Features** - They provide the "why" for your components
4. **Don't over-complicate** - A simpler graph is more useful than a complete one
5. **Don't violate layers** - Infrastructure should never depend on application

---

## Validation Rules

After building the graph, validate with `gid check`:

| Rule | What it Checks |
|------|----------------|
| `no-circular-dependency` | No cycles in the dependency graph |
| `no-orphan-nodes` | All nodes are connected |
| `feature-has-implementation` | Every Feature has implementing Components |
| `component-implements-feature` | Components should implement Features |
| `layer-dependency-direction` | Layer boundaries are respected |
| `high-coupling-warning` | Warns on high fan-in/fan-out |

---

## Tips for AI Assistants

When helping users build graphs:

1. **Ask about the project's purpose** - What does it do for users?
2. **Identify the main features first** - What can users accomplish?
3. **Map the technical architecture** - What services/modules exist?
4. **Trace dependencies** - What calls what?
5. **Validate the result** - Run `gid check` to find issues

### Common Patterns

**Microservices:**
```yaml
# Each service is a Component with its own dependencies
UserService → UserDB
OrderService → OrderDB, UserService (via API)
```

**Monolith with layers:**
```yaml
Controllers → Services → Repositories → Database
```

**Frontend + Backend:**
```yaml
Frontend → API Gateway → Backend Services → Database
```

---

## Quick Reference

```yaml
# Minimal valid graph
nodes:
  MyFeature:
    type: Feature
    description: "What users can do"

  MyComponent:
    type: Component
    description: "How it's implemented"

edges:
  - from: MyComponent
    to: MyFeature
    relation: implements
```

---

## GID CLI Commands Reference

After building a graph, use these commands to work with it:

### Installation

```bash
npm install -g gid-cli
# or use npx
npx gid-cli --help
```

### `gid init` - Initialize a Graph

Creates a new `.gid/graph.yml` with a starter template.

```bash
gid init                    # Interactive mode
gid init --template minimal # Use minimal template
gid init --force            # Overwrite existing graph
```

**When to use**: Starting a new project or adding GID to an existing project.

### `gid extract` - Auto-Extract Dependencies

Automatically scans TypeScript/JavaScript code and generates a dependency graph.

```bash
gid extract .                           # Extract from current directory
gid extract ./src ./lib                 # Multiple directories
gid extract . --lang typescript         # Specify language (typescript/javascript)
gid extract . --ignore "*.test.ts"      # Ignore patterns
gid extract . --group                   # Group files into components by directory
gid extract . --dry-run                 # Preview without writing
gid extract . --interactive             # Guided extraction
```

**When to use**: Quick start for TS/JS projects. Creates a "code structure graph" that can be enhanced manually into a "semantic graph" with Features.

**Note**: Currently only supports TypeScript and JavaScript.

### `gid check` - Validate Graph

Runs integrity checks on the graph.

```bash
gid check                    # Run all checks
gid check --json             # Output as JSON
gid check --threshold 10     # Custom coupling threshold
gid check rules              # List available rules
```

**Available rules**:
| Rule | Description |
|------|-------------|
| `no-circular-dependency` | Detect circular dependencies |
| `no-orphan-nodes` | Find disconnected nodes |
| `feature-has-implementation` | Features must have implementing components |
| `component-implements-feature` | Components should implement features |
| `high-coupling-warning` | Warn on high fan-in/fan-out |
| `layer-dependency-direction` | Enforce layer boundaries |

**When to use**: After building/modifying a graph to ensure it's valid.

### `gid query` - Query Dependencies

Query the dependency graph for insights.

#### Impact Analysis
What is affected by changing a node?

```bash
gid query impact UserService
```

Output:
```
Impact Analysis for UserService
══════════════════════════════════════════════════
Direct dependents (3):
  ├── UserController
  ├── OrderService
  └── AuthService

Affected Features (2):
  ├── UserRegistration
  └── OrderPlacement

⚠ Changes to UserService may affect 3 component(s)
```

**When to use**: Before refactoring, to understand blast radius.

#### Dependency Lookup
What does a node depend on?

```bash
gid query deps UserService           # What UserService depends on
gid query deps UserService --reverse # What depends on UserService
```

**When to use**: Understanding a component's dependencies.

#### Common Cause Analysis
Find shared dependencies between two nodes (useful for debugging).

```bash
gid query common-cause OrderService PaymentService
```

Output:
```
Shared dependencies (2):
  ├── DatabaseService
  └── ConfigService

If both nodes fail together, check these common dependencies first.
```

**When to use**: Debugging why two services fail together.

#### Path Finding
Find dependency path between two nodes.

```bash
gid query path UserController Database
```

Output:
```
Path (3 hops):
  ├── UserController → ...
  ├── UserService → ...
  └── Database
```

**When to use**: Understanding how two components are connected.

### `gid serve` - Visualize Graph

Start a web-based graph visualization.

```bash
gid serve                    # Default port 3000
gid serve --port 8080        # Custom port
```

Opens an interactive D3.js visualization at `http://localhost:3000` with:
- Force-directed graph layout
- Node search
- Click to see dependencies
- Health score display

**When to use**: Exploring and presenting the architecture visually.

### `gid design` - AI-Assisted Design (Pro)

AI-assisted graph design (requires API key).

```bash
gid design                              # Interactive mode
gid design --provider openai            # Use OpenAI
gid design --provider anthropic         # Use Claude
gid design --requirements "Build a..." # Non-interactive
```

**When to use**: Getting AI help to design a new system architecture.

### `gid history` - Version Management

Manage graph versions (when using `--incremental`).

```bash
gid history list              # List versions
gid history diff <version>    # Compare versions
gid history restore <version> # Restore a version
```

**When to use**: Tracking graph evolution over time.

---

## Workflow for AI Assistants

When helping a user with GID:

### 1. For New Projects (No existing code)
```
User: "Help me design a system for X"
AI:
1. Ask about features/requirements
2. Create .gid/graph.yml with Features and Components
3. Run `gid check` to validate
4. Run `gid serve` to visualize
```

### 2. For Existing TS/JS Projects
```
User: "Add GID to my project"
AI:
1. Run `gid init`
2. Run `gid extract .` to get code structure
3. Help add Features to make it a semantic graph
4. Run `gid check` to validate
```

### 3. For Existing Non-TS/JS Projects
```
User: "Add GID to my Python/Go/etc. project"
AI:
1. Run `gid init`
2. Read the codebase structure
3. Manually create graph.yml with Features and Components
4. Run `gid check` to validate
```

### 4. For Refactoring
```
User: "I want to refactor X"
AI:
1. Run `gid query impact X` to see affected components
2. Run `gid query deps X` to understand dependencies
3. Plan refactoring based on impact analysis
4. Update graph after refactoring
5. Run `gid check` to ensure no new issues
```

### 5. For Debugging
```
User: "Services A and B keep failing together"
AI:
1. Run `gid query common-cause A B`
2. Investigate shared dependencies
3. Run `gid query path A B` to understand connection
```

---

*For more information, see the [GID Methodology](https://github.com/tonioyeme/graph-indexed-development)*
