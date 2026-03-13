
# 岗位-职位-技能图谱系统规范（Spec v1.0）

## 1. 目标

构建一套统一的AI  workforce能力模型，支持以下核心能力：

1. 标准化表示 **职业 / 职位 / 岗位 / 技能**
2. 建立 **职位-岗位-技能** 映射关系
3. 从 JD、简历、课程等文本中抽取技能
4. 支持人岗匹配、技能推荐、职业路径分析
5. 支持动态图谱更新与版本管理

---

## 2. 设计原则

### 2.1 以终为始

系统最终服务于 4 类场景：

* 招聘匹配
* AI  workforce盘点
* 学习推荐
* 课程/岗位设计

### 2.2 标准优先，市场补充

优先采用权威标准数据作为骨架，再用市场数据补足动态变化。

* 标准层：O*NET / OaSIS / ESCO
* 市场层：招聘 JD / 简历 / 课程 / 社区文本

### 2.3 先统一语义，再做算法

先定义清楚概念、主键、关系、字段和置信度，再做模型与推荐。

### 2.4 所有关系可追溯

每个结论都必须能追溯到：

* 来源数据
* 提取方式
* 置信度
* 版本号
* 生效时间

---

## 3. 概念定义

## 3.1 Occupation（职业）

社会层面通用职业类别，不依赖具体公司。

例：

* 软件工程师
* 数据分析师
* 产品经理

特点：

* 粒度较粗
* 适合做标准化映射和宏观统计

---

## 3.2 Position（职位）

组织中的职位头衔，是职业在某个组织里的具体实例。

定义公式：

**Position = Occupation + Organization + Level + Functional Focus**

例：

* 腾讯 T9 Java 后端开发工程师
* 字节跳动 高级数据分析师
* 某银行 风控产品经理

---

## 3.3 Job / Role（岗位）

一组稳定职责与任务的集合，通常与 JD 直接对应。

例：

* 负责订单系统后端开发
* 负责推荐模型训练与部署
* 负责用户增长实验设计

说明：
在本 spec 中，**Job** 表示具体职责层，不强调组织头衔，强调“做什么”。

---

## 3.4 Task（任务）

岗位职责中可拆解的最小工作单元。

例：

* 设计微服务接口
* 编写 ETL 脚本
* 监控模型效果

---

## 3.5 Skill（技能）

完成某项任务所需的可习得能力，包括硬技能、软技能和工具能力。

分为：

* Hard Skill：Python、SQL、Kubernetes
* Soft Skill：沟通、协作、问题解决
* Domain Skill：风控建模、财务分析
* Tool Skill：Tableau、Excel、Figma

---

## 3.6 Knowledge / Ability / Tool

为了兼容 O*NET 等标准，可以扩展为：

* Knowledge：知识
* Skill：技能
* Ability：能力
* Tool / Technology：工具或技术栈

但 v1 产品统一收敛为 **Skill Canonical Layer**，对外先都以 Skill 统一表达。

---

# 4. 系统边界

## 4.1 In Scope

本期纳入：

* 职业标准化
* 职位解析
* 岗位职责抽取
* 技能抽取
* 技能标准化
* 岗位技能画像
* 候选人技能画像
* 基础匹配分计算
* 技能缺口分析
* 图谱版本管理

## 4.2 Out of Scope

本期不纳入：

* 面试问答生成
* 复杂绩效评估
* 薪酬预测
* 劳动法规合规判断
* 自动生成完整组织架构

---

# 5. 核心实体模型

## 5.1 Occupation

字段：

* `occupation_id`
* `canonical_name`
* `aliases`
* `description`
* `taxonomy_source`
* `taxonomy_code`
* `parent_occupation_id`
* `status`
* `version`

示例：

```json
{
  "occupation_id": "occ_software_engineer",
  "canonical_name": "Software Engineer",
  "aliases": ["Software Developer", "软件工程师"],
  "taxonomy_source": "ONET",
  "taxonomy_code": "15-1252.00",
  "status": "active",
  "version": "2026.01"
}
```

---

## 5.2 Position

字段：

* `position_id`
* `raw_title`
* `canonical_title`
* `occupation_id`
* `organization_name`
* `level`
* `employment_type`
* `location`
* `language`
* `source_id`
* `status`

说明：
`raw_title` 保留原始职位名，`canonical_title` 为标准化后的职位名。

---

## 5.3 JobRole

字段：

* `job_role_id`
* `position_id`
* `role_summary`
* `responsibility_blocks`
* `seniority_scope`
* `function_area`
* `domain`
* `status`

---

## 5.4 Task

字段：

