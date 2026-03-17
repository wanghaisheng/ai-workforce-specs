下面给你一版 **AI Workforce Graph（AWG）核心数据模型 / 真正可落地的 schema v1**。
目标不是学术完美，而是满足三件事：

1. **能表达工作结构**
2. **能支持执行与编排**
3. **能支持评估与持续更新**

我会按这几个层次来定义：

**Graph worldview → Core entities → Core edges → Evidence & versioning → Execution & evaluation extensions → Minimal SQL-like schema**

---

# 1. 先定义 AWG 到底是什么

**AI Workforce Graph** 不是普通知识图谱。
它是一个把下面五类对象连起来的 **可计算工作图谱**：

```text
Work Intent
→ Work Structure
→ Capability Requirements
→ Execution Resources
→ Outcomes / Evaluation
```

更具体一点：

```text
Occupation
Position
Role
Task
Capability
Worker
Tool
Workflow
Artifact
Evaluation
```

这里最关键的一点是：

> **AWG 不是“描述 AI 有什么能力”的图谱，而是“描述工作如何被定义、分配、执行、评估”的图谱。**

所以它的本体不是单纯的 skill graph，而是 **work graph**。

---

# 2. 设计原则

先把 schema 的原则定死，不然后面会越做越乱。

## 2.1 工作优先，不是 Agent 优先

顶层抽象必须是：

```text
Work → Task → Capability
```

而不是：

```text
Agent → can do something
```

因为 agent 只是 workforce 的一种实现。

## 2.2 能力与执行分离

必须区分：

* **Capability**：抽象能力
* **Worker / Tool / Model**：具体实现

例如：

* Capability = `sql_query`
* Implementation = `duckdb_tool`, `bigquery_agent`, `postgres_worker`

## 2.3 规范层与实例层分离

必须区分两类图节点：

### 规范层（Definition Layer）

回答“应该是什么”

* Occupation
* Role
* Task Template
* Capability Definition

### 实例层（Runtime Layer）

回答“这次执行是什么”

* Task Run
* Worker Instance
* Workflow Run
* Output Artifact
* Evaluation Record

## 2.4 一切关系都必须可追溯

所有核心 edge 都需要：

* source
* confidence
* version
* valid time
* evidence

---

# 3. Graph 的分层模型

推荐把 AWG 拆成 4 层。

## Layer A: Work Definition Layer

定义工作的静态结构。

* Occupation
* Position Template
* Role
* Task Template
* Capability

## Layer B: Workforce Resource Layer

定义谁/什么可以执行。

* Worker
* Agent
* Human
* Tool
* Model
* Workflow Module

## Layer C: Runtime Execution Layer

定义一次具体执行。

* Work Order
* Task Run
* Assignment
* Artifact
* Event

## Layer D: Evaluation Layer

定义结果如何被评估。

* Metric
* Evaluation
* Benchmark
* Capability Score
* Reliability Record

---

# 4. 核心实体（Core Nodes）

下面是推荐的核心 node types。

---

## 4.1 Occupation

表示宏观工作类别。

**用途**
用于把 AI workforce 映射回传统工作世界。

### 字段

```json
{
  "occupation_id": "occ_data_analyst",
  "canonical_name": "Data Analyst",
  "description": "Analyzes business and operational data to generate insights.",
  "taxonomy_source": "internal|ONET|ESCO",
  "taxonomy_code": "15-2051.00",
  "domain": "data",
  "parent_occupation_id": null,
  "status": "active",
  "version": "2026.03.1"
}
```

---

## 4.2 PositionTemplate

表示某类岗位定义，不绑定单个人或单个 agent。

比如：

* AI Research Analyst
* AI Support Specialist
* Revenue Ops Copilot

### 字段

```json
{
  "position_template_id": "pos_ai_research_analyst_v1",
  "canonical_name": "AI Research Analyst",
  "occupation_id": "occ_research_analyst",
  "seniority": "mid",
  "function_area": "research",
  "scope": "market intelligence",
  "description": "Produces structured research outputs using AI and external tools.",
  "status": "active",
  "version": "2026.03.1"
}
```

---

## 4.3 Role

Role 比 position 更聚焦职责，不强调组织头衔。

例子：

* Literature Review
* Customer Triage
* SQL Reporting
* Draft Generation

### 字段

```json
{
  "role_id": "role_sql_reporting",
  "canonical_name": "SQL Reporting",
  "description": "Produces recurring reports using structured queries and summarized outputs.",
  "role_type": "functional",
  "domain": "analytics",
  "status": "active",
  "version": "2026.03.1"
}
```

