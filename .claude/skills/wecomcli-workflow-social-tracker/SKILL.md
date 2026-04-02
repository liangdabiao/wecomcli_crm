---
name: wecomcli-workflow-social-tracker
description: 企业微信社交媒体数据追踪技能。维护社交媒体账号配置，记录每日数据快照，支持趋势分析和跨平台对比。当用户说"社交数据"、"平台数据"、"粉丝趋势"、"内容表现"、"数据追踪"、"添加账号"、"YouTube/Instagram/TikTok"等涉及社交媒体管理的场景时触发。
---

# 企业微信社交媒体数据追踪

通过 wecom-cli 操作企业微信智能表格，记录和追踪社交媒体账号的运营数据。

## 首次配置（自动初始化）

**执行任何操作前，先检查 memory 中是否已有社交追踪配置。** 如果没有，按以下流程自动创建：

### 步骤 1：创建智能表格

```bash
wecom-cli doc create_doc '{"doc_type": 10, "doc_name": "社交媒体追踪"}'
```

从返回结果中获取 `docid`。

### 步骤 2：创建子表和字段

```bash
# 创建子表1: 社交账号配置
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "社交账号"}'

# 创建子表2: 每日快照
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "每日快照"}'

# 创建子表3: 内容表现
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "内容表现"}'
```

获取各子表的 `sheet_id`，然后添加字段：

**社交账号字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "平台", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "账号名", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "主页链接", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "状态", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "备注", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

**每日快照字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "平台", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "账号名", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "日期", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "粉丝数", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "新增粉丝", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "内容总数", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "互动率", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "备注", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

**内容表现字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "平台", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "账号名", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "内容ID", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "标题", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "发布日期", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "点赞", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "评论", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "分享", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "播放数", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "备注", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

### 步骤 3：保存配置到 memory

将以下信息保存到 memory 文件 `social-tracker-config.md`：

```markdown
---
name: social-tracker-config
description: 社交媒体追踪智能表格配置信息，包含 docid、各子表 sheet_id
type: project
---

## 社交追踪表配置

- docid: <DOCID>
- url: <创建时返回的URL>
- 表格名称: 社交媒体追踪
- 社交账号 sheet_id: <SHEET_ID>
- 每日快照 sheet_id: <SHEET_ID>
- 内容表现 sheet_id: <SHEET_ID>
```

---

## 操作流程

> 以下命令中 `<DOCID>` 和各 `<SHEET_ID>` 从 memory 的 social-tracker-config 中读取。

### 模式 1：账号管理

**添加账号：**

```bash
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<社交账号SHEET_ID>", "records": [{"values": {
  "平台": [{"text": "YouTube"}],
  "账号名": [{"type": "text", "text": "@mychannel"}],
  "主页链接": [{"type": "text", "text": "https://youtube.com/@mychannel"}],
  "状态": [{"text": "活跃"}],
  "备注": [{"type": "text", "text": ""}]
}}]}'
```

**查询已添加的账号：**

```bash
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<社交账号SHEET_ID>"}'
```

**更新账号状态：**

```bash
wecom-cli doc smartsheet_update_records '{"docid": "<DOCID>", "sheet_id": "<社交账号SHEET_ID>", "key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE", "records": [{"record_id": "RECORD_ID", "values": {
  "状态": [{"text": "暂停"}]
}}]}'
```

### 模式 2：手动录入快照

用户直接告诉 AI 数据，AI 计算增量并写入。

**交互流程：**
1. 用户："YouTube 今天 1000 粉丝了，发了 3 条内容，互动率 4.5%"
2. AI 查询上一次快照，计算新增粉丝
3. 写入快照记录
4. 如果达到里程碑（如每 500 粉丝），发送消息通知

```bash
# 先查询上一次快照获取上次粉丝数
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<每日快照SHEET_ID>"}'

# 写入新快照
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<每日快照SHEET_ID>", "records": [{"values": {
  "平台": [{"type": "text", "text": "YouTube"}],
  "账号名": [{"type": "text", "text": "@mychannel"}],
  "日期": [{"type": "text", "text": "2026-04-02"}],
  "粉丝数": [{"type": "text", "text": "1000"}],
  "新增粉丝": [{"type": "text", "text": "12"}],
  "内容总数": [{"type": "text", "text": "45"}],
  "互动率": [{"type": "text", "text": "4.5%"}],
  "备注": [{"type": "text", "text": ""}]
}}]}'
```

