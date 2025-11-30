# Cursor 编辑器进阶指南

---

## 目录

1. [Cursor Rule（规则）详解](#cursor-rule规则详解)
    - 1.1 规则简介
    - 1.2 规则类型与存储位置
    - 1.3 规则格式与结构
    - 1.4 编辑规则的方法
    - 1.5 编辑器选择与常见问题
    - 1.6 规则的应用与调用
    - 1.7 最佳实践与建议
    - 1.8 参考资料与进阶阅读
    - 1.9 全局与项目规则应用案例
2. [MCP（Model Context Protocol）详解](#mcpmodel-context-protocol详解)
    - 2.1 MCP简介
    - 2.2 典型应用场景与案例
    - 2.3 MCP架构与工作原理
    - 2.4 MCP服务器的配置方法
    - 2.5 MCP工具的使用与管理
    - 2.6 常见问题与注意事项
    - 2.7 参考资料
3. [Cursor 代理与VPN使用指南](#cursor-代理与vpn使用指南)
    - 3.1 常见VPN/代理场景说明
    - 3.2 Cursor对VPN的支持政策与常见问题
    - 3.3 常见连接故障及解决方法
    - 3.4 实用技巧与社区经验
    - 3.5 参考资料

---

## 1. Cursor Rule（规则）详解

### 1.1 规则简介

Cursor Rule（规则）是 Cursor 编辑器为 AI 助手（如代码补全、解释、重构等）提供"持久化指令"的机制。通过规则，你可以让 AI 始终按照你的风格、项目规范、团队约定等来工作，实现高度个性化和自动化的开发体验。

---

### 1.2 规则类型与存储位置

- **全局规则（User Rules）**：在 Cursor 设置中配置，适用于所有项目。
- **项目规则（Project Rules）**：存放在项目目录下的 `.cursor/rules/` 文件夹，支持版本控制和多级目录嵌套。
- **旧版规则（.cursorrules）**：项目根目录的 `.cursorrules` 文件，已被新机制替代但仍兼容。

详细说明可参考官方文档：[Cursor Rules 官方文档](https://docs.cursor.com/context/rules)

---

### 1.3 规则格式与结构

#### 1.3.1 MDC 格式（推荐）

项目规则采用 `.mdc` 文件（Markdown with Context），支持元数据和正文内容。例如：

```mdc
---
description: RPC Service boilerplate
globs: 
alwaysApply: false
---

- 使用公司内部的 RPC 模式定义服务
- 服务名称统一用 snake_case
```

- `description`：规则描述
- `globs`：文件匹配模式
- `alwaysApply`：是否总是应用

正文部分写具体的指令、规范、模板等。

#### 1.3.2 旧版 .cursorrules 格式

直接用纯文本写指令，简单明了，但不支持元数据和高级功能。

---

### 1.4 编辑规则的方法

#### 1.4.1 编辑全局规则

1. 打开 Cursor 设置（Settings）
2. 进入 `Rules` 或 `General > Rules for AI`
3. 在文本框中输入你的全局规则，保存即可

#### 1.4.2 编辑项目规则

1. 在项目目录下新建或打开 `.cursor/rules/` 文件夹
2. 新建或编辑 `.mdc` 文件（如 `my_rule.mdc`）
3. 按 MDC 格式填写内容，保存即可

**快捷方式**：  
在 Cursor 中按 `Cmd + Shift + P`，输入 "New Cursor Rule"，可快速创建规则文件。

#### 1.4.3 编辑 .cursorrules（兼容旧项目）

1. 在项目根目录找到 `.cursorrules` 文件
2. 直接用文本编辑器修改内容，保存即可

---

### 1.5 编辑器选择与常见问题

- **MDC 编辑器问题**：有用户反馈用 Cursor 自带的 MDC Editor 编辑 `.mdc` 文件时，内容有时不会实时保存。建议右键文件选择"Open with..."，用"Text Editor"打开并编辑，确保内容写入磁盘（参考：[社区讨论](https://forum.cursor.com/t/bug-how-to-make-chat-composer-edit-a-cursor-rule-mdc-editor-gotchas/53208)）。
- **嵌套规则**：可以在项目的不同子目录下建立 `.cursor/rules/`，实现分层管理，适合大型/单体仓库。
- **规则生效问题**：确保规则类型、文件名、路径正确，且 MDC 文件头部元数据填写完整。

---

### 1.6 规则的应用与调用

- **自动应用**：如 `alwaysApply: true`，则每次 AI 响应都会加载该规则。
- **手动调用**：在对话中用 `@规则名` 触发特定规则。
- **自动关联**：通过 `globs` 字段，指定规则只在特定文件/目录下生效。

---

### 1.7 最佳实践与建议

- **规则要具体、可操作**，避免模糊描述。
- **分拆大规则为多个小规则**，便于复用和维护。
- **多用示例和模板**，让 AI 理解你的意图。
- **团队协作时**，可将规则文件纳入版本控制，保持风格统一。

---

### 1.8 参考资料与进阶阅读

- [Cursor 官方规则文档](https://docs.cursor.com/context/rules)
- [Cursor 101：规则定制与实战](https://cursor101.com/article/cursor-rules-customizing-ai-behavior)
- [社区规则分享与讨论](https://forum.cursor.com/t/bug-how-to-make-chat-composer-edit-a-cursor-rule-mdc-editor-gotchas/53208)

---

### 1.9 全局与项目规则应用案例

#### 1.9.1 全局规则应用案例

**适用场景说明**：全局规则适用于所有项目，常用于统一AI助手的风格、语言、通用开发习惯等。例如，要求AI始终用中文回复、代码风格统一、避免重复性内容等。

**规则示例（全局规则为纯文本，设置于Cursor Settings > Rules）**：
```
请始终用简明、专业的中文回复，避免冗余和重复表达。
所有代码建议需遵循PEP8（Python）或Prettier（JS）等主流风格。
遇到不确定的问题时，先分析再给出建议，避免直接生成不完整代码。
```
**说明**：全局规则为AI助手设定了统一的行为基调和风格，适合个人或团队日常开发的通用需求。

---

#### 1.9.2 项目规则应用案例

**适用场景说明**：项目规则针对具体项目的业务需求、技术栈、开发流程等进行定制。例如大模型API调用、ETL开发、特定框架规范等。

**规则示例1：大模型对话调用场景（MDC格式，存放于`.cursor/rules/llm_api.mdc`）**：
```mdc
---
description: LLM API 调用规范
globs: ["*.py", "*.js"]
alwaysApply: true
---
- 所有大模型API调用必须封装在单独的函数中，便于维护和复用。
- 必须对API响应进行异常捕获和错误日志记录。
- 不允许在代码中硬编码API Key，需通过环境变量读取。
- 对于敏感信息，务必加注释说明其安全性处理方式。
- 推荐使用`requests`（Python）或`axios`（JS）等主流库发起HTTP请求。
```
**规则示例2：ETL开发场景（MDC格式，存放于`.cursor/rules/etl.mdc`）**：
```mdc
---
description: ETL 脚本开发规范
globs: ["etl_*.py", "*.ipynb"]
alwaysApply: true
---
- ETL脚本应分为extract、transform、load三个主要函数，每个函数职责单一。
- 必须在每个阶段添加详细日志，便于追踪数据流转。
- 所有外部依赖（如数据库连接、API Key）需通过配置文件或环境变量管理。
- 对于数据异常和失败情况，需有重试和告警机制。
- 代码需加注释，说明每一步的业务逻辑。
```
**说明**：项目规则让AI在特定项目中自动遵循业务和技术规范，提升代码质量和团队协作效率。

---

#### 1.9.3 参考与更多案例

- [awesome-cursorrules：丰富的开源规则案例库](https://github.com/PatrickJS/awesome-cursorrules)
- [Medium：Cursor Rules with Examples](https://medium.com/realworld-ai-use-cases/cursor-rules-with-examples-the-secret-trick-to-building-bigger-and-better-projects-b13931f2bcae)

> 建议结合实际需求，合理区分全局与项目规则，让AI助手在不同层面都能高效服务于你的开发流程。

---

## 2. MCP（Model Context Protocol）详解

### 2.1 MCP简介

MCP（Model Context Protocol，模型上下文协议）是Cursor提出的一套开放协议，标准化了应用如何为大模型（LLM）提供上下文和工具。它本质上是Cursor的插件系统，允许你通过标准接口将外部工具、数据源与Cursor AI深度集成，极大扩展了AI助手的能力。

详见：[官方MCP文档](https://docs.cursor.com/context/model-context-protocol)

---

### 2.2 典型应用场景与案例

- **数据库集成**：让Cursor直接查询数据库，无需手动输入schema或数据。
- **Notion集成**：读取Notion内容，辅助AI理解需求或生成代码。
- **GitHub集成**：让Cursor自动创建PR、分支、查找代码等。
- **记忆/知识库**：让AI在会话中记住并调用历史信息。
- **第三方API**：如Stripe、Convex等，直接操作业务数据。

案例参考：[Convex MCP Server官方教程](https://docs.convex.dev/ai/using-cursor)

---

### 2.3 MCP架构与工作原理

MCP服务器是一个轻量级程序，通过标准协议暴露能力，充当Cursor与外部系统的中间层。

- **stdio模式**：MCP服务运行在本地，由Cursor自动管理，通过标准输入输出（stdout）通信，适合本地开发。
- **SSE模式**：MCP服务可本地或远程运行，通过网络（SSE协议）通信，适合分布式或团队共享。

MCP服务器可用任何支持stdout或HTTP的语言实现。

---

### 2.4 MCP服务器的配置方法

#### 2.4.1 mcp.json配置示例

```json
{
  "mcpServers": {
    "convex": {
      "command": "npx",
      "args": ["-y", "convex@latest", "mcp", "start"]
    }
  }
}
```
- `command`：启动MCP服务的命令
- `args`：命令参数
- `env`：可选，环境变量（如API_KEY等）

#### 2.4.2 配置文件位置
- **项目级**：项目目录下 `.cursor/mcp.json`，仅对当前项目生效。
- **全局级**：用户主目录下 `~/.cursor/mcp.json`，对所有项目生效。

#### 2.4.3 启用与管理
- 在Cursor设置 > MCP页面添加、启用、禁用MCP服务器。
- 支持一键启用/禁用，或在配置文件中管理。

---

### 2.5 MCP工具的使用与管理

- MCP工具会在Cursor的"可用工具"列表中自动显示。
- AI Agent会根据上下文自动调用相关MCP工具，也可在对话中直接指定工具名或描述让AI调用。
- 默认情况下，Agent调用MCP工具时会请求用户审批（可在设置中开启自动运行/YOLO模式）。
- 工具调用结果会在对话中展示，支持参数和响应内容的展开查看。
- 支持返回图片（base64编码），AI可直接分析图片内容。

---

### 2.6 常见问题与注意事项

- **工具数量限制**：当前最多支持40个MCP工具同时可用。
- **远程开发限制**：MCP服务器需本地或可访问，远程SSH开发时可能受限。
- **资源支持**：目前MCP仅支持工具（tool），资源（resource）支持尚在开发中。
- **配置建议**：建议将敏感信息（如API_KEY）通过`env`字段传递，避免硬编码。
- **管理建议**：如需批量管理MCP服务器，建议直接编辑mcp.json文件。

更多社区讨论与建议可参考：[MCP安装与管理建议](https://forum.cursor.com/t/mcp-install-config-and-management-suggestions/49283)

---

### 2.7 参考资料

- [Cursor官方MCP文档](https://docs.cursor.com/context/model-context-protocol)
- [Convex官方MCP集成教程](https://docs.convex.dev/ai/using-cursor)
- [社区MCP配置与管理讨论](https://forum.cursor.com/t/mcp-install-config-and-management-suggestions/49283)

---

> MCP极大扩展了Cursor的AI能力，适合有自定义工具、数据源、自动化需求的开发者和团队。建议结合实际项目需求，尝试集成并优化MCP工具链。

---

## 3. Cursor 代理与VPN使用指南（详细版）

### 3.1 VPN/代理原理与常见类型

- **VPN（虚拟专用网络）**：通过加密隧道将本地流量转发到远程服务器，实现网络加速、隐私保护或突破地理限制。常见协议有Wireguard、OpenVPN、IKEv2等。
- **企业代理**：如Zscaler、AWS VPN、公司自建代理，通常用于企业安全、流量审计和访问控制，可能会对部分加密协议或端口做特殊处理。
- **常见场景**：
  - 科学上网/跨境访问
  - 保护个人隐私
  - 企业内网远程办公

### 3.2 Cursor对VPN的官方态度与风险分析

- 官方明确表示**不会主动封禁VPN用户**，大部分主流VPN（如Wireguard、ProtonVPN等）可正常使用Cursor。
- 但后台有智能风控机制，部分高风险IP段或数据中心IP可能被限制，极少数情况下会影响连接。
- 企业代理环境下，部分安全策略（如SSL中间人、流量劫持）可能导致Cursor功能异常。
- 官方建议如遇误封或持续连接问题，可通过社区或邮件联系申诉。

引用：[Cursor官方社区VPN讨论](https://forum.cursor.com/t/cursor-with-vpn/951)

### 3.3 不同VPN/代理下的常见问题表现

- **连接失败/Generating卡住**：常见于企业代理、部分VPN节点不稳定或被风控。
- **Chat/对话功能不可用**：新版Cursor禁用HTTP2后，Chat功能会失效（见下文）。
- **证书/握手错误**：企业代理（如Zscaler）未正确配置证书，或Node.js信任库未导入。
- **部分功能正常，部分功能异常**：如侧边栏可用但内联编辑/对话不可用，多见于代理协议兼容性问题。

相关社区讨论：[连接失败与VPN问题](https://forum.cursor.com/t/the-connection-failed-check-connection-vpn/1952)

### 3.4 详细排查与解决步骤

#### 3.4.1 HTTP2设置与版本兼容性
- 某些VPN或企业代理对HTTP2协议兼容性较差，可能导致Cursor部分功能失效。
- 解决方法：
  - 在老版本（如v0.45.17及之前）可通过设置`cursor.general.disableHttp2: true`解决大部分代理兼容问题。
  - 新版本（如v0.46.x及以后）禁用HTTP2会导致Chat/对话功能不可用，需权衡使用。
  - 如需兼容代理且保留全部功能，可临时降级Cursor版本，等待官方后续支持HTTP/1.1。
- 设置方法：
  - VSCode命令面板（Cmd+Shift+P）→ Open VSCode Settings → 搜索http2 → 设置`cursor.general.disableHttp2`为`true`。
  - 或在用户设置（JSON）中添加：
    ```json
    "cursor.general.disableHttp2": true
    ```
- 参考：[GitHub Issue #1534](https://github.com/getcursor/cursor/issues/1534)、[社区经验](https://forum.cursor.com/t/cant-use-vpn-get-connection-failed-error-v0-46-8/59251)

#### 3.4.2 证书、白名单与企业代理配置
- 企业代理如Zscaler需将自签证书导入系统和Node.js信任库。
- 某些功能需IT部门协助将Cursor相关域名加入白名单。
- 启动Cursor时可带上代理环境变量（如`HTTPS_PROXY`、`HTTP_PROXY`）。
- 有时需从命令行启动Cursor并指定代理参数。

#### 3.4.3 VPN节点选择与协议建议
- 多位用户反馈Wireguard协议（尤其UDP）兼容性和稳定性最佳。
- ProtonVPN、NordVPN等主流服务在大多数场景下表现良好。
- 若遇到节点不稳定或被风控，建议切换不同地区或协议。

### 3.5 实用建议与经验总结

- 遇到连接异常时，优先尝试切换VPN节点或协议。
- 企业用户建议与IT部门协作，确保代理和证书配置正确。
- 若因HTTP2设置导致功能受限，可临时降级Cursor版本（如v0.45.17），等待官方后续修复。
- 遇到持续性问题，建议收集日志并通过社区或官方支持渠道反馈。
- 参考社区经验和官方FAQ，往往能快速定位并解决问题。

### 3.6 常见问题FAQ

**Q1：Cursor会封禁VPN用户吗？**  
A：不会主动封禁，但部分高风险IP段可能被风控，遇到误封可申诉。

**Q2：企业代理下功能异常怎么办？**  
A：优先检查证书和白名单配置，必要时禁用HTTP2或降级Cursor版本。

**Q3：禁用HTTP2后Chat不可用怎么办？**  
A：可临时降级到v0.45.17等老版本，或等待官方支持HTTP/1.1。

**Q4：如何联系官方支持？**  
A：可通过[官方社区](https://forum.cursor.com/)或[支持页面](https://cursor.com/support)提交工单。

**Q5：还有哪些社区经验可参考？**  
A：详见[Cursor官方社区VPN讨论](https://forum.cursor.com/t/cursor-with-vpn/951)、[连接失败与VPN问题](https://forum.cursor.com/t/the-connection-failed-check-connection-vpn/1952)、[GitHub Issue #1534](https://github.com/getcursor/cursor/issues/1534)。

> 如遇到特殊网络环境下的连接问题，建议优先参考上述社区经验和官方文档，或直接联系Cursor官方技术支持。
