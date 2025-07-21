# JManus AI 智能助手平台 - 组件图 (Component Diagram)

本文档展示 JManus AI 智能助手平台的组件图，可视化系统组件及其依赖关系，用于架构设计与重构。

## 文档说明

**使用场景**: 可视化系统组件及其依赖关系  
**应用阶段**: 架构设计与重构  
**关键优势**: 支持模块化，易于演进  

## 系统整体组件图

```plantuml
@startuml
!theme plain
skinparam component {
  BackgroundColor lightblue
  BorderColor darkblue
}
skinparam package {
  BackgroundColor lightyellow
  BorderColor orange
}
skinparam interface {
  BackgroundColor lightgreen
  BorderColor darkgreen
}

title JManus AI 智能助手平台 - 系统组件图

' ===== 用户接口层 =====
package "用户接口层 (Presentation Layer)" {
  component [Vue3 Web UI] as WebUI
  component [REST API Gateway] as APIGateway
  
  interface "HTTP/HTTPS" as HTTP
  interface "WebSocket" as WS
  interface "REST API" as REST
}

' ===== API控制器层 =====
package "API控制器层 (Controller Layer)" {
  component [Agent Controller] as AgentCtrl
  component [Model Controller] as ModelCtrl
  component [Manus Controller] as ManusCtrl
  component [Prompt Controller] as PromptCtrl
  component [MCP Controller] as McpCtrl
  component [Config Controller] as ConfigCtrl
  
  interface "Controller Interface" as CtrlInterface
}

' ===== 业务服务层 =====
package "业务服务层 (Service Layer)" {
  component [Agent Service] as AgentService
  component [Model Service] as ModelService
  component [Prompt Service] as PromptService
  component [MCP Service] as McpService
  component [Config Service] as ConfigService
  component [User Input Service] as UserInputService
  
  interface "Service Interface" as ServiceInterface
}

' ===== 核心组件层 =====
package "核心组件层 (Core Components)" {
  component [Planning Coordinator] as PlanCoordinator
  component [Plan Creator] as PlanCreator
  component [Plan Executor] as PlanExecutor
  component [Plan Finalizer] as PlanFinalizer
  component [Dynamic Agent Loader] as AgentLoader
  component [LLM Service] as LLMService
  
  interface "Planning Interface" as PlanInterface
  interface "Agent Interface" as AgentInterface
  interface "LLM Interface" as LLMInterface
}

' ===== 智能体层 =====
package "智能体层 (Agent Layer)" {
  component [Base Agent] as BaseAgent
  component [Dynamic Agent] as DynamicAgent
  component [Agent State Manager] as AgentStateManager
  component [Tool Calling Manager] as ToolManager
  
  interface "Agent Execution Interface" as AgentExecInterface
}

' ===== 工具生态层 =====
package "工具生态层 (Tool Ecosystem)" {
  component [Planning Tool] as PlanningTool
  component [Browser Tool] as BrowserTool
  component [Python Execute Tool] as PythonTool
  component [Bash Tool] as BashTool
  component [Form Input Tool] as FormTool
  component [MCP Tool Integration] as McpToolIntegration
  component [Doc Loader Tool] as DocTool
  
  interface "Tool Interface" as ToolInterface
}

' ===== 数据访问层 =====
package "数据访问层 (Data Access Layer)" {
  component [Agent Repository] as AgentRepo
  component [Model Repository] as ModelRepo
  component [Prompt Repository] as PromptRepo
  component [MCP Config Repository] as McpRepo
  component [Config Repository] as ConfigRepo
  component [Plan Execution Repository] as PlanRepo
  
  interface "Repository Interface" as RepoInterface
}

' ===== 基础设施层 =====
package "基础设施层 (Infrastructure Layer)" {
  component [Database Connection Pool] as DBPool
  component [HTTP Client Pool] as HTTPPool
  component [Thread Pool Manager] as ThreadPool
  component [Cache Manager] as CacheManager
  component [Log Manager] as LogManager
  
  interface "Infrastructure Interface" as InfraInterface
}

' ===== 外部系统 =====
package "外部系统 (External Systems)" {
  component [AI Model Services] as AIServices
  component [MCP Servers] as McpServers
  component [Browser Engine] as BrowserEngine
  component [Python Runtime] as PythonRuntime
  component [Database Systems] as DatabaseSystems
  
  interface "External API" as ExtAPI
  interface "MCP Protocol" as MCPProtocol
  interface "WebDriver Protocol" as WebDriverProtocol
}

' ===== 接口连接 =====
WebUI ..> HTTP
APIGateway ..> REST
APIGateway ..> WS

HTTP -- AgentCtrl
REST -- ManusCtrl
REST -- ModelCtrl
REST -- PromptCtrl
REST -- McpCtrl
REST -- ConfigCtrl

AgentCtrl ..> CtrlInterface
ManusCtrl ..> CtrlInterface
ModelCtrl ..> CtrlInterface
PromptCtrl ..> CtrlInterface
McpCtrl ..> CtrlInterface
ConfigCtrl ..> CtrlInterface

CtrlInterface -- AgentService
CtrlInterface -- ModelService
CtrlInterface -- PromptService
CtrlInterface -- McpService
CtrlInterface -- ConfigService
CtrlInterface -- UserInputService

AgentService ..> ServiceInterface
ModelService ..> ServiceInterface
PromptService ..> ServiceInterface
McpService ..> ServiceInterface
ConfigService ..> ServiceInterface

ServiceInterface -- PlanCoordinator
ServiceInterface -- LLMService
ServiceInterface -- AgentLoader

PlanCoordinator ..> PlanInterface
PlanCreator ..> PlanInterface
PlanExecutor ..> PlanInterface
PlanFinalizer ..> PlanInterface

PlanInterface -- BaseAgent
PlanInterface -- DynamicAgent
PlanInterface -- ToolManager

BaseAgent ..> AgentInterface
DynamicAgent ..> AgentInterface
AgentStateManager ..> AgentInterface

AgentInterface -- PlanningTool
AgentInterface -- BrowserTool
AgentInterface -- PythonTool
AgentInterface -- BashTool
AgentInterface -- FormTool
AgentInterface -- McpToolIntegration

PlanningTool ..> ToolInterface
BrowserTool ..> ToolInterface
PythonTool ..> ToolInterface
BashTool ..> ToolInterface
FormTool ..> ToolInterface
McpToolIntegration ..> ToolInterface
DocTool ..> ToolInterface

ToolInterface -- AgentRepo
ToolInterface -- ModelRepo
ToolInterface -- PromptRepo
ToolInterface -- McpRepo
ToolInterface -- ConfigRepo

AgentRepo ..> RepoInterface
ModelRepo ..> RepoInterface
PromptRepo ..> RepoInterface
McpRepo ..> RepoInterface
ConfigRepo ..> RepoInterface
PlanRepo ..> RepoInterface

RepoInterface -- DBPool
RepoInterface -- CacheManager
RepoInterface -- LogManager

DBPool ..> InfraInterface
HTTPPool ..> InfraInterface
ThreadPool ..> InfraInterface
CacheManager ..> InfraInterface
LogManager ..> InfraInterface

' ===== 外部依赖 =====
LLMService ..> ExtAPI
McpToolIntegration ..> MCPProtocol
BrowserTool ..> WebDriverProtocol

ExtAPI -- AIServices
MCPProtocol -- McpServers
WebDriverProtocol -- BrowserEngine
PythonTool -- PythonRuntime
RepoInterface -- DatabaseSystems

@enduml
```

