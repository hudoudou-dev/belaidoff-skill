# 被裁员.skill —— 产品需求文档 v1.0

---

## 一、产品概述

**被裁员.skill** 是一个运行在 OpenClaw 上的 meta-skill，专为面临裁员的员工提供权益保护和心理支持。

用户通过对话式交互提供个人情况和相关证据，系统自动生成一个可独立运行的**被裁员应对 Skill**，帮助员工：
- 计算应得赔偿
- 制定谈判策略
- 提供心理疏导
- 准备法律文件

生成的 Skill 由两个独立部分组成：
- **Part A — 权益保护能力**：劳动法知识、赔偿计算、谈判策略、文件模板
- **Part B — 个人画像**：员工个人情况、家庭负担、心理状态、沟通风格

两部分可以独立使用，也可以组合运行（默认组合）。生成后的 Skill 支持通过追加文件或对话纠正持续进化。

---

## 二、用户流程

```
用户触发 /create-belaidoff
        ↓
[Step 1] 个人情况录入（全部可跳过）
  - 姓名/昵称
  - 基本信息：公司、所在城市、入职时间、职级、年薪、绩效情况
  - 个人状态：年龄、学历、婚姻状况、子女情况、父母情况、房贷情况
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
[Step 3] 自动分析
  - 分析线路 A：提取法律依据、计算赔偿、制定策略 → 权益保护能力
  - 分析线路 B：提取个人情况、家庭负担、心理状态 → 个人画像
        ↓
[Step 4] 生成预览，用户确认
  - 分别展示权益保护摘要和个人画像摘要
  - 用户可直接确认或修改
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

age:         年龄                       # 可选，如：35
education:   学历                       # 可选，如：本科/硕士/博士
marital:     婚姻状况                   # 可选：未婚/已婚/离异
children:    子女情况                   # 可选：无/1个/2个
parents:     父母情况                   # 可选：无/1个/2个
mortgage:    房贷情况                   # 可选：无/有（月供X元）

layoff_stage: 裁员阶段                  # 可选：口头通知/书面通知/正在谈判/已离职
demands:     诉求                       # 可选：2N赔偿/N+1赔偿/N+3赔偿/恢复工作/转岗  
```

### 3.2 个人状态标签

**家庭负担**
- `无负担` / `有子女` / `有房贷` / `上有老下有小` / `有父母` / `无父母` / `已婚未育`

**心理状态**
- `冷静理性` / `焦虑不安` / `愤怒维权` / `迷茫无助` / `暴力冲突`

**沟通风格**
- `直接强硬` / `委婉协商` / `沉默被动` / `据理力争` / `拿钱走人`

**风险承受能力**
- `低风险`（希望尽快解决） / `中风险`（可接受协商） / `高风险`（坚持维权到底）

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

```
① 赔偿计算
   - N+1 计算方式
   - N+3 计算方式
   - 2N 赔偿条件
   - 年终奖/未休年假折算
   - 社保公积金缴纳核查

② 谈判策略
   - 公司常见压榨手段及应对
   - 与 HR/法务沟通的话术
   - 什么该签、什么不该签
   - 取证建议（录音、保存证据）

③ 文件模板
   - 被迫离职通知书
   - 劳动仲裁申请书框架
   - 与公司沟通的邮件模板
   - 证据清单模板

④ 法律知识库
   - 劳动合同法相关条款
   - 各地裁员工伤规定
   - 特殊群体保护（孕妇、哺乳期、工伤）
   - 仲裁/诉讼流程指南
```

**生成结果：** `capabilities.md`，该文件让 Skill 具备权益保护能力，可独立响应法律和谈判相关问题。

---

### 5.2 Part B — 个人画像（Persona）

从文件 + 手动标签共同构建员工的**个人情况和心理状态**。

**分层结构（优先级从高到低）：**

