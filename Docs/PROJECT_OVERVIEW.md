# 复杂任务筹划可视化推理系统 — 项目功能文档

> 硕士论文展示系统 | 答辩时间：2026-03-30

---

## 一、项目概述

本系统是一个**纯前端 Mock 演示系统**，用于可视化展示"基于大小模型协同的复杂任务筹划推理框架"的完整推理过程。以法律适用推理为主场景，通过动画驱动的双层工作流，直观呈现从问题输入到最终法律意见生成的全链路推理过程。

---

## 二、技术栈

| 层次 | 技术 | 版本 | 用途 |
|------|------|------|------|
| 框架 | Vue 3 (Composition API) | ^3.5.30 | 核心 UI 框架 |
| 构建 | Vite | ^8.0.0 | 开发服务器 + 构建工具 |
| 状态管理 | Pinia | ^3.0.4 | 全局状态（推理状态、任务队列） |
| 路由 | Vue Router | ^4.6.4 | SPA 路由管理 |
| UI 组件库 | Naive UI | ^2.44.1 | 基础 UI 组件 |
| 图标 | @vicons/ionicons5 | ^0.13.0 | 图标库 |
| 流程图 | LogicFlow | ^2.1.11 | 工作流节点图（扩展备用） |
| 动画 | GSAP | ^3.14.2 | 高性能动画 |
| 数据策略 | 全量 Mock | — | 无后端，所有数据来自 mock/ 目录 |
| 主题 | 深色科幻风 | — | CSS 变量驱动，variables.css |

---

## 三、页面路由结构

```
/                    → 首页（HomeView）
/reasoning/:taskId   → 推理展示页（ReasoningView）
/data-management     → 数据管理页（DataManagementView）
/system-config       → 系统配置页（SystemConfigView）
```

---

## 四、功能架构总览

```
复杂任务筹划可视化推理系统
├── 首页（任务入口）
│   ├── 场景选择（法律适用推理 / 军事推演）
│   ├── 示例案例一键填充
│   ├── 主问题输入框
│   ├── 多主问题检测与任务队列
│   └── 开始推理 / 加入队列
│
├── 推理展示页（核心功能）
│   ├── 左侧：任务概览面板
│   ├── 中间：双层工作流
│   │   ├── 上层：主工作流（M1~M6 子问题迭代总览）
│   │   └── 下层：子工作流（S1~S8 当前子问题内部闭环）
│   ├── 右侧：节点详情面板
│   ├── 最终结论面板
│   └── 异常决策弹窗
│
├── 数据管理页（RAG 平台数据资产）
│   ├── 数据源管理（P0）
│   ├── 领域数据库（P0）
│   │   └── 实体流行度知识库（内部 Tab）
│   ├── 历史数据（P1）
│   ├── 检索数据（P1）
│   └── 索引与持久化（P2，规划中）
│
└── 系统配置页
    ├── 方法原理说明（主/子工作流 + 核心创新点）
    └── 推理参数配置（检索、实体流行度、状态判别、补检循环）
```

---

## 五、各页面功能详解

### 5.1 首页（HomeView）

**文件**：[`frontend/src/views/HomeView.vue`](frontend/src/views/HomeView.vue)

**功能**：
- **场景选择**：法律适用推理（主场景）、军事推演（扩展场景），卡片式选择
- **示例案例**：预置"合同违约损害赔偿纠纷"案例，点击一键填充输入框
- **主问题输入**：textarea 输入，支持自由输入或示例填充
- **多主问题检测**：自动检测输入中是否包含多个主问题（按序号/换行分割），检测到时显示警告提示并列出拆分结果
- **任务队列展示**：若已有队列任务，展示队列状态（排队中/推理中/已完成）
- **操作**：单问题直接跳转推理页；多问题加入任务队列后跳转

**实现原理**：
- 使用 `useTaskQueueStore` 管理多任务队列
- `detectMultipleQuestions()` 通过正则检测序号模式（`1.`、`①`、`问题一`等）或多行长文本
- Vue Router `push('/reasoning/:taskId')` 跳转

---

