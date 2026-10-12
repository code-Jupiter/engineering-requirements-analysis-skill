# 意图识别 `intent_intake`（主控阶段）

由主会话完成用户目标、材料类型、目标产物、范围、约束和缺失信息的识别，为流程匹配或动态规划生成事实摘要。不单独启动路由子智能体；不要制定方案或执行下游任务。

输入用户原始请求、材料或材料摘要及已确认决策。若当前会话已有计划、执行状态或过程制品，还要读取当前计划版本和状态，将新消息识别为补充事实、回答待确认问题、增加目标、修改约束、改变产物、取消/替换旧目标中的哪一类；不得仅凭新消息重置任务。区分事实、要求、假设和未知项；识别用户是否明确要求完整工作流或局部阶段；判断目标属于生成、审查、修订、一致性检查、可实施性审查或评审准备。

返回 `user_goal`、`material_type`、`expected_output`、`scope`、`constraints`、`confirmed_facts`、`missing_information`、`explicit_workflow`，以及已有计划场景下的 `plan_version`、`new_goal_type`、`affected_plan_steps`、`reusable_artifacts` 和 `reset_signal`。`reset_signal` 只表示是否需要进一步评估重置，不代表已经获准重置。

发布版还提取 `workspace_decision`、`target_audience`、`output_format`、`acceptance_focus` 和 `selected_mode`。不要从提示词示例继承个人路径或固定读者；工作区、业务侧读者和 Markdown 都是任务参数。扩展模式按 `references/extended-modes.md` 匹配，不自动执行整份需求文档工作流。
