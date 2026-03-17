# AI Workforce Graph / AWOS 实施蓝图（文档草案 v1）

## 1. 文档目的

本文档从“以终为始”的角度，定义 AI Workforce Operating System（AWOS）与 AI Workforce Graph（AWG）的目标状态、核心概念、数据模型与实现抽象。

重点补充一个关键现实约束：

> 在系统落地时，所谓一个 agent，并不一定是一个单体程序；它的底层实现很可能是 **IDE + 某个模型 + 提示词 + 工具链 + 运行环境** 所共同构成的一个 skill 或 role 的具体执行单元。

因此，本文档不仅定义 Work / Role / Task / Capability，也定义 **Implementation Layer（实现层）**，用于表达 agent 背后的实际执行形态。

---

## 2. 终态定义

系统全部实现后，我们不再把 AI 看成一个“聊天模型”，而是看成组织中的一种新型劳动力资源。

在这个终态里：

- 工作由 `Occupation → Position → Role → Task → Capability` 进行定义
- 执行由 `Worker / Agent / Tool / Model / IDE / Prompt / Environment` 进行实现
- 编排层根据任务需求把工作分配给合适的实现单元
- 评估层持续更新每个实现单元的表现、成本、延迟与可靠性

这意味着：

1. **上层定义工作结构**
2. **中层定义能力需求**
3. **下层定义实现方式**
4. **运行时完成编排与评估闭环**

---

## 3. 核心设计判断

### 3.1 Agent 不是最底层抽象

在业务语义上，我们常说“一个 agent 在做事”。

但在真实系统中，一个 agent 的实现很可能由以下部分组成：

- 一个 IDE 或工作台
- 一个主模型（如 GPT / Claude / Gemini / 开源模型）
- 一个或多个辅助模型
- 一组提示词模板
- 一组技能（skills）
- 一组工具调用能力
- 一个运行环境（代码执行、浏览器、数据库、文件系统等）

因此：

> **Agent 是组织可见的执行角色，Implementation 是工程可见的执行形态。**

### 3.2 Skill 和 Role 要区分

- `Role`：从工作语义看，表示职责，例如“Research Analyst”“SQL Reporting”
- `Skill`：从执行能力看，表示可复用的工作能力单元，例如“web search”“summarize findings”“generate SQL”

在实现层，常见情况是：

- 一个 `Role` 由多个 `Skills` 组合实现
- 一个 `Skill` 由 `IDE + Model + Prompt + Toolchain` 实现

### 3.3 实现必须可替换

同一个 role / skill，可能有多种实现：

- 基于 Claude 的版本
- 基于 GPT 的版本
- 基于开源模型的版本
- 基于 IDE 工作流的版本
- 基于纯 API orchestration 的版本

所以系统必须支持：

- 统一定义能力
- 多实现并存
- 运行时择优调度
- 持续比较效果

---

## 4. 分层架构

### 4.1 Work Definition Layer

定义“做什么”。

核心对象：

- Occupation
- PositionTemplate
- Role
- TaskTemplate
- Capability

### 4.2 Workforce Resource Layer

定义“谁能做”。

核心对象：

- Worker
- Agent
- Human
- Tool
- Model
- Workflow

### 4.3 Implementation Layer

定义“具体怎么做”。

核心对象：

- SkillImplementation
- RoleImplementation
- IDEProfile
- ModelBinding
- PromptTemplate
- Toolchain
- RuntimeEnvironment

### 4.4 Runtime Layer

定义“一次具体执行”。

核心对象：

- WorkOrder
- TaskRun
- Assignment
- ExecutionRun
- Artifact
- Event

### 4.5 Evaluation Layer

定义“做得怎么样”。

核心对象：

- Evaluation
- MetricResult
- CapabilityScore
- ReliabilityRecord
- CostRecord

---

## 5. 新增实现层（Implementation Layer）

这一层是本文档补充的重点。

### 5.1 为什么需要实现层

如果没有实现层，图谱只能表达：

- 哪个 role 需要哪些 task
- 哪个 task 需要哪些 capability
- 哪个 worker 提供哪些 capability

