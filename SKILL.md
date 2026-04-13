---
name: create-belaidoff
description: "Create a layoff response AI Skill. Collect personal information and evidence, generate rights protection capabilities and personal profile. | 创建被裁员应对 AI Skill，收集个人信息和证据，生成权益保护能力和个人画像。"
argument-hint: "[employee-name-or-slug]"
version: "1.0.0"
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
---

> **Language / 语言**: This skill supports both English and Chinese. Detect the user's language from their first message and respond in the same language throughout. Below are instructions in both languages — follow the one matching the user's language.
>
> 本 Skill 支持中英文。根据用户第一条消息的语言，全程使用同一语言回复。下方提供了两种语言的指令，按用户语言选择对应版本执行。

# 被裁员.skill 创建器（Claude Code 版）

## 触发条件

当用户说以下任意内容时启动：
- `/create-belaidoff`
- "帮我创建一个被裁员 skill"
- "我想做一个被裁员应对助手"
- "新建被裁员 skill"
- "给我做一个被裁员的 skill"

当用户对已有被裁员 Skill 说以下内容时，进入进化模式：
- "我有新文件" / "追加"
- "这不对" / "应该是"
- `/update-belaidoff {slug}`

当用户说 `/list-belaidoff` 时列出所有已生成的被裁员 Skill。

---

## 工具使用规则

本 Skill 运行在 Claude Code 环境，使用以下工具：

| 任务 | 使用工具 |
|------|---------|
| 读取 PDF 文档 | `Read` 工具（原生支持 PDF） |
| 读取图片截图 | `Read` 工具（原生支持图片） |
| 读取 MD/TXT 文件 | `Read` 工具 |
| 解析飞书消息 JSON 导出 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_parser.py` |
| 飞书全自动采集（推荐） | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py` |
| 飞书文档（浏览器登录态） | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_browser.py` |
| 飞书文档（MCP App Token） | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py` |
| 钉钉全自动采集 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/dingtalk_auto_collector.py` |
| 解析邮件 .eml/.mbox | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/email_parser.py` |
| 写入/更新 Skill 文件 | `Write` / `Edit` 工具 |
| 版本管理 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| 列出已有 Skill | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list` |

**基础目录**：Skill 文件写入 `./belaidoff/{slug}/`（相对于本项目目录）。
如需改为全局路径，用 `--base-dir ~/.openclaw/workspace/skills/belaidoff`。

---

## 主流程：创建新被裁员 Skill

### Step 1：个人情况录入（5 个问题）

参考 `${CLAUDE_SKILL_DIR}/prompts/intake.md` 的问题序列，问 5 个问题：

1. **姓名/昵称**（必填）
2. **工作基本信息**（一句话：公司、入职时间、职级、年薪、绩效情况）
   - 示例：`阿里巴巴 2020-01-01 3-1 60万/年 325`
3. **个人状态**（一句话：年龄、学历、婚姻状况、子女情况、房贷情况）
   - 示例：`32 硕士 已婚 1个 有（月供8000元）`
4. **裁员情况**（一句话：当前阶段和诉求）
   - 示例：`正在谈判 2N赔偿`
5. **个人状态标签**（可选）
   - 示例：`有子女 有房贷 焦虑不安 据理力争 中风险`

除姓名外均可跳过。收集完后汇总确认再进入下一步。

### Step 2：证据材料导入

询问用户提供证据材料，展示四种方式供选择：

```
证据材料怎么提供？

  [A] 飞书自动采集（推荐）
      输入姓名，自动拉取消息记录 + 文档 + 多维表格

  [B] 钉钉自动采集
      输入姓名，自动拉取文档 + 多维表格
      消息记录通过浏览器采集（钉钉 API 不支持历史消息）

  [C] 飞书链接
      直接给文档/Wiki 链接（浏览器登录态 或 MCP）

  [D] 上传文件
      PDF / 图片 / 导出 JSON / 邮件 .eml

  [E] 直接粘贴内容
      把文字复制进来

可以混用，也可以跳过（仅凭手动信息生成）。
```

---

#### 方式 A：飞书自动采集（推荐）

首次使用需配置：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py --setup
```

**群聊采集**（使用 tenant_access_token，需 bot 在群内）：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py \
  --name "{name}" \
  --output-dir ./evidence/{slug} \
  --msg-limit 1000 \
  --doc-limit 20
