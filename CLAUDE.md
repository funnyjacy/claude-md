1\.所有的回答使用中文，所有的claude.md文件全部使用中文

**缩写约定：** 用户说 **cc** 指 Claude Code CLI；说 **des**（没特别说明时）指 Claude Desktop。

**让 Claude「记住某件事」时，一律写进本文件（全局 CLAUDE.md），不要写进 memory 目录——memory 对用户无效。**

**编辑文字内容前，先向用户展示拟修改的版本；只有用户明确同意后，才能写入文件。**

2.下载软件优先从scoop下载；scoop没有就用winget；winget也没有再自己上网搜索下载。

3.读 PDF：本机 poppler 已通过 scoop 装好（pdftoppm 26.02.0、pdftotext、pdfinfo 均在 PATH，shims 在 <Scoop安装目录>\shims，真实目录 <Scoop安装目录>\apps\poppler\current\Library\bin），内置 Read 工具可以直接读 PDF。

  若内置 Read 报 "pdftoppm is not installed"：这是**进程内负缓存**，不是环境问题。Claude Code 的探测函数把结果存在模块级变量里（`if(cache!==undefined) return cache`），一个进程只探测一次；若某次探测发生在 poppler 装好之前，false 会被缓存到进程结束，之后装了也不会重新探测。解决办法是重启 Claude Code，不要反复重试 Read、也不要去查 PATH/shim（探测判定条件是 `exitCode===0 || stderr.length>0`，极宽松，装好后必然通过）。内置 Read 的渲染参数是 `pdftoppm -jpeg -r 100`，页数用 pdfinfo 取。

  当前进程内已被负缓存、又不想重启时的两条替代路径：

  (1) 取文字（轻量首选）：`pdftotext -layout "文件.pdf" -` 输出到 stdout。-layout 保留原始排版，表格和缩进不错乱；不加则为纯文字流。长文档用 `-f 起始页 -l 结束页` 限定范围，避免一次性灌满上下文。先 `pdfinfo "文件.pdf"` 看总页数。局限：丢失图表、公式和版式。

  (2) 看图表/公式/版式：手动渲染成 PNG 再用 Read 读图片，绕开被缓存的 PDF 分支。`pdftoppm -r 150 -f 4 -l 4 -png "文件.pdf" /tmp/p` 生成 /tmp/p-4.png，然后 Read 该 PNG。-r 150 清晰度够且体积可控（约 100KB/页），窄范围逐页渲染，用完 rm 清理。

4.读 Word/PPT/Excel：内置 Read 工具**不支持** Office 文档（docx/pptx/xlsx 本质是 zip 包，会被当二进制拒掉），不要尝试直接 Read，也不要临时写 Python 脚本。用已装好的 `office2md`（脚本在 C:\Users\<用户名>\bin\office2md，该目录已在 PATH 中）：

  ```bash
  office2md "文件.docx"              # 输出 Markdown 到 stdout
  office2md a.docx b.pptx c.xls      # 多文件，各段带 ===== 文件名 ===== 分隔
  ```

  它是对 markitdown（`pip install "markitdown[docx,pptx,xlsx,xls]"`）的封装，表格会转成 Markdown 表格，结构保留得不错。新格式 docx/pptx/xlsx 直读；老格式 doc/ppt/xls 由脚本自动调本机 Office COM（WINWORD 在 C:\Program Files\Microsoft Office\root\Office15\.先转成新格式再解析，对使用者透明。

  三个已经踩过的坑，改脚本时别改回去：
  - markitdown 单独在 bash 里跑会输出 GBK 乱码，必须设 `PYTHONUTF8=1 PYTHONIOENCODING=utf-8`（脚本内已 export）。
  - COM 调用必须用 PowerShell 7（`pwsh`），不能用系统自带的 `powershell`（5.1）——5.1 传中文路径会乱码，导致 Documents.Open 返回 null。
  - 传给 COM 的路径必须是**绝对** Windows 路径，用 `cygpath -wa`（不能用 `cygpath -w`，它对相对路径原样返回，Word 找不到文件）。

  pandoc 3.10.2 也已装好（`pandoc -t markdown 文件.docx`），排版比 markitdown 更贴近原文，但**只支持 docx，不支持 doc，也不支持 pptx/xlsx**。默认用 office2md，需要更精确的段落格式时再试 pandoc。