但这还不够，因为在实际工程里，我们还需要知道：

- 这个 worker 是怎么实现的
- 它依赖哪个 IDE / 模型 / 提示词 / 工具链
- 它到底是一个 role implementation，还是一个 skill implementation
- 当某个模型退化或成本过高时，替换路径是什么

因此，实现层负责连接：

`Capability / Skill / Role  <->  具体工程实现`

---

## 6. Implementation Layer 的核心实体

### 6.1 SkillImplementation

表示某个 skill 的具体实现版本。

示例：

- `web_search_skill_v1`
- `sql_analysis_skill_v3`
- `report_drafting_skill_v2`

建议字段：

- `skill_impl_id`
- `canonical_name`
- `skill_id`
- `implementation_type`
- `primary_model_id`
- `prompt_template_id`
- `toolchain_id`
- `runtime_env_id`
- `ide_profile_id`
- `status`
- `version`

`implementation_type` 可选值：

- `prompt_workflow`
- `ide_agent`
- `tool_orchestrated`
- `code_executor`
- `hybrid`

### 6.2 RoleImplementation

表示某个 role 的具体实现版本。

例子：

- `research_analyst_role_impl_v1`
- `sql_reporting_role_impl_v2`

一个 role implementation 通常由多个 skill implementations 组成。

建议字段：

- `role_impl_id`
- `canonical_name`
- `role_id`
- `composition_type`
- `primary_worker_id`
- `status`
- `version`

### 6.3 IDEProfile

定义某类 IDE / 工作台配置。

例子：

- Cursor Workspace
- Claude Code Workspace
- Internal Research IDE

建议字段：

- `ide_profile_id`
- `canonical_name`
- `ide_type`
- `workspace_config_ref`
- `permissions_profile`
- `status`

### 6.4 ModelBinding

定义某个实现绑定了哪些模型。

例子：

- 主模型：GPT-5
- 辅助模型：embedding model / reranker / vision model

建议字段：

- `model_binding_id`
- `subject_type`
- `subject_id`
- `model_id`
- `binding_role`
- `temperature_profile`
- `context_window_profile`
- `status`

`binding_role` 可选值：

- `primary_reasoner`
- `secondary_reasoner`
- `retriever`
- `reranker`
- `vision`
- `coder`

### 6.5 PromptTemplate

定义提示词模板。

建议字段：

- `prompt_template_id`
- `canonical_name`
- `template_text`
- `template_type`
- `input_schema_ref`
- `output_schema_ref`
- `status`
- `version`

`template_type` 可选值：

- `system`
- `task_instruction`
- `reflection`
- `evaluation`
- `repair`

### 6.6 Toolchain

定义实现依赖的工具链。

例子：

- browser
- python runtime
- sql engine
- file system
- vector store

建议字段：

- `toolchain_id`
- `canonical_name`
- `description`
- `tool_list`
- `permissions_profile`
- `status`

### 6.7 RuntimeEnvironment

定义运行环境。

例子：

- sandbox python
- container runtime
- secure browser
- enterprise DB runtime

建议字段：

- `runtime_env_id`
- `canonical_name`
- `environment_type`
- `resource_limits`
- `network_policy`
- `filesystem_policy`
- `status`

---

## 7. 新增关系定义

### 7.1 Skill → SkillImplementation

`SKILL_IMPLEMENTED_BY`

含义：某个 skill 可以由一个或多个具体实现提供。

### 7.2 Role → RoleImplementation

`ROLE_IMPLEMENTED_BY`

含义：某个 role 可以由一个或多个具体实现提供。

### 7.3 RoleImplementation → SkillImplementation

`ROLE_IMPL_COMPOSED_OF_SKILL_IMPL`

含义：一个 role 的实现由多个 skill 的实现组成。

### 7.4 SkillImplementation → Model

`SKILL_IMPL_POWERED_BY_MODEL`

### 7.5 SkillImplementation → PromptTemplate

`SKILL_IMPL_DRIVEN_BY_PROMPT`

### 7.6 SkillImplementation → Toolchain

