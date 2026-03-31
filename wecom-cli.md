# wecom-cli

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Rust](https://img.shields.io/badge/rust-%3E%3D1.75-orange.svg)](https://www.rust-lang.org/)

> 💬 扫码加入企业微信交流群：
>
> <img src="https://wwcdn.weixin.qq.com/node/wework/images/202603241759.3fb01c32cc.png" alt="扫码入群交流" width="200" />

企业微信命令行工具 — 让人类和 AI Agent 都能在终端中操作企业微信。覆盖通讯录、待办、会议、消息、日程、文档、智能表格等核心业务域，提供 7 大品类及 12 个 AI Agent [Skills](https://github.com/WecomTeam/wecom-cli/tree/main/skills)。

[安装](#安装与快速开始) · [AI Agent Skills](#agent-skills) · [命令](#命令参考) · [品类一览](#品类与能力一览)

## 为什么选 wecom-cli？

- **为 AI Agent 所设计** — 开箱即用的 [Skills](https://github.com/WecomTeam/wecom-cli/tree/main/skills)， 适配主流 AI 工具，Agent 可直接操作企业微信，无需额外适配
- **覆盖用户核心需求** — 7 大业务品类、12 个 AI Agent [Skills](https://github.com/WecomTeam/wecom-cli/tree/main/skills)，覆盖通讯录、待办、会议、消息、日程、文档与智能表格
- **快速上手** — `init` 配置凭证，直接调用品类工具，从安装到第一次 API 调用只需两步

## 功能

| 类别         | 能力                                                                          |
| ------------ | ----------------------------------------------------------------------------- |
| 👤 通讯录   | 获取可见范围成员列表、按姓名/别名搜索                                        |
| ✅ 待办     | 创建、查询列表、查询详情、更新、删除待办，变更用户处理状态                    |
| 🎥 会议     | 创建预约会议、取消会议、更新受邀成员、查询会议列表、获取会议详情              |
| 💬 消息     | 会话列表查询、消息记录拉取（文本/图片/文件/语音/视频）、多媒体下载、发送文本  |
| 📅 日程     | 日程 CRUD、参与人管理、多成员闲忙查询                                         |
| 📄 文档     | 文档创建/读取/编辑                                                            |
| 📊 智能表格   | 智能表格创建、子表与字段管理、表格记录增删改查                                  |

## 安装与快速开始

### 环境要求

- Node.js（`npm`/`npx`）
- 企业微信机器人的 Bot ID 和 Secret

### 安装

```bash
# 安装 CLI
npm install -g @wecom/cli

# 安装 CLI SKILL（必需）
npx skills add WeComTeam/wecom-cli -y -g
```

### 快速开始

```bash
# 1. 配置企业微信机器人凭证（交互式，仅需一次）
wecom-cli init

# 2. 调用工具
wecom-cli contact get_userlist '{}'
```

## Agent Skills

| Skill | 品类 | 说明 |
| ----- | ---- | ---- |
| `wecomcli-lookup-contact` | contact | 通讯录成员查询，按姓名/别名搜索 |
| `wecomcli-get-todo-list` | todo | 待办列表查询，按时间过滤和分页 |
| `wecomcli-get-todo-detail` | todo | 待办详情批量查询 |
| `wecomcli-edit-todo` | todo | 待办创建、更新、删除、状态变更 |
| `wecomcli-create-meeting` | meeting | 创建预约会议 |
| `wecomcli-edit-meeting` | meeting | 取消会议、更新受邀成员 |
| `wecomcli-get-meeting` | meeting | 查询会议列表和详情 |
| `wecomcli-get-msg` | msg | 会话列表、消息记录、媒体下载、文本发送 |
| `wecomcli-manage-schedule` | schedule | 日程 CRUD、参与人管理、闲忙查询 |
| `wecomcli-manage-doc` | doc | 文档创建/读取/编辑 |
| `wecomcli-manage-smartsheet-schema` | smartsheet | 智能表格子表与字段管理 |
| `wecomcli-manage-smartsheet-data` | smartsheet | 智能表格记录增删改查 |

## 命令参考

### `--help`

列出所有支持的命令和品类。

```bash
wecom-cli --help
```

输出示例：

```
Usage: wecom-cli <COMMAND>

Commands:
  init      Documentation for init
  contact   通讯录 — 成员查询和搜索
  doc       文档 — 文档/智能表格创建和管理
  meeting   会议 — 创建/管理/查询视频会议
  msg       消息 — 聊天列表、发送/接收消息、媒体下载
  schedule  日程 — 日程增删改查和可用性查询
  todo      待办事项 — 创建/查询/编辑待办项

Options:
  -h, --help     Print help
  -V, --version  Print version
```

### `init`

交互式配置企业微信机器人凭证，加密存储到本地。仅需执行一次。

```bash
wecom-cli init
```

凭证存储位置：`~/.config/wecom/bot.enc`

### 品类调用

每个品类作为独立子命令使用。不传方法名时列出该品类下所有可用工具，传方法名时调用指定工具。

```bash
# 列出品类下的所有工具
wecom-cli <category>

# 调用品类下的指定工具
wecom-cli <category> <method> [json_args]
```

示例：

```bash
# 列出通讯录品类下的工具
wecom-cli contact

# 列出待办品类下的工具
wecom-cli todo

# 调用工具（传 JSON 参数）
wecom-cli contact get_userlist '{}'

# 调用工具（无参数）
wecom-cli contact get_userlist
```

## 品类与能力一览

### contact — 通讯录

| 工具 | 说明 |
|------|------|
| `get_userlist` | 获取当前用户可见范围内的通讯录成员（userid、姓名、别名） |

```bash
# 获取全量通讯录成员
wecom-cli contact get_userlist '{}'
```

### todo — 待办

| 工具 | 说明 |
|------|------|
| `get_todo_list` | 查询待办列表，支持按时间过滤和分页 |
| `get_todo_detail` | 根据待办 ID 批量查询完整详情 |
| `create_todo` | 创建待办，可指定内容、分派人、提醒时间 |
| `update_todo` | 更新待办内容、状态、分派人或提醒时间 |
| `delete_todo` | 删除待办（不可撤销） |
| `change_todo_user_status` | 变更当前用户在待办中的状态 |

```bash
# 查询待办列表
wecom-cli todo get_todo_list '{}'

# 创建待办
wecom-cli todo create_todo '{"content": "完成Q2规划文档", "remind_time": "2026-06-01 09:00:00"}'

# 批量查询待办详情
wecom-cli todo get_todo_detail '{"todo_id_list": ["TODO_ID_1", "TODO_ID_2"]}'

# 标记待办完成
wecom-cli todo update_todo '{"todo_id": "TODO_ID", "todo_status": 0}'

# 删除待办
wecom-cli todo delete_todo '{"todo_id": "TODO_ID"}'
```

### meeting — 会议

| 工具 | 说明 |
|------|------|
| `create_meeting` | 创建预约会议，支持设置参数、邀请参与人、安全设置 |
| `cancel_meeting` | 取消指定的预约会议 |
| `set_invite_meeting_members` | 更新会议受邀成员（全量覆盖） |
| `list_user_meetings` | 查询用户在时间范围内的会议列表（当日前后 30 天） |
| `get_meeting_info` | 获取会议完整详情 |

```bash
# 查询本周会议
wecom-cli meeting list_user_meetings '{"begin_datetime": "2026-03-23 00:00", "end_datetime": "2026-03-29 23:59", "limit": 100}'

# 创建会议
wecom-cli meeting create_meeting '{"title": "技术方案评审", "meeting_start_datetime": "2026-03-30 15:00", "meeting_duration": 3600, "invitees": {"userid": ["zhangsan", "lisi"]}}'

# 获取会议详情
wecom-cli meeting get_meeting_info '{"meetingid": "MEETING_ID"}'

# 取消会议
wecom-cli meeting cancel_meeting '{"meetingid": "MEETING_ID"}'
```

### msg — 消息

| 工具 | 说明 |
|------|------|
| `get_msg_chat_list` | 按时间范围查询有消息的会话列表 |
| `get_message` | 拉取会话消息记录（支持文本/图片/文件/语音/视频） |
| `get_msg_media` | 下载消息中的多媒体文件到本地 |
| `send_message` | 向单聊或群聊发送文本消息 |

```bash
# 获取最近一周会话列表
wecom-cli msg get_msg_chat_list '{"begin_time": "2026-03-22 00:00:00", "end_time": "2026-03-29 23:59:59"}'

# 拉取聊天记录
wecom-cli msg get_message '{"chat_type": 1, "chatid": "zhangsan", "begin_time": "2026-03-29 09:00:00", "end_time": "2026-03-29 18:00:00"}'

# 发送文本消息
wecom-cli msg send_message '{"chat_type": 1, "chatid": "zhangsan", "msgtype": "text", "text": {"content": "hello"}}'

# 下载多媒体文件
wecom-cli msg get_msg_media '{"media_id": "MEDIA_ID"}'
```

### schedule — 日程

| 工具 | 说明 |
|------|------|
| `get_schedule_list_by_range` | 查询时间范围内的日程 ID 列表（当日前后 30 天） |
| `get_schedule_detail` | 批量获取日程详情（1~50 个） |
| `create_schedule` | 创建日程，支持设置提醒、参与人 |
| `update_schedule` | 修改日程（只传需修改的字段） |
| `cancel_schedule` | 取消日程 |
| `add_schedule_attendees` | 添加日程参与人 |
| `del_schedule_attendees` | 移除日程参与人 |
| `check_availability` | 查询多成员闲忙状态（1~10 人） |

```bash
# 查询今天的日程
wecom-cli schedule get_schedule_list_by_range '{"start_time": "2026-03-29 00:00:00", "end_time": "2026-03-29 23:59:59"}'

# 获取日程详情
wecom-cli schedule get_schedule_detail '{"schedule_id_list": ["SCHEDULE_ID"]}'

# 创建日程
wecom-cli schedule create_schedule '{"schedule": {"start_time": "2026-03-30 14:00:00", "end_time": "2026-03-30 15:00:00", "summary": "需求评审", "attendees": [{"userid": "zhangsan"}], "reminders": {"is_remind": 1, "remind_before_event_secs": 900, "timezone": 8}}}'

# 查询闲忙
wecom-cli schedule check_availability '{"check_user_list": ["zhangsan", "lisi"], "start_time": "2026-03-30 09:00:00", "end_time": "2026-03-30 18:00:00"}'
```

### doc — 文档

| 工具 | 说明 |
|------|------|
| `create_doc` | 创建文档（doc_type=3） |
| `get_doc_content` | 获取文档内容（Markdown 格式，异步轮询） |
| `edit_doc_content` | 用 Markdown 覆写文档正文 |

```bash
# 创建文档
wecom-cli doc create_doc '{"doc_type": 3, "doc_name": "项目周报"}'

# 读取文档内容（首次调用）
wecom-cli doc get_doc_content '{"docid": "DOC_ID", "type": 2}'

# 读取文档内容（轮询，携带 task_id）
wecom-cli doc get_doc_content '{"docid": "DOC_ID", "type": 2, "task_id": "TASK_ID"}'

# 编辑文档
wecom-cli doc edit_doc_content '{"docid": "DOC_ID", "content": "# 标题\n\n正文内容", "content_type": 1}'
```

### doc — 智能表格

| 工具 | 说明 |
|------|------|
| `create_doc` | 创建智能表格（通过 doc create_doc，doc_type=10） |
| `smartsheet_get_sheet` | 查询智能表格的所有子表 |
| `smartsheet_add_sheet` | 添加子表 |
| `smartsheet_update_sheet` | 修改子表标题 |
| `smartsheet_delete_sheet` | 删除子表（不可逆） |
| `smartsheet_get_fields` | 查询子表的字段/列信息 |
| `smartsheet_add_fields` | 添加字段/列 |
| `smartsheet_update_fields` | 更新字段标题 |
| `smartsheet_delete_fields` | 删除字段/列（不可逆） |
| `smartsheet_get_records` | 查询子表全部记录 |
| `smartsheet_add_records` | 添加记录 |
| `smartsheet_update_records` | 更新记录 |
| `smartsheet_delete_records` | 删除记录（不可逆） |

```bash
# 创建智能表格
wecom-cli doc create_doc '{"doc_type": 10, "doc_name": "任务跟踪表"}'

# 查询智能表格子表
wecom-cli doc smartsheet_get_sheet '{"docid": "DOC_ID"}'

# 查询子表字段信息
wecom-cli doc smartsheet_get_fields '{"docid": "DOC_ID", "sheet_id": "SHEET_ID"}'

# 添加子表字段
wecom-cli doc smartsheet_add_fields '{"docid": "DOC_ID", "sheet_id": "SHEET_ID", "fields": [{"field_title": "状态", "field_type": "FIELD_TYPE_SINGLE_SELECT"}]}'

# 查询子表记录
wecom-cli doc smartsheet_get_records '{"docid": "DOC_ID", "sheet_id": "SHEET_ID"}'

# 添加记录
wecom-cli doc smartsheet_add_records '{"docid": "DOC_ID", "sheet_id": "SHEET_ID", "records": [{"values": {"标题": [{"type": "text", "text": "新任务"}]}}]}'

# 更新记录
wecom-cli doc smartsheet_update_records '{"docid": "DOC_ID", "sheet_id": "SHEET_ID", "key_type":"CELL_VALUE_KEY_TYPE_FIELD_TITLE", "records": [{"record_id": "RECORD_ID", "values": {"标题": [{"type": "text", "text": "已更新"}]}}]}'

# 删除记录
wecom-cli doc smartsheet_delete_records '{"docid": "DOC_ID", "sheet_id": "SHEET_ID", "record_ids": ["RECORD_ID"]}'
```

## 许可证

本项目基于 **MIT 许可证** 开源。