5.用 Docker 官方 MySQL 镜像 + /docker-entrypoint-initdb.d/ 自动导入 UTF-8 的 SQL 时，容器内 mysql 客户端默认连接字符集是 latin1，会把 UTF-8 当 latin1 再转存进 utf8mb4 列，导致中文双重编码乱码。预防：把字符集参数直接写进 docker-compose 的 command（--character-set-server=utf8mb4、--skip-character-set-client-handshake），不要用挂载 .cnf 文件的方式——Windows 挂载卷权限是 777，MySQL 安全机制会忽略任何 world-writable 的配置文件，导致 .cnf 完全不生效。skip-character-set-client-handshake 是关键（强制服务端忽略 initdb 客户端握手的 latin1）。已存坏的数据无法原地修复，只能清空数据卷（docker compose down -v）后重新初始化。验证要用 mysql --default-character-set=utf8mb4 连接读数据，若出现 è/æ 开头乱码即为双重编码。

6.给我出练习题（学习模式）时的文件格式约定，这几条是我明确反馈过的，别改回去：

  - **一道题一个文件**，命名 q1.js / q2.js……，不要把所有题塞进一个文件。三个理由：(1) 一次只跑一道题更聚焦，改错时不用重跑全部；(2) 含 setTimeout / Promise 的题会把输出顺序打乱（异步回调排在所有同步代码之后），混在一起根本看不出哪行对应哪题；(3) 我不熟悉运行方式，文件小才好定位。
  - 输出前缀统一用 `【第N题】`，一眼能对上题号。
  - **答题区集中在文件末尾**，用分隔线隔开：题干和代码在上，我的答案全部写在下面那一整块。不要把预测、追问、结论散落在代码各处 —— 那样拆得很散，看得很乱。
  - 答题区里每道追问单独成行 `Q:` / `A:`，留空行给我填。
  - 需要我填空的函数，配套的验证代码先注释掉，我写完再取消注释，避免第一次运行就抛错。
  - 每份练习开头写清运行命令（含 cd 路径），我可能不记得怎么跑。运行单个 js 文件就是 `node 文件名.js`。
  - **学习模式下他说「不会，直接给答案」时，输出在屏幕让他理解后自己写进去，不要直接改文件** —— 看答案 → 理解 → 自己写一遍，这个循环的学习效果比直接拿答案好得多。
  - **每学完一个知识点、且通过问答确认掌握之后，必须写一份笔记**，放在该学习项目的 `notes/` 目录下（如 `<学习项目目录>/notes/01-闭包.md`），并在 PROGRESS.md 里加链接。笔记的用途是**以后快速联想知识点并复习**，所以要点式、能扫读，不要写成教程重述。每份笔记包含：(1) 这东西解决什么问题、不用它会怎样；(2) 为什么重要（下游依赖什么）；(3) 最小可运行的例子（几行就够，能一眼看出机制）；(4) 我踩过的坑（从错题本里提炼）；(5) 和后端语言的对应关系。

