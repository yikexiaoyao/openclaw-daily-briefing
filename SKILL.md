---
name: openclaw-daily-briefing
description: 定时汇报每日/每周任务日程，通过 dida CLI 直连滴答清单 API，支持优先级分组、已完成/未完成分类展示。触发场景：晨报、晚报、周报、今日待办、任务汇报、查看任务、任务日程。
metadata:
  openclaw:
    emoji: "📅"
    always: false
    requires:
      bins: ["node"]
      env: []
---

# Daily Briefing Skill

定时汇报每日/每周任务日程。通过 `dida` CLI（@suibiji/dida-cli）直连滴答清单 API，获取实时任务数据。

## 前置条件

- 已安装 `@suibiji/dida-cli`（`npm i -g @suibiji/dida-cli`）
- 已执行 `dida auth login` 完成 OAuth 授权

## 数据源

| 命令 | 用途 |
|------|------|
| `dida task filter --status 0 --json` | 获取所有未完成任务（JSON） |
| `dida task completed` | 获取已完成任务 |

## 快速部署

### 确认推送配置

**Telegram 私聊：**
- `DELIVERY_TO`: `telegram:你的用户ID`
- `DELIVERY_CHANNEL`: `telegram`
- `DELIVERY_ACCOUNT`: `default`

### 创建任务

逐一用 `cron` 工具的 `action: "add"` 创建三个任务：

---

## 晨报（每天 8:00）

```json
{
  "name": "每日晨报 - 任务日程提醒",
  "schedule": { "kind": "cron", "expr": "0 8 * * *", "tz": "Asia/Shanghai" },
  "payload": {
    "kind": "agentTurn",
    "lightContext": true,
    "timeoutSeconds": 600,
    "message": "执行每日晨报任务。\n\n步骤：\n1. 执行 `dida task filter --status 0 --json` 获取所有未完成任务（JSON 格式）\n2. 从结果中按 due-date 分类整理\n\n汇报要求（严格遵守）：\n- **只输出最终结果**，不要输出任何分析、思考、推理过程\n- 标题格式：☀️ 晨报｜X月X日 周X\n- 按优先级分组展示：🔴 逾期 → 🟡 今日 → 🟢 近期 → 📋 无截止日期\n- 每条标注优先级（高/中/低/无）\n- 没有内容就一句话简要说明，不要写\"无\"、\"暂无\"等字样，不要留空模板或占位符\n- **不要有任何额外的解释、评价、感慨或模板文字**"
  },
  "sessionTarget": "isolated",
  "delivery": { "mode": "announce", "to": "<DELIVERY_TO>", "channel": "<DELIVERY_CHANNEL>", "accountId": "<DELIVERY_ACCOUNT>", "bestEffort": true },
  "enabled": true
}
```

## 晚报（每天 20:00）

```json
{
  "name": "每日晚报 - 任务日程汇总",
  "schedule": { "kind": "cron", "expr": "0 20 * * *", "tz": "Asia/Shanghai" },
  "payload": {
    "kind": "agentTurn",
    "timeoutSeconds": 600,
    "message": "执行每日晚报任务。\n\n步骤：\n1. 执行 `dida task completed` 获取已完成任务，筛选今天的\n2. 执行 `dida task filter --status 0 --json` 获取未完成任务，筛选今天截止但尚未完成的\n\n汇报要求：\n- 标题格式：🌃 晚报｜X月X日 周X\n- 分两部分：✅ 今日已完成 → ⏳ 今日未完成\n- 没有内容就一句话简要说明，不要写\"无\"、\"暂无\"等字样，不要留空模板或占位符\n- 语气简洁自然，不要铺垫废话，不要添加多余的评价或感慨\n- 只输出实际存在的内容，不编造、不推测"
  },
  "sessionTarget": "isolated",
  "delivery": { "mode": "announce", "to": "<DELIVERY_TO>", "channel": "<DELIVERY_CHANNEL>", "accountId": "<DELIVERY_ACCOUNT>", "bestEffort": true },
  "enabled": true
}
```

## 周报（每周日 20:30）

```json
{
  "name": "每周晚报 - 周任务汇总",
  "schedule": { "kind": "cron", "expr": "30 20 * * 0", "tz": "Asia/Shanghai" },
  "payload": {
    "kind": "agentTurn",
    "timeoutSeconds": 600,
    "message": "执行每周周报任务。\n\n步骤：\n1. 执行 `dida task completed` 获取已完成任务，筛选本周（周一到周日）的\n2. 执行 `dida task filter --status 0 --json` 获取未完成任务\n\n汇报要求：\n- 标题格式：📊 周报｜X月X日 - X月X日\n- 分两部分：✅ 本周完成 → ⏳ 未完成/待跟进\n- 没有内容就一句话简要说明，不要写\"无\"、\"暂无\"等字样，不要留空模板或占位符\n- 语气简洁自然，不要铺垫废话，不要添加多余的评价或感慨\n- 只输出实际存在的内容，不编造、不推测"
  },
  "sessionTarget": "isolated",
  "delivery": { "mode": "announce", "to": "<DELIVERY_TO>", "channel": "<DELIVERY_CHANNEL>", "accountId": "<DELIVERY_ACCOUNT>", "bestEffort": true },
  "enabled": true
}
```

---

## 自定义

### 修改时间

修改 `expr` 字段即可：

| 场景 | Cron 表达式 |
|------|------------|
| 工作日 7:30 | `"30 7 * * 1-5"` |
| 每天两次（7:00 和 19:00） | `"0 7,19 * * *"` |
| 每 6 小时 | `"0 */6 * * *"` |
| 周一和周五 9:00 | `"0 9 * * 1,5"` |

### 禁用某个汇报

将 `"enabled": false` 即可暂停，无需删除。

## 注意事项

- **dida CLI 需要登录态**：token 存储在本地，过期需重新 `dida auth login`
- **不编造内容**：如果某时段没有条目，简要说明即可
- **格式统一**：三个汇报的输出风格一致
- **仅依赖 `dida` CLI + `cron`**：无需 iCal 订阅链接
