# GID Tool Suite - Comprehensive Design

**Status: Design Draft (using GID methodology)**
**Author: Toni Tang & Claude**
**Created: 2025-01-24**

---

## 1. Overview: What We're Building

This document designs a **GID Tool Suite** that helps developers:
1. Generate graphs (top-down for new projects, bottom-up for existing systems)
2. Visualize and track development progress
3. Query graphs for feature changes and debugging
4. Evaluate and improve graph quality

We use the GID methodology itself to design this tool - practicing what we preach.

---

## 2. Features (Domain Layer)

Following GID, we first define the user-perceivable Features:

```
┌─────────────────────────────────────────────────────────────────┐
│                    GID Tool Suite Features                       │
└─────────────────────────────────────────────────────────────────┘

Generation Features (How to create the graph)
──────────────────────────────────────────────
F1. Top-down Graph Design          [core]
    User can interactively design a graph from requirements
    Input: Natural language description of features/system
    Output: graph.yml with Features, Components, and relationships

F2. Bottom-up Graph Extraction     [core]
    User can automatically extract a graph from existing code
    Input: Source code directory
    Output: graph.yml with extracted Components and dependencies

Visualization & Tracking Features (How to see progress)
──────────────────────────────────────────────────────────
F3. Graph Visualization            [core]
    User can visualize the dependency graph interactively
    Input: graph.yml
    Output: Interactive web-based graph view

F4. Development Progress Tracking  [core]
    User can see which nodes/edges are implemented
    Input: graph.yml + node status
    Output: Progress dashboard with completion percentages

Query Features (How to find information)
──────────────────────────────────────────────
F5. Impact Analysis Query          [core]
    User can query what's affected by changing a component
    Input: Component name
    Output: Affected components, features, and tests

F6. Feature Implementation Query   [core]
    User can query what needs to change for a feature
    Input: Feature name
    Output: Components to implement/modify, estimated scope

F7. Debug Root Cause Query         [core]
    User can find common causes when debugging
    Input: One or more symptomatic components
    Output: Common dependencies (potential root causes)

Evaluation Features (How to improve the graph)
──────────────────────────────────────────────
F8. Graph Health Evaluation        [core]
    User can evaluate graph quality and find issues
    Input: graph.yml
    Output: Health report with issues and suggestions

F9. Architecture Evolution (As-Is → To-Be)  [supporting]
    User can plan refactoring by comparing current vs target
    Input: as-is graph + to-be graph
    Output: Gap analysis and refactoring task list

F10. AI Improvement Suggestions    [supporting]
     User can get AI-powered suggestions to improve the graph
     Input: graph.yml
     Output: Suggested improvements (missing edges, descriptions, etc.)
```

---

## 3. Components (Design Layer)

Now we design Components that implement these Features:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Component Architecture                        │
└─────────────────────────────────────────────────────────────────┘

                    ┌────────────────────────────────────────┐
                    │           CLI Interface                │
                    │  (gid design/extract/query/check/serve)│
                    └────────────────────┬───────────────────┘
                                         │
            ┌────────────────────────────┼────────────────────────────┐
            │                            │                            │
            ▼                            ▼                            ▼
    ┌───────────────┐          ┌───────────────┐          ┌───────────────┐
    │   Designer    │          │   Extractor   │          │ Query Engine  │
    │  (Top-down)   │          │  (Bottom-up)  │          │               │
    └───────┬───────┘          └───────┬───────┘          └───────┬───────┘
            │                          │                          │
            │  ┌───────────────────────┼──────────────────────────┤
            │  │                       │                          │
            ▼  ▼                       ▼                          ▼
    ┌───────────────┐          ┌───────────────┐          ┌───────────────┐
    │  AI Client    │          │ Code Analyzer │          │  Validator    │
    │ (Claude API)  │          │ (LSP/Madge)   │          │  (Rules)      │
    └───────┬───────┘          └───────┬───────┘          └───────┬───────┘
            │                          │                          │
            └──────────────────────────┼──────────────────────────┘
                                       │
                                       ▼
                            ┌───────────────────┐
                            │   Graph Core      │
                            │  (Data Structure) │
                            └─────────┬─────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    │                 │                 │
                    ▼                 ▼                 ▼
            ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
            │ YAML Parser │   │ Visualizer  │   │ Progress    │
            │             │   │ (D3.js Web) │   │ Tracker     │
            └─────────────┘   └─────────────┘   └─────────────┘
```

### 3.1 Component Descriptions

```yaml
Components:
  # ═══════════════════════════════════════════════════════════
  # Interface Layer
  # ═══════════════════════════════════════════════════════════

  CLI:
    type: Component
    layer: interface
    description: |
      Command-line interface that exposes all tool functionality.
      Commands: init, design, extract, query, check, serve, export
    implements:
      - All Features (entry point)

  # ═══════════════════════════════════════════════════════════
  # Application Layer - Generation
  # ═══════════════════════════════════════════════════════════

  Designer:
    type: Component
    layer: application
    description: |
      Handles top-down graph design flow.
      Orchestrates AI-assisted feature decomposition and component design.
      Manages interactive user sessions for refinement.
    implements:
      - F1. Top-down Graph Design
    depends_on:
      - AIClient
      - GraphCore
      - PromptTemplates

  Extractor:
    type: Component
    layer: application
    description: |
      Handles bottom-up graph extraction from existing code.
      Coordinates multiple code analyzers and AI inference.
      Merges results and handles conflicts.
    implements:
      - F2. Bottom-up Graph Extraction
    depends_on:
      - CodeAnalyzer
      - AIClient
      - GraphCore

  # ═══════════════════════════════════════════════════════════
  # Application Layer - Query
  # ═══════════════════════════════════════════════════════════

  QueryEngine:
    type: Component
    layer: application
    description: |
      Executes graph queries for impact analysis, dependency lookup,
      root cause analysis, and path finding.
      Implements graph traversal algorithms.
    implements:
      - F5. Impact Analysis Query
      - F6. Feature Implementation Query
      - F7. Debug Root Cause Query
    depends_on:
      - GraphCore

  # ═══════════════════════════════════════════════════════════
  # Application Layer - Validation & Evaluation
  # ═══════════════════════════════════════════════════════════

  Validator:
    type: Component
    layer: application
    description: |
      Validates graph against integrity rules.
      Detects cycles, orphans, layer violations, high coupling.
      Generates health reports and suggestions.
    implements:
      - F8. Graph Health Evaluation
    depends_on:
      - GraphCore
      - RuleEngine

  EvolutionPlanner:
    type: Component
    layer: application
    description: |
      Compares As-Is and To-Be graphs.
      Generates gap analysis and refactoring task lists.
      Calculates migration paths.
    implements:
      - F9. Architecture Evolution (As-Is → To-Be)
    depends_on:
      - GraphCore
      - QueryEngine

  # ═══════════════════════════════════════════════════════════
  # Application Layer - Visualization
  # ═══════════════════════════════════════════════════════════

  Visualizer:
    type: Component
    layer: application
    description: |
      Renders interactive graph visualization.
      Supports filtering, zooming, node details, path highlighting.
      Web-based using D3.js.
    implements:
      - F3. Graph Visualization
    depends_on:
      - GraphCore
      - WebServer

  ProgressTracker:
    type: Component
    layer: application
    description: |
      Tracks development progress on nodes and edges.
      Calculates completion percentages by Feature.
      Integrates with git to detect implemented files.
    implements:
      - F4. Development Progress Tracking
    depends_on:
      - GraphCore
      - GitIntegration

  # ═══════════════════════════════════════════════════════════
  # Infrastructure Layer - AI
  # ═══════════════════════════════════════════════════════════

  AIClient:
    type: Component
    layer: infrastructure
    description: |
      Wrapper for AI API (Claude).
      Handles rate limiting, retries, and response parsing.
    depends_on:
      - PromptTemplates

  PromptTemplates:
    type: Component
    layer: infrastructure
    description: |
      Stores and manages prompt templates for different AI tasks:
      - Feature decomposition
      - Component design
      - Dependency inference
      - Code analysis

  AIInferrer:
    type: Component
    layer: application
    description: |
      Uses AI to infer missing information:
      - Component descriptions
      - Feature mappings (implements edges)
      - Architecture layers
      - Improvement suggestions
    implements:
      - F10. AI Improvement Suggestions
    depends_on:
      - AIClient
      - GraphCore

  # ═══════════════════════════════════════════════════════════
  # Infrastructure Layer - Code Analysis
  # ═══════════════════════════════════════════════════════════

  CodeAnalyzer:
    type: Component
    layer: infrastructure
    description: |
      Unified interface for code dependency extraction.
      Delegates to language-specific analyzers.
    depends_on:
      - TypeScriptAnalyzer
      - PythonAnalyzer
      - LSPAnalyzer

  TypeScriptAnalyzer:
    type: Component
    layer: infrastructure
    description: |
      Extracts dependencies from TypeScript/JavaScript code.
      Uses madge or ts-morph for AST analysis.

  PythonAnalyzer:
    type: Component
    layer: infrastructure
    description: |
      Extracts dependencies from Python code.
      Uses pydeps or AST analysis.

  LSPAnalyzer:
    type: Component
    layer: infrastructure
    description: |
      Universal analyzer using Language Server Protocol.
      Works with any language that has LSP support.

  # ═══════════════════════════════════════════════════════════
  # Domain Layer - Core
  # ═══════════════════════════════════════════════════════════

  GraphCore:
    type: Component
    layer: domain
    description: |
      Core graph data structure and operations.
      Defines Node, Edge, Graph types.
      Provides basic graph operations: add, remove, traverse.
      This is the heart of the system.

  RuleEngine:
    type: Component
    layer: domain
    description: |
      Defines and executes integrity rules.
      Rules can be built-in or user-defined.
      Returns validation results with suggestions.

  # ═══════════════════════════════════════════════════════════
  # Infrastructure Layer - Storage & I/O
  # ═══════════════════════════════════════════════════════════

  YAMLParser:
    type: Component
    layer: infrastructure
    description: |
      Parses and serializes graph.yml files.
      Handles schema validation and migration.

  WebServer:
    type: Component
    layer: infrastructure
    description: |
      Express server for visualization web app.
      Serves static files and graph data API.

  GitIntegration:
    type: Component
    layer: infrastructure
    description: |
      Integrates with git for:
      - Detecting which files exist (for progress tracking)
      - Reading change history
      - Linking commits to graph changes
