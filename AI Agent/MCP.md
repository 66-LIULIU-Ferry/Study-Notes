Model Context Protocol，Anthropic 在 2024 年推出的**开放协议**，用于标准化 AI 模型与外部工具、数据源之间的通信方式
![[Pasted image 20260512173323.png]]
- **MCP Host（宿主）**：运行 AI 模型的应用
- **MCP Client**：嵌入在 Host 里，内有MCP Tool，解析工具名称和参数
- **MCP Server**：处理请求，调用真实API

#### 通信协议基础：JSON-RPC 2.0
MCP 底层用的是标准的 JSON-RPC 2.0 消息格式，支持三种传输方式：
- **stdio**：本地子进程，最常见，安全隔离好
- **HTTP + SSE**：远程服务器，支持服务端推送
- **WebSocket**：双向实时通信

一次典型的工具调用流程是：
> 用户提问 → LLM 决定调用工具 → Client 发送 `tools/call` 请求 → Server 执行并返回结果 → LLM 把结果整合进回答

#### MCP集成（案例：智能旅行规划助手）
```python
# 创建MCP工具 
mcp_tool = MCPTool( 
	name="amap_mcp", 
	command="npx", 
	args=["-y", "@sugarforever/amap-mcp-server"], 
	env={"AMAP_API_KEY": settings.amap_api_key}, 
	auto_expand=True 
)

# 创建一个MCPTool 
mcp_tool = MCPTool(..., auto_expand=True) 
agent.add_tool(mcp_tool)
```
![[Pasted image 20260523160226.png]]
1. 当Agent生成工具调用标记，HelloAgents框架会解析这个标记，提取工具名称和参数，然后调用对应的Tool对象
2. Tool对象是`MCPTool`自动创建的，它会把调用请求发送给MCP服务器
3. MCP服务器接收到这个消息后，解析参数，然后调用高德地图的HTTP API。它会构造 HTTP 请求，添加 API 密钥，发送请求，接收响应
4. 高德地图 API 返回 JSON 格式的数据，包含景点列表、地址、坐标等信息。MCP 服务器解析这些数据，提取关键字段，然后构造响应消息，通过 stdout 返回给`MCPTool`
5. `MCPTool`接收到响应，提取文本内容，返回给 Agent。Agent 把这个结果作为工具调用的输出，继续生成最终的回复
##### 共享MCP实例
在多Agent系统中，Agent之间都共享一个MCPTool实例。
- 因为如果每个Agent都创建一个实例，那么就会有三个服务器进程同时运行，每个进程都会独立调用高德地图API，可能会超过API的速率限制，而且多个进程会占用更多的内存和CPU资源
- 如果所有Agent共享一个MCPTool，这样所有的API调用都通过这个进程进行，可以节省资源

#### 安全设计原则
MCP 的设计里有几个值得关注的安全思路：
- **最小权限**：Server 只暴露它需要暴露的能力
- **用户同意**：有副作用的 Tool 调用在执行前需要用户确认
- **隔离边界**：每个 Server 是独立进程，互相隔离
- **明确的能力声明**：Server 启动时声明自己有什么工具，不能动态扩权

#### 与Function Calling的关系
- Function Calling 是**模型层面**的能力——模型知道如何输出结构化的函数调用请求，但具体怎么执行、怎么管理工具的生命周期，完全由应用自己决定。
- MCP 是**协议层面**的标准——它规定了工具如何注册、如何通信、如何管理 session，是跨应用、跨模型可复用的。可以把 Function Calling 理解为 MCP 在执行阶段用到的底层机制之一。