`SKILL_IMPL_USES_TOOLCHAIN`

### 7.7 SkillImplementation → RuntimeEnvironment

`SKILL_IMPL_RUNS_ON_ENV`

### 7.8 SkillImplementation → IDEProfile

`SKILL_IMPL_USES_IDE_PROFILE`

### 7.9 Worker → RoleImplementation

`WORKER_REALIZES_ROLE_IMPL`

### 7.10 Worker → SkillImplementation

`WORKER_REALIZES_SKILL_IMPL`

---

## 8. 统一主链路

完整系统中的主链路应为：

`Occupation → PositionTemplate → Role → TaskTemplate → Capability → Skill → SkillImplementation → Worker → TaskRun → Evaluation`

如果需要从工程实现反推业务语义，也可以反向走：

`Model / Prompt / IDE / Toolchain → SkillImplementation → RoleImplementation → Role → Position → Occupation`

这使系统同时具备两种视角：

1. **业务视角**：组织在定义什么工作
2. **工程视角**：系统在如何实现这些工作

---

## 9. 终态系统中 agent 的真实定义

在这套框架下，一个 agent 不再被简单定义为“一个模型+提示词”。

更准确地说：

> **Agent = 对外可见的工作实体；其内部由一个或多个 role / skill implementations 驱动。**

例如：

### Research Agent

对外角色：

- `AI Research Analyst`

内部实现：

- search skill implementation
- source validation skill implementation
- synthesis skill implementation
- report drafting skill implementation

底层依赖：

- IDE：Research IDE
- 主模型：GPT-5 / Claude / other
- 提示词：research planner prompt / synthesis prompt
- 工具链：browser + python + doc writer
- 运行环境：secure container runtime

因此，一个 agent 只是组织层的“包装视图”；真正可维护、可替换、可评估的是内部 implementation graph。

---

## 10. 为什么这对 AWG 特别重要

加入实现层之后，AWG 才能回答下面这些关键问题：

### 工作定义问题

- 哪个 role 需要哪些 skill？
- 哪个 skill 支撑哪些 task？

### 工程实现问题

- 这个 skill 当前由哪种实现提供？
- 它依赖哪个模型、提示词和工具链？
- 如果模型替换，影响哪些 role？

### 调度问题

- 同一个 skill 有多个实现，应该选哪个？
- 当前任务优先优化成本、质量还是时延？

### 评估问题

- 是 role 设计有问题，还是底层 implementation 表现差？
- 问题来自模型、提示词、工具链，还是运行环境？

---

## 11. 推荐的最小实现范围（MVP）

建议第一版先支持以下新增实体：

- `SkillImplementation`
- `RoleImplementation`
- `PromptTemplate`
- `ModelBinding`
- `Toolchain`
- `RuntimeEnvironment`

如果第一版就已经明确大量 IDE 驱动模式，再加入：

- `IDEProfile`

---

## 12. 一句话定义这次补充

这次补充的本质是：

> **把“agent 是怎么被实现出来的”从隐式工程细节，升级为 AWG 中的显式数据模型。**

这样系统就不只是一个 work graph，而是一个真正连接：

- 工作定义
- 能力结构
- 工程实现
- 运行调度
- 结果评估

的完整 AI Workforce Operating System 文档基础。


---

## 13. Formal Schema 扩展（Schema v1）

本节把 Implementation Layer 扩成正式 schema，目标是让数据、工程、评估三方都能围绕同一套实体与关系协作。

### 13.1 新增实体：Skill

虽然前文主链路中已经使用了 Skill，但为了让 Capability 与具体可复用技能单元分离，这里将 Skill 明确定义为独立实体。

#### Skill

建议字段：

- `skill_id`
- `canonical_name`
- `description`
- `skill_type`
- `parent_skill_id`
- `status`
- `version`

说明：

- `Capability` 更偏抽象能力接口
- `Skill` 更偏组织与工程可复用的能力单元
- 一个 capability 可以映射多个 skill，一个 skill 也可能复合多个 capability

---

### 13.2 Schema：SkillImplementation

#### 定义

表示某个 skill 的一个可执行实现版本。