```

**私聊采集**（需要 user_access_token + 私聊 chat_id）：

私聊消息只能通过用户身份（user_access_token）获取，应用身份无权访问私聊。

**前置条件**：

用户需要提供以下信息：
1. **飞书应用凭证**：`app_id` 和 `app_secret`（在飞书开放平台创建自建应用获取）
2. **用户权限**：应用需开通以下用户权限（scope）：
   - `im:message` — 以用户身份读取/发送消息
   - `im:chat` — 以用户身份读取会话列表
3. **OAuth 授权码（code）**：用户在浏览器中完成 OAuth 授权后，从回调 URL 中获取

如果用户缺少以上任何信息，引导他们完成配置。不要假设用户已经配好了。

**获取 user_access_token 的完整流程**：

当用户提供了 app_id、app_secret，并确认已开通用户权限后：

1. 帮用户生成 OAuth 授权链接：
   ```
   https://open.feishu.cn/open-apis/authen/v1/authorize?app_id={APP_ID}&redirect_uri=http://www.example.com&scope=im:message%20im:chat
   ```
   > ⚠️ 注意：`redirect_uri` 需要在飞书应用的「安全设置 → 重定向 URL」中添加 `http://www.example.com`
   
2. 用户在浏览器打开链接，登录并授权
3. 页面会跳转到 `http://www.example.com?code=xxx`，用户复制 code 给你
4. 用 code 换取 token：
   ```bash
   python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py --exchange-code {CODE}
   ```
   或者你自己写 Python 脚本调飞书 API 换取：
   ```python
   # 1. 获取 app_access_token
   POST https://open.feishu.cn/open-apis/auth/v3/app_access_token/internal
   Body: {"app_id": "xxx", "app_secret": "xxx"}
   
   # 2. 用 code 换 user_access_token
   POST https://open.feishu.cn/open-apis/authen/v1/oidc/access_token
   Header: Authorization: Bearer {app_access_token}
   Body: {"grant_type": "authorization_code", "code": "xxx"}
   ```

**获取私聊 chat_id**：

用户通常不知道 chat_id。当用户有了 user_access_token 但没有 chat_id 时，你应该**自己写 Python 脚本**来获取：

- **方法**：用 user_access_token 向对方的 open_id 发一条消息，返回值中会包含 chat_id
  ```python
  POST https://open.feishu.cn/open-apis/im/v1/messages?receive_id_type=open_id
  Header: Authorization: Bearer {user_access_token}
  Body: {"receive_id": "{对方open_id}", "msg_type": "text", "content": "{\"text\":\"你好\"}"}
  # 返回值中的 chat_id 就是私聊会话 ID
  ```
- **注意**：`GET /im/v1/chats` 不会返回私聊会话，这是飞书 API 的限制，不是权限问题，不要尝试用这个接口找私聊
- 如果用户不知道对方的 open_id，可以用 tenant_access_token 调通讯录 API 搜索：
  ```python
  GET https://open.feishu.cn/open-apis/contact/v3/scopes
  # 返回应用可见范围内所有用户的 open_id
  ```

**执行采集**：

拿到 user_access_token 和 chat_id 后：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py \
  --open-id {对方open_id} \
  --p2p-chat-id {chat_id} \
  --user-token {user_access_token} \
  --name "{name}" \
  --output-dir ./evidence/{slug} \
  --msg-limit 1000
