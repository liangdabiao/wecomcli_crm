# CRM 智能表格 Schema 参考

本文档说明 CRM 表的字段类型格式规则和 webhook 推送格式。具体的 field_id、option_id 等表结构信息在首次配置时动态发现，存储在 memory 的 `crm-config` 中。

## 字段类型 → 写入格式映射

通过 CLI (`smartsheet_add_records` / `smartsheet_update_records`) 写入时，使用**字段标题**作为 key：

### TEXT 字段

```json
"字段标题": [{"type": "text", "text": "值"}]
```

适用于：公司名称、对接人、备注等纯文本字段。

### SINGLE_SELECT 字段

```json
"字段标题": [{"text": "选项文本"}]
```

- 传选项的文本值即可，**不需要 option_id**
- 文本必须与表中已有选项精确匹配
- 适用场景：状态、行业、地区等分类字段

### PHONE_NUMBER 字段

```json
"字段标题": "13800138000"
```

- 直接传字符串，不要用数组包裹
- 不要用 TEXT 格式 `[{"type":"text","text":"..."}]`

### USER 字段

```json
"字段标题": [{"userid": "用户ID"}]
```

- userid 通过 `wecom-cli contact get_userlist '{}'` 获取
- 支持多选：`[{"userid": "ID1"}, {"userid": "ID2"}]`

### 写入格式速查表

| 字段类型 | CLI 格式 | 要点 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 必须带 type 和 text |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 传文本，不需要 id |
| PHONE_NUMBER | 直接传字符串 | 不是 TEXT，不要用数组 |
| USER | `[{"userid":"用户ID"}]` | userid 从通讯录查 |

## 更新操作 key_type

更新记录时**必须传** `key_type`，否则字段标题无法识别：

```json
{
  "key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE",
  "records": [{"record_id": "xxx", "values": {"字段标题": <值>}}]
}
```

## Webhook 推送格式

外部系统通过 webhook 推送数据时，使用 **field_id** 作为 key（field_id 在首次配置时从 `smartsheet_get_fields` 获取）：

```json
{
  "schema": {
    "<FIELD_ID_1>": "字段标题1",
    "<FIELD_ID_2>": "字段标题2"
  },
  "add_records": [
    {
      "values": {
        "<TEXT_FIELD_ID>": "文本值",
        "<SELECT_FIELD_ID>": [{"text": "选项文本"}],
        "<USER_FIELD_ID>": [{"user_id": "用户ID"}]
      }
    }
  ]
}
```

### CLI 与 Webhook 格式差异

| 差异点 | CLI | Webhook |
|--------|-----|---------|
| Key | 字段标题（如 `公司名称`） | field_id（如 `fMo0cT`） |
| TEXT 字段 | `[{"type":"text","text":"值"}]` | 直接传字符串 `"值"` |
| SINGLE_SELECT | `[{"text":"值"}]` | `[{"text":"值"}]`（相同） |
| USER 字段 key | `userid` | `user_id` |
