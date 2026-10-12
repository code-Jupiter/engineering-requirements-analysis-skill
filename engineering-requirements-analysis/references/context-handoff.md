# 上下文与交接

## 主控状态

主控保留：`source_material`、`confirmed_facts`、`constraints`、`workflow_match`、`workflow_proposal`、`confirmed_workflow`、`requirement_detail_outline`、`confirmed_requirement_detail_outline`、`change_impact_matrix`、`output_file_decision`、`artifacts`、`subskill_outputs`、`pending_questions`、`review_question_ledgers`、`review_decision_mode`、`blocking_points`、`user_decisions`、`initial_conversation_model`、`special_content_model_rules` 和 `model_resolutions`。

若已有方案或执行中的任务，还必须保留当前计划状态，至少包括：

`plan_version`、`plan_goal`、`plan_status`、`completed_steps`、`in_progress_steps`、`pending_steps`、`reusable_artifacts`、`confirmed_decisions`、`new_user_goal`、`plan_change_reason`、`affected_steps`、`superseded_items`、`reset_assessment`、`review_question_ledgers`、`review_decision_mode`、`initial_conversation_model`、`special_content_model_rules` 和 `model_resolutions`。

计划版本变化时，已完成步骤和制品默认继承；只有经过“推翻重来”判断并获得用户确认，才能标记制品失效或重新执行。

## 工作流外动态方案摘要

只有两条标准工作流无法覆盖新增目标或用户要求重新组合核心步骤时，才把下面的“增量摘要”交给按主技能“模型解析规则”确定的规划子智能体。若当前已经有计划，不得只发送新目标，也不得从完整对话重建一份全新计划：

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
review_decision_mode:
review_question_ledgers:
user_goal:
available_materials:
expected_output:
confirmed_constraints:
missing_information:
new_user_goal:
change_impact:
reset_assessment:
dynamic_gap:
expected_delta_plan:
selected_internal_subskills:
selection_reasons:
required_dependencies:
possible_parallel_steps:
initial_conversation_model:
user_step_model_override:
special_content_model_rule:
model_resolution_basis:
```

要求规划子智能体返回复用的已完成事项、受影响步骤、增量后的后续计划、产物变化和重规划判断。若返回 `RESET_CANDIDATE`，主会话必须把它当作待用户确认的候选，不得直接执行。

不要把完整对话、未筛选的所有材料或无关制品交给规划子智能体。

## 子技能输入包

```yaml
subskill:
purpose:
input_payload:
confirmed_facts:
constraints:
upstream_artifact_keys:
expected_output:
stop_conditions:
model_resolution_basis:
```

每个输入包只包含本步骤所需信息。子技能不能自行读取主控的完整状态。

审查子技能必须使用以下独立审查输入约束：

```yaml
review_agent_role:
blind_first_read: true
reading_protocol: sequential_first_read_with_question_capture
material_scope:
confirmed_facts_minimal:
review_scope:
excluded_context:
  - full_conversation
  - author_intent
  - historical_discussion
  - other_review_outputs
  - optimization_or_final_decisions
expected_question_ledger: true
review_decision_mode:
```

`excluded_context` 中的内容不得通过摘要、提示词或上游制品间接传入。并行审查 agent 之间不共享上下文。`review_qa_pack` 需要读取审查发现时，只作为独立整理阶段接收已筛选的结果，不得把整理结果回传给已经完成的盲审 agent。

## 子技能输出包

```yaml
status: DONE | NEED_INFO | BLOCKED | NEED_USER_DECISION
summary:
artifact_key:
findings:
questions:
question_ledger:
  - id:
    first_raised_at:
    evidence:
    question:
    impact:
    suggestion:
    status: UNRESOLVED | ANSWERED_LATER | PARTIALLY_ANSWERED | CONFLICTING_ANSWER
    answer_evidence:
    conversation_returned: true
blocking_points:
assumptions:
```

长内容可以保存为独立审查制品，但主线程必须在对话中展示完整的问题台账、材料后续解答、建议和处理状态；不能只展示摘要，也不能把台账写入需求说明书正文。`MODEL_AUTONOMOUS` 需记录采纳/未采纳/保留状态；`USER_DECIDES` 需在修改前停下等待用户决定。
