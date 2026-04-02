---
name: wecomcli-workflow-business-advisor
description: 企业微信AI商业顾问团技能。8个AI专家角色自动分析业务数据，汇总去重后发送编号清单，支持展开详细分析。当用户说"商业分析"、"业务报告"、"顾问分析"、"夜间分析"、"业务建议"、"帮我分析一下业务"等涉及综合业务分析的场景时触发。
---

# 企业微信 AI 商业顾问团

通过 8 个 AI 顾问角色自动分析业务数据，生成综合商业分析报告。

## 首次配置（自动初始化）

**执行任何操作前，先检查 memory 中是否已有商业顾问团配置。** 如果没有，按以下流程自动创建：

### 步骤 1：创建智能表格

```bash
wecom-cli doc create_doc '{"doc_type": 10, "doc_name": "商业顾问团"}'
```

从返回结果中获取 `docid`。

### 步骤 2：创建子表和字段

```bash
# 创建子表1: 顾问分析记录
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "分析记录"}'

# 创建子表2: 用户反馈
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "用户反馈"}'
```

**分析记录字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "分析日期", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "顾问角色", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "分析摘要", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "优先级", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "状态", "field_type": "FIELD_TYPE_SINGLE_SELECT"}
]}'
```

**用户反馈字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "分析ID", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "用户选择", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "展开详情", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "处理时间", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

### 步骤 3：保存配置到 memory

将以下信息保存到 memory 文件 `business-advisor-config.md`：

```markdown
---
name: business-advisor-config
description: 商业顾问团智能表格配置信息
type: project
---

## 商业顾问团表配置

- docid: <DOCID>
- url: <创建时返回的URL>
- 表格名称: 商业顾问团
- 分析记录 sheet_id: <SHEET_ID>
- 用户反馈 sheet_id: <SHEET_ID>
```

---

## 操作流程

> 以下命令中 `<DOCID>` 和各 `<SHEET_ID>` 从 memory 的 business-advisor-config 中读取。
> 顾问角色 Prompt 模板见 [prompt-templates.md](references/prompt-templates.md)。

### 模式 1：运行完整分析

**步骤 1：采集所有可用数据**

```bash
# 联系人数据（如果个人 CRM 已初始化）
wecom-cli doc smartsheet_get_records '{"docid": "<个人CRM_DOCID>", "sheet_id": "<联系人SHEET_ID>"}'

# 社交数据（如果社交追踪已初始化）
wecom-cli doc smartsheet_get_records '{"docid": "<社交追踪_DOCID>", "sheet_id": "<每日快照SHEET_ID>"}'
wecom-cli doc smartsheet_get_records '{"docid": "<社交追踪_DOCID>", "sheet_id": "<内容表现SHEET_ID>"}'

# 创意库数据（如果内容管道已初始化）
wecom-cli doc smartsheet_get_records '{"docid": "<内容管道_DOCID>", "sheet_id": "<创意库SHEET_ID>"}'

# 投资数据（如果股票分析已初始化）
wecom-cli doc smartsheet_get_records '{"docid": "<股票分析_DOCID>", "sheet_id": "<自选股SHEET_ID>"}'
wecom-cli doc smartsheet_get_records '{"docid": "<股票分析_DOCID>", "sheet_id": "<分析记录SHEET_ID>"}'

# 待办数据
wecom-cli todo get_todo_list '{}'

# 日程数据
wecom-cli schedule get_schedule_list_by_range '{"start_time": "<今天>", "end_time": "<今天>"}'
```

**步骤 2：8 个顾问并行分析**

AI 使用 8 个顾问角色 prompt 模板（见 references/prompt-templates.md），每个顾问独立分析可用数据。

> 如果某个数据源未初始化，对应顾问标注"数据不足，跳过"。

**步骤 3：汇总去重排序**

AI 使用汇总秘书 prompt，合并所有顾问的建议，去重排序。

**步骤 4：发送编号清单**

```bash
wecom-cli msg send_message '{"chat_type": 1, "chatid": "<USERID>", "msgtype": "text", "text": {"content": "## 今日商业顾问团报告（4/2）\n\n### 总览\n今日共 6 条建议，来自 5 位顾问。\n数据来源：CRM(有)、社交追踪(有)、创意库(有)、投资(无)\n\n### 建议清单\n1. 【关系】张三（XX科技）已 45 天未联系，建议主动跟进\n2. 【财务】投资组合本周下跌 5%，建议检查持仓比例\n3. 【内容】本周发布频率下降 40%，建议安排时间完成选题\n4. 【运营】7 个任务已过期未完成\n5. 【风险】竞品 YY 本周发布新功能\n6. 【增长】Instagram 增长势头良好，建议加大投入\n\n回复编号展开详细分析"}}'
```

**步骤 5：保存分析记录**

```bash
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<分析记录SHEET_ID>", "records": [
  {"values": {"分析日期": [{"type":"text","text":"2026-04-02"}], "顾问角色": [{"text":"关系BD顾问"}], "分析摘要": [{"type":"text","text":"张三已45天未联系"}], "优先级": [{"text":"高"}], "状态": [{"text":"待读"}]}},
  {"values": {"分析日期": [{"type":"text","text":"2026-04-02"}], "顾问角色": [{"text":"财务顾问"}], "分析摘要": [{"type":"text","text":"投资组合本周下跌5%"}], "优先级": [{"text":"中"}], "状态": [{"text":"待读"}]}}
]}'
```

### 模式 2：展开详细分析

用户回复编号，AI 展开该条建议的详细分析。

**交互流程：**
1. 用户："第 3 条详细说说"
2. AI 找到对应建议
3. 使用来源顾问的 prompt 进行深度分析
4. 输出详细分析（原始建议、数据支撑、建议行动步骤）

**输出格式：**

```
## 第 3 条展开分析

