---
name: wecomcli-workflow-multi-agent-dev
description: 企业微信多编程代理协同技能。支持竞赛模式、分工模式、流水线模式三种工作模式，通过企业微信追踪任务和结果。当用户说"并行开发"、"多代理"、"竞赛模式"、"分工模式"、"流水线协作"、"代码对比"、"两个代理同时"等涉及多代理编程的场景时触发。
---

# 企业微信多编程代理协同

通过 Agent Tool 启动多个子代理并行工作，企业微信智能表格追踪任务，消息通知结果。

## 首次配置（自动初始化）

**执行任何操作前，先检查 memory 中是否已有多代理协同配置。** 如果没有，按以下流程自动创建：

### 步骤 1：创建智能表格

```bash
wecom-cli doc create_doc '{"doc_type": 10, "doc_name": "多代理开发任务"}'
```

从返回结果中获取 `docid`。

### 步骤 2：创建子表和字段

```bash
# 创建子表1: 开发任务
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "开发任务"}'

# 创建子表2: 竞赛记录
wecom-cli doc smartsheet_add_sheet '{"docid": "<DOCID>", "sheet_name": "竞赛记录"}'
```

**开发任务字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "任务描述", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "工作模式", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "分配代理", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "状态", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "结果摘要", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "耗时", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "创建时间", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

**竞赛记录字段：**

```bash
wecom-cli doc smartsheet_add_fields '{"docid": "<DOCID>", "sheet_id": "<SHEET_ID>", "fields": [
  {"field_title": "任务描述", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "代理1方案", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "代理1评分", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "代理2方案", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "代理2评分", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "推荐方案", "field_type": "FIELD_TYPE_SINGLE_SELECT"},
  {"field_title": "对比分析", "field_type": "FIELD_TYPE_TEXT"},
  {"field_title": "创建时间", "field_type": "FIELD_TYPE_TEXT"}
]}'
```

### 步骤 3：保存配置到 memory

将以下信息保存到 memory 文件 `multi-agent-dev-config.md`：

```markdown
---
name: multi-agent-dev-config
description: 多代理协同智能表格配置信息
type: project
---

## 多代理协同表配置

- docid: <DOCID>
- url: <创建时返回的URL>
- 表格名称: 多代理开发任务
- 开发任务 sheet_id: <SHEET_ID>
- 竞赛记录 sheet_id: <SHEET_ID>
```

---

## 操作流程

> 以下命令中 `<DOCID>` 和各 `<SHEET_ID>` 从 memory 的 multi-agent-dev-config 中读取。

### 模式 1：竞赛模式

同一任务分配给 2-3 个代理各自独立完成，AI 对比选最优方案。

**适用场景：** 需要最优方案、架构选择、算法实现

**交互流程：**
1. 用户："用竞赛模式帮我实现一个用户登录模块，两个代理各写一份"
2. AI 记录任务到智能表格
3. 使用 Agent Tool 启动 2 个子代理（`subagent_type: "general-purpose"`）
4. 两个代理各自完成实现
5. AI 对比两个方案，评分选优
6. 发送结果到消息

**Agent Tool 调用示例：**

```
使用 Agent Tool 并行启动 2 个 general-purpose 子代理：

代理1 Prompt：实现一个用户登录模块。要求：
- 使用 JWT + Refresh Token 方案
- 包含密码强度校验
- 包含输入验证和错误处理
- 代码规范、有注释

代理2 Prompt：实现一个用户登录模块。要求：
- 使用 Session-based 方案
- 包含密码强度校验
- 包含输入验证和错误处理
- 代码规范、有注释
```

**记录结果：**

```bash
# 记录各代理任务
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<开发任务SHEET_ID>", "records": [
  {"values": {"任务描述": [{"type":"text","text":"用户登录模块"}], "工作模式": [{"text":"竞赛"}], "分配代理": [{"type":"text","text":"Agent1"}], "状态": [{"text":"已完成"}], "结果摘要": [{"type":"text","text":"JWT方案，完整性好"}], "耗时": [{"type":"text","text":"45s"}], "创建时间": [{"type":"text","text":"2026-04-02"}]}},
  {"values": {"任务描述": [{"type":"text","text":"用户登录模块"}], "工作模式": [{"text":"竞赛"}], "分配代理": [{"type":"text","text":"Agent2"}], "状态": [{"text":"已完成"}], "结果摘要": [{"type":"text","text":"Session方案，简洁"}], "耗时": [{"type":"text","text":"38s"}], "创建时间": [{"type":"text","text":"2026-04-02"}]}}
]}'

# 记录竞赛对比
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<竞赛记录SHEET_ID>", "records": [{"values": {
  "任务描述": [{"type":"text","text":"用户登录模块"}],
  "代理1方案": [{"type":"text","text":"JWT + Refresh Token，完整性好"}],
  "代理1评分": [{"type":"text","text":"8.5"}],
  "代理2方案": [{"type":"text","text":"Session-based，简洁清晰"}],
  "代理2评分": [{"type":"text","text":"7.0"}],
  "推荐方案": [{"text":"代理1"}],
  "对比分析": [{"type":"text","text":"JWT方案更符合现代最佳实践，Session方案更简洁但缺少密码强度校验"}],
  "创建时间": [{"type":"text","text":"2026-04-02"}]
}}]}'
```

### 模式 2：分工模式

不同任务分配给不同代理并行执行。

**适用场景：** 多个独立任务、批量 bug 修复、并行功能开发

