# JManus 复合计划执行流程对象图 (Object Diagram)

本文档展示 JManus 系统在处理复合计划"用浏览器基于百度，查询今天阿里巴巴的股价，并返回最新股价"时的核心对象及其在特定时刻的关系。对象图提供了系统结构的快照，捕捉了存在的实例及其关联的静态视图。

## 用例场景

用户输入："用浏览器基于百度，查询今天阿里巴巴的股价，并返回最新股价"，系统需要理解复合任务需求，分解为多步骤计划，使用浏览器工具执行网页搜索，提取股价信息并返回结果。

## PlantUML 对象图

```plantuml
@startuml Complex_Plan_Dialog_Object_Diagram
!theme plain

title JManus 复合计划执行流程对象图\n用例：股价查询任务

' 用户请求对象
object "request: ExecutionRequest" as request {
  userRequest = "用浏览器基于百度，查询今天阿里巴巴的股价，并返回最新股价"
  agentName = "browser-agent"
  planType = "COMPLEX"
  maxSteps = 10
  requiredTools = ["BrowserUseTool"]
  terminateColumns = []
}

' AI模型配置对象
object "model: DynamicModelEntity" as model {
  id = 1
  modelName = "qwen-max"
  baseUrl = "https://dashscope.aliyuncs.com/api/v1"
  apiKey = "sk-****"
  type = "CHAT"
  modelDescription = "阿里云通义千问大模型"
  provider = "dashscope"
}

' 浏览器智能体配置对象
object "browserAgent: DynamicAgentEntity" as browserAgent {
  id = 2
  agentName = "browser-agent"
  agentDescription = "浏览器自动化智能体"
  nextStepPrompt = "你是一个专业的浏览器自动化智能体..."
  className = "com.alibaba.cloud.ai.example.manus.agent.ReActAgent"
  namespace = "browser"
  availableToolKeys = ["browser-use-tool", "llm-extract-tool"]
}

' 复合执行计划对象
object "complexPlan: ExecutionPlan" as complexPlan {
  planId = "plan-20250115-stock-002"
  planType = "COMPLEX"
  totalSteps = 7
  currentStepIndex = 3
  steps = [
    "初始化浏览器环境",
    "导航到百度搜索首页",
    "执行搜索查询",
    "等待搜索结果加载",
    "解析搜索结果页面",
    "提取股价关键信息",
    "验证数据并整理回答"
  ]
}

' 计划执行记录对象
object "planRecord: PlanExecutionRecord" as planRecord {
  id = 3001
  currentPlanId = "plan-20250115-stock-002"
  rootPlanId = "plan-20250115-stock-002"
  title = "阿里巴巴股价查询任务"
  userRequest = "用浏览器基于百度，查询今天阿里巴巴的股价"
  startTime = "2025-01-15T14:00:00"
  endTime = "2025-01-15T14:01:45"
  completed = true
  summary = "成功获取阿里巴巴最新股价信息"
  modelName = "qwen-max"
  currentStepIndex = 6
}

' 智能体执行记录对象
object "agentRecord: AgentExecutionRecord" as agentRecord {
  id = 4001
  conversationId = "conv-stock-001"
  agentName = "browser-agent"
  agentDescription = "浏览器自动化智能体"
  startTime = "2025-01-15T14:00:00"
  endTime = "2025-01-15T14:01:45"
  maxSteps = 10
  currentStep = 7
  status = "FINISHED"
  agentRequest = "查询阿里巴巴股价"
  result = "阿里巴巴(BABA)今日股价$75.23，上涨+$1.45(+1.96%)"
  modelName = "qwen-max"
}

' 浏览器启动 - 思考行动记录
object "thinkAct1: ThinkActRecord" as thinkAct1 {
  id = 5001
  parentExecutionId = 4001
  thinkStartTime = "2025-01-15T14:00:00"
  thinkEndTime = "2025-01-15T14:00:05"
  actStartTime = "2025-01-15T14:00:05"
  actEndTime = "2025-01-15T14:00:10"
  thinkInput = "需要启动浏览器并导航到百度"
  thinkOutput = "使用BrowserUseTool启动Chrome浏览器"
  actionNeeded = true
  actionDescription = "启动浏览器并导航到百度首页"
  actionResult = "浏览器启动成功，百度首页加载完成"
  status = "SUCCESS"
  toolName = "browser-use-tool"
  toolParameters = "{\"action\":\"launch\",\"url\":\"https://www.baidu.com\"}"
}

' 搜索执行 - 思考行动记录
object "thinkAct2: ThinkActRecord" as thinkAct2 {
  id = 5002
  parentExecutionId = 4001
  thinkStartTime = "2025-01-15T14:00:10"
  thinkEndTime = "2025-01-15T14:00:15"
  actStartTime = "2025-01-15T14:00:15"
  actEndTime = "2025-01-15T14:00:25"
  thinkInput = "在百度搜索框输入阿里巴巴股价关键词"
  thinkOutput = "在搜索框输入'阿里巴巴股价 今日'并点击搜索"
  actionNeeded = true
  actionDescription = "执行搜索操作"
  actionResult = "搜索成功，获得相关结果页面"
  status = "SUCCESS"
  toolName = "browser-use-tool"
  toolParameters = "{\"action\":\"search\",\"query\":\"阿里巴巴股价 今日\"}"
}

' 数据提取 - 思考行动记录
object "thinkAct3: ThinkActRecord" as thinkAct3 {
  id = 5003
  parentExecutionId = 4001
  thinkStartTime = "2025-01-15T14:00:25"
  thinkEndTime = "2025-01-15T14:00:30"
  actStartTime = "2025-01-15T14:00:30"
  actEndTime = "2025-01-15T14:01:35"
  thinkInput = "需要从搜索结果页面提取股价信息"
  thinkOutput = "分析页面HTML内容，识别股价数据区域"
  actionNeeded = true
  actionDescription = "提取并解析股价信息"
  actionResult = "成功提取股价：$75.23，涨跌：+$1.45(+1.96%)"
  status = "SUCCESS"
  toolName = "llm-extract-tool"
  toolParameters = "{\"content_type\":\"stock_price\",\"target\":\"BABA\"}"
}

' 浏览器工具信息对象
object "browserTool: ActToolInfo" as browserTool {
  name = "browser-use-tool"
  parameters = "{\"action\":\"search\",\"query\":\"阿里巴巴股价 今日\"}"
  result = "页面内容已加载，找到股价相关信息"
  id = "browser-tool-001"
}

' 数据提取工具信息对象
object "extractTool: ActToolInfo" as extractTool {
  name = "llm-extract-tool"
  parameters = "{\"content_type\":\"stock_price\",\"target\":\"BABA\"}"
  result = "股价信息：$75.23 (+1.96%)"
  id = "extract-tool-001"
}

' 浏览器配置对象
object "browserConfig: BrowserInput" as browserConfig {
  url = "https://www.baidu.com"
  action = "navigate_and_search"
  waitForLoad = true
  headless = false
  timeout = 30000
  searchQuery = "阿里巴巴股价 今日"
}

' Chrome驱动服务对象
object "chromeService: ChromeDriverService" as chromeService {
  driverPath = "/usr/bin/chromedriver"
  windowSize = "1920,1080"
  userAgent = "JManus-Browser-Agent/1.0"
  sessionId = "chrome-session-001"
  status = "ACTIVE"
}

' 浏览器操作记录对象
object "browserRecord: BrowserOperationRecord" as browserRecord {
  id = 6001
  thinkActRecordId = 5002
  operationType = "SEARCH"
  targetUrl = "https://www.baidu.com/s?wd=阿里巴巴股价+今日"
  actionDetails = "在搜索框输入关键词并点击搜索按钮"
  executionTimeMs = 8000
  screenshotPath = "/tmp/search_result_screenshot.png"
  errorMessage = null
}

' 股价数据对象
object "stockData: StockPriceInfo" as stockData {
  symbol = "BABA"
  currentPrice = "75.23"
  change = "+1.45"
  changePercent = "+1.96%"
  updateTime = "2025-01-15 15:30"
  currency = "USD"
  source = "百度财经"
}

' 执行上下文对象
object "context: ExecutionContext" as context {
  userRequest = "查询阿里巴巴股价"
  currentAgent = "browser-agent"
  executionStatus = "COMPLETED"
  stepIndex = 6
  maxSteps = 10
  activeTools = ["browser-use-tool", "llm-extract-tool"]
}

' 计划协调器对象
object "coordinator: PlanningCoordinator" as coordinator {
  currentPlanId = "plan-20250115-stock-002"
  executionStatus = "COMPLETED"
  startTime = "2025-01-15T14:00:00"
  endTime = "2025-01-15T14:01:45"
  totalExecutionTime = "105秒"
}

' 执行结果对象
object "result: ExecutionResult" as result {
  planId = "plan-20250115-stock-002"
  status = "COMPLETED"
  result = "阿里巴巴(BABA)今日最新股价为$75.23，较昨日上涨$1.45(+1.96%)，数据更新时间：2025-01-15 15:30"
  executionTime = "105秒"
  stepCount = 7
  toolsUsed = ["browser-use-tool", "llm-extract-tool"]
}

' 关系定义
request ||..|| coordinator : "触发"
coordinator ||--|| complexPlan : "执行"
complexPlan ||--|| planRecord : "关联"

model ||--o{ browserAgent : "被使用"
browserAgent ||--|| agentRecord : "对应"
planRecord ||--o{ agentRecord : "包含"

agentRecord ||--o{ thinkAct1 : "包含"
agentRecord ||--o{ thinkAct2 : "包含"
agentRecord ||--o{ thinkAct3 : "包含"

thinkAct1 ||--o{ browserTool : "调用"
thinkAct2 ||--o{ browserTool : "调用"
thinkAct3 ||--o{ extractTool : "调用"

browserConfig ||--|| chromeService : "配置"
thinkAct2 ||--|| browserRecord : "产生"
thinkAct3 ||--|| stockData : "提取"

coordinator ||--|| context : "维护"
context ||--|| result : "产生"

browserTool ||..|| chromeService : "使用"
extractTool ||..|| stockData : "生成"

note top of request : 复合任务请求\n包含多步骤执行需求
note top of complexPlan : 复合执行计划\n7个步骤的详细分解
note right of thinkAct2 : 关键执行步骤\n浏览器搜索操作
note bottom of stockData : 最终提取数据\n结构化股价信息
note left of coordinator : 复合任务协调器\n管理多步骤执行流程

@enduml
```
```plantuml
@startuml
package "配置阶段" {
object model
object browserAgent
object browserConfig
object chromeService
}

package "请求处理阶段" {
object request
object coordinator
object complexPlan
}

package "计划执行阶段" {
object planRecord
object agentRecord
object thinkAct1
object thinkAct2
object thinkAct3
object browserTool
object extractTool
object browserRecord
}

package "数据处理阶段" {
object stockData
object context
}

package "结果返回阶段" {
object result
}

@enduml


```
## 复合计划执行对象生命周期分析

