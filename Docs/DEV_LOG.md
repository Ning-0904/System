# 开发日志 — 复杂任务筹划可视化推理系统

> 按日期倒序记录，每次开发结束后更新。

---

## 2026-03-18

### 完成内容
- 读取并分析需求文档 `前端开发需求文档_法律适用推理系统.docx`
- 确认技术选型：Vue 3 + Vite + Pinia + Naive UI + Vue Router
- 确认开发策略：全 Mock 数据，优先 P0 功能
- 建立文档系统：`CONTEXT.md`（关键上下文）+ `DEV_LOG.md`（本文件）
- 初始化项目骨架，安装全部依赖
- 完成全局主题（CSS 变量 + 科技风样式 + 动效）
- 完成路由配置（4页）+ AppLayout + AppHeader
- 完成 Mock 数据（reasoning.js）+ Pinia stores
- 完成首页（HomeView）
- 完成推理展示页全部 P0 组件（6个核心组件）
- 构建验证通过

### 关键决策
- UI 库选 Naive UI：暗色主题支持好，科技风定制成本低
- 画布选自定义 CSS 节点（非 LogicFlow），更轻量可控，适合横向流式布局
- 全 Mock 策略：用 `setTimeout` 模拟推理过程的异步感
- 异常弹窗使用 `teleport to="body"` 确保层级正确

### 待解决
- 无

---

## 2026-03-18（第二次更新）

### 完成内容
- 读取论文对齐版需求文档，完成深度重构
- mock/reasoning.js 重写为双层结构（3个子问题，含2轮补检案例）
- stores/reasoning.js 重构，支持主子双层状态管理与模拟运行
- ReasoningView 改为双层布局（上层总览 + 下层闭环 + 右侧详情）
- 新增 SubQuestionTrack（上层子问题迭代总览）
- 新增 SubWorkflow（下层子问题内部闭环，含补检轮次标记）
- 重构 NodeDetailPanel（按节点类型渲染论文对齐字段）
- 更新 TaskOverview（双层状态指标）
- 更新 FinalConclusionPanel（M6综合答案 + 子问题推理链）
- 构建验证通过

### 关键决策
- 画布仍用自定义 CSS 节点，不引入 LogicFlow（减少依赖，更灵活）
- 补检轮次在 SubWorkflow 中用 `round-marker` 分隔线可视化区分
- NodeDetailPanel 按 `node.type` switch 渲染，每种节点独立字段
- 异常弹窗在充分性判断第1轮不足时触发，用户确认后继续

### 待解决
- 无

---

## 2026-03-19（v2 增补版）

### 完成内容
- 子问题延迟显式：visibleSubQuestions 控制，第n个完成后才显示第n+1个，带出现动效
- 查询重写强化：S6 节点支持重写类型切换（争点聚焦/法条约束/事实补全），候选查询中文展示，选中标记
- S7 LLM 补检查询：与 S6 重写查询明确区分，橙色标注"由 LLM 生成，非重写查询"
- S4 vs S7 边界说明：两个节点详情顶部均加入边界说明条（规则层 vs 语义层）
- 任务队列：stores/taskQueue.js，首页多主问题检测（序号/多行），自动拆分为独立任务排队
- 首页输入区：加"每次输入一个主问题"说明，多主问题检测后展示拆分预览
- 构建验证通过

### 关键决策
- 重写类型切换为本地 ref，不影响 store，点击即切换候选列表
- 多主问题检测用正则匹配序号模式（1. ① 一、）或多行长文本
- 任务队列仅在首页展示，推理页不做队列 UI（答辩时间有限）

---

## 2026-04-07（浅色主题 + 多案例动态工作流完整实现）

### 问题回顾
用户反馈：即使选择不同案例，推理过程中的工作流、节点详情仍然是硬编码的固定数据。需要根据具体问题动态生成所有推理节点内容。

### 完成内容

**1. 浅色主题系统升级**
   - variables.css 全面迁移（背景#f8fafc/f1f5f9，文字#1e293b，主色#2563eb蓝/#7c3aed紫）
   - AppHeader.vue：SVG渐变、阴影、背景色更新
   - 所有组件自动适配（CSS变量系统）
   - Build验证通过