### 5.2 推理展示页（ReasoningView）

**文件**：[`frontend/src/views/ReasoningView.vue`](frontend/src/views/ReasoningView.vue)

**布局**：三栏 Grid（220px 左侧 | 1fr 中间 | 340px 右侧）

#### 5.2.1 任务概览面板（TaskOverview）

**文件**：[`frontend/src/components/reasoning/TaskOverview.vue`](frontend/src/components/reasoning/TaskOverview.vue)

- 展示任务名称、场景标签、任务状态
- 显示子问题完成进度（已完成数 / 总数）
- 展示当前迭代轮次
- "开始推理"按钮触发 `store.simulateRun()`

#### 5.2.2 主工作流 — 子问题迭代总览（SubQuestionTrack）

**文件**：[`frontend/src/components/reasoning/SubQuestionTrack.vue`](frontend/src/components/reasoning/SubQuestionTrack.vue)

- 横向展示 M1（任务分解）→ 各子问题节点 → M6（最终综合）的完整主链路
- 子问题节点逐步显式（前一个完成后才出现下一个），使用 `TransitionGroup` 动画
- 子问题节点显示状态（pending/running/completed）、答案摘要
- 点击子问题节点切换当前活跃子问题
- 子问题间连线（connector）随状态点亮（`lit` 类）
- 最后一个子问题到 M6 的连线在 `finalAnswer` 出现后显示

#### 5.2.3 子工作流 — 当前子问题内部闭环（SubWorkflow）

**文件**：[`frontend/src/components/reasoning/SubWorkflow.vue`](frontend/src/components/reasoning/SubWorkflow.vue)

- 展示当前活跃子问题的 S1~S8 节点流程
- 节点状态：pending → processing（脉冲动画）→ completed / insufficient
- 节点间连线随完成状态点亮
- 点击节点触发 `store.selectNode(nodeId)`，右侧面板同步更新

**S1~S8 节点说明**：

| 节点 | 名称 | 功能 |
|------|------|------|
| S1 | 是否检索判定 | 基于实体流行度阈值（0.55）决定是否触发外部检索 |
| S2 | 初始检索 | BM25+Dense 混合检索，召回候选文档 |
| S3 | 文档重排序 | CrossEncoder 精排，过滤低相关性文档 |
| S4 | 文档级状态判别 | 规则层：支撑分数阈值判定充分/不确定/不足 |
| S5 | 状态路由 | 根据聚合状态路由至证据精炼/查询重写/双路并行 |
| S6 | 查询重写 | 从争点聚焦、法条约束、事实补全三视角生成补检查询 |
| S7 | 集合级充分性判断 | LLM 语义层判断证据集合是否完备，不足则触发补检 |
| S8 | 子问题答案生成 | 基于充分证据生成子问题答案，写入历史轨迹 |

#### 5.2.4 节点详情面板（NodeDetailPanel）

**文件**：[`frontend/src/components/reasoning/NodeDetailPanel.vue`](frontend/src/components/reasoning/NodeDetailPanel.vue)

按节点类型渲染不同详情内容：

| 节点类型 | 展示内容 |
|----------|----------|
| `retrieval-decision` | 实体列表、流行度分数、检索决策结果 |
| `initial-retrieval` | 检索候选文档列表 |
| `reranking` | 重排前/后文档对比 |
| `doc-state-classifier` | 每篇文档的状态判别结果（充分/不确定/不足） |
| `state-router` | 路由决策说明 |
| `query-rewrite` | 三视角重写查询列表 |
| `sufficiency-check` | 充分性判断结果，不足时展示缺口分析 |
| `answer-generation` | 生成的子问题答案 |

#### 5.2.5 最终结论面板（FinalConclusionPanel）

**文件**：[`frontend/src/components/reasoning/FinalConclusionPanel.vue`](frontend/src/components/reasoning/FinalConclusionPanel.vue)

- 在所有子问题完成后显示（`v-if="store.finalAnswer"`）
- 展示子问题推理链（subQuestionChain）
- 展示关键证据摘要（keyEvidenceSummary）
- 展示最终法律意见

