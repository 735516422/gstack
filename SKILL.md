---
name: gstack
version: 1.1.0
description: |
  用于 QA 测试和网站自用的快速无头浏览器。导航任何 URL，与元素交互，
  验证页面状态，比较操作前后的差异，拍摄带注释的截图，检查
  响应式布局，测试表单和上传，处理对话框，以及断言元素状态。
  每条命令约 100ms。当你需要测试功能、验证部署、自用用户流程，
  或用证据提交 bug 时使用。
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion

---
<!-- 从 SKILL.md.tmpl 自动生成 — 不要直接编辑 -->
<!-- 重新生成: bun run gen:skill-docs -->

## 前导 (首先运行)

```bash
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || .claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
_SESSIONS=$(find ~/.gstack/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find ~/.gstack/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(~/.claude/skills/gstack/bin/gstack-config get gstack_contributor 2>/dev/null || true)
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
```

如果输出显示 `UPGRADE_AVAILABLE <old> <new>`: 阅读 `~/.claude/skills/gstack/gstack-upgrade/SKILL.md` 并遵循"内联升级流程"(如果配置则自动升级，否则 AskUserQuestion 有 4 个选项，如果被拒绝则写入暂停状态)。如果 `JUST_UPGRADED <from> <to>`: 告诉用户"正在运行 gstack v{to} (刚刚更新!)"并继续。

## AskUserQuestion 格式

**每次 AskUserQuestion 调用务必遵循此结构:**
1. **重新落地:** 说明项目、当前分支 (使用前导打印的 `_BRANCH` 值 — 不是对话历史或 gitStatus 中的任何分支) 以及当前计划/任务。(1-2 句话)
2. **简化:** 用普通中文解释问题，聪明的 16 岁青少年可以理解。没有原始函数名，没有内部行话，没有实现细节。使用具体的例子和类比。说明它**做**什么，而不是它叫什么。
3. **推荐:** `RECOMMENDATION: 选择 [X] 因为 [一行理由]`
4. **选项:** 字母选项: `A) ... B) ... C) ...`

假设用户 20 分钟没看过这个窗口，没有打开代码。如果你需要阅读源代码才能理解你自己的解释，那就太复杂了。

特定技能说明可能在此基线之上添加额外的格式规则。

## 贡献者模式

如果 `_CONTRIB` 是 `true`: 你处于**贡献者模式**。你是 gstack 用户，也帮助让它变得更好。

**在每个主要工作流步骤结束时** (不是在每个单条命令之后)，反思你使用的 gstack 工具。给你的体验 0 到 10 分。如果不是 10 分，想一想为什么。如果有一个明显的、可操作的 bug 或一个有洞察力的、有趣的事情，gstack 代码或 skill markdown 可以做得更好 — 提交现场报告。也许我们的贡献者会帮助让我们变得更好!

**校准 — 这就是标准:** 例如，`$B js "await fetch(...)"` 曾经失败并显示 `SyntaxError: await is only valid in async functions`，因为 gstack 没有在 async 上下文中包装表达式。很小，但输入是合理的，gstack 应该处理它 — 这就是值得提交的那种事情。比这后果更小的事情，忽略。

**不值得提交:** 用户的应用程序 bug、用户 URL 的网络错误、用户网站上的身份验证失败、用户自己的 JS 逻辑 bug。

**提交方式:** 编写 `~/.gstack/contributor-logs/{slug}.md` 并包含**下面的所有部分** (不要截断 — 包括每个部分直到日期/版本页脚):

```
# {标题}

嘿 gstack 团队 — 在使用 /{skill-name} 时遇到了这个问题:

**我试图做什么:** {用户/代理试图做什么}
**实际发生了什么:** {实际发生了什么}
**我的评分:** {0-10} — {一句话说明为什么不是 10}

## 重现步骤
1. {步骤}

## 原始输出
```
{在此粘贴实际错误或意外输出}
```

## 什么会让它成为 10
{一句话: gstack 应该有什么不同的做法}

**日期:** {YYYY-MM-DD} | **版本:** {gstack 版本} | **技能:** /{skill}
```

Slug: 小写，连字符，最多 60 个字符 (例如 `browse-js-no-await`)。如果文件已存在则跳过。每会话最多 3 个报告。内联提交并继续 — 不要停止工作流。告诉用户: "已提交 gstack 现场报告: {标题}"

# gstack browse: QA 测试与自用

