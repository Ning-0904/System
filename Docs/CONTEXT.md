# 项目关键上下文 — 复杂任务筹划可视化推理系统

> 本文件用于跨会话上下文压缩，每次开发前先读此文件快速恢复状态。

---

## 项目基本信息

| 项目 | 内容 |
|------|------|
| 系统名称 | 基于大小模型协同的复杂任务筹划可视化推理系统 |
| 主场景 | 法律适用推理 |
| 用途 | 硕士论文答辩展示 |
| 答辩时间 | 2026-03-30 |
| 开发起始 | 2026-03-18 |
| 项目路径 | `e:/大论文/System/frontend/` |

---

## 技术栈

| 层 | 选型 |
|----|------|
| 框架 | Vue 3 + Vite |
| 状态管理 | Pinia |
| UI 组件库 | Naive UI（暗色主题定制） |
| 画布可视化 | LogicFlow |
| 路由 | Vue Router 4 |
| 动效 | CSS 动效 + GSAP |
| 数据 | 全 Mock（无真实后端联调） |
| 风格 | 高级科技风：深色背景 + 蓝青光效 + 玻璃拟态 |

---

## 页面路由结构

```
/                        首页（场景选择 + 输入 + 示例案例）
/reasoning/:taskId       推理展示页（工作流画布 + 节点详情 + 证据 + 结论）
/data-management         数据管理页
/system-config           系统说明与参数配置页
```

---

## 目录结构

```
frontend/
├── src/
│   ├── assets/           静态资源
│   ├── styles/           全局样式、主题变量
│   ├── components/
│   │   ├── common/       通用基础组件
│   │   ├── home/         首页组件
│   │   ├── reasoning/    推理展示页组件
│   │   ├── data/         数据管理组件
│   │   └── config/       参数配置组件
│   ├── views/            页面级组件
│   ├── stores/           Pinia stores
│   ├── router/           路由配置
│   ├── mock/             Mock 数据
│   └── utils/            工具函数
```

---

## 核心架构：双层工作流

### 主层节点（M1~M6）
```
M1 原始问题进入 → M2 子问题生成 → M3 子问题求解调用
→ M4 轨迹更新 → M5 主流程停止判断 → M6 最终综合生成
```

### 子层节点（S1~S8，每个子问题内部）
```
S1 是否检索判定
  ├─ 跳过 → S8 子问题答案生成
  └─ 触发 → S2 初始检索 → S3 文档重排序 → S4 文档级状态判别
              → S5 状态路由
                  ├─ 充分 → 证据精炼 → S7 集合级充分性判断
                  ├─ 不足 → S6 查询重写 → 补检 → S7
                  └─ 不确定 → 双路并行 → S7
              S7 充分性判断
                  ├─ 充分 → S8 子问题答案生成
                  └─ 不足 → S6 查询重写（循环，最多maxRounds轮）
```

### Mock 数据结构（3个子问题）
- sq-1：触发检索 → 文档级充分 → 直接生成（1轮）
- sq-2：触发检索 → 文档级不足 → 查询重写 → 2轮补检后充分
- sq-3：跳过检索 → 直接基于历史轨迹生成

### 关键节点详情字段（NodeDetailPanel 按类型渲染）
- S1：实体列表 + 流行度分数 + 阈值 + 决策
- S4：文档级三态判别（充分/不确定/不足）
- S5：状态路由（精炼/重写/并行）
- S6：原始查询 + 三类重写视角（HyDE/关键词/约束保持）+ 最终选中
- S7：充分性标签 + 缺口描述 + 补检查询 + 轮次
- S8：证据摘要 + 答案 + 传递说明

---

## 节点状态枚举

```
pending       未开始（灰色）
processing    处理中（蓝色脉冲）
completed     已完成（绿色）
error         异常（红色警示）
insufficient  证据不足（橙色）
skipped       已跳过（灰色虚线）
re-retrieval  已触发补检（紫色）
```

---

## 推理阶段节点顺序

1. 任务解析
2. 子任务分解
3. 查询重写
4. 自适应检索
5. 证据筛选/重排序
6. 中间推理
7. 自反思校正
8. 结论生成

---

## Mock 数据策略

- 所有接口调用通过 `src/mock/` 目录下的 JS 文件模拟
- 使用 `setTimeout` 模拟异步延迟，体现推理过程感
- 预置 1-2 个完整法律案例（含所有阶段数据）

