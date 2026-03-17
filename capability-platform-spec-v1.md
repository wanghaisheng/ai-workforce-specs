# Capability Platform Spec v1.0

## 1. 目标

Capability Platform 是 AWOS 中负责管理和分发可执行工作能力的基础平台。

它的职责不是存“模型”或“prompt”，而是存：

- `Skill Implementation`
- `Role Implementation`

以及这些实现的：

- 输入输出契约
- 依赖关系
- 运行约束
- 评估结果
- 版本状态
- 路由元信息

一句话定义：

> **Capability Platform 是 Work OS 中的能力注册、发布、发现、评估与路由平台。**

---

## 2. 核心设计原则

### 2.1 工作语义优先
平台的基本流通单位不是 API endpoint，而是能完成某类工作的实现单元。

### 2.2 抽象与实现分离
必须区分：

- `Capability / Skill / Role`：抽象定义
- `Implementation`：具体实现

### 2.3 多实现并存
同一个 skill / role 可以有多个实现版本同时在线。

### 2.4 可比较、可替换、可治理
所有实现都必须能被：

- benchmark
- route
- approve / reject
- retire

### 2.5 运行时可选择
平台不仅支持“发布和存储”，还支持“基于约束的动态选择”。

---

## 3. 对象模型

Capability Platform 的核心对象分 3 层：

- Definition Layer
- Implementation Layer
- Operational Layer

### 3.1 Definition Layer

#### 3.1.1 Capability

表示抽象能力接口。

```json
{
  "capability_id": "cap_sql_query",
  "canonical_name": "SQL Query",
  "description": "Ability to write and execute SQL queries against structured data.",
  "capability_type": "data_operation",
  "input_schema_ref": "schema_sql_query_input_v1",
  "output_schema_ref": "schema_sql_query_output_v1",
  "status": "active",
  "version": "2026.03.1"
}
```

#### 3.1.2 Skill

表示组织和工程层面的可复用能力单元。

```json
{
  "skill_id": "skill_sql_analysis",
  "canonical_name": "SQL Analysis",
  "description": "Analyze data using SQL and structured summarization.",
  "skill_type": "hard_skill",
  "capability_refs": ["cap_sql_query", "cap_table_summarization"],
  "status": "active",
  "version": "2026.03.1"
}
```

#### 3.1.3 Role

表示更高层的职责抽象。

```json
{
  "role_id": "role_research_analyst",
  "canonical_name": "Research Analyst",
  "description": "Produces structured research outputs from multi-source evidence.",
  "required_skill_refs": [
    "skill_web_search",
    "skill_source_validation",
    "skill_synthesis",
    "skill_report_drafting"
  ],
  "status": "active",
  "version": "2026.03.1"
}
```

### 3.2 Implementation Layer

这是 Capability Platform 的核心。

#### 3.2.1 SkillImplementation

```json
{
  "skill_impl_id": "skillimpl_sql_analysis_v3",
  "canonical_name": "SQL Analysis Skill v3",
  "implementation_type": "tool_orchestrated",
  "skill_id": "skill_sql_analysis",
  "provides_capabilities": [
    "cap_sql_query",
    "cap_table_summarization"
  ],
  "input_schema_ref": "schema_sql_query_input_v1",
  "output_schema_ref": "schema_sql_query_output_v1",
  "model_bindings": ["mb_001", "mb_002"],
  "prompt_template_refs": ["prompt_sql_planner_v2"],
  "toolchain_ref": "toolchain_sql_python_v1",
  "runtime_env_ref": "runtime_secure_container_v1",
  "ide_profile_ref": null,
  "owner_team": "analytics-platform",
  "publish_status": "published",
  "lifecycle_stage": "production",
  "version": "3.1.0"
}
```

字段建议：

- `skill_impl_id`
- `canonical_name`
- `implementation_type`
- `skill_id`
- `provides_capabilities[]`
- `input_schema_ref`
- `output_schema_ref`
- `model_bindings[]`
- `prompt_template_refs[]`
- `toolchain_ref`
- `runtime_env_ref`
- `ide_profile_ref`
- `owner_team`
- `publish_status`
- `lifecycle_stage`
- `version`

`implementation_type` 建议值：

- `prompt_workflow`
- `tool_orchestrated`
- `code_executor`
- `ide_agent`
- `hybrid`