```

---

## 4. The GID Graph for This Tool

Here's the actual graph.yml for this tool:

```yaml
# gid-tool-graph.yml
# GID graph for the GID Tool Suite itself

nodes:
  # ═══════════════════════════════════════════════════════════
  # Features
  # ═══════════════════════════════════════════════════════════

  F1-TopDownDesign:
    type: Feature
    description: "User can interactively design a graph from requirements"
    priority: core
    status: draft

  F2-BottomUpExtraction:
    type: Feature
    description: "User can automatically extract a graph from existing code"
    priority: core
    status: draft

  F3-GraphVisualization:
    type: Feature
    description: "User can visualize the dependency graph interactively"
    priority: core
    status: draft

  F4-ProgressTracking:
    type: Feature
    description: "User can see which nodes/edges are implemented"
    priority: core
    status: draft

  F5-ImpactAnalysis:
    type: Feature
    description: "User can query what's affected by changing a component"
    priority: core
    status: draft

  F6-FeatureImplementationQuery:
    type: Feature
    description: "User can query what needs to change for a feature"
    priority: core
    status: draft

  F7-DebugRootCause:
    type: Feature
    description: "User can find common causes when debugging"
    priority: core
    status: draft

  F8-GraphHealthEvaluation:
    type: Feature
    description: "User can evaluate graph quality and find issues"
    priority: core
    status: draft

  F9-ArchitectureEvolution:
    type: Feature
    description: "User can plan refactoring by comparing As-Is vs To-Be"
    priority: supporting
    status: draft

  F10-AIImprovementSuggestions:
    type: Feature
    description: "User can get AI-powered suggestions to improve the graph"
    priority: supporting
    status: draft

  # ═══════════════════════════════════════════════════════════
  # Components - Interface Layer
  # ═══════════════════════════════════════════════════════════

  CLI:
    type: Component
    description: "Command-line interface for all tool functionality"
    layer: interface
    status: draft

  # ═══════════════════════════════════════════════════════════
  # Components - Application Layer
  # ═══════════════════════════════════════════════════════════

  Designer:
    type: Component
    description: "Handles top-down graph design flow with AI assistance"
    layer: application
    status: draft

  Extractor:
    type: Component
    description: "Handles bottom-up graph extraction from code"
    layer: application
    status: draft

  QueryEngine:
    type: Component
    description: "Executes graph queries (impact, deps, root cause)"
    layer: application
    status: draft

  Validator:
    type: Component
    description: "Validates graph against integrity rules"
    layer: application
    status: draft

  EvolutionPlanner:
    type: Component
    description: "Compares As-Is and To-Be graphs, generates migration plan"
    layer: application
    status: draft

  Visualizer:
    type: Component
    description: "Renders interactive graph visualization"
    layer: application
    status: draft

  ProgressTracker:
    type: Component
    description: "Tracks development progress on nodes and edges"
    layer: application
    status: draft

  AIInferrer:
    type: Component
    description: "Uses AI to infer missing information and suggest improvements"
    layer: application
    status: draft

  # ═══════════════════════════════════════════════════════════
  # Components - Domain Layer
  # ═══════════════════════════════════════════════════════════

  GraphCore:
    type: Component
    description: "Core graph data structure and operations"
    layer: domain
    status: draft

  RuleEngine:
    type: Component
    description: "Defines and executes integrity rules"
    layer: domain
    status: draft

  # ═══════════════════════════════════════════════════════════
  # Components - Infrastructure Layer
  # ═══════════════════════════════════════════════════════════

  AIClient:
    type: Component
    description: "Wrapper for AI API (Claude)"
    layer: infrastructure
    status: draft

  PromptTemplates:
    type: Component
    description: "Stores prompt templates for AI tasks"
    layer: infrastructure
    status: draft

  CodeAnalyzer:
    type: Component
    description: "Unified interface for code dependency extraction"
    layer: infrastructure
    status: draft

  TypeScriptAnalyzer:
    type: Component
    description: "Extracts dependencies from TypeScript/JavaScript"
    layer: infrastructure
    status: draft

  PythonAnalyzer:
    type: Component
    description: "Extracts dependencies from Python code"
    layer: infrastructure
    status: draft

  LSPAnalyzer:
    type: Component
    description: "Universal analyzer using Language Server Protocol"
    layer: infrastructure
    status: draft

  YAMLParser:
    type: Component
    description: "Parses and serializes graph.yml files"
    layer: infrastructure
    status: draft

  WebServer:
    type: Component
    description: "Express server for visualization web app"
    layer: infrastructure
    status: draft

  GitIntegration:
    type: Component
    description: "Integrates with git for progress tracking"
    layer: infrastructure
    status: draft

