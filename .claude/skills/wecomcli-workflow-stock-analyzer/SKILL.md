---
name: wecomcli-workflow-stock-analyzer
description: 企业微信股票研究助手技能。四层架构：自选股监控、财报提醒、深度研究（4个AI分析师并行）、自动选股。当用户说"股票分析"、"自选股"、"研究XX股票"、"持仓快照"、"财报分析"、"选股"、"添加自选"等涉及股票研究的场景时触发。
---

# 企业微信股票研究助手

通过 wecom-cli 操作企业微信智能表格和文档，构建完整的股票研究系统。

## 首次配置（自动初始化）

**执行任何操作前，先检查 memory 中是否已有股票分析配置。** 如果没有，按以下流程自动创建：

### 步骤 1：创建智能表格

```bash
wecom-cli doc create_doc '{"doc_type": 10, "doc_name": "股票研究"}'
```

从返回结果中获取 `docid`。

### 步骤 2：创建子表和字段

```bash
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "自选股"}'
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "持仓"}'
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "分析记录"}'
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "财报日程"}'
```

**自选股字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "股票代码", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "股票名称", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "市场", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "目标价", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "方向", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "当前价", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "备注", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

**持仓字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "股票代码", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "股票名称", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "买入价", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "持仓量", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "买入日期", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "备注", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

**分析记录字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "股票代码", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "分析日期", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "分析类型", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "结论摘要", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "看多理由", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "看空理由", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "信心评分", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "建议", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "报告链接", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

**财报日程字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "股票代码", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "股票名称", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "财报日期", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "状态", "field_type": "FIELD_TYPE_SINGLE_SELECT"}
]}'
```

### 步骤 3：保存配置到 memory

将以下信息保存到 memory 文件 `stock-analyzer-config.md`：

```markdown
---
name: stock-analyzer-config
description: 股票研究助手智能表格配置信息
type: project
---

## 股票研究表配置

- docid: <DOCID>
- url: <创建时返回的URL>
- 表格名称: 股票研究
- 自选股 sheet_id: <SHEET_ID>
- 持仓 sheet_id: <SHEET_ID>
- 分析记录 sheet_id: <SHEET_ID>
- 财报日程 sheet_id: <SHEET_ID>
```

---

## 操作流程

> 以下命令中 `<DOCID>` 和各 `<SHEET_ID>` 从 memory 的 stock-analyzer-config 中读取。

### 第一层：自选股监控

**添加自选股：**

```bash
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<自选股SHEET_ID>", "records": [{"values": {
  "股票代码": [{"type": "text", "text": "AAPL"}],
  "股票名称": [{"type": "text", "text": "Apple"}],
  "市场": [{"text": "美股"}],
  "目标价": [{"type": "text", "text": "200"}],
  "方向": [{"text": "看涨"}],
  "当前价": [{"type": "text", "text": "185"}],
  "备注": [{"type": "text", "text": ""}]
}}]}'
```

**更新当前价并检查目标价：**

```bash
# 先查询所有自选股
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<自选股SHEET_ID>"}'

# AI 使用 WebSearch 获取最新股价
# 对比目标价，如果到达则发送通知

# 更新当前价
wecom-cli doc smartsheet_update_records '{"docid": "<DOCID>", "sheet_id": "<自选股SHEET_ID>", "key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE", "records": [{"record_id": "RECORD_ID", "values": {
  "当前价": [{"type": "text", "text": "198"}]
}}]}'

# 发送目标价到达通知
wecom-cli msg send_message '{"chat_type": 1, "chatid": "<USERID>", "msgtype": "text", "text": {"content": "目标价提醒：AAPL 当前 $198，接近目标价 $200（看涨）"}}'
```

### 第二层：持仓快照

**生成持仓快照：**

```bash
# 查询持仓
wecom-cli doc smartsheet_get_records '{"docid": "<DOCID>", "sheet_id": "<持仓SHEET_ID>"}'

# AI 使用 WebSearch 获取各股票当前价
# 计算市值 = 当前价 × 持仓量
# 计算盈亏 = (当前价 - 买入价) × 持仓量
```

**输出格式：**

```
## 持仓快照（2026-04-02）

| 股票 | 买入价 | 现价 | 持仓量 | 市值 | 盈亏 |
|------|--------|------|--------|------|------|
| AAPL | $170 | $185 | 10 | $1,850 | +$150 (+8.8%) |
| GOOGL | $140 | $145 | 5 | $725 | +$25 (+3.6%) |
| **合计** | | | | **$2,575** | **+$175 (+7.3%)** |
```

### 第三层：深度研究

启动 4 个 AI 分析师并行研究一只股票。

**交互流程：**
1. 用户："帮我深度分析一下 NVDA"
2. AI 启动 4 个分析视角并行研究
3. 汇总生成报告

**4 个分析师角色：**

| 角色 | 关注点 | 分析内容 |
|------|--------|---------|
| 基本面分析师 | 收入、利润、增长趋势 | 营收增速、利润率、自由现金流 |
| 护城河分析师 | 竞争优势 | 技术壁垒、品牌、网络效应、转换成本 |
| 估值分析师 | 估值合理性 | P/E、P/B、DCF、同行业对比 |
| 风险分析师 | 潜在风险 | 行业风险、政策风险、客户集中度 |

**生成深度研究报告：**

```bash
# 创建报告文档
wecom-cli doc create_doc '{"doc_type": 3, "doc_name": "深度研究：NVDA 2026-04-02"}'

