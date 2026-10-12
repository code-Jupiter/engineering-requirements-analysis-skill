# 增量重规划协议

## 适用范围

当当前会话已经形成或确认过方案，或已经执行并产生过程制品，用户又提出新目标、补充要求、范围调整或纠正时，必须使用本协议。

这不是第三条工作流，也不是新的原子能力，而是两条标准工作流和工作流外动态方案共用的计划状态管理规则。

## 核心原则

- 新消息默认是对当前任务的延续或增量调整，不默认视为新任务，不从零重做。
- 已完成步骤、已确认事实、用户决策和有效制品继续保留；没有受到影响的步骤不重新执行。
- 主会话先读取并更新当前计划状态，再判断新增目标影响哪些后续步骤；只有现有计划无法覆盖新增目标时，才考虑动态方案。
- 如果需要调用动态规划子智能体，交给它的是“既有计划状态 + 新目标 + 变化影响”，要求它设计增量方案，不是把完整对话重新总结成一份全新计划。模型按主技能“模型解析规则”确定。
- 任何推翻旧计划的判断都必须先向用户展示依据和影响，明确标记为 `NEED_USER_DECISION`，在用户确认前不得重置、丢弃或重复执行旧步骤。

## 计划状态

主会话应持续维护下列状态；如果某项不存在，明确记为“无”而不是重新猜测：

```yaml
plan_version: V1
plan_goal:
plan_status: DRAFT | CONFIRMED | EXECUTING | PAUSED | COMPLETED
selected_workflow:
completed_steps:
  - step_id:
    name:
    status: DONE
    artifact_keys:
    confirmed_decisions:
in_progress_steps:
pending_steps:
artifacts:
confirmed_facts:
pending_questions:
review_question_ledgers:
review_decision_mode: MODEL_AUTONOMOUS | USER_DECIDES
new_user_goal:
plan_change_reason:
affected_steps:
superseded_items:
reset_assessment: KEEP | INCREMENTAL_UPDATE | RESET_CANDIDATE
initial_conversation_model:
special_content_model_rules:
model_resolutions:
```

步骤状态至少使用 `已完成`、`进行中`、`待执行`、`已取消/被替代` 四类。完成过的步骤不能因为计划版本递增而改回“待执行”，除非重规划判断明确说明其制品失效并得到用户确认。

## 增量重规划判断

收到新目标后，主会话按以下顺序处理：

1. 识别新消息属于补充事实、回答待确认问题、增加目标、修改约束、改变产物，还是明确取消/替换旧目标。
2. 将新内容与当前 `plan_goal`、已确认决策、已完成制品和待执行步骤逐项比对。
3. 为每个步骤判断：保持不变、增加前置条件、调整内容、调整顺序、增加并行分支、取消或被替代。
4. 只把变化部分纳入新版本；原计划中未受影响的已完成项和待执行项继续沿用。
5. 明确写出“不重做原因”：哪些已有制品仍然有效、哪些决策仍然有效，以及为什么只需修改后续路径。

以下情况通常可以增量更新：

- 新增一个与原目标兼容的子目标，可追加为后续步骤或独立并行分支；
- 补充事实只影响某个模块、页面、字段、角色、评审视角或输出格式；
- 用户调整了尚未执行步骤的顺序、模型、粒度或交付方式；
- 已完成的分析制品仍可复用，只需让下游步骤读取更新后的摘要。

以下情况只能提出重置候选，不能静默推翻：

- 用户明确改变核心目标，导致原目标不再成立；
- 新事实与已确认的核心业务规则、范围或基准文档直接冲突；
- 已完成制品依赖的关键前提被证明错误，且无法通过局部修订恢复有效性；
- 用户明确要求废弃当前方案、换一套方法或从头开始。

提出重置候选时，必须同时说明：失效的旧前提、受影响的已完成制品、不能复用的具体原因、建议的新起点、将保留的可复用内容，以及用户需要确认的唯一决策。用户未明确同意前，保持原计划状态并暂停受影响步骤。

## 增量方案确认界面

当新增目标会改变尚未执行的步骤、交付物或执行路径时，必须展示 `V(n+1)`，至少包含：

1. `方案版本`：新版本号及上一版本号；
2. `当前状态`：当前计划状态、已完成步骤、进行中步骤、待执行步骤及对应制品；
3. `本次新增/修正目标`：只列本次用户新增或改变的内容；
4. `保持不变的内容`：继续沿用的工作流、步骤、决策和制品；
5. `计划变化`：逐项列明新增、调整、删除、替代、顺序变化及原因；
6. `接下来具体做什么`：按执行顺序列出下一步及后续步骤、输入、输出、依赖和模型；
7. `产物变化`：已有产物复用方式、新增产物和最终交付物变化；
8. `不重做原因`：说明为什么不从零开始；若为重置候选，改为展示重置依据；
9. `当前缺口/待确认`：只有影响继续执行的问题才列出；
10. `审查决策模式变化`：如本次调整审查，写明 `MODEL_AUTONOMOUS` / `USER_DECIDES` 的沿用或变化。
11. `请确认`：等待用户确认增量版本；若是 `RESET_CANDIDATE`，必须明确等待用户批准重置。

若新消息只是补充信息且不改变计划路径，更新状态即可，不必无意义地生成完整的新方案；但要记录该信息被写入了哪个步骤或制品。审查问题台账和后续解答不能因版本变化丢失，且仍须在最终对话中呈现。若当前方案尚未确认，则合并新信息后更新当前草案并继续等待一次确认，不得提前执行。

## 交给动态规划子智能体的最小增量输入

只有主会话判断需要动态方案时才调用按主技能“模型解析规则”确定的规划子智能体，并使用以下输入：

```yaml
prior_plan_version:
prior_plan_status:
plan_goal:
selected_workflow:
completed_steps:
in_progress_steps:
pending_steps:
reusable_artifacts:
confirmed_decisions:
new_user_goal:
change_impact:
reset_assessment:
dynamic_gap:
expected_delta_plan:
initial_conversation_model:
review_decision_mode:
user_step_model_override:
special_content_model_rule:
model_resolution_basis:
```

要求规划子智能体返回：

- 复用的已完成步骤和制品；
- 新增目标影响的步骤；
- 增量后的后续计划及依赖；
- 新增或变化的产物；
- `KEEP`、`INCREMENTAL_UPDATE` 或 `RESET_CANDIDATE` 判断及依据。

规划子智能体不得自行执行、删除制品、重跑未受影响步骤或把 `RESET_CANDIDATE` 当作用户已经同意。
