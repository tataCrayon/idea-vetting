# 🔍 想法体检报告：让 AI 自动读 CI 失败日志，直接生成修复 PR

## 1. 想法拆解（FRAME）

把"做一个 AI 自动修 CI 的机器人"压成 4 条可证伪主张——它要成立，以下必须为真：

- **C1 诊断可靠**：LLM 能从 CI 失败日志中准确定位根因（而不是被噪声日志误导）。
- **C2 失败可码修**：你们 CI 失败中"可通过代码补丁修复"的比例足够高，值得为其自动化。
- **C3 PR 质量可接受**：生成的修复 PR 达到"人类评审后愿意合并"的水准，而不是制造评审噪音。
- **C4 可安全集成**：给 agent 读日志 + 写代码 + 开 PR 的权限，不会引入新的攻击面。

**尚未验证的成立前提**：你们 CI 失败的实际类型分布（代码 bug vs flaky/环境/依赖/基础设施）；模型需要的上下文与工具权限；团队对 AI PR 的评审带宽。这三条都只能在你们自己的仓库里量出来，本报告给不出答案。

## 2. 已有实践（RECON）

| 谁在做 | 形态 | 状态 | 证据 |
|---|---|---|---|
| GitHub Copilot Autofix（CodeQL 告警→AI 修复建议，多文件，人一键接受） | 商业（GHAS） | 活跃，已 GA | https://github.blog/2024-03-20-found-means-fixed-introducing-code-scanning-autofix-powered-by-github-copilot-and-codeql/ |
| anthropics/claude-code-action（在 issue/PR 里 @claude 即可让它改代码开 PR，可被 CI 失败事件触发） | 开源 | 非常活跃（8892 stars，最近推送 2026-09-17） | https://api.github.com/repos/anthropics/claude-code-action |
| remorses/self-healing-ci（GitHub Action：CI 失败→Claude 修复→自动开 PR，与你的想法几乎一字不差） | 开源 | 停滞（2 stars，最后推送 2025-07-16） | https://api.github.com/repos/remorses/self-healing-ci 、README：https://raw.githubusercontent.com/remorses/self-healing-ci/main/README.md |
| Dagger 官方"Self-Healing Pipelines"架构指南（agent 循环：读文件/跑测试/改代码直到绿，产出 diff 供人接受） | 开源厂商实践 | 活跃 | https://dagger.io/blog/automate-your-ci-fixes-self-healing-pipelines-with-ai-agents |
| 一批同名小项目：X24LABS/STITCH-AGENT、adnanafik/ops-pilot、pablofelix/ci-autohealing、axle-blaze/agentic-ai-cicd-orchestrator 等（GitHub 搜索命中 16 个） | 开源 | 全部为 0-4 star 的个人实验 | https://api.github.com/search/repositories?q=fix+CI+failure+LLM+agent&sort=stars&order=desc |
| FlyCI Wingman（2024 年 Show HN 的"AI 自动修失败构建"商业产品） | 商业 | 疑似已死：博客域名 www.flyci.net 现已无法解析 | HN 检索：https://hn.algolia.com/api/v1/search?query=AI%20agent%20fix%20CI%20failures&tags=story |
| purplefish-ai/factory-factory（"Ratcheting mode"：盯 CI 失败自动修） | 开源 | 早期 | 同上 HN 检索结果 |
| METR 对"AI 修复 PR 真实可合并性"的评测 | 评测机构 | 活跃 | https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/ |
| Wiz Red Agent：Snowflake 公开仓库 CI 中的 GitHub Actions 注入事故（恶意 issue 文本→CI 内执行任意命令→外泄 Jira 凭据，存活 5 天） | 安全研究/失败案例 | 已公开，GitHub 对 Copilot 归因有异议 | https://www.wiz.io/blog/red-agent-snowflake-copilot-cicd-bug |

- **成熟度定位：早期采用**。依据（均为检索到的事实）：相邻场景已被大厂产品化（Copilot Autofix 已 GA），通用积木现成且极活跃（claude-code-action）；但"CI 失败→自动修复 PR"这一**直接场景**仍由 0-4 star 的个人项目主导，最早的商业尝试（FlyCI Wingman）已无踪迹，相关 Show HN 帖普遍只有 1-7 分、0 评论——从业者情绪温冷，最佳实践未收敛。
- **检索盲区声明**：未检索到"跑了一年以上的自动修复 PR 生产复盘/lessons learned"文章；未检索到中文社区实践；GitLab CI 方向只有一条 2 评论的 Ask HN，未检索到生产案例；GitHub 官方产品页直连 404/超时，对 Copilot Autofix 的描述以官方 blog 为准。以上均为"未检索到"，不等于"不存在"。

## 3. 理论与最佳实践（THEORY）

