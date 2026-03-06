---
name: cockpit-fuzzy-alarm-reminder
description: 在车载语音场景下，将用户的模糊自然语言请求转换为可执行的闹钟/提醒（创建/修改/取消/静音）；解析“工作日/平时/休息”等日历语义和“待会儿/早上/中午”等模糊时间；按重要性与置信度选择低打扰读回、显式确认或草稿，并对“别再提醒了/闭嘴/取消这个”等抑制指令优先响应。
---

# cockpit-fuzzy-alarm-reminder

## 目标与取舍

- 优先单轮完成；追问最多一轮，只问最关键的不确定项。
- 宁可多提醒一点，也不要漏提醒；但避免靠滥发提醒兜底。
- 高重要性事项宁可先草稿也不要静默误设。
- 用户说“停/别提醒/取消”时先停再理解；反馈要极短。
- 用户纠偏要可学习，但避免过拟合一次性例外。

## 范围边界（触发后也要做分流）

- 地点/路线触发（如“到公司提醒我”）→ 转交地点/导航触发能力。
- 复杂日程规划（如“下个月每次开会前两小时提醒我”）→ 转交日程规划能力。
- 任务内容缺失（如“提醒我一下”）→ 仅在允许追问时追问一次，否则不创建。
- 高重要性且关键信息无法确认 → 只建 `draft`，不要静默 `arm`。

## 输入（期望的上下文）

- `utterance`：当前用户话语
- `conversation_context`：最近几轮相关对话（用于绑定“改一下/取消这个”）
- `now` / `timezone` / `locale`
- `drive_state`：`parked | low_load | high_load | maneuvering`
- 可选：`user_preferences`、`holiday_calendar`、`calendar_context`、`currently_firing_reminder`

## 输出（结构化指令）

产出一个结构化指令对象，至少包含：

- `action`：`create_alarm | create_reminder | update | cancel_instance | cancel_series | stop_speaking_only | draft | ask_confirmation | noop`
- `resolved_schedule`：单次 ISO 时间或重复规则（RRULE / 周期 + 时间点）
- `task_label`
- `importance_level`：`low | normal | high | critical`
- `confidence`：`0.0-1.0`
- `confirmation_mode`：`none | low_touch_readback | explicit_confirm | hard_block`
- `spoken_response`：车载可播报的 1-2 句短话
- `memory_update`：需要沉淀的偏好（如“早上=07:30”）

## 主流程（按优先级执行）

1. **先处理抑制/取消**：一旦命中“闭嘴/别再提醒了/取消这个”等，先停播报/暂停本次触发，再决定取消实例/序列（见 `references/interaction_policies.md`）。
2. **判定类型**：区分闹钟 vs 提醒；两者都不明确时默认提醒（更低打扰、更可逆）。
3. **抽取槽位**：任务内容、日期、时间/时段、相对时间、重复规则、修改/取消指向对象、重要性线索。
4. **生成候选解释**：最多 2–3 个；结合 `user_preferences`、节假日、`now`、上下文打分；禁止落到过去时间（见 `references/time_rules.md`）。
5. **判定重要性与确认策略**：按“重要性 × 置信度”决定：直接执行+读回 / 问一个关键问题 / 显式确认 / 仅草稿（见 `references/importance_confirmation.md`）。
6. **执行动作**：调用闹钟/提醒服务创建、修改、取消；必要时把不确定项记为 `draft`。
7. **生成播报**：普通事项读回一句即可；高重要性确认不超过两句；避免抢占导航关键播报窗口（见 `references/interaction_policies.md`）。
8. **偏好沉淀**：仅在用户明确声明规则或对同类解释连续纠正时更新偏好；不要从一次性例外学习（见 `references/interaction_policies.md`）。

## 参考文件（按需打开）

- `references/time_rules.md`：工作日/平时/休息、相对时间、模糊时段锚点、过去时间顺延规则
- `references/importance_confirmation.md`：重要性分级、确认矩阵、读回/确认话术模板、默认执行策略
- `references/interaction_policies.md`：顺带提醒、停止/取消高响应、修改绑定、偏好学习、车载交互约束
- `references/examples.md`：对话与解析示例
- `references/eval_cases.md`：最小评测用例清单

