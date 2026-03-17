# gstack 开发

## 命令

```bash
bun install          # 安装依赖
bun test             # 运行免费测试 (browse + snapshot + skill 验证)
bun run test:evals   # 运行付费评估: LLM judge + E2E (~$4/次)
bun run test:e2e     # 仅运行 E2E 测试 (~$3.85/次)
bun run dev <cmd>    # 开发模式运行 CLI，如 bun run dev goto https://example.com
bun run build        # 生成文档 + 编译二进制文件
bun run gen:skill-docs  # 从模板重新生成 SKILL.md 文件
bun run skill:check  # 所有技能的健康状态仪表板
bun run dev:skill    # 监视模式: 自动重新生成 + 更改时验证
bun run eval:list    # 列出 ~/.gstack-dev/evals/ 中所有评估运行
bun run eval:compare # 比较两次评估运行 (自动选择最近两次)
bun run eval:summary # 跨所有评估运行的聚合统计
```

`test:evals` 需要 `ANTHROPIC_API_KEY`。E2E 测试实时流式传输进度
(通过 `--output-format stream-json --verbose` 逐个工具输出)。结果持久化
到 `~/.gstack-dev/evals/`，并自动与上一次运行进行比较。

## 项目结构

```
gstack/
├── browse/          # 无头浏览器 CLI (Playwright)
│   ├── src/         # CLI + 服务器 + 命令
│   │   ├── commands.ts  # 命令注册表 (单一事实来源)
│   │   └── snapshot.ts  # SNAPSHOT_FLAGS 元数据数组
│   ├── test/        # 集成测试 + 固定装置
│   └── dist/        # 编译的二进制文件
├── scripts/         # 构建 + DX 工具
│   ├── gen-skill-docs.ts  # 模板 → SKILL.md 生成器
│   ├── skill-check.ts     # 健康仪表板
│   └── dev-skill.ts       # 监视模式
├── test/            # Skill 验证 + 评估测试
│   ├── helpers/     # skill-parser.ts, session-runner.ts, llm-judge.ts, eval-store.ts
│   ├── fixtures/    # 基准 JSON, 埋藏的 bug 固定装置, 评估基线
│   ├── skill-validation.test.ts  # 层级 1: 静态验证 (免费, <1s)
│   ├── gen-skill-docs.test.ts    # 层级 1: 生成器质量 (免费, <1s)
│   ├── skill-llm-eval.test.ts   # 层级 3: LLM-as-judge (~$0.15/次)
│   └── skill-e2e.test.ts         # 层级 2: 通过 claude -p E2E (~$3.85/次)
├── qa-only/         # /qa-only skill (仅报告 QA, 无修复)
├── plan-design-review/  # /plan-design-review skill (仅报告设计审计)
├── qa-design-review/    # /qa-design-review skill (设计审计 + 修复循环)
├── ship/            # Ship workflow skill
├── review/          # PR review skill
├── plan-ceo-review/ # /plan-ceo-review skill
├── plan-eng-review/ # /plan-eng-review skill
├── retro/           # Retrospective skill
├── document-release/ # /document-release skill (发布后文档更新)
├── setup            # 一次性设置: 构建二进制文件 + 符号链接技能
├── SKILL.md         # 从 SKILL.md.tmpl 生成 (不要直接编辑)
├── SKILL.md.tmpl    # 模板: 编辑这个, 运行 gen:skill-docs
└── package.json     # browse 的构建脚本
```

## SKILL.md 工作流

SKILL.md 文件是从 `.tmpl` 模板**生成**的。要更新文档:

1. 编辑 `.tmpl` 文件 (例如 `SKILL.md.tmpl` 或 `browse/SKILL.md.tmpl`)
2. 运行 `bun run gen:skill-docs` (或 `bun run build` 会自动执行)
3. 同时提交 `.tmpl` 和生成的 `.md` 文件

要添加新的 browse 命令: 将其添加到 `browse/src/commands.ts` 并重新构建。
要添加 snapshot 标志: 将其添加到 `browse/src/snapshot.ts` 的 `SNAPSHOT_FLAGS` 中并重新构建。

## 编写 SKILL 模板

SKILL.md.tmpl 文件是 **Claude 读取的提示模板**，不是 bash 脚本。
每个 bash 代码块在单独的 shell 中运行 — 变量不会在代码块之间持久化。

