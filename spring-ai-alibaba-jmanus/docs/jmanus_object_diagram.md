# JManus AI 智能助手平台 - 对象图 (Object Diagram)

本文档展示 JManus AI 智能助手平台的对象图，快照式展示对象实例及关联，用于调试与示例。

## 文档说明

**使用场景**: 快照式展示对象实例及关联  
**应用阶段**: 调试与示例  
**关键优势**: 具体实例演示，便于问题复现  

## 核心业务场景对象图

### 1. 任务执行实例快照

```plantuml
@startuml
!theme plain
skinparam object {
  BackgroundColor lightblue
  BorderColor darkblue
}

title JManus 任务执行实例快照 - 2025年1月15日 14:30:00

object "taskReq001 : TaskExecutionRequest" as taskReq {
  planId = "plan-20250115-001"
  userRequest = "分析最近一周的销售数据并生成报表"
  useMemory = true
  namespace = "data-analysis"
  submitTime = "2025-01-15T14:30:00Z"
}

object "context001 : ExecutionContext" as context {
  currentPlanId = "plan-20250115-001"
  rootPlanId = "plan-20250115-001"
  userRequest = "分析最近一周的销售数据并生成报表"
  useMemory = true
  success = false
  currentStatus = "EXECUTING"
}

object "plan001 : ExecutionPlan" as plan {
  planId = "plan-20250115-001"
  planType = "SEQUENTIAL"
  totalSteps = 4
  currentStep = 2
  status = "IN_PROGRESS"
}

object "step001 : ExecutionStep" as step1 {
  stepIndex = 1
  stepRequirement = "收集销售数据"
  status = "COMPLETED"
  agentName = "DataCollectorAgent"
  startTime = "2025-01-15T14:30:05Z"
  endTime = "2025-01-15T14:31:20Z"
}

object "step002 : ExecutionStep" as step2 {
  stepIndex = 2
  stepRequirement = "数据清洗和预处理"
  status = "IN_PROGRESS"
  agentName = "DataProcessorAgent"
  startTime = "2025-01-15T14:31:25Z"
  endTime = null
}

object "agent001 : DataCollectorAgent" as agent1 {
  agentId = "agent-collector-001"
  name = "DataCollectorAgent"
  state = "COMPLETED"
  currentPlanId = "plan-20250115-001"
  maxSteps = 10
  currentStep = 3
}

object "agent002 : DataProcessorAgent" as agent2 {
  agentId = "agent-processor-001"
  name = "DataProcessorAgent"
  state = "IN_PROGRESS"
  currentPlanId = "plan-20250115-001"
  maxSteps = 15
  currentStep = 5
}

object "record001 : PlanExecutionRecordEntity" as record {
  id = 1001
  planId = "plan-20250115-001"
  rootPlanId = "plan-20250115-001"
  userRequest = "分析最近一周的销售数据并生成报表"
  executionStatus = "IN_PROGRESS"
  startTime = "2025-01-15T14:30:00Z"
  endTime = null
}

' 对象关联关系
taskReq --> context : creates
context --> plan : contains
plan --> step1 : includes
plan --> step2 : includes
step1 --> agent1 : executed_by
step2 --> agent2 : executing_by
context --> record : recorded_in

@enduml
```

### 2. 智能体配置实例

```plantuml
@startuml
!theme plain
title JManus 智能体配置实例快照

object "agentConfig001 : AgentConfig" as agentConfig {
  id = "data-analyst-v1.2"
  name = "DataAnalystAgent"
  description = "专门用于数据分析的智能体"
  namespace = "data-analysis"
  version = "1.2.0"
  enabled = true
}

object "modelConfig001 : ModelConfig" as modelConfig {
  id = 1001
  baseUrl = "https://dashscope.aliyuncs.com/api/v1"
  modelName = "qwen-max"
  modelDescription = "通义千问最大版本"
  type = "DASHSCOPE"
  maxTokens = 8192
  temperature = 0.7
}

object "toolList001 : List<String>" as toolList {
  tools = ["python_execute", "data_loader", "chart_generator", "file_manager"]
}

object "promptTemplate001 : String" as promptTemplate {
  template = "你是一个专业的数据分析师..."
  variables = ["user_request", "data_context", "analysis_type"]
}

object "agentEntity001 : DynamicAgentEntity" as agentEntity {
  id = 1001
  name = "DataAnalystAgent"
  description = "专门用于数据分析的智能体"
  namespace = "data-analysis"
  availableTools = "python_execute,data_loader,chart_generator,file_manager"
  nextStepPrompt = "根据用户需求进行数据分析..."
  createTime = "2025-01-10T10:00:00Z"
  updateTime = "2025-01-15T09:30:00Z"
}

' 对象关联
agentConfig --> modelConfig : uses
agentConfig --> toolList : hasTools
agentConfig --> promptTemplate : usesTemplate
agentEntity --> agentConfig : mapsTo

@enduml
```