edges:
  # ═══════════════════════════════════════════════════════════
  # implements relationships (Component → Feature)
  # ═══════════════════════════════════════════════════════════

  - { from: Designer, to: F1-TopDownDesign, relation: implements }
  - { from: Extractor, to: F2-BottomUpExtraction, relation: implements }
  - { from: Visualizer, to: F3-GraphVisualization, relation: implements }
  - { from: ProgressTracker, to: F4-ProgressTracking, relation: implements }
  - { from: QueryEngine, to: F5-ImpactAnalysis, relation: implements }
  - { from: QueryEngine, to: F6-FeatureImplementationQuery, relation: implements }
  - { from: QueryEngine, to: F7-DebugRootCause, relation: implements }
  - { from: Validator, to: F8-GraphHealthEvaluation, relation: implements }
  - { from: EvolutionPlanner, to: F9-ArchitectureEvolution, relation: implements }
  - { from: AIInferrer, to: F10-AIImprovementSuggestions, relation: implements }

  # ═══════════════════════════════════════════════════════════
  # depends_on relationships (Component → Component)
  # ═══════════════════════════════════════════════════════════

  # CLI depends on all application components
  - { from: CLI, to: Designer, relation: depends_on }
  - { from: CLI, to: Extractor, relation: depends_on }
  - { from: CLI, to: QueryEngine, relation: depends_on }
  - { from: CLI, to: Validator, relation: depends_on }
  - { from: CLI, to: Visualizer, relation: depends_on }
  - { from: CLI, to: ProgressTracker, relation: depends_on }

  # Application layer dependencies
  - { from: Designer, to: AIClient, relation: depends_on }
  - { from: Designer, to: GraphCore, relation: depends_on }
  - { from: Designer, to: PromptTemplates, relation: depends_on }

  - { from: Extractor, to: CodeAnalyzer, relation: depends_on }
  - { from: Extractor, to: AIClient, relation: depends_on }
  - { from: Extractor, to: GraphCore, relation: depends_on }

  - { from: QueryEngine, to: GraphCore, relation: depends_on }

  - { from: Validator, to: GraphCore, relation: depends_on }
  - { from: Validator, to: RuleEngine, relation: depends_on }

  - { from: EvolutionPlanner, to: GraphCore, relation: depends_on }
  - { from: EvolutionPlanner, to: QueryEngine, relation: depends_on }

  - { from: Visualizer, to: GraphCore, relation: depends_on }
  - { from: Visualizer, to: WebServer, relation: depends_on }

  - { from: ProgressTracker, to: GraphCore, relation: depends_on }
  - { from: ProgressTracker, to: GitIntegration, relation: depends_on }

  - { from: AIInferrer, to: AIClient, relation: depends_on }
  - { from: AIInferrer, to: GraphCore, relation: depends_on }

  # Infrastructure layer dependencies
  - { from: AIClient, to: PromptTemplates, relation: depends_on }

  - { from: CodeAnalyzer, to: TypeScriptAnalyzer, relation: depends_on }
  - { from: CodeAnalyzer, to: PythonAnalyzer, relation: depends_on }
  - { from: CodeAnalyzer, to: LSPAnalyzer, relation: depends_on }

  - { from: GraphCore, to: YAMLParser, relation: depends_on }
```

---

## 5. Detailed Feature Specifications

### 5.1 F1: Top-down Graph Design (最重要的Feature)

```
┌─────────────────────────────────────────────────────────────────┐
│                    Top-down Design Flow                          │
└─────────────────────────────────────────────────────────────────┘

User Input                    AI Processing                 Output
──────────────────────────────────────────────────────────────────
"I want to build             →  Decompose into Features  →  Features list
 an e-commerce                  (with priority)
 platform..."

[User confirms/edits]        →  For each Feature,        →  Components list
                                design Components            per Feature

[User confirms/edits]        →  Infer dependencies       →  depends_on edges
                                between Components

[User confirms/edits]        →  Generate graph.yml       →  graph.yml
                                + visualization              + web view
```

**CLI Interface:**
```bash
$ gid design

? Describe your project or feature:
> A task management app where users can create projects, add tasks,
> assign team members, and track progress with Kanban boards

Analyzing with AI...

┌─────────────────────────────────────────────────────────────────┐
│  Identified Features (5)                                         │
├─────────────────────────────────────────────────────────────────┤
│  [x] User Authentication              (core)                     │
│  [x] Project Management               (core)                     │
│  [x] Task CRUD & Assignment           (core)                     │
│  [x] Kanban Board View                (core)                     │
│  [x] Progress Analytics               (supporting)               │
└─────────────────────────────────────────────────────────────────┘
[Enter] Confirm  [e] Edit  [a] Add  [d] Delete

? Components for "Task CRUD & Assignment":
┌─────────────────────────────────────────────────────────────────┐
│  TaskService                                                     │
│  ├── Handles task creation, update, delete                       │
│  ├── depends_on: UserService, ProjectService                     │
│  └── implements: Task CRUD & Assignment                          │
├─────────────────────────────────────────────────────────────────┤
│  AssignmentService                                               │
│  ├── Handles task assignment to team members                     │
│  ├── depends_on: TaskService, UserService                        │
│  └── implements: Task CRUD & Assignment                          │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 F2: Bottom-up Extraction

```
┌─────────────────────────────────────────────────────────────────┐
│                  Three-Layer Extraction                          │
└─────────────────────────────────────────────────────────────────┘

Layer 1: Auto Extract (Tools)
─────────────────────────────────────────────────────────────────
Source Code → madge/pydeps/LSP → Basic Nodes & Edges

Extracts:
├── File structure → Component nodes
├── import/require → depends_on edges
├── API routes → Interface nodes
├── DB schemas → Data nodes
└── Test files → Test nodes


Layer 2: AI Inference (Semi-auto)
─────────────────────────────────────────────────────────────────
Basic Graph + Code → AI Analysis → Enriched Graph

Infers:
├── Component descriptions
├── Component types (UI/Logic/State)
├── Architecture layers
├── Feature mappings (implements edges)
└── Missing dependencies


Layer 3: Human Review (Required)
─────────────────────────────────────────────────────────────────
AI-enriched Graph → Human Review → Final Graph

Human adds:
├── Business semantics
├── Decision records (why designed this way)
├── Priority markings
├── Corrections to AI mistakes
```

**CLI Interface:**
```bash
$ gid extract ./src --lang typescript

Scanning ./src...
Found 47 files, 23 potential components

Layer 1: Extracting dependencies...
├── madge analysis: 67 import relationships
├── API routes: 12 endpoints
└── Done!

Layer 2: AI inference...
├── Analyzing component responsibilities...
├── Inferring Features...
├── Identifying layers...
└── Done!

Results:
├── 23 Component nodes
├── 67 depends_on edges
├── 5 inferred Features
├── 12 Interface nodes
└── 8 Data nodes

? Review and edit results? [Y/n]
> Y

[Opens interactive review interface]
```

### 5.3 F4: Development Progress Tracking

This is a feature you specifically mentioned - tracking where development is at.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Progress Tracking Model                       │
└─────────────────────────────────────────────────────────────────┘

Node Status:
─────────────────────────────────────────────────────────────────
draft        → Designed but not started
in_progress  → Currently being implemented
active       → Implemented and working
deprecated   → Planned for removal

Progress Calculation:
─────────────────────────────────────────────────────────────────
Feature Progress = Components(active) / Components(total) * 100%

Project Progress = Features(complete) / Features(total) * 100%

A Feature is "complete" when all implementing Components are "active"
```

**Progress Dashboard:**
```
┌─────────────────────────────────────────────────────────────────┐
│  GID Progress Dashboard                       Overall: 45%       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Features                                                        │
│  ─────────────────────────────────────────────────────────────  │
│  User Authentication      ████████████████████░░░░  80%  [core] │
│  ├── UserService          ████████████████████████  active      │
│  ├── AuthController       ████████████████████████  active      │
│  ├── SessionManager       ████████████████░░░░░░░░  in_progress │
│  └── PasswordHasher       ████████████████████████  active      │
│                                                                  │
│  Order & Payment          ████████░░░░░░░░░░░░░░░░  33%  [core] │
│  ├── OrderService         ████████████████████████  active      │
│  ├── PaymentService       ░░░░░░░░░░░░░░░░░░░░░░░░  draft       │
│  └── InventoryService     ░░░░░░░░░░░░░░░░░░░░░░░░  draft       │
│                                                                  │
│  Notification System      ░░░░░░░░░░░░░░░░░░░░░░░░  0%   [supp] │
│  └── NotificationService  ░░░░░░░░░░░░░░░░░░░░░░░░  draft       │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  Legend: ████ active  ░░░░ draft  ▓▓▓▓ in_progress              │
└─────────────────────────────────────────────────────────────────┘
```

**CLI Interface:**
```bash
$ gid progress