---

## 4.4 TaskTemplate

这是整个图谱最重要的节点之一。
表示“可重复定义的任务类型”。

例子：

* Extract entities from document
* Generate weekly sales report
* Classify support ticket
* Draft partner outreach email

### 字段

```json
{
  "task_template_id": "task_generate_weekly_sales_report",
  "canonical_name": "Generate Weekly Sales Report",
  "description": "Aggregate weekly sales data, compute KPIs, and produce a narrative report.",
  "task_type": "analysis",
  "input_schema_ref": "schema_sales_report_input_v1",
  "output_schema_ref": "schema_sales_report_output_v1",
  "priority_class": "medium",
  "is_decomposable": true,
  "status": "active",
  "version": "2026.03.1"
}
```

---

## 4.5 Capability

Capability 是抽象能力，不绑定具体模型/工具。

例子：

* search_web
* summarize_text
* sql_query
* python_execute
* retrieve_knowledge
* classify_text
* generate_chart

### 字段

```json
{
  "capability_id": "cap_sql_query",
  "canonical_name": "SQL Query",
  "description": "Ability to write and execute SQL queries against a structured dataset.",
  "capability_type": "data_operation",
  "input_modality": ["structured_query", "table_schema"],
  "output_modality": ["table", "metric"],
  "requires_tooling": true,
  "safety_class": "standard",
  "status": "active",
  "version": "2026.03.1"
}
```

---

## 4.6 Worker

Worker 是执行实体的统一抽象。
它可以是：

* AI Agent
* Human
* Tool-backed automation
* Service worker

所以不要只建 `Agent`，而要建 `Worker` 超类。

### 字段

```json
{
  "worker_id": "wrk_research_agent_v2",
  "canonical_name": "Research Agent v2",
  "worker_type": "agent",
  "implementation_type": "llm_toolchain",
  "provider": "internal",
  "status": "active",
  "version": "2026.03.1"
}
```

`worker_type` 枚举建议：

* `agent`
* `human`
* `tool`
* `workflow`
* `service`

---

## 4.7 Tool

如果你希望把工具独立建模，而不是塞进 worker metadata，建议单独建 node。

### 字段

```json
{
  "tool_id": "tool_duckdb",
  "canonical_name": "DuckDB",
  "tool_type": "database_engine",
  "interface_type": "sql",
  "provider": "duckdb",
  "status": "active",
  "version": "2026.03.1"
}
```

---

## 4.8 Model

如果需要评估不同模型能力差异，建议单建。

### 字段

```json
{
  "model_id": "mdl_gpt_x",
  "canonical_name": "GPT-X",
  "model_family": "llm",
  "provider": "openai",
  "modalities": ["text"],
  "status": "active",
  "version": "2026.03.1"
}
```

---

## 4.9 WorkflowTemplate

表示多步协作模式。

### 字段

```json
{
  "workflow_template_id": "wf_market_research_v1",
  "canonical_name": "Market Research Workflow",
  "description": "Search, collect, analyze, summarize, review.",
  "status": "active",
  "version": "2026.03.1"
}
```

---

## 4.10 ArtifactType / OutputSchema

输出物也是关键对象，因为工作不是抽象完成，而是产出 artifact。

例子：

* report
* dataset
* chart
* code patch
* email draft

### 字段

```json
{
  "artifact_type_id": "art_report",
  "canonical_name": "Report",
  "mime_family": "document",
  "schema_ref": "schema_report_v1",
  "status": "active"
}
```

---

# 5. 运行时实体（Runtime Nodes）

接下来是实例层。

---

## 5.1 WorkOrder

表示一个具体目标或请求。

例子：

* “给我出本周销售复盘”
* “分析竞争对手新产品”
* “处理这一批客服工单”

### 字段

```json
{
  "work_order_id": "wo_20260317_001",
  "title": "Weekly Sales Review",
  "goal_statement": "Produce a weekly sales summary for APAC.",
  "requester_type": "human|system",
  "priority": "high",
  "status": "open",
  "created_at": "2026-03-17T09:00:00Z"
}
```

---

## 5.2 TaskRun

表示某个 task template 的一次执行实例。

### 字段

```json
{
  "task_run_id": "tr_20260317_001",
  "task_template_id": "task_generate_weekly_sales_report",
  "work_order_id": "wo_20260317_001",
  "status": "completed",
  "started_at": "2026-03-17T09:05:00Z",
  "completed_at": "2026-03-17T09:07:12Z"
}
```

---