#### 字段

- `skill_impl_id` TEXT PRIMARY KEY
- `canonical_name` TEXT NOT NULL
- `skill_id` TEXT NOT NULL
- `implementation_type` TEXT NOT NULL
- `primary_model_id` TEXT
- `prompt_template_id` TEXT
- `toolchain_id` TEXT
- `runtime_env_id` TEXT
- `ide_profile_id` TEXT
- `status` TEXT NOT NULL
- `version` TEXT NOT NULL
- `created_at` TIMESTAMP NOT NULL
- `updated_at` TIMESTAMP NOT NULL

#### 语义说明

- 一个 skill 可以有多个实现版本
- 多个实现版本可以用于 A/B test、成本优化、质量优化、模型替换
- `primary_model_id` 是便捷字段，完整多模型绑定仍通过 `ModelBinding` 表达

#### SQL 草案

```sql
CREATE TABLE skill (
  skill_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  description TEXT,
  skill_type TEXT,
  parent_skill_id TEXT,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);

CREATE TABLE skill_implementation (
  skill_impl_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  skill_id TEXT NOT NULL,
  implementation_type TEXT NOT NULL,
  primary_model_id TEXT,
  prompt_template_id TEXT,
  toolchain_id TEXT,
  runtime_env_id TEXT,
  ide_profile_id TEXT,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);
```

---

### 13.3 Schema：RoleImplementation

#### 定义

表示某个 role 的一个可执行实现版本。

#### 字段

- `role_impl_id` TEXT PRIMARY KEY
- `canonical_name` TEXT NOT NULL
- `role_id` TEXT NOT NULL
- `composition_type` TEXT NOT NULL
- `primary_worker_id` TEXT
- `status` TEXT NOT NULL
- `version` TEXT NOT NULL
- `created_at` TIMESTAMP NOT NULL
- `updated_at` TIMESTAMP NOT NULL

#### `composition_type` 建议值

- `single_skill`
- `multi_skill_pipeline`
- `supervisor_worker`
- `hybrid_human_ai`

#### SQL 草案

```sql
CREATE TABLE role_implementation (
  role_impl_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  role_id TEXT NOT NULL,
  composition_type TEXT NOT NULL,
  primary_worker_id TEXT,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);
```

---

### 13.4 Schema：IDEProfile

#### 定义

表示一类 IDE / 工作台配置，用于描述 agent 是否依赖特定工作环境。

#### 字段

- `ide_profile_id` TEXT PRIMARY KEY
- `canonical_name` TEXT NOT NULL
- `ide_type` TEXT NOT NULL
- `workspace_config_ref` TEXT
- `permissions_profile` JSON
- `status` TEXT NOT NULL
- `version` TEXT NOT NULL
- `created_at` TIMESTAMP NOT NULL
- `updated_at` TIMESTAMP NOT NULL

#### SQL 草案

```sql
CREATE TABLE ide_profile (
  ide_profile_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  ide_type TEXT NOT NULL,
  workspace_config_ref TEXT,
  permissions_profile JSON,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);
```

---

### 13.5 Schema：ModelBinding

#### 定义

表示某个实现绑定了哪些模型，以及各模型在实现中的职责。

#### 字段

- `model_binding_id` TEXT PRIMARY KEY
- `subject_type` TEXT NOT NULL
- `subject_id` TEXT NOT NULL
- `model_id` TEXT NOT NULL
- `binding_role` TEXT NOT NULL
- `temperature_profile` TEXT
- `context_window_profile` TEXT
- `status` TEXT NOT NULL
- `version` TEXT NOT NULL
- `created_at` TIMESTAMP NOT NULL
- `updated_at` TIMESTAMP NOT NULL

#### `subject_type` 建议值

- `skill_implementation`
- `role_implementation`
- `worker`

#### `binding_role` 建议值

- `primary_reasoner`
- `secondary_reasoner`
- `retriever`
- `reranker`
- `vision`
- `coder`
- `evaluator`

#### SQL 草案