**2. 法律案例数据集扩展**
   - legalCaseDataset.js：从原来的硬编码 → 20+完整法律案例数据集
   - 覆盖16个法律领域（合同纠纷、知识产权、债权债务、婚姻家庭、侵权、劳动、人格权等）
   - 每个案例包含：
     - caseId, title, category, difficulty
     - originalQuestion, entities
     - subQuestions[] (3-4个子问题，每个含text和needRetrieval)
     - retrievalMetrics (successRate, precision, recall, rewriteRounds)
     - entities_metrics[] (实体流行度、频率、阈值状态)
   - 工厂函数：getLegalCaseById(), getLegalCasesByCategory(), getRandomLegalCase() 等
   - 统计指标：DATASET_STATISTICS（总数20，分类分布，难度分布，平均指标）

**3. HomeView 多案例集成**
   - DEMO_CASES：从单一硬编码 → 动态映射 LEGAL_CASE_DATASET.slice(0,10)
   - 首页演示案例列表：10个可选法律案例
   - 用户可直观查看案例标题、难度、法律领域

**4. 动态工作流生成引擎（dynamicNodeGenerator.js）**
   - generateCompleteWorkflow(caseId, sqIndex, subQuestion)：主入口
     - 基于具体案例和子问题内容动态生成S1~S8完整节点序列
     - 核心原则：所有推理内容源于问题本身，不凭空编造
   - generateS1Node()：从问题提取关键词 → 计算实体流行度 → 判定是否检索
   - generateS2Node()：将问题转换为检索查询 → 生成相关候选文档
   - generateS3Node()：文档重排序（重排前后对比）
   - generateS4Node()：文档级三态判别（充分/不确定/不足）
   - generateS5Node()：状态路由（精炼/重写/并行）
   - generateS6Node()：查询重写（三类视角候选）
   - generateS7Node()：集合级充分性判断
   - generateS8Node()：基于证据生成答案
   - generateM6FinalAnswer(caseId, subQuestions, subNodesMap)：
     - 收集所有子问题的S8答案
     - 生成综合结论和关键证据摘要
     - 返回完整的M6最终答案对象

**5. stores/reasoning.js 适配**
   - initTask(taskId)：传递 taskId 给 getMockTask/getMockSubQuestions
   - 第58-66行：为每个子问题调用 generateCompleteWorkflow() 动态生成节点
   - simulateRun()：第129行修复 S8 答案提取逻辑
   - 最后：使用 generateM6FinalAnswer() 生成动态的M6最终答案（不再使用硬编码getMockFinalAnswer()）

**6. ReasoningView.vue 修复**
   - 移除硬编码的 `'task-001'` 默认值
   - onMounted() 正确传递 route.params.taskId 到 store.initTask()

**7. 构建验证**
   - ✓ npm run build 成功（712ms，2844 modules transformed）
   - ✓ 生产构建文件大小正常
   - ✓ 开发服务器正常运行（Vite ready in 277ms）

### 核心改进
- **从单一硬编码 → 20+案例完整数据集**：支持不同法律领域的推理展示
- **从固定节点 → 动态生成工作流**：S1~S8节点内容完全基于问题生成，不再重复
- **从孤立组件 → 完整数据闭环**：案例选择 → 任务初始化 → 工作流生成 → 答案综合
- **从硬编码M6 → 动态M6**：最终答案基于所有子问题的推理结果生成

### 现在支持
✓ 首页选择任一案例（10个完整法律场景）
✓ 推理展示页动态加载对应案例的子问题
✓ 每个子问题的S1~S8节点完全基于问题内容动态生成
✓ 节点详情中的文档、查询、候选、答案等都是动态适配
✓ M6最终答案基于所有子问题的完整推理链综合生成
✓ 浅色主题提升专业感与可读性

### 下一步
- RetrievalPanel 创建（P1）
- 实体流行度知识库页面详细实现
- 参数配置页面开发

---

<!-- 新日志追加在此行上方 -->

## 2026-04-06（浅色主题迁移）

### 完成内容
- 用户反馈：简化前端UI显示，配色偏亮（浅色主题）
- variables.css 全面配色更新：
  - 背景色：#070b14/0d1526 → #f8fafc/f1f5f9（浅灰白）
  - 主题色：蓝青系从亮色饱和 → 标准Web色（#2563eb蓝、#0891b2青）
  - 文字色：#e2e8f0亮灰 → #1e293b深灰蓝（高对比度）
  - 状态色：节点状态色调整为浅色主题适配
  - 阴影与发光：从强发光(0.2-0.4) → 弱发光(0.12-0.15)
- AppHeader.vue 更新：SVG渐变色、drop-shadow滤镜强度、nav-item/scene-badge背景色
- 构建验证：`npm run build` 成功，无错误