```

**灵活性原则**：以上 API 调用不一定要用 collector 脚本，如果脚本跑不通或者场景不匹配，你可以直接写 Python 脚本调飞书 API 完成任务。核心 API 参考：
- 获取 token：`POST /auth/v3/app_access_token/internal`、`POST /authen/v1/oidc/access_token`
- 发消息（获取 chat_id）：`POST /im/v1/messages?receive_id_type=open_id`
- 拉消息：`GET /im/v1/messages?container_id_type=chat&container_id={chat_id}`
- 查通讯录：`GET /contact/v3/scopes`、`GET /contact/v3/users/{user_id}`

自动采集内容：
- 群聊：所有与 HR/管理层的群聊消息（过滤系统消息、表情包）
- 私聊：与 HR/管理层的私聊完整对话（含双方消息，用于理解对话语境）
- 相关飞书文档和 Wiki（如裁员通知、绩效考核等）
- 相关多维表格（如有权限）

采集完成后用 `Read` 读取输出目录下的文件：
- `evidence/{slug}/messages.txt` → 消息记录（群聊 + 私聊）
- `evidence/{slug}/docs.txt` → 文档内容
- `evidence/{slug}/collection_summary.json` → 采集摘要

如果采集失败，根据报错自行判断原因并尝试修复，常见问题：
- 群聊采集：bot 未添加到群聊
- 私聊采集：user_access_token 过期（有效期 2 小时，可用 refresh_token 刷新）
- 权限不足：引导用户在飞书开放平台开通对应权限并重新授权
- 或改用方式 B/C

---

#### 方式 B：钉钉自动采集

首次使用需配置：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/dingtalk_auto_collector.py --setup
```

然后输入姓名，一键采集：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/dingtalk_auto_collector.py \
  --name "{name}" \
  --output-dir ./evidence/{slug} \
  --msg-limit 500 \
  --doc-limit 20 \
  --show-browser   # 首次使用加此参数，完成钉钉登录
```

采集内容：
- 相关钉钉文档和知识库（如裁员通知、绩效考核等）
- 多维表格
- 消息记录（⚠️ 钉钉 API 不支持历史消息拉取，自动切换浏览器采集）

采集完成后 `Read` 读取：
- `evidence/{slug}/docs.txt`
- `evidence/{slug}/bitables.txt`
- `evidence/{slug}/messages.txt`

如消息采集失败，提示用户截图聊天记录后上传。

---

#### 方式 D：上传文件

- **PDF / 图片**：`Read` 工具直接读取
- **飞书消息 JSON 导出**：
  ```bash
  python3 ${CLAUDE_SKILL_DIR}/tools/feishu_parser.py --file {path} --target "{name}" --output /tmp/feishu_out.txt
  ```
  然后 `Read /tmp/feishu_out.txt`
- **邮件文件 .eml / .mbox**：
  ```bash
  python3 ${CLAUDE_SKILL_DIR}/tools/email_parser.py --file {path} --target "{name}" --output /tmp/email_out.txt
  ```
  然后 `Read /tmp/email_out.txt`
- **Markdown / TXT**：`Read` 工具直接读取

---

#### 方式 C：飞书链接

用户提供飞书文档/Wiki 链接时，询问读取方式：

```
检测到飞书链接，选择读取方式：

  [1] 浏览器方案（推荐）
      复用你本机 Chrome 的登录状态
      ✅ 内部文档、需要权限的文档都能读
      ✅ 无需配置 token
      ⚠️  需要本机安装 Chrome + playwright

  [2] MCP 方案
      通过飞书 App Token 调用官方 API
      ✅ 稳定，不依赖浏览器
      ✅ 可以读消息记录（需要群聊 ID）
      ⚠️  需要先配置 App ID / App Secret
      ⚠️  内部文档需要管理员给应用授权

选择 [1/2]：
```

**选 1（浏览器方案）**：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_browser.py \
  --url "{feishu_url}" \
  --target "{name}" \
  --output /tmp/feishu_doc_out.txt
```
首次使用若未登录，会弹出浏览器窗口要求登录（一次性）。

**选 2（MCP 方案）**：

首次使用需初始化配置：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py --setup
```

之后直接读取：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py \
  --url "{feishu_url}" \
  --output /tmp/feishu_doc_out.txt
```

读取消息记录（需要群聊 ID，格式 `oc_xxx`）：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py \
  --chat-id "oc_xxx" \
  --target "{name}" \
  --limit 500 \
  --output /tmp/feishu_msg_out.txt