#### 5.2.6 异常决策弹窗（ExceptionDecisionModal）

**文件**：[`frontend/src/components/reasoning/ExceptionDecisionModal.vue`](frontend/src/components/reasoning/ExceptionDecisionModal.vue)

- S7 充分性判断不足时弹出
- 展示异常类型（insufficient）、触发节点、所属子问题
- 用户确认后关闭，推理继续（补检轮次 +1）

---

### 5.3 推理状态管理（ReasoningStore）

**文件**：[`frontend/src/stores/reasoning.js`](frontend/src/stores/reasoning.js)

**核心状态**：

| 状态 | 类型 | 说明 |
|------|------|------|
| `task` | ref | 当前任务信息 |
| `subQuestions` | ref | 全量子问题列表 |
| `visibleSubQuestions` | ref | 已显式的子问题（延迟出现） |
| `finalAnswer` | ref | 最终综合答案 |
| `isRunning` | ref | 推理运行状态 |
| `activeSubQuestionId` | ref | 当前选中子问题 ID |
| `subNodesMap` | ref | sqId → nodes[] 映射 |
| `activeNodeId` | ref | 当前选中节点 ID |
| `exceptionModal` | ref | 异常弹窗状态 |

**核心方法**：

- `initTask(taskId)`：初始化任务，预加载所有子问题节点
- `simulateRun()`：异步模拟推理全流程，逐子问题推进
- `runSubQuestionNodes(sq, nodes)`：逐节点执行，处理 insufficient 异常
- `waitForExceptionClose()`：轮询等待弹窗关闭（Promise + setInterval）
- `selectSubQuestion(sqId)` / `selectNode(nodeId)`：交互选中

**关键设计**：
- 子问题延迟显式：第 n 个完成后才将第 n+1 个推入 `visibleSubQuestions`
- `insufficient` 节点状态保持：通过 `continue` 跳过 `node.status = 'completed'`，避免状态覆盖
- 随机延迟（700~1200ms）模拟真实推理耗时

---

### 5.4 数据管理页（DataManagementView）

**文件**：[`frontend/src/views/DataManagementView.vue`](frontend/src/views/DataManagementView.vue)

**布局**：左侧导航（200px）+ 右侧内容区

**左侧导航模块**：

| 模块 ID | 名称 | 优先级 |
|---------|------|--------|
| datasource | 数据源管理 | P0 |
| domain | 领域数据库 | P0 |
| history | 历史数据 | P1 |
| retrieval | 检索数据 | P1 |
| index | 索引与持久化 | P2（规划中） |

**底部汇总指标**：总文档数、活跃数据源数、索引覆盖率（来自 `MOCK_DM_METRICS`）

#### 5.4.1 数据源管理（DataSourcePanel）

**文件**：[`frontend/src/components/data/DataSourcePanel.vue`](frontend/src/components/data/DataSourcePanel.vue)

- 展示多个数据源（法条库、判例库等）的连接状态、文档数量、最后同步时间
- 数据来自 `MOCK_DATA_SOURCES`

#### 5.4.2 领域数据库（DomainDbPanel）

**文件**：[`frontend/src/components/data/DomainDbPanel.vue`](frontend/src/components/data/DomainDbPanel.vue)

内部 Tab 结构：
- **实体流行度知识库**：展示法律实体（罪名、法条、概念等）的流行度分数、等级、变更历史；支持搜索、类型筛选；数据来自 `MOCK_ENTITIES`
- 其他领域数据库管理 Tab（术语映射、法律表格等）

#### 5.4.3 历史数据（HistoryPanel）

**文件**：[`frontend/src/components/data/HistoryPanel.vue`](frontend/src/components/data/HistoryPanel.vue)

- 展示历史推理记录，数据来自 `MOCK_HISTORY_RECORDS`

#### 5.4.4 检索数据（RetrievalPanel）

**文件**：[`frontend/src/components/data/RetrievalPanel.vue`](frontend/src/components/data/RetrievalPanel.vue)

- 展示检索日志记录，数据来自 `MOCK_RETRIEVAL_RECORDS`

---

### 5.5 系统配置页（SystemConfigView）