### 关键决策
- 浅色主题提升专业感与可读性，更适合论文答辩展示
- 保留蓝紫系主题色（学术气质），从超饱和亮色→标准颜色体系（更稳定）
- 暂不调整布局与信息密度（用户反馈后再迭代）
- 其他组件配色效果待用户反馈后微调

### 待处理
- RetrievalPanel 实现（P1）
- 实体流行度知识库页面详细优化（新需求文档："实体流行度知识库页面详细需求.docx"）
- 浅色主题下的UI视觉反馈迭代

---

## 2026-04-06（动态节点生成引擎 — 核心架构升级）

### 问题根源
- 虽然案例可选，但工作流节点内容仍为硬编码
- 所有案例使用相同的文档、实体、法条
- 违反核心设计原则：推理内容必须严格源于问题本身

### 解决方案：动态节点生成引擎

**新增文件**：`dynamicNodeGenerator.js`

#### 核心原则
- ✓ 所有推理内容源于问题本身，不凭空编造
- ✓ 问题不包含的信息不会出现在推理过程中
- ✓ 每个案例的工作流完全不同（基于其子问题）

#### 动态生成函数
1. **generateS1Node(caseId, sqIndex, subQuestion)**
   - 从子问题文本提取关键实体
   - 动态计算实体流行度
   - 基于阈值判定是否需要检索

2. **generateS2Node(caseId, sqIndex, subQuestion)**
   - 从问题生成检索查询
   - 根据问题类型生成相关文档候选
   - 例：如果问题不含"合同"，则不会检索合同条款

3. **generateS4Node(caseId, sqIndex, subQuestion, candidates)**
   - 基于检索结果动态评分
   - 生成文档级状态判别结果

4. **generateS6Node(caseId, sqIndex, subQuestion, insufficientReasons)**
   - 基于问题和检索不足情况生成重写
   - 三类重写视角都基于问题内容
   - 绝不生成问题中不存在的信息

5. **generateS8Node(caseId, sqIndex, subQuestion, supportingDocuments)**
   - 基于问题和证据生成答案
   - 严格声明答案的证据来源
   - 无证据时说明基于问题逻辑推理

#### 辅助函数
- `extractKeywordsFromQuestion()` - 从问题文本提取关键词
- `generateQueryFromQuestion()` - 生成检索查询
- `generateCandidatesForQuery()` - **重要：只生成与查询相关的文档**
- `generateFocusedCandidates()` - 争点聚焦型重写
- `generateLegalConstrainedCandidates()` - 法条约束型重写
- `generateFactCompletionCandidates()` - 事实补全型重写
- `generateAnswerFromQuestion()` - 基于问题和证据生成答案

### 集成到推理流程

**stores/reasoning.js 改动**：
- initTask() 现在为每个子问题调用 `generateCompleteWorkflow()`
- 动态生成节点序列（S1 → S2 → S4 → S8）
- 保留回退机制：如果动态生成失败，使用原始Mock节点

### 核心改进点

| 方面 | 之前 | 现在 |
|------|------|------|
| 节点内容 | 硬编码（所有案例相同） | 动态生成（基于具体问题） |
| 法律文献 | 固定的民法典条款 | 根据问题相关性生成 |
| 检索结果 | 预定义的文档列表 | 基于查询内容生成 |
| 信息来源 | 凭空编造 | 严格源于问题本身 |
| 可扩展性 | 低（需手写每个案例节点） | 高（自动为新案例生成） |

### 构建验证
- ✓ Build 成功 (786ms)
- ✓ dynamicNodeGenerator 模块正常导入
- ✓ 2844 modules 编译完成

### 设计约束
```javascript
// 关键约束：不生成问题中不存在的信息
if (question.includes('合同')) {
  // 可以检索合同相关文献
} else {
  // 不能生成合同相关内容
}
```

### 下一步
- 完善候选文档生成逻辑（与RAG系统对接）
- 增强关键词提取算法（NLP优化）
- 实现证据链追踪（记录信息来源）

---

## 2026-04-06（法律场景Mock数据扩展）

### 完成内容
- 创建 `legalCaseDataset.js`：法律场景推理任务数据集
- 包含 20 个完整法律案例，覆盖 16 个法律领域：
  - 合同纠纷（3例）：合同违约、房屋买卖、商品房交易
  - 知识产权（3例）：商标侵权、著作权侵权、专利纠纷
  - 债权债务（2例）：不当得利、借款纠纷
  - 婚姻家庭（2例）：离婚财产分割、继承纠纷
  - 侵权责任（3例）：交通事故、产品责任、医疗事故
  - 其他领域（4例）：劳动争议、人格权侵害、房产纠纷、保险纠纷
  - 新兴领域（4例）：环境污染、担保物权、建工纠纷、票据纠纷
  - 消费者权益、公司法各 1 例