`publish_status` 建议值：

- `draft`
- `submitted`
- `published`
- `deprecated`
- `retired`

`lifecycle_stage` 建议值：

- `experimental`
- `staging`
- `production`

#### 3.2.2 RoleImplementation

```json
{
  "role_impl_id": "roleimpl_research_analyst_v2",
  "canonical_name": "Research Analyst Role v2",
  "role_id": "role_research_analyst",
  "composition_type": "multi_skill_pipeline",
  "skill_impl_refs": [
    "skillimpl_web_search_v2",
    "skillimpl_source_validation_v1",
    "skillimpl_synthesis_v3",
    "skillimpl_report_drafting_v2"
  ],
  "orchestration_policy_ref": "orch_research_standard_v1",
  "input_schema_ref": "schema_research_request_v1",
  "output_schema_ref": "schema_research_report_v1",
  "owner_team": "research-platform",
  "publish_status": "published",
  "lifecycle_stage": "production",
  "version": "2.0.0"
}
```

#### 3.2.3 Supporting Objects

Role/Skill implementation 依赖这些辅助对象：

- `PromptTemplate`
- `Toolchain`
- `RuntimeEnvironment`
- `IDEProfile`
- `ModelBinding`
- `EvaluationProfile`
- `PolicyProfile`

### 3.3 Operational Layer

这是平台运行时要维护的对象。

#### 3.3.1 BenchmarkProfile

定义某类实现应该怎么测。

```json
{
  "benchmark_profile_id": "bench_sql_analysis_v1",
  "subject_type": "skill_implementation",
  "task_type": "sql_analysis",
  "metric_weights": {
    "accuracy": 0.5,
    "latency": 0.2,
    "cost": 0.1,
    "format_correctness": 0.2
  },
  "dataset_ref": "dataset_sql_benchmark_v1",
  "version": "1.0.0"
}
```

#### 3.3.2 ImplementationScore

```json
{
  "implementation_score_id": "is_001",
  "subject_type": "skill_implementation",
  "subject_id": "skillimpl_sql_analysis_v3",
  "overall_score": 0.91,
  "quality_score": 0.95,
  "cost_score": 0.72,
  "latency_score": 0.83,
  "reliability_score": 0.89,
  "sample_size": 328,
  "window": "30d",
  "updated_at": "2026-03-17T00:00:00Z"
}
```

#### 3.3.3 RoutePolicy

```json
{
  "route_policy_id": "route_sql_default_v1",
  "target_type": "capability",
  "target_id": "cap_sql_query",
  "selection_strategy": "weighted_best_fit",
  "constraints": {
    "min_quality_score": 0.9,
    "max_cost_per_run": 0.5,
    "max_latency_ms": 8000
  },
  "fallback_order": [
    "skillimpl_sql_analysis_v3",
    "skillimpl_sql_analysis_v2"
  ],
  "version": "1.0.0"
}
```

---

## 4. 发布协议

Capability Platform 必须支持从草稿到生产的完整发布生命周期。

### 4.1 发布流程

建议标准流程：

```text
Draft
→ Validate
→ Submit
→ Benchmark
→ Review
→ Publish
→ Route Eligible
```

### 4.2 发布步骤定义

#### 4.2.1 Draft
开发者创建 implementation 草稿。

要求最少提供：

- 基本元信息
- skill / role 绑定
- input/output schema
- 依赖声明

#### 4.2.2 Validate
平台自动校验：

- schema 完整性
- 依赖对象存在性
- 输入输出契约合法性
- 安全策略兼容性

校验失败不得提交。

#### 4.2.3 Submit
提交后进入候选状态。

#### 4.2.4 Benchmark
运行基准测试：

- 质量
- 时延
- 成本
- 成功率
- 格式正确率

#### 4.2.5 Review
人工或策略审核：

- 业务有效性
- 风险级别
- 合规要求
- 是否可进 production

#### 4.2.6 Publish
发布进入 registry，可被 discover。

#### 4.2.7 Route Eligible
只有满足条件的实现才能被默认路由选中。

### 4.3 发布对象清单

发布一个 `SkillImplementation` 必须带：

- Manifest
- Input schema
- Output schema
- Dependency declarations
- Evaluation profile
- Ownership metadata
- Version
- Change log

### 4.4 Manifest 规范