**文件**：[`frontend/src/views/SystemConfigView.vue`](frontend/src/views/SystemConfigView.vue)

**Tab 1：方法原理说明**

- 系统整体介绍（大小模型协同、主子双层工作流）
- 主工作流卡片流（M1→M6），每张卡片含节点 ID、名称、功能描述
- 子工作流卡片流（S1→S8），同上
- 核心创新点 4 项：
  1. 实体流行度感知检索（S1）
  2. 多视角查询重写（S6）
  3. 双层充分性判断（S4+S7）
  4. 历史轨迹传递（M4/S8）

**Tab 2：推理参数配置**

| 参数组 | 参数 | 默认值 | 说明 |
|--------|------|--------|------|
| 检索配置 | 检索器 | BM25+Dense | 混合检索策略 |
| 检索配置 | 重排序器 | CrossEncoder | 精排模型 |
| 检索配置 | Top-K | 5 | 保留文档数 |
| 检索配置 | 重排过滤阈值 | 0.6 | 低于此值过滤 |
| 实体流行度（S1） | 检索触发阈值 | 0.55 | 低于此值触发外部检索 |
| 文档状态判别（S4） | 充分上阈值 | 0.80 | ≥此值判定充分 |
| 文档状态判别（S4） | 不足下阈值 | 0.50 | ≤此值判定不足 |
| 补检循环（S6/S7） | 最大补检轮次 | 3 | 超出后强制进入答案生成 |
| 补检循环（S6/S7） | 大语言模型 | GPT-4o | S7/S8/M6 使用 |

阈值可视化：实体流行度阈值以进度条形式展示触发区间与跳过区间。

---

## 六、Mock 数据结构

### 6.1 推理 Mock（`frontend/src/mock/reasoning.js`）

- `MOCK_TASK`：任务基本信息（名称、场景、状态）
- `MOCK_SUB_QUESTIONS`：3 个子问题（subQuestionId、text、order、status）
- `SQ1_NODES` / `SQ2_NODES` / `SQ3_NODES`：每个子问题的 S1~S8 节点数组，含完整 `detail` 字段
- `MOCK_FINAL_ANSWER`：最终综合答案（subQuestionChain、keyEvidenceSummary、最终意见）
- `getMockSubNodes(sqId)`：按子问题 ID 返回对应节点数组

### 6.2 数据管理 Mock（`frontend/src/mock/dataManagement.js`）

- `MOCK_DATA_SOURCES`：5 个数据源配置
- `MOCK_ENTITIES`：12 个法律实体，含 popularityScore、popularityLevel、changeHistory
- `ENTITY_TYPE_CONFIG` / `ENTITY_STATUS_CONFIG`：实体类型/状态枚举配置
- `MOCK_TERM_MAPPINGS`：术语映射记录
- `MOCK_LEGAL_TABLES`：法律表格记录
- `MOCK_HISTORY_RECORDS`：历史推理记录
- `MOCK_RETRIEVAL_RECORDS`：检索日志记录
- `MOCK_DM_METRICS`：汇总指标（totalDocs、activeSources、indexedRatio）

---

## 七、核心算法与推理流程

### 7.1 主工作流（M1→M6）

```
M1 原始问题进入
  ↓
M2 子问题生成（大模型分解）
  ↓
M3 子问题求解调用（循环）
  ↓ ← ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
M4 轨迹更新（写入历史）           │
  ↓                               │
M5 停止判断 ──── 未完成 ──────────┘
  ↓ 已完成
M6 最终综合生成
```

### 7.2 子工作流（S1→S8，每个子问题内部）

```
S1 实体流行度判定
  ├── 流行度 ≥ 0.55 → 跳过检索，直接 S8
  └── 流行度 < 0.55 → S2
      ↓
S2 初始检索（BM25+Dense）
  ↓
S3 文档重排序（CrossEncoder）
  ↓
S4 文档级状态判别（规则层）
  ↓
S5 状态路由
  ├── 充分 → 证据精炼 → S7
  ├── 不足 → S6 查询重写 → 补检 → S3
  └── 不确定 → 双路并行
      ↓
S7 集合级充分性判断（LLM 语义层）
  ├── 充分 → S8
  └── 不足（且未超最大轮次）→ S6 补检循环
      ↓
S8 子问题答案生成
```

