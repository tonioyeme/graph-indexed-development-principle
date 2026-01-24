# Graph-Indexed Development (GID)

## A Software Development Methodology for the AI Era

**Author: Toni Tang**
**First Published: January 23, 2025**
**License: AGPL-3.0**

---

## 1. What Is This

Graph-Indexed Development is a software development methodology for the AI era.

**Core idea**: Organize all components of a software project—features, components, interfaces, data models, code files, tests—and their relationships into a **queryable graph**. This graph serves as the project's Single Source of Truth.

```
Graph = Nodes + Edges

Nodes: Feature, Component, Interface, Data, File, Test
Edges: implements, depends_on, calls, reads, writes, tested_by
```

### What This Graph Can Do

| Category | Capabilities |
|----------|--------------|
| **Development** | Clear path from Feature to code, precise impact analysis, automatic task decomposition |
| **AI Collaboration** | AI understands system through graph, Agent to Sub-agent task distribution |
| **Quality** | Auto-generate test plans, causal debugging, risk identification |
| **Project Management** | Time estimation, critical path analysis, resource allocation |
| **Knowledge** | Explicit architecture, new employee onboarding, decision tracing |

---

## 2. Why Do We Need It

### Current Pain Points

**Impact analysis relies on human memory:**
```
Scenario: Need to modify UserService's interface

Traditional: Developer recalls from memory, uses IDE search,
             misses indirect dependencies, repeats process every time

Graph-Indexed: Query "who depends_on UserService?" → Complete list instantly
               Works across packages, services, and AI can query too
```

**Direction is one-way (mostly Bottom-up):**
```
Current:
├── Change code → manually judge impact ✓ (common, but relies on experience)
└── New feature → auto-analyze required changes ✗ (difficult)

With Graph:
├── Bottom-up: Changed UserService → query upward → affects "Order", "Payment" Features
└── Top-down: Need "Export Orders" → query downward → need OrderService, new ExportService
```

**Knowledge stuck in people's heads:**
```
Current: "Why does UserService call CacheService?" → Only senior employees know
         Key person leaves → knowledge is lost

With Graph: Architecture is explicit, decisions recorded, new employees read the graph
```

**AI Coding bottleneck:**
```
Current: AI writes code quickly but doesn't understand full system picture
         Generated code may conflict with existing architecture

With Graph: AI knows dependencies, avoids breaking existing functionality
            Can decompose tasks for parallel sub-agent execution
```

---

## 3. Core Concepts

### 3.1 Node Types

| Type | Description | Examples |
|------|-------------|----------|
| **Feature** | User-perceivable functionality | User Registration, Order Payment |
| **Component** | Module/Service/Class | UserService, OrderService |
| **Interface** | API/Protocol | POST /api/users, gRPC Service |
| **Data** | Data Model/Schema | User, Order |
| **File** | Code file | user_service.py |
| **Test** | Test case | test_user_registration |

### 3.2 Edge Types

| Type | Direction | Description |
|------|-----------|-------------|
| **implements** | Component → Feature | Component implements feature |
| **depends_on** | Component → Component | A depends on B |
| **calls** | Component → Interface | Calls API |
| **reads/writes** | Component → Data | Reads/writes data |
| **tested_by** | Component → Test | Covered by test |

### 3.3 Direction and Causality

```
A ──depends_on──▶ B

Meaning:
├── A depends on B
├── If B changes, A may be affected
└── If A has problems, B may be the cause

Forward query: "Who is affected if B changes?" → Impact Analysis
Reverse query: "What might cause A's problem?" → Root Cause Analysis
```

---

## 4. Basic Usage

### 4.1 Minimum Viable Graph

```yaml
# .gid/graph.yml
nodes:
  UserRegistration:
    type: Feature

  UserService:
    type: Component

  OrderService:
    type: Component

edges:
  - { from: UserService, to: UserRegistration, relation: implements }
  - { from: OrderService, to: UserService, relation: depends_on }
```

### 4.2 Key Queries

| Query | Use Case |
|-------|----------|
| `getDependents(UserService)` | What is affected if I change UserService? |
| `getDependencies(OrderService)` | What does OrderService depend on? |
| `getImpactedFeatures(UserService)` | Which Features are affected? |
| `findCommonCause(A, B)` | A and B both failing—shared dependency? |

---

## 5. Two Generation Methods

### New Project (Top-down)
```
Idea → Feature doc → Component design → Build graph → Write code
Graph exists before code
```

### Existing Project (Bottom-up)
```
Code → Auto extract → AI inference → Manual completion → Complete graph
Graph is reverse-generated from code
```

---

## 6. Use Cases

### Impact Analysis
```
Input: I want to change UserService's interface
Output: Will affect OrderService, PaymentService, NotificationService
        Involves Features "User Order", "Payment Settlement"
        Need to update 3 unit tests and 2 integration tests
```

### Root Cause Analysis
```
Input: OrderService and PaymentService are both slow
Output: They share dependency on DatabaseService
        Likely a database issue—check one place
```

### Development Planning
```
Input: Need Feature "User can export order history"
Output: Create ExportService
        Depends on OrderService, StorageService
        3 tasks, 2 can be parallel
```

### AI Task Distribution
```
Input: Graph + Feature "Add shipping address management"
Output: AI decomposes into:
        - Sub-agent 1: Extend UserService
        - Sub-agent 2: Modify OrderService
        - Sub-agent 3: Update tests
```

---

## 7. Progressive Adoption

| Level | What to Do | Time |
|-------|-----------|------|
| L0 | Understand concepts | 1 hour |
| L1 | Create graph.yml, maintain manually | 2-4 hours |
| L2 | Write query functions, do impact analysis | Half day |
| L3 | Use tools for auto extraction | 1-2 days |
| L4 | AI-assisted inference | As needed |

**Start with Level 1**: 5-10 nodes is enough to be useful.

---

## 8. Tools

### GID CLI

Command-line tool for working with GID graphs:

```bash
npm install -g gid-cli

gid init          # Initialize graph
gid extract .     # Auto-extract from code (TS/JS)
gid check         # Validate graph integrity
gid query impact UserService   # Impact analysis
gid serve         # Web visualization
```

For full documentation, see: [github.com/tonioyeme/graph-indexed-development-gid-cli](https://github.com/tonioyeme/graph-indexed-development-gid-cli)

---

## 9. Relationship to Existing Knowledge

| Concept | What We Borrow |
|---------|----------------|
| **Graph Theory** | Nodes, edges, traversal, cycle detection |
| **Causal Inference** | Common Cause, Common Effect, Chain |
| **Knowledge Graph** | Typed nodes/edges, queryable |
| **C4 Model** | Abstraction levels (simplified) |
| **Clean Architecture** | Layer direction, dependency rules |
| **DDD** | Core/Supporting/Generic priority |

**Innovation**: Combines these for software engineering with bidirectional queries, auto-generation, and AI-friendliness.

---

## 10. Summary

```
1. Software projects need a Map (Single Source of Truth)
2. Use graph structure (Nodes + Edges + Direction)
3. Support bidirectional analysis (Top-down + Bottom-up)
4. Two generation methods (New project vs Existing project)
5. Progressive adoption (Start with YAML + core types)
```

**Core philosophy**: Map first, code second.

---

## Learn More

- **GID CLI Tool**: [github.com/tonioyeme/graph-indexed-development-gid-cli](https://github.com/tonioyeme/graph-indexed-development-gid-cli)
- **Full Documentation**: Available with GID Pro
- **Commercial Inquiries**: toni@gid.dev

---

*Document Version: 1.0 (Public)*
*Last Updated: 2025-01-24*