Overall Progress: 45% (4/9 components active)

Features:
  User Authentication     80%  ████████████████████░░░░
  Order & Payment         33%  ████████░░░░░░░░░░░░░░░░
  Notification System     0%   ░░░░░░░░░░░░░░░░░░░░░░░░

$ gid progress --update UserService --status active
Updated UserService: draft → active

$ gid progress --feature "Order & Payment"
Order & Payment: 33% complete

Components:
  ├── OrderService         active       (src/services/order.ts exists)
  ├── PaymentService       draft        (file not found)
  └── InventoryService     draft        (file not found)

Missing implementations:
  - PaymentService: expected at src/services/payment.ts
  - InventoryService: expected at src/services/inventory.ts
```

### 5.4 F5-F7: Query Features

**Impact Analysis (F5):**
```bash
$ gid query impact UserService

Impact Analysis for UserService
═══════════════════════════════════════════════════════════════════

Direct dependents (5):
├── OrderService
├── PaymentService
├── AuthController
├── SessionManager
└── NotificationService

Affected Features (3):
├── User Authentication      (via AuthController, SessionManager)
├── Order & Payment          (via OrderService, PaymentService)
└── Notification System      (via NotificationService)

Recommended actions:
├── Run tests: test_user_*.py, test_order_*.py, test_payment_*.py
├── Review: 5 components may need interface updates
└── Notify: Teams owning Order, Payment, Notification features
```

**Feature Implementation Query (F6):**
```bash
$ gid query feature "User can export order history"

Feature Implementation Plan
═══════════════════════════════════════════════════════════════════

Feature: User can export order history (new)

Suggested Components:
├── [NEW] ExportService
│   └── Handles file generation (CSV, PDF)
├── [MODIFY] OrderService
│   └── Add getOrderHistory() method
└── [DEPENDS] StorageService
    └── Already exists, store generated files

Dependencies:
ExportService
├── depends_on: OrderService (get order data)
├── depends_on: StorageService (store files)
└── depends_on: UserService (verify permissions)

Estimated scope:
├── 1 new component
├── 1 modified component
├── 2 existing dependencies
└── Complexity: Medium
```

**Debug Root Cause Query (F7):**
```bash
$ gid query common-cause OrderService PaymentService

Common Cause Analysis
═══════════════════════════════════════════════════════════════════

Symptom: Both OrderService and PaymentService are affected

Common dependencies found:
┌─────────────────────────────────────────────────────────────────┐
│                     Common Cause Diagram                         │
│                                                                  │
│            ┌─── DatabaseService ───┐     ← Check this first!    │
│            │     (common cause)     │                            │
│            ▼                       ▼                             │
│      OrderService            PaymentService                      │
│                                                                  │
│            ┌─── UserService ───────┐                             │
│            │   (common cause)      │                             │
│            ▼                       ▼                             │
│      OrderService            PaymentService                      │
└─────────────────────────────────────────────────────────────────┘

Debugging suggestions:
1. Check DatabaseService first (shared by both)
   - Connection pool exhausted?
   - Slow queries?

2. If DatabaseService is OK, check UserService
   - Authentication delays?
   - User lookup cache miss?

3. If no common cause, investigate independently
   - OrderService: specific order processing logic
   - PaymentService: payment gateway connectivity
```

### 5.5 F8-F9: Evaluation Features

**Graph Health Evaluation (F8):**
```bash
$ gid check

Graph Health Report
═══════════════════════════════════════════════════════════════════

Checks:                                          Status
────────────────────────────────────────────────────────
✓ no-circular-dependency                         PASSED
✗ core-feature-tested                            FAILED
⚠ high-coupling-warning                          WARNING
✓ domain-no-infrastructure-dep                   PASSED
⚠ orphan-component                               WARNING

Details:
────────────────────────────────────────────────────────

✗ core-feature-tested
  Feature "Order & Payment" (core) has untested components:
  └── PaymentService has no test coverage

  Suggestion: Add tests for PaymentService
              Priority: HIGH (core feature)

⚠ high-coupling-warning
  UserService has 8 dependents (threshold: 5)
  Dependents: OrderService, PaymentService, AuthController,
              SessionManager, NotificationService, ReportService,
              AdminService, ProfileService

  Suggestion: Consider splitting UserService
              - AuthService (authentication logic)
              - ProfileService (user profile management)

⚠ orphan-component
  HelperUtils implements no Feature

  Suggestion: Either link to a Feature or consider removing

Summary: 2 passed, 1 failed, 2 warnings
Health Score: 60/100
```

**Architecture Evolution (F9):**
```bash
$ gid evolve --from as-is.yml --to to-be.yml

Architecture Evolution Plan
═══════════════════════════════════════════════════════════════════

Changes detected:
────────────────────────────────────────────────────────

Added (2):
  + EventBus (Component)
  + PaymentCompletedEvent (Interface)

Removed (1):
  - PaymentService → OrderService (depends_on edge)

Modified (2):
  ~ PaymentService: now sends_to EventBus
  ~ OrderService: now depends_on EventBus

Migration Tasks:
────────────────────────────────────────────────────────

1. [ ] Create EventBus component
       - Implement pub/sub mechanism
       - Add PaymentCompletedEvent interface

2. [ ] Modify PaymentService
       - Replace direct OrderService call with event emission
       - Add EventBus dependency

3. [ ] Modify OrderService
       - Subscribe to PaymentCompletedEvent
       - Remove direct PaymentService import

4. [ ] Update tests
       - Add EventBus unit tests
       - Update integration tests for async flow

5. [ ] Remove old dependency
       - Delete PaymentService → OrderService edge from graph

Estimated effort: 3-5 development tasks
Risk: Medium (changing payment flow)
```

---

## 6. Implementation Phases

### Phase 1: Core Foundation (MVP)

**Goal: Basic graph operations + simple queries**

```
Priority: P0 (Must have)

Components to implement:
├── GraphCore (data structure, basic operations)
├── YAMLParser (read/write graph.yml)
├── QueryEngine (basic queries: deps, dependents)
├── CLI (init, query commands)
└── Validator (circular dependency check only)

Features enabled:
├── F5. Impact Analysis (basic)
└── F8. Graph Health (cycle detection only)
```

### Phase 2: Extraction

**Goal: Bottom-up graph generation**

```
Priority: P0 (Must have)

Components to implement:
├── CodeAnalyzer (unified interface)
├── TypeScriptAnalyzer (madge integration)
└── Extractor (orchestration)

Features enabled:
└── F2. Bottom-up Extraction (TypeScript only)
```

### Phase 3: AI Integration

**Goal: AI-assisted design and inference**

```
Priority: P1 (Should have)

Components to implement:
├── AIClient (Claude API)
├── PromptTemplates
├── Designer (top-down flow)
└── AIInferrer (enrich extracted graphs)

Features enabled:
├── F1. Top-down Design
└── F10. AI Improvement Suggestions
```

### Phase 4: Visualization & Tracking

**Goal: See and track progress**

```
Priority: P1 (Should have)

Components to implement:
├── WebServer
├── Visualizer (D3.js frontend)
├── ProgressTracker
└── GitIntegration

Features enabled:
├── F3. Graph Visualization
└── F4. Development Progress Tracking
```

### Phase 5: Advanced Features

**Goal: Full evaluation and evolution**

```
Priority: P2 (Nice to have)