```sql
CREATE TABLE model_binding (
  model_binding_id TEXT PRIMARY KEY,
  subject_type TEXT NOT NULL,
  subject_id TEXT NOT NULL,
  model_id TEXT NOT NULL,
  binding_role TEXT NOT NULL,
  temperature_profile TEXT,
  context_window_profile TEXT,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);
```

---

### 13.6 Schema：PromptTemplate

#### 定义

表示实现所使用的提示词模板。

#### 字段

- `prompt_template_id` TEXT PRIMARY KEY
- `canonical_name` TEXT NOT NULL
- `template_text` TEXT NOT NULL
- `template_type` TEXT NOT NULL
- `input_schema_ref` TEXT
- `output_schema_ref` TEXT
- `status` TEXT NOT NULL
- `version` TEXT NOT NULL
- `created_at` TIMESTAMP NOT NULL
- `updated_at` TIMESTAMP NOT NULL

#### SQL 草案

```sql
CREATE TABLE prompt_template (
  prompt_template_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  template_text TEXT NOT NULL,
  template_type TEXT NOT NULL,
  input_schema_ref TEXT,
  output_schema_ref TEXT,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);
```

---

### 13.7 Schema：Toolchain

#### 定义

表示一组工具能力集合。

#### 字段

- `toolchain_id` TEXT PRIMARY KEY
- `canonical_name` TEXT NOT NULL
- `description` TEXT
- `tool_list` JSON NOT NULL
- `permissions_profile` JSON
- `status` TEXT NOT NULL
- `version` TEXT NOT NULL
- `created_at` TIMESTAMP NOT NULL
- `updated_at` TIMESTAMP NOT NULL

#### SQL 草案

```sql
CREATE TABLE toolchain (
  toolchain_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  description TEXT,
  tool_list JSON NOT NULL,
  permissions_profile JSON,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);
```

---

### 13.8 Schema：RuntimeEnvironment

#### 定义

表示实际运行环境与安全边界。

#### 字段

- `runtime_env_id` TEXT PRIMARY KEY
- `canonical_name` TEXT NOT NULL
- `environment_type` TEXT NOT NULL
- `resource_limits` JSON
- `network_policy` TEXT
- `filesystem_policy` TEXT
- `status` TEXT NOT NULL
- `version` TEXT NOT NULL
- `created_at` TIMESTAMP NOT NULL
- `updated_at` TIMESTAMP NOT NULL

#### SQL 草案

```sql
CREATE TABLE runtime_environment (
  runtime_env_id TEXT PRIMARY KEY,
  canonical_name TEXT NOT NULL,
  environment_type TEXT NOT NULL,
  resource_limits JSON,
  network_policy TEXT,
  filesystem_policy TEXT,
  status TEXT NOT NULL,
  version TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);
```

---

## 14. Edge Taxonomy 扩展（Implementation Layer）

### 14.1 核心业务边

- `OCCUPATION_HAS_ROLE`
- `POSITION_INCLUDES_ROLE`
- `ROLE_REQUIRES_TASK`
- `TASK_REQUIRES_CAPABILITY`
- `CAPABILITY_REALIZED_BY_SKILL`

### 14.2 实现层边

- `SKILL_IMPLEMENTED_BY`
  - Skill → SkillImplementation
- `ROLE_IMPLEMENTED_BY`
  - Role → RoleImplementation
- `ROLE_IMPL_COMPOSED_OF_SKILL_IMPL`
  - RoleImplementation → SkillImplementation
- `SKILL_IMPL_POWERED_BY_MODEL`
  - SkillImplementation → Model
- `SKILL_IMPL_DRIVEN_BY_PROMPT`
  - SkillImplementation → PromptTemplate
- `SKILL_IMPL_USES_TOOLCHAIN`
  - SkillImplementation → Toolchain
- `SKILL_IMPL_RUNS_ON_ENV`
  - SkillImplementation → RuntimeEnvironment
- `SKILL_IMPL_USES_IDE_PROFILE`
  - SkillImplementation → IDEProfile
- `ROLE_IMPL_USES_IDE_PROFILE`
  - RoleImplementation → IDEProfile