## 5.3 Assignment

表示某个 task run 被分配给谁执行。

### 字段

```json
{
  "assignment_id": "asg_001",
  "task_run_id": "tr_20260317_001",
  "worker_id": "wrk_reporting_agent_v1",
  "assignment_type": "primary",
  "planned_by": "scheduler",
  "status": "accepted",
  "assigned_at": "2026-03-17T09:05:10Z"
}
```

---

## 5.4 ArtifactInstance

表示一次执行产出的具体对象。

### 字段

```json
{
  "artifact_instance_id": "arti_001",
  "artifact_type_id": "art_report",
  "task_run_id": "tr_20260317_001",
  "uri": "s3://.../weekly_sales_report.pdf",
  "status": "generated",
  "created_at": "2026-03-17T09:07:10Z"
}
```

---

## 5.5 Event

所有过程事件都应统一进 event log。

### 字段

```json
{
  "event_id": "evt_001",
  "event_type": "task_started",
  "subject_type": "task_run",
  "subject_id": "tr_20260317_001",
  "payload": {},
  "occurred_at": "2026-03-17T09:05:00Z"
}
```

---

# 6. 核心关系（Core Edges）

现在是图谱里真正最值钱的部分：edge。

---

## 6.1 Occupation → Role

### `OCCUPATION_HAS_ROLE`

```json
{
  "edge_type": "OCCUPATION_HAS_ROLE",
  "from_id": "occ_data_analyst",
  "to_id": "role_sql_reporting",
  "weight": 0.81,
  "confidence": 0.93
}
```

含义：某职业通常包含哪些角色。

---

## 6.2 PositionTemplate → Role

### `POSITION_INCLUDES_ROLE`

某岗位模板包含哪些角色职责。

---

## 6.3 Role → TaskTemplate

### `ROLE_REQUIRES_TASK`

这是非常关键的一条边。

例子：

* Role: SQL Reporting
* Task: Generate Weekly Sales Report

---

## 6.4 TaskTemplate → Capability

### `TASK_REQUIRES_CAPABILITY`

最关键 edge 之一。

```json
{
  "edge_type": "TASK_REQUIRES_CAPABILITY",
  "from_id": "task_generate_weekly_sales_report",
  "to_id": "cap_sql_query",
  "requirement_level": "core",
  "weight": 0.95,
  "confidence": 0.96
}
```

`requirement_level` 建议枚举：

* `core`
* `important`
* `optional`
* `conditional`

---

## 6.5 Capability → Worker

### `WORKER_PROVIDES_CAPABILITY`

表示谁具备某项能力。

```json
{
  "edge_type": "WORKER_PROVIDES_CAPABILITY",
  "from_id": "wrk_reporting_agent_v1",
  "to_id": "cap_sql_query",
  "proficiency": 0.89,
  "cost_score": 0.30,
  "latency_score": 0.72,
  "reliability_score": 0.91
}
```

注意：这里建议方向固定成：

* `Worker PROVIDES Capability`
  或
* `Capability PROVIDED_BY Worker`

只选一种，别混。

---

## 6.6 Worker → Tool

### `WORKER_USES_TOOL`

一个 worker 的实现依赖哪些工具。

---

## 6.7 Worker → Model

### `WORKER_POWERED_BY_MODEL`

当 worker 是 AI agent 时，底层由什么模型支撑。

---

## 6.8 WorkflowTemplate → TaskTemplate

### `WORKFLOW_CONTAINS_TASK`

多步工作流的结构。

---

## 6.9 TaskTemplate → ArtifactType

### `TASK_PRODUCES_ARTIFACT`

定义任务的标准产出物。

---

## 6.10 TaskRun → Worker

### `TASKRUN_ASSIGNED_TO_WORKER`

运行时分配边。

---

## 6.11 TaskRun → ArtifactInstance

### `TASKRUN_PRODUCED_ARTIFACT`

运行时产出边。

---

## 6.12 TaskRun → Evaluation

### `TASKRUN_HAS_EVALUATION`

把运行结果与评估挂钩。

---

# 7. 评估模型（Evaluation Schema）

AWG 如果没有 evaluation，就只是静态 catalog，不是 operating graph。

---

## 7.1 EvaluationRecord

### 字段

```json
{
  "evaluation_id": "eval_001",
  "subject_type": "task_run|worker|capability|workflow",
  "subject_id": "tr_20260317_001",
  "metric_set_id": "metrics_report_generation_v1",
  "overall_score": 0.87,
  "judge_type": "human|llm|rule|hybrid",
  "status": "final",
  "created_at": "2026-03-17T09:08:00Z"
}
```