Components to implement:
├── RuleEngine (custom rules)
├── EvolutionPlanner
├── PythonAnalyzer
└── LSPAnalyzer

Features enabled:
├── F8. Graph Health (full rule set)
├── F9. Architecture Evolution
└── F2. Bottom-up (Python + more languages)
```

---

## 7. Technical Implementation Details

### 7.1 TypeScript Type Definitions

```typescript
// src/core/types.ts
// Core type definitions for the GID tool

// ═══════════════════════════════════════════════════════════════════════════════
// Node Types
// ═══════════════════════════════════════════════════════════════════════════════

export type NodeType =
  | 'Feature'
  | 'Component'
  | 'Interface'
  | 'Data'
  | 'File'
  | 'Test'
  | 'Decision'
  | 'Constraint'
  | 'Risk';

export type NodeStatus = 'draft' | 'in_progress' | 'active' | 'deprecated';

export type FeaturePriority = 'core' | 'supporting' | 'generic';

export type ComponentLayer = 'interface' | 'application' | 'domain' | 'infrastructure';

export type ComponentType = 'UI' | 'Logic' | 'State' | 'Config';

export interface BaseNode {
  type: NodeType;
  description?: string;
  status?: NodeStatus;
}

export interface FeatureNode extends BaseNode {
  type: 'Feature';
  priority?: FeaturePriority;
}

export interface ComponentNode extends BaseNode {
  type: 'Component';
  layer?: ComponentLayer;
  componentType?: ComponentType;  // renamed to avoid conflict with 'type'
  owner?: string;
}

export interface InterfaceNode extends BaseNode {
  type: 'Interface';
  signature?: string;
  interfaceType?: 'sync_api' | 'async_event' | 'data_contract';
}

export interface DataNode extends BaseNode {
  type: 'Data';
  schema?: Record<string, string>;
}

export interface FileNode extends BaseNode {
  type: 'File';
  path: string;
  language?: string;
}

export interface TestNode extends BaseNode {
  type: 'Test';
  testType?: 'unit' | 'integration' | 'e2e';
}

export interface DecisionNode extends BaseNode {
  type: 'Decision';
  title: string;
  rationale: string;
  date?: string;
  decisionStatus?: 'proposed' | 'accepted' | 'deprecated' | 'superseded';
}

export type Node =
  | FeatureNode
  | ComponentNode
  | InterfaceNode
  | DataNode
  | FileNode
  | TestNode
  | DecisionNode;

// ═══════════════════════════════════════════════════════════════════════════════
// Edge Types
// ═══════════════════════════════════════════════════════════════════════════════

export type EdgeRelation =
  | 'implements'
  | 'depends_on'
  | 'calls'
  | 'reads'
  | 'writes'
  | 'tested_by'
  | 'defined_in'
  | 'decided_by'
  | 'constrained_by';

export type CouplingLevel = 'tight' | 'loose';

export interface BaseEdge {
  from: string;
  to: string;
  relation: EdgeRelation;
}

export interface DependsOnEdge extends BaseEdge {
  relation: 'depends_on';
  coupling?: CouplingLevel;
  optional?: boolean;
}

export interface CallsEdge extends BaseEdge {
  relation: 'calls';
  sync?: boolean;
  technology?: string;
}

export type Edge = BaseEdge | DependsOnEdge | CallsEdge;

// ═══════════════════════════════════════════════════════════════════════════════
// Graph Structure
// ═══════════════════════════════════════════════════════════════════════════════

export interface Graph {
  nodes: Record<string, Node>;
  edges: Edge[];
  integrity_rules?: IntegrityRule[];
}

export interface IntegrityRule {
  name: string;
  description: string;
  severity: 'error' | 'warning' | 'info';
  check?: string;
  threshold?: number;
}

// ═══════════════════════════════════════════════════════════════════════════════
// Query Results
// ═══════════════════════════════════════════════════════════════════════════════

export interface ImpactResult {
  node: string;
  directDependents: string[];
  transitiveDependents: string[];
  affectedFeatures: string[];
  suggestedTests: string[];
}

export interface DependencyResult {
  node: string;
  direct: string[];
  transitive: string[];
  depth: number;
}

export interface CommonCauseResult {
  nodeA: string;
  nodeB: string;
  commonDependencies: string[];
  suggestion: string;
}

export interface PathResult {
  from: string;
  to: string;
  paths: string[][];  // Multiple possible paths
  shortestPath: string[];
  hops: number;
}

export type QueryResult =
  | ImpactResult
  | DependencyResult
  | CommonCauseResult
  | PathResult;

// ═══════════════════════════════════════════════════════════════════════════════
// Validation Results
// ═══════════════════════════════════════════════════════════════════════════════

export interface ValidationIssue {
  rule: string;
  severity: 'error' | 'warning' | 'info';
  message: string;
  nodes?: string[];
  edges?: Edge[];
  suggestion?: string;
}

export interface ValidationResult {
  passed: boolean;
  issues: ValidationIssue[];
  healthScore: number;  // 0-100
  summary: {
    errors: number;
    warnings: number;
    info: number;
  };
}

// ═══════════════════════════════════════════════════════════════════════════════
// Error Types
// ═══════════════════════════════════════════════════════════════════════════════

export class GIDError extends Error {
  constructor(
    message: string,
    public code: GIDErrorCode,
    public details?: Record<string, unknown>
  ) {
    super(message);
    this.name = 'GIDError';
  }
}

export type GIDErrorCode =
  | 'PARSE_ERROR'           // YAML syntax error
  | 'SCHEMA_ERROR'          // Invalid graph structure
  | 'NODE_NOT_FOUND'        // Referenced node doesn't exist
  | 'EDGE_INVALID'          // Edge references non-existent nodes
  | 'CYCLE_DETECTED'        // Circular dependency found
  | 'FILE_NOT_FOUND'        // graph.yml not found
  | 'AI_API_ERROR'          // Claude API error
  | 'EXTRACTION_ERROR';     // Code analysis failed
```

### 7.2 Project Structure

```
gid-cli/
├── package.json
├── tsconfig.json
├── .gid/
│   └── config.yml              # Tool configuration
│
├── src/
│   ├── index.ts                # CLI entry point
│   │
│   ├── commands/               # CLI commands
│   │   ├── init.ts             # gid init
│   │   ├── query.ts            # gid query (impact, deps, etc.)
│   │   ├── check.ts            # gid check
│   │   ├── design.ts           # gid design (Phase 3)
│   │   ├── extract.ts          # gid extract (Phase 2)
│   │   └── serve.ts            # gid serve (Phase 4)
│   │
│   ├── core/                   # Core domain logic (Phase 1)
│   │   ├── types.ts            # TypeScript types (above)
│   │   ├── graph.ts            # Graph class with operations
│   │   ├── parser.ts           # YAML parser + schema validation
│   │   ├── query-engine.ts     # Query algorithms
│   │   └── validator.ts        # Rule validation
│   │
│   ├── extractors/             # Code extractors (Phase 2)
│   │   ├── index.ts            # Unified interface
│   │   ├── typescript.ts       # TS/JS extractor (madge)
│   │   └── python.ts           # Python extractor (pydeps)
│   │
│   ├── ai/                     # AI integration (Phase 3)
│   │   ├── client.ts           # Claude API wrapper
│   │   ├── prompts/            # Prompt templates
│   │   │   ├── feature-decomposition.ts
│   │   │   ├── component-design.ts
│   │   │   └── dependency-inference.ts
│   │   └── designer.ts         # Top-down design orchestration
│   │
│   └── web/                    # Visualization (Phase 4)
│       ├── server.ts           # Express server
│       └── public/
│           ├── index.html
│           ├── graph.js        # D3.js visualization
│           └── styles.css
│
├── templates/
│   └── graph.yml               # Template for gid init
│
└── tests/
    ├── core/
    │   ├── parser.test.ts
    │   ├── query-engine.test.ts
    │   └── validator.test.ts
    └── fixtures/
        └── sample-graph.yml