- `WORKER_REALIZES_SKILL_IMPL`
  - Worker → SkillImplementation
- `WORKER_REALIZES_ROLE_IMPL`
  - Worker → RoleImplementation

### 14.3 运行时边

- `TASKRUN_ASSIGNED_TO_WORKER`
- `TASKRUN_EXECUTES_SKILL_IMPL`
- `TASKRUN_EXECUTES_ROLE_IMPL`
- `TASKRUN_PRODUCED_ARTIFACT`
- `TASKRUN_HAS_EVALUATION`

### 14.4 评估边

- `WORKER_HAS_CAPABILITY_SCORE`
- `SKILL_IMPL_HAS_EVALUATION`
- `ROLE_IMPL_HAS_EVALUATION`
- `MODEL_IMPACTS_SKILL_IMPL_PERFORMANCE`

---

## 15. 通用 Edge Meta 规范

所有重要边统一进入 `graph_edge` 表。

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
  updated_at TIMESTAMP NOT NULL,
  UNIQUE(edge_type, from_id, to_id, version)
);
```

建议补充 `edge_evidence` 表：

```sql
CREATE TABLE edge_evidence (
  edge_evidence_id TEXT PRIMARY KEY,
  edge_id TEXT NOT NULL,
  evidence_type TEXT NOT NULL,
  source_uri TEXT,
  excerpt TEXT,
  created_at TIMESTAMP NOT NULL
);
```

---

## 16. 推荐查询主链路

### 16.1 从业务定义到实现

`Occupation -> Role -> TaskTemplate -> Capability -> Skill -> SkillImplementation -> Worker`

### 16.2 从工程实现反推业务含义

`Model / PromptTemplate / IDEProfile / Toolchain -> SkillImplementation -> RoleImplementation -> Role -> PositionTemplate -> Occupation`

### 16.3 从故障定位到责任归因

`Failed TaskRun -> Worker -> SkillImplementation -> PromptTemplate / ModelBinding / RuntimeEnvironment`

这样可以区分失败原因来自：

- role 定义不清
- task 设计不合理
- skill implementation 不稳定
- prompt 退化
- model 替换影响
- toolchain 故障
- runtime 限制

---

## 17. 面向团队协作的 PRD / 架构说明（以终为始）

本节用更偏产品和架构的语言，把系统最终形态写清楚，方便面向产品、平台、算法、业务团队共识。

### 17.1 产品愿景

AWOS 的目标不是做一个更强的聊天入口，而是建立一套：

> **让 AI 能像组织中的劳动力一样被定义、调用、编排、评估和持续优化的系统。**

这意味着系统最终管理的不是“模型调用”，而是：

- 工作定义
- 角色分工
- 能力供给
- 实现单元
- 任务执行
- 结果评估

---

### 17.2 终态用户体验

在终态系统中，业务方不会直接说“调用某个模型”。

业务方会说：

- 需要一个 Research Analyst 角色
- 需要一个 SQL Reporting 角色
- 需要一个 Customer Triage 角色
- 需要一个能完成 Partner Outreach 的 AI workforce

系统会自动完成以下动作：

1. 把需求映射到 position / role / task
2. 识别每个 task 所需的 capability
3. 找到最合适的 skill / role implementation
4. 选择合适的 worker 执行
5. 收集结果、产出 artifact
6. 做质量、成本、时延、稳定性评估
7. 持续优化实现版本

---

### 17.3 核心用户对象

#### 业务用户

关心：

- 我需要什么角色
- 系统能否完成工作
- 结果质量是否稳定

#### 平台 / 工程团队

关心：

- role / skill 的实现结构
- model、prompt、toolchain 如何治理
- 如何替换实现而不影响上层语义

#### 算法 / 评估团队

关心：

- 哪些实现表现更好
- 哪些能力仍不稳定
- 哪些任务应由人机协作完成

#### 管理者 / 组织设计者

关心：

- 哪些岗位已能被 AI workforce 覆盖
- 哪些任务最适合自动化
- 成本和产出如何最优配置

---

### 17.4 核心产品能力

#### 能力一：工作定义

系统支持定义：

- Occupation
- PositionTemplate
- Role
- TaskTemplate
- Capability
- Skill

#### 能力二：实现治理

系统支持定义：

- SkillImplementation
- RoleImplementation
- PromptTemplate
- ModelBinding
- Toolchain
- RuntimeEnvironment
- IDEProfile

#### 能力三：执行编排

系统支持：

- WorkOrder 创建
- Task decomposition
- Worker selection
- Multi-implementation routing
- Runtime execution

#### 能力四：评估闭环

系统支持：

- Task-level evaluation
- Skill-level evaluation
- Role-level evaluation
- Worker-level evaluation
- 成本 / 时延 / 成功率 / 可靠性监控

---

### 17.5 关键产品原则

#### 原则一：语义稳定，实现可替换

Role / Task / Skill 是组织语义，不应随模型更换而频繁变化。

Model / Prompt / Toolchain / IDE 是实现细节，应可替换。

#### 原则二：先定义工作，再绑定实现

任何 agent 的设计都应先回答：

- 它承担什么 role
- 它完成什么 task
- 它依赖什么 capability

再回答：

- 它用什么模型
- 它用什么 prompt
- 它跑在哪个 IDE / runtime 上

#### 原则三：以评估驱动实现演进

系统不假设某个实现永远最优，而是通过 evaluation 持续比较：

- 哪个实现更便宜
- 哪个实现更快
- 哪个实现质量更高
- 哪个实现更稳定

---

### 17.6 核心运行流程

#### Step 1: 定义工作

组织定义一个 position / role。

#### Step 2: 定义任务与能力

系统把 role 分解为 task，并为 task 绑定 capability / skill。

#### Step 3: 绑定实现

平台团队为 skill / role 提供一个或多个 implementations。

#### Step 4: 运行执行

调度器根据目标、约束、成本、时延、质量要求选择合适实现与 worker。

#### Step 5: 生成产出

系统输出 artifact，例如：

- report
- code patch
- SQL result
- email draft
- knowledge summary

#### Step 6: 评估与优化

系统记录 execution run 与 evaluation，反哺 capability score 与 routing 策略。

---

### 17.7 终态系统中的 agent 定义

在 PRD 视角下，agent 的定义应被写成：

> **Agent 是面向组织暴露的工作实体，内部由 role implementations 与 skill implementations 组成。**

这意味着：

- agent 是产品层对象
- skill implementation 是工程层对象
- capability 是抽象能力层对象
- role 是组织语义层对象

这样才能同时满足传播、产品设计和工程治理。

---

### 17.8 示例：Research Analyst Agent

#### 对外定义

- PositionTemplate: AI Research Analyst
- Role: Research Synthesis
- 产出：Research Report

#### 对内实现

- SkillImplementation: web_search_skill_v2
- SkillImplementation: source_validation_skill_v1
- SkillImplementation: synthesis_skill_v3
- SkillImplementation: report_drafting_skill_v2

#### 实现依赖

- IDEProfile: Research IDE
- ModelBinding: GPT-5 as primary_reasoner
- PromptTemplate: research_planner_prompt_v3
- Toolchain: browser + python + docs
- RuntimeEnvironment: secure container

#### 运行结果

- TaskRun 完成
- 输出 report artifact
- Evaluation score 0.89
- 成本 1.7 units
- 时延 42s

---

### 17.9 MVP 范围建议

第一阶段产品不追求完整组织仿真，只做最小闭环：

1. 定义 Role / Task / Capability / Skill
2. 接入 SkillImplementation / RoleImplementation
3. 管理 Prompt / Model / Toolchain / Runtime
4. 能执行 WorkOrder
5. 能记录 TaskRun 与 Evaluation
6. 能基于评估做多实现选择

这样就已经是一个真正可工作的 AI workforce 平台雏形。

---

### 17.10 一句话定义 PRD 版本

这份 PRD / 架构说明的核心句子可以定为：

> **AWOS 不是一个 agent 集合，而是一套把工作语义、能力抽象、实现治理、执行编排和结果评估统一起来的 AI 劳动力操作系统。**

