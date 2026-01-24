# Graph-Indexed Development (GID)

## AI 时代的软件开发方法论

**作者: Toni Tang**
**首次发布: 2025年1月23日**
**许可证: AGPL-3.0**

---

## 一、这是什么

Graph-Indexed Development 是一套 AI 时代的软件开发方法论。

**核心思想**：把软件项目的所有组成部分——功能需求、组件、接口、数据模型、代码文件、测试——以及它们之间的关系，组织成一张**可查询的图**。这张图作为项目的 Single Source of Truth。

```
图 = 节点 + 边

节点: Feature, Component, Interface, Data, File, Test
边: implements, depends_on, calls, reads, writes, tested_by
```

### 这张图能做什么

| 类别 | 能力 |
|------|------|
| **开发** | Feature 到代码的清晰路径、精确影响分析、自动任务拆解 |
| **AI 协作** | AI 通过图理解系统、Agent 到 Sub-agent 任务分配 |
| **质量** | 自动生成测试计划、因果调试、风险识别 |
| **项目管理** | 时间估算、关键路径分析、资源分配 |
| **知识** | 架构显式化、新人入职、决策追溯 |

---

## 二、为什么需要它

### 当前的痛点

**影响分析靠人脑：**
```
场景：要修改 UserService 的接口

传统做法：开发者凭记忆、IDE 搜索、容易遗漏间接依赖，每次都要重复

Graph-Indexed：查询"谁 depends_on UserService？" → 立即得到完整列表
              跨包、跨服务都能查，AI 也能查
```

**方向单一（Bottom-up 多，Top-down 少）：**
```
现状：
├── 改代码 → 人工判断影响 ✓（常做，但依赖经验）
└── 新功能 → 自动分析要改什么 ✗（很难）

有了图：
├── Bottom-up：改了 UserService → 往上查 → 影响"下单"、"支付"等 Feature
└── Top-down：要做"导出订单" → 往下查 → 需要 OrderService、新增 ExportService
```

**知识在人脑里：**
```
现状："UserService 为什么调用 CacheService？" → 只有老员工知道
      关键人员离职 → 知识丢失

有了图：架构显式化、决策有记录、新人看图就懂
```

**AI Coding 的瓶颈：**
```
现状：AI 写代码快，但不理解系统全貌
      生成的代码可能与现有架构冲突

有了图：AI 知道依赖关系，避免破坏现有功能
        可以拆解任务给多个 sub-agent 并行执行
```

---

## 三、核心概念

### 3.1 节点类型

| 类型 | 描述 | 示例 |
|------|------|------|
| **Feature** | 用户可感知的功能 | 用户注册、订单支付 |
| **Component** | 模块/服务/类 | UserService, OrderService |
| **Interface** | API/协议 | POST /api/users, gRPC 服务 |
| **Data** | 数据模型 | User, Order |
| **File** | 代码文件 | user_service.py |
| **Test** | 测试用例 | test_user_registration |

### 3.2 边类型

| 类型 | 方向 | 描述 |
|------|------|------|
| **implements** | Component → Feature | 组件实现功能 |
| **depends_on** | Component → Component | A 依赖 B |
| **calls** | Component → Interface | 调用接口 |
| **reads/writes** | Component → Data | 读写数据 |
| **tested_by** | Component → Test | 被测试覆盖 |

### 3.3 方向与因果

```
A ──depends_on──▶ B

含义：
├── A 依赖 B
├── B 变化，A 可能受影响
└── A 出问题，B 可能是原因

正向查询："B 变了谁受影响？" → 影响分析
反向查询："A 出问题可能是什么原因？" → 根因分析
```

---

## 四、基本用法

### 4.1 最小可用图

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

### 4.2 核心查询

| 查询 | 用途 |
|------|------|
| `getDependents(UserService)` | 改 UserService 会影响什么？ |
| `getDependencies(OrderService)` | OrderService 依赖什么？ |
| `getImpactedFeatures(UserService)` | 影响哪些 Feature？ |
| `findCommonCause(A, B)` | A 和 B 都出问题——共同依赖？ |

---

## 五、两种生成方式

### 新项目（Top-down）
```
想法 → Feature 文档 → Component 设计 → 建图 → 写代码
图在代码之前存在
```

### 现有项目（Bottom-up）
```
代码 → 自动提取 → AI 推断 → 人工补充 → 完整图
图从代码反向生成
```

---

## 六、使用场景

### 影响分析
```
输入：要修改 UserService 的接口
输出：会影响 OrderService, PaymentService, NotificationService
      涉及 Feature "用户下单"、"支付结算"
      需要更新 3 个单元测试和 2 个集成测试
```

### 根因分析
```
输入：OrderService 和 PaymentService 都很慢
输出：它们共同依赖 DatabaseService
      可能是数据库问题——只需检查一处
```

### 开发规划
```
输入：需要 Feature "用户导出订单历史"
输出：创建 ExportService
      依赖 OrderService, StorageService
      3 个任务，2 个可并行
```

### AI 任务分配
```
输入：图 + Feature "添加收货地址管理"
输出：AI 拆解为：
      - Sub-agent 1：扩展 UserService
      - Sub-agent 2：修改 OrderService
      - Sub-agent 3：更新测试
```

---

## 七、渐进采用

| 级别 | 做什么 | 时间 |
|------|--------|------|
| L0 | 理解概念 | 1 小时 |
| L1 | 创建 graph.yml，手动维护 | 2-4 小时 |
| L2 | 写查询函数，做影响分析 | 半天 |
| L3 | 用工具自动提取 | 1-2 天 |
| L4 | AI 辅助推断 | 按需 |

**从 L1 开始**：5-10 个节点就足够有用。

---

## 八、工具

### GID CLI

命令行工具：

```bash
npm install -g gid-cli

gid init          # 初始化图
gid extract .     # 从代码自动提取（TS/JS）
gid check         # 验证图完整性
gid query impact UserService   # 影响分析
gid serve         # 网页可视化
```

完整文档：[github.com/tonioyeme/graph-indexed-development-gid-cli](https://github.com/tonioyeme/graph-indexed-development-gid-cli)

---

## 九、与现有知识的关系

| 概念 | 借鉴内容 |
|------|----------|
| **图论** | 节点、边、遍历、环检测 |
| **因果推断** | Common Cause, Common Effect, Chain |
| **知识图谱** | 有类型的节点/边，可查询 |
| **C4 模型** | 抽象层次（简化版） |
| **Clean Architecture** | 层级方向、依赖规则 |
| **DDD** | Core/Supporting/Generic 优先级 |

**创新点**：将这些概念组合用于软件工程，支持双向查询、自动生成、对 AI 友好。

---

## 十、总结

```
1. 软件项目需要一张 Map（Single Source of Truth）
2. 用图结构表示（节点 + 边 + 方向）
3. 支持双向分析（Top-down + Bottom-up）
4. 两种生成方式（新项目 vs 现有项目）
5. 渐进采用（从 YAML + 核心类型开始）
```

**核心理念**：先建图，再写码。

---

## 了解更多

- **GID CLI 工具**: [github.com/tonioyeme/graph-indexed-development-gid-cli](https://github.com/tonioyeme/graph-indexed-development-gid-cli)
- **完整文档**: GID Pro 提供
- **商业咨询**: toni@gid.dev

---

*文档版本: 1.0（公开版）*
*最后更新: 2025-01-24*