### 1. 配置阶段对象（静态配置）
这些对象在系统启动时创建，在整个执行过程中保持稳定：

#### DynamicModelEntity (AI模型配置)
- **生命周期**: 系统配置时创建，长期存在
- **作用**: 提供AI模型的连接和参数配置
- **状态**: 在复合任务执行过程中保持不变

#### DynamicAgentEntity (浏览器智能体配置)
- **生命周期**: 系统配置时创建，长期存在
- **作用**: 定义浏览器自动化智能体的能力和工具集
- **关键特性**: 包含BrowserUseTool等专用工具

#### BrowserInput (浏览器配置)
- **生命周期**: 任务开始时创建，任务结束时销毁
- **作用**: 配置浏览器的启动参数和行为策略
- **配置内容**: URL、等待策略、超时设置等

#### ChromeDriverService (Chrome驱动服务)
- **生命周期**: 首次使用时创建，可跨任务复用
- **作用**: 管理Chrome浏览器实例的生命周期
- **状态管理**: 维护浏览器会话和资源清理

### 2. 请求处理阶段对象（动态创建）

#### ExecutionRequest (复合任务请求)
- **创建时机**: 用户发起复合任务请求时
- **生命周期**: 从请求接收到处理完成
- **特征**: 包含多工具需求和复杂步骤预期

