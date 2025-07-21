# JManus AI 智能助手平台 - 时序图 (Sequence Diagram)

本文档展示 JManus AI 智能助手平台的时序图，描述对象/组件间消息传递顺序，用于设计、开发和调试阶段。

## 文档说明

**使用场景**: 展示对象/组件间消息传递顺序  
**应用阶段**: 设计、开发、调试  
**关键优势**: 可视化流程、识别性能瓶颈  

## 核心业务流程时序图

### 1. 任务提交与执行流程

```plantuml
@startuml
!theme plain
skinparam participant {
  BackgroundColor lightblue
  BorderColor darkblue
}
skinparam actor {
  BackgroundColor lightyellow
  BorderColor darkred
}

title JManus 任务提交与执行时序图

actor "最终用户" as User
participant "Vue3 Web UI" as WebUI
participant "ManusController" as Controller
participant "PlanningFactory" as Factory
participant "PlanningCoordinator" as Coordinator
participant "PlanCreator" as Creator
participant "LlmService" as LLM
participant "PlanExecutor" as Executor
participant "BaseAgent" as Agent
participant "ToolExecutor" as Tool
participant "PlanExecutionRecorder" as Recorder
database "Database" as DB

User -> WebUI: 提交任务请求
activate WebUI
WebUI -> Controller: executeTask(request)
activate Controller

Controller -> Factory: createPlanningCoordinator(planId)
activate Factory
Factory -> Coordinator: new PlanningCoordinator()
activate Coordinator
Factory --> Controller: coordinator
deactivate Factory

Controller -> Coordinator: createAndExecutePlan(context)
Coordinator -> Creator: createPlan(context)
activate Creator

Creator -> LLM: generatePlan(userRequest)
activate LLM
LLM -> LLM: 调用AI服务生成计划
LLM --> Creator: planContent
deactivate LLM

Creator -> Recorder: recordPlanCreation(planId, content)
activate Recorder
Recorder -> DB: save(planRecord)
Recorder --> Creator: success
deactivate Recorder

Creator --> Coordinator: planCreated
deactivate Creator

Coordinator -> Executor: executeAllSteps(context)
activate Executor

loop 执行计划中的每个步骤
    Executor -> Agent: execute()
    activate Agent
    
    Agent -> Tool: executeWithTools()
    activate Tool
    Tool -> Tool: 执行具体工具操作
    Tool --> Agent: toolResult
    deactivate Tool
    
    Agent -> LLM: getNextStep(context)
    LLM --> Agent: nextAction
    Agent --> Executor: stepResult
    deactivate Agent
end

Executor --> Coordinator: executionComplete
deactivate Executor

Coordinator -> Recorder: recordExecutionResult(planId, result)
Recorder -> DB: updateRecord(result)
Recorder --> Coordinator: recorded
deactivate Recorder

Coordinator --> Controller: executionResult
deactivate Coordinator

Controller --> WebUI: ResponseEntity<ExecutionResult>
deactivate Controller

WebUI --> User: 显示执行结果
deactivate WebUI

@enduml
```

### 2. 智能体动态加载流程

```plantuml
@startuml
!theme plain
title JManus 智能体动态加载时序图

participant "AgentService" as Service
participant "DynamicAgentRepository" as Repository
participant "DynamicAgentLoader" as Loader
participant "BaseAgent" as Agent
participant "ToolCallingManager" as ToolManager
participant "LlmService" as LLM
database "Database" as DB

Service -> Repository: findByName(agentName)
activate Repository
Repository -> DB: SELECT * FROM dynamic_agent WHERE name = ?
DB --> Repository: agentEntity
Repository --> Service: DynamicAgentEntity
deactivate Repository

Service -> Loader: loadAgent(agentEntity)
activate Loader

Loader -> Agent: createDynamicAgent(config)
activate Agent
Agent -> LLM: initializeLLMConnection()
LLM --> Agent: connectionReady
Agent -> ToolManager: loadAvailableTools(toolList)
activate ToolManager
ToolManager -> ToolManager: 初始化工具集合
ToolManager --> Agent: toolsLoaded
deactivate ToolManager
Agent --> Loader: agentInstance
deactivate Agent

Loader --> Service: loadedAgent
deactivate Loader

@enduml
```