```
Layer 0 — 核心诉求（最高优先级，任何情况下不得违背）
  示例："你坚决要求 2N 赔偿，不会接受 N+1"
  示例："你希望尽快解决，愿意在赔偿金额上做出一定让步"

Layer 1 — 身份与情况
  "你是 [姓名]，[公司] [职级]，入职于 [入职时间]。"
  "你的年薪是 [X]，绩效 [Y]。"
  "你 [年龄] 岁，[学历]，[婚姻状况]，[子女情况]，[房贷情况]。"
  "你目前处于 [裁员阶段]，诉求是 [诉求]。"

Layer 2 — 心理状态
  - 情绪状态（焦虑/愤怒/冷静/迷茫）
  - 风险承受能力（低/中/高）
  - 对未来的担忧（经济压力/职业发展/家庭影响）

Layer 3 — 沟通风格
  - 表达习惯（直接/委婉/沉默）
  - 谈判策略偏好（强硬/协商/被动）
  - 压力下的行为变化

Layer 4 — 家庭负担
  - 经济压力程度
  - 家庭责任（子女教育/老人赡养/房贷）
  - 对解决时间的期望

Layer 5 — Correction 层（对话纠正追加，滚动更新）
  - 每条 correction 记录场景 + 错误行为 + 正确行为
  - 示例："[场景：HR 威胁] 不应该害怕，应该明确表示会依法维权"
```

**生成结果：** `persona.md`

---

### 5.3 完整组合 SKILL.md

将 `capabilities.md` + `persona.md` 合并，生成可直接运行的完整 Skill。

默认行为：**先以个人画像理解用户需求，再用权益保护能力提供具体方案**。

```
用户问赔偿计算 → 结合个人情况 + 法律依据回答
用户问谈判策略 → 考虑心理状态 + 沟通风格提供建议
用户情绪低落 → 提供心理支持 + 实际解决方案
```

---

## 六、进化机制

### 6.1 追加文件进化

```
用户: 我又收到了公司的新通知 @附件
        ↓
系统分析新内容
        ↓
判断新内容更新哪个部分：
  - 包含法律文件/赔偿信息 → merge 进 capabilities.md
  - 包含个人情况/心理状态 → merge 进 persona.md
  - 两者都有 → 分别 merge
        ↓
对比新旧内容，只追加增量，不覆盖已有结论
        ↓
保存新版本，提示用户变更摘要
```

### 6.2 对话纠正进化

```
用户: "我其实更希望尽快解决，不想走法律途径"
用户: "我有房贷压力，需要更快拿到赔偿"
用户: "HR 说要扣我年终奖，这合法吗？"
        ↓
系统识别 correction 意图
        ↓
判断属于权益保护能力还是个人画像的纠正
        ↓
写入对应文件的 Correction 层
        ↓
立即生效，后续交互以新规则为准
```

### 6.3 版本管理

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
│   ├── SKILL.md                          # 主入口
│   │                                     # 触发词: /create-belaidoff
│   │                                     # 描述: 创建一个被裁员应对 Skill
│   │
│   ├── prompts/                          # Prompt 模板（不执行，供 SKILL.md 引用）
│   │   ├── intake.md                     # 引导用户录入个人情况的对话脚本
│   │   ├── capabilities_analyzer.md      # 从原材料提取权益保护能力的 prompt
│   │   ├── persona_analyzer.md           # 从原材料提取个人画像的 prompt
│   │   ├── capabilities_builder.md       # 生成 capabilities.md 的模板
│   │   ├── persona_builder.md            # 生成 persona.md 的模板
│   │   ├── merger.md                     # 合并增量内容时使用的 prompt
│   │   └── correction_handler.md         # 处理对话纠正的 prompt
│   │
│   └── tools/                            # 工具脚本
│       ├── feishu_parser.py              # 解析飞书消息导出 JSON
│       ├── email_parser.py               # 解析 .eml 邮件
│       ├── skill_writer.py               # 写入/更新生成的 Skill 文件
│       └── version_manager.py            # 版本存档与回滚
│
└── belaidoff/                           # 生成的被裁员 Skills 存放处
    │
    └── {slug}/                 # 每个员工一个目录，slug = 姓名拼音或自定义
        │
        ├── SKILL.md                      # 完整组合版，可直接运行
        │                                 # 触发词: /{slug}
        │
        ├── capabilities.md               # Part A：权益保护能力（可独立运行）
        │                                 # 触发词: /{slug}-capabilities
        │
        ├── persona.md                    # Part B：个人画像（可独立运行）
        │                                 # 触发词: /{slug}-persona
        │
        ├── meta.json                     # 元数据
        │                                 # 包含：创建时间、版本号、原材料清单、
        │                                 #        个人情况、家庭负担、诉求
        │
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
- [ ] `tools/feishu_parser.py` 飞书消息 JSON 解析
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
- 生成的 Skill 提供的是法律建议，不构成正式法律咨询
- Correction 层最多保留 50 条，超出后合并归纳
- 版本存档最多保留 10 个版本
- 仅支持中国大陆地区的劳动法律法规