---

## 7.2 MetricResult

### 字段

```json
{
  "metric_result_id": "mr_001",
  "evaluation_id": "eval_001",
  "metric_name": "accuracy",
  "metric_value": 0.91,
  "metric_unit": "score",
  "weight": 0.4
}
```

常见 metric：

* accuracy
* completeness
* factuality
* latency
* cost
* policy_compliance
* user_satisfaction

---

## 7.3 CapabilityScore

能力不应只在定义层静态存在，还要有动态评分。

### 字段

```json
{
  "capability_score_id": "cs_001",
  "worker_id": "wrk_reporting_agent_v1",
  "capability_id": "cap_sql_query",
  "score": 0.89,
  "sample_size": 183,
  "last_updated_at": "2026-03-17T00:00:00Z"
}
```

这张表很关键，它使调度器能根据真实表现选 worker。

---

# 8. 证据、版本与时效（必须有）

这是工程里最容易偷懒，但未来最贵的部分。

---

## 8.1 EvidenceRecord

每条重要 edge 都应该能挂 evidence。

### 字段

```json
{
  "evidence_id": "evd_001",
  "evidence_type": "document|execution_log|human_review|benchmark",
  "source_uri": "s3://.../eval.json",
  "excerpt": "Worker completed SQL aggregation with 98% field accuracy.",
  "created_at": "2026-03-17T09:08:10Z"
}
```

---

## 8.2 EdgeMeta

建议所有 edge 单独有元数据表，而不是只做裸三元组。

### 字段

```json
{
  "edge_id": "edge_001",
  "edge_type": "TASK_REQUIRES_CAPABILITY",
  "from_type": "task_template",
  "from_id": "task_generate_weekly_sales_report",
  "to_type": "capability",
  "to_id": "cap_sql_query",
  "weight": 0.95,
  "confidence": 0.96,
  "source_type": "human_defined",
  "version": "2026.03.1",
  "valid_from": "2026-03-01T00:00:00Z",
  "valid_to": null,
  "review_status": "approved"
}
```

---

# 9. 最小可用关系集（MVP Graph）

如果你先做 v1，不需要一次建全。
最小建议只做这 8 种 node 和 8 种 edge。

## Nodes

* occupation
* position_template
* role
* task_template
* capability
* worker
* task_run
* evaluation

## Edges

* `OCCUPATION_HAS_ROLE`
* `POSITION_INCLUDES_ROLE`
* `ROLE_REQUIRES_TASK`
* `TASK_REQUIRES_CAPABILITY`
* `WORKER_PROVIDES_CAPABILITY`
* `TASKRUN_ASSIGNED_TO_WORKER`
* `TASKRUN_HAS_EVALUATION`
* `TASKRUN_PRODUCED_ARTIFACT`

这已经够支撑：

* 工作定义
* 任务分配
* 能力匹配
* 执行评估

---

# 10. SQL 风格 schema 草案

下面给你一个偏关系型、同时兼容图查询的 schema。

---

## 10.1 entity tables

```sql
CREATE TABLE occupation (
  occupation_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  description TEXT,
  taxonomy_source TEXT,
  taxonomy_code TEXT,
  domain TEXT,
  parent_occupation_id TEXT,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);

CREATE TABLE position_template (
  position_template_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  occupation_id TEXT,
  seniority TEXT,
  function_area TEXT,
  scope TEXT,
  description TEXT,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);

CREATE TABLE role (
  role_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  description TEXT,
  role_type TEXT,
  domain TEXT,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);

CREATE TABLE task_template (
  task_template_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  description TEXT,
  task_type TEXT,
  input_schema_ref TEXT,
  output_schema_ref TEXT,
  priority_class TEXT,
  is_decomposable BOOLEAN NOT NULL,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);

CREATE TABLE capability (
  capability_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  description TEXT,
  capability_type TEXT,
  requires_tooling BOOLEAN NOT NULL,
  safety_class TEXT,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);

CREATE TABLE worker (
  worker_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  worker_type TEXT NOT NULL,
  implementation_type TEXT,
  provider TEXT,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);
```

---

## 10.2 runtime tables

