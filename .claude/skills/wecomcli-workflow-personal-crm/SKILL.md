---
name: wecomcli-workflow-personal-crm
description: 企业微信个人CRM技能。自动从日程和消息中提取联系人，维护关系热度分，支持自然语言查询和超期提醒。当用户说"联系人管理"、"个人CRM"、"谁联系过我"、"多久没联系"、"联系人热度"、"关系管理"、"上个月和谁有互动"等涉及联系人管理的场景时触发。
---

# 企业微信个人 CRM

通过 wecom-cli 操作企业微信日程、消息、通讯录和智能表格，自动维护个人联系人关系网络。

> 注意：本 Skill 与 `wecomcli-crm`（客户管理）不同。wecomcli-crm 管理的是**商业客户**，本 Skill 管理的是**所有联系人关系**，包括同事、朋友、行业人脉等。

## 首次配置（自动初始化）

**执行任何操作前，先检查 memory 中是否已有个人CRM配置。** 如果没有，按以下流程自动创建：

### 步骤 1：创建智能表格

```bash
wecom-cli doc create_doc '{"doc_type": 10, "doc_name": "个人CRM"}'
```

从返回结果中获取 `docid`。

### 步骤 2：创建子表和字段

```bash
# 创建子表1: 联系人主表
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "联系人"}'

# 创建子表2: 互动记录
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "互动记录"}'
```

获取各子表的 `sheet_id`，然后添加字段：

**联系人主表字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "姓名", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "公司", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "职位", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "邮箱", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "手机", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "来源", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "关系类型", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "重要程度", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "热度分", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "最后联系时间", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "最后联系内容", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "备注", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

**互动记录字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "关联联系人", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "互动时间", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "互动类型", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "互动摘要", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "涉及主题", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

### 步骤 3：保存配置到 memory

将以下信息保存到 memory 文件 `personal-crm-config.md`：

```markdown
---
name: personal-crm-config
description: 个人CRM智能表格配置信息
type: project
---

## 个人CRM表配置

- docid: <DOCID>
- url: <创建时返回的URL>
- 表格名称: 个人CRM
- 联系人 sheet_id: <SHEET_ID>
- 互动记录 sheet_id: <SHEET_ID>
```

---

## 操作流程

> 以下命令中 `<DOCID>` 和各 `<SHEET_ID>` 从 memory 的 personal-crm-config 中读取。

### 模式 1：自动扫描提取联系人

从日程和消息中自动提取联系人信息。

**步骤 1：扫描日程参与者**

```bash
# 获取过去30天的日程
wecom-cli schedule get_schedule_list_by_range '{"start_time": "2026-03-03 00:00:00", "end_time": "2026-04-02 23:59:59"}'

# 获取日程详情（含参与者信息）
wecom-cli schedule get_schedule_detail '{"schedule_id_list": ["SCHEDULE_ID_1", "SCHEDULE_ID_2"]}'
```

**步骤 2：扫描消息互动**

```bash
# 获取最近30天的会话列表
wecom-cli msg get_msg_chat_list '{"begin_time": "2026-03-03 00:00:00", "end_time": "2026-04-02 23:59:59"}'

# 拉取各会话的消息记录（提取互动摘要）
wecom-cli msg get_message '{"chat_type": 1, "chatid": "<CHATID>", "begin_time": "2026-03-03 00:00:00", "end_time": "2026-04-02 23:59:59"}'
```

**步骤 3：匹配通讯录信息**

```bash
wecom-cli contact get_userlist '{}'
```

**步骤 4：AI 处理并写入**

- 合并去重：同一人可能出现在日程和消息中
- 提取姓名、公司、职位信息
- 生成互动摘要
- 计算热度分
- 写入联系人表和互动记录表

```bash
# 写入新联系人（使用 upsert 逻辑：先查询是否存在）
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<联系人SHEET_ID>"}'

# 不存在则新增
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<联系人SHEET_ID>", "records": [{"values": {
  "姓名": [{"type": "text", "text": "张三"}],
  "公司": [{"type": "text", "text": "XX科技"}],
  "职位": [{"type": "text", "text": "产品经理"}],
  "来源": [{"text": "日程"}],
  "关系类型": [{"text": "行业人脉"}],
  "重要程度": [{"text": "中"}],
  "热度分": [{"type": "text", "text": "65"}],
  "最后联系时间": [{"type": "text", "text": "2026-04-01"}],
  "最后联系内容": [{"type": "text", "text": "讨论了Q2产品规划"}],
  "备注": [{"type": "text", "text": ""}]
}}]}'

# 写入互动记录
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<互动记录SHEET_ID>", "records": [{"values": {
  "关联联系人": [{"type": "text", "text": "张三"}],
  "互动时间": [{"type": "text", "text": "2026-04-01"}],
  "互动类型": [{"text": "会议"}],
  "互动摘要": [{"type": "text", "text": "Q2产品规划会，讨论了路线图和时间表"}],
  "涉及主题": [{"type": "text", "text": "产品规划, Q2目标"}]
}}]}'
```

