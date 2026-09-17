# idea-vetting

**Vet an idea before investing time or money into it.**

投资一个想法之前，先给它做体检：它成立的前提是什么？有没有人已经在做？有没有最佳实践或理论支持？——然后给出判决：**采纳 / 改造 / 先验证 / 放弃**。可作为 Claude Code / Codex / Cursor / ZCode 等的 Skill 使用。

> Before investing in an idea, force it into falsifiable claims, then go outside: search who is already doing it, what theory and practice say, and how others failed. Every external claim must carry a real source; every verdict must carry the cheapest validation and a kill signal. Refuse to end with "worth thinking about" — land on adopt / amend / validate / kill.
>
> 投资一个想法之前，先把它压成可证伪的主张，然后向外走：检索谁在做、理论与实践怎么说、别人怎么失败的。每条外部结论必须带真实出处；每个判决必须附最便宜的验证实验和放弃信号。禁止以"值得思考"收尾——必须落到 采纳 / 改造 / 先验证 / 放弃。

想法不值钱，验证才值钱。最贵的两种错误：**造一个已经存在的东西**（重复造轮子），和**造一个理论早就预言会失败的东西**。本 Skill 让 AI 在你投入之前跑一轮真实检索的体检——不是凭印象说"我觉得可以"，而是拿着带出处的证据给判决。

## 功能特性

- **5 步体检循环**：`FRAME`（压成可证伪主张）→ `RECON`（真实检索已有实践）→ `THEORY`（理论支点与反模式）→ `STRESS`（拷问想法）→ `VERDICT`（判决四选一）
- **向外走，不空想**：RECON 强制用 WebSearch/WebFetch 检索开源项目、商业产品、社区讨论三类来源，每条结论带真实 URL
- **失败报告优先**：检索模式内置"production experience / lessons learned / postmortem"——别人怎么失败的比成功故事值钱
- **成熟度定位四档**：前沿探索 / 早期采用 / 主流成熟 / 正在退场，每档必须给依据
- **判决必须落地**：四选一 + 最值钱的一步 + ≤1 天的最便宜验证 + 放弃信号，禁止"值得思考"收尾
- **反谄媚纪律**：判决允许且有时必须是"别做"或"别造，用现成的"；无证据的夸奖禁止出口

## 安装

### 方式一：skills.sh 一键安装

```bash
npx skills add tataCrayon/idea-vetting
```

### 方式二：手动安装（推荐软链接，仓库更新自动生效）

```bash
git clone https://github.com/tataCrayon/idea-vetting.git
cd idea-vetting

# Claude Code
mkdir -p ~/.claude/skills && ln -s "$(pwd)" ~/.claude/skills/idea-vetting

# Codex
mkdir -p ~/.codex/skills && ln -s "$(pwd)" ~/.codex/skills/idea-vetting

# ZCode / 其他遵循 ~/.agents/skills 约定的客户端
mkdir -p ~/.agents/skills && ln -s "$(pwd)" ~/.agents/skills/idea-vetting
```

Windows 用户请将 `ln -s` 替换为 `mklink /D`（管理员 CMD）或直接复制整个目录。

安装后**重启客户端或新开会话**，Skill 才会被检测到。需要宿主客户端具备联网检索能力（WebSearch/WebFetch）；没有检索能力时 Skill 会自动声明降级模式，结论标注"未经外部验证"。

## 使用

### 作为 Skill

在会话中直接说：

- *"这个想法靠谱吗？有没有人做过？"*
- *"帮我评估一下这个想法的可行性"*
- *"有没有最佳实践或理论支持？"*
- *"别让我重复造轮子——vet this idea"*

### 触发方式

| 方式 | 说明 |
|---|---|
| **硬触发** | 用户主动要求评估想法的可行性/成熟度/已有实践 |
| **软触发** | 用户提出想法且带不确定信号（"我在想要不要…"），且一旦投入就难回头 |
| **不触发** | 用户已决定只要执行、纯事实查询、微小可逆的想法 |

### 看一个真实例子

用户问："我在想要不要自己写一个 Markdown 转微信公众号排版的工具……这个想法靠谱吗？"完整体检报告见 [examples/example-report-md2wechat.md](examples/example-report-md2wechat.md)，判决部分长这样：