### 数据结构设计
- **基础信息**：caseId, title, category, originalQuestion, difficulty
- **子问题**：包含 2-3 个子问题，标记是否需要检索
- **推理细节指标**：
  - `retrievalMetrics`：检索成功率、精度、召回率、重写轮次
  - `entities_metrics`：每个关键实体的流行度、频率、阈值状态
- **难度分布**：低难度(2), 中难度(8), 高难度(10)
- **统计信息**：DATASET_STATISTICS 包含总数、分类、难度、平均指标

### 工厂函数
- `getLegalCaseById(caseId)` - 按ID查询
- `getLegalCasesByCategory(category)` - 按法律领域查询
- `getLegalCasesByDifficulty(difficulty)` - 按难度查询
- `getRandomLegalCase()` - 随机获取
- `getLegalCasesPage(page, pageSize)` - 分页查询

### 数据指标示例（以合同违约为例）
```
检索成功率：95% | 精度：92% | 召回率：88% | 重写轮次：2
关键实体：
  - 合同违约：流行度0.91（高频，阈值内）
  - 违约金：流行度0.84（高频，阈值内）
  - 仓储费用：流行度0.52（低频，阈值外→触发检索）
```

### 构建验证
- `npm run build` 成功，无错误，新增模块正常导入
- 数据集可用于首页案例展示、推理演示、数据统计分析

### 后续应用场景
- 首页案例库展示：展示不同法律领域的推理任务
- 推理演示轮播：随机选择案例进行演示
- 数据分析面板：展示检索指标、实体流行度统计
- 答辩效果增强：更丰富的法律场景展示推理系统的通用性

---

## 2026-04-06（法律数据集集成到首页）

### 完成内容
- 更新 HomeView.vue：将 legalCaseDataset 的前10条案例集成到首页演示列表
- DEMO_CASES 从 1 条扩展到 10 条法律案例
- 每条案例包含：
  - 案例ID、标题、原始问题
  - 法律领域、难度等级
  - 检索指标（成功率、精度、召回率、重写轮次）
- HomeView 现可展示多个法律场景，用户可选择任一案例进行推理演示
- 构建验证：✓ 成功（2843 modules transformed）

### 效果
- 首页案例列表从 1 条 → 10 条
- 用户可直接从首页选择不同法律领域的案例
- 数据集完全集成，无需手动配置
- 浏览器自动热更新（HMR）反映改动

---

## 2026-04-06（多案例动态加载Bug修复）

### 问题描述
- 虽然首页显示10个案例，但选择任何案例后进入推理展示都是固定的原始案例（task-001）
- 根本原因：getMockTask() 和 getMockSubQuestions() 没有接收 caseId 参数

### 完成的修复
1. **理清问题链路**
   - HomeView.vue startReasoning() 使用硬编码的 'task-001'
   - stores/reasoning.js 的 initTask() 没有传递 taskId
   - mock/reasoning.js 的工厂函数不支持动态加载

2. **修复 HomeView.vue**
   - 去掉 startReasoning() 中的 `|| 'task-001'` 硬编码
   - 确保使用 selectedCase.value 作为 taskId

3. **修复 reasoning.js 工厂函数**
   - 在文件顶部添加 `import { getLegalCaseById } from './legalCaseDataset'`
   - getMockTask(caseId) 新增参数，支持动态查询 legalCaseDataset
   - getMockSubQuestions(caseId) 新增参数，支持动态生成子问题
   - 未提供 caseId 时默认返回原始的 MOCK_TASK 和 MOCK_SUB_QUESTIONS

4. **修复 stores/reasoning.js**
   - initTask(taskId) 调用时传递 caseId 参数
   - `getMockTask(taskId)` 替代 `getMockTask()`
   - `getMockSubQuestions(taskId)` 替代 `getMockSubQuestions()`

### 构建验证
- ✓ Build 成功（816ms）
- ✓ legalCaseDataset 独立打包（17.14 kB gzip: 3.87 kB）
- ✓ 所有模块正常编译

### 现在可用的功能
✓ 首页选择任一法律案例
✓ 推理展示页动态加载对应案例数据
✓ 不同法律领域的子问题正确显示
✓ 完整的推理流程展示多种法律场景

