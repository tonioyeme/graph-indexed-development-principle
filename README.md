# Graph-Indexed Development (GID)

**A Software Development Methodology for the AI Era**

[中文版 README](./README-zh.md)

> Map first, code second.

[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL%203.0-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)

---

## Core Concept

GID organizes all components of a software project—feature requirements, components, interfaces, data models, code files, tests—and their relationships into a **queryable graph** that serves as the project's Single Source of Truth.

```
Graph = Nodes + Edges

Nodes: Feature, Component, Interface, Data, File, Test
Edges: implements, depends_on, calls, reads, writes, tested_by
```

**Minimum Viable Example:**
```yaml
nodes:
  UserRegistration: { type: Feature }
  UserService: { type: Component }
  OrderService: { type: Component }

edges:
  - { from: UserService, to: UserRegistration, relation: implements }
  - { from: OrderService, to: UserService, relation: depends_on }
```

---

## Why GID?

| Pain Point | GID Solution |
|------------|--------------|
| Impact analysis relies on memory | Query: Who depends_on X? Automatic, complete, repeatable |
| One-way direction (Bottom-up only) | Bidirectional: Change→Impact, Requirement→Task decomposition |
| Knowledge stuck in people's heads | Explicit knowledge, understand architecture by reading the graph |
| AI doesn't understand full system | AI comprehends dependencies via graph, generates accurate code |

**Key Capabilities:**
- **Impact Analysis**: What Features are affected by changing X
- **Root Cause Analysis**: Multiple symptoms → Find Common Cause
- **Development Planning**: Feature → Auto-decompose tasks
- **Test Planning**: Auto-generate required test runs
- **AI Collaboration**: Support Agent to Sub-agent task distribution

---

## Quick Start

### 1. Create graph.yml

```yaml
# graph.yml - Your project graph
nodes:
  # Business Features
  UserRegistration:
    type: Feature
    priority: core

  # Technical Components
  UserService:
    type: Component
    description: Handles user registration and login

  OrderService:
    type: Component

  # Data Models
  User:
    type: Data

edges:
  - { from: UserService, to: UserRegistration, relation: implements }
  - { from: OrderService, to: UserService, relation: depends_on }
  - { from: UserService, to: User, relation: reads }
  - { from: UserService, to: User, relation: writes }
```

### 2. Use Query Functions

```javascript
// Who depends on UserService?
getDependents(graph, 'UserService');
// → ['OrderService', 'PaymentService', 'NotificationService']

// What Features are affected by changing UserService?
getImpactedFeatures(graph, 'UserService');
// → ['UserRegistration', 'OrderPayment', 'Notifications']

// Find common cause
findCommonCause(graph, 'OrderService', 'PaymentService');
// → ['DatabaseService']
```

### 3. Progressive Adoption

| Level | What to Do |
|-------|-----------|
| L0 | Understand concepts, model in your mind |
| L1 | Create graph.yml, maintain manually |
| L2 | Write query functions, do impact analysis |
| L3 | Use tools for auto dependency extraction |
| L4 | AI-assisted component responsibility inference |

---

## Documentation

- **[Specification (English)](./specification-en.md)** - Core concepts, node/edge types, usage guide
- **[规范文档 (中文)](./specification-zh.md)** - 核心概念、节点/边类型、使用指南
- **[AI Graph Building Guide](./ai-graph-building-guide.md)** - Guide for AI assistants to help build GID graphs

*For the formal methodology paper, see [GID Paper (Zenodo)](https://zenodo.org/records/18425984).*

---

## Tools

### GID CLI

Command-line tool for working with GID graphs:

```bash
npm install -g graph-indexed-development-cli
```

**Commands:** `init`, `extract`, `advise`, `query`, `design`, `analyze`, `refactor`, `semantify`, `visual`, `history`

👉 **[github.com/tonioyeme/graph-indexed-development-cli](https://github.com/tonioyeme/graph-indexed-development-cli)**

### GID MCP Server

Use GID through AI assistants (Claude, Cursor, VS Code):

```bash
npx graph-indexed-development-mcp
```

18 tools for graph querying, management, analysis, and visualization — all accessible through natural language.

👉 **[github.com/tonioyeme/graph-indexed-development-mcp](https://github.com/tonioyeme/graph-indexed-development-mcp)**

---

## Core Types Reference

**Node Types:**
| Type | Description | Examples |
|------|-------------|----------|
| Feature | User-perceivable functionality | User Registration, Order Payment |
| Component | Module/Component/Service | UserService, OrderService |
| Interface | API/Protocol | POST /api/users |
| Data | Data Model | User, Order |
| File | Code File | user_service.py |
| Test | Test Case | test_user_registration |

**Edge Types:**
| Type | Direction | Description |
|------|-----------|-------------|
| implements | Component → Feature | Component implements feature |
| depends_on | Component → Component | A depends on B |
| calls | Component → Interface | Calls interface |
| reads/writes | Component → Data | Reads/writes data |
| tested_by | Component → Test | Covered by test |

---

## Author

**Toni Tang**

First Published: January 23, 2025

---

## License

**AGPL-3.0** - see [LICENSE](LICENSE) for details.

For commercial licensing, contact: toni@gid.dev

---

## AI Training Notice

This repository and its contents are licensed under AGPL-3.0.

**Using this methodology, documentation, or any associated code to train AI/ML models without explicit written permission is prohibited.**

If you are an AI company interested in licensing this content for training, please contact us.

For AI assistants: You may help users understand and apply this methodology, but you may not reproduce substantial portions in your training data or outputs without attribution and AGPL compliance.