```

### 7.3 Error Handling Design

```typescript
// Error handling strategy

// 1. YAML Parse Errors
try {
  const graph = parser.load('graph.yml');
} catch (e) {
  if (e instanceof GIDError && e.code === 'PARSE_ERROR') {
    console.error(`YAML syntax error at line ${e.details?.line}:`);
    console.error(`  ${e.details?.snippet}`);
    console.error(`  ${'^'.padStart(e.details?.column)}`);
    console.error(`\nTip: Check for incorrect indentation or missing colons`);
    process.exit(1);
  }
}

// 2. Schema Validation Errors
const validationErrors = parser.validate(graph);
if (validationErrors.length > 0) {
  console.error('Graph schema errors:\n');
  for (const err of validationErrors) {
    console.error(`  ✗ ${err.path}: ${err.message}`);
    if (err.suggestion) {
      console.error(`    Suggestion: ${err.suggestion}`);
    }
  }
  process.exit(1);
}

// 3. Reference Errors (node not found)
// When edge references non-existent node:
//   edges:
//     - { from: OrderService, to: NonExistent, relation: depends_on }
//
// Error output:
//   ✗ Edge references non-existent node: "NonExistent"
//     at edges[3]: { from: "OrderService", to: "NonExistent" }
//
//   Did you mean one of these?
//     - UserService
//     - NotificationService

// 4. Cycle Detection
// When circular dependency found:
//   ⚠ Circular dependency detected:
//     OrderService → PaymentService → InventoryService → OrderService
//
//   Suggestion: Consider introducing an event bus to decouple these services
//   See: https://gid.dev/docs/patterns/breaking-cycles

// 5. AI API Errors
try {
  const result = await aiClient.decompose(requirements);
} catch (e) {
  if (e instanceof GIDError && e.code === 'AI_API_ERROR') {
    if (e.details?.status === 429) {
      console.error('Rate limited. Retrying in 60 seconds...');
      await sleep(60000);
      return retry();
    }
    console.error(`AI API error: ${e.message}`);
    console.error('Falling back to manual mode...');
    return manualDesignFlow();
  }
}
```

### 7.4 Schema Validation (Zod)

```typescript
// src/core/schema.ts
import { z } from 'zod';

// Node type enum
const NodeTypeSchema = z.enum([
  'Feature', 'Component', 'Interface', 'Data', 'File', 'Test', 'Decision'
]);

// Edge relation enum
const EdgeRelationSchema = z.enum([
  'implements', 'depends_on', 'calls', 'reads', 'writes',
  'tested_by', 'defined_in', 'decided_by'
]);

// Base node schema
const BaseNodeSchema = z.object({
  type: NodeTypeSchema,
  description: z.string().optional(),
  status: z.enum(['draft', 'in_progress', 'active', 'deprecated']).optional(),
}).passthrough();  // Allow additional properties

// Feature node
const FeatureNodeSchema = BaseNodeSchema.extend({
  type: z.literal('Feature'),
  priority: z.enum(['core', 'supporting', 'generic']).optional(),
});

// Component node
const ComponentNodeSchema = BaseNodeSchema.extend({
  type: z.literal('Component'),
  layer: z.enum(['interface', 'application', 'domain', 'infrastructure']).optional(),
});

// Node schema (union)
const NodeSchema = z.union([
  FeatureNodeSchema,
  ComponentNodeSchema,
  BaseNodeSchema,  // Fallback for other types
]);

// Edge schema
const EdgeSchema = z.object({
  from: z.string().min(1),
  to: z.string().min(1),
  relation: EdgeRelationSchema,
  coupling: z.enum(['tight', 'loose']).optional(),
  optional: z.boolean().optional(),
  sync: z.boolean().optional(),
});

// Full graph schema
export const GraphSchema = z.object({
  nodes: z.record(z.string(), NodeSchema),
  edges: z.array(EdgeSchema),
  integrity_rules: z.array(z.object({
    name: z.string(),
    description: z.string(),
    severity: z.enum(['error', 'warning', 'info']),
  })).optional(),
});

// Validate function
export function validateGraph(data: unknown): ValidationResult {
  const result = GraphSchema.safeParse(data);

  if (!result.success) {
    return {
      valid: false,
      errors: result.error.issues.map(issue => ({
        path: issue.path.join('.'),
        message: issue.message,
        suggestion: getSuggestion(issue),
      })),
    };
  }

  // Additional semantic validation
  const semanticErrors = validateSemantics(result.data);

  return {
    valid: semanticErrors.length === 0,
    errors: semanticErrors,
  };
}

// Semantic validation (references, etc.)
function validateSemantics(graph: Graph): ValidationError[] {
  const errors: ValidationError[] = [];
  const nodeIds = new Set(Object.keys(graph.nodes));

  // Check edge references
  for (const [index, edge] of graph.edges.entries()) {
    if (!nodeIds.has(edge.from)) {
      errors.push({
        path: `edges[${index}].from`,
        message: `Node "${edge.from}" not found`,
        suggestion: findSimilarNode(edge.from, nodeIds),
      });
    }
    if (!nodeIds.has(edge.to)) {
      errors.push({
        path: `edges[${index}].to`,
        message: `Node "${edge.to}" not found`,
        suggestion: findSimilarNode(edge.to, nodeIds),
      });
    }
  }

  return errors;
}
```

### 7.5 AI Interaction Design

```typescript
// src/ai/designer.ts
// Top-down design flow with AI assistance

export class AIDesigner {
  private client: AIClient;
  private context: DesignContext;
  private history: ConversationTurn[] = [];

  constructor(client: AIClient) {
    this.client = client;
    this.context = { features: [], components: [], edges: [] };
  }

  // ═══════════════════════════════════════════════════════════════════════════
  // Step 1: Feature Decomposition
  // ═══════════════════════════════════════════════════════════════════════════

  async decomposeFeatures(requirements: string): Promise<FeatureProposal[]> {
    const prompt = `
You are helping design a software system. Given the following requirements,
identify the user-perceivable Features.

Requirements:
${requirements}

Return a JSON array of features:
[
  {
    "name": "Feature Name",
    "description": "One sentence description",
    "priority": "core" | "supporting" | "generic"
  }
]

Guidelines:
- A Feature is something a user can perceive or interact with
- "core" = essential for MVP
- "supporting" = enhances core features
- "generic" = nice to have
`;

    const response = await this.client.complete(prompt);

    // Parse with retry on failure
    try {
      return this.parseFeatures(response);
    } catch (e) {
      // Ask AI to fix the format
      return this.retryWithFeedback(response, e.message);
    }
  }

  // ═══════════════════════════════════════════════════════════════════════════
  // Step 2: Component Design (per Feature)
  // ═══════════════════════════════════════════════════════════════════════════

  async designComponents(feature: Feature): Promise<ComponentProposal[]> {
    const prompt = `
Design the Components needed to implement this Feature.

Feature: ${feature.name}
Description: ${feature.description}

Existing Components in the system:
${this.context.components.map(c => `- ${c.name}: ${c.description}`).join('\n')}

Return a JSON array:
[
  {
    "name": "ComponentName",
    "description": "Responsibility description",
    "layer": "interface" | "application" | "domain" | "infrastructure",
    "isNew": true,  // or false if reusing existing
    "dependsOn": ["OtherComponent"]
  }
]
`;

    const response = await this.client.complete(prompt);
    return this.parseComponents(response);
  }

  // ═══════════════════════════════════════════════════════════════════════════
  // User Feedback Handling
  // ═══════════════════════════════════════════════════════════════════════════

