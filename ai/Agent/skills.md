
https://mp.weixin.qq.com/s/jUylk813LYbKw0sLiIttTQ
Skill 就像给 Agent 准备的工作流程 SOP。
- MCP 是一种开放标准的协议，关注的是 AI 如何以统一方式调用外部的工具、数据和服务，本身不定义任务逻辑或执行流程。
- Skill 则教 Agent 如何完整处理特定工作，它将执行方法、工具调用方式以及相关知识材料，封装为一个完整的「能力扩展包」，使 Agent 具备稳定、可复用的做事方法。

![Agent-vs-MCP-vs-Skills](./imsges/Agent-vs-MCP-vs-Skills.png) 
MCP 提供能做什么（底层工具能力）
Skill 提供怎么做（执行方法与流程）
Agent 负责整合并落地做什么、怎么做（智能决策与执行）



## OpenClaw
- 核心优势：local first + 较高权限；
- 较高权限意味着安全风险较高。

https://www.51cto.com/article/836597.html
OpenClaw 选用的底层框架 pi-mono，在我看来就是"less structure, more intelligence"这种工程哲学最极端的实践，核心只有四个工具，系统提示词不到一千个 token，完全不依赖 LangChain / LangGraph。
- read   → 读取文件内容（文本/图片，可指定行范围）
- write  → 创建新文件或完全重写（自动创建目录）
- edit   → 精确替换文本（oldText 必须完全匹配）
- bash   → 执行命令（返回 stdout + stderr，可设超时）

背后的逻辑说白了就一句话：写代码这件事拆到底，就是读、写、改、跑四个动作。你让 AI 去理解一个项目，它自然会先 read 几个核心文件；发现 bug 了，edit 精确改指定行然后 bash 跑一把测试看看过不过；要做大重构，就 write 整个新文件覆盖掉旧的。日常编程 90% 的操作其实都能拆成这四个原语的排列组合。

#### 四层架构
除了极简的工具设计，pi-mono 在技术栈上也做了清晰的分层。
https://github.com/badlogic/pi-mono


OpenClaw 的核心是一个 Gateway，但不是那种轻量的 API 网关，而是一个长生命周期的守护进程（daemon）：
Gateway 做的事情包括：维护各渠道连接、暴露 typed WebSocket API、做 schema 校验、发事件流（agent / chat / presence / cron…）。它用 TypeBox 做协议/数据结构的单一事实源，驱动校验和代码生成。
一个很重要的工程决策是 Lane Queue 串行执行。即使用户在多个渠道发消息，核心的 Agent Loop 也是有序处理的，防止 AI 在处理复杂任务时回复错乱。

记忆系统：用 Markdown 文件做记忆
OpenClaw 的记忆系统是我觉得设计得最巧妙的部分。做 RAG 项目的人都知道，记忆存储通常意味着向量库、嵌入模型、复杂的检索策略。但 OpenClaw 走了一条完全不同的路：直接用文件系统里的 Markdown 文件当记忆。

从部署来看，~/.openclaw/workspace/ 下是这样的结构：
```
~/.openclaw/workspace/
├── IDENTITY.md    # Agent 身份定义
├── SOUL.md        # 行为准则（"灵魂"）
├── USER.md        # 用户画像（你告诉它关于你的信息）
├── AGENTS.md      # Agent 能力配置
├── TOOLS.md       # 工具列表
├── BOOTSTRAP.md   # 启动引导
├── glossary.md    # 热词表（语音识别纠错用）
└── HEARTBEAT.md   # 心跳配置
```
记忆分两层，说白了就是日记本 + 备忘录：
短期记忆：memory/YYYY-MM-DD.md，每天一个文件，对话和操作记录持续追加长期记忆：MEMORY.md，你可以直接打开这个文件编辑
舒服的是透明性。想知道 AI 记住了什么？打开文件看一眼就行。想让它忘掉某件事？删掉那行字。做过 RAG 项目的都知道，向量库里存的东西用户基本看不懂也改不了，客户经常问“AI 到底记住了什么”，你很难给出一个直观的答案。OpenClaw 这个设计就轻松解决了这个问题。

身份系统的分离设计也值得提一下：把 Agent 的灵魂（SOUL.md，行为哲学）、身份（IDENTITY.md，对外呈现）、能力（AGENTS.md / TOOLS.md，工具配置）做了清晰的文件级分离。这不仅方便调试，更方便二次开发，也就是改一个文件就能改变 Agent 的一个维度，不影响其他维度。

OpenClaw 成功的本质不是技术上有什么突破，而是一个有足够工程能力和产品品味的个人开发者，做了一件大公司不屑于做、创业公司不划算做的事情。它的壁垒也不在代码里的某个算法，而在于那 1.3 万+ 个 commits 积累出来的产品完成度和工程细节。


LangGraph 管理"确定性的复杂"，OpenClaw 管理"不确定性的简单"。
编排型 vs 自主型
拿之前在公众号里分享过的报价项目来说，流程是确定的：用户上传招标文件 → 解析意图 → 从 7000+ 历史报价中检索最相似的 → 结合 110 个 SKU 库校验 → 用模板 + JSON 填充生成 Excel。每个节点的输入输出都是明确的，中间还有循环节点（SKU 校验失败→重试、追问→重新解析），用 LangGraph 的有向图来管理状态流转非常合适。
而 OpenClaw 的场景完全不同。用户可能说"帮我查一下 XX 公司的工商信息”，也可能说“把昨天那份合同第三条改一下”。Agent 自己判断要调哪些工具，不需要预定义状态图。
简单说，LangGraph 是你告诉 Agent 该怎么走（编排型），OpenClaw 是Agent 自己决定怎么走（自主型）。pi-mono 适合要完全掌控上下文、知道自己在干啥的开发者。NanoClaw 适合安全和信任是底线、宁可少功能也不愿意裸奔的技术用户。