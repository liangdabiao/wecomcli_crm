     企业微信 CLI 版 — 10 大工作流 Skill 实施计划

     Context

     用户已有基于 wecom-cli 的 CRM 系统（.claude/skills/wecomcli-crm/），现参照飞书版的 10 大工作流 Skill，在企业微信
     CLI 上实现相同的 10 个功能。每个 Skill 遵循现有 CRM skill 的 SKILL.md 格式，数据统一存储到智能表格，配置保存到
     memory。

     实施范围

     创建 10 个 skill 目录，每个包含 SKILL.md 和 references/ 目录：

     .claude/skills/
     ├── wecomcli-workflow-health-diary/SKILL.md
     ├── wecomcli-workflow-social-tracker/SKILL.md
     ├── wecomcli-workflow-meeting-todo/SKILL.md
     ├── wecomcli-workflow-personal-crm/SKILL.md
     ├── wecomcli-workflow-knowledge-base/SKILL.md
     ├── wecomcli-workflow-content-pipeline/SKILL.md
     ├── wecomcli-workflow-stock-analyzer/SKILL.md
     ├── wecomcli-workflow-morning-brief/SKILL.md
     ├── wecomcli-workflow-business-advisor/SKILL.md
     └── wecomcli-workflow-multi-agent-dev/SKILL.md

     每个 SKILL.md 统一结构

     参照现有 wecomcli-crm/SKILL.md 模式，每个 skill 包含：

     ---
     name: wecomcli-workflow-<name>
     description: <触发描述>
     ---

     # <标题>

     ## 首次配置（自动初始化）
     - 创建智能表格 (doc_type=10)
     - 创建子表和字段 (smartsheet_add_sheet + smartsheet_add_fields)
     - 保存配置到 memory

     ## 操作流程
     - 各工作模式的完整命令示例

     ## 字段类型写入格式
     - 引用 crm-schema.md 通用规则

     ## 实操注意事项

     字段类型规则（复用现有）

     来自 references/crm-schema.md：
     - TEXT: [{"type":"text","text":"值"}]
     - SINGLE_SELECT: [{"text":"选项文本"}]
     - PHONE_NUMBER: 直接传字符串
     - USER: [{"userid":"用户ID"}]
     - 更新必须带 key_type: "CELL_VALUE_KEY_TYPE_FIELD_TITLE"

     实施步骤（按 Phase 顺序）

     Phase 1（2个 Skill）

     1.1 wecomcli-workflow-health-diary
     - 文件: .claude/skills/wecomcli-workflow-health-diary/SKILL.md
     - 3张子表: 饮食记录、身体状态、关联分析
     - 命令: msg(接收食物描述) + smartsheet(记录) + doc(周报) + msg(提醒)
     - 5种模式: 文字记录、照片识别、关联分析、周报生成、主动提醒

     1.2 wecomcli-workflow-social-tracker
     - 文件: .claude/skills/wecomcli-workflow-social-tracker/SKILL.md
     - 3张子表: 社交账号、每日快照、内容表现
     - 命令: smartsheet(账号管理+快照存储) + msg(通知) + WebSearch(获取数据)
     - 4种模式: 账号管理、手动录入快照、趋势分析、定时采集

     Phase 2（3个 Skill）

     2.1 wecomcli-workflow-meeting-todo
     - 文件: .claude/skills/wecomcli-workflow-meeting-todo/SKILL.md
     - 1张子表: 会议待办追踪
     - 命令: meeting(获取会议) + todo(创建待办) + smartsheet(追踪) + msg(确认)
     - 4种模式: 批量提取、单会议提取、完成检查、自动归档

     2.2 wecomcli-workflow-personal-crm
     - 文件: .claude/skills/wecomcli-workflow-personal-crm/SKILL.md
     - 2张子表: 联系人主表、互动记录
     - 命令: schedule(日程参与者) + contact(通讯录) + msg(互动提取) + smartsheet(存储)
     - 无邮件 → 用消息记录替代

     2.3 wecomcli-workflow-knowledge-base
     - 文件: .claude/skills/wecomcli-workflow-knowledge-base/SKILL.md
     - 1张子表: 知识库索引 + doc 存储
     - 命令: doc(创建文档保存内容) + smartsheet(索引) + WebFetch(抓取网页)
     - 无知识库 → 智能表格索引 + 文档存储

     Phase 3（3个 Skill）

     3.1 wecomcli-workflow-content-pipeline
     - 文件: .claude/skills/wecomcli-workflow-content-pipeline/SKILL.md
     - 2张子表: 创意库、创意研究
     - 命令: smartsheet(创意管理) + todo(任务跟踪) + doc(方案文档) + WebSearch(话题研究)

     3.2 wecomcli-workflow-stock-analyzer
     - 文件: .claude/skills/wecomcli-workflow-stock-analyzer/SKILL.md
     - 4张子表: 自选股、持仓、分析记录、财报日程
     - 命令: smartsheet(数据存储) + doc(分析报告) + WebSearch(股价/财报)
     - 4层架构: 自选股监控、财报提醒、深度研究(4AI分析师)、自动选股

     3.3 wecomcli-workflow-morning-brief
     - 文件: .claude/skills/wecomcli-workflow-morning-brief/SKILL.md
     - 无独立建表 → 复用 CRM + 社交追踪数据
     - 命令: schedule(日程) + todo(待办) + msg(消息摘要+发送简报) + smartsheet(读取联动数据)

     Phase 4（2个 Skill）

     4.1 wecomcli-workflow-business-advisor
     - 文件: .claude/skills/wecomcli-workflow-business-advisor/SKILL.md
     - references/prompt-templates.md — 8个AI顾问角色 prompt
     - 2张子表: 顾问分析记录、用户反馈
     - 依赖其他 Skill 数据联动

     4.2 wecomcli-workflow-multi-agent-dev
     - 文件: .claude/skills/wecomcli-workflow-multi-agent-dev/SKILL.md
     - 2张子表: 开发任务、竞赛记录
     - 命令: Agent Tool(多代理) + smartsheet(任务追踪) + todo(待办) + msg(通知)
     - 3种模式: 竞赛、分工、流水线

     验证方式

     每个 Skill 完成后：
     1. 确认 SKILL.md 格式正确（frontmatter + 工作流 + 命令示例）
     2. 确认字段类型写入格式与 crm-schema.md 一致
     3. 确认初始化流程完整（创建表格 → 保存 memory）
     4. 确认所有 wecom-cli 命令语法正确