### 原始建议
本周内容发布频率下降 40%，建议安排时间完成待执行选题。

### 来源顾问：内容策略顾问

### 详细分析
- 上周发布 5 条内容，本周仅 3 条
- 创意库中有 4 个"方案已完成"状态但未执行
- YouTube 表现最好的内容类型是教程类（互动率 4.2%）

### 建议行动步骤
1. 优先执行创意库中评分最高的"AI编程工具对比"选题
2. 本周至少发布 2 条教程类内容
3. 安排 30 分钟完成"企业微信自动化"选题的资料整理
```

### 模式 3：保存归档报告

将每日分析保存为企业微信文档归档。

```bash
wecom-cli doc create_doc '{"doc_type": 3, "doc_name": "商业顾问团报告 2026-04-02"}'

wecom-cli doc edit_doc_content '{"docid": "<报告DOCID>", "content": "# 商业顾问团报告 — 2026年4月2日\n\n## 总览\n...\n\n## 各顾问分析\n### 财务顾问\n...\n### 营销顾问\n...\n\n## 汇总建议清单\n1. ...\n2. ...", "content_type": 1}'
```

---

## 8 个顾问角色

| 角色 | 关注领域 | 数据来源 |
|------|---------|---------|
| 财务顾问 | 收入趋势、投资表现 | 股票分析 |
| 营销顾问 | 内容表现、互动率 | 社交追踪 + 内容管道 |
| 增长顾问 | 粉丝增长、渠道效率 | 社交追踪 |
| 运营顾问 | 任务完成率、日程密度 | 待办 + 日程 |
| 内容策略顾问 | 选题质量、发布节奏 | 内容管道 + 社交追踪 |
| 关系/BD 顾问 | 人脉动态、合作机会 | 个人 CRM |
| 竞争/行业顾问 | 行业趋势、竞品动态 | 知识库 + WebSearch |
| 综合风险顾问 | 潜在风险、薄弱环节 | 全部数据 |

---

## 字段类型写入格式（通用规则）

> 完整字段类型格式说明参见 [crm-schema](../wecomcli-crm/references/crm-schema.md)

| 字段类型 | 写入格式 | 示例 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 分析摘要等 |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 顾问角色、优先级、状态等 |

---

## 实操注意事项

### 常见踩坑

1. **更新时不传 key_type** — 必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **API 调用失败时** — 检查错误码：301085 表示 docid 无效（改用 url）；2022004 表示字段未找到（先 add_fields）；2022003 表示记录不存在（先 add_records）。遇到错误先用 `smartsheet_get_sheet '{"url": "<URL>"}'` 验证表结构。
3. **数据源未初始化** — 联动 Skill 可能未初始化，AI 需要优雅降级（标注"数据不足"而非报错）
4. **消息长度限制** — 编号清单消息需要精简，详细内容在展开时提供
5. **分析记录去重** — 同一天多次运行分析时，只保留最新一次的记录
6. **SINGLE_SELECT 创建时 options 不生效** — `smartsheet_add_fields` 传入的 `property.options` 不会预创建选项，选项在首次写入记录时通过 `[{"text": "选项"}]` 自动创建。创建字段时可省略 `property` 参数。

### 定时自动化建议

| 任务 | 建议时间 | Cron |
|------|---------|------|
| 运行商业顾问团分析 | 工作日 22:00 | `0 22 * * 1-5` |
| 归档日报告 | 工作日 22:30 | `30 22 * * 1-5` |

### 数据联动依赖

本 Skill 读取以下 Skill 的数据（已初始化才有效）：
- `wecomcli-workflow-personal-crm` — 联系人动态
- `wecomcli-workflow-social-tracker` — 社交数据
- `wecomcli-workflow-content-pipeline` — 创意/内容数据
- `wecomcli-workflow-stock-analyzer` — 投资数据