```

两种方式输出后均用 `Read` 读取结果文件，进入分析流程。

---

#### 方式 E：直接粘贴

用户粘贴的内容直接作为文本原材料，无需调用任何工具。

---

如果用户说"没有文件"或"跳过"，仅凭 Step 1 的手动信息生成 Skill。

### Step 3：分析原材料

将收集到的所有原材料和用户填写的个人情况汇总，按以下两条线分析：

**线路 A（权益保护能力）**：
- 参考 `${CLAUDE_SKILL_DIR}/prompts/capabilities_analyzer.md` 中的提取维度
- 提取：赔偿计算、谈判策略、文件模板、法律知识库
- 根据裁员阶段和诉求重点提取相关内容

**线路 B（个人画像）**：
- 参考 `${CLAUDE_SKILL_DIR}/prompts/persona_analyzer.md` 中的提取维度
- 将用户填写的标签翻译为具体行为规则
- 从原材料中提取：心理状态、沟通风格、家庭负担

### Step 4：生成并预览

参考 `${CLAUDE_SKILL_DIR}/prompts/capabilities_builder.md` 生成权益保护能力内容。
参考 `${CLAUDE_SKILL_DIR}/prompts/persona_builder.md` 生成个人画像内容（5 层结构）。

向用户展示摘要（各 5-8 行），询问：
```
权益保护能力摘要：
  - 赔偿计算：{xxx}
  - 谈判策略：{xxx}
  - 法律依据：{xxx}
  ...

个人画像摘要：
  - 核心诉求：{xxx}
  - 心理状态：{xxx}
  - 家庭负担：{xxx}
  ...

确认生成？还是需要调整？
```

### Step 5：写入文件

用户确认后，执行以下写入操作：

**1. 创建目录结构**（用 Bash）：
```bash
mkdir -p belaidoff/{slug}/versions
mkdir -p belaidoff/{slug}/evidence/contracts
mkdir -p belaidoff/{slug}/evidence/communications
mkdir -p belaidoff/{slug}/evidence/financial
mkdir -p belaidoff/{slug}/evidence/performance
```

**2. 写入 capabilities.md**（用 Write 工具）：
路径：`belaidoff/{slug}/capabilities.md`

**3. 写入 persona.md**（用 Write 工具）：
路径：`belaidoff/{slug}/persona.md`

**4. 写入 meta.json**（用 Write 工具）：
路径：`belaidoff/{slug}/meta.json`
内容：
```json
{
  "name": "{name}",
  "slug": "{slug}",
  "created_at": "{ISO时间}",
  "updated_at": "{ISO时间}",
  "version": "v1",
  "profile": {
    "company": "{company}",
    "hire_date": "{hire_date}",
    "level": "{level}",
    "salary": "{salary}",
    "performance": "{performance}",
    "age": {age},
    "education": "{education}",
    "marital": "{marital}",
    "children": "{children}",
    "mortgage": "{mortgage}",
    "layoff_stage": "{layoff_stage}",
    "demands": "{demands}"
  },
  "tags": {
    "family_burden": [...],
    "mental_state": [...],
    "communication_style": [...],
    "risk_tolerance": [...]
  },
  "impression": "{impression}",
  "evidence_sources": [...已导入文件列表],
  "corrections_count": 0
}
```

**5. 生成完整 SKILL.md**（用 Write 工具）：
路径：`belaidoff/{slug}/SKILL.md`

SKILL.md 结构：
```markdown
---
name: belaidoff-{slug}
description: {name}，{company} 被裁员应对助手
user-invocable: true
---

# {name}

{company} {level}，入职于 {hire_date}
{age}岁，{education}，{marital}，{children}，{mortgage}
当前处于 {layoff_stage}，诉求是 {demands}

---

## PART A：权益保护能力

{capabilities.md 全部内容}

---

## PART B：个人画像

{persona.md 全部内容}

---

## 运行规则

1. 先由 PART B 判断：用户的需求和心理状态是什么？
2. 再由 PART A 执行：提供具体的权益保护方案
3. 输出时考虑用户的沟通风格和风险承受能力
4. PART B Layer 0 的规则优先级最高，任何情况下不得违背
```

告知用户：
```
✅ 被裁员应对 Skill 已创建！

文件位置：belaidoff/{slug}/
触发词：/{slug}（完整版）
        /{slug}-capabilities（仅权益保护能力）
        /{slug}-persona（仅个人画像）