```sql
CREATE TABLE work_order (
  work_order_id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  goal_statement TEXT,
  requester_type TEXT,
  priority TEXT,
  status TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);

CREATE TABLE task_run (
  task_run_id TEXT PRIMARY KEY,
  task_template_id TEXT NOT NULL,
  work_order_id TEXT,
  status TEXT NOT NULL,
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  created_at TIMESTAMP NOT NULL
);

CREATE TABLE assignment (
  assignment_id TEXT PRIMARY KEY,
  task_run_id TEXT NOT NULL,
  worker_id TEXT NOT NULL,
  assignment_type TEXT NOT NULL,
  planned_by TEXT,
  status TEXT NOT NULL,
  assigned_at TIMESTAMP NOT NULL
);
```

---

## 10.3 evaluation tables

```sql
CREATE TABLE evaluation (
  evaluation_id TEXT PRIMARY KEY,
  subject_type TEXT NOT NULL,
  subject_id TEXT NOT NULL,
  metric_set_id TEXT,
  overall_score REAL,
  judge_type TEXT,
  status TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL
);

CREATE TABLE metric_result (
  metric_result_id TEXT PRIMARY KEY,
  evaluation_id TEXT NOT NULL,
  metric_name TEXT NOT NULL,
  metric_value REAL,
  metric_unit TEXT,
  weight REAL
);

CREATE TABLE capability_score (
  capability_score_id TEXT PRIMARY KEY,
  worker_id TEXT NOT NULL,
  capability_id TEXT NOT NULL,
  score REAL NOT NULL,
  sample_size INTEGER,
  last_updated_at TIMESTAMP NOT NULL
);
```

---

## 10.4 generic edge table

最推荐做法是**一个通用 edge 表 + 少量专用明细表**。

```sql
CREATE TABLE graph_edge (
  edge_id TEXT PRIMARY KEY,
  edge_type TEXT NOT NULL,
  from_type TEXT NOT NULL,
  from_id TEXT NOT NULL,
  to_type TEXT NOT NULL,
  to_id TEXT NOT NULL,
  weight REAL,
  confidence REAL,
  requirement_level TEXT,
  source_type TEXT,
  version TEXT NOT NULL,
  valid_from TIMESTAMP,
  valid_to TIMESTAMP,
  review_status TEXT,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);
```

建议唯一约束：

```sql
UNIQUE(edge_type, from_id, to_id, version)
```

---

# 11. 推荐的查询能力

有了这个 schema，你就能做出这些关键查询：

## 11.1 某个角色需要哪些任务

```text
Role -> ROLE_REQUIRES_TASK -> TaskTemplate
```

## 11.2 某个任务需要哪些能力

```text
TaskTemplate -> TASK_REQUIRES_CAPABILITY -> Capability
```

## 11.3 当前有哪些 worker 可以执行该任务

```text
TaskTemplate -> required Capability
Capability -> provided by Worker
```

## 11.4 哪个 worker 最适合

综合：

* capability_score
* latency
* cost
* reliability
* safety constraints

## 11.5 某个职业如何被 AI workforce 覆盖

```text
Occupation -> Role -> Task -> Capability -> Worker
```

这是最关键的一个组织级分析能力。

---

# 12. 这套 schema 的真正价值

这套 AWG schema 的意义，不只是“把 AI 能力存起来”，而是让你可以回答下面这些问题：

### 工作定义类

* 一个岗位到底由哪些 role / task 组成？
* 一个 task 真正需要哪些 capability？

### 调度类

* 应该给哪个 worker 执行？
* 哪些任务适合 human，哪些适合 AI，哪些适合混合？

### 评估类

* 这个 worker 在什么 capability 上真的强？
* 它在哪些 task 上表现稳定？

### 组织设计类

* 一个 occupation 被 AI 覆盖了多少？
* 哪些 role 已经可以自动化，哪些还不行？

---

# 13. 我对 v2 的建议方向

如果你下一步继续演进，这 4 个扩展最重要：

## 13.1 Capability dependency

能力之间的依赖关系：

* `cap_data_cleaning` prerequisite_of `cap_reporting`

## 13.2 Constraint model

任务执行约束：

* 数据权限
* 成本预算
* SLA
* 风险等级

## 13.3 Multi-worker collaboration

一个 task run 由多个 worker 协作完成。

## 13.4 Human override / approval

关键任务需要人审。

---

# 14. 一句话收束

**AI Workforce Graph 的核心 schema，本质上是在建一张“工作—能力—执行—结果”图。**

最核心的主链路应该永远保持清晰：

```text
Occupation
→ Role
→ TaskTemplate
→ Capability
→ Worker
→ TaskRun
→ Evaluation
```

如果你愿意，我下一步可以直接继续补两样里最实用的一样：

**1. Neo4j / 图数据库版 schema**
或者
**2. 完整 ER 图 + edge taxonomy 清单**