持久无头 Chromium。首次调用自动启动 (~3s)，然后每条命令约 100-200ms。
空闲 30 分钟后自动关闭。状态在调用之间持久化 (cookie、标签页、会话)。

## 设置 (在任何 browse 命令之前运行此检查)

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$B" ] && B=~/.claude/skills/gstack/browse/dist/browse
if [ -x "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

如果 `NEEDS_SETUP`:
1. 告诉用户: "gstack browse 需要一次性构建 (~10 秒)。可以继续吗?" 然后停止并等待。
2. 运行: `cd <SKILL_DIR> && ./setup`
3. 如果没有安装 `bun`: `curl -fsSL https://bun.sh/install | bash`

## 重要事项

- 通过 Bash 使用编译的二进制文件: `$B <command>`
- 切勿使用 `mcp__claude-in-chrome__*` 工具。它们慢且不可靠。
- 浏览器在调用之间持久化 — cookie、登录会话和标签页会延续。
- 对话框 (alert/confirm/prompt) 默认自动接受 — 没有浏览器锁定。

## QA 工作流

### 测试用户流程 (登录、注册、结账等)

```bash
# 1. 转到页面
$B goto https://app.example.com/login

# 2. 查看可交互的内容
$B snapshot -i

# 3. 使用引用填写表单
$B fill @e3 "test@example.com"
$B fill @e4 "password123"
$B click @e5

# 4. 验证它是否有效
$B snapshot -D              # diff 显示点击后发生了什么变化
$B is visible ".dashboard"  # 断言仪表板出现了
$B screenshot /tmp/after-login.png
```

### 验证部署 / 检查生产环境

```bash
$B goto https://yourapp.com
$B text                          # 读取页面 — 它加载了吗?
$B console                       # 有 JS 错误吗?
$B network                       # 有失败的请求吗?
$B js "document.title"           # 标题正确吗?
$B is visible ".hero-section"    # 关键元素存在吗?
$B screenshot /tmp/prod-check.png
```

### 端到端自用功能

```bash
# 导航到功能
$B goto https://app.example.com/new-feature

# 拍摄带注释的截图 — 显示每个带标签的可交互元素
$B snapshot -i -a -o /tmp/feature-annotated.png

# 查找所有可点击的内容 (包括带有 cursor:pointer 的 div)
$B snapshot -C

# 走过流程
$B snapshot -i          # 基线
$B click @e3            # 交互
$B snapshot -D          # 什么改变了? (统一 diff)

# 检查元素状态
$B is visible ".success-toast"
$B is enabled "#next-step-btn"
$B is checked "#agree-checkbox"

# 交互后检查控制台错误
$B console
```

### 测试响应式布局

```bash
# 快速: 移动/平板/桌面 3 张截图
$B goto https://yourapp.com
$B responsive /tmp/layout

# 手动: 特定视口
$B viewport 375x812     # iPhone
$B screenshot /tmp/mobile.png
$B viewport 1440x900    # 桌面
$B screenshot /tmp/desktop.png

# 元素截图 (裁剪到特定元素)
$B screenshot "#hero-banner" /tmp/hero.png
$B snapshot -i
$B screenshot @e3 /tmp/button.png

# 区域裁剪
$B screenshot --clip 0,0,800,600 /tmp/above-fold.png

# 仅视口 (无滚动)
$B screenshot --viewport /tmp/viewport.png
```

### 测试文件上传

```bash
$B goto https://app.example.com/upload
$B snapshot -i
$B upload @e3 /path/to/test-file.pdf
$B is visible ".upload-success"
$B screenshot /tmp/upload-result.png
```

### 测试带验证的表单

```bash
$B goto https://app.example.com/form
$B snapshot -i

# 提交空 — 检查验证错误是否出现
$B click @e10                        # 提交按钮
$B snapshot -D                       # diff 显示错误消息出现
$B is visible ".error-message"

# 填写并重新提交
$B fill @e3 "valid input"
$B click @e10
$B snapshot -D                       # diff 显示错误消失，成功状态
```

### 测试对话框 (删除确认、提示)

```bash
# 在触发之前设置对话框处理
$B dialog-accept              # 将自动接受下一个 alert/confirm
$B click "#delete-button"     # 触发确认对话框
$B dialog                     # 查看出现什么对话框
$B snapshot -D                # 验证项目已删除

# 对于需要输入的提示
$B dialog-accept "my answer"  # 接受并输入文本
$B click "#rename-button"     # 触发提示
```

### 测试经过身份验证的页面 (导入真实浏览器 cookie)

```bash
# 从真实浏览器导入 cookie (打开交互式选择器)
$B cookie-import-browser

# 或直接导入特定域
$B cookie-import-browser comet --domain .github.com

# 现在测试经过身份验证的页面
$B goto https://github.com/settings/profile
$B snapshot -i
$B screenshot /tmp/github-profile.png
```

### 比较两个页面 / 环境

```bash
$B diff https://staging.app.com https://prod.app.com
```

### 多步骤链 (对长流程高效)

```bash
echo '[
  ["goto","https://app.example.com"],
  ["snapshot","-i"],
  ["fill","@e3","test@test.com"],
  ["fill","@e4","password"],
  ["click","@e5"],
  ["snapshot","-D"],
  ["screenshot","/tmp/result.png"]
]' | $B chain
```

## 快速断言模式

```bash
# 元素存在且可见
$B is visible ".modal"

# 按钮启用/禁用
$B is enabled "#submit-btn"
$B is disabled "#submit-btn"

# 复选框状态
$B is checked "#agree"

# 输入可编辑
$B is editable "#name-field"

# 元素有焦点
$B is focused "#search-input"

# 页面包含文本
$B js "document.body.textContent.includes('Success')"

# 元素计数
$B js "document.querySelectorAll('.list-item').length"

# 特定属性值
$B attrs "#logo"    # 以 JSON 形式返回所有属性

# CSS 属性
$B css ".button" "background-color"
```

## Snapshot 系统

Snapshot 是你理解和与页面交互的主要工具。

```
-i        --interactive           仅可交互元素 (按钮、链接、输入)，带 @e 引用
-c        --compact               紧凑 (无空结构节点)
-d <N>    --depth                 限制树深度 (0 = 仅根，默认: 无限制)
-s <sel>  --selector              限定到 CSS 选择器
-D        --diff                  与前一个 snapshot 的统一 diff (首次调用存储基线)
-a        --annotate              带红色叠加框和引用标签的带注释截图
-o <path> --output                带注释截图的输出路径 (默认: /tmp/browse-annotated.png)
-C        --cursor-interactive    光标可交互元素 (@c 引用 — 带有 pointer, onclick 的 div)
```

所有标志可以自由组合。`-o` 仅在也使用 `-a` 时适用。
例如: `$B snapshot -i -a -C -o /tmp/annotated.png`

**引用编号:** @e 引用按树顺序顺序分配 (@e1, @e2, ...)。
来自 `-C` 的 @c 引用单独编号 (@c1, @c2, ...)。

Snapshot 后，在任何命令中使用 @refs 作为选择器:
```bash
$B click @e3       $B fill @e4 "value"     $B hover @e1
$B html @e2        $B css @e5 "color"      $B attrs @e6
$B click @c1       # 光标可交互引用 (来自 -C)
```

**输出格式:** 带有 @ref ID 的缩进可访问性树，每行一个元素。
```
  @e1 [heading] "欢迎" [level=1]
  @e2 [textbox] "电子邮件"
  @e3 [button] "提交"
```

导航时引用失效 — 在 `goto` 后再次运行 `snapshot`。

## 命令参考

### 导航
| 命令 | 描述 |
|---------|-------------|
| `back` | 历史后退 |
| `forward` | 历史前进 |
| `goto <url>` | 导航到 URL |
| `reload` | 重新加载页面 |
| `url` | 打印当前 URL |

### 读取
| 命令 | 描述 |
|---------|-------------|
| `accessibility` | 完整 ARIA 树 |
| `forms` | 表单字段为 JSON |
| `html [selector]` | 选择器的 innerHTML (如果未找到则抛出)，如果没有给定选择器则为完整页面 HTML |
| `links` | 所有链接为 "文本 → href" |
| `text` | 清理的页面文本 |

### 交互
| 命令 | 描述 |
|---------|-------------|
| `click <sel>` | 点击元素 |
| `cookie <name>=<value>` | 在当前页域设置 cookie |
| `cookie-import <json>` | 从 JSON 文件导入 cookie |
| `cookie-import-browser [browser] [--domain d]` | 从 Comet、Chrome、Arc、Brave 或 Edge 导入 cookie (打开选择器，或使用 --domain 直接导入) |
| `dialog-accept [text]` | 自动接受下一个 alert/confirm/prompt。可选文本作为提示响应发送 |
| `dialog-dismiss` | 自动忽略下一个对话框 |
| `fill <sel> <val>` | 填写输入 |
| `header <name>:<value>` | 设置自定义请求头 (冒号分隔，敏感值自动编辑) |
| `hover <sel>` | 悬停元素 |
| `press <key>` | 按键 — Enter、Tab、Escape、ArrowUp/Down/Left/Right、Backspace、Delete、Home、End、PageUp、PageDown，或 Shift+Enter 等修饰键 |
| `scroll [sel]` | 将元素滚动到视图中，如果没有选择器则滚动到页面底部 |
| `select <sel> <val>` | 通过值、标签或可见文本选择下拉选项 |
| `type <text>` | 在聚焦元素中输入 |
| `upload <sel> <file> [file2...]` | 上传文件 |
| `useragent <string>` | 设置用户代理 |
| `viewport <WxH>` | 设置视口大小 |
| `wait <sel|--networkidle|--load>` | 等待元素、网络空闲或页面加载 (超时: 15s) |

### 检查
| 命令 | 描述 |
|---------|-------------|
| `attrs <sel|@ref>` | 元素属性为 JSON |
| `console [--clear|--errors]` | 控制台消息 (--errors 过滤到错误/警告) |
| `cookies` | 所有 cookie 为 JSON |
| `css <sel> <prop>` | 计算的 CSS 值 |
| `dialog [--clear]` | 对话框消息 |
| `eval <file>` | 从文件运行 JavaScript 并将结果作为字符串返回 (路径必须在 /tmp 或 cwd 下) |
| `is <prop> <sel>` | 状态检查 (visible/hidden/enabled/disabled/checked/editable/focused) |
| `js <expr>` | 运行 JavaScript 表达式并将结果作为字符串返回 |
| `network [--clear]` | 网络请求 |
| `perf` | 页面加载时间 |
| `storage [set k v]` | 将所有 localStorage + sessionStorage 读取为 JSON，或设置 <key> <value> 写入 localStorage |

### 视觉
| 命令 | 描述 |
|---------|-------------|
| `diff <url1> <url2>` | 页面之间的文本差异 |
| `pdf [path]` | 保存为 PDF |
| `responsive [prefix]` | 移动 (375x812)、平板 (768x1024)、桌面 (1280x720) 的截图。保存为 {prefix}-mobile.png 等 |
| `screenshot [--viewport] [--clip x,y,w,h] [selector|@ref] [path]` | 保存截图 (支持通过 CSS/@ref 元素裁剪、--clip 区域、--viewport) |

### Snapshot
| 命令 | 描述 |
|---------|-------------|
| `snapshot [flags]` | 带有用于元素选择的 @e 引用的可访问性树。标志: -i 仅可交互、-c 紧凑、-d N 深度限制、-s sel 范围、-D 与前一个 diff、-a 带注释截图、-o 路径输出、-C 光标可交互 @c 引用 |

### 元
| 命令 | 描述 |
|---------|-------------|
| `chain` | 从 JSON stdin 运行命令。格式: [["cmd","arg1",...],...] |

### 标签页
| 命令 | 描述 |
|---------|-------------|
| `closetab [id]` | 关闭标签页 |
| `newtab [url]` | 打开新标签页 |
| `tab <id>` | 切换到标签页 |
| `tabs` | 列出打开的标签页 |

### 服务器
| 命令 | 描述 |
|---------|-------------|
| `restart` | 重启服务器 |
| `status` | 健康检查 |
| `stop` | 关闭服务器 |

## 提示

1. **导航一次，查询多次。** `goto` 加载页面；然后 `text`、`js`、`screenshot` 都立即命中已加载的页面。
2. **先使用 `snapshot -i`。** 查看所有可交互元素，然后通过引用点击/填写。没有 CSS 选择器猜测。
3. **使用 `snapshot -D` 验证。** 基线 → 操作 → diff。确切地看到什么改变了。
4. **使用 `is` 进行断言。** `is visible .modal` 比解析页面文本更快更可靠。
5. **使用 `snapshot -a` 作为证据。** 带注释的截图非常适合 bug 报告。
6. **使用 `snapshot -C` 处理棘手的 UI。** 查找可访问性树遗漏的可点击 div。
7. **操作后检查 `console`。** 捕获不会在视觉上显示的 JS 错误。
8. **对长流程使用 `chain`。** 单个命令，无每步 CLI 开销。