如果用起来感觉哪里不对，直接说"应该是"，我来更新。
```

---

## 进化模式：追加文件

用户提供新文件或文本时：

1. 按 Step 2 的方式读取新内容
2. 用 `Read` 读取现有 `belaidoff/{slug}/capabilities.md` 和 `persona.md`
3. 参考 `${CLAUDE_SKILL_DIR}/prompts/merger.md` 分析增量内容
4. 存档当前版本（用 Bash）：
   ```bash
   python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action backup --slug {slug} --base-dir ./belaidoff
   ```
5. 用 `Edit` 工具追加增量内容到对应文件
6. 重新生成 `SKILL.md`（合并最新 capabilities.md + persona.md）
7. 更新 `meta.json` 的 version 和 updated_at

---

## 进化模式：对话纠正

用户表达"不对"/"应该是"时：

1. 参考 `${CLAUDE_SKILL_DIR}/prompts/correction_handler.md` 识别纠正内容
2. 判断属于 Capabilities（法律/赔偿/策略）还是 Persona（心理/沟通/负担）
3. 生成 correction 记录
4. 用 `Edit` 工具追加到对应文件的 `## Correction 记录` 节
5. 重新生成 `SKILL.md`

---

## 管理命令

`/list-belaidoff`：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./belaidoff
```

`/belaidoff-rollback {slug} {version}`：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action rollback --slug {slug} --version {version} --base-dir ./belaidoff
```

`/delete-belaidoff {slug}`：
确认后执行：
```bash
rm -rf belaidoff/{slug}
```

---
---

# English Version

# Belaidoff.skill Creator (Claude Code Edition)

## Trigger Conditions

Activate when the user says any of the following:
- `/create-belaidoff`
- "Help me create a layoff response skill"
- "I want to make a layoff应对 assistant"
- "New layoff skill"
- "Make a skill for layoff"

Enter evolution mode when the user says:
- "I have new files" / "append"
- "That's wrong" / "It should be"
- `/update-belaidoff {slug}`

List all generated layoff skills when the user says `/list-belaidoff`.

---

## Tool Usage Rules

This Skill runs in the Claude Code environment with the following tools:

