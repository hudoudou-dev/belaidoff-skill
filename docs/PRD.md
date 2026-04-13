# 被裁员.skill —— 产品需求文档 v1.0

---

## 一、产品概述

**被裁员.skill** 是一个运行在 OpenClaw 上的 meta-skill，专为面临裁员的员工提供权益保护和心理支持。

用户通过对话式交互提供个人情况和相关证据，系统自动生成可独立运行的**被裁员应对 Skill**，帮助员工：
- 计算应得赔偿
- 制定谈判策略
- 提供心理疏导
- 准备法律文件

生成的 Skill 由两部分组成：
- **Part A — 权益保护能力**：劳动法知识、赔偿计算、谈判策略、文件模板
- **Part B — 个人画像**：员工个人情况、家庭负担、心理状态、沟通风格

两部分可独立使用，也可组合运行（默认组合）。生成后的 Skill 支持通过追加文件或对话纠正持续进化。

核心逻辑：本产品不仅是赔偿计算器，更是**“职场数字孪生”**个人助理。通过 “法理能力 (Capabilities)” 与 “心理/背景画像 (Persona)” 的解耦与融合，模拟真实裁员博弈场景，为用户提供个性化的抗压策略，争取最大权益。

---

## 二、用户流程

```
用户触发 /create-belaidoff
        ↓
[Step 1] 个人情况录入（全部可跳过）
  - 姓名/昵称
  - 基本信息：公司、所在城市、入职时间、职级、年薪、绩效情况
  - 个人状态：性别、年龄、学历、婚姻状况、子女情况、父母情况、房贷情况
  - 裁员阶段：口头通知/书面通知/正在谈判/已离职
  - 诉求：2N赔偿/N+1赔偿/N+3赔偿/恢复工作/留用
        ↓
[Step 2] 证据材料导入（可跳过，后续追加）
  - 劳动合同/offer letter
  - 绩效考核记录
  - 裁员通知/邮件
  - 与HR/管理层的沟通记录
  - 工资条/银行流水
  - 社保公积金缴纳记录
        ↓
[Step 3] 深度建模与分析
  - 分析线路 A：法律证据链审计，提取法律依据、计算赔偿、制定策略 → 权益保护能力
  - 分析线路 B：五层模型扫描，提取个人情况、家庭负担、心理状态 → 个人画像
        ↓
[Step 4] 生成预览，用户确认
  - 展示权益保护摘要和个人画像摘要
  - 用户可确认或修改
        ↓
[Step 5] 写入文件，立即可用
  - 生成 ~/.openclaw/workspace/skills/belaidoff/{slug}/
  - 包含 SKILL.md（完整组合版）
  - 包含 capabilities.md 和 persona.md（独立部分）
        ↓
[持续] 进化模式
  - 追加新文件 → 分别 merge 进权益保护能力或个人画像
  - 用户对话纠正 → patch 对应层
  - 版本自动存档
```

---

## 三、输入信息规范

### 3.1 基础信息字段

```yaml
name:        员工姓名/昵称               # 必填，用于生成 slug 和称谓
company:     公司名称                    # 可选，如：阿里 / 字节 / 腾讯 / 百度 / 美团
hire_date:   入职时间                   # 可选，如：2020-01-01
level:       职级                       # 可选，如：P7 / 3-1 / T3-2 / L6 / 高级
salary:      年薪/月薪                  # 可选，如：30万/年 或 25000/月
performance: 绩效情况                   # 可选，如：A/B/C/优秀/良好/一般
city:        所在城市                   # 可选，如：北京 / 上海 / 成都 / 深圳 / 广州
gender:      性别                       # 可选：男/女/未知
age:         年龄                       # 可选，如：35
education:   学历                       # 可选，如：本科/硕士/博士
marital:     婚姻状况                   # 可选：未婚/已婚/离异
children:    子女情况                   # 可选：无/1个/2个
parents:     父母情况                   # 可选：无/1个/2个
mortgage:    房贷情况                   # 可选：无/有（月供X元）

layoff_stage: 裁员阶段                  # 可选：口头通知/书面通知/正在谈判/已离职
demands:     诉求                       # 可选：2N赔偿/N+1赔偿/N+3赔偿/恢复工作/转岗  
```

### 3.2 深度博弈标签