**支持（检索到的事实）**
- 能力曲线真实：SWE-bench 论文（https://export.arxiv.org/abs/2310.06770）中 2023 年最好模型仅解 1.96% 的真实 GitHub issue；2026 年厂商宣称 SWE-bench Verified 达 ~80%（https://www.minimax.io/news/minimax-m25 ，经 HN 检索命中）。"读代码库→产出通过测试的补丁"三年间从玩具变成可用。
- 窄域已被验证可行：Copilot Autofix 官方称对 CodeQL 告警"超过三分之二的漏洞修复几乎无需编辑"——前提是**告警类型窄、有 CodeQL 精确定位**。
- 架构共识（Dagger 指南）：不要"把日志扔给模型要补丁"，而是给 agent 一个受约束的工具循环（读文件、跑测试、重跑 lint、循环到绿），产出 diff 由人 accept。这与自动程序修复（APR）研究里"执行反馈提升补丁质量"的路线一致（如 RewardRepair，经 arXiv 检索命中：https://export.arxiv.org/api/query?search_query=all:%22program%20repair%22+AND+all:%22overfitting%22 ）。

**反对/警示（检索到的事实）**
- **补丁过拟合是 APR 领域的经典失败模式**：QuixBugs 基准研究（同上 arXiv 检索命中）发现通过测试的自动修复补丁中 53.3% 是过拟合补丁——"CI 变绿"≠"修对了"。这是对你这个想法最硬的理论反对。
- **通过测试 ≠ 会被合并**：METR 实测很多能通过 SWE-bench 的 PR，真实工程师根本不会合并（链接见 RECON 表）。C3 有直接的负面证据。
- **公开基准数字要打折**：SWE-bench 存在数据污染与题目缺陷（https://github.com/SWE-bench/SWE-bench/issues/465 ；https://arxiv.org/abs/2410.06992 ，均经 HN 检索命中），OpenAI 已公开表示不再用 SWE-bench Verified 衡量前沿编码能力（https://openai.com/index/why-we-no-longer-evaluate-swe-bench-verified/ ）。
- **安全反模式已被实证**：Wiz 事故证明"外部文本（issue 标题/日志）进入 CI 内 agent 上下文"是真实的注入攻击面；该 PR 还绕过了 GHAS 扫描。无论补丁是不是 AI 写的，你的设计里日志=不可信输入。
- **理论与实践的冲突记录**：厂商宣称的修复成功率（高）vs 独立评测的"可合并性差 + 过拟合率过半"（低）——决策应偏向独立评测一侧。

**我的推理（非检索结论）**：CI 失败中相当大比例是 flaky 测试、环境/凭据/依赖漂移、基础设施抖动，这些不是"读日志→改代码"能修的；这决定了该想法的收益上限，必须用你们自己的数据来测。

## 4. 压力测试（STRESS）

1. **最可能为假的是 C2**：如果你们 60% 的失败是 flaky/环境类，自动修复 PR 的天花板就很低。回答：这正是"最便宜的验证"要测的第一个数——失败分类分布。
2. **最强反方案是"根本不做/不重造"**：claude-code-action 已把"CI 失败事件→Claude 修复→开 PR"做成现成积木（self-healing-ci 就是它的一个薄封装）。自研只有在跨平台（GitLab/Bitbucket）、合规、成本或深度定制上有硬理由时才成立。
3. **人不会省下来，只会搬家**：修复工作从"人修"变成"人评审 AI PR"。若可合并率低（METR 证据支持此风险），你制造的是新的工作而不是消除工作。
4. **有人试过然后消失了吗**：FlyCI Wingman 2024 年高调发布、域名现已无法解析（检索到的事实）；它当时是否认为自己是例外，未检索到答案——但同方向大量低 star 项目集体停滞，是"想法易、留存难"的模式信号。

## 5. 判决（VERDICT）

- **🔧 改造**——方向成立且有现成积木，但别自研 agent、别追求全自动：把想法改造成"用 claude-code-action（或 GHAS 用户直接开 Copilot Autofix）+ 收窄到可码修失败类型 + 人审 PR + 按注入攻击面加固"。
- **最值钱的一步**：今天就挑一个非关键仓库，接一条"workflow 失败→触发 @claude 修复→开 draft PR"的流水线（现成 Action，半天配置量），让它跑一周真实失败。
- **最便宜的验证（≤1 天）**：拉你们仓库最近 30-50 条真实 CI 失败，人工分类为"代码 bug / flaky / 环境依赖 / 基础设施"四类（验证 C2）；再从中挑 10 条"代码 bug 类"跑上述流水线，统计根因诊断正确率与 PR 可合并率（验证 C1、C3）。
- **放弃信号**：① "可码修"失败占比 <30% → 放弃自动修复，降级为"AI 只做失败分诊与根因摘要"；② 生成 PR 的人工可合并率 <30%（评审者开始把它当噪音）→ 停手；③ 任何一次发现 agent 上下文里混入了 secrets 或日志内容影响了其行为 → 立即下线，先补安全边界（最小权限 token、无 secrets 环境、日志视为不可信输入）再谈重启。
