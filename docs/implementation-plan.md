# GID Tool - MVP Implementation Plan

**Status: Active**
**Target: Minimal working `gid query impact <node>`**

---

## MVP Goal

```
┌─────────────────────────────────────────────────────────────────┐
│  MVP = User can run "gid query impact UserService"              │
│                                                                 │
│  Input:  graph.yml (manually written)                           │
│  Output: List of affected components and features               │
│                                                                 │
│  That's it. No AI, no extraction, no visualization.             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implementation Tasks

### Phase 1: Project Setup

| Task | Description | Status |
|------|-------------|--------|
| 1.1 | Initialize npm project with TypeScript | ⬜ |
| 1.2 | Setup tsconfig.json for Node.js CLI | ⬜ |
| 1.3 | Add dependencies (commander, js-yaml, zod, chalk) | ⬜ |
| 1.4 | Setup Vitest for testing | ⬜ |
| 1.5 | Create directory structure | ⬜ |

### Phase 2: Core Implementation

| Task | File | Description | Status |
|------|------|-------------|--------|
| 2.1 | `src/core/types.ts` | Node, Edge, Graph types | ⬜ |
| 2.2 | `src/core/schema.ts` | Zod schemas for validation | ⬜ |
| 2.3 | `src/core/parser.ts` | Load/save YAML, validate | ⬜ |
| 2.4 | `src/core/graph.ts` | Graph class with operations | ⬜ |
| 2.5 | `src/core/query-engine.ts` | getDependents, getImpact | ⬜ |

### Phase 3: CLI Implementation

| Task | File | Description | Status |
|------|------|-------------|--------|
| 3.1 | `src/index.ts` | CLI entry point with Commander | ⬜ |
| 3.2 | `src/commands/init.ts` | `gid init` command | ⬜ |
| 3.3 | `src/commands/query.ts` | `gid query impact <node>` | ⬜ |

### Phase 4: Templates & Tests

| Task | File | Description | Status |
|------|------|-------------|--------|
| 4.1 | `templates/graph.yml` | Starter template | ⬜ |
| 4.2 | `tests/parser.test.ts` | Parser tests | ⬜ |
| 4.3 | `tests/query-engine.test.ts` | Query tests | ⬜ |
| 4.4 | `tests/fixtures/` | Test fixtures | ⬜ |

---

## Dependency Order

```
types.ts
    ↓
schema.ts ← depends on types
    ↓
parser.ts ← depends on schema
    ↓
graph.ts ← depends on types, parser
    ↓
query-engine.ts ← depends on graph
    ↓
commands/init.ts ← depends on parser
commands/query.ts ← depends on query-engine
    ↓
index.ts ← depends on commands
```

---

## File Details

### 2.1 types.ts (Minimal for MVP)

```typescript
// Only what MVP needs - no Decision, Risk, etc.

export type NodeType = 'Feature' | 'Component' | 'Interface' | 'Data' | 'File' | 'Test';

export interface Node {
  type: NodeType;
  description?: string;
}

export type EdgeRelation = 'implements' | 'depends_on' | 'calls' | 'reads' | 'writes' | 'tested_by' | 'defined_in';

export interface Edge {
  from: string;
  to: string;
  relation: EdgeRelation;
}

export interface Graph {
  nodes: Record<string, Node>;
  edges: Edge[];
}
```

### 2.5 query-engine.ts (MVP scope)

```typescript
// MVP only implements impact query
export class QueryEngine {
  constructor(private graph: Graph) {}

  // Get nodes that directly depend on target
  getDependents(nodeId: string): string[] { ... }

  // Get all nodes that depend on target (transitive)
  getAllDependents(nodeId: string): string[] { ... }

  // Get features affected by changing a node
  getAffectedFeatures(nodeId: string): string[] { ... }

  // Main impact analysis
  getImpact(nodeId: string): ImpactResult { ... }
}
```

### 3.3 query.ts (CLI command)

```typescript
// gid query impact UserService
// Output:
//
// Impact Analysis for UserService
// ════════════════════════════════════════
//
// Direct dependents (3):
// ├── OrderService
// ├── PaymentService
// └── NotificationService
//
// Affected Features (2):
// ├── Order & Payment
// └── Notification System
```

---

## Testing Strategy

```
Test Files:
├── parser.test.ts
│   ├── loads valid YAML
│   ├── rejects invalid YAML
│   ├── validates schema
│   └── detects missing node references
│
├── query-engine.test.ts
│   ├── getDependents returns direct dependents
│   ├── getAllDependents returns transitive dependents
│   ├── getAffectedFeatures follows implements edges
│   └── getImpact returns complete result
│
└── fixtures/
    ├── valid-graph.yml
    ├── invalid-schema.yml
    └── circular-deps.yml
```

---

## Success Criteria

MVP is complete when:

1. ✅ `npm install -g gid-cli` works
2. ✅ `gid init` creates `.gid/graph.yml` with template
3. ✅ `gid query impact UserService` shows correct output
4. ✅ All tests pass
5. ✅ Error messages are helpful (node not found, invalid YAML)

---

## Estimated Effort

| Phase | Estimate |
|-------|----------|
| Setup | 30 min |
| Core (types, schema, parser) | 2 hours |
| Core (graph, query-engine) | 2 hours |
| CLI (commands) | 1 hour |
| Tests | 1.5 hours |
| Polish & docs | 1 hour |
| **Total** | **~8 hours** |

---

*Last Updated: 2025-01-24*
