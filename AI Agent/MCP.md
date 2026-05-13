Model Context Protocol，Anthropic 在 2024 年推出的**开放协议**，用于标准化 AI 模型与外部工具、数据源之间的通信方式
![[Pasted image 20260512173323.png]]
- **MCP Host（宿主）**：运行 AI 模型的应用，比如 Claude.ai、VS Code 插件、你自己写的 App。它负责创建和管理 MCP Client
- **MCP Client**：嵌入在 Host 里，负责和 MCP Server 保持一对一的长连接，处理协议细节
- **MCP Server**：轻量级进程，对外暴露三类能力，可以运行在本地也可以运行在远端
	- Tools：执行操作
	- Resources：只读资源
	- Prompts：预定义的工作流模板

#### 通信协议基础：JSON-RPC 2.0
MCP 底层用的是标准的 JSON-RPC 2.0 消息格式，支持三种传输方式：
- **stdio**：本地子进程，最常见，安全隔离好
- **HTTP + SSE**：远程服务器，支持服务端推送
- **WebSocket**：双向实时通信

一次典型的工具调用流程是：
> 用户提问 → LLM 决定调用工具 → Client 发送 `tools/call` 请求 → Server 执行并返回结果 → LLM 把结果整合进回答

#### 安全设计原则
MCP 的设计里有几个值得关注的安全思路：
- **最小权限**：Server 只暴露它需要暴露的能力
- **用户同意**：有副作用的 Tool 调用在执行前需要用户确认
- **隔离边界**：每个 Server 是独立进程，互相隔离
- **明确的能力声明**：Server 启动时声明自己有什么工具，不能动态扩权

#### 与Function Calling的关系
- Function Calling 是**模型层面**的能力——模型知道如何输出结构化的函数调用请求，但具体怎么执行、怎么管理工具的生命周期，完全由应用自己决定。
- MCP 是**协议层面**的标准——它规定了工具如何注册、如何通信、如何管理 session，是跨应用、跨模型可复用的。可以把 Function Calling 理解为 MCP 在执行阶段用到的底层机制之一。