| Task | Tool |
|------|------|
| Read PDF documents | `Read` tool (native PDF support) |
| Read image screenshots | `Read` tool (native image support) |
| Read MD/TXT files | `Read` tool |
| Parse Feishu message JSON export | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_parser.py` |
| Feishu auto-collect (recommended) | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py` |
| Feishu docs (browser session) | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_browser.py` |
| Feishu docs (MCP App Token) | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py` |
| DingTalk auto-collect | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/dingtalk_auto_collector.py` |
| Parse email .eml/.mbox | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/email_parser.py` |
| Write/update Skill files | `Write` / `Edit` tool |
| Version management | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| List existing Skills | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list` |

**Base directory**: Skill files are written to `./belaidoff/{slug}/` (relative to the project directory).
For a global path, use `--base-dir ~/.openclaw/workspace/skills/belaidoff`.

---

## Main Flow: Create a New Belaidoff Skill

### Step 1: Personal Info Collection (5 questions)

Refer to `${CLAUDE_SKILL_DIR}/prompts/intake.md` for the question sequence. Ask 5 questions:

1. **Name / Nickname** (required)
2. **Work basic info** (one sentence: company, hire date, level, salary, performance)
   - Example: `ByteDance 2020-01-01 3-1 400k/year B`
3. **Personal status** (one sentence: age, education, marital status, children, mortgage)
   - Example: `32 master married 1 child has mortgage (8000/month)`
4. **Layoff situation** (one sentence: current stage and demands)
   - Example: `negotiating 2N compensation`
5. **Personal status tags** (optional)
   - Example: `has children has mortgage anxious assertive medium risk`

Everything except the name can be skipped. Summarize and confirm before moving to the next step.

### Step 2: Evidence Import

Ask the user how they'd like to provide evidence:

```
How would you like to provide evidence?

  [A] Feishu Auto-Collect (recommended)
      Enter name, auto-pull messages + docs + spreadsheets

  [B] DingTalk Auto-Collect
      Enter name, auto-pull docs + spreadsheets
      Messages collected via browser (DingTalk API doesn't support message history)

  [C] Feishu Link
      Provide doc/Wiki link (browser session or MCP)

  [D] Upload Files
      PDF / images / exported JSON / email .eml

  [E] Paste Text
      Copy-paste text directly

Can mix and match, or skip entirely (generate from manual info only).
```

---

#### Option A: Feishu Auto-Collect (Recommended)

First-time setup:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py --setup
```

**Group chat collection** (uses tenant_access_token, bot must be in the group):
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py \
  --name "{name}" \
  --output-dir ./evidence/{slug} \
  --msg-limit 1000 \
  --doc-limit 20
```

**Private chat (P2P) collection** (requires user_access_token + p2p chat_id):

Private messages can only be accessed via user identity (user_access_token). App identity cannot access private chats.

**Prerequisites**:

The user needs to provide:
1. **Feishu app credentials**: `app_id` and `app_secret` (from Feishu Open Platform)
2. **User scopes**: The app must have these user scopes enabled:
   - `im:message` — read/send messages as user
   - `im:chat` — read chat list as user
3. **OAuth authorization code**: obtained after user completes OAuth in browser

If the user is missing any of these, guide them through setup. Don't assume anything is pre-configured.

**Getting user_access_token**:

Once the user provides app_id, app_secret, and confirms scopes are enabled:

1. Generate the OAuth URL for them:
   ```
   https://open.feishu.cn/open-apis/authen/v1/authorize?app_id={APP_ID}&redirect_uri=http://www.example.com&scope=im:message%20im:chat
   ```
   > ⚠️ The redirect_uri must be added in the app's "Security Settings → Redirect URLs"

2. User opens URL, logs in, authorizes
3. Page redirects to `http://www.example.com?code=xxx`, user copies the code
4. Exchange code for token:
   ```bash
   python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py --exchange-code {CODE}
   ```
   Or write a Python script to call the Feishu API directly:
   ```python
   # 1. Get app_access_token
   POST https://open.feishu.cn/open-apis/auth/v3/app_access_token/internal
   Body: {"app_id": "xxx", "app_secret": "xxx"}
   
   # 2. Exchange code for user_access_token
   POST https://open.feishu.cn/open-apis/authen/v1/oidc/access_token
   Header: Authorization: Bearer {app_access_token}
   Body: {"grant_type": "authorization_code", "code": "xxx"}
   ```

**Getting the p2p chat_id**:

Users typically don't know their chat_id. When the user has a user_access_token but no chat_id, **write a Python script yourself** to obtain it:

- **Method**: Send a message to the other user's open_id — the response includes the chat_id
  ```python
  POST https://open.feishu.cn/open-apis/im/v1/messages?receive_id_type=open_id
  Header: Authorization: Bearer {user_access_token}
  Body: {"receive_id": "{target_open_id}", "msg_type": "text", "content": "{\"text\":\"hello\"}"}
  # The chat_id in the response is the p2p chat ID
  ```
- **Important**: `GET /im/v1/chats` does NOT return p2p chats — this is a Feishu API limitation, not a permission issue. Do not try to use it for finding private chats.
- If the user doesn't know the target's open_id, use tenant_access_token to search contacts:
  ```python
  GET https://open.feishu.cn/open-apis/contact/v3/scopes
  # Returns open_ids of all users visible to the app
  ```

**Running collection**:

Once you have user_access_token and chat_id:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py \
  --open-id {target_open_id} \
  --p2p-chat-id {chat_id} \
  --user-token {user_access_token} \
  --name "{name}" \
  --output-dir ./evidence/{slug} \
  --msg-limit 1000
```

**Flexibility principle**: The above API calls don't have to go through the collector script. If the script doesn't work or doesn't fit the scenario, write Python scripts directly to call Feishu APIs. Key API reference:
- Get token: `POST /auth/v3/app_access_token/internal`, `POST /authen/v1/oidc/access_token`
- Send message (get chat_id): `POST /im/v1/messages?receive_id_type=open_id`
- Fetch messages: `GET /im/v1/messages?container_id_type=chat&container_id={chat_id}`
- Search contacts: `GET /contact/v3/scopes`, `GET /contact/v3/users/{user_id}`

Auto-collected content:
- Group chats: messages with HR/management (system messages and stickers filtered)
- Private chats: full conversation with HR/management (for context understanding)
- Relevant Feishu docs and Wikis (e.g., layoff notices, performance reviews)
- Related spreadsheets (if accessible)

After collection, `Read` the output files:
- `evidence/{slug}/messages.txt` → messages (group + private)
- `evidence/{slug}/docs.txt` → document content
- `evidence/{slug}/collection_summary.json` → collection summary

If collection fails, diagnose the error and attempt to fix it. Common issues:
- Group chat: bot not added to the group
- Private chat: user_access_token expired (2-hour TTL, refresh with refresh_token)
- Insufficient permissions: guide user to enable scopes and re-authorize
- Or switch to Option B/C

---

#### Option B: DingTalk Auto-Collect

First-time setup:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/dingtalk_auto_collector.py --setup
```

Then enter the name:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/dingtalk_auto_collector.py \
  --name "{name}" \
  --output-dir ./evidence/{slug} \
  --msg-limit 500 \
  --doc-limit 20 \
  --show-browser   # add this flag on first use to complete DingTalk login
```

Collected content:
- Relevant DingTalk docs and knowledge bases (e.g., layoff notices, performance reviews)
- Spreadsheets
- Messages (⚠️ DingTalk API doesn't support message history — auto-switches to browser scraping)

After collection, `Read`:
- `evidence/{slug}/docs.txt`
- `evidence/{slug}/bitables.txt`
- `evidence/{slug}/messages.txt`

If message collection fails, prompt user to upload chat screenshots.

---

#### Option D: Upload Files

- **PDF / Images**: `Read` tool directly
- **Feishu message JSON export**:
  ```bash
  python3 ${CLAUDE_SKILL_DIR}/tools/feishu_parser.py --file {path} --target "{name}" --output /tmp/feishu_out.txt
  ```
  Then `Read /tmp/feishu_out.txt`
- **Email files .eml / .mbox**:
  ```bash
  python3 ${CLAUDE_SKILL_DIR}/tools/email_parser.py --file {path} --target "{name}" --output /tmp/email_out.txt
  ```
  Then `Read /tmp/email_out.txt`
- **Markdown / TXT**: `Read` tool directly

---

#### Option C: Feishu Link

When the user provides a Feishu doc/Wiki link, ask which method to use:

```
Feishu link detected. Choose read method:

  [1] Browser Method (recommended)
      Reuses your local Chrome login session
      ✅ Works with internal docs requiring permissions
      ✅ No token configuration needed
      ⚠️  Requires Chrome + playwright installed locally

  [2] MCP Method
      Uses Feishu App Token via official API
      ✅ Stable, no browser dependency
      ✅ Can read messages (needs chat ID)
      ⚠️  Requires App ID / App Secret setup
      ⚠️  Internal docs need admin authorization for the app

Choose [1/2]:
```

**Option 1 (Browser)**:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_browser.py \
  --url "{feishu_url}" \
  --target "{name}" \
  --output /tmp/feishu_doc_out.txt
```
First use will open a browser window for login (one-time).

**Option 2 (MCP)**:

First-time setup:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py --setup
```

Then read directly:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py \
  --url "{feishu_url}" \
  --output /tmp/feishu_doc_out.txt
```

Read messages (needs chat ID, format `oc_xxx`):
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py \
  --chat-id "oc_xxx" \
  --target "{name}" \
  --limit 500 \
  --output /tmp/feishu_msg_out.txt
```

Both methods output to files, then use `Read` to load results into analysis.

---

#### Option E: Paste Text

User-pasted content is used directly as text material. No tools needed.

---

If the user says "no files" or "skip", generate Skill from Step 1 manual info only.

### Step 3: Analyze Source Material

Combine all collected materials and user-provided info, analyze along two tracks:

**Track A (Capabilities)**:
- Refer to `${CLAUDE_SKILL_DIR}/prompts/capabilities_analyzer.md` for extraction dimensions
- Extract: compensation calculation, negotiation strategies, document templates, legal knowledge
- Emphasize content relevant to the layoff stage and demands

**Track B (Persona)**:
- Refer to `${CLAUDE_SKILL_DIR}/prompts/persona_analyzer.md` for extraction dimensions
- Translate user-provided tags into concrete behavior rules
- Extract from materials: mental state, communication style, family burden

### Step 4: Generate and Preview

Use `${CLAUDE_SKILL_DIR}/prompts/capabilities_builder.md` to generate capabilities content.
Use `${CLAUDE_SKILL_DIR}/prompts/persona_builder.md` to generate persona content (5-layer structure).

Show the user a summary (5-8 lines each), ask:
```
Capabilities Summary:
  - Compensation calculation: {xxx}
  - Negotiation strategies: {xxx}
  - Legal basis: {xxx}
  ...

Persona Summary:
  - Core demands: {xxx}
  - Mental state: {xxx}
  - Family burden: {xxx}
  ...

Confirm generation? Or need adjustments?
```

### Step 5: Write Files

After user confirmation, execute the following:

**1. Create directory structure** (Bash):
```bash
mkdir -p belaidoff/{slug}/versions
mkdir -p belaidoff/{slug}/evidence/contracts
mkdir -p belaidoff/{slug}/evidence/communications
mkdir -p belaidoff/{slug}/evidence/financial
mkdir -p belaidoff/{slug}/evidence/performance
```

**2. Write capabilities.md** (Write tool):
Path: `belaidoff/{slug}/capabilities.md`

**3. Write persona.md** (Write tool):
Path: `belaidoff/{slug}/persona.md`

**4. Write meta.json** (Write tool):
Path: `belaidoff/{slug}/meta.json`
Content:
```json
{
  "name": "{name}",
  "slug": "{slug}",
  "created_at": "{ISO_timestamp}",
  "updated_at": "{ISO_timestamp}",
  "version": "v1",
  "profile": {
    "company": "{company}",
    "hire_date": "{hire_date}",
    "level": "{level}",
    "salary": "{salary}",
    "performance": "{performance}",
    "age": {age},
    "education": "{education}",
    "marital": "{marital}",
    "children": "{children}",
    "mortgage": "{mortgage}",
    "layoff_stage": "{layoff_stage}",
    "demands": "{demands}"
  },
  "tags": {
    "family_burden": [...],
    "mental_state": [...],
    "communication_style": [...],
    "risk_tolerance": [...]
  },
  "impression": "{impression}",
  "evidence_sources": [...imported file list],
  "corrections_count": 0
}
```

**5. Generate full SKILL.md** (Write tool):
Path: `belaidoff/{slug}/SKILL.md`

SKILL.md structure:
```markdown
---
name: belaidoff-{slug}
description: {name}, {company} layoff response assistant
user-invocable: true
---

# {name}

{company} {level}, hired on {hire_date}
{age} years old, {education}, {marital}, {children}, {mortgage}
Currently in {layoff_stage}, demands {demands}

---

## PART A: Rights Protection Capabilities

{full capabilities.md content}

---

## PART B: Personal Profile

{full persona.md content}

---

## Execution Rules

1. PART B decides first: what are the user's needs and mental state?
2. PART A executes: provide specific rights protection solutions
3. Consider the user's communication style and risk tolerance in output
4. PART B Layer 0 rules have the highest priority and must never be violated
```

Inform user:
```
✅ Layoff response Skill created!

Location: belaidoff/{slug}/
Commands: /{slug} (full version)
          /{slug}-capabilities (rights protection only)
          /{slug}-persona (personal profile only)

If something feels off, just say "it should be" and I'll update it.
```

---

## Evolution Mode: Append Files

When user provides new files or text:

1. Read new content using Step 2 methods
2. `Read` existing `belaidoff/{slug}/capabilities.md` and `persona.md`
3. Refer to `${CLAUDE_SKILL_DIR}/prompts/merger.md` for incremental analysis
4. Archive current version (Bash):
   ```bash
   python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action backup --slug {slug} --base-dir ./belaidoff
   ```
5. Use `Edit` tool to append incremental content to relevant files
6. Regenerate `SKILL.md` (merge latest capabilities.md + persona.md)
7. Update `meta.json` version and updated_at

---

## Evolution Mode: Conversation Correction

When user expresses "that's wrong" / "it should be":

1. Refer to `${CLAUDE_SKILL_DIR}/prompts/correction_handler.md` to identify correction content
2. Determine if it belongs to Capabilities (legal/compensation/strategy) or Persona (mental/communication/burden)
3. Generate correction record
4. Use `Edit` tool to append to the `## Correction Log` section of the relevant file
5. Regenerate `SKILL.md`

---

## Management Commands

`/list-belaidoff`:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./belaidoff
```

`/belaidoff-rollback {slug} {version}`:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action rollback --slug {slug} --version {version} --base-dir ./belaidoff
```

`/delete-belaidoff {slug}`:
After confirmation:
```bash
rm -rf belaidoff/{slug}
```