### 3. MCP工具调用流程

```plantuml
@startuml
!theme plain
title JManus MCP工具调用时序图

participant "BaseAgent" as Agent
participant "McpService" as McpService
participant "McpClient" as McpClient
participant "External MCP Server" as McpServer
participant "ToolExecutor" as Tool
participant "CacheManager" as Cache

Agent -> Tool: executeMcpTool(toolName, params)
activate Tool

Tool -> Cache: checkCache(toolName, params)
activate Cache
Cache --> Tool: cacheResult (可能为null)
deactivate Cache

alt 缓存未命中
    Tool -> McpService: callMcpTool(toolName, params)
    activate McpService
    
    McpService -> McpClient: sendRequest(toolName, params)
    activate McpClient
    
    McpClient -> McpServer: HTTP/WebSocket Request
    activate McpServer
    McpServer -> McpServer: 执行工具逻辑
    McpServer --> McpClient: toolResult
    deactivate McpServer
    
    McpClient --> McpService: response
    deactivate McpClient
    
    McpService --> Tool: mcpResult
    deactivate McpService
    
    Tool -> Cache: storeCache(toolName, params, result)
    Cache --> Tool: cached
else 缓存命中
    Tool -> Tool: 使用缓存结果
end

Tool --> Agent: toolExecutionResult
deactivate Tool

@enduml
```

### 4. 用户输入处理流程

```plantuml
@startuml
!theme plain
title JManus 用户输入处理时序图

participant "BaseAgent" as Agent
participant "FormInputTool" as FormTool
participant "UserInputService" as InputService
participant "Vue3 Web UI" as WebUI
participant "ManusController" as Controller
actor "最终用户" as User

Agent -> FormTool: run(formInputRequest)
activate FormTool

FormTool -> InputService: storeFormInputTool(planId, tool)
activate InputService
InputService -> InputService: 存储表单工具实例
InputService --> FormTool: stored
deactivate InputService

FormTool -> InputService: waitForUserInput(planId, timeout)
InputService -> InputService: 创建等待状态
InputService -> WebUI: 通知需要用户输入
activate WebUI

WebUI -> User: 显示输入表单
User -> WebUI: 填写并提交表单
WebUI -> Controller: submitUserInput(planId, inputs)
activate Controller

Controller -> InputService: submitUserInput(planId, inputs)
InputService -> InputService: 处理用户输入
InputService --> Controller: inputProcessed
deactivate Controller

InputService --> FormTool: userInputResult
deactivate WebUI

FormTool --> Agent: inputResult
deactivate FormTool

@enduml
```

### 5. 计划模板管理流程

```plantuml
@startuml
!theme plain
title JManus 计划模板管理时序图

actor "系统管理员" as Admin
participant "Vue3 Web UI" as WebUI
participant "PlanTemplateController" as Controller
participant "PlanTemplateService" as Service
participant "PromptService" as PromptService
participant "LlmService" as LLM
database "Database" as DB

Admin -> WebUI: 创建计划模板
activate WebUI

WebUI -> Controller: createPlanTemplate(request)
activate Controller

Controller -> Service: createTemplate(templateData)
activate Service

Service -> PromptService: renderPromptTemplate(template, variables)
activate PromptService
PromptService -> PromptService: 处理模板变量
PromptService --> Service: renderedPrompt
deactivate PromptService

Service -> LLM: validatePlanTemplate(prompt)
activate LLM
LLM -> LLM: 验证模板有效性
LLM --> Service: validationResult
deactivate LLM

alt 模板有效
    Service -> DB: savePlanTemplate(template)
    DB --> Service: templateSaved
    Service --> Controller: success(templateId)
else 模板无效
    Service --> Controller: error(validationError)
end
deactivate Service

Controller --> WebUI: ResponseEntity<Result>
deactivate Controller

WebUI --> Admin: 显示创建结果
deactivate WebUI

@enduml
```

