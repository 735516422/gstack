# gstack

**gstack 将 Claude Code 从一个通用的助手转变为一组您可以按需调用的专家团队。**

十三个针对 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 的意见化工作流技能。计划审查、设计审查、代码审查、一键发布、浏览器自动化、QA 测试、工程回顾和发布后文档 — 全部作为斜杠命令提供。

### 没有 gstack

- 代理会按字面意思理解您的请求 — 从不询问您是否在构建正确的东西
- 它会完全按照您所说的来实现，即使真正的产品更大
- "审查我的 PR" 每次都给出不一致的深度
- "发布这个" 会变成关于要做什么的长时间来回讨论
- 代理可以写代码但看不到您的应用 — 它是半盲的
- 您仍然手动进行 QA：打开浏览器，四处点击，检查页面，眯着眼睛看布局

### 使用 gstack

| Skill | Mode | What it does |
|-------|------|--------------|
| `/plan-ceo-review` | 创始人 / CEO | 重新思考问题。找到请求中隐藏的 10 星级产品。 |
| `/plan-eng-review` | 工程经理 / 技术主管 | 锁定架构、数据流、图表、边界情况和测试。 |
| `/plan-design-review` | 高级产品设计师 | 设计师眼光审查。80 项检查清单、字母评分、AI 废话检测、DESIGN.md 推断。仅报告 — 从不接触代码。 |
| `/design-consultation` | 设计顾问 | 从头开始构建完整的设计系统。研究竞争对手，提出美学 + 字体 + 颜色 + 间距 + 动效，生成预览页面，并编写 DESIGN.md。 |
| `/review` | 偏执的资深工程师 | 找到通过 CI 但在生产中崩溃的 bug。分类 Greptile 审查评论。 |
| `/ship` | 发布工程师 | 同步 main，运行测试，解决 Greptile 审查，推送，打开 PR。适用于准备好的分支，不用于决定构建什么。 |
| `/browse` | QA 工程师 | 给代理眼睛。它登录，点击您的应用，截图，捕获损坏。60 秒内完成完整 QA。 |
| `/qa` | QA + 修复工程师 | 测试应用，查找 bug，用原子提交修复它们，重新验证。修复前后的健康评分和发布就绪摘要。三个级别：快速、标准、详尽。 |
| `/qa-only` | QA 报告员 | 仅报告的 QA 测试。与 /qa 相同的方法论但从不修复任何内容。当您想要没有代码更改的纯 bug 报告时使用。 |
| `/qa-design-review` | 设计师 + 前端工程师 | 与 /plan-design-review 相同的设计审查，然后修复发现的问题。原子 `style(design):` 提交，修复前后截图，CSS 安全的自我调节。 |
| `/setup-browser-cookies` | 会话管理器 | 将 cookie 从您的真实浏览器（Comet、Chrome、Arc、Brave、Edge）导入到无头会话中。无需手动登录即可测试经过身份验证的页面。 |
| `/retro` | 工程经理 | 团队感知的回顾：您的深入剖析 + 每位贡献者的赞扬和成长机会。 |
| `/document-release` | 技术文档编写者 | 更新 README、ARCHITECTURE、CONTRIBUTING 和项目文档以匹配您刚刚发布的内容。 |

## 演示：一个功能，五种模式

我几乎每个功能都从计划模式开始。我描述我想构建的内容，然后使用 `/plan-ceo-review` 来压力测试我是否在构建正确的东西。只有在产品方向确定后，我才会转向工程、审查、发布和测试。

```
You:   [进入计划模式，描述功能]
       我想添加卖家照片上传到列表应用。卖家应该能够上传他们物品的照片，
       我们从中创建列表。

You:   /plan-ceo-review

Claude: "照片上传"不是功能。真正的工作是帮助卖家创建真正能销售的列表。
        10 星级版本的样子：从照片中自动识别产品，从网上拉取规格和定价对比，
        起草标题和描述，建议最佳主图...

You:   /plan-eng-review

Claude: [架构图，上传 → 分类 → 丰富 → 起草管道的状态机，异步作业边界，
        失败模式，测试矩阵]

You:   [退出计划模式，实施计划]

You:   /review

Claude: 竞态条件：两个选项卡可以覆盖封面照片选择。
        信任边界：将网页数据拉入草稿生成会创建提示注入向量。
        缺失：失败上传的孤立清理。

You:   （修复问题）

You:   /ship

Claude: [同步 main，运行测试，推送分支，打开 PR — 6 个工具调用，完成]

You:   /qa

Claude: 分析分支差异... 8 个文件更改，3 个路由受影响。
        [在 localhost:3000 上测试 /listings/new, /listings/:id, /api/listings]
        所有 3 个路由正常工作。上传 + 丰富流程端到端通过。

You:   /setup-browser-cookies staging.myapp.com

Claude: 从 Chrome 导入了 staging.myapp.com 的 8 个 cookie。

You:   /qa https://staging.myapp.com --quick

Claude: [冒烟测试：首页 + 5 个页面，30 秒]
        健康评分：91/100。无关键问题。1 个中等：移动导航重叠。

You:   /plan-design-review https://staging.myapp.com

Claude: 设计评分：B  |  AI 废话评分：C

        "网站传达能力但不传达信心。"
        "我注意到各部分之间的间距是统一的 — 没有什么感觉重要。"
        "我的眼睛首先看到的 3 个东西是：渐变主图、3 列带圆形图标的特性网格，
         和一个模板化的 CTA。"
        "如果我用一个词来描述：模板。"

        主要问题：通用字体（到处都是 Inter），AI 废话模式（3 列图标网格，
        紫色渐变），扁平的标题缩放。

        [完整报告及字母评分、12 个发现、推断的设计系统]
        想要我将其保存为您的 DESIGN.md 吗？

You:   /qa-design-review

Claude: [运行相同的审查，然后修复 8 个设计问题]
        style(design): FINDING-001 — 将 Inter 替换为系统字体堆栈
        style(design): FINDING-003 — 添加标题缩放对比
        style(design): FINDING-007 — 将 3 列网格拆分为不对称布局
        设计评分：B → A-  |  AI 废话评分：C → A
```

