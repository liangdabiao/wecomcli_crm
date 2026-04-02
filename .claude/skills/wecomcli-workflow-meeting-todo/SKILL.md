---
name: wecomcli-workflow-meeting-todo
description: 企业微信会议待办追踪技能。从会议中提取待办事项，通过消息确认后自动创建待办，定期检查完成情况。当用户说"会议待办"、"会后的待办"、"追踪待办"、"提取会议行动项"、"整理会议纪要待办"、"会议待办完成了吗"等涉及会议待办管理的场景时触发。
---

# 企业微信会议待办追踪

通过 wecom-cli 操作企业微信会议、待办和智能表格，追踪会议产生的待办事项。

## 首次配置（自动初始化）

**执行任何操作前，先检查 memory 中是否已有会议待办配置。** 如果没有，按以下流程自动创建：

### 步骤 1：创建智能表格

```bash
wecom-cli doc create_doc '{"doc_type": 10, "doc_name": "会议待办追踪"}'
```

从返回结果中获取 `docid`。

### 步骤 2：创建子表和字段

```bash
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "会议待办"}'
```

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "会议主题", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "会议日期", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "待办内容", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "待办类型", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "负责人", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "建议截止日", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "确认状态", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "关联待办ID", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "会议ID", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

### 步骤 3：保存配置到 memory

将以下信息保存到 memory 文件 `meeting-todo-config.md`：

```markdown
---
name: meeting-todo-config
description: 会议待办追踪智能表格配置信息
type: project
---

## 会议待办表配置

- docid: <DOCID>
- url: <创建时返回的URL>
- 表格名称: 会议待办追踪
- 会议待办 sheet_id: <SHEET_ID>
```

---

## 操作流程

> 以下命令中 `<DOCID>` 和 `<SHEET_ID>` 从 memory 的 meeting-todo-config 中读取。

### 模式 1：批量提取

从指定时间范围的所有会议中提取待办。

**步骤 1：获取会议列表**

```bash
wecom-cli meeting list_user_meetings '{"begin_datetime": "2026-03-26 00:00", "end_datetime": "2026-04-02 23:59", "limit": 100}'
```

**步骤 2：逐个获取会议详情**

```bash
wecom-cli meeting get_meeting_info '{"meetingid": "<MEETING_ID>"}'
```

**步骤 3：AI 从会议描述中提取待办**

- 分析会议主题、描述、参与者信息
- 区分"我方待办"和"对方待办"
- 推断建议截止日（如"这周五"、"下周三"）
- 过滤掉不重要的记录性内容

**步骤 4：发送确认消息**

```bash
wecom-cli msg send_message '{"chat_type": 1, "chatid": "<USERID>", "msgtype": "text", "text": {"content": "## 会议待办提取结果\n\n### Q2产品规划会（3/28）\n**我方待办：**\n1. 把产品路线图发给张三（建议：本周五）\n2. 调研竞品定价策略（建议：下周三）\n**对方待办：**\n1. 提供用户调研数据 — 李四\n\n请确认需要创建待办的条目（回复编号，如 1,2 或 全部/跳过）"}}'
```

**步骤 5：确认后创建待办**

```bash
# 创建企业微信待办
wecom-cli todo create_todo '{"content": "把产品路线图发给张三（来源：Q2产品规划会）", "remind_time": "2026-04-04 09:00:00"}'

# 获取返回的 todo_id，更新追踪表
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "records": [{"values": {
  "会议主题": [{"type": "text", "text": "Q2产品规划会"}],
  "会议日期": [{"type": "text", "text": "2026-03-28"}],
  "待办内容": [{"type": "text", "text": "把产品路线图发给张三"}],
  "待办类型": [{"text": "我方待办"}],
  "负责人": [{"type": "text", "text": "我"}],
  "建议截止日": [{"type": "text", "text": "2026-04-04"}],
  "确认状态": [{"text": "已确认"}],
  "关联待办ID": [{"type": "text", "text": "<TODO_ID>"}],
  "会议ID": [{"type": "text", "text": "<MEETING_ID>"}]
}}]}'
```

### 模式 2：单会议提取

只处理用户指定的某一场会议。