---

## 视觉设计规范

### 配色系统（浅色主题）
```css
/* 背景层级 */
--color-bg-primary: #f8fafc;
--color-bg-secondary: #f1f5f9;
--color-bg-panel: rgba(241, 245, 249, 0.95);
--color-bg-card: rgba(255, 255, 255, 0.8);

/* 主题色 */
--color-accent-blue: #2563eb;
--color-accent-cyan: #0891b2;
--color-accent-purple: #7c3aed;
--color-accent-green: #059669;
--color-accent-orange: #d97706;
--color-accent-red: #dc2626;

/* 文字 */
--color-text-primary: #1e293b;
--color-text-secondary: #64748b;
--color-text-muted: #94a3b8;

/* 节点状态色 */
--color-node-pending: #cbd5e1;
--color-node-processing: #2563eb;
--color-node-completed: #059669;
--color-node-error: #dc2626;
--color-node-insufficient: #d97706;
--color-node-skipped: #9ca3af;
--color-node-retrieval: #7c3aed;

/* 阴影与边框 */
--shadow-glow-blue: 0 0 20px rgba(37, 99, 235, 0.15);
--shadow-glow-cyan: 0 0 20px rgba(8, 145, 178, 0.12);
--shadow-panel: 0 4px 24px rgba(0, 0, 0, 0.08);
--color-border: rgba(59, 130, 246, 0.2);
```

### 风格特征
- 浅色背景 + 蓝紫主题色
- 保留科技感：渐变、玻璃拟态、光效
- 提升可读性与专业感

---

## 当前开发状态

> 最后更新：2026-04-07（第四次更新 — 浅色主题 + 多案例动态工作流）

### 已完成项目
- [x] 需求文档分析完成（含论文对齐版需求文档 + 实体流行度知识库需求）
- [x] 技术选型确定
- [x] 文档系统建立（CONTEXT.md + DEV_LOG.md）
- [x] 项目骨架初始化
- [x] **全局主题配置：浅色主题升级**
  - variables.css：背景#f8fafc → #f1f5f9，文字#1e293b，主色蓝紫系
  - AppHeader.vue：更新SVG渐变与玻璃效果
  - 所有组件自动适配浅色（通过CSS变量）
  - Build验证通过
- [x] 路由与 Layout
- [x] 首页开发（多问题检测、任务队列、案例选择）
- [x] **双层工作流重构完成（论文对齐版）**
  - mock/reasoning.js 重写为双层数据结构（主层M1~M6 + 子层S1~S8）
  - stores/reasoning.js 重构支持主子双层状态管理（visibleSubQuestions延迟显示）
  - ReasoningView 改为双层布局（上层总览 + 下层闭环）
  - 各层级组件完整实现
- [x] **数据管理页骨架搭建**
  - DataManagementView（5模块导航）
  - DataSourcePanel（P0完成）
  - DomainDbPanel（P0完成：实体流行度、术语映射、法律结构库）
  - HistoryPanel（P1占位完成）
  - RetrievalPanel（待创建）
- [x] **浅色主题迁移完成**
- [x] **多案例数据集集成**
  - legalCaseDataset.js：20+完整法律案例（涵盖合同/知识产权/债权/婚姻/侵权等16个领域）
  - 每案例包含：标题、原始问题、难度、3-4个子问题、推理指标、实体流行度
  - HomeView：从单一案例升级为10个案例选择（可扩展）
  - DEMO_CASES：动态从 LEGAL_CASE_DATASET 映射生成
- [x] **动态工作流生成完成**
  - getMockTask(caseId)：基于具体案例动态生成任务
  - getMockSubQuestions(caseId)：基于案例子问题生成
  - generateCompleteWorkflow()：基于问题内容动态生成S1-S8节点
  - generateM6FinalAnswer()：基于所有子问题答案动态生成最终结论
  - dynamicNodeGenerator.js：S1-S8节点生成函数集（共8个）
  - ReasoningView.vue：移除硬编码的task-001默认值，正确传递taskId
- [x] 构建验证完成（无错误）
- [x] 开发服务器运行中

### 待完成项目
- [ ] RetrievalPanel 创建（P1）
- [ ] 实体流行度知识库页面详细实现（基于新需求文档）
- [ ] 参数配置页（P1）