### 3. MCP工具调用实例

```plantuml
@startuml
!theme plain
title JManus MCP工具调用实例快照

object "mcpConfig001 : McpConfigEntity" as mcpConfig {
  id = 1001
  serverName = "data-tools-server"
  command = "node"
  args = ["/app/mcp-server.js"]
  env = {"NODE_ENV": "production", "API_KEY": "***"}
  cwd = "/app"
  timeout = 30000
  enabled = true
}

object "mcpClient001 : McpClient" as mcpClient {
  serverId = "data-tools-server"
  connectionId = "conn-20250115-001"
  status = "CONNECTED"
  lastHeartbeat = "2025-01-15T14:32:15Z"
  protocolVersion = "0.1.0"
}

object "toolCall001 : McpToolCall" as toolCall {
  callId = "call-20250115-001"
  toolName = "excel_analyzer"
  methodName = "analyze_sales_data"
  status = "IN_PROGRESS"
  startTime = "2025-01-15T14:32:00Z"
  timeout = 60000
}

object "toolParams001 : Map<String, Object>" as toolParams {
  file_path = "/data/sales_2025_w2.xlsx"
  analysis_type = "weekly_summary"
  include_charts = true
  output_format = "json"
}

object "toolResult001 : ToolExecuteResult" as toolResult {
  success = true
  resultType = "JSON"
  executionTime = 2500
  message = "数据分析完成"
  data = "{...analysis results...}"
}

object "cacheEntry001 : CacheEntry" as cacheEntry {
  key = "excel_analyzer:analyze_sales_data:hash123"
  value = "{...cached result...}"
  expireTime = "2025-01-15T15:32:00Z"
  hitCount = 3
}

' 对象关联
mcpConfig --> mcpClient : configures
mcpClient --> toolCall : executes
toolCall --> toolParams : hasParameters
toolCall --> toolResult : produces
toolResult --> cacheEntry : cached_as

@enduml
```

### 4. 工具执行链实例

```plantuml
@startuml
!theme plain
title JManus 工具执行链实例快照

object "browserTool001 : BrowserTool" as browserTool {
  toolId = "browser-tool-001"
  browserType = "CHROME"
  headless = true
  windowSize = "1920x1080"
  status = "RUNNING"
}

object "webDriver001 : WebDriver" as webDriver {
  sessionId = "session-chrome-001"
  browserVersion = "120.0.6099.109"
  driverVersion = "120.0.6099.109"
  currentUrl = "https://example.com/dashboard"
  windowHandle = "CDwindow-123"
}

object "elementLocator001 : WebElement" as element {
  elementId = "element-btn-download"
  tagName = "button"
  text = "下载报表"
  xpath = "//button[@id='download-btn']"
  displayed = true
  enabled = true
}

object "pythonTool001 : PythonExecuteTool" as pythonTool {
  toolId = "python-tool-001"
  pythonVersion = "3.11.5"
  virtualEnv = "/venv/data-analysis"
  workingDir = "/tmp/jmanus/plan-20250115-001"
  status = "IDLE"
}

object "pythonScript001 : PythonScript" as pythonScript {
  scriptId = "script-data-analysis-001"
  fileName = "analyze_data.py"
  codeLength = 1250
  imports = ["pandas", "matplotlib", "numpy"]
  lastModified = "2025-01-15T14:25:00Z"
}

object "execution001 : ScriptExecution" as execution {
  executionId = "exec-001"
  startTime = "2025-01-15T14:30:00Z"
  endTime = "2025-01-15T14:31:45Z"
  exitCode = 0
  outputLines = 45
  errorLines = 0
}

' 工具链关联
browserTool --> webDriver : controls
webDriver --> element : locates
pythonTool --> pythonScript : executes
pythonScript --> execution : produces

@enduml
```