规则:
- **使用自然语言表达逻辑和状态。** 不要使用 shell 变量在
  代码块之间传递状态。相反，告诉 Claude 记住什么，并在散文中引用它
  (例如，"步骤 0 中检测到的基础分支")。
- **不要硬编码分支名称。** 通过 `gh pr view` 或 `gh repo view` 动态检测 `main`/`master`/等。
  对于面向 PR 的技能使用 `{{BASE_BRANCH_DETECT}}`。在散文中使用"基础分支"，在代码块占位符中使用 `<base>`。
- **保持 bash 块自包含。** 每个代码块应该独立工作。
  如果一个块需要来自前一个步骤的上下文，在上面的散文中重新说明。
- **用英语表达条件。** 不要在 bash 中使用嵌套的 `if/elif/else`，
  写编号的决策步骤: "1. 如果 X，做 Y。2. 否则，做 Z。"

## 浏览器交互

当需要与浏览器交互 (QA、dogfooding、cookie 设置) 时，使用
`/browse` skill 或直接通过 `$B <command>` 运行 browse 二进制文件。**切勿**使用
`mcp__claude-in-chrome__*` 工具 — 它们慢、不可靠，且不是本项目使用的。

## 供应商符号链接感知

开发 gstack 时，`.claude/skills/gstack` 可能是一个指回此
工作目录的符号链接 (gitignored)。这意味着 skill 更改是**即时生效的** —
非常适合快速迭代，但在大重构期间有风险，因为一半编写的技能可能会破坏
同时使用 gstack 的其他 Claude Code 会话。

**每会话检查一次:** 运行 `ls -la .claude/skills/gstack` 以查看它是否是
符号链接或真实副本。如果它是指向工作目录的符号链接，请注意:
- 模板更改 + `bun run gen:skill-docs` 立即影响所有 gstack 调用
- 对 SKILL.md.tmpl 文件的破坏性更改可能破坏并发的 gstack 会话
- 在大重构期间，删除符号链接 (`rm .claude/skills/gstack`) 以便使用
  `~/.claude/skills/gstack/` 处的全局安装

**对于计划审查:** 当审查修改 skill 模板或
gen-skill-docs 流程的计划时，考虑更改是否应该
在上线前进行隔离测试 (特别是如果用户在其他窗口中积极使用 gstack)。

## CHANGELOG 风格

CHANGELOG.md 是**给用户的**，不是给贡献者的。像产品发布说明一样写它:

- 首先说明用户现在可以**做**以前做不到的事情。推销该功能。
- 使用简单的语言，而不是实现细节。"你现在可以..."而不是"重构了..."
- 将贡献者/内部更改放在底部的单独"致贡献者"部分。
- 每个条目应该让某人想到"噢不错，我想试试那个。"
- 没有行话: 说"每个问题现在告诉你你在哪个项目和分支"而不是
  "通过前导解析器标准化了 skill 模板中的 AskUserQuestion 格式。"

## 本地计划

贡献者可以将长期愿景文档和设计文档存储在 `~/.gstack-dev/plans/`。
这些是本地专用的 (不签入)。审查 TODOS.md 时，检查 `plans/` 中可能已准备好
提升为 TODOs 或实施的候选。

## E2E 评估失败归责协议

当 `/ship` 或任何其他工作流中的 E2E 评估失败时，**切勿在没有证明的情况下声称"不
与我们的更改相关"。** 这些系统有看不见的耦合 — 前导文本更改影响代理行为，
新的 helper 更改时序，重新生成的 SKILL.md 转移提示上下文。

**在将失败归因于"预先存在"之前必须:**
1. 在 main (或基础分支) 上运行相同的评估并显示它在那里也失败
2. 如果它在 main 上通过但在分支上失败 — 它**是**你的更改。追踪责任。
3. 如果无法在 main 上运行，说"未验证 — 可能或可能不相关"并将其标记
   为 PR 正文中的风险

没有收据的"预先存在"是懒惰的说法。证明它或不要说它。

## 部署到活动技能

活动技能位于 `~/.claude/skills/gstack/`。进行更改后:

1. 推送分支
2. 在技能目录中 fetch 并 reset: `cd ~/.claude/skills/gstack && git fetch origin && git reset --hard origin/main`
3. 重新构建: `cd ~/.claude/skills/gstack && bun run build`

或者直接复制二进制文件: `cp browse/dist/browse ~/.claude/skills/gstack/browse/dist/browse`
