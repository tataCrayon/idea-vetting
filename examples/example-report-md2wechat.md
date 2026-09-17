# 🔍 想法体检报告：自研 Markdown 转微信公众号排版工具，把团队技术文档一键发到公众号

## 1. 想法拆解（FRAME）

把它压成 4 条可证伪主张——想法要成立，以下必须为真：

- **C1（排版未被解决）**：现有工具不能满足"技术文档（代码块/图片/表格）→ 公众号 HTML"的排版需求，需要自研。
- **C2（一键发布可行）**：团队的公众号账号类型与凭证允许通过官方服务端 API（草稿箱/发布接口）自动发文。
- **C3（集成成本更低）**：把团队文档源（仓库里的 Markdown）接入现成工具的自动化成本，低于自研整条链路。
- **C4（维护成本可控）**：自研排版引擎对微信编辑器样式规则变化的长期维护成本可接受。

**尚未验证的前提**：你们账号是订阅号还是服务号、是否认证；文档源格式是否规整；"一键"是指人点一下按钮还是 CI 全自动。这些影响 C2/C3，但不影响 C1 的判断（下面有证据）。

## 2. 已有实践（RECON）

以下为**实际检索到的事实**（star 数与最近推送时间来自 GitHub API，检索日 2026-09-17）：

| 谁在做 | 形态 | 状态 | 证据 |
|---|---|---|---|
| doocs/md「微信 Markdown 编辑器」：Markdown 即时渲染为微信图文，自定义主题、代码高亮、多图床、Chrome/Firefox 扩展、CLI、Docker 自托管 | 开源（13,335★，最近推送 2026-09-16，活跃） | 活跃 | https://github.com/doocs/md 、在线版 https://md.doocs.org |
| lyricat/wechat-format：Markdown→微信特制 HTML，README 明确处理"ul/ol 样式被微信编辑器重置"等坑 | 开源（4,550★，最近推送 2025-09-13） | README 自述"已停止维护" | https://github.com/lyricat/wechat-format |
| geekjourneyx/md2wechat-skill：CLI 一键排版**并发布**到公众号，40+ 主题、批量发布、多账号管理；API 模式需 md2wechat API Key，涉及微信 IP 白名单 | 开源 + 商业 API（3,651★，最近推送 2026-09-12） | 活跃 | https://github.com/geekjourneyx/md2wechat-skill |
| isjiamu/gzh-design-skill：AI-agent 技能，Markdown→可直接粘进公众号编辑器的 HTML，主题生成器+双关卡校验 | 开源（3,710★，最近推送 2026-07-08） | 活跃 | https://github.com/isjiamu/gzh-design-skill |
| caol64/wenyan-mcp：MCP Server，让 AI 自动把 Markdown 排版后发布至微信公众号 | 开源（1,314★，最近推送 2026-04-29） | 活跃 | https://github.com/caol64/wenyan-mcp |
| xiaohuailabs/xiaohu-wechat-format：Claude Code 技能，Markdown→微信兼容 HTML→推送草稿箱，30 套主题 | 开源（693★，最近推送 2026-06-12） | 活跃 | https://github.com/xiaohuailabs/xiaohu-wechat-format |
| 秀米 xiumi.us：公众号排版 + H5 商业编辑器，有 VIP/企业会员付费体系 | 商业 | 活跃 | https://xiumi.us |
| 社区实践信号：doocs/md 的 issue/PR 显示用户在要"转换 API、接 n8n 自动化、发到草稿箱" | 社区讨论 | — | https://github.com/doocs/md/issues/716 、 https://github.com/doocs/md/issues/1325 、 https://github.com/doocs/md/pull/1016 |

**平台侧关键事实**：微信官方"新增草稿"接口文档（/cgi-bin/draft/add）写明：接口须在服务器端调用、使用 access_token，且"本接口在不同账号类型下的可调用情况：公众号 ✔、服务号 ✔"。出处： https://developers.weixin.qq.com/doc/subscription/api/draftbox/draftmanage/api_draft_add.html