## 核心模块组件图

### 1. 任务执行核心组件

```plantuml
@startuml
!theme plain
title JManus 任务执行核心组件

package "任务执行模块" {
  component [Planning Coordinator] as Coordinator
  component [Plan Creator] as Creator
  component [Plan Executor] as Executor
  component [Plan Finalizer] as Finalizer
  
  component [Execution Context] as Context
  component [Execution Step] as Step
  component [Plan Template Manager] as TemplateManager
}

package "智能体模块" {
  component [Dynamic Agent] as Agent
  component [Agent State Manager] as StateManager
  component [Tool Calling Manager] as ToolManager
}

package "LLM集成模块" {
  component [LLM Service] as LLMService
  component [Prompt Service] as PromptService
  component [Model Configuration] as ModelConfig
}

interface "Planning Interface" as IPlan
interface "Agent Interface" as IAgent
interface "LLM Interface" as ILLM

Coordinator ..> IPlan
Creator ..> IPlan
Executor ..> IPlan
Finalizer ..> IPlan

IPlan -- Context
IPlan -- Step
IPlan -- TemplateManager

Agent ..> IAgent
StateManager ..> IAgent
ToolManager ..> IAgent

LLMService ..> ILLM
PromptService ..> ILLM
ModelConfig ..> ILLM

IAgent -- IPlan
ILLM -- IPlan

@enduml
```

