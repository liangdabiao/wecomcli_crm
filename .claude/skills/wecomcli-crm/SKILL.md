---
name: wecomcli-crm
description: 企业微信 CRM 客户管理技能。通过 wecom-cli 对企业微信智能表格进行客户的新增、查询、更新、删除操作。当用户说"新增客户"、"添加客户"、"录入客户"、"查询客户"、"客户列表"、"更新客户信息"、"修改客户状态"、"删除客户"、"CRM"等涉及客户管理的场景时触发。
---

# 企业微信 CRM 客户管理

通过 wecom-cli 操作企业微信智能表格，完成客户数据管理。

## 首次配置（必须）

**执行任何 CRM 操作前，先检查 memory 中是否已有 CRM 配置。** 如果没有，按以下流程引导用户：

### 步骤 1：获取 CRM 表 URL

向用户询问：
> "请提供您的 CRM 客户管理智能表格的链接（在企业微信中打开表格 → 右上角分享 → 复制链接）"

### 步骤 2：发现表结构

用用户提供的 URL 自动发现子表和字段：

```bash
# 获取子表列表
wecom-cli doc smartsheet_get_sheet '{"url": "<用户提供的URL>"}'

# 获取第一个子表的字段定义
wecom-cli doc smartsheet_get_fields '{"url": "<用户提供的URL>", "sheet_id": "<从上一步获取>"}'
```

### 步骤 3：保存配置到 memory

将以下信息保存到 memory 文件 `crm-config.md`：

```markdown
---
name: crm-config
description: CRM 智能表格配置信息，包含 URL、sheet_id 和字段 schema
type: project
---

## CRM 表配置

- URL: <用户提供的URL>
- sheet_id: <从 API 获取>
- 子表名称: <从 API 获取>

## 字段 Schema
<从 smartsheet_get_fields 返回结果中记录每个字段的 field_title、field_type、选项列表>
```

### 步骤 4：后续操作

配置保存后，所有 CRM 操作直接从 memory 读取 URL 和 sheet_id，无需再次询问。

---

## 操作流程

> 以下命令中 `<URL>` 和 `<SHEET_ID>` 从 memory 的 crm-config 中读取。

### 1. 查询客户

```bash
wecom-cli doc smartsheet_get_records '{"url": "<URL>", "sheet_id": "<SHEET_ID>"}'
```

返回所有记录及 `record_id`。**更新和删除前必须先查询获取 record_id。**

### 2. 新增客户

```bash
wecom-cli doc smartsheet_add_records '{"url": "<URL>", "sheet_id": "<SHEET_ID>", "records": [{"values": {<字段值>}}]}'
```

字段值格式取决于字段类型，详见下方"字段类型写入格式"。

支持批量，`records` 数组一次最多 500 条。

### 3. 更新客户

**必须传 `key_type`**，否则字段标题无法识别：

```bash
wecom-cli doc smartsheet_update_records '{"url": "<URL>", "sheet_id": "<SHEET_ID>", "key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE", "records": [{"record_id": "RECORD_ID", "values": {<需更新的字段>}}]}'
```

- `key_type` 固定为 `"CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
- `values` 中只传需要修改的字段
- 创建时间、编辑时间、创建人、编辑人字段不可更新

### 4. 删除客户

```bash
wecom-cli doc smartsheet_delete_records '{"url": "<URL>", "sheet_id": "<SHEET_ID>", "record_ids": ["RECORD_ID"]}'
```

**删除不可逆**，确认 record_id 后再调用。支持批量，`record_ids` 数组一次最多 500 条。

---

## 字段类型写入格式（通用规则）

无论用户表格的具体字段名是什么，写入格式由字段类型决定：

| 字段类型 | 写入格式 | 示例 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 公司名称、备注等文本字段 |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 状态、行业等单选字段 |
| PHONE_NUMBER | 直接传字符串 | `"13800138000"` |
| USER | `[{"userid":"用户ID"}]` | 销售、交付等人员字段 |

详细说明和 webhook 格式见 [crm-schema.md](references/crm-schema.md)。

---

## 实操注意事项

### 常见踩坑

1. **更新时不传 key_type** — 字段标题无法识别，必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **PHONE_NUMBER 用了 TEXT 格式** — 联系电话不是文本字段，直接传字符串，不要用 `[{"type":"text","text":"..."}]`
3. **删除/更新前未查询** — record_id 只能从查询结果获取，新增时也会返回 record_id
4. **USER 字段 key 名** — CLI 中是 `userid`，webhook 中是 `user_id`，两者不同
5. **选项文本必须精确匹配** — SINGLE_SELECT 传的 text 必须与表中已有的选项文本完全一致

### 辅助命令

```bash
# 重新查表结构（字段变更时用）
wecom-cli doc smartsheet_get_fields '{"url": "<URL>", "sheet_id": "<SHEET_ID>"}'

# 查通讯录（获取 userid 用于 USER 字段）
wecom-cli contact get_userlist '{}'
```