7.学习模式的**首要原则：顺畅学习感 > 内容覆盖度**。以学生当前的实际能力来安排考验强度，不是以「这个知识点应该讲多少」来安排。

  - **如果我在某个模块卡了太久，你要主动停下来反思安排是否得当**，而不是继续往下推。卡住通常不是我笨，是内容顺序、表述方式或难度阶梯有问题。这时候要问我「是不是安排不合适？要不要换种方式」，并给出具体的改法建议。
  - **不要一上来就抛属性名和定义**。我的知识体系不完备，看到「用 flex-wrap: wrap 实现换行」这种句子只会懵——我连「换行问题」指的是什么现象都不知道。正确顺序是：**先让我看到有问题的现象 → 我用已有知识去试 → 发现不够用 → 这时才引入新属性**。这样新属性是「用来解决我刚才撞到的墙」，而不是凭空记一条规则。
  - **题干描述要具体到现象，不要用抽象术语**。「自动换行」这种标题对我没有信息量，得说清「容器变窄时，六张卡片会被压成 48px 挤在一行，我希望它们分成两行」。
  - **每一步只推进一小格**，讲完立刻让我动手，动手完立刻量结果给我看。别一次讲三个属性再让我写。
  - 挫败感一起来，学习效率就归零了——所以宁可慢，也不要让我在懵的状态下硬撑。

8\.chrome-devtools MCP 启动浏览器时，默认的 C 盘 Chrome 路径用不了，必须指定 scoop 安装的 Chrome（scoop shims 在 <Scoop安装目录>\shims）。

9\.网络搜索优先使用 **exa MCP 工具**（`mcp__plugin_exa_exa__web_search_exa`），而不是内置的 `web_search`。exa 的搜索结果质量更高、提供的 highlights 更有价值，适合深度研究和复杂查询。只有在 exa 不可用或明确不适合时，才退回 `web_search`。

11.Claude Desktop 用的是它**自带的 Claude Code**，不是 scoop 装的那个，而它自己启动时会把所有插件（MCP）都关掉。**每次会话开始前要先检查插件的开启情况**，特别是 exa（见第 9 条）——发现被关了就先提醒用户去把插件打开，不要在插件缺失的状态下直接用内置工具顶替。

12.浏览器操作工具怎么选（2026-10-28 实测结论，agent-browser 是 Vercel 出的 vercel-labs/agent-browser，Rust 原生 CLI，scoop 已装）：

  - **默认一律用 agent-browser CLI**（走 Bash），不要用 chrome-devtools MCP：它一条 Bash 能批量串多个动作（`batch` 子命令），避开 MCP 每个动作一次模型往返的开销，实测 token 开销低约 10 倍。要操作带登录态的 Chrome 先 `agent-browser connect 9222`（本机 9222 调试 Chrome 的约定见各项目 CLAUDE.md）。常用套路：`open <url>` → `snapshot -i`（交互元素 + @eN 引用）→ `click @e3` / `fill @e2 "..."` → 改动后再 `snapshot -i`。
  - **只在四个场景切回 chrome-devtools MCP**（agent-browser 没有这些）：性能追踪 Insight 解读（performance_start_trace）、Lighthouse 审计（lighthouse_audit）、CSS 级联检查（get_css_styles，能看被覆盖规则）、堆快照（take_heapsnapshot）。
  - agent-browser 独有而 chrome-devtools MCP 没有的，需要时直接用：网络 mock（`network route`）、HAR 录制、视频录制（`record`）、多会话并行、Core Web Vitals（`vitals`）、axe-core 无障碍审计（`a11y`）。
13.操作 chrome://extensions 详情页（或任何 Chrome WebUI / 深度 shadow DOM 页面）时的两个已踩过的坑（2026-10-28 同类错误连犯两次）：

  - **禁止"从标签文字向上爬 DOM 找开关"**。详情页每一行设置是独立的 shadow DOM（`<extensions-toggle-row>`），从行内文字向上 parentElement 爬会穿不透 shadow 边界，一路爬到页头抓到「已启用」总开关，得出完全错误的开关状态。正确做法：**递归枚举页面所有 `<extensions-toggle-row>`，按页面显示顺序与各行设置一一对应**，再读各自 `shadowRoot` 里的 `cr-toggle` 的 `aria-checked`（row 的 textContent 可能取不到，label 在更深的 slot 里，按顺序对应即可）。同理：篡改猴面板（chrome-extension://…/options.html）的 tab 和脚本列表也是嵌套结构，textContent.trim() 精确匹配会找不到元素，要用「文本节点递归遍历」或按行顺序对应。
  - **排查油猴脚本不生效，第一步看脚本编辑页顶部的横幅**（如「请启用"允许用户脚本"扩展设置」），这是最直接的判据；不要先去 chrome://extensions 遍历开关读状态——开关容易读串（见上条），且 Chrome 138+ 默认关闭「允许用户脚本」，不开时篡改猴装什么脚本都不会执行。

