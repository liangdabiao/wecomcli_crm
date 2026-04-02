---
name: wecomcli-workflow-content-pipeline
description: 企业微信内容创作管道技能。用户提出内容创意，AI 自动研究话题、检查重复选题、评估价值、生成完整方案并创建待办跟踪。当用户说"内容创意"、"灵感记录"、"选题评估"、"内容方案"、"创意管道"、"我想写一篇关于"等涉及内容创作管理的场景时触发。
---

# 企业微信内容创作管道

通过 wecom-cli 操作企业微信智能表格、文档和待办，管理从灵感到完整方案的内容创作全流程。

## 首次配置（自动初始化）

**执行任何操作前，先检查 memory 中是否已有内容管道配置。** 如果没有，按以下流程自动创建：

### 步骤 1：创建智能表格

```bash
wecom-cli doc create_doc '{"doc_type": 10, "doc_name": "内容创意管道"}'
```

从返回结果中获取 `docid`。

### 步骤 2：创建子表和字段

```bash
# 创建子表1: 创意库
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "创意库"}'

# 创建子表2: 创意研究
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "创意研究"}'
```

**创意库字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "创意标题", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "来源", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "话题领域", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "评估分数", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "状态", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "关联待办ID", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "方案文档链接", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "创建时间", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

**创意研究字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "关联创意", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "研究来源", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "研究类型", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "摘要", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "相关性", "field_type": "FIELD_TYPE_SINGLE_SELECT"}
]}'
```

### 步骤 3：保存配置到 memory

将以下信息保存到 memory 文件 `content-pipeline-config.md`：

```markdown
---
name: content-pipeline-config
description: 内容创意管道智能表格配置信息
type: project
---

## 内容管道表配置

- docid: <DOCID>
- url: <创建时返回的URL>
- 表格名称: 内容创意管道
- 创意库 sheet_id: <SHEET_ID>
- 创意研究 sheet_id: <SHEET_ID>
```

---

## 操作流程

> 以下命令中 `<DOCID>` 和各 `<SHEET_ID>` 从 memory 的 content-pipeline-config 中读取。

### 模式 1：创意评估（完整管道）

用户提出一个创意，AI 自动完成全流程。

**交互流程：**
1. 用户："我有个内容创意，想写一篇关于 AI Agent 在企业落地的文章"
2. AI 执行以下步骤

**Step 1/5：搜索话题最新动态**

```bash
# AI 使用 WebSearch 搜索相关话题
# 搜索关键词："AI Agent 企业落地 2026"、"AI Agent implementation enterprise"
```

**Step 2/5：检查知识库中是否有类似内容**

```bash
# 如果知识库 Skill 已初始化，读取索引
wecom-cli doc smartsheet_get_records '{"docid": "<知识库DOCID>", "sheet_id": "<知识库SHEET_ID>"}'
# AI 按标签和标题匹配检查重复
```

**Step 3/5：检查已有创意库避免重复**

```bash
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<创意库SHEET_ID>"}'
# AI 检查是否有类似选题
```

**Step 4/5：评估创意价值**

AI 从 5 个维度评分（每项 1-10）：

| 维度 | 权重 | 说明 |
|------|------|------|
| 时效性 | 25% | 话题是否当下热门 |
| 差异化 | 25% | 已有类似内容的多少 |
| 受众价值 | 20% | 目标受众关注程度 |
| 可执行性 | 15% | 资料丰富度、可产出的难度 |
| 延展性 | 15% | 是否可拆分为系列 |

**Step 5/5：生成完整方案**

包含：标题建议、内容类型、目标字数、详细大纲。

### 模式 2：保存创意

将评估结果保存到创意库。

```bash
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<创意库SHEET_ID>", "records": [{"values": {
  "创意标题": [{"type": "text", "text": "AI Agent 在企业落地实战"}],
  "来源": [{"type": "text", "text": "用户创意"}],
  "话题领域": [{"type": "text", "text": "科技"}],
  "评估分数": [{"type": "text", "text": "7.6"}],
  "状态": [{"text": "灵感"}],
  "关联待办ID": [{"type": "text", "text": ""}],
  "方案文档链接": [{"type": "text", "text": ""}],
  "创建时间": [{"type": "text", "text": "2026-04-02"}]
}}]}'
```

### 模式 3：生成方案文档

为高评分创意生成详细方案文档。

```bash
# 创建方案文档
wecom-cli doc create_doc '{"doc_type": 3, "doc_name": "方案：AI Agent 企业落地实战"}'

