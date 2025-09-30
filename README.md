**阅读目录**

* [〇、前言](#_label0)
* [一、MCP 简介](#_label1)
  + [1.1 什么是 MCP？](#_label1_0)
  + [1.2 核心架构：Host-Client-Server](#_label1_1)
  + [1.3 MCP 是如何工作的？](#_label1_2)
* [二、Windows 环境，基于 Ollama 和 .net 的简单示例](#_label2)
  + [2.1 Ollama 的安装和大模型下载](#_label2_0)
  + [2.2 简单调用示例（.net 8.0）【完整代码】【使用旧版的大模型：qwen:7b】](#_label2_1)
    - [2.2.1 服务端代码](#_label2_1_0)
    - [2.2.2 客户端代码](#_label2_1_1)
    - [2.2.3 测试结果](#_label2_1_2)
  + [2.3 使用新版的千问大模型测试：qwen3:8b【推荐使用】](#_label2_2)

---

[回到顶部](#_labelTop)

## 〇、前言

从 2020 年到 2025 年，AI 在多个维度实现了跨越式发展。

从 ChatGPT 到 DeepSeek 等大型语言模型（LLM），以及多模态融合，从一开始的文本到后续的图片、文档、视频，还有辅助编程等等，都在说明 AI 不是“未来趋势”，而是正在重塑开发方式的“当下现实”。

因此多接触一些 AI 方面的知识，变的很有必要。

那么本文将就 MCP 这一块，先做下简单介绍 MCP 是什么，然后通过简单的示例，来看下怎么用。

[回到顶部](#_labelTop):[flyint](https://huashutang.com)

## 一、MCP 简介

### 1.1 什么是 MCP？

**MCP（Model Context Protocol：模型上下文协议）**是由 Anthropic 于 2024 年 11 月推出的开源协议，旨在为【大型语言模型（Large Language Model：LLM）】与【外部数据源、工具和服务】提供**标准化、安全的双向通信接口**。

MCP 可以**将 AI 模型与外部世界的连接**简化为**“即插即用”的接口**，类似 USB-C 的通用性，避免为每个工具单独定制集成方案。

它可以通过统一协议，实现 AI 模型与数据库、API、文件系统等的无缝协作，解决传统“碎片化集成”的痛点。

![image](https://img2024.cnblogs.com/blog/1868241/202509/1868241-20250908172759694-1476216000.png)

四大优势：

* **标准化（Standardization）**

MCP 遵循统一的技术规范和接口定义，确保不同系统、组件或服务之间的兼容性。标准化可以带来的好处：

**降低集成成本：**通过定义清晰的数据格式（如JSON Schema、Protobuf）、API规范（如OpenAPI）和通信协议（如REST、gRPC），MCP使得异构系统能够快速对接。**提升可维护性：**标准化的接口和日志格式便于监控、调试和团队协作。**促进生态发展：**第三方开发者可基于标准快速开发插件或扩展，形成丰富的工具链。**类比技术：**类似HTTP/REST在Web服务中的角色，或Kubernetes的CRD（自定义资源定义）标准。

* **安全性（Security）**

MCP **内置多层次安全机制**，保障数据传输、存储和访问控制的安全。好处如下：

**端到端加密：**支持TLS/SSL等加密传输，防止数据泄露。**身份认证与授权：**集成OAuth2、JWT、RBAC（基于角色的访问控制）等机制，确保只有合法用户和系统可访问资源。**审计与合规：**记录操作日志，满足GDPR、等保等法规要求。**防御常见攻击：**如防重放攻击、DDoS缓解、输入验证等。**应用场景：**适用于金融、医疗等对数据隐私要求高的行业。

* **灵活性（Flexibility）**

MCP 支持多种部署模式、可扩展架构和动态配置，适应不同业务需求。好处如下：

**模块化设计：**功能组件可插拔，用户可根据需要启用或替换模块。**支持多语言/多框架：**提供 SDK 或 API 支持 Python、Java、Go 等多种语言，便于现有系统集成。**动态配置与热更新：**无需重启服务即可调整参数或升级功能。**适应不同规模：**从小型应用到大规模分布式系统均可部署。**类比架构：**类似于微服务网关（如Istio）或消息中间件（如Kafka）的灵活集成能力。

* **跨平台（Cross-Platform）**

MCP 可在不同操作系统、硬件架构和云环境中运行。

**操作系统兼容：**支持 Windows、Linux、macOS、嵌入式 RTOS 等。**云原生支持：**可在 AWS、Azure、阿里云等公有云及私有云环境中部署，支持容器化（Docker）和编排（k8s）。**边缘计算兼容：**可在边缘设备（如 IoT 网关）与中心云之间无缝协同。**统一管理界面：**无论底层平台如何，提供一致的管理、监控和开发体验。**技术基础：**通常基于跨平台运行时（如 Java JVM、.NET Core）或轻量级协议（如 MQTT、CoAP）。

MCP 通过**标准化接口**和**客户端-服务器架构**，解决了LLM与外部工具集成的难题。它不仅让 LLM 能够访问本地和远程资源，还通过动态上下文管理和安全机制，提升了AI应用的灵活性和可靠性。

随着MCP 生态的扩展，开发者可以更高效地构建智能应用，推动AI技术的实际落地。

### 1.2 核心架构：Host-Client-Server

MCP 采用**客户端-服务器架构（Client-Server Architecture）**，由以下三类组件构成：

* **MCP Host（主机）**

它是发起请求的 AI 应用程序，例如聊天机器人、AI 驱动的 IDE（如 Cursor）、Claude Desktop 等。

主要职责：接收用户输入，与 LLM 交互生成指令，并在 LLM 需要调用外部工具时触发 MCP 流程。

* **MCP Client（客户端）**

也可以叫做**“协议转换层”**，作为 MCP Host 与 MCP Server 之间的**桥梁**。

主要职责：

与 MCP Server 建立持久连接（如通过 JSON-RPC 2.0 协议）。将 LLM 的请求翻译为 MCP 标准格式，并发送给 MCP Server。接收 MCP Server 的响应，返回给 MCP Host。

* **MCP Server（服务器）**

也可以叫做**“功能提供层”**，封装具体的数据源或工具的访问逻辑。

主要职责：

提供标准化的接口，支持访问本地资源（如文件系统、数据库）或远程服务（如 API、SaaS 平台）。执行具体的工具调用（如查询数据库、发送邮件），并返回结果。

关系图如下：

![image](https://img2024.cnblogs.com/blog/1868241/202509/1868241-20250912192343988-242763048.png)

### 1.3 MCP 是如何工作的？

MCP 的工作流程大概可以分为如下几个步骤。

1）建立连接

MCP Client 与 MCP Server 建立通信连接，通常通过以下方式：

本地通信（STDIO）：适用于同一设备内的进程间通信（如 AI 助手调用本地文件）。远程通信（HTTP+SSE）：适用于跨网络访问远程服务（如调用云 API）。

2）资源发现

MCP Client 向 MCP Server 查询可用的工具和数据源，获取其功能描述和接口规范（如 OpenAPI 文档）。示例：AI 助手需要查询天气时，MCP Client 会找到支持天气查询的 MCP Server，并获取其 API 参数。

3）工具调用

MCP Host（通过 LLM）解析用户需求，生成调用外部工具的指令。

MCP Client 将指令转换为 MCP 标准格式（JSON-RPC 2.0），并发送给 MCP Server。

示例：用户要求“查询北京的天气”，LLM 生成调用天气 API 的指令，MCP Client 将其发送到天气 MCP Server。

4）数据交互

MCP Server 执行具体的工具调用（如访问数据库、调用 API），获取或更新数据。

安全机制：MCP Server 通过权限控制和沙箱机制，确保 LLM 只能访问授权的资源。

5）结果返回

MCP Server 将处理结果返回给 MCP Client。

MCP Client 将结果转换为 LLM 可理解的格式，并传递给 MCP Host。

MCP Host 将结果呈现给用户或用于后续处理。

**这过程中涉及到的通信协议主要有三种：**

1）JSON-RPC 2.0 轻量级、跨语言支持，适合远程过程调用（RPC）

```
|  |  |
| --- | --- |
|  | { |
|  | "jsonrpc": "2.0", |
|  | "method": "get_weather", |
|  | "params": {"location": "Beijing"}, |
|  | "id": 1 |
|  | } |
```

2）SSE（Server-Sent Events）支持服务器向客户端的流式推送，适合实时数据交互。

应用场景：长轮询天气更新、实时日志输出。

3）STDIO（标准输入输出）适用于本地进程间通信，延迟低。

应用场景：AI助手读写本地文件或调用本地工具。

[回到顶部](#_labelTop)

## 二、Windows 环境，基于 Ollama 和 .net 的简单示例

### 2.1 Ollama 的安装和大模型下载

* **简介**

Ollama 是一个开源的本地大模型运行框架，旨在让开发者能够在本地机器上轻松地运行、管理和使用大型语言模型（LLM）。它支持多种主流模型架构（如 Llama、Qwen、Mistral、Gemma、Phi 等），提供简洁的命令行接口（CLI）和 REST API，便于集成到应用中。

其核心特点：

本地运行：所有模型都在你的设备上运行，保护数据隐私。轻量高效：优化了模型加载和推理性能，支持 GPU 加速（CUDA、Metal 等）。简单易用：通过简单的 ollama run 命令即可下载并运行模型。跨平台支持：支持 macOS、Linux 和 Windows。可定制化：支持自定义模型参数、创建 Modelfile 定制提示词、上下文长度等。

* **下载和安装**

官网下载地址*：[https://ollama.com/download](https://ollama.com/download "https://ollama.com/download")。*然后双击开始安装即可。

```
|  |  |
| --- | --- |
|  | // 查看是否安装成功 |
|  | // 命令行执行 -v： |
|  | C:\Windows\system32>ollama -v |
|  | ollama version is 0.12.0 |
|  |  |
|  | // 使用 run 命令可以安装和运行指定模型 |
|  | // 首次运行时会自动从 Ollama 的模型库下载 qwen:7b 模型文件 |
|  | // 文件约 4-5 GB，具体大小取决于量化版本，需要稳定的网络连接 |
|  | C:\Windows\system32>ollama run qwen:7b |
|  |  |
|  | // 查看已安装模型 |
|  | C:\Windows\system32>ollama list |
|  | NAME       ID              SIZE      MODIFIED |
|  | qwen:7b    2091ee8c8d8f    4.5 GB    3 days ago |
|  |  |
|  | // 运行模型 |
|  | C:\Windows\system32>ollama run qwen:7b |
|  | >>> 你是谁？ |
|  | 我是阿里云研发的大规模语言模型，我叫通义千问。 |
|  | >>> Send a message (/? for help) |
```

* **可调用的 API**

Ollama 提供了一组简洁但功能完整的 RESTful API，用于与本地运行的大模型进行交互。这些 API 默认运行在 http://localhost:11434 地址上，支持外部程序调用，实现模型推理、管理、生成等功能。

以下是 Ollama 中可供外部调用的主要 API 接口及特点：

| API 地址 | 方法 | 主要用途 | 是否流式 | 适用场景 |
| --- | --- | --- | --- | --- |
| /api/generate | POST | 单轮文本生成 | ✅ 支持 | 内容生成、简单问答 |
| /api/chat | POST | 多轮对话生成 | ✅ 支持 | 聊天机器人、对话系统 |
| /api/embeddings | POST | 文本向量化 | ❌ 不支持 | RAG、语义搜索 |
| /api/tags | GET | 列出模型 | ❌ | 模型管理前端展示 |
| /api/create | POST | 创建自定义模型 | ✅（进度流） | 模型定制 |
| /api/delete | POST | 删除模型 | ❌ | 清理资源 |
| /api/show | POST | 查看模型详情 | ❌ | 调试、集成配置 |

### 2.2 简单调用示例（.net 8.0）【完整代码】【使用旧版的大模型：qwen:7b】

首先安装包：ModelContextProtocolServer.Sse。如下包简介：

![image](https://img2024.cnblogs.com/blog/1868241/202509/1868241-20250929193521769-1386175397.png)

#### 2.2.1 服务端代码

*注意：博主在尝试通过属性：McpServerTool，来自动获取 tool 列表时没有成功，因此直接从 Controller 中通过代码获取。*

```
|  |  |
| --- | --- |
|  | // Program.cs |
|  | var builder = WebApplication.CreateBuilder(args); |
|  | // Add services to the container. |
|  | // 添加 MCP 服务器服务 |
|  | builder.Services.AddMcpServer().WithToolsFromAssembly(); // 自动注册当前程序集中的工具 |
|  | builder.Services.AddMcpServer().WithHttpTransport(); |
|  | builder.Services.AddControllers(); |
|  | builder.Services.AddEndpointsApiExplorer(); |
|  |  |
|  | var app = builder.Build(); |
|  | app.UseHttpsRedirection(); |
|  |  |
|  | app.MapGet("/", () => "MCP Time Server is running!"); |
|  | app.MapMcp(); // 映射 MCP 端点 |
|  |  |
|  | app.UseAuthorization(); |
|  | app.MapControllers(); |
|  | app.Run(); |
```

```
|  |  |
| --- | --- |
|  | // TimeTool.cs |
|  | using ModelContextProtocol.Server; |
|  | using System.ComponentModel; |
|  |  |
|  | namespace McpTimeServer.Tools; |
|  |  |
|  | [McpServerToolType, Description("Time-related tools")] |
|  | public static class TimeTool |
|  | { |
|  | [McpServerTool, Description("Get the current time for a city")] |
|  | public static string GetCurrentTime([Description("The name of the city")] string city) |
|  | { |
|  | // 简化逻辑，直接返回当前时间，仅供测试用 |
|  | var now = DateTime.Now; |
|  | return @$"{{""result"":""The current time in {city} is {now:HH:mm} on {now:yyyy-MM-dd}.""}}"; |
|  | } |
|  | } |
```

```
|  |  |
| --- | --- |
|  | // McpController.cs |
|  | using Microsoft.AspNetCore.Mvc; |
|  | using ModelContextProtocol.Server; |
|  | using System.ComponentModel; |
|  | using System.Reflection; |
|  | using System.Text.Json; |
|  |  |
|  | namespace McpTimeServer; |
|  |  |
|  | [ApiController] |
|  | [Route("[controller]")] |
|  | public class McpController : ControllerBase |
|  | { |
|  | private readonly Dictionary<string, ToolInfo> _registeredTools; |
|  | public McpController() |
|  | { |
|  | _registeredTools = DiscoverTools(); |
|  | } |
|  | /// |
|  | /// GET: /mcp/tools - 返回所有可用工具的元数据 |
|  | /// |
|  | [HttpGet("tools")] |
|  | public IActionResult GetTools() |
|  | { |
|  | var tools = _registeredTools.Values.Select(t => new |
|  | { |
|  | name = t.Name, |
|  | description = t.Description, |
|  | inputSchema = t.InputSchema |
|  | }); |
|  | return Ok(tools); |
|  | } |
|  | /// |
|  | /// POST: /mcp/invoke - 调用指定工具 |
|  | /// |
|  | [HttpPost("invoke")] |
|  | public async Task InvokeTool([FromBody] JsonElement body) |
|  | { |
|  | if (!body.TryGetProperty("name", out var nameElement) || nameElement.ValueKind != JsonValueKind.String) |
|  | return BadRequest(new { error = "Invalid or missing 'name' in request." }); |
|  | var toolName = nameElement.GetString()!; |
|  | if (!_registeredTools.TryGetValue(toolName, out var tool)) |
|  | return NotFound(new { error = $"Tool '{toolName}' not found." }); |
|  | JsonElement js = new JsonElement(); |
|  | // 提取 arguments |
|  | var arguments = body.TryGetProperty("arguments", out var args) ? args : js; |
|  | try |
|  | { |
|  | JsonElement actualArguments; |
|  | if (arguments.ValueKind == JsonValueKind.String) |
|  | { |
|  | // 如果是字符串，尝试解析它为 JSON |
|  | string jsonString = arguments.GetString(); |
|  | using JsonDocument innerDoc = JsonDocument.Parse(jsonString); |
|  | actualArguments = innerDoc.RootElement.Clone(); // Clone 以脱离 document 生命周期 |
|  | } |
|  | else |
|  | { |
|  | actualArguments = arguments; |
|  | } |
|  | // 绑定参数并调用 |
|  | var methodArgs = BindArguments(tool.Method, actualArguments); |
|  | var result = tool.Method.Invoke(null, methodArgs); // 静态方法，实例为 null |
|  | return Ok(result ?? "null"); |
|  | } |
|  | catch (TargetInvocationException ex) |
|  | { |
|  | return StatusCode(500, new { error = "Tool execution failed.", message = ex.InnerException?.Message }); |
|  | } |
|  | catch (Exception ex) |
|  | { |
|  | return StatusCode(500, new { error = "Tool binding failed.", message = ex.Message }); |
|  | } |
|  | } |
|  | /// |
|  | /// 工具列表 |
|  | /// |
|  | /// |
|  | private Dictionary<string, ToolInfo> DiscoverTools() |
|  | { |
|  | var tools = new Dictionary<string, ToolInfo>(); |
|  | var assemblies = AppDomain.CurrentDomain.GetAssemblies(); |
|  | foreach (var assembly in assemblies) |
|  | { |
|  | try |
|  | { |
|  | var types = assembly.GetTypes() |
|  | .Where(t => t.IsDefined(typeof(McpServerToolTypeAttribute), false) && t.IsAbstract && t.IsSealed); // 静态类 |
|  |  |
|  | foreach (var type in types) |
|  | { |
|  | var methods = type.GetMethods(BindingFlags.Public | BindingFlags.Static) |
|  | .Where(m => m.IsDefined(typeof(McpServerToolAttribute), false)); |
|  | foreach (var method in methods) |
|  | { |
|  | var attr = method.GetCustomAttribute()!; |
|  | var description = method.GetCustomAttribute()?.Description ?? "No description."; |
|  | var name = attr.Name ?? method.Name; |
|  |  |
|  | var inputSchema = GenerateInputSchema(method); |
|  |  |
|  | tools[name] = new ToolInfo |
|  | { |
|  | Name = name, |
|  | Method = method, |
|  | Description = description, |
|  | InputSchema = inputSchema |
|  | }; |
|  | } |
|  | } |
|  | } |
|  | catch(Exception ex) |
|  | {            } |
|  | } |
|  | return tools; |
|  | } |
|  | private object[] BindArguments(MethodInfo method, JsonElement arguments) |
|  | { |
|  | var parameters = method.GetParameters(); |
|  | var args = new object?[parameters.Length]; |
|  | for (int i = 0; i < parameters.Length; i++) |
|  | { |
|  | try |
|  | { |
|  | var param = parameters[i]; |
|  | string paramName = param.Name==null?"": param.Name.ToLower().ToString(); |
|  |  |
|  | if (arguments.TryGetProperty(paramName, out var valueElement)) |
|  | { |
|  | args[i] = valueElement.ValueKind switch |
|  | { |
|  | JsonValueKind.String => valueElement.GetString(), |
|  | JsonValueKind.Number when param.ParameterType == typeof(int) => valueElement.GetInt32(), |
|  | JsonValueKind.Number when param.ParameterType == typeof(double) => valueElement.GetDouble(), |
|  | JsonValueKind.True or JsonValueKind.False => valueElement.GetBoolean(), |
|  | _ => valueElement.GetRawText() // fallback |
|  | }; |
|  | } |
|  | else if (param.HasDefaultValue) |
|  | { |
|  | args[i] = param.DefaultValue; |
|  | } |
|  | else |
|  | { |
|  | throw new ArgumentException($"Missing required parameter: {param.Name}"); |
|  | } |
|  | } |
|  | catch(Exception ex) |
|  | {            } |
|  | } |
|  | return args; |
|  | } |
|  | private object GenerateInputSchema(MethodInfo method) |
|  | { |
|  | var properties = new Dictionary<string, object>(); |
|  | var required = new List<string>(); |
|  |  |
|  | foreach (var param in method.GetParameters()) |
|  | { |
|  | var paramName = param.Name!.ToLower(); |
|  | var description = param.GetCustomAttribute()?.Description ?? ""; |
|  | properties[paramName] = new |
|  | { |
|  | type = "string", // 简化：所有参数视为 string |
|  | description |
|  | }; |
|  | if (!param.HasDefaultValue) |
|  | required.Add(paramName); |
|  | } |
|  | return new |
|  | { |
|  | type = "object", |
|  | properties, |
|  | required = required.Count > 0 ? required : (object)Array.Empty<string>() |
|  | }; |
|  | } |
|  | } |
|  | // 工具元数据 |
|  | internal class ToolInfo |
|  | { |
|  | public required string Name { get; init; } |
|  | public required MethodInfo Method { get; init; } |
|  | public required string Description { get; init; } |
|  | public required object InputSchema { get; init; } |
|  | } |
|  | // 请求体 |
|  | public class InvokeToolRequest |
|  | { |
|  | public string? Name { get; set; } |
|  | public JsonElement Arguments { get; set; } |
|  | } |
```

#### 2.2.2 客户端代码

```
|  |  |
| --- | --- |
|  | // Program.cs |
|  | using McpClientApp.Models; |
|  | using Newtonsoft.Json; |
|  | using System; |
|  | using System.Net.Http; |
|  | using System.Net.Http.Json; |
|  | using System.Text; |
|  | using System.Text.Json; |
|  | using System.Text.Json.Serialization; |
|  | using System.Threading.Tasks; |
|  |  |
|  | // .NET 8 顶级语句：无需 class/Program/main |
|  | Console.WriteLine("MCP Client with Qwen (via Ollama) - .NET 8 - Type 'exit' to quit"); |
|  |  |
|  | // 创建 HttpClient (生产环境建议使用 IHttpClientFactory) |
|  | using var httpClient = new HttpClient(); |
|  |  |
|  | while (true) |
|  | { |
|  | Console.Write("\nYou: "); |
|  | var userQuestion = Console.ReadLine(); |
|  | if (string.IsNullOrWhiteSpace(userQuestion) || userQuestion.ToLower() == "exit") |
|  | break; |
|  | await ProcessQuestionAsync(userQuestion, httpClient); |
|  | } |
|  |  |
|  | async Task ProcessQuestionAsync(string userQuestion, HttpClient client) |
|  | { |
|  | const string ollamaUrl = "http://localhost:11434/api/chat"; |
|  | const string mcpServerUrl = "http://localhost:5113"; |
|  | // 1. 动态获取工具列表并生成提示词 |
|  | var systemPrompt = await GetAvailableToolsAsync(client, mcpServerUrl); |
|  | // 2. 调用 Ollama (Qwen) |
|  | object[] messages =[ |
|  | new { role = "system", content = systemPrompt }, |
|  | new { role = "user", content = userQuestion } |
|  | ]; |
|  | var ollamaResponse = await CallOllamaAsync("qwen:7b", messages, client, ollamaUrl); |
|  | if (string.IsNullOrEmpty(ollamaResponse)) |
|  | { |
|  | Console.WriteLine("Error: No response from Ollama."); |
|  | return; |
|  | } |
|  | Console.WriteLine($"LLM Export: {ollamaResponse}"); |
|  | // 3. 尝试解析为工具调用 |
|  | try |
|  | { |
|  | ollamaResponse = ollamaResponse.Replace("}}}", "}}"); |
|  | using var jsonDoc = JsonDocument.Parse(ollamaResponse); |
|  | var root = jsonDoc.RootElement; |
|  | if (root.TryGetProperty("tool", out var toolElement) && |
|  | root.TryGetProperty("arguments", out var argsElement)) |
|  | { |
|  | var toolName = toolElement.GetString(); |
|  | var argumentsJson = argsElement.GetRawText(); // 保持原始 JSON 字符串 |
|  | Console.WriteLine($"Calling MCP tool: {toolName} with {argumentsJson}"); |
|  | // 4. 调用 MCP 服务端 |
|  | var toolResult = await InvokeMcpToolAsync(toolName, argumentsJson, client, mcpServerUrl); |
|  | // 5. 让 LLM 生成最终回复 |
|  | object[] finalMessages = [ |
|  | new { role = "system", content = "You are a helpful assistant." }, |
|  | new { role = "user", content = userQuestion }, |
|  | new { role = "assistant", content = ollamaResponse }, |
|  | new { role = "tool", content = toolResult } // MCP 工具结果 |
|  | ]; |
|  | var finalResponse = await CallOllamaAsync("qwen:7b", finalMessages, client, ollamaUrl); |
|  | Console.WriteLine($"Final Answer（MCP）: {finalResponse}"); |
|  | return; |
|  | } |
|  | } |
|  | catch (Exception ex) |
|  | { |
|  | // 不是有效的 JSON，说明是直接回答 |
|  | } |
|  | // 直接回答 |
|  | Console.WriteLine($"Answer（No MCP）: {ollamaResponse}"); |
|  | } |
|  |  |
|  | // 【调用 Ollama API】 |
|  | async Task<string> CallOllamaAsync(string model, object[] messages, HttpClient client, string ollamaUrl) |
|  | { |
|  | var payload = new |
|  | { |
|  | model, |
|  | messages, |
|  | stream = false |
|  | }; |
|  | var response = await client.PostAsJsonAsync(ollamaUrl, payload); |
|  | response.EnsureSuccessStatusCode(); |
|  | var jsonResponse = await response.Content.ReadFromJsonAsync(); |
|  | // ValueKind = Object : "{"model":"qwen:7b","created_at":"2025-09-24T11:53:43.645783Z","message":{"role":"assistant","content":"{\"tool\": \"GetCurrentTime\", \"arguments\": {\"city\": \"London\"}}}"},"done":true,"done_reason":"stop","total_duration":2392332700,"load_duration":47250000,"prompt_eval_count":112,"prompt_eval_duration":176531800,"eval_count":18,"eval_duration":2167583600}" |
|  | return jsonResponse.GetProperty("message").GetProperty("content").ToString().Replace("\n",""); |
|  | } |
|  |  |
|  | // 【调用 MCP 服务端工具】 |
|  | async Task<string> InvokeMcpToolAsync(string toolName, string argumentsJson, HttpClient client, string mcpServerUrl) |
|  | { |
|  | var payload = new { name = toolName, argumentsJson }; |
|  | var json = System.Text.Json.JsonSerializer.Serialize(payload); |
|  | var content = new StringContent(json, Encoding.UTF8, "application/json"); |
|  | var response = await client.PostAsync($"{mcpServerUrl.TrimEnd('/')}/mcp/invoke", content); |
|  | if (!response.IsSuccessStatusCode) |
|  | { |
|  | var error = await response.Content.ReadAsStringAsync(); |
|  | throw new Exception($"MCP tool call failed: {error}"); |
|  | } |
|  | var jsonResponse = await response.Content.ReadFromJsonAsync(); |
|  | return jsonResponse.TryGetProperty("result", out var result) ? result.GetString()! : "No result returned(Calling MCP)."; |
|  | } |
|  | // 【获取全部可用的 MCP 工具】 |
|  | async Task<string> GetAvailableToolsAsync(HttpClient client, string mcpServerUrl) |
|  | { |
|  | try |
|  | { |
|  | var response = await client.GetStringAsync($"{mcpServerUrl.TrimEnd('/')}/mcp/tools"); |
|  | var tools = System.Text.Json.JsonSerializer.Deserialize(response); |
|  | if (tools == null || tools.Length == 0) |
|  | return "No tools available."; |
|  | // 动态生成工具描述 |
|  | var toolDescriptions = string.Join("\n", tools.Select(t => |
|  | { |
|  | var paramsText = string.Join(",", t.InputSchema.Required); |
|  | //return $"- {t.Name}({paramsText}): {t.Description}"; |
|  | //return $"- {t.Name}: {t.Description}"; |
|  | return $@"- {t.Name}({paramsText}): {t.Description} |
|  | If you need to use a tool, respond with a JSON object in this exact format: |
|  | {{""tool"": ""{t.Name}"", ""arguments"": {{""{paramsText}"": ""value""}}}}"; |
|  | })); |
|  | return @$""" |
|  | You can use these external tools: |
|  | {toolDescriptions} |
|  | Only output the JSON when calling a tool. Otherwise, answer normally. |
|  | """; |
|  | } |
|  | catch (Exception ex) |
|  | { |
|  | Console.WriteLine($"Warning: Could not fetch tools from MCP server: {ex.Message}"); |
|  | return "No external tools available."; // 降级：不启用工具调用 |
|  | } |
|  | } |
```

```
|  |  |
| --- | --- |
|  | // Mcptool.cs |
|  | using System; |
|  | using System.Collections.Generic; |
|  | using System.Linq; |
|  | using System.Text; |
|  | using System.Text.Json.Serialization; |
|  | using System.Threading.Tasks; |
|  |  |
|  | namespace McpClientApp.Models |
|  | { |
|  | // 根对象：工具列表 |
|  | public class McpTool |
|  | { |
|  | [JsonPropertyName("name")] |
|  | public string Name { get; set; } |
|  |  |
|  | [JsonPropertyName("description")] |
|  | public string Description { get; set; } |
|  |  |
|  | [JsonPropertyName("inputSchema")] |
|  | public InputSchema InputSchema { get; set; } |
|  | } |
|  |  |
|  | // inputSchema 对象 |
|  | public class InputSchema |
|  | { |
|  | [JsonPropertyName("type")] |
|  | public string Type { get; set; } // 通常是 "object" |
|  |  |
|  | [JsonPropertyName("properties")] |
|  | public Dictionary<string, SchemaProperty> Properties { get; set; } |
|  |  |
|  | [JsonPropertyName("required")] |
|  | public string[] Required { get; set; } |
|  | } |
|  |  |
|  | // 属性定义（如 city） |
|  | public class SchemaProperty |
|  | { |
|  | [JsonPropertyName("type")] |
|  | public string Type { get; set; } // string, number, boolean 等 |
|  |  |
|  | [JsonPropertyName("description")] |
|  | public string Description { get; set; } |
|  | } |
|  | } |
```

#### 2.2.3 测试结果

```
|  |  |
| --- | --- |
|  | You: 你是谁？ |
|  | LLM Export: 我是阿里云研发的一款超大规模语言模型，我叫通义千问。 |
|  | Answer（No MCP）: 我是阿里云研发的一款超大规模语言模型，我叫通义千问。 |
|  |  |
|  | You: 现在北京时间是多少？ |
|  | LLM Export: { "tool": "GetCurrentTime", "arguments": { "city": "北京" } } |
|  | Calling MCP tool: GetCurrentTime with { "city": "北京" } |
|  | Final Answer（MCP）:  根据当前系统，北京时间是：2023-3-16 15:48:19 但请注意，这可能不是一个准确的实时时间，因为我在处理 请求时的时间可能会有些延迟。建议您通过官方时间服务或手机自带的时间同步功能获取最精确的时间信息。 |
```

注意：针对本示例中使用的大模型"qwen:7b"，输出并不稳定，存在本地时间返回异常情况，仅供理解思路吧。后续博主还会继续测试，逐步优化。

### 2.3 使用新版的千问大模型测试：qwen3:8b【推荐使用】

* **为什么要使用新版的大模型？**

对比旧版的大模型，新版的输出就比较稳定了。

另外新版大模型在接口调用中返回了标签内容，标识每次都会对指令进行分析，所得出的结果也比较准确。

```
|  |  |
| --- | --- |
|  | You: 现在北京时间是多少？ |
|  | LLM Export: 好的，用户问现在北京时间是多少。我需要用 GetCurrentTime 工具来获取。工具的参数是城市，所以我要指定城 市为北京。然后按照要求返回JSON对象，调用工具。确保格式正确，不添加其他内容。检查一下参数是否正确，城市名称是否准确。确认无误后生成响应。{"tool": "GetCurrentTime", "arguments": {"city": "北京"}} |
|  | Calling MCP tool: GetCurrentTime with {"city": "北京"} |
|  | Calling MCP toolResult: The current time in  is 13:50 on 2025-09-30. |
|  | Final Answer（MCP）: 好的，用户之前问了北京时间，我调用了GetCurrentTime工具，参数是北京。现在工具返回了结果，显示当前时间是2025年9月30日13:50。需要确认返回的格式是否正确，有没有错误。用户可能是在确认时间，或者有安排需要知道准确时间。可能用户所在时区不同，需要明确是北京时间。另外，检查日期和时间是否正确，确保没有时区错误。然后用自然的中文回复用户，保持简洁。比如直接告诉用户当前北京时间是13:50，日期是2025年9月30日。不需要额外信息，除非用户有进一步问题。确保回答清晰准确。当前北京时间是2025年9月30日13:50。 |
```

其中，最终的输出为：Final Answer（MCP）:

`好的，用户之前问了北京时间，我调用了GetCurrentTime工具，参数是北京。现在工具返回了结果，显示当前时间是2025年9月30日13:50。需要确认返回的格式是否正确，有没有错误。用户可能是在确认时间，或者有安排需要知道准确时间。可能用户所在时区不同，需要明确是北京时间。另外，检查日期和时间是否正确，确保没有时区错误。然后用自然的中文回复用户，保持简洁。比如直接告诉用户当前北京时间是13:50，日期是2025年9月30日。不需要额外信息，除非用户有进一步问题。确保回答清晰准确。当前北京时间是2025年9月30日13:50。`

前一段为思考过程，后边为实际返回的结果。

* **代码中要调整的内容**

具体的代码可以查看上一章节，此处不再赘述，只需调整其中的两点：

1）在第三步，得到大模型第一次返回后，解析返回结果时，要先把思考内容剔除，如下：

```
|  |  |
| --- | --- |
|  | // 3. 尝试解析为工具调用 |
|  | var res_arr = ollamaResponse.Split(""); |
|  | using var jsonDoc = JsonDocument.Parse(res_arr[1]); |
```

2）把代码中的模型版本换成新的：`qwen3:8b`。

* **在 ollama 中安装新版大模型**

```
|  |  |
| --- | --- |
|  | // 在 ollama 中安装 |
|  | C:\Users>ollama pull qwen3:8b |
|  | // 查看已安装的大模型 |
|  | C:\Users>ollama list |
|  | NAME                    ID              SIZE      MODIFIED |
|  | qwen3:8b                500a1f067a9f    5.2 GB    1 hours ago |
|  | qwen:7b                 2091ee8c8d8f    4.5 GB    1 days ago |
|  | // 运行新版模型尝试对话 |
|  | C:\Users>ollama run qwen3:8b |
|  | >>> 你是谁？ |
|  | Thinking... |
|  | 嗯，用户问我是谁，这应该是一个常见的问题。首先，我需要确认用户是想了解我的基本身份，还是有更深层次的需求。可能用户刚 |
|  | 接触这个系统，或者对我的功能不太清楚。 |
|  |  |
|  | 接下来，我应该简要介绍我的身份，比如我是通义千问，由通义实验室研发，基于大规模语言模型。同时，要强调我的功能，比如回 |
|  | 答问题、创作文字、编程等，这样用户能了解我的用途。 |
|  |  |
|  | 还要注意用户的潜在需求，比如他们可能想知道我是否可靠，或者是否有特定的功能需要使用。这时候可以提到我的训练数据和应用 |
|  | 场景，让用户更放心。 |
|  |  |
|  | 另外，用户可能没有明确说明他们需要什么帮助，所以主动询问是否需要进一步协助是个好主意。这样既能提供帮助，又能引导用户 |
|  | 更具体地表达需求。 |
|  |  |
|  | 最后，保持回答简洁明了，避免使用过于专业的术语，让用户容易理解。同时，保持友好和专业的语气，让用户感到舒适和信任。 |
|  | ...done thinking. |
|  |  |
|  | 我是通义千问，由通义实验室研发的超大规模语言模型。我能够回答问题、创作文字、编程、逻辑推理以及多语言理解等。我的训练 |
|  | 数据来自互联网上的大量文本，能够理解和生成多种语言的内容。如果你有任何问题或需要帮助，欢迎随时告诉我！需要我帮你做些 |
|  | 什么吗？ |
|  |  |
|  | >>> Send a message (/? for help) |
```

*MCP C# 示例参考：[https://github.com/edisontalk/p/-/introduction-to-mcp-csharp-sdk](https://github.com/edisontalk/p/-/introduction-to-mcp-csharp-sdk "https://github.com/edisontalk/p/-/introduction-to-mcp-csharp-sdk")**。*