* `task_id`
* `job_role_id`
* `task_text`
* `task_type`
* `importance_score`
* `frequency_score`
* `source_id`

---

## 5.5 Skill

字段：

* `skill_id`
* `canonical_name`
* `aliases`
* `skill_type`
* `skill_cluster`
* `parent_skill_id`
* `description`
* `status`
* `version`

示例：

```json
{
  "skill_id": "skill_python",
  "canonical_name": "Python",
  "aliases": ["Python3", "Python programming"],
  "skill_type": "hard_skill",
  "skill_cluster": "programming_language",
  "status": "active",
  "version": "2026.01"
}
```

---

# 6. 核心关系模型

## 6.1 Occupation-Skill

表示标准职业与技能的关系。

字段：

* `occupation_id`
* `skill_id`
* `importance`
* `level`
* `evidence_source`
* `confidence`
* `version`

---

## 6.2 Position-Skill

表示某个职位实际需要的技能。

字段：

* `position_id`
* `skill_id`
* `weight`
* `skill_role_type`
* `confidence`
* `source_type`
* `evidence_span`
* `version`

`skill_role_type` 枚举：

* `core`
* `complementary`
* `optional`
* `transversal`

---

## 6.3 Task-Skill

表示完成某任务所需技能。

字段：

* `task_id`
* `skill_id`
* `weight`
* `confidence`
* `source_type`

---

## 6.4 Skill-Skill

表示技能之间的关系。

字段：

* `from_skill_id`
* `to_skill_id`
* `relation_type`
* `weight`
* `version`

`relation_type` 枚举：

* `similar_to`
* `prerequisite_of`
* `part_of`
* `tool_for`
* `substitute_for`
* `co_occurs_with`

---

# 7. 统一标识规范

## 7.1 ID 规范

必须全局唯一、稳定、可重建。

建议前缀：

* `occ_`：职业
* `pos_`：职位
* `role_`：岗位
* `task_`：任务
* `skill_`：技能
* `src_`：来源
* `edge_`：关系

---

## 7.2 Canonical Name 规范

每个实体必须有唯一标准名：

* 保留原始名称 `raw_*`
* 映射标准名称 `canonical_*`
* 保留别名 `aliases`

例如：

* raw: `Sr. Backend Dev (Java)`
* canonical: `Senior Backend Engineer`

---

# 8. 数据来源规范

## 8.1 数据源分层

### A. Authority Source

权威标准源

* O*NET
* OaSIS
* ESCO

用途：

* 职业分类
* 标准技能骨架
* 基础重要度评分

### B. Market Source

市场动态源

* JD
* 招聘平台
* 企业岗位库
* 课程目录

用途：

* 新兴技能
* 细粒度岗位画像
* 地区/行业差异

### C. User Source

用户数据源

* 简历
* 用户填写资料
* 个人项目经历

用途：

* 候选人技能画像

---

## 8.2 Source 元数据字段

每条事实必须记录：

* `source_id`
* `source_type`
* `source_name`
* `source_url`
* `ingested_at`
* `license`
* `language`
* `region`

---

# 9. 抽取与标准化规范

## 9.1 职位解析流程

输入原始职位标题：

`腾讯 T9 Java 后端开发工程师`

解析输出：

* organization_name = 腾讯
* level = T9
* functional_focus = Java 后端
* occupation = Software Engineer
* canonical_title = Senior Backend Engineer

---

## 9.2 技能抽取

输入文本：

“熟悉 Python、SQL，具备 Airflow 和 Spark 使用经验”

输出：

* Python
* SQL
* Airflow
* Spark

每个技能必须带：

* `skill_id`
* `mention_text`
* `offset_start`
* `offset_end`
* `confidence`

---

## 9.3 技能标准化

目标：不同写法归一到统一技能。

例如：

* `Py`
* `Python3`
* `Python编程`

统一映射到：

* `skill_python`

标准化策略优先级：

1. 字典精确匹配
2. 别名词典
3. 规则归一
4. embedding 相似度
5. LLM 归一判定

---

# 10. 权重与置信度规范

## 10.1 Weight

表示某技能对职位/岗位的重要性，范围：

`0.0 ~ 1.0`

建议含义：

* 0.9~1.0：核心不可缺
* 0.7~0.9：高相关
* 0.4~0.7：补充相关
* 0.1~0.4：弱相关

---

## 10.2 Confidence

表示系统对结论的确信程度，范围：

`0.0 ~ 1.0`

来源可综合：

* 来源可靠性
* 提取模型置信度
* 多源一致性
* 人工校验状态

---

## 10.3 Human Verified

关键关系支持审核状态：

* `unreviewed`
* `reviewed`
* `approved`
* `rejected`