### 6. 系统配置管理流程

```plantuml
@startuml
!theme plain
title JManus 系统配置管理时序图

actor "系统管理员" as Admin
participant "ConfigController" as Controller
participant "ConfigService" as Service
participant "ConfigValidator" as Validator
participant "ConfigRepository" as Repository
database "Database" as DB

Admin -> Controller: batchUpdateConfigs(configs)
activate Controller

Controller -> Service: batchUpdate(configList)
activate Service

loop 处理每个配置项
    Service -> Validator: validateConfig(config)
    activate Validator
    Validator -> Validator: 验证配置格式和值
    Validator --> Service: validationResult
    deactivate Validator
    
    alt 配置有效
        Service -> Repository: save(config)
        activate Repository
        Repository -> DB: UPDATE config SET value = ? WHERE key = ?
        DB --> Repository: updateResult
        Repository --> Service: saved
        deactivate Repository
    else 配置无效
        Service -> Service: 记录错误信息
    end
end

Service --> Controller: batchUpdateResult
deactivate Service

Controller --> Admin: ResponseEntity<BatchResult>
deactivate Controller

@enduml
```

### 7. AI模型切换流程

```plantuml
@startuml
!theme plain
title JManus AI模型切换时序图

actor "系统管理员" as Admin
participant "ModelController" as Controller
participant "ModelService" as Service
participant "LlmService" as LlmService
participant "ConnectionPool" as Pool
participant "External AI Service" as AIService

Admin -> Controller: switchModel(newModelConfig)
activate Controller

Controller -> Service: switchToModel(modelConfig)
activate Service

Service -> Service: 验证模型配置
Service -> LlmService: createNewConnection(modelConfig)
activate LlmService

LlmService -> AIService: testConnection(config)
activate AIService
AIService --> LlmService: connectionTest
deactivate AIService

alt 连接成功
    LlmService -> Pool: updateConnectionPool(newConnection)
    activate Pool
    Pool -> Pool: 关闭旧连接，建立新连接
    Pool --> LlmService: poolUpdated
    deactivate Pool
    
    LlmService --> Service: switchSuccess
    Service --> Controller: success
else 连接失败
    LlmService --> Service: connectionError
    Service --> Controller: error(message)
end
deactivate LlmService

deactivate Service
Controller --> Admin: ResponseEntity<Result>
deactivate Controller

@enduml
```

## 异常处理时序图

### 任务执行异常处理

```plantuml
@startuml
!theme plain
title JManus 任务执行异常处理时序图

participant "PlanExecutor" as Executor
participant "BaseAgent" as Agent
participant "ErrorHandler" as ErrorHandler
participant "PlanExecutionRecorder" as Recorder
participant "NotificationService" as Notification

Executor -> Agent: execute()
activate Agent

Agent -> Agent: 执行任务步骤
Agent -> Agent: 发生异常
Agent -> ErrorHandler: handleException(exception, context)
activate ErrorHandler

ErrorHandler -> ErrorHandler: 分析异常类型
ErrorHandler -> Recorder: recordError(planId, error)
activate Recorder
Recorder --> ErrorHandler: errorRecorded
deactivate Recorder

alt 可恢复异常
    ErrorHandler -> Agent: suggestRetry(retryStrategy)
    Agent -> Agent: 重试执行
    Agent --> Executor: retryResult
else 不可恢复异常
    ErrorHandler -> Notification: notifyFailure(planId, error)
    activate Notification
    Notification --> ErrorHandler: notificationSent
    deactivate Notification
    
    ErrorHandler --> Agent: terminateExecution
    Agent --> Executor: executionFailed
end

deactivate ErrorHandler
deactivate Agent

@enduml
```

