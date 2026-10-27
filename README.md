# eve-online-access

面向 **EVE Online 中文攻略写作** 的 Agent Skill（[Agent Skills 规范](https://agentskills.io/specification)）。

解决的问题：EVE 的机制每 4–6 周调整一次，攻略材料最大的敌人不是"写得不好"，而是**数据过时、来源不明、风险提示缺失**——读者照着做过时的攻略会直接损船。本技能把一套经过长期实践检验的写法固化为可复用的规范。

## 能做什么

- 撰写四类文体：**机制手册 / 实操攻略 / 速查表 / 新人入门**（每类有可复制骨架）
- 校对既有文档：核数据时点、核术语、核来源、核风险提示
- 把英文/其他来源的资料翻译、整理、本地化为中文
- 为军团/联盟整理内部服务与流程文档
- **对齐玩家实际表述**：开篇模式、第二人称、条件句、风险句式（取自 58 篇实战教程的统计）
- **俗称/黑话换算**：把「YST」「泡泡」「反跳」等口语换回官方名，标题可检索、正文像人话

## 目录结构

```
eve-online-access/
├── SKILL.md                      # 主技能：何时用、8 条铁律、工作流、玩家表述、常见错误
├── README.md                     # 安装、使用与贡献指南
├── LICENSE                       # MIT
└── references/
    ├── 结构模板.md                # 四类文体的可复制骨架 + 检查点
    ├── 玩家教程实证.md            # 58 篇实战教程的写法统计与改写示例
    ├── 黑话速查.md                # 俗称 → 官方名（舰级/舰船/模块/行为/数值/区域）
    ├── 官方数据源 - SDE 与 ESI.md  # 查官方中文译名 + 反查术语的工作流（含脚本）
    ├── 写作规范.md                # 共同特点：内容取向、格式规范、语气范式
    ├── 资料来源.md                # source 优先级、公开来源清单、核对与冲突处理
    └── 术语表.md                  # 中英术语、社区黑话、使用注意
```

## 安装

把整个目录放到 Agent Skill 的发现位置。以 [Pi](https://github.com/badlogic/pi-mono) 为例：

```bash
# 全局可用（用户级）
ln -s "$PWD/eve-online-access" ~/.pi/agent/skills/eve-online-access

# 或仅在某个项目内可用（项目级）
mkdir -p /path/to/project/.agents/skills
ln -s "$PWD/eve-online-access" /path/to/project/.agents/skills/eve-online-access
```

> 项目级 skill 受 **project trust** 保护：首次使用需批准信任（交互式会提示；非交互加 `--approve`，或运行 `/trust` 保存决定），否则项目 skill 会被静默跳过。

其他实现（Claude Code 等）遵循同一规范，通常识别 `~/.claude/skills/` 或项目内 `.claude/skills/`。

## 怎么用

模型会在任务匹配时自动加载。强制加载：

```
/skill:eve-online-access 帮我写一篇关于行星工业的新人攻略
```

## 贡献

欢迎补充，尤其欢迎以下内容：

| 方向 | 说明 |
|------|------|
| **组织适配** | 各军团/联盟的入口类型不同（注册链接、认证平台、快递与回收服务）。建议**不要**直接改 `references/资料来源.md` 的占位符，而是在自己 fork 的版本中填入，或另建 `references/组织来源-<组织名>.md` |
| **文体扩展** | 目前覆盖四类文体。PvP 战后复盘、舰船配置集、教学视频脚本等尚未成文，欢迎补充到 `references/结构模板.md` |
| **术语补充** | `references/术语表.md` 的舰船昵称部分留白较多（社区昵称地域差异大），欢迎按“昵称 → 正式名 → 来源”补充。**具体物品名的官方译名不靠手写，靠查 SDE/ESI**——如发现文档中译名与官方不符，请直接修正并注明校验方式 |
| **来源维护** | 公开来源会失效（已知至少一个社区 Wiki 已下线）。发现失效请在 `references/资料来源.md` 标注失效日期，不要直接删除 |

提交前请自查：

- [ ] 未包含**个人可识别信息**（真实姓名、游戏角色名、邮箱、账号）
- [ ] 未包含**组织内部信息**（内部链接、返利政策、内部价格、频道名、集结点）
- [ ] 未包含**带参数的私有链接**（注册返利链接、带 token 的认证链接）
- [ ] 引用来源均可公开访问，或已标注为占位符
- [ ] 示例数值标注为"示例"，不冒充当前有效数据

## 脱敏说明

本仓库的内容来自实际写作实践，发布前已做脱敏：

- 个人标识（作者名、角色名、账号）→ 已移除
- 组织内部链接（认证平台、注册链接、快递与回收服务）→ 已替换为 `<占位符>`
- 内部价格与政策 → 已泛化为"示例数值"
- 内部频道与集结点 → 已替换为角色/用途描述

`SKILL.md` 与 `references/` 中的示例仍保留真实的方法论与结构，仅去除了可识别信息。

## 许可

[MIT](LICENSE) © 2026 FrozMN

## 相关资源

- [Agent Skills 规范](https://agentskills.io/specification)
- [Anthropic skills 示例集](https://github.com/anthropics/skills)
- [Pi skills 示例集](https://github.com/badlogic/pi-skills)
- EVE Online 官方：[中文站](https://www.eveonline.com/zh/) ｜ [帮助中心](https://support.eveonline.com/hc/zh-cn/)
- 社区：[EVE University Wiki](https://wiki.eveuniversity.org/) ｜ [EVE Ref 中文库](https://everef.net/zh/)