建议定义统一 manifest 格式，例如：

```yaml
kind: skill_implementation
id: skillimpl_sql_analysis_v3
name: SQL Analysis Skill v3
skill_id: skill_sql_analysis
version: 3.1.0
implementation_type: tool_orchestrated

provides_capabilities:
  - cap_sql_query
  - cap_table_summarization

contracts:
  input_schema_ref: schema_sql_query_input_v1
  output_schema_ref: schema_sql_query_output_v1

dependencies:
  model_bindings:
    - mb_sql_primary_v2
  prompt_templates:
    - prompt_sql_planner_v2
  toolchain_ref: toolchain_sql_python_v1
  runtime_env_ref: runtime_secure_container_v1

policies:
  safety_class: standard
  review_required: false

ownership:
  owner_team: analytics-platform
  maintainer: team-analytics-ai

lifecycle:
  publish_status: submitted
  lifecycle_stage: staging
```

---

## 5. 版本规则

实现平台如果没有强版本规则，很快会失控。

### 5.1 版本对象

以下对象必须版本化：

- Capability
- Skill
- Role
- SkillImplementation
- RoleImplementation
- PromptTemplate
- Toolchain
- RuntimeEnvironment
- RoutePolicy
- BenchmarkProfile

### 5.2 版本格式

建议：

- Definitions 使用：`YYYY.MM.patch`
- Implementations 使用：`MAJOR.MINOR.PATCH`

例如：

- `Skill version = 2026.03.1`
- `SkillImplementation version = 3.1.0`

### 5.3 SemVer 规则

#### MAJOR
破坏性变更：

- 输入 schema 改变
- 输出 schema 不兼容
- 行为语义变化

#### MINOR
非破坏性增强：

- 提高质量
- 增加可选字段
- 优化 routing metadata

#### PATCH
修复性改动：

- prompt 微调
- bug fix
- 文案修复
- 小幅性能优化

### 5.4 Tag 规则

建议支持：

- `latest`
- `stable`
- `experimental`
- `production`
- `canary`

例如：

```text
skillimpl_sql_analysis_v3@stable
skillimpl_sql_analysis_v4@canary
```

### 5.5 兼容性规则

发布新版本时必须声明：

- `backward_compatible: true|false`
- `replaces_version`
- `migration_notes`

### 5.6 生命周期规则

实现状态建议：

```text
draft
submitted
validated
published
deprecated
retired
```

补充运行级阶段：

```text
experimental
staging
production
```

---

## 6. 评分机制

Capability Platform 不只是 catalog，必须能“比较实现”。

### 6.1 评分对象

可评分对象包括：

- SkillImplementation
- RoleImplementation
- Worker
- Capability fulfillment quality

### 6.2 评分维度

#### 6.2.1 Quality Score
质量分，来源于 benchmark 和在线评估。

子指标可包括：

- accuracy
- completeness
- factuality
- format correctness
- task success

#### 6.2.2 Cost Score
成本分。

来源：

- token cost
- tool cost
- runtime cost
- human review cost

#### 6.2.3 Latency Score
时延分。

来源：

- p50
- p95
- timeout rate

#### 6.2.4 Reliability Score
稳定性分。

来源：

- failure rate
- retry rate
- incident count

#### 6.2.5 Safety / Risk Score
风险分。

来源：

- policy violation rate
- unsafe output frequency
- review fail rate

### 6.3 总分公式

建议默认公式：

```text
overall_score =
  w1 * quality_score +
  w2 * reliability_score +
  w3 * latency_score +
  w4 * cost_score +
  w5 * safety_score
```

默认可设：

```text
quality = 0.4
reliability = 0.2
latency = 0.15
cost = 0.15
safety = 0.1
```

不同 task type 可覆盖。

### 6.4 在线与离线分离

#### Offline Score
来自 benchmark 数据集。

#### Online Score
来自真实 task run。

#### Blended Score
最终用于 routing。

建议：

```text
blended_score =
  0.4 * offline_score + 0.6 * online_score
```

### 6.5 评分窗口

建议每个 score 都带时间窗口：

- `24h`
- `7d`
- `30d`
- `all_time`

### 6.6 置信度

评分必须带：

- `sample_size`
- `confidence_interval`
- `freshness`

样本太少时不得默认用于 production routing。

---

## 7. 路由接口