### 5. 数据流转实例

```plantuml
@startuml
!theme plain
title JManus 数据流转实例快照

object "userInput001 : UserFormInput" as userInput {
  planId = "plan-20250115-001"
  formId = "data-source-selection"
  fields = {"database": "sales_db", "table": "orders", "date_range": "2025-01-08 to 2025-01-14"}
  submitTime = "2025-01-15T14:28:30Z"
}

object "promptContext001 : PromptContext" as promptContext {
  templateId = "data-analysis-prompt-v1"
  variables = {"user_request": "...", "data_source": "sales_db.orders", "analysis_period": "week"}
  renderedPrompt = "请分析sales_db.orders表中2025年第2周的销售数据..."
  tokenCount = 256
}

object "llmRequest001 : LlmRequest" as llmRequest {
  requestId = "llm-req-20250115-001"
  model = "qwen-max"
  messages = [{"role": "system", "content": "..."}, {"role": "user", "content": "..."}]
  temperature = 0.7
  maxTokens = 2048
  timestamp = "2025-01-15T14:30:15Z"
}

object "llmResponse001 : LlmResponse" as llmResponse {
  responseId = "llm-resp-20250115-001"
  content = "我需要执行以下步骤来分析销售数据：\n1. 连接数据库\n2. 查询订单数据\n3. 计算统计指标\n4. 生成可视化图表"
  usage = {"prompt_tokens": 256, "completion_tokens": 128, "total_tokens": 384}
  finishReason = "stop"
  responseTime = 1500
}

object "toolCallDecision001 : ToolCallDecision" as toolDecision {
  decisionId = "decision-001"
  selectedTool = "python_execute"
  reason = "需要执行数据库查询和统计分析"
  confidence = 0.95
  alternatives = ["data_loader", "sql_executor"]
}

object "executionResult001 : ExecutionResult" as execResult {
  resultId = "result-20250115-001"
  success = true
  data = {"total_orders": 1250, "total_revenue": 125000.50, "avg_order_value": 100.00}
  charts = [{"type": "bar", "file": "sales_by_day.png"}, {"type": "pie", "file": "category_distribution.png"}]
  executionTime = 5500
}

' 数据流转关联
userInput --> promptContext : provides_data
promptContext --> llmRequest : generates
llmRequest --> llmResponse : produces
llmResponse --> toolDecision : leads_to
toolDecision --> execResult : results_in

@enduml
```

## 调试场景对象图

### 1. 错误处理实例

```plantuml
@startuml
!theme plain
title JManus 错误处理实例快照

object "errorContext001 : ErrorContext" as errorContext {
  errorId = "error-20250115-001"
  planId = "plan-20250115-001"
  stepIndex = 3
  errorType = "TOOL_EXECUTION_FAILED"
  timestamp = "2025-01-15T14:33:45Z"
  severity = "HIGH"
}

object "exception001 : ToolExecutionException" as exception {
  message = "Python script execution failed: ModuleNotFoundError: No module named 'pandas'"
  stackTrace = "Traceback (most recent call last):\n  File 'analyze_data.py', line 1, in <module>\n    import pandas as pd\nModuleNotFoundError: No module named 'pandas'"
  errorCode = "MODULE_NOT_FOUND"
  causeType = "DEPENDENCY_MISSING"
}

object "retryStrategy001 : RetryStrategy" as retryStrategy {
  maxRetries = 3
  currentAttempt = 1
  retryInterval = 5000
  backoffMultiplier = 2.0
  lastAttemptTime = "2025-01-15T14:33:45Z"
  nextRetryTime = "2025-01-15T14:33:50Z"
}

object "errorRecovery001 : ErrorRecoveryAction" as recovery {
  recoveryId = "recovery-001"
  actionType = "INSTALL_DEPENDENCY"
  description = "安装缺失的pandas模块"
  script = "pip install pandas"
  estimatedTime = 30000
  autoExecute = true
}

object "errorLog001 : ErrorLogEntry" as errorLog {
  logId = "log-error-001"
  level = "ERROR"
  message = "工具执行失败，尝试自动恢复"
  context = {"planId": "plan-20250115-001", "tool": "python_execute", "error": "MODULE_NOT_FOUND"}
  timestamp = "2025-01-15T14:33:45Z"
}

' 错误处理关联
errorContext --> exception : contains
errorContext --> retryStrategy : uses
errorContext --> recovery : triggers
errorContext --> errorLog : logged_as

@enduml
```