#### PlanningCoordinator (计划协调器)
- **创建时机**: 接收到复合任务请求时
- **生命周期**: 管理整个复合任务执行过程
- **职责**: 协调多步骤执行、工具调用、异常处理

#### ExecutionPlan (复合执行计划)
- **创建时机**: LLM分析完成复合任务后
- **特征**: 包含7个详细的执行步骤
- **复杂性**: 涉及浏览器操作、数据提取、验证等多个环节

### 3. 计划执行阶段对象（核心执行）

#### PlanExecutionRecord (计划执行记录)
- **创建时机**: 开始执行复合计划时
- **执行时间**: 约105秒（相比简单任务显著增加）
- **记录内容**: 完整的多步骤执行轨迹

#### AgentExecutionRecord (智能体执行记录)
- **特征**: 单个智能体处理复杂的多步骤任务
- **步骤数**: 7个主要执行步骤
- **工具使用**: 同时使用多种工具（浏览器、提取工具）

#### ThinkActRecord (思考-行动记录序列)
复合任务产生多个ThinkActRecord，每个对应一个执行步骤：

**ThinkActRecord 1 (浏览器启动)**
- **思考内容**: 分析浏览器启动需求
- **行动结果**: 成功启动Chrome并导航到百度

**ThinkActRecord 2 (搜索执行)**
- **思考内容**: 确定搜索关键词和策略
- **行动结果**: 成功执行搜索并获得结果页面