Capability Platform 的终极价值是：  
**根据任务约束，返回最合适的 implementation。**

### 7.1 路由目标

路由可以发生在三层：

#### Capability Routing
给一个 capability 找最佳 skill implementation

#### Skill Routing
给一个 skill 找最佳 implementation

#### Role Routing
给一个 role 找最佳 role implementation

### 7.2 核心路由输入

一个标准路由请求至少包括：

- `target_type`
- `target_id`
- `task_context`
- `constraints`
- `preferences`

例如：

```json
{
  "target_type": "capability",
  "target_id": "cap_sql_query",
  "task_context": {
    "task_type": "sql_analysis",
    "domain": "sales",
    "risk_level": "standard"
  },
  "constraints": {
    "max_cost_per_run": 0.5,
    "max_latency_ms": 8000,
    "min_quality_score": 0.9
  },
  "preferences": {
    "optimize_for": "quality",
    "lifecycle_stage": "production"
  }
}
```

### 7.3 路由输出

```json
{
  "selected_subject_type": "skill_implementation",
  "selected_subject_id": "skillimpl_sql_analysis_v3",
  "selection_reason": {
    "quality_score": 0.95,
    "cost_score": 0.72,
    "latency_score": 0.83,
    "constraint_satisfaction": true
  },
  "fallback_candidates": [
    "skillimpl_sql_analysis_v2",
    "skillimpl_sql_analysis_v1"
  ],
  "route_policy_id": "route_sql_default_v1"
}
```

### 7.4 路由策略类型

建议支持这些策略：

- `highest_quality`
- `lowest_cost`
- `lowest_latency`
- `weighted_best_fit`
- `policy_constrained_best_fit`
- `canary_split`
- `manual_pin`

### 7.5 Fallback 规则

当主实现不可用时：

1. 优先选同 lifecycle stage 的兼容版本
2. 其次选 lower score 但满足硬约束的版本
3. 再次触发 human review / escalation
4. 最后返回 no_route_available

### 7.6 路由接口草案

#### POST `/route/select`

请求：

```json
{
  "target_type": "skill",
  "target_id": "skill_sql_analysis",
  "constraints": {
    "max_cost_per_run": 0.5,
    "min_quality_score": 0.9
  },
  "preferences": {
    "optimize_for": "quality"
  }
}
```

响应：

```json
{
  "selected_id": "skillimpl_sql_analysis_v3",
  "selected_version": "3.1.0",
  "fallbacks": [
    {
      "id": "skillimpl_sql_analysis_v2",
      "version": "2.4.0"
    }
  ]
}
```

#### GET `/implementations/{id}`

返回实现详情、依赖、评分、状态。

#### POST `/implementations/publish`

提交新版本。

#### POST `/implementations/benchmark`

触发 benchmark。

#### GET `/implementations/{id}/scores`

查看分数。

#### GET `/catalog/search`

按 capability / role / domain 搜索。

---

## 8. 治理规则

Capability Platform 必须有治理，不然会退化成 prompt 仓库。

### 8.1 权限层级

建议角色：

- `author`
- `reviewer`
- `publisher`
- `router_admin`
- `platform_admin`

### 8.2 发布门禁

进入 production 至少要满足：

- schema 校验通过
- benchmark 达标
- 依赖合法
- 风险策略通过
- owner 明确
- 回滚路径明确

### 8.3 审计要求

所有关键动作都记录：

- created
- updated
- submitted
- benchmarked
- approved
- published
- deprecated
- retired
- routed

---

## 9. 最小可用版本（MVP）

Capability Platform v1 建议只做这些：

### 对象
- Capability
- Skill
- Role
- SkillImplementation
- RoleImplementation

### 核心能力
- 发布
- 搜索
- 版本管理
- 基础评分
- 简单路由

### 暂缓
- 复杂 marketplace 交易逻辑
- 收费分账
- 自动策略学习
- 跨组织共享治理

---

## 10. 一句话收束

**Capability Platform 不是模型目录，不是 prompt 仓库，也不是普通 registry。**

它的本质是：

> **Work OS 中用于分发、比较、选择和治理“可执行工作能力”的供应平台。**

你可以把它理解成：

> **npm registry for work capabilities**

但更准确一点是：

> **A registry + marketplace + routing plane for skill and role implementations.**
