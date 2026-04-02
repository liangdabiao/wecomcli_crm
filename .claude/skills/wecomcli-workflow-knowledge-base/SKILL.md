---
name: wecomcli-workflow-knowledge-base
description: 企业微信个人知识库技能。AI 自动抓取链接内容生成摘要和标签，存入企业微信文档和智能表格索引，支持自然语言搜索和内容关联分析。当用户说"知识库"、"保存文章"、"搜索知识库"、"之前保存过"、"知识管理"、"帮我保存这个链接"等涉及知识管理的场景时触发。
---

# 企业微信个人知识库

通过 wecom-cli 操作企业微信文档和智能表格，构建个人知识管理系统。用智能表格做索引，用文档做内容存储。

> 注意：企业微信无飞书"知识库"模块，本 Skill 用**智能表格索引 + 云文档**替代。

## 首次配置（自动初始化）

**执行任何操作前，先检查 memory 中是否已有知识库配置。** 如果没有，按以下流程自动创建：

### 步骤 1：创建智能表格（索引表）

```bash
wecom-cli doc create_doc '{"doc_type": 10, "doc_name": "个人知识库索引"}'
```

从返回结果中获取 `docid`。

### 步骤 2：创建子表和字段

```bash
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "知识索引"}'
```

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "标题", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "URL", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "内容类型", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "来源平台", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "标签", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "摘要", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "关键观点", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "文档链接", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "保存时间", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "关联条目", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

### 步骤 3：保存配置到 memory

将以下信息保存到 memory 文件 `knowledge-base-config.md`：

```markdown
---
name: knowledge-base-config
description: 个人知识库智能表格配置信息
type: project
---

## 知识库配置

- 索引表 docid: <DOCID>
- 索引表 url: <创建时返回的URL>
- 索引表 sheet_id: <SHEET_ID>
- 表格名称: 个人知识库索引
```

---

## 操作流程

> 以下命令中 `<DOCID>` 和 `<SHEET_ID>` 从 memory 的 knowledge-base-config 中读取。

### 模式 1：保存链接内容

用户发送链接，AI 自动抓取、生成摘要、保存到文档和索引表。

**步骤 1：用户发送链接**

用户："帮我保存这篇文章 https://example.com/ai-trends-2026"

**步骤 2：AI 抓取内容并生成摘要**

使用 WebSearch/WebFetch 工具抓取网页内容，AI 自动：
- 提取标题
- 生成 200 字以内的摘要
- 自动生成标签（3-8 个）
- 提取关键观点
- 判断内容类型和来源平台

**步骤 3：检查重复**

```bash
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>"}'
# AI 比对 URL 或标题是否已存在
```

**步骤 4：检查关联内容**

AI 分析新内容与已有条目的关联性（标签重叠、主题相似），找到相关条目。

**步骤 5：创建文档保存完整内容**

```bash
wecom-cli doc create_doc '{"doc_type": 3, "doc_name": "AI行业十大趋势 2026"}'
```

获取返回的 `docid`（文档 docid），然后写入内容：

```bash
wecom-cli doc edit_doc_content '{"docid": "<文档DOCID>", "content": "# AI行业十大趋势 2026\n\n> 来源：https://example.com/ai-trends-2026\n> 保存时间：2026-04-02\n> 标签：AI、趋势、2026、行业分析、大模型\n\n---\n\n## 原文内容\n\n<完整文章内容>\n\n---\n\n## AI 摘要\n\n<200字摘要>\n\n## 关键观点\n\n1. ...\n2. ...\n3. ...", "content_type": 1}'
```

**步骤 6：写入索引表**

```bash
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "records": [{"values": {
  "标题": [{"type": "text", "text": "AI行业十大趋势 2026"}],
  "URL": [{"type": "text", "text": "https://example.com/ai-trends-2026"}],
  "内容类型": [{"text": "文章"}],
  "来源平台": [{"text": "网站"}],
  "标签": [{"type": "text", "text": "AI, 趋势, 2026, 行业分析, 大模型"}],
  "摘要": [{"type": "text", "text": "本文分析了2026年AI行业的十大发展趋势，包括大模型成本下降、多模态应用爆发等..."}],
  "关键观点": [{"type": "text", "text": "1. 大模型成本持续下降 2. 多模态成为标配 3. AI Agent大规模落地"}],
  "文档链接": [{"type": "text", "text": "https://doc.weixin.qq.com/docx/<DOCID>"}],
  "保存时间": [{"type": "text", "text": "2026-04-02"}],
  "关联条目": [{"type": "text", "text": "大模型成本下降分析"}]
}}]}'
```