### 2. 工具生态组件

```plantuml
@startuml
!theme plain
title JManus 工具生态组件

package "工具框架" {
  component [Abstract Base Tool] as BaseTool
  component [Tool Registry] as ToolRegistry
  component [Tool Executor] as ToolExecutor
  component [Tool Result Handler] as ResultHandler
}

package "内置工具" {
  component [Planning Tool] as PlanningTool
  component [Form Input Tool] as FormTool
  component [Terminate Tool] as TerminateTool
  component [Doc Loader Tool] as DocTool
}

package "浏览器工具" {
  component [Browser Tool] as BrowserTool
  component [WebDriver Manager] as WebDriverManager
  component [Page Element Locator] as ElementLocator
  component [Action Executor] as ActionExecutor
}

package "代码执行工具" {
  component [Python Execute Tool] as PythonTool
  component [Bash Tool] as BashTool
  component [Code Validator] as CodeValidator
  component [Execution Sandbox] as Sandbox
}

package "MCP工具集成" {
  component [MCP Tool Integration] as McpIntegration
  component [MCP Client] as McpClient
  component [Tool Cache Manager] as CacheManager
  component [Protocol Handler] as ProtocolHandler
}

interface "Tool Interface" as ITool
interface "Browser Interface" as IBrowser
interface "Code Execution Interface" as ICodeExec
interface "MCP Interface" as IMCP

BaseTool ..> ITool
ToolRegistry ..> ITool
ToolExecutor ..> ITool

PlanningTool --|> BaseTool
FormTool --|> BaseTool
TerminateTool --|> BaseTool
DocTool --|> BaseTool

BrowserTool ..> IBrowser
WebDriverManager ..> IBrowser
BrowserTool --|> BaseTool

PythonTool ..> ICodeExec
BashTool ..> ICodeExec
PythonTool --|> BaseTool
BashTool --|> BaseTool

McpIntegration ..> IMCP
McpClient ..> IMCP
McpIntegration --|> BaseTool

@enduml
```

### 3. 数据管理组件

```plantuml
@startuml
!theme plain
title JManus 数据管理组件

package "配置管理" {
  component [Agent Configuration] as AgentConfig
  component [Model Configuration] as ModelConfig
  component [MCP Configuration] as McpConfig
  component [System Configuration] as SysConfig
  component [Configuration Validator] as ConfigValidator
}

package "数据访问层" {
  component [Agent Repository] as AgentRepo
  component [Model Repository] as ModelRepo
  component [MCP Repository] as McpRepo
  component [Config Repository] as ConfigRepo
  component [Plan Repository] as PlanRepo
  component [Execution Record Repository] as RecordRepo
}

package "数据实体" {
  component [Dynamic Agent Entity] as AgentEntity
  component [Dynamic Model Entity] as ModelEntity
  component [MCP Config Entity] as McpEntity
  component [Config Entity] as ConfigEntity
  component [Plan Execution Record] as RecordEntity
}

package "缓存管理" {
  component [Redis Cache] as RedisCache
  component [Local Cache] as LocalCache
  component [Cache Strategy Manager] as CacheStrategy
}

interface "Repository Interface" as IRepo
interface "Cache Interface" as ICache
interface "Configuration Interface" as IConfig

AgentConfig ..> IConfig
ModelConfig ..> IConfig
McpConfig ..> IConfig
SysConfig ..> IConfig

AgentRepo ..> IRepo
ModelRepo ..> IRepo
McpRepo ..> IRepo
ConfigRepo ..> IRepo
PlanRepo ..> IRepo
RecordRepo ..> IRepo

RedisCache ..> ICache
LocalCache ..> ICache
CacheStrategy ..> ICache

IRepo -- AgentEntity
IRepo -- ModelEntity
IRepo -- McpEntity
IRepo -- ConfigEntity
IRepo -- RecordEntity

IConfig -- IRepo
ICache -- IRepo

@enduml
```

### 4. MCP协议集成组件