# 写入报告
wecom-cli doc edit_doc_content '{"docid": "<报告DOCID>", "content": "# NVIDIA (NVDA) 深度研究报告\n\n## 总览\n| 维度 | 评分(1-10) | 摘要 |\n|------|-----------|------|\n| 基本面 | 9 | 营收同比增长 122% |\n| 护城河 | 9 | CUDA 生态壁垒极高 |\n| 估值 | 6 | P/E 65x 高于历史均值 |\n| 风险 | 5 | 依赖 AI 投资周期 |\n\n## 看多理由\n1. ...\n\n## 看空理由\n1. ...\n\n## 信心评分：7.5/10\n## 建议：当前持仓可继续持有，但不宜追高加仓", "content_type": 1}'

# 保存分析记录
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<分析记录SHEET_ID>", "records": [{"values": {
  "股票代码": [{"type": "text", "text": "NVDA"}],
  "分析日期": [{"type": "text", "text": "2026-04-02"}],
  "分析类型": [{"text": "深度研究"}],
  "结论摘要": [{"type": "text", "text": "基本面强、护城河深、估值偏高、风险可控"}],
  "看多理由": [{"type": "text", "text": "1. 数据中心收入持续爆发 2. CUDA生态壁垒 3. Blackwell架构量产"}],
  "看空理由": [{"type": "text", "text": "1. 估值充分反映增长 2. 大客户自研芯片 3. 出口管制风险"}],
  "信心评分": [{"type": "text", "text": "7.5"}],
  "建议": [{"type": "text", "text": "继续持有，不宜追高，等待回调至$120以下"}],
  "报告链接": [{"type": "text", "text": "https://doc.weixin.qq.com/docx/<DOCID>"}]
}}]}'
```

### 第四层：自动选股

按用户设定的条件筛选股票。

**交互流程：**
1. 用户："帮我找 ROE 大于 15%、PE 小于 20 的公司"
2. AI 使用 WebSearch 搜索符合条件的股票
3. 生成筛选结果列表

**输出格式：**

```
| 排名 | 股票 | PE | ROE | 行业 |
|------|------|-----|-----|------|
| 1 | A公司 | 15.2 | 22% | 金融 |
| 2 | B公司 | 18.5 | 18% | 制造 |
```

### 财报提醒

**添加财报日程：**

```bash
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<财报日程SHEET_ID>", "records": [{"values": {
  "股票代码": [{"type": "text", "text": "AAPL"}],
  "股票名称": [{"type": "text", "text": "Apple"}],
  "财报日期": [{"type": "text", "text": "2026-04-25"}],
  "状态": [{"text": "待发布"}]
}}]}'
```

---

## 字段类型写入格式（通用规则）

> 完整字段类型格式说明参见 [crm-schema](../wecomcli-crm/references/crm-schema.md)

| 字段类型 | 写入格式 | 示例 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 股票代码、价格、日期等（数字以文本存储） |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 市场、方向、分析类型、状态等 |

---

## 实操注意事项

### 常见踩坑

1. **更新时不传 key_type** — 必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **API 调用失败时** — 检查错误码：301085 表示 docid 无效（改用 url）；2022004 表示字段未找到（先 add_fields）；2022003 表示记录不存在（先 add_records）。遇到错误先用 `smartsheet_get_sheet '{"url": "<URL>"}'` 验证表结构。
3. **股价数据** — 通过 WebSearch 获取，可能存在延迟，建议标注数据时间
4. **数字计算** — 所有数值以 TEXT 存储，AI 读取后自行转换为数字计算
5. **报告文档** — 深度研究报告建议保存为独立文档，便于回顾
6. **SINGLE_SELECT 创建时 options 不生效** — `smartsheet_add_fields` 传入的 `property.options` 不会预创建选项，选项在首次写入记录时通过 `[{"text": "选项"}]` 自动创建。创建字段时可省略 `property` 参数。

### 定时自动化建议

| 任务 | 建议时间 | Cron |
|------|---------|------|
| 持仓快照 | 工作日 18:00 | `0 18 * * 1-5` |
| 目标价检查 | 工作日 9:30, 15:30 | `30 9,15 * * 1-5` |
| 财报检查 | 每周一 9:00 | `0 9 * * 1` |

### 数据联动

- 本 Skill 数据会被以下 Skill 读取：
  - `wecomcli-workflow-business-advisor` — 财务顾问分析投资表现