---

# 11. 图谱更新规范

## 11.1 更新模式

支持两种模式：

### 批量更新

按日/周重建部分边

### 增量更新

新 JD / 简历进来后局部更新

---

## 11.2 版本机制

所有核心实体与关系必须支持版本号：

格式建议：

`YYYY.MM.patch`

例如：

* `2026.03.1`

---

## 11.3 生效时间

每条关系记录：

* `valid_from`
* `valid_to`

用于支持历史回溯和趋势分析。

---

# 12. 匹配规范

## 12.1 候选人技能画像

候选人画像由以下信息构成：

* 显式技能
* 经历反推技能
* 项目反推技能
* 课程与证书
* 技能熟练度

输出：

```json
{
  "candidate_id": "cand_001",
  "skills": [
    {"skill_id": "skill_python", "proficiency": 0.88},
    {"skill_id": "skill_sql", "proficiency": 0.81}
  ]
}
```

---

## 12.2 职位匹配分

匹配分范围：

`0 ~ 100`

建议构成：

* 核心技能覆盖：50%
* 补充技能覆盖：20%
* 相关经验映射：20%
* 软技能匹配：10%

示例：

`MatchScore = 100 * (0.5 * CoreCoverage + 0.2 * ComplementCoverage + 0.2 * ExperienceFit + 0.1 * SoftSkillFit)`

---

## 12.3 缺口分析

输出必须能解释：

* 已满足技能
* 缺失技能
* 邻近技能
* 推荐学习路径

---

# 13. API 规范

## 13.1 查询职位画像

`GET /positions/{position_id}/profile`

返回：

* 标准职位名
* 关联职业
* 核心技能
* 补充技能
* 任务列表
* 版本号

---

## 13.2 查询技能画像

`GET /skills/{skill_id}`

返回：

* 标准名
* 别名
* 技能类型
* 上下游关系
* 相关职位
* 趋势信息

---

## 13.3 文本技能抽取

`POST /extract/skills`

输入：

```json
{
  "text": "熟悉 Python、SQL，有 Airflow 使用经验"
}
```

输出：

```json
{
  "skills": [
    {"skill_id": "skill_python", "canonical_name": "Python", "confidence": 0.98},
    {"skill_id": "skill_sql", "canonical_name": "SQL", "confidence": 0.96},
    {"skill_id": "skill_airflow", "canonical_name": "Apache Airflow", "confidence": 0.92}
  ]
}
```

---

## 13.4 人岗匹配

`POST /match/candidate-position`

输入：

* candidate_profile
* position_id

输出：

* match_score
* missing_skills
* matched_skills
* explanations

---

# 14. 质量指标（KPI）

## 14.1 数据质量

* 技能标准化准确率
* 重复实体率
* 无效边比例
* 缺失字段率

## 14.2 模型质量

* 技能抽取 Precision / Recall / F1
* 技能归一准确率
* 职位归类准确率
* 匹配结果人工满意度

## 14.3 产品质量

* 匹配解释可读性
* 推荐采纳率
* 查询响应时间
* 图谱更新延迟

---

# 15. 非功能性要求

## 15.1 可解释

所有匹配与推荐必须可解释，不能只给分数。

## 15.2 可扩展

支持新行业、新语言、新地区接入。

## 15.3 可审计

所有边与实体变更可追踪。

## 15.4 多语言兼容

至少支持：

* 中文
* 英文

字段支持：

* `canonical_name_en`
* `canonical_name_zh`

---

# 16. 最小可用版本（MVP）

v1 只做这 6 件事：

1. 定义职业/职位/岗位/技能统一模型
2. 接入 1 个权威标准库
3. 接入 1 类 JD 数据
4. 完成技能抽取与标准化
5. 生成岗位技能画像
6. 输出基础人岗匹配结果

---

# 17. 验收标准

系统上线前，至少满足：

### 数据层

* 80% 以上职位可成功映射到标准职业
* 85% 以上技能 mention 可归一到 canonical skill

### 模型层

* 技能抽取 F1 ≥ 0.85
* 技能归一准确率 ≥ 0.90

### 业务层

* Top 20 重点岗位可生成稳定画像
* 匹配结果可输出缺口解释
* 用户能看到“为什么匹配/不匹配”

---

# 18. 一句话定义这套 spec

这套 spec 的本质是：

**把“职业、职位、岗位、任务、技能”定义成一套可追溯、可计算、可更新的统一语义系统。**

---

如果你愿意，我下一步可以直接继续补两份最有用的东西之一：

1. **ER 图 + 表结构 SQL spec**
2. **API spec（OpenAPI 风格）**