### 模式 2：批量保存

用户一次发送多个链接。

**交互流程：**
1. 用户："把这几个链接都保存一下\nhttps://a.com/article1\nhttps://b.com/video2\nhttps://c.com/thread3"
2. AI 逐个抓取、生成摘要
3. 检查重复
4. 批量创建文档和索引

### 模式 3：搜索知识库

用自然语言搜索已保存的内容。

**交互示例：**

| 用户问 | AI 处理 |
|--------|--------|
| "关于 AI 芯片的文章有哪些？" | 搜索标签含"AI"+"芯片"，或摘要中含"芯片" |
| "谁写过关于 OpenAI 估值分析的？" | 搜索标题和摘要含"OpenAI"+"估值" |
| "之前保存过一个关于 Nvidia 的视频" | 搜索标签含"Nvidia"且类型="视频" |

```bash
# 获取所有索引记录
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>"}'

# AI 在内存中按关键词、标签、类型进行过滤和匹配
# 返回匹配结果列表，包含标题、标签、摘要、文档链接
```

### 模式 4：关联发现

新保存内容时自动发现关联。

**AI 处理逻辑：**
1. 提取新内容的标签集合
2. 遍历已有索引记录，计算标签重叠度
3. 重叠度 > 30% 的标记为关联条目
4. 在保存时告知用户："发现 1 条关联内容：你三周前保存过《大模型成本下降分析》，从另一个角度讨论了AI行业。"

---

## 支持的内容格式

| 格式 | 处理方式 |
|------|---------|
| 文章网页 | WebFetch 抓取全文 → 生成摘要 |
| YouTube 视频 | WebSearch 搜索视频信息 → 提取标题/描述 |
| Twitter/X 推文 | WebFetch 抓取（含完整帖子串） |
| PDF | 通过 URL 下载 → AI 提取内容 |
| 播客 | WebSearch 搜索节目信息 → 提取摘要 |

---

## 字段类型写入格式（通用规则）

> 完整字段类型格式说明参见 [crm-schema](../wecomcli-crm/references/crm-schema.md)

| 字段类型 | 写入格式 | 示例 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 标题、URL、标签、摘要等 |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 内容类型、来源平台等 |

---

## 实操注意事项

### 常见踩坑

1. **更新时不传 key_type** — 必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **API 调用失败时** — 检查错误码：301085 表示 docid 无效（改用 url）；2022004 表示字段未找到（先 add_fields）；2022003 表示记录不存在（先 add_records）。遇到错误先用 `smartsheet_get_sheet '{"url": "<URL>"}'` 验证表结构。
3. **SINGLE_SELECT 选项不匹配** — 内容类型和来源平台必须与已有选项精确匹配
4. **SINGLE_SELECT 创建时 options 不生效** — `smartsheet_add_fields` 传入的 `property.options` 不会预创建选项，选项在首次写入记录时通过 `[{"text": "选项"}]` 自动创建。创建字段时可省略 `property` 参数。
5. **文档内容写入** — `edit_doc_content` 的 content 是 Markdown 格式，`content_type: 1` 表示 Markdown
6. **URL 重复检查** — 完整 URL 精确匹配 + 标题模糊匹配双重去重
7. **WebFetch 失败** — 如果抓取失败，告知用户并建议手动提供内容

### 知识库 vs wecomcli-crm

| 维度 | 知识库 | CRM |
|------|--------|-----|
| 存储对象 | 文章/视频/资料 | 客户信息 |
| 存储方式 | 文档(全文) + 索引表 | 智能表格 |
| 搜索方式 | 标签 + 关键词 + 摘要 | 字段精确匹配 |
| 自动化 | 自动抓取+摘要 | 手动录入为主 |

### 数据联动

- 本 Skill 数据会被以下 Skill 读取：
  - `wecomcli-workflow-business-advisor` — 竞争/行业顾问分析行业趋势
  - `wecomcli-workflow-content-pipeline` — 检查重复选题