```plantuml
@startuml
!theme plain
title JManus MCP协议集成组件

package "MCP核心" {
  component [MCP Service] as McpService
  component [MCP Client Manager] as ClientManager
  component [Connection Pool] as ConnectionPool
  component [Protocol Handler] as ProtocolHandler
}

package "工具管理" {
  component [Tool Registry] as ToolRegistry
  component [Tool Proxy] as ToolProxy
  component [Tool Cache] as ToolCache
  component [Tool Validator] as ToolValidator
}

package "连接管理" {
  component [Connection Monitor] as ConnMonitor
  component [Heartbeat Manager] as Heartbeat
  component [Reconnection Manager] as Reconnection
  component [Load Balancer] as LoadBalancer
}

package "消息处理" {
  component [Message Router] as MessageRouter
  component [Request Formatter] as RequestFormatter
  component [Response Parser] as ResponseParser
  component [Error Handler] as ErrorHandler
}

interface "MCP Protocol Interface" as IMCPProtocol
interface "Tool Interface" as ITool
interface "Connection Interface" as IConnection
interface "Message Interface" as IMessage

McpService ..> IMCPProtocol
ClientManager ..> IMCPProtocol
ProtocolHandler ..> IMCPProtocol

ToolRegistry ..> ITool
ToolProxy ..> ITool
ToolCache ..> ITool

ConnMonitor ..> IConnection
Heartbeat ..> IConnection
Reconnection ..> IConnection
LoadBalancer ..> IConnection

MessageRouter ..> IMessage
RequestFormatter ..> IMessage
ResponseParser ..> IMessage
ErrorHandler ..> IMessage

IMCPProtocol -- ITool
IMCPProtocol -- IConnection
IMCPProtocol -- IMessage

@enduml
```

## 部署视图组件

### 1. 微服务架构组件

```plantuml
@startuml
!theme plain
title JManus 微服务架构组件部署

package "Web层" {
  component [Nginx Load Balancer] as Nginx
  component [Vue3 Frontend] as Frontend
}

package "API网关层" {
  component [API Gateway] as Gateway
  component [Authentication Service] as Auth
  component [Rate Limiting Service] as RateLimit
}

package "核心服务" {
  component [Manus Core Service] as CoreService
  component [Agent Management Service] as AgentService
  component [Model Management Service] as ModelService
  component [MCP Integration Service] as McpService
}

package "工具服务" {
  component [Browser Tool Service] as BrowserService
  component [Python Execution Service] as PythonService
  component [Bash Execution Service] as BashService
}

package "数据层" {
  component [PostgreSQL Cluster] as PostgreSQL
  component [Redis Cluster] as Redis
  component [MongoDB] as MongoDB
}

package "外部服务" {
  component [DashScope API] as DashScope
  component [OpenAI API] as OpenAI
  component [MCP Servers] as McpServers
}

Frontend --> Gateway : HTTPS
Gateway --> CoreService : HTTP
Gateway --> AgentService : HTTP
Gateway --> ModelService : HTTP
Gateway --> McpService : HTTP

CoreService --> BrowserService : HTTP
CoreService --> PythonService : HTTP
CoreService --> BashService : HTTP

AgentService --> PostgreSQL : JDBC
ModelService --> PostgreSQL : JDBC
McpService --> PostgreSQL : JDBC

CoreService --> Redis : TCP
AgentService --> Redis : TCP

McpService --> McpServers : MCP Protocol
ModelService --> DashScope : HTTPS
ModelService --> OpenAI : HTTPS

@enduml
```

### 2. 单体架构组件

```plantuml
@startuml
!theme plain
title JManus 单体架构组件部署

package "JManus Application" {
  component [Spring Boot Application] as SpringApp
  
  package "Web Layer" {
    component [Vue3 Static Resources] as StaticResources
    component [REST Controllers] as Controllers
  }
  
  package "Service Layer" {
    component [Business Services] as Services
    component [Planning Engine] as PlanningEngine
    component [Agent Runtime] as AgentRuntime
  }
  
  package "Tool Layer" {
    component [Tool Implementations] as Tools
    component [MCP Client] as McpClient
  }
  
  package "Data Layer" {
    component [JPA Repositories] as Repositories
    component [Configuration Management] as ConfigMgmt
  }
}

package "Embedded Components" {
  component [H2 Database] as H2DB
  component [Local Cache] as LocalCache
}

package "External Dependencies" {
  component [Chrome Browser] as Chrome
  component [Python Runtime] as Python
  component [External MCP Servers] as ExtMcpServers
  component [AI Model APIs] as AIAPIs
}

SpringApp --> H2DB : JDBC
SpringApp --> LocalCache : Direct Access
SpringApp --> Chrome : WebDriver Protocol
SpringApp --> Python : Process Execution
SpringApp --> ExtMcpServers : MCP Protocol
SpringApp --> AIAPIs : HTTPS

@enduml
```