### 模式 3：趋势分析

查询指定时间范围的快照数据，生成趋势报告。

**交互流程：**
1. 用户："这周哪个平台增长最快？"
2. AI 查询过去 7 天快照
3. 计算增长率、内容产出
4. 生成对比表格

```bash
# 查询所有快照记录
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<每日快照SHEET_ID>"}'

# AI 根据日期过滤后计算：
# - 每个平台的起始粉丝数和结束粉丝数
# - 增长率 = (结束-起始) / 起始 * 100%
# - 新增内容数
# - 生成趋势报告
```

**趋势报告格式示例：**

```
## 社交媒体趋势报告（3/26-4/2）

| 平台 | 起始粉丝 | 结束粉丝 | 增长率 | 新增内容 |
|------|---------|---------|--------|---------|
| YouTube | 950 | 1000 | +5.3% | 3 |
| Instagram | 500 | 550 | +10% | 8 |

增长最快：Instagram（+10%）
```

### 模式 4：记录内容表现

用户手动录入单条内容的表现数据。

```bash
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<内容表现SHEET_ID>", "records": [{"values": {
  "平台": [{"type": "text", "text": "YouTube"}],
  "账号名": [{"type": "text", "text": "@mychannel"}],
  "内容ID": [{"type": "text", "text": "vid_001"}],
  "标题": [{"type": "text", "text": "AI编程工具对比"}],
  "发布日期": [{"type": "text", "text": "2026-04-01"}],
  "点赞": [{"type": "text", "text": "150"}],
  "评论": [{"type": "text", "text": "32"}],
  "分享": [{"type": "text", "text": "18"}],
  "播放数": [{"type": "text", "text": "5000"}],
  "备注": [{"type": "text", "text": ""}]
}}]}'
```

**查询最佳表现内容：**

```bash
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<内容表现SHEET_ID>"}'
# AI 按播放数/互动率排序，找出表现最好的内容
```

---

## 里程碑通知

当账号粉丝数达到里程碑时，通过消息通知用户：

```bash
wecom-cli msg send_message '{"chat_type": 1, "chatid": "<USERID>", "msgtype": "text", "text": {"content": "YouTube @mychannel 粉丝突破 1000！本周新增 12 粉丝。"}}'
```

---

## 数据来源说明

当前阶段支持三种数据来源：
1. **AI WebSearch** — 搜索 Social Blade 等公开数据获取粉丝信息
2. **用户手动录入** — 用户直接告诉 AI 数值
3. **API 接入（未来）** — 预留 YouTube/Instagram/Twitter/TikTok API 接入点

---

## 字段类型写入格式（通用规则）

> 完整字段类型格式说明参见 [crm-schema](../wecomcli-crm/references/crm-schema.md)

| 字段类型 | 写入格式 | 示例 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 账号名、粉丝数、日期等 |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 平台、状态等单选字段 |

> 数字字段（粉丝数、互动率等）以 TEXT 类型存储，AI 读取后自行转换为数字进行计算。

---

## 实操注意事项

### 常见踩坑

1. **更新时不传 key_type** — 字段标题无法识别，必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **API 调用失败时** — 检查错误码：301085 表示 docid 无效（改用 url）；2022004 表示字段未找到（先 add_fields）；2022003 表示记录不存在（先 add_records）。遇到错误先用 `smartsheet_get_sheet '{"url": "<URL>"}'` 验证表结构。
3. **SINGLE_SELECT 选项不匹配** — 平台名和状态必须与表中已有选项完全一致
4. **SINGLE_SELECT 创建时 options 不生效** — `smartsheet_add_fields` 传入的 `property.options` 不会预创建选项，选项在首次写入记录时通过 `[{"text": "选项"}]` 自动创建。创建字段时可省略 `property` 参数。
5. **新增粉丝需手动计算** — 查询上次快照，用当前粉丝数减去上次粉丝数
6. **日期格式统一** — 使用 `YYYY-MM-DD` 格式

### 定时自动化建议

| 任务 | 建议时间 | Cron |
|------|---------|------|
| 每日快照采集 | 每天 22:30 | `30 22 * * *` |
| 周趋势报告 | 每周日 21:00 | `0 21 * * 0` |

### 数据联动

- 本 Skill 独立运行，但数据会被以下 Skill 读取：
  - `wecomcli-workflow-morning-brief` — 简报中包含昨日数据表现
  - `wecomcli-workflow-business-advisor` — 营销顾问和增长顾问分析用