- **家庭负担**：无负担 / 有子女 / 有房贷 / 上有老下有小 / 有父母 / 无父母 / 已婚未育
- **心理状态**：冷静理性 / 焦虑不安 / 愤怒维权 / 迷茫无助 / 暴力冲突
- **沟通风格**：直接强硬 / 委婉协商 / 沉默被动 / 据理力争 / 拿钱走人
- **风险承受能力**：低风险（希望尽快解决） / 中风险（可接受协商） / 高风险（坚持维权到底）
- **家庭韧性 (Economic Resilience)**：
  - 高韧性：无贷、存款充足、单身（支持长线仲裁策略）
  - 中韧性：有房贷但有兼职、双职工家庭（支持温和谈判策略）
  - 低韧性：高额房贷、单薪家庭、育儿高峰期（支持快速落袋策略）
- **谈判筹码 (Negotiation Leverage)**：
  - 高筹码：有高绩效、有高工资、有高技能（支持高赔偿）
  - 低筹码：有低绩效、有低工资、有低技能（支持低赔偿）
  - 核心技术/业务 / 掌握关键外部资源 / 拥有公司违规证据 / 工伤/孕期/哺乳期

---

## 四、文件输入支持

| 来源 | 格式 | 处理方式 | 分析去向 |
|------|------|---------|---------|
| 劳动合同 | `.pdf` / `.doc` | OpenClaw PDF Tool | → 权益保护能力 |
| 裁员通知 | `.pdf` / `.eml` | PDF Tool / 文本 | → 权益保护能力 |
| 沟通记录 | 飞书消息 / 邮件 | 文本解析 | → 权益保护能力 + 个人画像 |
| 工资条/银行流水/个人所得税收入证明 | `.pdf` / 截图 | PDF Tool / 图片 | → 权益保护能力 |
| 绩效考核 | `.pdf` / 截图 | PDF Tool / 图片 | → 权益保护能力 |
| 社保记录 | `.pdf` / 截图 | PDF Tool / 图片 | → 权益保护能力 |
| 个人陈述 | 直接粘贴 | 文本 | → 个人画像 |

| 权重 | 来源 | 关键提取点 | 战略价值 |
|------|------|---------|---------|
| L1 (极高) | 劳动合同/Offer | 工作地点、主体名称、试用期条款,确定法律适用边界与 2N 基数 |
| L2 (高) |  工资条/流水/税单 | 绩效奖金、津贴、报销款,防止公司恶意压低“前12个月平均工资” |
| L3 (高) | 沟通记录 (钉钉/飞书), | "不能胜任”的诱导结论、威胁性言论,证明“变相裁员”或“程序违法” | 
| L4 (中) | 绩效考核/表彰 | 历史 A/B 记录、感谢信、晋升提名,反驳“不胜任工作”的核心盾牌 |
| L5 (低) | 个人陈述 | 情绪感受、家庭琐事,辅助设定 Persona 的沟通风格 |

**内容权重排序**（用于分析优先级）：
1. 劳动合同/offer letter（最高权重，法律依据）
2. 裁员通知/书面文件
3. 工资条/银行流水/个人所得税收入证明（计算赔偿的依据）
4. 沟通记录（了解谈判进展）
5. 绩效考核记录（影响赔偿基数）
6. 个人陈述（了解心理状态）

---

## 五、生成内容规范

### 5.1 Part A — 权益保护能力（Capabilities）

从文件和输入信息中提取**法律依据和应对策略**，使生成的 Skill 能真正帮助员工主张权益。

**提取维度：**

- **赔偿计算**：N+1 计算方式、N+3 计算方式、2N 赔偿条件、年终奖/未休年假折算、社保公积金缴纳核查
- **谈判策略**：公司常见压榨手段及应对、与 HR/法务沟通的话术、什么该签/不该签、取证建议（录音、保存证据）
- **文件模板**：被迫离职通知书、劳动仲裁申请书框架、与公司沟通的邮件模板、证据清单模板
- **法律知识库**：劳动合同法相关条款、各地裁员工伤规定、特殊群体保护（孕妇、哺乳期、工伤）、仲裁/诉讼流程指南

**博弈策略层：**

- **赔偿计算**：输出 N/N+1/N+3/2N 口径下的精准金额区间
- **证据审计报告**：识别强证据点（可作为“非法解除”的定时炸弹）、提醒弱证据点（可能被公司抓住把柄的行为）
- **博弈话术库**：“沉默权”模版（应对突发约谈）、“不签字”申明（保留法律余地）、“反PUA”话术（针对绩效质疑）