### 模式 2：自然语言查询

支持灵活的中文查询。

**常见查询示例：**

| 用户问 | AI 处理方式 |
|--------|-----------|
| "上个月谁联系过我？" | 查询互动记录，按月过滤，汇总去重 |
| "XX公司的人最后一次聊了什么？" | 按公司过滤联系人，查找最近互动记录 |
| "有没有超过3个月没联系的重要关系？" | 查询热度分 < 30 且重要程度 = 高的联系人 |
| "多久没联系张三了？" | 按姓名查询，返回最后联系时间和间隔天数 |

```bash
# 查询所有联系人
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<联系人SHEET_ID>"}'

# 查询所有互动记录
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<互动记录SHEET_ID>"}'

# AI 在内存中进行过滤和计算
```

### 模式 3：手动添加/更新联系人

```bash
# 更新联系人信息
wecom-cli doc smartsheet_update_records '{"docid": "<DOCID>", "sheet_id": "<联系人SHEET_ID>", "key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE", "records": [{"record_id": "RECORD_ID", "values": {
  "公司": [{"type": "text", "text": "新公司"}],
  "职位": [{"type": "text", "text": "新职位"}],
  "重要程度": [{"text": "高"}]
}}]}'
```

### 模式 4：超期提醒

检查超过 N 天未联系的重要关系，发送提醒。

```bash
# 查询所有联系人
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<联系人SHEET_ID>"}'

# AI 筛选：重要程度="高" 且 最后联系时间距今 > 30/60/90 天
# 发送提醒
wecom-cli msg send_message '{"chat_type": 1, "chatid": "<USERID>", "msgtype": "text", "text": {"content": "⚠️ 联系人提醒\n\n以下重要关系已超过60天未联系：\n1. 赵六（ZZ集团）— 最后联系：95天前\n2. ..."}}'
```

---

## 热度分计算规则

热度分 = 频率分(40) + 近期分(40) + 重要度分(20)，满分 100。

| 维度 | 权重 | 计算方式 |
|------|------|---------|
| 频率分 | 40分 | 过去90天互动次数，0次=0分，20次+=40分（线性插值） |
| 近期分 | 40分 | 最后互动距今天数，7天内=40分，90天+=0分（线性衰减） |
| 重要度分 | 20分 | 高=20分，中=12分，低=4分 |

---

## 字段类型写入格式（通用规则）

> 完整字段类型格式说明参见 [crm-schema](../wecomcli-crm/references/crm-schema.md)

| 字段类型 | 写入格式 | 示例 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 姓名、公司、职位、热度分等 |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 来源、关系类型、重要程度、互动类型等 |

---

## 实操注意事项

### 常见踩坑

1. **更新时不传 key_type** — 必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **API 调用失败时** — 检查错误码：301085 表示 docid 无效（改用 url）；2022004 表示字段未找到（先 add_fields）；2022003 表示记录不存在（先 add_records）。遇到错误先用 `smartsheet_get_sheet '{"url": "<URL>"}'` 验证表结构。
3. **SINGLE_SELECT 选项不匹配** — 选项文本必须与表中已有选项精确一致
4. **SINGLE_SELECT 创建时 options 不生效** — `smartsheet_add_fields` 传入的 `property.options` 不会预创建选项，选项在首次写入记录时通过 `[{"text": "选项"}]` 自动创建。创建字段时可省略 `property` 参数。
5. **通讯录匹配** — `contact get_userlist` 返回的是 userid，需要和消息中的 chatid 对应
6. **消息扫描范围** — 会话列表一次最多返回有限数量，需要分批拉取
7. **去重逻辑** — 同一人可能在不同场景出现，AI 需要智能去重合并

### 与 wecomcli-crm 的区别

| 维度 | wecomcli-crm | wecomcli-workflow-personal-crm |
|------|-------------|-------------------------------|
| 定位 | 商业客户管理 | 个人关系网络 |
| 数据源 | 手动录入/外部导入 | 日程+消息自动扫描 |
| 热度分 | 无 | 有（自动计算） |
| 互动记录 | 无 | 有（自动追踪） |
| 超期提醒 | 无 | 有（自动提醒） |

### 数据联动

- 本 Skill 数据会被以下 Skill 读取：
  - `wecomcli-workflow-morning-brief` — 晨间简报中显示参会人背景
  - `wecomcli-workflow-business-advisor` — 关系/BD 顾问分析人脉动态