  async handleFeedback(feedback: UserFeedback): Promise<FeatureProposal[]> {
    // Store in history for context
    this.history.push({ role: 'user', content: feedback.message });

    const prompt = `
The user provided feedback on your previous suggestions.

Previous suggestions:
${JSON.stringify(this.context.features, null, 2)}

User feedback: "${feedback.message}"

Please revise your suggestions based on this feedback.
Return the updated JSON array.
`;

    const response = await this.client.complete(prompt);
    this.history.push({ role: 'assistant', content: response });

    return this.parseFeatures(response);
  }

  // ═══════════════════════════════════════════════════════════════════════════
  // Retry Logic
  // ═══════════════════════════════════════════════════════════════════════════

  private async retryWithFeedback(
    originalResponse: string,
    errorMessage: string,
    maxRetries = 2
  ): Promise<any> {
    for (let i = 0; i < maxRetries; i++) {
      const prompt = `
Your previous response could not be parsed as JSON.

Your response:
${originalResponse}

Error: ${errorMessage}

Please provide a valid JSON response.
`;

      const response = await this.client.complete(prompt);
      try {
        return JSON.parse(response);
      } catch (e) {
        errorMessage = e.message;
        originalResponse = response;
      }
    }

    throw new GIDError(
      'Failed to get valid JSON from AI after retries',
      'AI_API_ERROR',
      { originalResponse }
    );
  }
}
```

### 7.6 Progress Detection Strategies

```typescript
// src/core/progress.ts
// Multiple strategies for detecting implementation status

export interface ProgressDetector {
  detect(node: ComponentNode, graph: Graph): NodeStatus;
}

// Strategy 1: File existence
export class FileExistenceDetector implements ProgressDetector {
  detect(node: ComponentNode, graph: Graph): NodeStatus {
    // Find defined_in edge
    const fileEdge = graph.edges.find(
      e => e.from === node.id && e.relation === 'defined_in'
    );

    if (!fileEdge) return 'draft';

    const fileNode = graph.nodes[fileEdge.to] as FileNode;
    if (!fileNode?.path) return 'draft';

    // Check if file exists and is not empty
    if (fs.existsSync(fileNode.path)) {
      const content = fs.readFileSync(fileNode.path, 'utf-8');
      if (content.trim().length > 0) {
        return 'active';
      }
    }

    return 'draft';
  }
}

// Strategy 2: Test pass status
export class TestPassDetector implements ProgressDetector {
  detect(node: ComponentNode, graph: Graph): NodeStatus {
    const testEdges = graph.edges.filter(
      e => e.from === node.id && e.relation === 'tested_by'
    );

    if (testEdges.length === 0) return 'draft';

    // Run tests (simplified)
    const allTestsPass = testEdges.every(edge => {
      const testNode = graph.nodes[edge.to] as TestNode;
      return this.runTest(testNode);
    });

    return allTestsPass ? 'active' : 'in_progress';
  }

  private runTest(test: TestNode): boolean {
    // Implementation depends on test runner
    return true;
  }
}

// Strategy 3: Git-based (file exists in git)
export class GitBasedDetector implements ProgressDetector {
  detect(node: ComponentNode, graph: Graph): NodeStatus {
    const fileEdge = graph.edges.find(
      e => e.from === node.id && e.relation === 'defined_in'
    );

    if (!fileEdge) return 'draft';

    const fileNode = graph.nodes[fileEdge.to] as FileNode;

    // Check if file is tracked in git
    try {
      execSync(`git ls-files --error-unmatch "${fileNode.path}"`, {
        stdio: 'pipe'
      });
      return 'active';
    } catch {
      return 'draft';
    }
  }
}

// Composite detector (user-configurable)
export class CompositeDetector implements ProgressDetector {
  constructor(private strategies: ProgressDetector[]) {}

  detect(node: ComponentNode, graph: Graph): NodeStatus {
    for (const strategy of this.strategies) {
      const status = strategy.detect(node, graph);
      if (status !== 'draft') return status;
    }
    return 'draft';
  }
}

// Configuration
// .gid/config.yml
// progress:
//   detection:
//     - file_exists
//     - git_tracked
//   allow_manual_override: true
```

---

## 8. Refined MVP Scope

Based on feedback, here's a **minimal** MVP:

```
┌─────────────────────────────────────────────────────────────────┐
│                    MVP (Minimum Viable Product)                  │
│                                                                  │
│  Goal: "I can query impact of changes in my project"             │
│                                                                  │
│  Components (3 only):                                            │
│  ├── GraphCore     - Load/save graph, basic operations           │
│  ├── QueryEngine   - Impact query only                           │
│  └── CLI           - gid query impact <node>                     │
│                                                                  │
│  Commands (2 only):                                              │
│  ├── gid init      - Create empty graph.yml                      │
│  └── gid query impact <node>                                     │
│                                                                  │
│  No AI, no extraction, no visualization, no validation           │
│  Just: read YAML → run query → print result                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

Why this is enough:
1. User manually writes graph.yml (we provide template)
2. User can query "what's affected if I change X"
3. This alone provides value and validates the concept

After MVP works:
├── Phase 1.5: Add gid check (validation)
├── Phase 2: Add gid extract (auto-extraction)
├── Phase 3: Add gid design (AI-assisted)
└── Phase 4: Add gid serve (visualization)
```

### MVP Implementation Checklist

```
[ ] Core
    [ ] types.ts - Define Node, Edge, Graph types
    [ ] parser.ts - Load/save YAML, basic validation
    [ ] graph.ts - Graph class with getNode, getEdges methods
    [ ] query-engine.ts - getDependents, getImpact functions

[ ] CLI
    [ ] index.ts - Commander setup
    [ ] commands/init.ts - Create .gid/graph.yml from template
    [ ] commands/query.ts - gid query impact <node>

[ ] Templates
    [ ] graph.yml - Minimal template with example

[ ] Tests
    [ ] parser.test.ts
    [ ] query-engine.test.ts

Estimated: 2-3 days to working MVP
```

---

## 9. Remaining Design Questions

### Q1: Storage Format
Should we start with YAML only, or support JSON/SQLite from the start?

**Recommendation**: Start with YAML only. Add JSON export later. SQLite only if > 1000 nodes.

### Q2: Visualization Library
D3.js (flexible but complex) vs Mermaid (simple but limited)?

**Recommendation**: Start with Mermaid for CLI output, D3.js for web interface.

### Q3: How to auto-detect progress?
How to know if a Component is "implemented"?

**Options**:
- Check if defined_in file exists
- Check if tests pass
- Manual update only

**Recommendation**: Check file existence + allow manual override.

### Q4: Graph file location
Where should graph.yml live?

**Options**:
- Project root: `graph.yml`
- Hidden dir: `.gid/graph.yml`
- Custom via config

**Recommendation**: `.gid/graph.yml` with config override option.

---

## 10. Technology Stack Decisions

This section documents the technology choices and their rationale. Following GID's Decision node concept, we record "why we designed it this way".

### 10.1 Language: TypeScript

**Decision**: Use TypeScript as the primary language.

**Alternatives Considered**:

| Language | Pros | Cons |
|----------|------|------|
| TypeScript | Best ecosystem for code analysis tools | N/A |
| Python | Good AI SDK, popular | Weak JS/TS analysis, need separate frontend |
| Go | Fast CLI, single binary | Limited code analysis ecosystem |
| Rust | Best performance | Highest dev cost, smallest ecosystem |

**Rationale**:

```
1. Code Analysis Ecosystem Match
   ├── madge: The de-facto standard for JS/TS dependency analysis
   ├── ts-morph: Powerful TypeScript AST manipulation
   ├── dependency-cruiser: Dependency rule checking
   └── These tools are all written in JS/TS, integration is natural