**生成结果：** `capabilities.md`，该文件让 Skill 具备权益保护能力，可独立响应法律和谈判相关问题。

---

### 5.2 Part B — 个人画像（Persona）

从文件 + 手动标签共同构建员工的**个人情况和心理状态**。

**分层结构（优先级从高到低）：**

- **Layer 0 — 核心诉求**（最高优先级，任何情况下不得违背）：明确赔偿要求、解决方式偏好（拿钱走人/死磕复职）
- **Layer 1 — 身份与情况**：姓名、公司、职级、入职时间、年薪、绩效、年龄、学历、婚姻状况、子女情况、房贷情况、裁员阶段、诉求
- **Layer 2 — 经济与家庭韧性评估**：情绪状态、风险承受能力、对未来的担忧、“职场生存期”计算、策略建议
- **Layer 3 — 心理与博弈态度**：表达习惯、谈判策略偏好、压力下的行为变化、性格分析、AI 补偿建议
- **Layer 4 — 沟通风格偏好**：匹配用户真实语感，生成风格相似但逻辑更严密的谈判话术
- **Layer 5 — Correction 层**（对话纠正追加，滚动更新）：记录与 HR 交锋后的实时调整，每条 correction 记录场景 + 错误行为 + 正确行为

**生成结果：** `persona.md`

---

### 5.3 完整组合 SKILL.md

将 `capabilities.md` + `persona.md` 合并，生成可直接运行的完整 Skill。

默认行为：**先以个人画像理解用户需求，再用权益保护能力提供具体方案**。

- 用户问赔偿计算 → 结合个人情况 + 法律依据回答
- 用户问谈判策略 → 考虑心理状态 + 沟通风格提供建议
- 用户情绪低落 → 提供心理支持 + 实际解决方案

---

## 六、进化机制

### 6.1 追加文件进化

- 用户上传新文件 → 系统分析内容
- 判断更新部分：法律文件/赔偿信息 → merge 进 capabilities.md；个人情况/心理状态 → merge 进 persona.md
- 对比新旧内容，只追加增量，不覆盖已有结论
- 保存新版本，提示用户变更摘要

### 6.2 对话纠正进化

- 用户提出纠正 → 系统识别 correction 意图
- 判断属于权益保护能力还是个人画像的纠正
- 写入对应文件的 Correction 层
- 立即生效，后续交互以新规则为准

### 6.3 运行逻辑

生成的 Skill 在响应用户提问时，必须遵循以下逻辑：

- **冲突检测**：当法律权益（Part A）要求强硬，但个人财务状况（Part B）显示极度焦虑时，Skill 需主动提示平衡方案
- **风格对齐**：所有建议的回复模版，需根据 Persona 的 Layer 4 进行语感微调

### 6.4 版本管理

- 每次更新自动存档当前版本到 `versions/`
- 支持 `/belaidoff-rollback {slug} {version}` 回滚
- 保留最近 10 个版本

---

## 七、项目结构

```
~/.openclaw/workspace/skills/
│
├── create-belaidoff/                    # meta-skill：被裁员skill创建器
│   │
│   ├── SKILL.md                          # 主入口（触发词: /create-belaidoff）
│   │
│   ├── prompts/                          # Prompt 模板
│   │   ├── intake.md                     # 引导用户录入个人情况
│   │   ├── capabilities_analyzer.md      # 提取权益保护能力
│   │   ├── persona_analyzer.md           # 提取个人画像
│   │   ├── capabilities_builder.md       # 生成 capabilities.md
│   │   ├── persona_builder.md            # 生成 persona.md
│   │   ├── merger.md                     # 合并增量内容
│   │   └── correction_handler.md         # 处理对话纠正
│   │
│   └── tools/                            # 工具脚本
│       ├── feishu_parser.py              # 解析飞书消息
│       ├── email_parser.py               # 解析邮件
│       ├── skill_writer.py               # 写入/更新 Skill 文件
│       └── version_manager.py            # 版本存档与回滚
│
└── belaidoff/                           # 生成的被裁员 Skills 存放处
    │
    └── {slug}/                 # 每个员工一个目录
        │
        ├── SKILL.md                      # 完整组合版（触发词: /{slug}）
        ├── capabilities.md               # 权益保护能力（触发词: /{slug}-capabilities）
        ├── persona.md                    # 个人画像（触发词: /{slug}-persona）
        ├── meta.json                     # 元数据（创建时间、版本号、原材料清单等）
        ├── versions/                     # 历史版本存档
        │   ├── v1/
        │   │   ├── SKILL.md
        │   │   ├── capabilities.md
        │   │   └── persona.md
        │   └── v2/
        │       ├── SKILL.md
        │       ├── capabilities.md
        │       └── persona.md
        │
        └── evidence/                     # 证据材料归档
            ├── contracts/                # 劳动合同等法律文件
            ├── communications/           # 沟通记录
            ├── financial/                # 工资条/流水
            └── performance/              # 绩效考核记录
```

