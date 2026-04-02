---
name: wecomcli-workflow-morning-brief
description: 企业微信每日晨间简报技能。整合日程、待办、消息等数据，生成今日综合简报并发送到企业微信消息。当用户说"晨间简报"、"今日安排"、"今天做什么"、"早报"、"今日概览"、"今天有什么会议"等涉及每日概览的场景时触发。
---

# 企业微信每日晨间简报

通过 wecom-cli 整合企业微信日程、待办和消息数据，生成每日综合简报。

## 前置依赖

本 Skill 需要以下底层能力（已内置）：
- `wecom-cli schedule` — 日程查询
- `wecom-cli todo` — 待办查询
- `wecom-cli msg` — 消息查询和发送

可选联动（已初始化才有效）：
- `wecomcli-workflow-personal-crm` — 参会人背景信息
- `wecomcli-workflow-social-tracker` — 昨日数据表现

**本 Skill 无需创建独立智能表格**，直接读取底层数据和联动 Skill 数据。

> **注意**：本 Skill 不创建智能表格，因此无需保存 docid/url。当联动其他 Skill 时，使用对应 Skill memory 中的 url（如有）来定位智能表格，优先使用 url 而非 docid。

---

## 操作流程

### 模式 1：生成今日简报

**步骤 1：获取今日日程**

```bash
# 获取今天的日程列表
wecom-cli schedule get_schedule_list_by_range '{"start_time": "2026-04-02 00:00:00", "end_time": "2026-04-02 23:59:59"}'

# 获取日程详情（含参与者、地点、描述）
wecom-cli schedule get_schedule_detail '{"schedule_id_list": ["SCHEDULE_ID_1", "SCHEDULE_ID_2"]}'
```

**步骤 2：获取今日待办**

```bash
wecom-cli todo get_todo_list '{}'
# AI 筛选今天到期或即将到期的待办
```

**步骤 3：获取最近消息摘要**

```bash
# 获取最近的消息会话列表（替代邮件）
wecom-cli msg get_msg_chat_list '{"begin_time": "2026-04-01 18:00:00", "end_time": "2026-04-02 08:00:00"}'
```

**步骤 4：获取联动数据（可选）**

```bash
# 如果个人 CRM 已初始化，查询参会人背景
wecom-cli doc smartsheet_get_records '{"docid": "<个人CRM_DOCID>", "sheet_id": "<联系人SHEET_ID>"}'

# 如果社交追踪已初始化，查询昨日数据
wecom-cli doc smartsheet_get_records '{"docid": "<社交追踪DOCID>", "sheet_id": "<每日快照SHEET_ID>"}'
```

**步骤 5：AI 汇总生成简报**

**简报格式：**

```
## 2026年4月2日晨间简报（星期四）

### 日程安排（共 3 场）
| 时间 | 事件 | 组织者 | 备注 |
|------|------|--------|------|
| 09:00-10:00 | Q2预算审批 | 张三 | 上次交流：3/25讨论了预算方案 |
| 14:00-15:00 | 技术方案讨论 | 李四 | — |
| 16:00-17:00 | 1:1 with 王五 | 王五 | 关注合作进展（上次：45天前） |

### 待办事项（共 4 项）
- [ ] 发送产品路线图给张三（截止：今天）⚠️ 即将到期
- [ ] 调研竞品定价策略（截止：周四）
- [x] ~~更新周报~~（已完成）

### 最近消息（2 条未读会话）
| 联系人 | 最新消息 | 时间 | 紧急度 |
|--------|---------|------|--------|
| 张三 | Q2预算终版确认 | 昨天 10:30 | 高 |
| HR | 年度体检通知 | 昨天 09:00 | 低 |

### 社交数据（昨日）
| 平台 | 粉丝 | 日增 | 互动率 |
|------|------|------|--------|
| YouTube | 1000 | +5 | 4.5% |

### 小结
- 共 3 场会议，4 项待办（1 项即将到期）
- 张三的预算消息需要今天回复
- 空闲时段：10:00-14:00、15:00-16:00
- 需要关注：王五（XX公司）已 45 天未联系
```

### 模式 2：发送简报到企业微信

```bash
wecom-cli msg send_message '{"chat_type": 1, "chatid": "<USERID>", "msgtype": "text", "text": {"content": "<简报内容>"}}'
```

发送到群聊：

```bash
wecom-cli msg send_message '{"chat_type": 2, "chatid": "<GROUP_CHATID>", "msgtype": "text", "text": {"content": "<简报内容>"}}'
```

### 模式 3：简问快答

快速回答用户关于今天的简单问题。

| 用户问 | AI 处理 |
|--------|--------|
| "今天几场会？" | 查询日程数量 |
| "今天有急事吗？" | 查询即将到期的待办 + 重要消息 |
| "下午有空吗？" | 查询日程空闲时段 |

---

## 智能摘要逻辑

### 日程参会人背景增强

如果 `wecomcli-workflow-personal-crm` 已初始化：
- 从联系人表中查找参会人信息
- 显示：公司、职位、最后联系时间、最后联系内容
- 如果超过 30 天未联系，标注 ⚠️

### 消息紧急度判断

AI 根据以下规则判断消息紧急度：
- **高**：包含"紧急"、"尽快"、"今天"、"截止"等关键词，或来自上级/重要客户
- **中**：常规工作消息
- **低**：通知、群消息、自动消息

### 空闲时段计算

AI 从日程列表中提取忙碌时段，计算空闲间隔：
- 日程之间间隔 > 1 小时标记为"可用空闲时段"

---

## 实操注意事项

### 常见踩坑

1. **日程时间范围** — `get_schedule_list_by_range` 仅支持前后 30 天
2. **API 调用失败时** — 检查错误码：301085 表示 docid 无效（改用 url）；2022004 表示字段未找到（先 add_fields）；2022003 表示记录不存在（先 add_records）。遇到错误先用 `smartsheet_get_sheet '{"url": "<URL>"}'` 验证表结构。
3. **待办筛选** — `get_todo_list` 返回全部待办，AI 需要自行按时间过滤
4. **消息权限** — 群聊消息需要用户是群成员才能获取
5. **联动 Skill 未初始化** — 如果 CRM 或社交追踪未初始化，跳过对应板块，不报错
6. **SINGLE_SELECT 创建时 options 不生效** — `smartsheet_add_fields` 传入的 `property.options` 不会预创建选项，选项在首次写入记录时通过 `[{"text": "选项"}]` 自动创建。创建字段时可省略 `property` 参数。

### 定时自动化建议

| 任务 | 建议时间 | Cron |
|------|---------|------|
| 生成并发送晨间简报 | 工作日 8:00 | `0 8 * * 1-5` |

### 数据联动

本 Skill 读取以下 Skill 的数据：
- `wecomcli-workflow-personal-crm` — 参会人背景信息
- `wecomcli-workflow-social-tracker` — 昨日社交数据
- `wecomcli-workflow-meeting-todo` — 会议待办完成情况
