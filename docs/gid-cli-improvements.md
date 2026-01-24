# GID CLI 改进计划

本文档记录了在实际项目（ideaSpark）上测试 `gid` 工具时发现的问题和改进建议。

## 测试环境

- 项目: ideaSpark (Next.js + TypeScript 项目)
- 项目结构: `app/`, `lib/`, `components/` 等目录
- 构建产物: `.next/` 目录

---

## 核心概念：两层图模型

### 代码结构图 (Code Structure Graph)

由 `gid extract` 自动生成，反映代码的**物理结构**：

```yaml
# 节点类型: File
lib/supabase:
  type: File
  path: lib/supabase.ts

# 边: 基于 import/require 语句
- from: app/page
  to: lib/supabase
  relation: depends_on
```

**特点**:
- ✅ 自动生成，无需人工
- ✅ 100% 准确反映代码依赖
- ❌ 无语义抽象
- ❌ 无层级概念
- ❌ 无 Feature 映射

### 语义图 (Semantic Graph)

由人工或 AI 辅助创建，反映系统的**逻辑架构**：

```yaml
# 节点类型: Feature (用户视角)
IdeaCapture:
  type: Feature
  description: "用户可以快速捕捉和保存想法"
  priority: core

# 节点类型: Component (开发视角)
SupabaseClient:
  type: Component
  layer: infrastructure    # 层级概念
  path: lib/supabase.ts

# 边: implements (Feature ← Component)
- from: MainPage
  to: IdeaCapture
  relation: implements
```

**特点**:
- ✅ 有 Feature 层（用户可感知功能）
- ✅ 有 Component 层（开发模块）
- ✅ 有架构层级 (interface → application → domain → infrastructure)
- ✅ 支持层级违规检测
- ❌ 需要人工/AI 辅助创建

### 推荐工作流

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: 自动提取                                            │
│  gid extract . → 代码结构图 (File 级别)                       │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: AI/人工升级                                         │
│  - 将 File 归类为 Component                                  │
│  - 分配 layer (infrastructure/domain/application/interface) │
│  - 识别并添加 Feature 节点                                   │
│  - 添加 implements 边                                        │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: 验证与查询                                          │
│  gid check  → 健康检查                                       │
│  gid query  → 依赖分析、影响分析                              │
│  gid serve  → 可视化                                         │
└─────────────────────────────────────────────────────────────┘
```

### 分数差异分析

在 ideaSpark 项目测试中：

| 对比项 | 代码结构图 (提取) | 语义图 (手动) |
|--------|-------------------|---------------|
| 健康分数 | 87/100 | 95/100 |
| 节点类型 | File | Feature + Component |
| 孤立节点 | 4 (config 文件) | 0 |
| 层级信息 | 无 | 4 层架构 |

**分数差异原因**:
- 提取图包含了 config 文件 (next.config.js 等)，它们无依赖关系，成为孤立节点
- 语义图经过人工筛选，只包含有意义的组件

**改进方向**:
- `gid extract` 应默认忽略 `*.config.js` 等配置文件
- 提供 `gid upgrade` 命令，AI 辅助将代码结构图升级为语义图

---

## 手动建图过程记录 (gid upgrade 需求规格)

在 ideaSpark 项目测试中，由于 `gid extract` 当时有 bug，我们通过 AI 对话手动创建了语义图。过程如下：

### Step 1: 读取项目结构
```bash
# AI 执行
ls app/ lib/ components/
# 了解目录组织
```

### Step 2: 识别 Features (用户视角)
根据项目名称和功能推断：
```yaml
IdeaCapture:      # 核心功能
AIClassification: # AI 分类
AIStepGeneration: # 步骤生成
AIRefine:         # 优化想法
SemanticSearch:   # 语义搜索
StripePayment:    # 支付
InviteSystem:     # 邀请协作
```

### Step 3: 识别 Components 并分配 Layer
根据文件路径推断层级：
```yaml
# infrastructure (基础设施)
lib/supabase.ts    → SupabaseClient
lib/stripe.ts      → StripeClient
lib/ai/openai.ts   → OpenAIClient
lib/ai/claude.ts   → ClaudeClient
lib/ai/embedding.ts → EmbeddingService