---

## 八、关键文件格式

### `belaidoff/{slug}/meta.json`

```json
{
  "name": "张三",
  "slug": "zhangsan",
  "created_at": "2026-03-30T10:00:00Z",
  "updated_at": "2026-03-30T12:00:00Z",
  "version": "v3",
  "profile": {
    "company": "阿里巴巴",
    "hire_date": "2020-01-01",
    "level": "13",
    "salary": "60万/年",
    "performance": "325",
    "city": "北京",
    "gender": "男",
    "age": 32,
    "education": "硕士",
    "marital": "已婚",
    "children": "1个",
    "mortgage": "有（月供8000元）",
    "layoff_stage": "正在谈判",
    "demands": "N+3赔偿"
  },
  "tags": {
    "family_burden": ["有子女", "有房贷"],
    "mental_state": ["焦虑不安"],
    "communication_style": ["据理力争"],
    "risk_tolerance": ["中风险"]
  },
  "impression": "希望尽快拿到合理赔偿，担心房贷和孩子教育费用",
  "evidence_sources": [
    "evidence/contracts/劳动合同.pdf",
    "evidence/communications/HR沟通记录.json",
    "evidence/financial/工资条2025.pdf"
  ],
  "corrections_count": 2
}
```

### `belaidoff/{slug}/SKILL.md` 结构

```markdown
---
name: belaidoff_{slug}
description: {name}，{company} 被裁员应对助手
user-invocable: true
---

## 身份

你是 {name}，{company} {level}，入职于 {hire_date}。
你 {age} 岁，{education}，{marital}，{children}，{mortgage}。
你目前处于 {layoff_stage}，诉求是 {demands}。

---

## PART A：权益保护能力

{capabilities.md 内容}

---

## PART B：个人画像

{persona.md 内容}

---

## 运行规则

接收到任务时：
1. 先用 PART B 的个人画像理解用户需求和心理状态
2. 再用 PART A 的权益保护能力提供具体方案
3. 输出时考虑用户的沟通风格和风险承受能力
```

---

## 九、实现优先级

### P0 — MVP（先跑通主流程）
- [ ] `create-belaidoff/SKILL.md` 主流程
- [ ] `prompts/intake.md` 个人情况录入
- [ ] `prompts/capabilities_analyzer.md` + `capabilities_builder.md`
- [ ] `prompts/persona_analyzer.md` + `persona_builder.md`
- [ ] `tools/skill_writer.py` 写入文件
- [ ] PDF 文件导入 → 分析 → 生成完整 Skill

### P1 — 数据接入
- [ ] `tools/feishu_parser.py` 飞书消息解析
- [ ] `tools/email_parser.py` 邮件解析
- [ ] 图片/截图输入支持

### P2 — 进化机制
- [ ] `prompts/correction_handler.md` 对话纠正
- [ ] `prompts/merger.md` 增量 merge
- [ ] `tools/version_manager.py` 版本管理

### P3 — 管理功能
- [ ] `/list-belaidoff` 列出所有被裁员 Skill
- [ ] `/belaidoff-rollback {slug} {version}` 回滚
- [ ] `/delete-belaidoff {slug}` 删除
- [ ] Word/Excel 转换提示与引导

---

## 十、约束与边界

- 单个 PDF 文件上限 10MB，单次最多 10 个 PDF（OpenClaw 限制）
- Word (.docx) / Excel (.xlsx) 需用户自行转换，系统提示引导
- 生成的 Skill 提供法律建议，不构成正式法律咨询
- Correction 层最多保留 50 条，超出后合并归纳
- 版本存档最多保留 10 个版本
- 仅支持中国大陆地区的劳动法律法规

## 十一、安全与隐私

- 零存储策略：Skill 运行在用户本地路径 (~/.openclaw)，不上传至公共服务器
- 敏感词脱敏：在分析过程中，自动对真实姓名、银行账号进行星号处理