### 2. 缓存系统实例

```plantuml
@startuml
!theme plain
title JManus 缓存系统实例快照

object "cacheManager001 : CacheManager" as cacheManager {
  instanceId = "cache-mgr-001"
  cacheType = "REDIS_CLUSTER"
  totalMemory = "4GB"
  usedMemory = "1.2GB"
  hitRate = 0.78
  totalOps = 15420
}

object "promptCache001 : CacheEntry" as promptCache {
  key = "prompt:data-analysis-v1:hash456"
  value = "请分析以下销售数据..."
  size = 1024
  ttl = 3600
  createdTime = "2025-01-15T14:25:00Z"
  lastAccessed = "2025-01-15T14:32:15Z"
  accessCount = 5
}

object "modelCache001 : CacheEntry" as modelCache {
  key = "model:qwen-max:response:hash789"
  value = "{\"content\": \"数据分析结果...\", \"usage\": {...}}"
  size = 2048
  ttl = 1800
  createdTime = "2025-01-15T14:30:15Z"
  lastAccessed = "2025-01-15T14:30:15Z"
  accessCount = 1
}

object "toolCache001 : CacheEntry" as toolCache {
  key = "tool:python_execute:result:hash101"
  value = "{\"success\": true, \"data\": {...}}"
  size = 4096
  ttl = 7200
  createdTime = "2025-01-15T14:31:45Z"
  lastAccessed = "2025-01-15T14:32:00Z"
  accessCount = 2
}

object "cacheStats001 : CacheStatistics" as cacheStats {
  totalEntries = 1250
  hitCount = 978
  missCount = 272
  evictionCount = 45
  averageLoadTime = 150
  lastResetTime = "2025-01-15T00:00:00Z"
}

' 缓存关联
cacheManager --> promptCache : manages
cacheManager --> modelCache : manages
cacheManager --> toolCache : manages
cacheManager --> cacheStats : tracks

@enduml
```

### 3. 配置加载实例

```plantuml
@startuml
!theme plain
title JManus 配置加载实例快照

object "configSource001 : ConfigurationSource" as configSource {
  sourceType = "FILE"
  filePath = "/app/config/application.yml"
  lastModified = "2025-01-15T10:00:00Z"
  checksum = "md5:a1b2c3d4e5f6"
  priority = 100
}

object "springConfig001 : SpringConfiguration" as springConfig {
  profileActive = "production"
  serverPort = 8080
  datasourceUrl = "jdbc:h2:mem:jmanus"
  redisHost = "localhost"
  redisPort = 6379
  loaded = true
}

object "manusProps001 : ManusProperties" as manusProps {
  browserHeadless = true
  browserRequestTimeout = 30000
  debugDetail = false
  maxSteps = 20
  defaultNamespace = "default"
  enableMcp = true
}

object "modelConfigs001 : List<ModelConfig>" as modelConfigs {
  size = 3
  configs = [
    {"name": "qwen-max", "baseUrl": "https://dashscope.aliyuncs.com/api/v1"},
    {"name": "gpt-4", "baseUrl": "https://api.openai.com/v1"},
    {"name": "claude-3", "baseUrl": "https://api.anthropic.com/v1"}
  ]
}

object "toolConfigs001 : List<ToolConfig>" as toolConfigs {
  size = 8
  enabledTools = ["python_execute", "browser", "bash", "form_input", "planning", "mcp_integration"]
  disabledTools = ["deprecated_tool", "experimental_tool"]
}

' 配置加载关联
configSource --> springConfig : loads
configSource --> manusProps : provides
springConfig --> modelConfigs : contains
springConfig --> toolConfigs : contains

@enduml
```

## 性能监控对象图

### 1. 系统监控实例

