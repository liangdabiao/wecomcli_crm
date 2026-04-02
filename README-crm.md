# wecomcli-crm

基于 [wecom-cli](https://github.com/WecomTeam/wecom-cli) + [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 的企业微信 CRM 客户管理系统。

通过自然语言驱动企业微信智能表格，完成客户的新增、查询、更新、删除全流程操作。

## 快速开始

### 1. 安装依赖

```bash
npm install -g @wecom/cli
npx skills add WeComTeam/wecom-cli -y -g
```

### 2. 配置凭证

```bash
wecom-cli init    # 交互式输入企业微信机器人 Bot ID 和 Secret
```

### 3. 首次使用 CRM

在 Claude Code 中说：

> "帮我新增一个客户"

AI 会自动引导你完成 CRM 表配置（提供智能表格链接 → 自动发现字段结构 → 保存配置），之后即可自由操作。

## 使用示例

```
"帮我新增一个客户：深圳XX科技有限公司，对接人李明，华南区，IT行业"
"查一下华南区有哪些客户"
"把XX公司的状态改成合作中"
"删除XX公司的记录"
```

## 项目结构

```
D:\qyweixin\
├── .claude/
│   ├── settings.local.json          # wecom-cli 自动执行权限
│   └── skills/
│       └── wecomcli-crm/            # CRM Skill 核心
│           ├── SKILL.md             # Skill 指令（首次引导 + CRUD 流程）
│           └── references/
│               └── crm-schema.md    # 字段类型格式参考
├── README.md                        # 本文件
└── wecom-cli.md                     # wecom-cli 完整命令参考
```

## CRM Skill 设计

`wecomcli-crm` 是一个自定义的 Claude Code Skill，核心特点：

- **动态配置** — 不硬编码表格 URL，首次使用时自动发现并保存到 memory
- **字段类型感知** — 根据字段类型（TEXT / SINGLE_SELECT / PHONE_NUMBER / USER）自动选择正确写入格式
- **全 CRUD** — 查询、新增、更新、删除，一条自然语言搞定

### 操作命令

| 操作 | CLI 命令 | 要点 |
|------|----------|------|
| 查询 | `wecom-cli doc smartsheet_get_records` | 返回记录及 record_id |
| 新增 | `wecom-cli doc smartsheet_add_records` | 批量最多 500 条 |
| 更新 | `wecom-cli doc smartsheet_update_records` | **必须带 `key_type`** |
| 删除 | `wecom-cli doc smartsheet_delete_records` | 不可逆，需先查询获取 record_id |

### 字段类型写入格式

| 字段类型 | CLI 写入格式 | 注意 |
|----------|-------------|------|
| TEXT | `[{"type":"text","text":"值"}]` | — |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 文本必须精确匹配 |
| PHONE_NUMBER | 直接传字符串 `"138xxxx"` | 不是 TEXT，不要用数组 |
| USER | `[{"userid":"用户ID"}]` | userid 通过通讯录查询 |

### 常见踩坑

1. **更新时不传 key_type** — 必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **PHONE_NUMBER 用了 TEXT 格式** — 直接传字符串，不要用数组包裹
3. **删除/更新前未查询** — record_id 只能从查询结果获取
4. **USER 字段 key 名** — CLI 中是 `userid`，webhook 中是 `user_id`

## 更多 wecom-cli 命令

完整的品类命令参考见 [wecom-cli.md](wecom-cli.md)，涵盖通讯录、待办、会议、消息、日程、文档、智能表格等 7 大品类。
