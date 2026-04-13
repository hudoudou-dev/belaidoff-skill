<div align="center">

# 被裁员.skill

> *"当裁员来临时，你不是一个人在战斗。让 AI 成为你最有力的权益保护助手。"*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://python.org)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

<br>

你是否面临裁员不知如何主张权利？<br>
你是否担心被企业压榨不知如何谈判？<br>
你是否需要心理支持度过艰难时期？<br>
你是否需要法律知识保护合法权益？<br>
你是否需要减少信息不对称与企业沟通？<br>

让 **被裁员.skill** 成为你的权益保护助手，提供专业的法律建议和谈判策略！

**愿每一位被裁员的打工人都能被温柔以待。裁员不可怕，只是下一段人生的开始。**

与**[裁员.skill](https://github.com/hudoudou-dev/layoff-skill)**（by hudoudou-dev）互为镜像。

<br>

[数据来源](#支持的数据来源) · [安装](#安装) · [使用](#使用) · [效果示例](#效果示例) · [详细安装说明](INSTALL.md)

[**English**](README.md)

</div>

---

上传个人信息和相关裁员材料（工作邮件、飞书/钉钉聊天记录、合同等），生成**个性化的被裁员应对 AI Skill**，帮助你计算赔偿、制定谈判策略、提供心理支持、准备法律文件。

Created by [@hudoudou-dev](https://github.com/hudoudou-dev) | Powered by hudoudou-personal

## 支持的数据来源

> **说明**：本项目基于 **同事.skill** 项目修改，专为面临裁员的员工提供权益保护和心理支持。

> 目前为 alpha 测试版本，后续将支持更多数据源，敬请关注！

| 来源 | 消息记录 | 文档 / Wiki | 多维表格 | 备注 |
|------|:-------:|:-----------:|:-------:|------|
| 飞书（自动采集） | ✅ API | ✅ | ✅ | 输入姓名即可，全自动 |
| 钉钉（自动采集） | ⚠️ 浏览器 | ✅ | ✅ | 钉钉 API 不支持历史消息 |
| PDF | — | ✅ | — | 手动上传 |
| 图片 / 截图 | ✅ | — | — | 手动上传 |
| 飞书 JSON 导出 | ✅ | ✅ | — | 手动上传 |
| 邮件 `.eml` / `.mbox` | ✅ | — | — | 手动上传 |
| Markdown | ✅ | ✅ | — | 手动上传 |
| 直接粘贴文字 | ✅ | — | — | 手动输入 |

---

## 安装

### Claude Code

> **重要**：Claude Code 从 **git 仓库根目录** 的 `.claude/skills/` 查找 skill，请在正确位置执行。

```bash
# 安装到当前项目（在 git 仓库根目录执行）
mkdir -p .claude/skills
git clone https://github.com/hudoudou-dev/belaidoff-skill .claude/skills/create-belaidoff

# 或安装到全局（所有项目可用）
git clone https://github.com/hudoudou-dev/belaidoff-skill ~/.claude/skills/create-belaidoff
```

### OpenClaw

```bash
git clone https://github.com/hudoudou-dev/belaidoff-skill ~/.openclaw/workspace/skills/create-belaidoff   
```

### 依赖（可选）

```bash
pip3 install -r requirements.txt
```

> 飞书/钉钉自动采集需配置 App 凭证，详见 [INSTALL.md](INSTALL.md)

---

## 使用

在 Claude Code 中输入：

```
/create-belaidoff
```

按提示输入个人信息（姓名、工作基本信息、个人状态、裁员情况、个人状态标签），然后选择数据来源。所有字段均可跳过，仅凭描述也能生成。

完成后用 `/{slug}` 调用该被裁员应对 Skill。

### 管理命令

| 命令 | 说明 |
|------|------|
| `/list-belaidoff` | 列出所有被裁员应对 Skill |
| `/{slug}` | 调用完整 Skill（权益保护能力 + 个人画像） |
| `/{slug}-capabilities` | 仅权益保护能力 |
| `/{slug}-persona` | 仅个人画像 |
| `/belaidoff-rollback {slug} {version}` | 回滚到历史版本 |
| `/delete-belaidoff {slug}` | 删除 |

---

## 效果示例

> 输入：`张三 阿里巴巴 2020-01-01 13 60万/年 325 32 男 硕士 已婚 1个 有（月供8000元），正在谈判 N+3赔偿 有子女 有房贷 焦虑不安 据理力争 中风险`

**场景一：赔偿计算**

```
用户      ❯ 我能拿到多少赔偿？
被裁员.skill ❯ 根据你的情况，计算如下：
             - 工作年限：4年
             - 月平均工资：50000元
             - 经济补偿金：4 × 50000 = 200000元
             - 如公司违法解除劳动合同，赔偿金：2 × 50000 = 1000000元
             - 未提前30天通知，代通知金：50000元
             合计最高可达：350000元左右
```

**场景二：谈判策略**

```
用户      ❯ HR 说只能给 N+1，我该怎么谈判？
被裁员.skill ❯ 建议：
             1. 收集证据：劳动合同、工资条、社保记录、聊天记录等
             2. 强调自身贡献：列举为公司做出的业绩和成果
             3. 引用法律法规：根据《劳动合同法》第87条，违法解除需支付2N赔偿
             4. 保持冷静，理性沟通，避免情绪化
             5. 设定底线，做好劳动仲裁准备
```

**场景三：心理支持**

```
用户      ❯ 我很焦虑，不知道该怎么办
被裁员.skill ❯ 理解你的感受，建议：
             1. 深呼吸练习，缓解焦虑情绪
             2. 与家人朋友沟通，寻求情感支持
             3. 制定求职计划，转移注意力
             4. 回顾过往成就，肯定自身价值
             5. 保持积极心态，相信未来会更好
```

---

## 功能特性

### 生成的 Skill 结构

每个被裁员应对 Skill 由两部分组成，共同驱动输出：

| 部分 | 内容 |
|------|------|
| **Part A — 权益保护能力** | 负责赔偿计算、谈判策略、法律知识库、文件模板、心理支持 |
| **Part B — 个人画像** | 5 层结构：核心诉求 → 心理状态 → 家庭负担 → 沟通风格 → 风险承受能力 |

运行逻辑：`接到任务 → 个人画像判断需求和心理状态 → 权益保护能力提供具体方案 → 结合用户沟通风格和风险承受能力输出`

### 支持的标签

- **家庭负担**：有子女 · 有房贷 · 经济压力大 · 家庭支柱 · 无负担
- **心理状态**：焦虑不安 · 愤怒 · 抑郁 · 恐惧 · 冷静理性 · 乐观积极
- **沟通风格**：据理力争 · 温和协商 · 沉默寡言 · 咄咄逼人 · 有理有据
- **风险承受能力**：高风险 · 中风险 · 低风险 · 保守 · 激进

### 进化机制

- **追加文件**：自动分析增量，merge 进对应部分，不覆盖已有结论
- **对话纠正**：说「应该是 xxx」，写入 Correction 层，立即生效
- **版本管理**：每次更新自动存档，支持回滚到任意历史版本

---

## 项目结构

本项目遵循 [AgentSkills](https://agentskills.io) 开放标准，整个 repo 即为一个 skill 目录：

```
create-belaidoff/
├── SKILL.md              # skill 入口（官方 frontmatter）
├── prompts/              # Prompt 模板
│   ├── intake.md         #   对话式信息录入
│   ├── capabilities_analyzer.md # 权益保护能力提取
│   ├── capabilities_builder.md # 权益保护能力生成模板
│   ├── persona_analyzer.md # 个人画像提取
│   ├── persona_builder.md # 个人画像五层结构模板
│   ├── merger.md         # 增量 merge 逻辑
│   └── correction_handler.md # 对话纠正处理
├── tools/                # Python 工具
│   ├── feishu_auto_collector.py  # 飞书全自动采集
│   ├── feishu_browser.py         # 飞书浏览器方案
│   ├── feishu_mcp_client.py      # 飞书 MCP 方案
│   ├── dingtalk_auto_collector.py # 钉钉全自动采集
│   ├── email_parser.py           # 邮件解析
│   ├── skill_writer.py           # Skill 文件管理
│   └── version_manager.py        # 版本存档与回滚
├── belaidoff/           # 生成的被裁员应对 Skill（gitignored）
├── docs/PRD.md
├── requirements.txt
└── LICENSE
```

---

## 注意事项

- **原材料质量决定 Skill 质量**：聊天记录 + 合同文件 > 仅手动描述
- 建议优先收集：与 HR/Leader 的文字聊天记录、劳动合同、工资条、社保记录、赔偿口头沟通后整理的文字版等
- 飞书自动采集需将 App bot 加入相关群聊
- 目前为 demo 版本，如有 bug 请提 issue！
- 本项目提供的信息仅供参考，具体法律问题请咨询专业律师

---

## 免责声明

本项目旨在提供信息和建议，不构成法律意见。所有内容仅供参考，具体法律问题请咨询专业律师。使用本项目产生的任何后果，本项目不承担责任。

---

## 致谢

本项目架构灵感来源于：

- **[同事.skill](https://github.com/titanwings/colleague-skill)**（by titanwings）— 首创"把人蒸馏成 AI Skill"的双层架构

---

## Star History

<a href="https://www.star-history.com/?repos=hudoudou-dev%2Fbelaidoff-skill&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/image?repos=hudoudou-dev/belaidoff-skill&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/image?repos=hudoudou-dev/belaidoff-skill&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/image?repos=hudoudou-dev/belaidoff-skill&type=date&legend=top-left" />
 </picture>
</a>

---

<div align="center">

MIT License © [hudoudou-dev](https://github.com/hudoudou-dev)

</div>