14.顺带实测（2026-10-28）：chrome-devtools MCP（连 9222 的 chrome-devtools-debug）**打不开 chrome:// 和 chrome-extension:// 页面**（明确报错拒绝），也**看不到/管理不了它自己没创建的已有标签页**；而且它和 agent-browser 同时连 9222 会互抢焦点、来回跳页。所以读扩展设置页这类任务只能用 agent-browser 的 eval（JS 遍历 shadow DOM）。

15.DOM 操作出问题时必须严谨分析，不确定就借助快照验证：写 eval 选择器/点击目标前，若对 DOM 结构不是 100% 确定（shadow DOM 边界、元素是否被遮挡、多个同名元素等），先用 `agent-browser snapshot`（或 `snapshot -i`）对照真实结构再下判断——snapshot 会正确呈现层级和遮挡关系，能提前暴露"抓错元素"的问题（如把总开关当行开关、querySelector 抓到多个 video 中的另一个）。结论要能对应到具体元素，凭"爬到的某个属性像"就下结论不算验证。

16.**生图一律用 `genimg` 脚本**（在 C:\Users\<用户名>\bin，全局可用，2026-10-29 已实测通过），不要用 SVG 转 img、也不要装 MCP 生图工具（MCP 生图输出存不了盘）：

  ```bash
  genimg "提示词" 输出.png              # 默认走阿里 qwen-image-3.0（百炼，¥0.18/张，1K/2K 同价，国内直连）
  genimg -r 16:9 "提示词" banner.png    # 指定宽高比（支持 1:1 16:9 9:16 4:3 3:4 3:2 2:3 21:9）
  genimg -e 图.jpg "编辑指令" out.png   # 编辑图片（默认模型 qwen-image-3.0 支持，¥0.2/次）
  genimg -a z-image-turbo "提示词" out.png  # 快速省钱档（¥0.1/张，不支持编辑，适合批量草稿）
  genimg -n "提示词" out.png            # Grsai nano-banana（¥0.022/张起，便宜主力，支持 -e 编辑，2026-10-29 实测通过）
  genimg -p "提示词" out.png            # 走 packy 中转 GPT（gpt-image-2，质量更高，较贵）
  genimg -p -e 图.jpg "编辑指令" out.png # GPT 编辑已有图片
  genimg -g "提示词" out.png            # 强制走 Gemini（国内需代理，免费层配额为 0，基本不用）
  ```

  - key 存放：百炼 `~/.dashscope_api_key`，packy（image 分组令牌）`~/.packy_api_key`，Grsai `~/.grsai_api_key`，Google `~/.gemini_api_key`（脚本自动兼容 UTF-16/BOM）。
  - **后端选择规则（用户 2026-11-10 指定）：默认一律走 packy 中转 GPT，即加 `-p` 参数（gpt-image-2）**——用户点名更换，此前的默认（阿里 qwen-image-3.0，不带参数直跑，¥0.18/张）降级为兜底选项。批量草稿要快/省时可用 `genimg -a z-image-turbo`（¥0.1/张）。也可用 `-a <模型名>` 指定其他阿里模型，或 `GENIMG_DS_MODEL` 环境变量改阿里默认。只有用户点名时才换后端——点名 **Gemini / nano banana 生图用 `-n`（Grsai）**。`-g`（官方 Gemini API）免费层配额为 0，基本不用。Grsai 默认模型 nano-banana-2-lite，可用 `GENIMG_GRS_MODEL` 换 nano-banana-2/pro 或 gpt-image-2。中转 GPT 生成较慢（1-2 分钟），脚本超时已设 300s。wan 系列走异步任务接口，genimg 暂不支持。
  - Grsai 已知特性（2026-10-29 实测）：接口是自有格式（提交 `/v1/draw/nano-banana` + 轮询 `/v1/draw/result`，gpt 系模型走 `/v1/draw/completions`），约 25-40 秒出图；编辑用本地图转 base64 data URI 放进 `urls` 字段直传；**仅 failure_reason=error 返还积分，违规不返**；低价 nano 渠道受谷歌风控影响偶有短暂异常（官网公告页可查），异常时换回 `-p` 兜底。
  - 生成后用 Read 工具读图确认效果，不满意就改提示词重跑（GPT ¥0.2/次左右、qwen ¥0.18/张、z-image-turbo ¥0.1/张，迭代成本不高）。
  - 注意：用 `echo 'key' > 文件` 写 key 必须在 bash 里执行，PowerShell 的 `>` 会写成 UTF-16（脚本虽已兼容，但别添乱）。
  - 大坑（已修，改脚本别改回去）：packy 的 edits 接口用 `curl -F "prompt=中文"` 时提示词会因 Windows 命令行编码被静默吞掉——HTTP 200 正常返回图但内容不变，极难察觉。必须把提示词写进临时文件用 `-F "prompt=<文件"` 传入（脚本内已实现）；纯 JSON 体的接口（generations、dashscope）用 `--data @文件` 不受此坑影响。
  - 已知限制：阿里内容审核较严，航天员/军事等题材可能报 DataInspectionFailed，换措辞即可；新百炼账号并发限流严，批量生图要串行不能并行。

