---
name: wecomcli-workflow-health-diary
description: 企业微信健康日记技能。通过企业微信消息记录饮食和身体状态，AI 分析饮食与身体状态的关联，生成健康周报。当用户说"记录饮食"、"健康日记"、"吃了什么"、"身体不舒服"、"健康追踪"、"午餐/晚餐吃了"等涉及健康记录的场景时触发。
---

# 企业微信健康日记

通过 wecom-cli 操作企业微信智能表格和文档，记录饮食和身体状态，发现饮食与健康之间的隐藏关联。

## 首次配置（自动初始化）

**执行任何操作前，先检查 memory 中是否已有健康日记配置。** 如果没有，按以下流程自动创建：

### 步骤 1：创建智能表格

```bash
wecom-cli doc create_doc '{"doc_type": 10, "doc_name": "健康日记"}'
```

从返回结果中获取 `docid`。

### 步骤 2：创建子表和字段

```bash
# 创建子表1: 饮食记录
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "饮食记录"}'

# 创建子表2: 身体状态
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "身体状态"}'

# 创建子表3: 关联分析
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "关联分析"}'
```

获取各子表的 `sheet_id`，然后为每个子表添加字段：

**饮食记录字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "日期", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "餐次", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "食物列表", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "份量说明", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "热量估算", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "备注", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

**身体状态字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "记录时间", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "状态描述", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "严重程度", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "持续时长", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "备注", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

**关联分析字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "食物", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "状态", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "出现次数", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "典型时间差", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "关联强度", "field_type": "FIELD_TYPE_SINGLE_SELECT"}
]}'
```

### 步骤 3：保存配置到 memory

将以下信息保存到 memory 文件 `health-diary-config.md`：

```markdown
---
name: health-diary-config
description: 健康日记智能表格配置信息，包含 docid、各子表 sheet_id
type: project
---

## 健康日记表配置

- docid: <DOCID>
- url: <创建时返回的URL>
- 表格名称: 健康日记
- 饮食记录 sheet_id: <SHEET_ID>
- 身体状态 sheet_id: <SHEET_ID>
- 关联分析 sheet_id: <SHEET_ID>
```

配置保存后，所有操作直接从 memory 读取，无需再次询问。

---

## 操作流程

> 以下命令中 `<DOCID>` 和各 `<SHEET_ID>` 从 memory 的 health-diary-config 中读取。

### 模式 1：文字描述记录饮食

用户用自然语言描述食物，AI 记录到智能表格。

**交互流程：**
1. 用户："午餐吃了红烧牛肉面和一杯冰美式"
2. AI 确认食物和份量
3. 用户确认后写入智能表格

```bash
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<饮食记录SHEET_ID>", "records": [{"values": {
  "日期": [{"type": "text", "text": "2026-04-02"}],
  "餐次": [{"text": "午餐"}],
  "食物列表": [{"type": "text", "text": "红烧牛肉面, 冰美式"}],
  "份量说明": [{"type": "text", "text": "一大碗, 大杯"}],
  "备注": [{"type": "text", "text": ""}]
}}]}'
```

### 模式 2：照片识别记录

用户发送食物照片到企业微信，AI 识别后记录。

**交互流程：**
1. 用户通过企业微信发送食物照片
2. AI 用多模态能力识别食物
3. 确认食物和份量后写入

```bash
# 获取最近消息中的图片
wecom-cli msg get_message '{"chat_type": 1, "chatid": "<USERID>", "begin_time": "<5分钟前>", "end_time": "<现在>"}'

# 下载图片后用 Claude 视觉能力识别
# 识别完成后写入饮食记录（命令同模式1）
```

### 模式 3：记录身体状态

用户报告身体不适，AI 记录并主动检查是否有饮食关联。

**交互流程：**
1. 用户："下午胃有点不舒服"
2. AI 查询最近饮食记录，检查是否有已知的食物-状态关联
3. 如果发现风险食物，主动提醒
4. 写入身体状态记录

```bash
# 写入身体状态
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<身体状态SHEET_ID>", "records": [{"values": {
  "记录时间": [{"type": "text", "text": "2026-04-02 15:00"}],
  "状态描述": [{"type": "text", "text": "胃部不适"}],
  "严重程度": [{"text": "轻微"}],
  "持续时长": [{"type": "text", "text": ""}],
  "备注": [{"type": "text", "text": ""}]
}}]}'

# 查询最近饮食记录检查关联
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<饮食记录SHEET_ID>"}'