### 7.3 实体流行度感知检索（核心创新）

- 每个子问题的关键实体在知识库中有 `popularityScore`（0~1）
- S1 节点对比阈值（默认 0.55）：低频长尾实体（分数低）触发外部检索，高频概念（分数高）跳过检索
- 避免对常见法律概念的冗余检索，提升效率

### 7.4 双层充分性判断（核心创新）

- **S4 规则层**：快速、低成本，基于支撑分数阈值（上 0.80 / 下 0.50）对每篇文档独立判别
- **S7 LLM 语义层**：深度、高成本，对整个证据集合进行语义完备性判断
- 两层独立运行，互为补充，兼顾效率与准确性

---

## 八、视觉设计规范

- **主题**：深色科幻风，背景 `#0a0e1a`，主色 `#4facfe`（蓝）
- **CSS 变量**：统一在 [`frontend/src/styles/variables.css`](frontend/src/styles/variables.css) 定义
- **Glass 效果**：`.glass-panel` 类，`backdrop-filter: blur` + 半透明背景
- **节点状态色**：
  - pending：灰色 `#4a5568`
  - processing：蓝色脉冲动画
  - completed：绿色 `#10b981`
  - insufficient：橙色 `#f59e0b`
- **连线点亮**：`.conn-line.lit` 使用主色渐变
- **过渡动画**：Vue `<Transition>` + `<TransitionGroup>`，`sq-appear` 动画类

---

## 九、文件目录结构

```
frontend/src/
├── views/
│   ├── HomeView.vue              # 首页
│   ├── ReasoningView.vue         # 推理展示页
│   ├── DataManagementView.vue    # 数据管理页
│   └── SystemConfigView.vue      # 系统配置页
├── components/
│   ├── common/
│   │   ├── AppHeader.vue         # 顶部导航栏
│   │   └── AppLayout.vue         # 全局布局
│   ├── reasoning/
│   │   ├── TaskOverview.vue      # 任务概览
│   │   ├── SubQuestionTrack.vue  # 主工作流轨迹
│   │   ├── SubWorkflow.vue       # 子工作流节点流
│   │   ├── NodeDetailPanel.vue   # 节点详情面板
│   │   ├── FinalConclusionPanel.vue  # 最终结论
│   │   └── ExceptionDecisionModal.vue  # 异常弹窗
│   └── data/
│       ├── DataSourcePanel.vue   # 数据源管理
│       ├── DomainDbPanel.vue     # 领域数据库（含实体流行度）
│       ├── HistoryPanel.vue      # 历史数据
│       └── RetrievalPanel.vue    # 检索数据
├── stores/
│   ├── reasoning.js              # 推理状态管理
│   ├── taskQueue.js              # 任务队列管理
│   └── config.js                 # 系统配置状态
├── mock/
│   ├── reasoning.js              # 推理 Mock 数据
│   └── dataManagement.js         # 数据管理 Mock 数据
├── router/
│   └── index.js                  # 路由配置
└── styles/
    ├── variables.css             # CSS 变量（主题色、间距等）
    └── global.css                # 全局样式
```

---

## 十、当前开发状态

| 模块 | 状态 | 说明 |
|------|------|------|
| 首页 | ✅ 完成 | 含多主问题检测与任务队列 |
| 推理展示页 | ✅ 完成 | 双层工作流 + 节点详情 + 异常弹窗 |
| 推理 Bug 修复 | ✅ 完成 | insufficient 状态覆盖 + M6 连线缺失 |
| 数据管理页 | ✅ 完成 | 5 个模块，实体流行度在 DomainDbPanel 内 |
| 系统配置页 | ✅ 完成 | 方法原理说明 + 推理参数配置 |
| 索引与持久化 | 🔲 规划中 | P2，占位页面已有 |
| 后端对接 | 🔲 不在范围 | 纯 Mock 演示系统 |