**ThinkActRecord 3 (数据提取)**
- **思考内容**: 分析页面结构，定位股价信息
- **行动结果**: 成功提取结构化的股价数据

### 4. 工具集成对象（专业化工具）

#### ActToolInfo (工具调用信息)
- **BrowserUseTool**: 处理浏览器自动化操作
- **LlmExtractTool**: 处理页面内容的智能解析

#### BrowserOperationRecord (浏览器操作记录)
- **创建时机**: 每次浏览器操作时
- **记录内容**: 操作类型、目标URL、执行时间、截图等
- **用途**: 用于调试、审计和性能分析

### 5. 数据对象（业务价值）

#### StockPriceInfo (股价信息)
- **创建时机**: 数据提取完成时
- **数据结构**: 包含股价、涨跌、时间等完整信息
- **验证**: 经过数据准确性和时效性验证

## 复合任务的对象交互特点

### 1. 工具链调用模式
```
AgentExecutionRecord -> ThinkActRecord -> BrowserUseTool -> ChromeDriverService
                    -> ThinkActRecord -> LlmExtractTool -> StockPriceInfo
```

### 2. 状态传递模式
```
ExecutionPlan -> PlanExecutionRecord -> AgentExecutionRecord -> ThinkActRecord[]
```
状态在对象链中逐层传递，支持复杂的执行状态跟踪。

### 3. 资源管理模式
```
ChromeDriverService <-> BrowserOperationRecord
                     <-> BrowserInput
```
浏览器资源需要专门的生命周期管理和异常处理。

## 复合任务与简单任务的对比

| 特征 | 简单任务 | 复合任务 |
|------|----------|----------|
| **执行时间** | 1-3秒 | 45-105秒 |
| **步骤数量** | 1个 | 7个 |
| **工具使用** | 1个（信息查询） | 2个（浏览器+提取） |
| **外部依赖** | 无 | 浏览器、网站 |
| **数据复杂度** | 简单文本 | 结构化数据 |
| **对象数量** | ~12个 | ~20个 |
| **记录粒度** | 粗粒度 | 细粒度 |
| **异常处理** | 简单重试 | 多层次容错 |

## 性能和资源特征

### 1. 内存使用
- **对象数量**: 约20个核心对象
- **数据量**: 包含页面内容、截图等大数据
- **峰值内存**: 浏览器实例占用主要内存

### 2. 外部资源依赖
- **浏览器进程**: Chrome实例及WebDriver
- **网络连接**: 多次HTTP请求
- **文件系统**: 截图和日志文件存储

### 3. 执行时间分布
- **浏览器启动**: 5秒 (5%)
- **页面导航**: 8秒 (8%)
- **搜索执行**: 12秒 (11%)
- **数据提取**: 65秒 (62%)
- **验证整理**: 15秒 (14%)

### 4. 可靠性考虑
- **网络异常**: 3次重试机制
- **页面解析失败**: 多套选择器备案
- **超时处理**: 30秒超时保护
- **资源清理**: 自动清理浏览器资源

---

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**参考**: JManus 复合计划执行流程分析 - 股价查询用例