**交互流程：**
1. 用户："我有3个任务要并行做：修bug #123、写登录页面、优化SQL查询"
2. AI 为每个任务启动一个子代理
3. 所有代理并行执行
4. 汇总结果

**Agent Tool 调用：**

```
并行启动 3 个 general-purpose 子代理：

代理1：修复 bug #123 — <具体 bug 描述>
代理2：写登录页面 — <具体需求>
代理3：优化 SQL 查询 — <具体优化目标>
```

**记录结果：**

```bash
wecom-cli doc smartsheet_add_records '{"docid": "<DOCID>", "sheet_id": "<开发任务SHEET_ID>", "records": [
  {"values": {"任务描述": [{"type":"text","text":"修bug #123"}], "工作模式": [{"text":"分工"}], "分配代理": [{"type":"text","text":"Agent1"}], "状态": [{"text":"已完成"}], "耗时": [{"type":"text","text":"45s"}], "创建时间": [{"type":"text","text":"2026-04-02"}]}},
  {"values": {"任务描述": [{"type":"text","text":"写登录页面"}], "工作模式": [{"text":"分工"}], "分配代理": [{"type":"text","text":"Agent2"}], "状态": [{"text":"已完成"}], "耗时": [{"type":"text","text":"120s"}], "创建时间": [{"type":"text","text":"2026-04-02"}]}},
  {"values": {"任务描述": [{"type":"text","text":"优化SQL查询"}], "工作模式": [{"text":"分工"}], "分配代理": [{"type":"text","text":"Agent3"}], "状态": [{"text":"已完成"}], "耗时": [{"type":"text","text":"60s"}], "创建时间": [{"type":"text","text":"2026-04-02"}]}}
]}'
```

**发送汇总通知：**

```bash
wecom-cli msg send_message '{"chat_type": 1, "chatid": "<USERID>", "msgtype": "text", "text": {"content": "分工任务全部完成！\n\n| 任务 | 代理 | 状态 | 耗时 |\n|------|------|------|------|\n| 修bug #123 | Agent1 | 完成 | 45s |\n| 写登录页面 | Agent2 | 完成 | 120s |\n| 优化SQL查询 | Agent3 | 完成 | 60s |\n\n总耗时 120s（串行需 225s，加速 1.9x）"}}'
```

### 模式 3：流水线模式

代码→审查→测试，按顺序由不同代理协作完成。

**适用场景：** 复杂功能开发、需要质量保障的任务

**交互流程：**
1. 用户："用流水线模式帮我重构这个模块"
2. 代理1：实现/重构代码
3. 代理2：代码审查，提出改进建议
4. 代理3：补充测试（可选）
5. 汇总结果

**Agent Tool 调用：**

```
按顺序启动 3 个子代理（注意：流水线是串行的，不是并行）：

阶段1 代理：重构 <模块名> 模块的代码。要求：<具体要求>

阶段2 代理（接收阶段1结果）：对以下代码进行审查。要求：
- 检查代码质量、潜在 bug、性能问题
- 提出 3-5 个改进建议
- 评估代码可维护性

阶段3 代理（接收阶段1+2结果）：为以下代码编写完整的单元测试。要求：
- 覆盖所有主要功能路径
- 包含边界情况测试
```

---

## 三种模式对比

| 模式 | 适用场景 | 代理数量 | 执行方式 | 加速比 |
|------|---------|---------|---------|--------|
| 竞赛 | 需要最优方案 | 2-3个 | 同一任务，各自完成，AI对比 | 1x（取最慢） |
| 分工 | 多个独立任务 | 按任务数 | 不同任务，并行完成 | Nx |
| 流水线 | 复杂功能，需质量保障 | 2-3个 | 代码→审查→测试，顺序协作 | ~0.7x（有审查开销） |

---

## 字段类型写入格式（通用规则）

> 完整字段类型格式说明参见 [crm-schema](../wecomcli-crm/references/crm-schema.md)

| 字段类型 | 写入格式 | 示例 |
|----------|----------|------|
| TEXT | `[{"type":"text","text":"值"}]` | 任务描述、代理名、耗时等 |
| SINGLE_SELECT | `[{"text":"选项文本"}]` | 工作模式、状态、推荐方案等 |

---

## 实操注意事项

### 常见踩坑

1. **更新时不传 key_type** — 必须带 `"key_type": "CELL_VALUE_KEY_TYPE_FIELD_TITLE"`
2. **API 调用失败时** — 检查错误码：301085 表示 docid 无效（改用 url）；2022004 表示字段未找到（先 add_fields）；2022003 表示记录不存在（先 add_records）。遇到错误先用 `smartsheet_get_sheet '{"url": "<URL>"}'` 验证表结构。
3. **Agent Tool 隔离** — 竞赛模式的两个代理需要在 `isolation: "worktree"` 中独立工作，避免互相干扰
4. **流水线串行依赖** — 流水线模式下，阶段 2 和 3 的 prompt 需要包含阶段 1 的输出
5. **任务粒度** — 分工模式的每个任务应足够独立，避免交叉依赖
6. **代理数量限制** — 建议最多同时启动 3 个代理，避免资源竞争
7. **SINGLE_SELECT 创建时 options 不生效** — `smartsheet_add_fields` 传入的 `property.options` 不会预创建选项，选项在首次写入记录时通过 `[{"text": "选项"}]` 自动创建。创建字段时可省略 `property` 参数。

### 数据联动

- 本 Skill 独立运行，不依赖其他 workflow Skill
- 使用底层 `wecom-cli todo` 创建跟踪待办（可选）
- 使用底层 `wecom-cli msg` 发送通知