```
## 5. 判决（VERDICT）
- 🔧 改造——"团队技术文档一键发公众号"方向成立且值得做，但放弃自研排版引擎：
  排版采纳 doocs/md（主流成熟），"一键"用现成发布通道，只自研最薄的一层胶水脚本。
- 最值钱的一步：今天就拿一篇最复杂的存量文档走 doocs/md 在线版转一次，看还原度。
- 最便宜的验证（≤1 天）：拿到 AppID 后用 wenyan-mcp 把同一篇文档自动推到草稿箱。
- 放弃信号：现成工具还原度 ≥95% 且 API 通路跑通 → 彻底放弃自研排版，只留胶水脚本。
```

报告里每条"谁在做"都带真实 GitHub 链接和 star 数/维护状态，包括一个 4.5k★ 先行者已停维的事实——这就是"失败报告比成功故事值钱"。

## 与相近概念的区别

| | 审的对象 | 核心动作 |
|---|---|---|
| **idea-vetting** | **想法（要不要做/有没有人做过）** | **向外检索证据，给投资判决** |
| grill-me | 需求（做什么、边界在哪） | 五层追问，逼人说清边界 |
| grill-method | 方法（怎么做的路线选择） | 内部推理拷问已选方法 |
| doubt-driven-development | 具体决策/制品（做得对不对） | 新上下文审查者反驳 |

**四者接力顺序**：idea-vetting（要不要做）→ grill-me（做什么）→ grill-method（怎么做）→ DDD（做得对不对）。

## 常见问题（FAQ）

| 问题 | 解答 |
|---|---|
| 和直接问 AI"这个想法怎么样"有什么区别？ | 裸问 AI 会得到凭印象的恭维或泛泛而谈。本 Skill 强制真实检索（带 URL 出处）、强制可证伪主张、强制判决四选一 + 放弃信号。 |
| 会不会把体检做成文献综述？ | 内置检索预算：默认 ≤10 次检索，证据收敛即停。体检服务于决策。 |
| 搜不到相关内容怎么办？ | 证据纪律规定：搜不到写"未检索到"，不写"没人做过"——证据缺失不是缺失的证据。Skill 会换关键词/换英文再搜才下结论。 |
| 判决说"别做"，但我很想做怎么办？ | 判决附理由和放弃信号，最终决定权在你。你可以带着"我知道风险在哪"的清醒去做——这比盲目乐观好得多。 |
| 没有联网检索能力的客户端能用吗？ | 能，但 Skill 会声明降级模式（仅内部推理），结论标注"未经外部验证"。 |
| 支持哪些客户端？ | 任何支持 Agent Skills 机制的客户端（Claude Code / Codex / Cursor / ZCode 等）。纯 Markdown 指令，无脚本依赖。 |

## 注意事项

- 本 Skill 审的是**投资前的想法**，不替代需求澄清（grill-me）、方法审查（grill-method）和决策审查（DDD）
- 外部结论必须带真实出处是红线——如果你发现报告里有打不开的链接，那是严重违规，欢迎提 issue
- **客户端兼容声明**：本 Skill 已在 ZCode 会话中验证（模拟评估 + 真实使用）；其余客户端对 description 的加载与软触发行为未逐一验证，不保证一致
- **版本策略**：SKILL.md 的行为性变更必须 bump frontmatter 中的 `version` 并打 git tag（当前 `v1.0.0`）

## 测试

`evals/evals.json` 内置 3 个测试用例（2 正 1 负），带 Skill / 不带 Skill 对照运行，11/12 断言通过：

| 用例 | 验证点 | 结果 |
|---|---|---|
| 自研公众号排版工具（重复造轮子型） | FRAME 可证伪主张、RECON 真实链接+盲区声明、判决三件套完整 | ✅ 5/5 |
| AI 自动修 CI 开 PR（前沿探索型） | 失败案例检索、理论冲突记录、便宜验证+放弃信号 | ✅ 4/5* |
| 负例：pandoc 转 PDF 直接开始 | 已决定的执行请求不触发体检 | ✅ 2/2 |

\* 唯一 FAIL 是首轮 eval2 检索过度（47 次调用）——已当场修复为"检索预算 ≤10 次"条款。两份完整评估报告见 [examples/](examples/)。

## License

[MIT](LICENSE)