## 性能优化时序图

### 并行执行时序图

```plantuml
@startuml
!theme plain
title JManus MapReduce并行执行时序图

participant "MapReducePlanExecutor" as Executor
participant "ExecutorService" as ThreadPool
participant "Agent1" as Agent1
participant "Agent2" as Agent2
participant "Agent3" as Agent3
participant "ResultAggregator" as Aggregator

Executor -> ThreadPool: submitMapTasks(tasks)
activate ThreadPool

par 并行执行Map阶段
    ThreadPool -> Agent1: executeMapTask(task1)
    activate Agent1
    Agent1 -> Agent1: 处理数据分片1
    Agent1 --> ThreadPool: result1
    deactivate Agent1
and
    ThreadPool -> Agent2: executeMapTask(task2)
    activate Agent2
    Agent2 -> Agent2: 处理数据分片2
    Agent2 --> ThreadPool: result2
    deactivate Agent2
and
    ThreadPool -> Agent3: executeMapTask(task3)
    activate Agent3
    Agent3 -> Agent3: 处理数据分片3
    Agent3 --> ThreadPool: result3
    deactivate Agent3
end

ThreadPool -> Aggregator: aggregate(results)
activate Aggregator
Aggregator -> Aggregator: 合并处理结果
Aggregator --> ThreadPool: finalResult
deactivate Aggregator

ThreadPool --> Executor: mapReduceComplete
deactivate ThreadPool

@enduml
```

## 系统集成时序图

### API调用集成时序图

```plantuml
@startuml
!theme plain
title JManus API调用集成时序图

actor "外部系统" as ExternalSystem
participant "API Gateway" as Gateway
participant "AuthenticationFilter" as Auth
participant "RateLimitFilter" as RateLimit
participant "ManusController" as Controller
participant "ValidationService" as Validator
participant "BusinessService" as Service

ExternalSystem -> Gateway: HTTP Request
activate Gateway

Gateway -> Auth: authenticate(request)
activate Auth
Auth -> Auth: 验证API密钥
Auth --> Gateway: authResult
deactivate Auth

alt 认证成功
    Gateway -> RateLimit: checkRateLimit(clientId)
    activate RateLimit
    RateLimit -> RateLimit: 检查调用频率
    RateLimit --> Gateway: rateLimitResult
    deactivate RateLimit
    
    alt 未超过限制
        Gateway -> Controller: routeRequest(request)
        activate Controller
        
        Controller -> Validator: validateRequest(request)
        activate Validator
        Validator --> Controller: validationResult
        deactivate Validator
        
        alt 请求有效
            Controller -> Service: processRequest(request)
            activate Service
            Service -> Service: 业务处理
            Service --> Controller: businessResult
            deactivate Service
            
            Controller --> Gateway: success(result)
        else 请求无效
            Controller --> Gateway: badRequest(errors)
        end
        deactivate Controller
        
    else 超过限制
        Gateway --> ExternalSystem: rateLimitExceeded
    end
else 认证失败
    Gateway --> ExternalSystem: unauthorized
end

Gateway --> ExternalSystem: HTTP Response
deactivate Gateway

@enduml
```

## 关键性能指标

### 时序分析要点

1. **关键路径识别**
   - 任务提交到执行完成的端到端时间
   - AI模型调用响应时间
   - 数据库操作延迟

2. **并发处理能力**
   - MapReduce并行执行效率
   - 多智能体协作性能
   - 系统资源利用率

3. **异常恢复时间**
   - 错误检测和处理速度
   - 系统自愈能力
   - 故障转移时间

4. **集成接口性能**
   - API响应时间
   - 外部服务调用延迟
   - 数据传输效率

---

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**时序图数量**: 10个核心时序图  
**涵盖场景**: 任务执行、智能体管理、工具调用、异常处理等  
**建模工具**: PlantUML