- **成熟度定位：主流成熟**——依据：同一问题存在多个 1k~13k★ 的活跃实现（doocs/md、md2wechat、wenyan-mcp、gzh-design-skill），功能点高度收敛（主题、代码高亮、图床、草稿箱发布），且已出现停止维护的先行者（wechat-format），说明赛道已充分开垦。按本 Skill 规则，主流成熟时**默认答案是"采纳，不是发明"**。
- **检索盲区声明**：Hacker News 检索 "wechat markdown"/"公众号" 返回 0 条（英文社区讨论未检索到，不代表不存在）；V2EX/知乎/Reddit 及 135editor 官网因网络超时未能核实；个人主体订阅号在草稿 API 上的具体限制未从官方页面直接确认（上引页面仅标注"公众号/服务号"两列）。

## 3. 理论与最佳实践（THEORY）

- **支持（支持的是"自动化发布"这半截，不是"自研排版"）**：Docs-as-Products / 单一事实源是技术团队通行实践——文档留在 Git 仓库的 Markdown 里，发布端做自动化转换，这与"仓库→公众号"的想法方向一致。
- **反对/警示（针对"自研排版引擎"）**：
  - 微信编辑器会重置部分 HTML 样式（wechat-format README 点名 ul/ol 被重置的问题，见上表），排版兼容性是**追着平台跑的苦役**，自研意味着无限期承接微信前端行为变化——这是已知反模式：与平台私有渲染规则长期对赌。
  - lyricat/wechat-format 这个 4.5k★ 先行者已停维（事实），说明"纯排版转换库"没有护城河，连维护者都撤了。
  - 我的推理：doocs/md issue #716/#1325 里用户要 API 化、接 n8n 而项目未直接满足（检索到的事实），说明"现成工具 + 团队专属流水线"之间确实有一小段缝隙——但那是**集成层的活，不是排版引擎的活**。

## 4. 压力测试（STRESS）

1. **"自研的核心价值是什么？"——排版不是，你真正缺的是"一键"。** 最强反方案：doocs/md 自托管 + md2wechat/wenyan-mcp 走草稿 API，排版和发布全有现成的。自研排版引擎在这个方案里没有任何一格是必需的。若你答不出"现成工具具体哪一步卡住了你们"，那就是还没开始用现成的。
2. **"团队文档有一键发布需求"这个前提最可能在哪塌？——账号与凭证。** 草稿/发布 API 需服务器端调用、access_token、IP 白名单（官方文档，事实）；若你们是个人主体订阅号或无认证，C2 直接为假，"一键"退化为"工具排版 + 人工粘贴"，而 doocs/md 在线版零成本就能覆盖。
3. **有没有人试过然后撤了？——有。** wechat-format 停维是检索到的事实；它当年解决的"样式被重置"问题今天仍会被微信以新花样重演。自研者大概率在 6 个月后变成"专职修样式的"，这是他们当时没料到的坑（此句为我的推理）。

## 5. 判决（VERDICT）

- **🔧 改造**——"团队技术文档一键发公众号"方向成立且值得做，但**放弃自研排版引擎**：排版采纳 doocs/md（主流成熟），"一键"用现成发布通道（md2wechat CLI 或 wenyan-mcp 推草稿箱），你们只自研最薄的一层"团队文档源 → 工具输入"的胶水脚本（封面、作者、术语表、批量迁移这类团队定制）。
- **最值钱的一步**：今天就拿一篇最复杂的存量文档（含代码块、图片、表格），走 doocs/md 在线版转一次、粘进公众号草稿箱，看还原度——这一步同时检验 C1 和你们真实需求的差距。
- **最便宜的验证（≤1 天）**：在拿到公众号 AppID/Secret 后，用 https://github.com/caol64/wenyan-mcp 或 https://github.com/geekjourneyx/md2wechat-skill 把同一篇文档**自动推到草稿箱**，验证 C2（账号权限与 API 通路）与 C3（集成成本）。时间预算：半天。
- **放弃信号**：① 现成工具还原度 ≥95% 且草稿 API 通路跑通 → 彻底放弃自研排版，只留胶水脚本；② 账号类型不支持草稿/发布 API 且无法调整 → 放弃"一键"，降级为"doocs/md 排版 + 人工粘贴"；③ 若发现必须深度魔改微信 HTML 才能过审/过样式，先给 doocs/md 提 issue/PR（它活跃，见上表），而不是自己 fork 养一套。