## 适用人群

您已经大量使用 Claude Code，并希望始终如一的高严格度工作流程，而不是一个模糊的通用模式。您想告诉模型现在应该使用什么样的大脑 — 创始人品味、工程严格度、偏执的审查或快速执行。

这不是给初学者的提示包。它是给发布产品的人的操作系统。

## 如何飞翔：同时运行 10 个会话

gstack 在一个 Claude Code 会话中很强大。在十个会话中则是革命性的。

[Conductor](https://conductor.build) 并行运行多个 Claude Code 会话 — 每个都有自己的独立工作区。这意味着您可以有一个会话在 staging 上运行 `/qa`，另一个在 PR 上执行 `/review`，第三个实现一个功能，还有七个在其他分支上工作。所有这些同时进行。

每个工作区自动获得自己的独立浏览器实例 — 独立的 Chromium 进程、cookie、选项卡和存储在每个项目根目录的 `.gstack/` 中的日志。没有端口冲突，没有共享状态，无需配置。`/browse` 和 `/qa` 会话永远不会相互干扰，即使跨越十个并行工作区。

这就是我使用的设置。一个人，十个并行代理，每个都有适合其任务的正确认知模式。这不是增量改进。这是构建软件的不同方式。

## 安装

**要求：** [Claude Code](https://docs.anthropic.com/en/docs/claude-code)、[Git](https://git-scm.com/)、[Bun](https://bun.sh/) v1.0+。`/browse` 编译原生二进制文件 — 适用于 macOS 和 Linux（x64 和 arm64）。

### 步骤 1：在您的机器上安装

打开 Claude Code 并粘贴此内容。Claude 将完成剩下的工作。

> 安装 gstack：运行 `git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup`，然后在 CLAUDE.md 中添加一个 "gstack" 部分，说明对于所有网页浏览使用 gstack 的 /browse 技能，永远不要使用 mcp__claude-in-chrome__* 工具，并列出可用技能：/plan-ceo-review、/plan-eng-review、/plan-design-review、/design-consultation、/review、/ship、/browse、/qa、/qa-only、/qa-design-review、/setup-browser-cookies、/retro、/document-release。然后询问用户是否也想将 gstack 添加到当前项目以便团队成员获得它。

### 步骤 2：添加到您的仓库以便团队成员获得它（可选）

> 将 gstack 添加到此项目：运行 `cp -Rf ~/.claude/skills/gstack .claude/skills/gstack && rm -rf .claude/skills/gstack/.git && cd .claude/skills/gstack && ./setup`，然后在此项目的 CLAUDE.md 中添加一个 "gstack" 部分，说明对于所有网页浏览使用 gstack 的 /browse 技能，永远不要使用 mcp__claude-in-chrome__* 工具，列出可用技能：/plan-ceo-review、/plan-eng-review、/plan-design-review、/design-consultation、/review、/ship、/browse、/qa、/qa-only、/qa-design-review、/setup-browser-cookies、/retro、/document-release，并告诉 Claude 如果 gstack 技能不工作，运行 `cd .claude/skills/gstack && ./setup` 来构建二进制文件并注册技能。

真实文件会提交到您的仓库（不是子模块），所以 `git clone` 就可以工作。二进制文件和 node_modules 被 gitignored — 团队成员只需运行 `cd .claude/skills/gstack && ./setup` 一次来构建（或者 `/browse` 在首次使用时自动处理）。

### 安装了什么

- 技能文件（Markdown 提示）在 `~/.claude/skills/gstack/`（或项目安装的 `.claude/skills/gstack/`）
- `~/.claude/skills/browse`、`~/.claude/skills/qa`、`~/.claude/skills/review` 等位置的符号链接指向 gstack 目录
- 浏览器二进制文件在 `browse/dist/browse`（约 58MB，gitignored）
- `node_modules/`（gitignored）
- `/retro` 将 JSON 快照保存到项目中的 `.context/retros/` 用于趋势跟踪

所有内容都在 `.claude/` 内部。没有任何内容触及您的 PATH 或在后台运行。

---

```
+----------------------------------------------------------------------------+
|                                                                            |
|   你是一位优秀的软件工程师，喜欢每天写 10K 行代码         |
|   并像 Garry 一样每天落地 10 个 PR 吗？                                        |
|                                                                            |
|   来 YC 工作：ycombinator.com/software                                |
|                                                                            |
|   极具竞争力的薪资和股权。                                 |
|   现在在旧金山 Dogpatch 区招聘。                          |
|   来加入革命吧。                                                |
|                                                                            |
+----------------------------------------------------------------------------+
```

---

## 我如何使用这些技能

由 [Garry Tan](https://x.com/garrytan) 创建，[Y Combinator](https://www.ycombinator.com/) 的总裁兼 CEO。

我构建 gstack 是因为我不希望 AI 编码工具困在一个模糊的模式中。

计划不是审查。审查不是发布。创始人品味不是工程严格度。如果您把所有这些混在一起，通常会得到四者的平庸混合体。

我想要明确的档位。

这些技能让我可以告诉模型我现在想要什么样的大脑。我可以按需切换认知模式 — 创始人、工程经理、偏执的审查者、发布机器。这就是解锁。

---

## `/plan-ceo-review`

这是我的**创始人模式**。

在这里我希望模型以品味、雄心、用户同理心和长远的时间框架来思考。我不希望它按字面意思理解请求。我希望它首先问一个更重要的问题：

**这个产品实际上是做什么的？**

我将其视为 **Brian Chesky 模式**。

重点不是实施明显的票据。重点是从用户的角度重新思考问题，找到感觉必然、令人愉悦甚至可能有点神奇的版本。

### 示例

假设我正在构建一个 Craigslist 风格的列表应用，我说：

> "让卖家为他们的物品上传一张照片。"

一个弱助手会添加一个文件选择器并保存图像。

这不是真正的产品。

在 `/plan-ceo-review` 中，我希望模型询问"照片上传"是否甚至是一个功能。也许真正的功能是帮助某人创建真正能销售的列表。

如果那是真正的工作，整个计划就会改变。

现在模型应该问：

* 我们能从照片中识别产品吗？
* 我们能推断 SKU 或型号吗？
* 我们能搜索网络并自动起草标题和描述吗？
* 我们能拉取规格、类别和定价对比吗？
* 我们能建议哪张照片作为主图效果最好吗？
* 我们能检测上传的照片是丑陋、黑暗、混乱还是低信任度的吗？
* 我们能让体验感觉高级，而不是像 2007 年的死表单吗？

这就是 `/plan-ceo-review` 为我做的。

它不只是问，"我如何添加这个功能？"
它问，**"这个请求中隐藏的 10 星级产品是什么？"**

那是一种非常不同的力量。

---

## `/plan-eng-review`

这是我的**工程经理模式**。

一旦产品方向正确，我需要一种完全不同的智能。我不想要更多的蔓延式构思。我不想要更多的"如果...会不会很酷"。我希望模型成为我最好的技术主管。

这种模式应该搞定：

* 架构
* 系统边界
* 数据流
* 状态转换
* 失败模式
* 边界情况
* 信任边界
* 测试覆盖

对我来说还有一个令人惊讶的大解锁：**图表**。

当您强制 LLM 绘制系统时，它们会变得更加完整。序列图、状态图、组件图、数据流图，甚至测试矩阵。图表将隐藏的假设暴露在明面上。它们使模糊的计划变得更加困难。

所以 `/plan-eng-review` 是我希望模型构建能够承载产品愿景的技术骨干的地方。

### 示例

使用相同的列表应用示例。

假设 `/plan-ceo-review` 已经完成了它的工作。我们决定真正的功能不仅仅是照片上传。它是一个智能列表流程，可以：

* 上传照片
* 识别产品
* 从网上丰富列表
* 起草强大的标题和描述
* 建议最佳主图

现在 `/plan-eng-review` 接管。

现在我希望模型回答这样的问题：

* 上传、分类、丰富和草稿生成的架构是什么？
* 哪些步骤同步发生，哪些进入后台作业？
* 应用服务器、对象存储、视觉模型、搜索/丰富 API 和列表数据库之间的边界在哪里？
* 如果上传成功但丰富失败会发生什么？
* 如果产品识别置信度低会发生什么？
* 重试如何工作？
* 我们如何防止重复作业？
* 何时持久化什么，什么可以安全地重新计算？

在这里我想要图表 — 架构图、状态模型、数据流图、测试矩阵。图表将隐藏的假设暴露在明面上。它们使模糊的计划变得更加困难。

这就是 `/plan-eng-review`。

不是"让想法变小。"
**让想法可构建。**

---

## `/plan-design-review`

这是我的**高级设计师模式**。

大多数开发者无法判断他们的网站看起来是否是 AI 生成的。我以前也不能，直到我开始注意。有一类正在增长的网站，它们功能完善但没有灵魂 — 它们工作正常，但大喊"AI 构建了这个，没有有品味的人看过它。"紫色渐变、3 列图标网格、所有东西上统一的圆角 border-radius、每个部分居中的文本、背景中漂浮的装饰斑点。ChatGPT 美学。

`/plan-design-review` 给代理一个设计师的眼睛。

它打开您的网站，并像 Stripe 或 Linear 设计师那样对它做出反应 — 立即、发自内心地，带有意见。第一个输出是结构化的直觉反应：网站在一瞥中传达了什么，眼睛被什么吸引，以及一个字的裁决。这是最有价值的部分。之后的所有内容都是支持性证据。

然后它通过读取 CSS 提取您的实际设计系统：您实际上在使用多少种字体？您的标题缩放是什么？有多少种颜色？您的间距是在网格上还是任意的？它将其结构化为**推断的设计系统**，并提议将其保存为 `DESIGN.md` — 所以现在您拥有了一个您以前没有的设计系统，从您实际发布的内容派生而来。

审查涵盖 10 个类别的 80 个项目。字体、间距、层次结构、颜色、响应式设计、交互状态、动效、内容质量、性能感觉，以及 — 头条功能 — **AI 废话检测**。十种特定的反模式，标记网站为机器生成的。

每个发现在每个类别中获得字母评分（A 到 F）。报告以两个头条分数开始：**设计评分**和**AI 废话评分**。"您的设计获得 B，但 AI 废话获得 D"确切地告诉您应该专注于哪里。

### 示例

```
You:   /plan-design-review https://myapp.com

Claude: 设计评分：C  |  AI 废话评分：D

        "网站传达通用 SaaS 能量。"
        "我注意到主图部分使用了蓝紫色渐变，下面是 3 列特性网格 —
         单一最可识别的 AI 生成布局模式。"
        "我的眼睛首先看到的 3 个东西是：渐变、对称图标网格，
         和一个看起来像每个其他"开始"按钮的"开始"按钮。"
        "如果我用一个词来描述：模板。"

        推断的设计系统：
        - 字体：Inter（正文），Inter（标题）— 1 个字体系列
        - 颜色：14 种独特的非灰色颜色（标志：>12）
        - 标题缩放：36/24/18/16 — 扁平，尺寸之间没有戏剧性
        - 间距：主要是 16px 和 24px — 没有节奏

        前 5 个设计改进：
        1. 打破 3 列图标网格（高，AI 废话）
        2. 添加标题缩放对比 — 至少 2:1 比例（高，字体）
        3. 用有意的层次结构替换统一的 border-radius（中等，间距）
        4. 消除渐变主图 — 使用真实图像或粗体排版（高，AI 废话）
        5. 为标题添加第二种字体 — 仅 Inter 读起来很通用（中等，字体）

        [完整报告保存到 .gstack/design-reports/]
        想要我保存这个推断的设计系统作为您的 DESIGN.md 吗？
```

这仅报告 — 从不接触您的代码。当您希望它修复发现的问题时，使用 `/qa-design-review`。

---

## `/design-consultation`

这是我的**设计合作伙伴模式**。

`/plan-design-review` 审查一个已经存在的网站。`/design-consultation` 是当您什么都没有时 — 没有设计系统，没有字体选择，没有调色板。您从零开始，希望高级设计师与您坐下来一起构建整个视觉标识。

这是一个对话，而不是表单。代理询问关于您的产品、您的用户和您的空间。如果您愿意，它会研究您类别中的顶级竞争对手在做什么 — 字体、颜色、布局模式、美学方向。然后它提议一个完整、连贯的设计系统：美学方向、字体（3 种以上具有特定角色的字体）、带有十六进制值的调色板、间距缩放、布局方法和动效策略。每个建议都有理由。每个选择都强化每个其他选择。

关键见解：单个设计决策很容易做出，但很难连贯地做出。选择一种字体很简单。选择一种与您的调色板、间距密度、美学方向和产品个性配合的字体 — 那才是真正的工作。`/design-consultation` 通过将所有内容作为一个包提议来解决连贯性问题，然后让您调整任何部分，同时检查其余部分仍然保持在一起。

在您同意系统后，它会生成交互式 HTML 预览页面 — 您的字体以真实大小渲染，您的调色板作为实际 UI 组件（按钮、卡片、表单输入、警报），明/暗模式切换，所有这些都使用您的产品名称而不是 lorem ipsum。您在浏览器中打开页面，并在编写一行产品代码之前看到设计系统呈现。

然后它将 `DESIGN.md` 写入您的仓库根目录 — 您项目的设计真实来源 — 并更新 `CLAUDE.md`，以便每个未来的 Claude Code 会话都遵守系统。从那时起，`/qa-design-review` 可以对其进行审计，并且任何在您前端工作的代理都知道规则。

```
You:   /design-consultation

Claude: 从 README 来看，这看起来像是一个面向工程团队的
        开发者分析仪表板。对吗？您希望我研究其他开发工具在设计方面做什么，
        还是我根据我已知的内容工作？

You:    是的，它是面向工程经理的仪表板。研究会很棒。

Claude: 我查看了 Linear、Vercel、Railway、Planetscale 和 Posthog。
        它们都采用干净的无衬线字体和柔和的调色板，但 Linear 以其暗色优先方法和紧密间距脱颖而出。
        差距：大多数这些感觉可以互换。这是我的提议：

        美学：工业/实用主义 — 数据优先，无装饰税
        字体：Geist（正文 + UI），Instrument Serif（展示），JetBrains Mono（数据）
        颜色：克制 — 板岩中性色，单一蓝绿色强调色（#0D9488）
        间距：4px 基数，紧凑密度
        布局：网格纪律，12 列
        动效：最小功能 — 仅有助于理解的过渡

        这是连贯的，因为实用主义美学需要紧密的间距网格和克制的颜色。
        衬线展示字体在不增加视觉噪声的情况下创建层次对比。
        数据表的等宽字体对工程受众来说感觉很原生。

        想要调整什么，还是我应该生成预览页面？

You:    生成它。

Claude: 预览在您的浏览器中打开。
        [/tmp/design-consultation-preview-1710000000.html]

        页面展示主图缩放的 Instrument Serif，正文段落中的 Geist，
        示例数据表中的 JetBrains Mono。颜色样本带有蓝绿色强调色和板岩灰色。
        按钮、卡片和警报在亮色和暗色模式下都呈现。

        感觉如何？在写 DESIGN.md 之前想要调整什么吗？

You:    看起来很棒。发布它。

Claude: 写入 DESIGN.md（字体、颜色、间距、布局、动效）。
        更新 CLAUDE.md 以包含设计系统指南。
        所有未来的会话都将遵守这些选择。
```

---

## `/qa-design-review`

这是我的**会编码的设计师模式**。

`/plan-design-review` 告诉您什么是错的。`/qa-design-review` 修复它。

它运行相同的 80 项审查，然后进入修复循环：对于每个设计发现，它定位源文件，进行最小的 CSS/样式更改，使用 `style(design): FINDING-NNN` 提交，重新导航以验证，并拍摄修复前后截图。每个修复一次提交，完全可二分查找。

自我调节启发式针对设计工作进行调整 — 仅 CSS 更改获得免费通行（它们本质上安全且可逆），但对组件 JSX/TSX 文件的更改计入风险预算。硬限制为 30 个修复。如果风险评分超过 20%，它会停止并询问。

```
You:   /qa-design-review https://myapp.com

Claude: [运行完整的设计审查 — 与 /plan-design-review 相同的输出]
        设计评分：C  |  AI 废话评分：D
        12 个发现（4 个高，5 个中，3 个润色）

        修复 9 个设计问题...

        style(design): FINDING-001 — 用不对称布局替换 3 列图标网格
        style(design): FINDING-002 — 添加标题缩放 48/32/24/18/16
        style(design): FINDING-003 — 消除渐变主图，使用粗体排版
        style(design): FINDING-004 — 为标题添加第二种字体
        style(design): FINDING-005 — 根据元素角色变化 border-radius
        style(design): FINDING-006 — 左对齐正文文本，将中心保留给标题
        style(design): FINDING-007 — 为所有交互元素添加悬停/焦点状态
        style(design): FINDING-008 — 添加 prefers-reduced-motion 媒体查询
        style(design): FINDING-009 — 将正文内容的最大宽度设置为 680px

        最终审查：
        设计评分：C → B+  |  AI 废话评分：D → A
        应用 9 个修复（8 个已验证，1 个尽力而为）。3 个推迟。

        [带修复前后截图的报告保存到 .gstack/design-reports/]
```

九次提交，每次触及一个问题。AI 废话评分从 D 上升到 A，因为三个最可识别的模式（渐变主图、3 列网格、统一半径）都消失了。设计评分提高了两个等级，因为字体现在有缩放，间距有层次结构，交互元素有适当的状态。

---

## `/review`

这是我的**偏执的资深工程师模式**。

通过测试并不意味着分支是安全的。

`/review` 存在是因为有一类 bug 可以在 CI 中幸存，并在生产环境中给您带来麻烦。这种模式不是关于梦想更大。不是关于让计划更漂亮。它是关于问：

**什么仍然可能出问题？**

这是一种结构审计，而不是风格挑剔。我希望模型寻找这样的东西：

* N+1 查询
* 过期读取
* 竞态条件
* 糟糕的信任边界
* 缺失的索引
* 转义 bug
* 破坏的不变量
* 糟糕的重试逻辑
* 在错过真实失败模式时通过的测试

### 示例

假设智能列表流程已实现，测试是绿色的。

`/review` 仍然应该问：

* 我在渲染列表照片或草稿建议时是否引入了 N+1 查询？
* 我是否信任客户端提供的文件元数据而不是验证实际文件？
* 两个选项卡可以竞争并覆盖封面照片选择或项目详细信息吗？
* 失败的上传会在存储中永远留下孤立文件吗？
* "恰好一个主图"规则在并发下会破坏吗？
* 如果丰富 API 部分失败，我会优雅地降级还是保存垃圾？
* 我是否通过将网页数据拉入草稿生成而意外创建了提示注入或信任边界问题？

这就是 `/review` 的重点。

我不希望这里奉承。
我希望模型在生产事故发生之前想象它。

---

## `/ship`

这是我的**发布机器模式**。

一旦我决定了构建什么，确定了技术计划，并进行了严肃的审查，我不想再说话了。我想要执行。

`/ship` 用于最后一公里。它用于准备好的分支，不用于决定构建什么。

在这里，模型应该停止表现得像头脑风暴伙伴，并开始表现得像自律的发布工程师：与 main 同步，运行正确的测试，确保分支状态正常，如果仓库期望则更新变更日志或版本控制，推送，并创建或更新 PR。

在这里动量很重要。

许多分支在有趣的工作完成而只有无聊的发布工作剩余时死亡。人类会拖延那部分。AI 不应该。

### 示例

假设智能列表流程已完成。

产品思考已完成。
架构已完成。
审查通过已完成。
现在分支只需要落地。

这就是 `/ship` 的用途。

它处理重复的发布卫生工作，所以我不会在这些事情上浪费精力：

* 与 main 同步
* 重新运行测试
* 检查奇怪的分支状态
* 更新变更日志/版本元数据
* 推送分支
* 打开或更新 PR

在这一点上，我不想要更多的构思。
我想要飞机着陆。

---

## Greptile 集成

[Greptile](https://greptile.com) 是一家 YC 公司，自动审查您的 PR。它捕获真正的 bug — 竞态条件、安全问题、通过 CI 并在生产环境中爆炸的东西。它真的多次救了我的命。我喜欢这些人。

**设置：** 在 [greptile.com](https://greptile.com) 上将 Greptile 安装到您的 GitHub 仓库 — 大约需要 30 秒。一旦它审查您的 PR，gstack 会自动获取其评论。无需额外配置。

任何自动审查者的问题都是分类。Greptile 很好，但并非每个评论都是真正的问题。有些是误报。有些标记您在三个提交前已经修复的东西。没有分类层，评论会堆积，您开始忽略它们 — 这违背了目的。

gstack 解决了这个问题。`/review` 和 `/ship` 现在具有 Greptile 感知能力。它们读取 Greptile 的评论，对每个评论进行分类，并采取行动：

- **有效问题**被添加到关键发现中并在发布前修复
- **已修复的问题**获得自动回复确认捕获
- **误报**被推回 — 您确认，并发出回复解释为什么它是错误的

结果是两层审查：Greptile 在 PR 上异步捕获内容，然后 `/review` 和 `/ship` 作为正常工作流程的一部分对这些发现进行分类。没有任何东西会漏掉。

它还会学习。您确认的每个误报都会保存到 `~/.gstack/greptile-history.md`。未来的运行会自动跳过您代码库的已知误报模式。`/retro` 会跟踪 Greptile 随时间的命中率 — 所以您可以看到信噪比是否在改善。

### 示例

```
You:   /ship

Claude: [同步 main，运行测试，预落地审查...]

        Greptile 在此 PR 上发现 3 条评论：

        [有效] app/services/payment_service.rb:47 — 竞态条件：
        并发费用可以双重扣款。建议在费用块周围使用数据库级建议锁。
        → https://github.com/you/app/pull/42#discussion_r123

        [已修复] app/models/user.rb:12 — 在 email 上小写之前缺少空检查。
        → 已在提交 abc1234 中修复。自动回复。

        [误报] lib/auth.rb:88 — "令牌比较应该使用常量时间比较。"
        我们已经在这里使用 ActiveSupport::SecurityUtils.secure_compare。
        → https://github.com/you/app/pull/42#discussion_r125

        修复 payment_service.rb 中的竞态条件...
        应用了建议锁。已提交：fix: address Greptile review — advisory lock on concurrent charges

        身份验证令牌评论是误报 — 我们已经在使用 secure_compare。

        误报的选项：
          A) 回复 Greptile 解释（推荐）
          B) 无论如何修复它
          C) 忽略

You:    A

Claude: 已回复 Greptile。修复后重新运行测试...
        所有测试通过。继续版本升级。

        [在正文中创建带有 Greptile 审查部分的 PR]
        PR: https://github.com/you/app/pull/42
```

三个 Greptile 评论。一个真正的修复。一个自动确认。一个带有回复的误报被推回。总额外时间：大约 30 秒。PR 正文有完整的审计跟踪。

---

## `/browse`

这是我的**QA 工程师模式**。

`/browse` 是关闭循环的技能。在它之前，代理可以思考和编码，但仍然是半盲的。它必须猜测 UI 状态、身份验证流程、重定向、控制台错误、空状态和损坏的布局。现在它可以直接去看。

这是一个真正的步骤变化。完整循环变为：计划、编码、运行应用、检查 UI、重现 bug、验证修复、发布。QA 不再是"手动四处点击"，而成为相同代理循环的一部分。它给代理眼睛。现在它可以自己做无聊、高上下文的 QA 工作：点击应用、捕获损坏、验证修复并继续。这感觉就像机器中有真正的操作员。

它是一个编译后的二进制文件，与持久的 Chromium 守护进程通信 — 由 Microsoft 基于 [Playwright](https://playwright.dev/) 构建。第一次调用启动浏览器（约 3 秒）。之后的每次调用：约 100-200 毫秒。浏览器在命令之间保持运行，所以 cookie、选项卡和 localStorage 会延续。

在推送分支后，我告诉 Claude 去检查 staging。它为我完成整个 QA 流程：

```
You:   /browse staging.myapp.com — 登录，测试注册流程，并检查
       我在此分支中更改的每个页面

Claude: [18 个工具调用，约 60 秒]

        > browse goto https://staging.myapp.com/signup
        > browse snapshot -i
        > browse fill @e2 "test@example.com"
        > browse fill @e3 "password123"
        > browse click @e5                    (Submit)
        > browse screenshot /tmp/signup.png
        > Read /tmp/signup.png

        注册有效。重定向到入门。现在检查更改的页面。

        > browse goto https://staging.myapp.com/dashboard
        > browse screenshot /tmp/dashboard.png
        > Read /tmp/dashboard.png
        > browse console

        仪表板加载。没有控制台错误。图表用示例数据呈现。

        > browse goto https://staging.myapp.com/settings/billing
        > browse screenshot /tmp/billing.png
        > Read /tmp/billing.png

        计费页面加载。计划选择器显示免费/专业/企业版。
        Stripe 元素正确呈现。

        > browse goto https://staging.myapp.com/api/health
        > browse text

        API 健康端点返回 {"status":"ok","version":"2.4.1"}。

        所有 4 个页面正确加载。没有控制台错误。没有损坏的布局。
        注册 → 入门 → 仪表板流程端到端有效。
```

18 个工具调用，大约一分钟。它注册了测试用户，导航了我更改的每个页面，截图，读取它们，检查控制台错误，并验证了 API。这是一个完整的 QA 流程。我没有打开浏览器。

它适用于阅读文档、验证部署、填写表单、截图、跨环境比较页面，以及 Claude 需要在实时 URL 上查看眼睛的任何其他情况。

**安全说明：** `/browse` 运行持久的 Chromium 会话。Cookie、localStorage 和会话状态在命令之间延续。除非您有意，否则不要将其用于敏感的生产环境 — 它是带有真实状态的真实浏览器。会话在空闲 30 分钟后自动关闭。

有关完整的命令参考、技术内部和架构详细信息，请参阅 [BROWSER.md](BROWSER.md)。

---

## `/qa`

这是我的**QA 主管模式**。

`/browse` 给代理眼睛。`/qa` 给它测试方法论。

最常见的用例：您在功能分支上，刚刚完成编码，并希望验证一切正常。只需说 `/qa` — 它读取您的 git 差异，识别您的更改影响哪些页面和路由，启动浏览器，并测试每个页面。不需要 URL。不需要手动测试计划。它根据您更改的代码弄清楚要测试什么。

```
You:   /qa

Claude: 分析分支与 main 的差异...
        12 个文件更改：3 个控制器，2 个视图，4 个服务，3 个测试

        受影响的路由：/listings/new, /listings/:id, /api/listings
        检测到应用在 localhost:3000 上运行。

        [测试每个受影响的页面 — 导航、填写表单、点击按钮、
         截图、检查控制台错误]

        QA 报告：3 个路由测试，全部正常。
        - /listings/new：上传 + 丰富流程端到端有效
        - /listings/:id：详情页面正确呈现
        - /api/listings：返回带有预期形状的 200
        没有控制台错误。相邻页面上没有回归。
```

四种模式：

- **差异感知**（在功能分支上自动）— 读取 `git diff main`，识别受影响的页面，专门测试它们。从"我刚刚写了代码"到"它有效"的最快路径。
- **完整** — 系统地探索整个应用。5-15 分钟，取决于应用大小。记录 5-10 个有证据的问题。
- **快速**（`--quick`）— 30 秒冒烟测试。首页 + 前 5 个导航目标。加载？控制台错误？损坏的链接？
- **回归**（`--regression baseline.json`）— 运行完整模式，然后与先前的基准进行比较。哪些问题已修复？哪些是新的？分数差异是多少？

```
You:   /qa https://staging.myapp.com

Claude: [探索 12 个页面，填写 3 个表单，测试 2 个流程]

        QA 报告：staging.myapp.com — 健康评分：72/100

        前 3 个问题：
        1. 关键：结账表单在必填字段为空时提交
        2. 高：移动导航菜单在选择项目后不关闭
        3. 中：仪表板图表在 1024px 以下与侧边栏重叠

        [完整报告及截图保存到 .gstack/qa-reports/]
```

报告和截图累积在 `.gstack/qa-reports/` 中，以便您可以随时间跟踪质量并比较运行。

**测试经过身份验证的页面：** 首先使用 `/setup-browser-cookies` 导入真实的浏览器会话，然后 `/qa` 可以测试登录后面的页面。

---

## `/setup-browser-cookies`

这是我的**会话管理器模式**。

在 `/qa` 或 `/browse` 可以测试经过身份验证的页面之前，它们需要 cookie。不用每次都通过无头浏览器手动登录，`/setup-browser-cookies` 直接从您的日常浏览器导入真实会话。

它自动检测已安装的 Chromium 浏览器（Comet、Chrome、Arc、Brave、Edge），通过 macOS 钥匙链解密 cookie，并将它们加载到 Playwright 会话中。交互式选择器 UI 让您准确选择要导入的域 — 永不显示 cookie 值。

```
You:   /setup-browser-cookies

Claude: Cookie 选择器已打开 — 在浏览器中选择您想要导入的域，
        然后在完成时告诉我。

        [您在浏览器 UI 中选择 github.com, myapp.com]

You:    完成

Claude: 已导入 2 个域（47 个 cookie）。会话已就绪。
```

或者完全跳过 UI：

```
You:   /setup-browser-cookies github.com

Claude: 已从 Comet 导入 github.com 的 12 个 cookie。
```

每种浏览器的第一次导入会触发 macOS 钥匙链提示 — 单击"允许"或"始终允许"。

---

## `/retro`

这是我的**工程经理模式**。

在周末结束时，我想知道实际发生了什么。不是感觉 — 是数据。`/retro` 分析提交历史、工作模式和发布速度，并撰写坦诚的回顾。

它是团队感知的。它识别谁在运行命令，给您自己的工作最深入的处理，然后用具体的赞扬和成长机会分解每个贡献者 — 您在 1:1 中实际会给出的反馈类型。它计算提交、LOC、测试比率、PR 大小和修复比率等指标。它从提交时间戳检测编码会话，找到热点文件，跟踪发布连续性，并识别本周最大的发布。

```
You:   /retro

Claude: 3 月 1 日这一周：47 次提交（3 位贡献者），3.2k LOC，38% 测试，
        12 个 PR，峰值：晚上 10 点 | 连续性：47 天

        ## 您的一周
        32 次提交，+2.4k LOC，41% 测试。高峰时间：晚上 9-11 点。
        最大发布：cookie 导入系统（浏览器解密 + 选择器 UI）。
        您做得好的地方：在一次集中推进中发布了一个带有加密、UI 和
         18 个单元测试的完整功能...

        ## 团队分解

        ### Alice
        12 次提交专注于 app/services/。每个 PR 低于 200 LOC — 自律。
        机会：测试比率 12% — 在支付变得更加复杂之前值得投资。

        ### Bob
        3 次提交 — 修复了仪表板上的 N+1 查询。小但影响大。
        机会：本周只有 1 个活跃日 — 检查是否被任何东西阻挡。

        [前 3 个团队胜利，3 件需要改进的事情，3 个下周习惯]
```

它将 JSON 快照保存到 `.context/retros/`，以便下一次运行可以显示趋势。运行 `/retro compare` 并排查看本周与上周。

---

## `/document-release`

这是我的**技术文档编写者模式**。

在 `/ship` 创建 PR 之后但在合并之前，`/document-release` 读取项目中的每个文档文件，并根据差异交叉引用它。它更新文件路径、命令列表、项目结构树以及任何其他漂移的内容。有风险或主观的更改作为问题提出 — 其他所有内容都自动处理。

```
You:   /document-release

Claude: 分析 3 次提交中更改的 21 个文件。发现 8 个文档文件。

        README.md：将技能计数从 9 更新到 10，将新技能添加到表格
        CLAUDE.md：将新目录添加到项目结构
        CONTRIBUTING.md：当前 — 无需更改
        TODOS.md：标记 2 个项目完成，添加 1 个新项目

        所有文档已更新并提交。PR 正文已更新文档差异。
```

它还会润色 CHANGELOG 语气（从不覆盖条目），清理完成的 TODOS，检查跨文档一致性，并仅在适当时询问 VERSION 升级。

---

## 故障排除

**技能没有在 Claude Code 中显示？**
运行 `cd ~/.claude/skills/gstack && ./setup`（或项目安装的 `cd .claude/skills/gstack && ./setup`）。这会重建符号链接，以便 Claude 可以发现技能。

**`/browse` 失败或找不到二进制文件？**
运行 `cd ~/.claude/skills/gstack && bun install && bun run build`。这会编译浏览器二进制文件。需要 Bun v1.0+。

**项目副本过时？**
运行 `/gstack-upgrade` — 它会自动更新全局安装和任何供应商项目副本。

**`bun` 未安装？**
安装它：`curl -fsSL https://bun.sh/install | bash`

## 升级

在 Claude Code 中运行 `/gstack-upgrade`。它会检测您的安装类型（全局或供应商），升级，同步任何项目副本，并显示新内容。

或者设置 `auto_upgrade: true` 在 `~/.gstack/config.yaml` 中，以便在新版本可用时自动升级。

## 卸载

将此粘贴到 Claude Code：

> 卸载 gstack：通过运行 `for s in browse plan-ceo-review plan-eng-review plan-design-review design-consultation review ship retro qa qa-only qa-design-review setup-browser-cookies document-release; do rm -f ~/.claude/skills/$s; done` 删除技能符号链接，然后运行 `rm -rf ~/.claude/skills/gstack` 并从 CLAUDE.md 中删除 gstack 部分。如果此项目在 .claude/skills/gstack 也有 gstack，通过运行 `for s in browse plan-ceo-review plan-eng-review plan-design-review review ship retro qa qa-only qa-design-review setup-browser-cookies document-release; do rm -f .claude/skills/$s; done && rm -rf .claude/skills/gstack` 删除它，并从项目 CLAUDE.md 中也删除 gstack 部分。

## 开发

请参阅 [CONTRIBUTING.md](CONTRIBUTING.md) 了解设置、测试和开发模式。请参阅 [ARCHITECTURE.md](ARCHITECTURE.md) 了解设计决策和系统内部。请参阅 [BROWSER.md](BROWSER.md) 了解 browse 命令参考。

### 测试

```bash
bun test                     # 免费静态测试（<5s）
EVALS=1 bun run test:evals   # 完整 E2E + LLM 评估（~$4，~20 分钟）
bun run eval:watch            # E2E 运行期间的实时仪表板
```

E2E 测试流式传输实时进度，写入机器可读的诊断，并持久化在杀进程中存活的局部结果。有关完整的评估基础设施，请参阅 CONTRIBUTING.md。

## 许可证

MIT