16\.给别人的仓库提 PR 时，标题和描述要**简洁说清改动内容和动机**（改了什么、为什么改、怎么验证的），不要只丢一个"Update xxx.md"之类没信息量的标题——描述清楚能明显提高作者快速合并的概率。正文长不等于好，几行要点即可；涉及 UI 改动附截图。**但 PR/issue 的标题和正文必须先完整展示给用户确认，用户明确同意后才能提交。**

18.修正已提交的 PR 描述用 `gh pr edit N --body-file 文件`——`gh` 不支持 `--body @文件` 语法，那样会把路径字符串当正文原样提交（2026-11-13 在 hongtaoh PR #283 上踩过）。

19.调整前端样式/布局时，**必须实际查看渲染效果再交付**，不能光改代码就说完成（2026-11-13 用户明确反馈过：音乐页提示放标题下显得突兀，"一看就是没有认真写前端"）。做法：构建后用截图实际检查元素位置、间距、主题（浅色+深色）下的观感，确认布局合理、不突兀才算完成；放置位置优先考虑页面的整体视觉流（如脚注类信息放列表末尾加分隔线，而不是硬塞在标题旁边）。

20.**PR 定稿前的多视角审计**（2026-11-16 教训：单一上下文自查漏掉了 macOS 兼容问题，靠 reviewer 的 Gemini 独立审计才发现）。凡是给外部仓库提交的 PR，在 commit/push 之前必须做一轮多视角 agent 审计：spawn 几个**互相独立的 review agent，各分配一个视角**，不得共用上下文——(1) 依赖合约视角：读 Cargo.toml/package.json 里钉死依赖的**真实源码**（本地 cargo/pnpm checkout，不要靠记忆），逐个校验传入数据是否满足其校验函数；(2) 平台视角 ×3：macOS/Linux/Windows 各一个，只回答"这条改动的链路在我的平台上会怎样"（无法真跑的平台就把平台约束转成对源码/文档的断言）；(3) 前端-后端边界视角：两端对同一数据的假设是否一致。分歧点才人工看。审计结论也要验证——agent 报的问题同样可能是错的。