# 写入方案内容
wecom-cli doc edit_doc_content '{"docid": "<方案DOCID>", "content": "# 方案：AI Agent 在企业落地实战\n\n## 评估总分：7.6/10\n\n### 评分明细\n| 维度 | 评分 | 说明 |\n|------|------|------|\n| 时效性 | 8/10 | AI Agent 是当下热门话题 |\n| 差异化 | 6/10 | 已有不少类似内容 |\n| 受众价值 | 9/10 | 企业用户高度关注 |\n| 可执行性 | 8/10 | 案例丰富 |\n| 延展性 | 7/10 | 可拆分为系列 |\n\n### 标题建议\n1. AI Agent 落地实战：5 个真实案例拆解\n2. 企业如何用 AI Agent 提效 300%\n\n### 大纲\n1. AI Agent 不是聊天机器人\n2. 案例一：自动化客服 — 成本降 60%\n3. 案例二：代码审查 — Bug 率降 40%\n...\n\n### 目标\n- 类型：深度文章\n- 字数：3000-5000 字", "content_type": 1}'

# 更新创意库状态
wecom-cli doc smartsheet_update_records '{"docid": "<DOCID>", "sheet_id": "<创意库SHEET_ID>", "key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE", "records": [{"record_id": "RECORD_ID", "values": {
  "状态": [{"text": "方案"}],
  "方案文档链接": [{"type": "text", "text": "https://doc.weixin.qq.com/docx/<DOCID>"}]
}}]}'
```

### 模式 4：创建跟踪待办

为创意创建企业微信待办跟踪执行进度。

```bash
wecom-cli todo create_todo '{"content": "创作「AI Agent 企业落地实战」（来源：内容创意管道）", "remind_time": "2026-04-09 09:00:00"}'

# 保存 todo_id 到创意库
wecom-cli doc smartsheet_update_records '{"docid": "<DOCID>", "sheet_id": "<创意库SHEET_ID>", "key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE", "records": [{"record_id": "RECORD_ID", "values": {
  "状态": [{"text": "执行中"}],
  "关联待办ID": [{"type": "text", "text": "<TODO_ID>"}]
}}]}'
```

### 模式 5：查看创意库

查询所有创意及其状态。

```bash
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<创意库SHEET_ID>"}'
```

**输出格式示例：**

```
| 创意标题 | 领域 | 评分 | 状态 |
|---------|------|------|------|
| AI Agent 落地实战 | 科技 | 7.6 | 执行中 |
| 飞书自动化指南 | 科技 | 8.2 | 方案 |
| 投资入门系列 | 商业 | 6.5 | 灵感 |
```

### 模式 6：保存研究资料

将搜索到的参考资料保存到创意研究表。

```bash
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<创意研究SHEET_ID>", "records": [{"values": {
  "关联创意": [{"type": "text", "text": "AI Agent 企业落地实战"}],
  "研究来源": [{"type": "text", "text": "https://example.com/agent-cases"}],
  "研究类型": [{"text": "网页"}],
  "摘要": [{"type": "text", "text": "介绍了3个AI Agent在客服、编程、数据分析中的实际应用案例..."}],
  "相关性": [{"text": "高"}]
}}]}'
```

---

## 字段类型写入格式（通用规则）

> 完整字段类型格式说明参见 [crm-schema](../wecomcli-crm/references/crm-schema.md)

| 字段类型 | 写入格式 | 示例 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 创意标题、来源、领域、分数等 |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 状态、研究类型、相关性等 |

---

## 实操注意事项

### 常见踩坑

1. **更新时不传 key_type** — 必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **API 调用失败时** — 检查错误码：301085 表示 docid 无效（改用 url）；2022004 表示字段未找到（先 add_fields）；2022003 表示记录不存在（先 add_records）。遇到错误先用 `smartsheet_get_sheet '{"url": "<URL>"}'` 验证表结构。
3. **评估分数** — 以 TEXT 类型存储，方便 AI 进行数值比较和排序
4. **创意去重** — 评估前先检查创意库和知识库，避免重复选题
5. **状态流转** — 灵感 → 研究中 → 方案 → 执行中 → 已完成/已放弃
6. **SINGLE_SELECT 创建时 options 不生效** — `smartsheet_add_fields` 传入的 `property.options` 不会预创建选项，选项在首次写入记录时通过 `[{"text": "选项"}]` 自动创建。创建字段时可省略 `property` 参数。
7. **知识库联动** — 需要 `wecomcli-workflow-knowledge-base` 已初始化才能检查重复

### 数据联动

- 本 Skill 数据会被以下 Skill 读取：
  - `wecomcli-workflow-business-advisor` — 内容策略顾问分析选题质量和发布节奏