# 查询已有关联分析
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<关联分析SHEET_ID>"}'
```

**主动提醒逻辑：**
- 查询关联分析表中"关联强度"="高"的记录
- 比对用户最近 6 小时的饮食记录
- 如果匹配到高风险食物，发送提醒消息

### 模式 4：关联分析

AI 分析饮食记录和身体状态记录，发现隐藏的关联模式。

**分析逻辑：**
1. 获取所有饮食记录和身体状态记录
2. 对每条身体状态记录，回溯 6-8 小时内的饮食
3. 统计"食物X → 状态Y"的出现频率
4. 出现 2 次以上标记为"中"，3 次以上标记为"高"
5. 计算典型时间差（食物摄入到症状出现的平均时间）

```bash
# 读取所有饮食记录
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<饮食记录SHEET_ID>"}'

# 读取所有身体状态
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<身体状态SHEET_ID>"}'

# 读取已有关联分析
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<关联分析SHEET_ID>"}'

# AI 执行关联分析后，更新关联分析表
wecom-cli doc smartsheet_update_records '{"docid": "<DOCID>", "sheet_id": "<关联分析SHEET_ID>", "key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE", "records": [{"record_id": "RECORD_ID", "values": {
  "出现次数": [{"type": "text", "text": "3"}],
  "关联强度": [{"text": "高"}]
}}]}'
```

### 模式 5：生成健康周报

每周生成一份健康报告，保存为企业微信文档。

**报告内容：**
1. 饮食概览（记录天数、总餐次、饮食规律性）
2. 身体状态（不适次数、主要症状）
3. 关联发现（食物-状态关联表格）
4. 建议（基于关联分析）

```bash
# 读取本周数据（假设 7 天）
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<饮食记录SHEET_ID>"}'
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<身体状态SHEET_ID>"}'
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<关联分析SHEET_ID>"}'

# AI 汇总分析后，创建报告文档
wecom-cli doc create_doc '{"doc_type": 3, "doc_name": "健康周报 2026-W14"}'

# 写入报告内容
wecom-cli doc edit_doc_content '{"docid": "<报告DOCID>", "content": "# 健康周报（3/27-4/2）\n\n## 饮食概览\n...\n\n## 身体状态\n...\n\n## 关联发现\n...\n\n## 建议\n...", "content_type": 1}'
```

---

## 热度分计算规则（关联强度）

| 条件 | 关联强度 |
|------|---------|
| 出现 1 次 | 低 |
| 出现 2-3 次 | 中 |
| 出现 4 次以上 | 高 |

---

## 字段类型写入格式（通用规则）

> 完整字段类型格式说明参见 [crm-schema](../wecomcli-crm/references/crm-schema.md)

| 字段类型 | 写入格式 | 示例 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 食物列表、日期、备注等文本字段 |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 餐次、严重程度、关联强度等单选字段 |

---

## 实操注意事项

### 常见踩坑

1. **更新时不传 key_type** — 字段标题无法识别，必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **API 调用失败时** — 检查错误码：301085 表示 docid 无效（改用 url）；2022004 表示字段未找到（先 add_fields）；2022003 表示记录不存在（先 add_records）。遇到错误先用 `smartsheet_get_sheet '{"url": "<URL>"}'` 验证表结构。
3. **SINGLE_SELECT 选项不匹配** — 传的文本必须与表中已有选项完全一致
4. **SINGLE_SELECT 创建时 options 不生效** — `smartsheet_add_fields` 传入的 `property.options` 不会预创建选项，选项在首次写入记录时通过 `[{"text": "选项"}]` 自动创建。创建字段时可省略 `property` 参数，直接写 `"field_type": "FIELD_TYPE_SINGLE_SELECT"` 即可
5. **日期格式** — 统一使用 `YYYY-MM-DD` 格式，时间使用 `YYYY-MM-DD HH:MM`
6. **关联分析时机** — 建议在每次记录身体状态时触发关联检查，每周日生成完整周报
7. **照片识别** — 需要用户先通过企业微信发送照片，然后通过 msg get_message 获取

### 定时自动化建议

| 任务 | 建议时间 | Cron |
|------|---------|------|
| 健康周报生成 | 每周日 20:00 | `0 20 * * 0` |
| 关联分析更新 | 每日 22:00 | `0 22 * * *` |

### 数据联动

- 本 Skill 独立运行，不依赖其他 Skill
- 健康周报可发送到企业微信消息通知用户
