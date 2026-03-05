# 使用 `agent-browser` 在 ClawHub 上找“个人管家助手”相关 skills：经验记录

## 目标
- 在 `https://clawhub.ai/skills?sort=downloads&nonSuspicious=true`（按下载量、过滤可疑项）中，快速筛出与“个人管家助手”相关的 skills（如：日程/提醒/待办/笔记/邮件/联系人/晨报等）。

## 实操流程（可复用）
1. 打开页面并抓取可访问性树（用于拿到元素 ref）
   - `agent-browser open 'https://clawhub.ai/skills?sort=downloads&nonSuspicious=true'`
   - `agent-browser snapshot`
2. 用过滤框做关键词检索（ref 以 snapshot 输出为准；本次过滤框常见为 `@e11`）
   - `agent-browser fill @e11 "calendar"`
   - 等待加载：`agent-browser wait --load networkidle` 或 `agent-browser wait 1500`
   - `agent-browser snapshot`
3. 迭代更换关键词，收集命中项的 `slug` 与链接（列表里每个条目都有 `/user/skill` URL）
4. 结束：`agent-browser close`

## 关键词策略（效果最好的一组）
按“个人管家”常见能力面拆分关键词，效率最高：
- **日历/日程**：`calendar`, `gcal`, `gog`, `outlook`
- **提醒**：`reminder`
- **待办/任务**：`todo`, `todoist`, `things`
- **笔记/知识库**：`notes`, `notion`, `obsidian`
- **邮件**：`email`, `gmail`, `imap`, `smtp`
- **联系人/关系维护**：`contacts`, `people`, `crm`
- **晨报/日报**：`brief`, `daily`, `briefing`
- **个人管家/助理泛词**：`assistant`, `personal`
- **Jarvis**：`jarvis`（注意不是 `javis`）

## 观察与坑
- **中文关键词命中率低**：例如 `管家/个人/助理` 在站内过滤中可能直接返回空；换成英文关键词更稳定。
- **加载是异步的**：`fill` 后经常先显示 “Loading skills…”，需要 `wait --load networkidle` 或固定等待（如 `wait 1500`）再 `snapshot`。
- **结果可能有“滚动加载更多”**：页面底部提示 “Scroll to load more”；如果你在“宽泛词”下想看更多结果，需要 `agent-browser scroll down` 后再 `snapshot`。
- **ref 会变化**：`@e11` 等 ref 不是硬编码，最好每次先 `snapshot` 确认过滤框 ref。
- **Jarvis 相关命中很少**：本次 `jarvis` 关键词只命中 `jarvis-mission-control`（更偏“多代理协作/任务看板”，不等同于个人生活管家）。

## 输出沉淀
- 最终筛选结果已整理为：`assistants.md`

