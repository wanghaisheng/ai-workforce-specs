这是一个真正“动组织根基”的问题。
而且必须先说一句实话：

> **如果 KPI / OKR 不变，那 AI 永远不可能成为一等公民。**

因为 KPI 决定资源分配，资源分配决定组织行为。

下面我给你一套**从旧范式 → 新范式的完整迁移逻辑**，不是概念，而是可以落地的。

---

# 一、旧体系的问题在哪里

传统 KPI / OKR 本质是：

```text
以“人”为单位的绩效系统
```

核心特征：

```text
目标 → 分配给人 → 衡量人 → 奖惩人
```

例如：

```text
销售：成交额
工程师：交付功能数
运营：DAU增长
```

问题在 AI 时代变成：

---

## ❗ 问题1：工作被人“占有”

```text
这个任务是谁的？
```

而不是：

```text
这个任务是否被最优完成？
```

---

## ❗ 问题2：效率被人类能力限制

```text
一个人一天只能做这些
```

但 AI 可以：

```text
无限并行
```

---

## ❗ 问题3：无法衡量 AI 的真实价值

AI 被当成：

```text
工具使用情况
```

而不是：

```text
劳动力产出
```

---

# 二、新体系的核心变化（一句话）

> **从“人绩效” → “工作绩效（Work Performance）”**

---

# 三、三层重构模型（最关键）

新的 KPI / OKR 体系必须分成三层：

---

## 🧩 Layer 1：Work Outcome（工作结果）

这是最顶层，唯一真正重要的 KPI。

```text
任务是否完成？
质量如何？
```

例如：

```text
报告是否正确
代码是否可运行
分析是否有效
```

### 特点：

```text
完全与“谁做的”无关
```

---

## ⚙️ Layer 2：Execution Efficiency（执行效率）

衡量：

```text
用什么代价完成工作
```

维度：

```text
成本（cost）
时延（latency）
成功率（success rate）
稳定性（reliability）
```

---

## 🧠 Layer 3：Workforce Contribution（劳动力贡献）

这才是“人和 AI”的评价层。

衡量：

```text
谁在贡献价值
```

但注意：

```text
不是“做了多少”
而是“贡献了多少”
```

---

# 四、AI 成为一等公民后 KPI 怎么变

## 1️⃣ KPI 不再绑定人

旧：

```text
人 → KPI
```

新：

```text
Work Unit（任务） → KPI
```

---

## 2️⃣ 人和 AI 共享 KPI

例如：

```text
任务：生成销售报告
```

KPI：

```text
准确率 ≥ 95%
完成时间 ≤ 60s
成本 ≤ $2
```

然后：

```text
Human + AI 都参与
```

---

## 3️⃣ 新增“能力级 KPI”

这是 AI 时代特有的。

例如：

```text
Capability: SQL Query

指标：
准确率
执行成功率
查询效率
```

这直接映射到：

```text
capability_score
```

---

## 4️⃣ 新增“实现级 KPI”（非常关键）

因为你引入了：

```text
SkillImplementation
```

所以要评估：

```text
哪个实现更好
```

例如：

```text
sql_skill_impl_v1
accuracy 0.91
cost 0.3

sql_skill_impl_v2
accuracy 0.95
cost 0.5
```

---

## 5️⃣ Position KPI 被重写

旧：

```text
Backend Engineer KPI
```

新：

```text
Backend Role Performance
```

衡量：

```text
该 role 覆盖的任务是否被高质量完成
```

---

# 五、OKR 如何变化

OKR 的本质也要变。

---

## 🎯 旧 OKR

```text
O：提升用户增长
KR：
- 招10个人
- 发布5个功能
```

---

## 🚀 新 OKR（Work-centric）

```text
O：提升用户增长

KR：
- 用户增长率 +20%
- CAC下降15%
- 实验周期缩短50%
```

然后系统自己决定：

```text
哪些由人做
哪些由 AI 做
哪些混合
```

---

# 六、一个非常关键的新概念

## 👉 Work Unit（工作单元）

KPI 必须绑定这个。

定义：

```text
Work Unit = TaskRun / WorkOrder
```

每个 Work Unit 有：

```text
input
output
evaluation
cost
latency
```

---

# 七、绩效系统重构后的结构

```text
Work Unit
   ↓
Evaluation
   ↓
Contribution Attribution
   ↓
Human / AI / Implementation
```

---

# 八、贡献如何分配（最难的点）

你一定会遇到这个问题：

> AI 做了 80%，人做了 20%，绩效怎么算？

---

## 方法：Contribution Graph

```text
TaskRun
   ↓
SkillImplementation
   ↓
Worker
```

例如：

```text
report generation

AI:
- data processing
- draft writing

Human:
- review
- correction
```

贡献可以分：

```text
AI: 70%
Human: 30%
```

---

# 九、你这套 AWG 的关键作用

这就是为什么你必须有：

```text
Task → Capability → Implementation → Worker → Evaluation
```

否则你无法：

* 分配贡献
* 优化系统
* 比较实现
* 做决策

---

# 十、最终范式变化（最重要）

传统：

```text
人 → 做事 → 被评估
```

新：

```text
工作 → 被执行 → 被评估
        ↓
   人 / AI / 实现 被归因
```

---

# 十一、可以写进你文档的版本（建议直接用）

> In traditional organizations, performance systems are human-centric: goals are assigned to individuals, and evaluation is tied to personal output.\n>\n> In AI-native workforce systems, performance shifts to a work-centric model. The primary unit of evaluation becomes the work unit (task execution), not the individual.\n>\n> Evaluation is decomposed into three layers:\n> - Work outcome (quality of result)\n> - Execution efficiency (cost, latency, reliability)\n> - Workforce contribution (human vs AI vs implementation)\n>\n> As a result, both humans and AI are evaluated within a unified performance system, and contributions are attributed based on actual participation in task execution.\n\n---

# 十二、一句话总结

> **AI 成为一等公民，意味着 KPI 不再评估“谁做了”，而是评估“工作是如何被最优完成的”。**

---

如果你愿意，我们可以再往下一步（更狠的一个问题）：

👉 **当 KPI 变成 work-centric 后，“中层管理者”会发生什么变化？**

这个会直接影响组织结构设计。