# domain (业务逻辑)
lib/ai/prompts.ts  → AIPrompts
lib/categories.ts  → Categories

# application (API 层)
app/api/**/route.ts → *API

# interface (界面层)
app/page.tsx       → MainPage
components/*.tsx   → *Dialog/*Component
```

### Step 4: 建立 Edges
```yaml
# implements: Component → Feature
- from: MainPage
  to: IdeaCapture
  relation: implements

# depends_on: 基于代码 import 关系
- from: ClassifyAPI
  to: SupabaseClient
  relation: depends_on
```

### gid upgrade 应自动化的部分

```bash
gid upgrade graph-extracted.yml --ai --output graph-semantic.yml
```

AI 应自动完成：
1. **File → Component 映射**: 将文件节点归类为组件
2. **Layer 推断**: 根据路径模式分配架构层级
3. **Feature 识别**: 分析代码功能，提取用户可感知的 Feature
4. **implements 边生成**: 连接 Component 到 Feature
5. **描述生成**: 为每个节点生成有意义的 description

---

## 技术架构：提取器实现

### 当前实现

```
gid extract
    ↓
madge 库 (第三方)
    ↓
┌─────────────────────────────────────────┐
│  TypeScript → ts.createProgram (TS编译器)│
│  JavaScript → detective (正则匹配)       │
└─────────────────────────────────────────┘
    ↓
依赖树 → GID Graph (File 级别)
```

**特点**:
- ✅ 简单易用，依赖 madge 库
- ✅ 支持 tsconfig.json 路径别名
- ❌ 不使用 LSP (Language Server Protocol)
- ❌ 只支持 JS/TS

### 未来架构：多语言支持

```
gid extract --lang <language>
    ↓
┌─────────────────────────────────────────────────────────────┐
│  语言适配器 (Language Adapters)                              │
├─────────────────────────────────────────────────────────────┤
│  TypeScript │ Python   │ Go        │ Rust      │ Java      │
│  ─────────  │ ──────   │ ────      │ ─────     │ ─────     │
│  madge      │ pydeps   │ go-deps   │ cargo     │ jdeps     │
│  或 LSP     │ 或 LSP   │ 或 LSP    │ 或 LSP    │ 或 LSP    │
└─────────────────────────────────────────────────────────────┘
    ↓
统一的 GID Graph 格式
```

### LSP vs 专用工具 对比

| 方案 | 优点 | 缺点 |
|------|------|------|
| **专用工具** (madge/pydeps) | 简单、快速、无需运行服务 | 每种语言需要不同工具 |
| **LSP** (Language Server) | 统一接口、精确分析 | 需要启动服务、配置复杂 |
| **混合方案** | 兼顾两者优点 | 维护成本高 |

### 推荐方案：Smart Detection（智能检测）

```
┌─────────────────────────────────────────────────────────────┐
│  默认: 轻量工具 (madge)                                      │
│  ─────────────────────────                                  │
│  • 无需额外依赖                                              │
│  • 启动快                                                   │
│  • 覆盖 90% 场景                                            │
└─────────────────────────────────────────────────────────────┘
                    ↓ 检测到复杂场景
┌─────────────────────────────────────────────────────────────┐
│  提示用户: "检测到复杂项目，建议启用 LSP 获得更精确分析"        │
│  ─────────────────────────────────────────────────────────  │
│  • 用户选择: gid extract --lsp                              │
│  • 或配置: .gid/config.yml → lsp: true                     │
└─────────────────────────────────────────────────────────────┘
```

**复杂度判断标准：**

| 指标 | 阈值 | 说明 |
|------|------|------|
| 文件数量 | > 500 | 大型项目 |
| 路径别名 | > 5 个 | 复杂 tsconfig paths |
| Monorepo | 检测到 | 多 package workspace |
| 动态导入 | > 10% | `import()` 语法 |
| 循环依赖 | > 3 个 | 需要精确分析 |

**配置文件示例：**

```yaml
# .gid/config.yml
extract:
  lsp: auto  # auto | always | never
  lsp_threshold:
    files: 500
    path_aliases: 5
    dynamic_imports_percent: 10
```

**命令行用法：**

```bash
# 默认：自动检测，轻量优先
gid extract .

# 强制使用 LSP
gid extract . --lsp

# 强制不使用 LSP
gid extract . --no-lsp

# 查看检测结果
gid extract . --dry-run
# 输出:
# Project complexity: MEDIUM
# Files: 127, Path aliases: 3, Dynamic imports: 5%
# Recommendation: Using lightweight extractor (madge)
# Use --lsp to enable Language Server for more accurate analysis
```

**LSP 作为可选依赖：**

```bash
# LSP 不是必须的，按需安装
npm install -g typescript-language-server  # TypeScript
pip install python-lsp-server              # Python
go install golang.org/x/tools/gopls@latest # Go

# gid 检测已安装的 LSP
gid doctor
# 输出:
# ✓ madge (built-in)
# ✓ typescript-language-server (found)
# ✗ python-lsp-server (not found)
# ✗ gopls (not found)
```

### 各语言推荐实现

| 语言 | 推荐工具 | 备选 (LSP) |
|------|----------|-----------|
| TypeScript/JS | madge | tsserver |
| Python | pydeps / pigar | pylsp |
| Go | `go list` / guru | gopls |
| Rust | cargo-deps | rust-analyzer |
| Java | jdeps | jdtls |
| C/C++ | include-what-you-use | clangd |

---

## 发现的问题

### 1. `gid extract` 扫描了构建产物目录 ✅ 已修复

**问题描述**:
运行 `gid extract .` 时，工具扫描了 `.next` 目录，导致生成了大量无意义的节点（43 个孤立节点），健康分数降为 0/100。

**已修复**: 添加了 20 个默认忽略目录:
```typescript
const DEFAULT_IGNORE_DIRS = [
  'node_modules', '.next', '.nuxt', '.output', 'dist', 'build', 'out',
  '.git', 'coverage', '__pycache__', '.cache', '.turbo', '.vercel',
  '.netlify', '.parcel-cache', '.vite', '.svelte-kit', '.angular',
  'vendor', '.bundle'
];
```

---

### 2. TypeScript 导入识别不完整 ✅ 已修复

**问题描述**:
`gid extract` 没有正确识别 TypeScript 文件中的导入依赖。

**已修复**: 改进了 tsconfig.json 解析，支持检查 base dir 和 cwd，配合 madge 正确解析 ES modules 和路径别名。

---

### 3. 多目录扫描只处理第一个目录 ✅ 已修复

**问题描述**:
运行 `gid extract ./app ./lib ./components` 时，只处理了第一个目录。

**已修复**: 现在支持多目录扫描:
```bash
gid extract ./app ./lib ./components
```

---

### 4. 缺少 `--ignore` 选项 ✅ 已修复

**已修复**: 添加了以下选项:
```bash
gid extract . --ignore ".storybook" "*.test.ts"  # 自定义忽略
gid extract . --no-default-ignore                 # 禁用默认忽略
gid extract ignore-list                           # 查看默认忽略列表
```

---

### 5. `gid serve` 缺少 Graph 方法 ✅ 已修复

**已修复**: 在 `core/graph.ts` 中添加了 `getNodes()` 和 `getEdges()` 方法。

---

### 6. 健康分数计算需要优化 ✅ 已修复

**已修复**: 采用分级评分:
- Errors (循环依赖): -10 each
- Critical warnings (层级违规, 高耦合): -5 each
- Minor warnings (孤立节点, 缺少实现): -2 each
- Info: -1 each

---

## 优先级排序 (更新)

| 优先级 | 问题 | 状态 |
|--------|------|------|
| P0 | 默认忽略构建产物目录 | ✅ 已修复 |
| P0 | 修复 TypeScript 导入识别 | ✅ 已修复 |
| P0 | `gid serve` 缺少 Graph 方法 | ✅ 已修复 |
| P1 | 支持多目录扫描 | ✅ 已修复 |
| P1 | 添加 --ignore 选项 | ✅ 已修复 |
| P1 | `gid check --graph <file>` 指定图文件 | 待实现 |
| P2 | 优化健康分数计算 | ✅ 已修复 |
| P2 | 支持 tsconfig.json | ✅ 已修复 |
| P2 | `gid extract` 忽略 config 文件 | 待实现 |
| P2 | `gid extract --group` 支持 Component 抽象 | 待验证 |
| P3 | --dry-run 模式 | ✅ 已完成 |
| P3 | 增量提取 | ✅ 已完成 |
| P3 | 交互式模式 | ✅ 已完成 |
| P4 | 架构模式识别 | Phase 3 (AI) |

---

## P3 功能设计讨论

### 7. --dry-run 模式

**用途**: 预览提取结果，不实际修改文件。适用于 CI/CD 验证和脚本。

**设计**:
```bash
gid extract . --dry-run

Dry Run - No files will be modified
════════════════════════════════════════

Directories to scan:
  ✓ ./app
  ✓ ./lib
  ✓ ./components

Excluded directories (found):
  ⊘ .next
  ⊘ node_modules

Files to process: 45
  app/page.tsx
  app/layout.tsx
  lib/supabase.ts
  ...

Estimated nodes: 45
Estimated edges: ~120

Use without --dry-run to execute extraction.
```

**实现优先级**: 高 (简单, 高价值)

---

### 8. 增量提取 (--incremental)

**用途**: 只处理自上次提取以来变更的文件，提高大项目性能。

**状态跟踪设计**:
```yaml
# .gid/state.yml
lastExtraction:
  timestamp: 2026-01-24T10:00:00Z
  gitCommit: abc123    # 如果在 git repo 中
  fileHashes:          # 非 git 项目的回退方案
    app/page.tsx: sha256:a1b2c3...
    lib/supabase.ts: sha256:d4e5f6...
```

**版本历史设计**:
```
.gid/
├── graph.yml          # 当前版本
├── state.yml          # 跟踪信息
└── history/           # 保留最近 N 个版本 (默认: 3)
    ├── graph.2026-01-24T09.yml
    └── graph.2026-01-23T15.yml
```

**命令设计**:
```bash
gid extract --incremental      # 增量合并变更
gid history list               # 显示版本历史
gid history diff <version>     # 对比版本差异
gid history restore <version>  # 恢复到指定版本
```

**文件删除处理**:
- 从图谱中移除对应节点
- 警告用户哪些节点被移除

**Git 替代方案**: 高级用户可以使用:
```bash
gid extract $(git diff --name-only HEAD~1)
```

**实现优先级**: 中 (复杂, 高价值)

---

### 9. 交互式模式 (--interactive)

**用途**: 引导新用户完成首次提取，支持 top-down 和 bottom-up 两种流程。

**Bottom-up (extract) 流程**:
```bash
gid extract . --interactive

GID Interactive Extraction
════════════════════════════════════════

Scanning directory...

⚠ Detected build directories:
  • .next (Next.js build output)
  • dist (Build artifacts)

? Exclude these directories? (Y/n) Y
✓ Excluded: .next, dist

Found 45 source files in:
  • app/ (12 files)
  • lib/ (8 files)
  • components/ (25 files)

? Continue with extraction? (Y/n) Y

Extracting dependencies...
✓ 45 nodes created
✓ 120 edges found
⚠ 2 circular dependencies detected

? Save graph to .gid/graph.yml? (Y/n) Y
✓ Graph saved successfully
```

**Top-down (design) 流程**: 已在 `gid design` 中实现交互式 AI 设计。

**实现优先级**: 中 (中等复杂度, 用户体验提升)

---

## AI 集成讨论

### 10. 架构模式识别 (Phase 3)

**设计**: 混合模式 - 规则 + AI

**规则引擎** (用户可配置):
```yaml
# .gid/patterns.yml
patterns:
  clean-architecture:
    description: "Clean Architecture with 4 layers"
    detect:
      layers: [interface, application, domain, infrastructure]
      rules:
        - domain must not depend on infrastructure directly
        - interface must not depend on domain directly

  repository-pattern:
    description: "Repository Pattern for data access"
    detect:
      - name matches: "*Repository"
      - has dependency to: database|orm|supabase

  service-layer:
    description: "Service Layer for business logic"
    detect:
      - name matches: "*Service"
      - layer: application
```

**可检测的架构模式**:
| 模式 | 检测方式 |
|------|----------|
| Clean Architecture | 4 层结构 + 依赖方向 |
| MVC | Controllers + Models + Views 目录 |
| Hexagonal/Ports-Adapters | ports/, adapters/ 目录结构 |
| Repository Pattern | *Repository 类 + 数据库依赖 |
| Service Layer | *Service 类 |
| Event-Driven | EventEmitter, pub/sub 模式 |
| API Gateway | 单一入口 + 多服务路由 |
| Facade Pattern | 简化复杂子系统的接口 |
| Factory Pattern | *Factory 类 |

**可操作的输出**:
```bash
gid analyze patterns

Detected Patterns
════════════════════════════════════════

✓ Clean Architecture (4 layers detected)
  Compliance: 85%
  Issues:
    • lib/auth.ts bypasses service layer

✓ Repository Pattern
  Found: SupabaseClient, UserRepository

⚠ Partial Service Layer
  Coverage: 3/7 features have services
  Missing:
    • Dashboard feature → Consider DashboardService
    • Settings feature → Consider SettingsService

Suggestions:
1. [High] Move lib/auth.ts logic to AuthService
2. [Medium] Create services for remaining features
3. [Low] Consider extracting shared utils to infrastructure layer
```

**实现优先级**: 低 (Phase 3, 依赖 AI 集成)

---

### 11. 问题修复辅助工具 (gid fix) - Phase 3

**设计**:
```bash
gid fix                          # 交互式修复所有问题
gid fix --issue layer-violation  # 修复特定类型问题
gid fix --dry-run                # 预览修复方案
gid fix --auto                   # 自动应用安全修复
```

---

### 12. 智能判断 (gid check --ai) - Phase 3

**混合模式设计**:
```bash
gid check                        # 基础规则检查
gid check --ai                   # AI 增强分析
gid check --json | claude-code   # 管道到外部 AI
```

---

## 实现路线图

### Phase 1: Pure CLI ✅ 完成
- [x] 基础查询 (impact, deps, path, common-cause)
- [x] 图谱检查 (check)
- [x] 依赖提取 (extract)
- [x] Web 可视化 (serve)
- [x] AI 辅助设计 (design)

### Phase 2: CLI 增强 ✅ 完成
- [x] --dry-run 预览模式
- [x] --incremental 增量提取
- [x] --interactive 交互模式
- [x] gid history 版本管理

### Phase 3: AI 深度集成
- [ ] 架构模式识别 (analyze patterns)
- [ ] 智能修复建议 (fix --ai)
- [ ] 上下文感知检查 (check --ai)
- [ ] 自然语言查询

---

## 下一步行动

1. ~~修复 P0/P1/P2 问题~~ ✅ 完成
2. ~~实现 --dry-run 模式~~ ✅ 完成
3. ~~实现 --incremental 增量提取~~ ✅ 完成
4. ~~实现 --interactive 交互模式~~ ✅ 完成
5. 在更多真实项目上测试
6. 收集用户反馈
7. Phase 3: AI 深度集成

---

*最后更新: 2026-01-24*