2. Isomorphic Advantage (Frontend + Backend)
   ├── CLI written in TypeScript
   ├── Web visualization also in TypeScript
   ├── Share type definitions (Graph, Node, Edge)
   └── No need to maintain two type systems

3. Target User Match
   ├── GID primarily serves Web/Frontend/Node.js projects first
   ├── These developers already use TypeScript
   └── Lowest barrier to adoption

4. AI SDK Support
   └── @anthropic-ai/sdk is well-maintained and TypeScript-native
```

### 10.2 CLI Framework: Commander.js

**Decision**: Use Commander.js + @inquirer/prompts for CLI.

**Alternatives Considered**:

| Framework | Pros | Cons |
|-----------|------|------|
| Commander.js | Simple, largest community, great docs | Needs inquirer for interactive |
| yargs | Feature-rich | Complex API |
| oclif | Enterprise-grade, plugin system | Over-engineered for our needs |
| clipanion | Type-safe | Small community |

**Rationale**:

```
1. Simplicity
   └── gid commands are not complex; we don't need oclif's plugin system

2. Largest Ecosystem
   ├── 100M+ weekly npm downloads
   ├── Easy to find answers to problems
   └── Long-term maintenance guaranteed

3. Classic Combination
   ├── Commander.js for command parsing
   ├── @inquirer/prompts for interactive design sessions
   └── This is the proven combination for Node.js CLIs
```

### 10.3 Visualization: D3.js + Mermaid

**Decision**: Use D3.js for web interface, Mermaid for CLI output.

**Alternatives Considered**:

| Library | Best For | Limitations |
|---------|----------|-------------|
| D3.js | High customization, complex interaction | Steeper learning curve |
| Mermaid | Documentation, quick diagrams | Limited interaction, no large graphs |
| Cytoscape.js | Graph analysis, bioinformatics | Overkill for simple cases |
| vis.js | Network graphs, timelines | Less customizable |

**Rationale**:

```
┌─────────────────────────────────────────────────────────────────┐
│  GID Visualization Needs                  D3.js Capability      │
├─────────────────────────────────────────────────────────────────┤
│  Click node to see details                ✓ Event binding       │
│  Highlight dependency paths               ✓ Dynamic styling     │
│  Filter by Feature/Layer                  ✓ Data-driven update  │
│  Zoom/pan large graphs                    ✓ d3-zoom             │
│  Hierarchical layout (Feature→Component) ✓ d3-hierarchy        │
│  Force-directed layout (dependency net)  ✓ d3-force            │
│  Custom node styles (colors by type)     ✓ Full control        │
└─────────────────────────────────────────────────────────────────┘

Why both D3.js AND Mermaid:
├── CLI output: Mermaid (simple, copy-pasteable, works in GitHub/docs)
│   $ gid query deps OrderService --format mermaid
│
│   graph TD
│     OrderService --> UserService
│     OrderService --> PaymentService
│
└── Web interface: D3.js (full interactivity, filtering, large graphs)
```

### 10.4 Complete Technology Stack

```
GID Tool Suite - Technology Stack
═══════════════════════════════════════════════════════════════════

Runtime & Language
──────────────────────────────────────────────────────────────────
Runtime:              Node.js 20+ (LTS)
Language:             TypeScript 5+
Package Manager:      npm or pnpm

CLI Layer
──────────────────────────────────────────────────────────────────
Command Framework:    Commander.js
Interactive Prompts:  @inquirer/prompts
Terminal Styling:     chalk (colors), ora (spinners)
Output Formatting:    cli-table3 (tables)

Core Layer
──────────────────────────────────────────────────────────────────
YAML Parsing:         js-yaml
Schema Validation:    zod
Graph Data:           Custom implementation (Map + Array)
                      (Simple enough, no need for external graph lib)

Code Analysis Layer
──────────────────────────────────────────────────────────────────
JavaScript/TS:        madge (dependency extraction)
                      ts-morph (AST analysis, optional)
Python:               Subprocess call to pydeps
                      (Keep Python as optional runtime dependency)
Universal (LSP):      vscode-languageclient
                      (For future language support)

AI Layer
──────────────────────────────────────────────────────────────────
Claude API:           @anthropic-ai/sdk (official SDK)
Prompt Management:    Custom template system (Handlebars or simple string)

Web Visualization Layer
──────────────────────────────────────────────────────────────────
Server:               Express (simple) or Fastify (faster)
Frontend:             Vanilla JS + D3.js (no heavy framework needed)
Build Tool:           Vite (dev) / esbuild (production bundle)
Diagram Export:       Mermaid (for static diagrams)

Testing
──────────────────────────────────────────────────────────────────
Unit Tests:           Vitest (fast, TypeScript-native)
E2E Tests:            Playwright (optional, for web UI)

Build & Distribution
──────────────────────────────────────────────────────────────────
Bundler:              tsup (simple CLI bundling)
Package:              npm publish
Binary (optional):    pkg or bun compile (for standalone binary)
```

### 10.5 Decision Records (ADRs)

Following GID's Decision node pattern, key architectural decisions:

```yaml
decisions:
  ADR-001-Language:
    title: "Use TypeScript as primary language"
    status: accepted
    context: "Need to build CLI + Web tool for code analysis"
    decision: "TypeScript for both CLI and Web"
    consequences:
      - "Can share types between CLI and Web"
      - "Best integration with JS/TS analysis tools"
      - "Requires Node.js runtime"

  ADR-002-NoGraphDB:
    title: "Use YAML files instead of graph database"
    status: accepted
    context: "Need to store dependency graphs"
    decision: "Start with YAML, add SQLite only if needed"
    consequences:
      - "Human-readable, git-friendly"
      - "Multi-hop queries require code (not native)"
      - "Sufficient for <1000 nodes"

  ADR-003-DualVisualization:
    title: "Use both Mermaid and D3.js for visualization"
    status: accepted
    context: "Need graph visualization for CLI and Web"
    decision: "Mermaid for CLI/docs, D3.js for web interface"
    consequences:
      - "CLI output is portable (paste into GitHub)"
      - "Web has full interactivity"
      - "Two rendering paths to maintain"

  ADR-004-PluginDeferral:
    title: "Defer plugin architecture to future version"
    status: accepted
    context: "Should we support custom extractors via plugins?"
    decision: "No plugin system in v1, hardcode supported languages"
    consequences:
      - "Faster initial development"
      - "Less flexibility for users"
      - "Can add plugin system in v2 if needed"
```

---

## 11. Summary

### What You're Building

```
┌─────────────────────────────────────────────────────────────────┐
│                    GID Tool Suite                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. GENERATE the graph                                           │
│     ├── Top-down: Idea → AI decompose → Interactive refine       │
│     └── Bottom-up: Code → Extract → AI enrich → Human verify     │
│                                                                  │
│  2. VISUALIZE and TRACK                                          │
│     ├── Interactive web-based graph view                         │
│     └── Progress dashboard (which nodes are done?)               │
│                                                                  │
│  3. QUERY for development tasks                                  │
│     ├── Impact: What breaks if I change X?                       │
│     ├── Feature: What do I need to implement Y?                  │
│     └── Debug: What's the common cause of A and B?               │
│                                                                  │
│  4. EVALUATE and IMPROVE                                         │
│     ├── Health check: Cycles? Orphans? High coupling?            │
│     └── Evolution: As-Is → To-Be → Refactoring tasks             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### This Document Uses GID

This design document itself follows GID methodology:
- Section 2: Defined Features first (domain layer)
- Section 3: Designed Components (design layer)
- Section 4: Created the actual graph.yml
- Section 5: Detailed each Feature
- Section 6: Implementation phases follow dependency order
- Section 7: Technical implementation details (types, errors, AI)
- Section 8: Refined MVP to minimum viable scope
- Section 10: Recorded technology decisions (Decision nodes)

---

*Document Version: 2.0*
*Last Updated: 2025-01-24*
