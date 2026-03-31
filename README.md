# 企业微信 Agent 项目

基于 [wecom-cli](https://github.com/WecomTeam/wecom-cli) 搭建的企业微信 Agent 工作台。通过 Claude Code + Skills 实现 AI 驱动的 CRM 客户管理、消息处理、会议日程等企业微信操作。

## 项目结构

```
D:\qyweixin\
├── .claude/                              # Claude Code 配置
│   ├── settings.local.json               # 权限配置（wecom-cli 自动执行）
│   └── skills/
│       ├── skill-creator/                # Skill 创建工具
│       └── wecomcli-crm/                 # CRM 客户管理 Skill ⭐
│           ├── SKILL.md                  # 核心指令文件（含首次引导流程）
│           └── references/
│               └── crm-schema.md         # 字段类型格式参考（通用）
├── README.md                             # 本文件
├── 接收数据到CRM-客户管理总表.txt           # CRM 表 webhook 配置
├── 接收数据到日报.txt                      # 日报表 webhook 配置
└── 接收数据到図客户线索(示例).txt            # 客户线索 webhook 示例
```

## 环境要求

- **Node.js**（npm / npx）
- **Claude Code** CLI（[安装指南](https://docs.anthropic.com/en/docs/claude-code)）
- **wecom-cli** — 企业微信命令行工具
- **企业微信机器人凭证** — Bot ID 和 Secret

## 安装与配置

### 1. 安装 wecom-cli 和 Skills

```bash
# 安装 CLI
npm install -g @wecom/cli

# 安装 wecom-cli 的 12 个 Agent Skills
npx skills add WeComTeam/wecom-cli -y -g
```

### 2. 配置企业微信凭证

```bash
# 交互式配置，仅需执行一次
wecom-cli init
```

按提示输入企业微信机器人的 Bot ID 和 Secret，凭证加密存储在 `~/.config/wecom/bot.enc`。

### 3. 验证安装

```bash
wecom-cli --version
wecom-cli contact get_userlist '{}'
```

## 已安装的 Agent Skills

### wecom-cli 官方 Skills（12 个）

| Skill | 品类 | 说明 |
|-------|------|------|
| `wecomcli-lookup-contact` | 通讯录 | 按姓名/别名搜索成员 |
| `wecomcli-get-todo-list` | 待办 | 待办列表查询 |
| `wecomcli-get-todo-detail` | 待办 | 待办详情批量查询 |
| `wecomcli-edit-todo` | 待办 | 待办创建/更新/删除 |
| `wecomcli-create-meeting` | 会议 | 创建预约会议 |
| `wecomcli-edit-meeting` | 会议 | 取消会议/更新成员 |
| `wecomcli-get-meeting` | 会议 | 查询会议列表和详情 |
| `wecomcli-get-msg` | 消息 | 会话列表/消息记录/发送 |
| `wecomcli-manage-schedule` | 日程 | 日程 CRUD/闲忙查询 |
| `wecomcli-manage-doc` | 文档 | 文档创建/读取/编辑 |
| `wecomcli-manage-smartsheet-schema` | 智能表格 | 子表和字段管理 |
| `wecomcli-manage-smartsheet-data` | 智能表格 | 记录增删改查 |

### 项目自定义 Skill

| Skill | 说明 |
|-------|------|
| `wecomcli-crm` | CRM 客户管理 — 首次引导配置 + 客户新增/查询/更新/删除 |

## CRM 客户管理

### 首次配置流程

CRM Skill 采用**动态配置**设计，不硬编码任何表格 URL。首次使用时自动引导：

```
用户: "帮我新增一个客户"
AI:  检查 memory 中无 CRM 配置
AI:  "请提供您的 CRM 客户管理智能表格链接（在企业微信中打开表格 → 分享 → 复制链接）"
用户: 提供链接
AI:  自动调用 API 发现子表和字段结构 → 保存到 memory
AI:  "CRM 表配置完成！共发现 N 个字段。现在可以操作客户数据了。"
```

后续使用直接从 memory 读取配置，无需重复配置。

### 使用方式

在 Claude Code 中直接用自然语言操作：

```
# 新增客户
"帮我新增一个客户：深圳XX科技有限公司，对接人李明，华南区，IT行业，状态已建联"

# 查询客户
"查一下华南区有哪些客户"
"看看状态是合作中的客户有哪些"

# 更新客户
"把XX公司的状态改成合作中"
"给XX公司分配销售员"

# 删除客户
"删除XX公司的记录"
```

### CLI 直接操作

也可以直接用命令行操作（将 `<URL>` 替换为实际的智能表格链接）：

```bash
# 查询所有客户
wecom-cli doc smartsheet_get_records '{"url": "<URL>", "sheet_id": "<SHEET_ID>"}'

# 查看表结构
wecom-cli doc smartsheet_get_fields '{"url": "<URL>", "sheet_id": "<SHEET_ID>"}'

# 查询子表列表
wecom-cli doc smartsheet_get_sheet '{"url": "<URL>"}'

# 新增客户
wecom-cli doc smartsheet_add_records '{"url": "<URL>", "sheet_id": "<SHEET_ID>", "records": [{"values": {"公司名称": [{"type": "text", "text": "XX公司"}], "联系电话": "13800138000", "状态": [{"text": "已建联"}]}}]}'

# 更新客户（必须带 key_type）
wecom-cli doc smartsheet_update_records '{"url": "<URL>", "sheet_id": "<SHEET_ID>", "key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE", "records": [{"record_id": "RECORD_ID", "values": {"状态": [{"text": "合作中"}]}}]}'

# 删除客户
wecom-cli doc smartsheet_delete_records '{"url": "<URL>", "sheet_id": "<SHEET_ID>", "record_ids": ["RECORD_ID"]}'
```

### 字段类型写入格式

| 字段类型 | CLI 格式 | 要点 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 必须带 type 和 text |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 传文本，不需要 id |
| PHONE_NUMBER | 直接传字符串 `"138xxxx"` | 不是 TEXT，不要用数组包裹 |
| USER | `[{"userid":"用户ID"}]` | userid 通过 `wecom-cli contact get_userlist` 查 |

### 常见踩坑

1. **更新时不传 key_type** — 必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **PHONE_NUMBER 用了 TEXT 格式** — 联系电话直接传字符串，不要用 `[{"type":"text","text":"..."}]`
3. **删除/更新前未查询** — record_id 只能从查询结果获取
4. **USER 字段 key 名** — CLI 中是 `userid`，webhook 中是 `user_id`

## Webhook 数据推送

智能表格支持通过 webhook 从外部系统写入数据。项目中有以下 webhook 配置文件：

| 文件 | 目标表 | 用途 |
|------|--------|------|
| `接收数据到CRM-客户管理总表.txt` | CRM-客户管理总表 | 客户数据入库 |
| `接收数据到日报.txt` | 日报 | 工作日报收集 |
| `接收数据到図客户线索(示例).txt` | 客户线索 | 线索收集（示例） |

### CLI 与 Webhook 格式差异

| 差异点 | CLI | Webhook |
|--------|-----|---------|
| Key | 字段标题（如 `公司名称`） | field_id（如 `fMo0cT`） |
| TEXT 字段 | `[{"type":"text","text":"值"}]` | 直接传字符串 `"值"` |
| USER 字段 | `{"userid": "ID"}` | `{"user_id": "ID"}` |
| SINGLE_SELECT | `[{"text":"值"}]` | 相同 |

### Webhook 推送示例

```bash
curl -X POST "<webhook_url>" \
  -H "Content-Type: application/json" \
  -d '{
    "schema": {
      "fMo0cT": "公司名称",
      "fzSueb": "公司对接人",
      "fSNPFZ": "联系电话"
    },
    "add_records": [
      {
        "values": {
          "fMo0cT": "XX科技有限公司",
          "fzSueb": "张三",
          "fSNPFZ": "13800138000"
        }
      }
    ]
  }'
```

> webhook 中的 field_id 对应关系参见各 `接收数据到*.txt` 配置文件。

## 品类命令参考

```bash
# 通讯录
wecom-cli contact get_userlist '{}'

# 待办
wecom-cli todo get_todo_list '{}'
wecom-cli todo create_todo '{"content": "完成Q2规划", "remind_time": "2026-06-01 09:00:00"}'

# 会议
wecom-cli meeting list_user_meetings '{"begin_datetime": "2026-03-31 00:00", "end_datetime": "2026-04-06 23:59", "limit": 100}'
wecom-cli meeting create_meeting '{"title": "周会", "meeting_start_datetime": "2026-04-01 10:00", "meeting_duration": 3600, "invitees": []}'

# 消息
wecom-cli msg get_msg_chat_list '{"begin_time": "2026-03-25 00:00:00", "end_time": "2026-03-31 23:59:59"}'
wecom-cli msg send_message '{"chat_type": 2, "chatid": "群ID", "msgtype": "text", "text": {"content": "消息内容"}}'

# 日程
wecom-cli schedule get_schedule_list_by_range '{"start_time": "2026-03-31 00:00:00", "end_time": "2026-04-06 23:59:59"}'
wecom-cli schedule check_availability '{"check_user_list": ["USERID"], "start_time": "2026-04-01 09:00:00", "end_time": "2026-04-01 18:00:00"}'

# 文档
wecom-cli doc create_doc '{"doc_type": 3, "doc_name": "文档名"}'
wecom-cli doc get_doc_content '{"docid": "DOC_ID", "type": 2}'
```

## 扩展：创建新的 Skill

使用项目内置的 `skill-creator` 创建新 skill：

```bash
# 初始化新 skill
python .claude/skills/skill-creator/scripts/init_skill.py <skill-name> --path .claude/skills

# 编辑 SKILL.md 和 references/ 后打包
python .claude/skills/skill-creator/scripts/package_skill.py .claude/skills/<skill-name>
```

参考 `wecomcli-crm` 的结构：SKILL.md 含首次引导流程 + references/ 存放通用格式参考 + 实际配置存 memory。