**交互流程：**
1. 用户："提取今天产品评审会的行动项"
2. 查找对应会议
3. AI 提取待办并发送确认
4. 流程同模式 1 的步骤 4-5

```bash
# 按关键词搜索会议
wecom-cli meeting list_user_meetings '{"begin_datetime": "2026-04-02 00:00", "end_datetime": "2026-04-02 23:59", "limit": 100}'
# AI 从返回结果中按主题关键词筛选目标会议
```

### 模式 3：完成检查

检查所有会议待办的完成情况。

**步骤 1：查询追踪表中未完成的待办**

```bash
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>"}'
# AI 筛选确认状态为"已确认"的记录
```

**步骤 2：逐个检查待办状态**

```bash
wecom-cli todo get_todo_detail '{"todo_id_list": ["TODO_ID_1", "TODO_ID_2", "..."]}'
```

**步骤 3：更新追踪表状态**

```bash
# 已完成的待办
wecom-cli doc smartsheet_update_records '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE", "records": [{"record_id": "RECORD_ID", "values": {
  "确认状态": [{"text": "已完成"}]
}}]}'
```

**输出格式示例：**

```
## 会议待办完成情况

| 待办 | 来源会议 | 截止日 | 状态 |
|------|---------|--------|------|
| 发送产品路线图 | Q2规划会 | 4/4 | 已完成 ✓ |
| 调研竞品定价 | Q2规划会 | 4/9 | 进行中 |
| 回复设计稿反馈 | UI评审会 | 4/1 | 已过期 ⚠️ |
```

### 模式 4：自动归档

超过 14 天未完成的待办自动归档。

```bash
# 查询所有"已确认"状态的待办
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>"}'

# AI 计算"建议截止日"距今是否超过 14 天
# 超过 14 天的批量更新为"已归档"
wecom-cli doc smartsheet_update_records '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE", "records": [
  {"record_id": "ID1", "values": {"确认状态": [{"text": "已归档"}]}},
  {"record_id": "ID2", "values": {"确认状态": [{"text": "已归档"}]}}
]}'
```

---

## 学习模式

如果用户多次拒绝某类待办（如"会议纪要类记录不需要记"），AI 应：
1. 记录用户的偏好到 memory
2. 下次提取待办时自动过滤类似内容
3. 在提取结果中说明"已按偏好过滤 N 条记录类待办"

---

## 字段类型写入格式（通用规则）

> 完整字段类型格式说明参见 [crm-schema](../wecomcli-crm/references/crm-schema.md)

| 字段类型 | 写入格式 | 示例 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 会议主题、待办内容、日期等 |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 待办类型、确认状态等 |

---

## 实操注意事项

### 常见踩坑

1. **更新时不传 key_type** — 必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **API 调用失败时** — 检查错误码：301085 表示 docid 无效（改用 url）；2022004 表示字段未找到（先 add_fields）；2022003 表示记录不存在（先 add_records）。遇到错误先用 `smartsheet_get_sheet '{"url": "<URL>"}'` 验证表结构。
3. **SINGLE_SELECT 选项不匹配** — 确认状态的选项必须精确匹配
4. **SINGLE_SELECT 创建时 options 不生效** — `smartsheet_add_fields` 传入的 `property.options` 不会预创建选项，选项在首次写入记录时通过 `[{"text": "选项"}]` 自动创建。创建字段时可省略 `property` 参数。
5. **会议时间范围** — `list_user_meetings` 仅支持前后 30 天
6. **待办 ID 关联** — 创建待办后保存返回的 `todo_id` 到追踪表，用于后续状态检查
7. **手动输入待办** — 如果用户直接说"今天会议我答应做X"，无需查会议，直接创建

### 定时自动化建议

| 任务 | 建议时间 | Cron |
|------|---------|------|
| 待办完成检查 | 工作日 10:00 | `0 10 * * 1-5` |
| 自动归档 | 每周日 22:00 | `0 22 * * 0` |

### 数据联动

- 本 Skill 创建的待办可通过 `wecomcli-workflow-morning-brief` 在晨间简报中展示
- 顾问团 Skill (`wecomcli-workflow-business-advisor`) 的运营顾问会检查待办完成率