## 组件依赖关系

### 1. 核心依赖关系

```plantuml
@startuml
!theme plain
title JManus 组件依赖关系图

component [User Interface] as UI
component [API Gateway] as Gateway
component [Controller Layer] as Controllers
component [Service Layer] as Services
component [Core Engine] as Core
component [Agent Runtime] as Agents
component [Tool Ecosystem] as Tools
component [Data Access] as DataAccess
component [Infrastructure] as Infrastructure

UI --> Gateway : depends on
Gateway --> Controllers : depends on
Controllers --> Services : depends on
Services --> Core : depends on
Core --> Agents : depends on
Agents --> Tools : depends on
Services --> DataAccess : depends on
DataAccess --> Infrastructure : depends on

note right of UI
  用户界面层
  - Vue3 Web应用
  - REST API客户端
end note

note right of Gateway
  API网关层
  - 请求路由
  - 认证授权
  - 限流控制
end note

note right of Controllers
  控制器层
  - HTTP请求处理
  - 参数验证
  - 响应格式化
end note

note right of Services
  服务层
  - 业务逻辑
  - 事务管理
  - 协调组件
end note

note right of Core
  核心引擎
  - 计划生成
  - 执行协调
  - 状态管理
end note

note right of Agents
  智能体运行时
  - 智能体执行
  - 状态跟踪
  - 工具调用
end note

note right of Tools
  工具生态
  - 内置工具
  - 外部集成
  - MCP协议
end note

note right of DataAccess
  数据访问层
  - 数据持久化
  - 缓存管理
  - 配置存储
end note

note right of Infrastructure
  基础设施层
  - 数据库连接
  - 网络通信
  - 系统资源
end note

@enduml
```

### 2. 接口定义

```plantuml
@startuml
!theme plain
title JManus 组件接口定义

interface "User Interface" as IUI {
  +submitTask(request)
  +getTaskStatus(taskId)
  +getTaskResult(taskId)
}

interface "Planning Interface" as IPlan {
  +createPlan(context)
  +executePlan(plan)
  +finalizePlan(result)
}

interface "Agent Interface" as IAgent {
  +execute()
  +getState()
  +setState(state)
  +cleanup()
}

interface "Tool Interface" as ITool {
  +getName()
  +getDescription()
  +execute(input)
  +validate(input)
}

interface "Repository Interface" as IRepo {
  +save(entity)
  +findById(id)
  +findAll()
  +delete(id)
}

interface "Cache Interface" as ICache {
  +put(key, value)
  +get(key)
  +remove(key)
  +clear()
}

interface "MCP Interface" as IMCP {
  +connect(serverConfig)
  +callTool(toolName, params)
  +disconnect()
  +getStatus()
}

note top of IUI
  用户界面统一接口
  支持Web和API两种方式
end note

note top of IPlan
  计划管理核心接口
  定义计划生命周期
end note

note top of IAgent
  智能体执行接口
  管理智能体状态和行为
end note

note top of ITool
  工具调用统一接口
  支持多种工具类型
end note

note top of IRepo
  数据访问统一接口
  支持多种数据源
end note

note top of ICache
  缓存管理接口
  提供高性能数据访问
end note

note top of IMCP
  MCP协议集成接口
  支持外部工具集成
end note

@enduml
```

## 组件特性分析

### 模块化设计特点
1. **高内聚低耦合**: 每个组件职责明确，依赖关系清晰
2. **接口驱动**: 通过接口定义组件契约，支持多种实现
3. **可插拔架构**: 工具和服务可以动态加载和卸载
4. **分层设计**: 清晰的分层架构，便于维护和扩展

### 可扩展性支持
1. **水平扩展**: 支持微服务架构，可独立扩展各个服务
2. **垂直扩展**: 组件内部支持负载均衡和资源池
3. **工具扩展**: 开放的工具接口，支持第三方工具集成
4. **协议扩展**: 支持MCP等标准协议，便于生态集成

### 容错和可靠性
1. **故障隔离**: 组件间故障不会相互影响
2. **自动恢复**: 支持连接重试和状态恢复
3. **监控告警**: 内置监控和日志组件
4. **数据一致性**: 事务管理和数据同步机制

---

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**组件图数量**: 8个核心组件图  
**架构支持**: 单体架构和微服务架构  
**建模工具**: PlantUML