```plantuml
@startuml
!theme plain
title JManus 系统监控实例快照

object "systemMetrics001 : SystemMetrics" as systemMetrics {
  timestamp = "2025-01-15T14:35:00Z"
  cpuUsage = 0.45
  memoryUsage = 0.62
  diskUsage = 0.38
  networkIn = "1.2MB/s"
  networkOut = "0.8MB/s"
}

object "jvmMetrics001 : JvmMetrics" as jvmMetrics {
  heapUsed = "512MB"
  heapMax = "1024MB"
  nonHeapUsed = "128MB"
  gcCount = 25
  gcTime = "1.2s"
  threadCount = 45
  activeThreads = 12
}

object "appMetrics001 : ApplicationMetrics" as appMetrics {
  activePlans = 3
  completedPlans = 127
  failedPlans = 8
  averageExecutionTime = "45.6s"
  qps = 12.5
  errorRate = 0.02
}

object "toolMetrics001 : ToolMetrics" as toolMetrics {
  browserToolCalls = 45
  pythonToolCalls = 89
  mcpToolCalls = 23
  averageToolTime = "2.3s"
  toolSuccessRate = 0.94
  toolTimeouts = 3
}

object "performanceAlert001 : PerformanceAlert" as perfAlert {
  alertId = "alert-20250115-001"
  alertType = "HIGH_MEMORY_USAGE"
  severity = "WARNING"
  threshold = 0.80
  currentValue = 0.85
  message = "内存使用率超过阈值"
  triggered = true
  triggerTime = "2025-01-15T14:34:30Z"
}

' 监控关联
systemMetrics --> perfAlert : triggers
jvmMetrics --> perfAlert : contributes_to
appMetrics --> perfAlert : influences
toolMetrics --> appMetrics : feeds_into

@enduml
```

### 2. 用户会话实例

```plantuml
@startuml
!theme plain
title JManus 用户会话实例快照

object "userSession001 : UserSession" as userSession {
  sessionId = "session-user-001"
  userId = "user123"
  userType = "API_CLIENT"
  ipAddress = "192.168.1.100"
  userAgent = "JManus-Client/1.0"
  loginTime = "2025-01-15T14:00:00Z"
  lastActivity = "2025-01-15T14:34:45Z"
  sessionTimeout = 1800
}

object "apiKey001 : ApiKey" as apiKey {
  keyId = "api-key-001"
  keyName = "DataAnalysisBot"
  hashedKey = "sha256:abc123..."
  permissions = ["TASK_SUBMIT", "TASK_QUERY", "AGENT_LIST"]
  rateLimit = 100
  currentUsage = 23
  resetTime = "2025-01-15T15:00:00Z"
}

object "requestLog001 : RequestLog" as requestLog {
  requestId = "req-20250115-045"
  method = "POST"
  endpoint = "/api/v1/tasks/execute"
  statusCode = 200
  responseTime = 250
  requestSize = 1024
  responseSize = 2048
  timestamp = "2025-01-15T14:34:45Z"
}

object "rateLimiter001 : RateLimiter" as rateLimiter {
  limiterId = "rate-limiter-001"
  windowSize = 3600
  maxRequests = 100
  currentRequests = 23
  windowStart = "2025-01-15T14:00:00Z"
  resetTime = "2025-01-15T15:00:00Z"
}

' 会话关联
userSession --> apiKey : authenticated_with
userSession --> requestLog : generates
apiKey --> rateLimiter : subject_to
requestLog --> rateLimiter : counted_by

@enduml
```

## 对象图使用指南

### 调试价值

1. **问题定位**: 通过对象实例快照快速定位问题发生的具体环节
2. **数据流追踪**: 清晰展示数据在系统中的流转路径和状态变化
3. **状态验证**: 验证对象在特定时刻的状态是否符合预期
4. **性能分析**: 通过具体实例分析系统性能瓶颈

### 示例演示

1. **功能展示**: 为新用户展示系统功能的具体运行情况
2. **测试用例**: 作为测试用例的数据基础和验证标准
3. **文档补充**: 补充类图和时序图，提供更直观的理解
4. **培训材料**: 用于开发团队培训和知识传递

### 维护建议

1. **定期更新**: 随着系统演进更新对象图中的实例数据
2. **版本管理**: 为不同版本的系统维护对应的对象图
3. **场景覆盖**: 覆盖正常流程、异常情况、边界条件等多种场景
4. **数据脱敏**: 在文档中使用脱敏后的示例数据

---

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**对象图数量**: 12个核心对象图  
**涵盖场景**: 业务执行、配置管理、错误处理、性能监控等  
**建模工具**: PlantUML