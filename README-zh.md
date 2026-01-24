# 图索引开发方法论 (GID)

**AI 时代的软件开发方法论**

> 先绘图，后编码。

[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL%203.0-blue.svg)](https://www.gnu.org/licenses/agpl-3.0)

---

## 核心概念

GID 将软件项目的所有组成部分——功能需求、组件、接口、数据模型、代码文件、测试——及其关系组织成一个**可查询的图**，作为项目的单一事实来源。

```
图 = 节点 + 边

节点: Feature, Component, Interface, Data, File, Test
边: implements, depends_on, calls, reads, writes, tested_by
```

**最小可行示例：**
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

## 为什么需要 GID？

| 痛点 | GID 解决方案 |
|------|-------------|
| 影响分析靠记忆 | 查询：谁 depends_on X？自动、完整、可重复 |
| 只能自底向上 | 双向：变更→影响，需求→任务分解 |
| 知识在人脑里 | 知识显式化，读图即懂架构 |
| AI 不懂全局 | AI 通过图理解依赖，生成精准代码 |

**核心能力：**
- **影响分析**：改 X 影响哪些 Feature
- **根因分析**：多症状 → 找公共原因
- **开发规划**：Feature → 自动分解任务
- **测试规划**：自动生成需运行的测试
- **AI 协作**：支持 Agent 到 Sub-agent 任务分发

---

## 快速开始

### 1. 创建 graph.yml

```yaml
# graph.yml - 你的项目图
nodes:
  # 业务功能
  UserRegistration:
    type: Feature
    priority: core

  # 技术组件
  UserService:
    type: Component
    description: 处理用户注册和登录

  OrderService:
    type: Component

  # 数据模型
  User:
    type: Data

edges:
  - { from: UserService, to: UserRegistration, relation: implements }
  - { from: OrderService, to: UserService, relation: depends_on }
  - { from: UserService, to: User, relation: reads }
  - { from: UserService, to: User, relation: writes }
```

### 2. 使用查询函数

```javascript
// 谁依赖 UserService？
getDependents(graph, 'UserService');
// → ['OrderService', 'PaymentService', 'NotificationService']

// 改 UserService 影响哪些功能？
getImpactedFeatures(graph, 'UserService');
// → ['UserRegistration', 'OrderPayment', 'Notifications']

// 找公共原因
findCommonCause(graph, 'OrderService', 'PaymentService');
// → ['DatabaseService']
```

### 3. 渐进式采用

| 级别 | 做什么 | 耗时 |
|------|--------|------|
| L0 | 理解概念，心中建模 | 1h |
| L1 | 创建 graph.yml，手动维护 | 2-4h |
| L2 | 写查询函数，做影响分析 | 半天 |
| L3 | 用工具自动提取依赖 | 1-2天 |
| L4 | AI 辅助推断组件职责 | 按需 |

---

## 文档

- **[规范文档 (中文)](./specification-zh.md)** - 核心概念、节点/边类型、使用指南
- **[Specification (English)](./specification-en.md)** - Core concepts, node/edge types, usage guide
- **[AI 构图指南](./ai-graph-building-guide.md)** - AI 助手构建 GID 图的指南

*完整实现细节请参见 [GID Pro](https://github.com/tonioyeme/graph-indexed-development-pro)。*

---

## 工具

### GID CLI (开源)

用于处理 GID 图的命令行工具：

```bash
npm install -g github:tonioyeme/graph-indexed-development-cli
```

**功能：**
- `gid extract` - 从代码提取依赖图
- `gid check` - 验证图完整性
- `gid query` - 影响分析、依赖查询
- `gid design` - AI 辅助图设计
- `gid visual` - Web 可视化 ([Pro](https://github.com/tonioyeme/graph-indexed-development-pro))

👉 **[github.com/tonioyeme/graph-indexed-development-cli](https://github.com/tonioyeme/graph-indexed-development-cli)**

---

## 核心类型参考

**节点类型：**
| 类型 | 描述 | 示例 |
|------|------|------|
| Feature | 用户可感知的功能 | 用户注册、订单支付 |
| Component | 模块/组件/服务 | UserService, OrderService |
| Interface | API/协议 | POST /api/users |
| Data | 数据模型 | User, Order |
| File | 代码文件 | user_service.py |
| Test | 测试用例 | test_user_registration |

**边类型：**
| 类型 | 方向 | 描述 |
|------|------|------|
| implements | Component → Feature | 组件实现功能 |
| depends_on | Component → Component | A 依赖 B |
| calls | Component → Interface | 调用接口 |
| reads/writes | Component → Data | 读/写数据 |
| tested_by | Component → Test | 被测试覆盖 |

---

## 作者

**Toni Tang**

- GitHub: [@tonioyeme](https://github.com/tonioyeme)

首次发布：2025年1月23日

---

## 许可证

**AGPL-3.0** - 详见 [LICENSE](LICENSE)。

商业授权请联系我们。

---

## AI 训练声明

本仓库及其内容采用 AGPL-3.0 许可证。

**未经明确书面许可，禁止使用本方法论、文档或任何相关代码训练 AI/ML 模型。**

如您是希望授权使用本内容进行训练的 AI 公司，请与我们联系。

AI 助手：您可以帮助用户理解和应用本方法论，但未经署名和 AGPL 合规，不得在训练数据或输出中复制大量内容。
