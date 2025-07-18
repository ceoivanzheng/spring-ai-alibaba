# JManus 核心对象图 (Object Diagram)

本文档展示 JManus 系统核心对象及其在特定时刻的关系。对象图提供了系统结构的快照，捕捉了存在的实例及其关联的静态视图。

## 系统核心对象概述

JManus 系统包含以下核心领域对象：

- **配置管理对象**: DynamicAgentEntity、DynamicModelEntity、McpConfigEntity
- **执行记录对象**: PlanExecutionRecord、AgentExecutionRecord、ThinkActRecord
- **工具信息对象**: ActToolInfo、UserInputWaitState

## PlantUML 对象图

```plantuml
@startuml JManus_Object_Diagram
!theme plain

title JManus 核心对象关系图

' 配置管理对象
object "model1: DynamicModelEntity" as model1 {
  id = 1
  modelName = "qwen-max"
  baseUrl = "https://dashscope.aliyuncs.com/api/v1"
  apiKey = "sk-****"
  type = "CHAT"
  modelDescription = "阿里云通义千问大模型"
}

object "agent1: DynamicAgentEntity" as agent1 {
  id = 1
  agentName = "ReActAgent"
  agentDescription = "反应式智能体"
  nextStepPrompt = "根据当前情况进行思考和行动..."
  className = "com.alibaba.cloud.ai.example.manus.agent.ReActAgent"
  namespace = "default"
  availableToolKeys = ["browser-tool", "python-tool"]
}

object "agent2: DynamicAgentEntity" as agent2 {
  id = 2
  agentName = "PlanAgent"
  agentDescription = "计划式智能体"
  nextStepPrompt = "制定详细的执行计划..."
  className = "com.alibaba.cloud.ai.example.manus.agent.PlanAgent"
  namespace = "default"
  availableToolKeys = ["file-tool", "bash-tool"]
}

object "mcpConfig1: McpConfigEntity" as mcpConfig1 {
  id = 1
  mcpServerName = "github-mcp"
  connectionType = "STUDIO"
  connectionConfig = "{\"command\": \"npx\", \"args\": [\"@modelcontextprotocol/server-github\"]}"
}

' 执行记录对象
object "planRecord1: PlanExecutionRecord" as planRecord1 {
  id = 1001
  currentPlanId = "plan-2025-001"
  rootPlanId = "plan-2025-001"
  title = "数据分析任务"
  userRequest = "分析销售数据并生成报告"
  startTime = "2025-01-18T10:00:00"
  endTime = "2025-01-18T10:30:00"
  completed = true
  summary = "成功完成数据分析任务"
  modelName = "qwen-max"
  currentStepIndex = 2
  steps = ["数据收集", "数据处理", "报告生成"]
}

object "agentExecRecord1: AgentExecutionRecord" as agentExecRecord1 {
  id = 2001
  conversationId = "conv-001"
  agentName = "ReActAgent"
  agentDescription = "反应式智能体"
  startTime = "2025-01-18T10:00:00"
  endTime = "2025-01-18T10:15:00"
  maxSteps = 5
  currentStep = 3
  status = "FINISHED"
  agentRequest = "分析销售数据"
  result = "数据分析完成"
  modelName = "qwen-max"
}

object "agentExecRecord2: AgentExecutionRecord" as agentExecRecord2 {
  id = 2002
  conversationId = "conv-001"
  agentName = "PlanAgent"
  agentDescription = "计划式智能体"
  startTime = "2025-01-18T10:15:00"
  endTime = "2025-01-18T10:30:00"
  maxSteps = 3
  currentStep = 3
  status = "FINISHED"
  agentRequest = "生成分析报告"
  result = "报告生成完成"
  modelName = "qwen-max"
}

object "thinkActRecord1: ThinkActRecord" as thinkActRecord1 {
  id = 3001
  parentExecutionId = 2001
  thinkStartTime = "2025-01-18T10:00:00"
  thinkEndTime = "2025-01-18T10:01:00"
  actStartTime = "2025-01-18T10:01:00"
  actEndTime = "2025-01-18T10:05:00"
  thinkInput = "需要分析销售数据"
  thinkOutput = "使用Python工具加载和分析数据"
  actionNeeded = true
  actionDescription = "执行Python数据分析脚本"
  actionResult = "成功加载数据，发现销售趋势"
  status = "SUCCESS"
  toolName = "python-tool"
  toolParameters = "{\"script\": \"sales_analysis.py\"}"
}

object "thinkActRecord2: ThinkActRecord" as thinkActRecord2 {
  id = 3002
  parentExecutionId = 2001
  thinkStartTime = "2025-01-18T10:05:00"
  thinkEndTime = "2025-01-18T10:06:00"
  actStartTime = "2025-01-18T10:06:00"
  actEndTime = "2025-01-18T10:10:00"
  thinkInput = "数据已分析，需要可视化"
  thinkOutput = "使用浏览器工具创建图表"
  actionNeeded = true
  actionDescription = "打开浏览器创建数据图表"
  actionResult = "成功创建销售趋势图表"
  status = "SUCCESS"
  toolName = "browser-tool"
  toolParameters = "{\"url\": \"chart.html\"}"
}

object "thinkActRecord3: ThinkActRecord" as thinkActRecord3 {
  id = 3003
  parentExecutionId = 2002
  thinkStartTime = "2025-01-18T10:15:00"
  thinkEndTime = "2025-01-18T10:16:00"
  actStartTime = "2025-01-18T10:16:00"
  actEndTime = "2025-01-18T10:25:00"
  thinkInput = "需要生成分析报告"
  thinkOutput = "使用文件工具创建报告文档"
  actionNeeded = true
  actionDescription = "创建销售分析报告"
  actionResult = "成功生成报告文档"
  status = "SUCCESS"
  toolName = "file-tool"
  toolParameters = "{\"filename\": \"sales_report.pdf\"}"
}

object "actToolInfo1: ActToolInfo" as actToolInfo1 {
  name = "python-tool"
  parameters = "{\"script\": \"sales_analysis.py\"}"
  result = "数据分析结果: 销售增长15%"
  id = "tool-001"
}

object "actToolInfo2: ActToolInfo" as actToolInfo2 {
  name = "browser-tool"
  parameters = "{\"url\": \"chart.html\"}"
  result = "图表创建成功"
  id = "tool-002"
}

object "userInputState1: UserInputWaitState" as userInputState1 {
  waitingForInput = false
  inputPrompt = "请确认是否继续执行"
  inputReceived = true
  userResponse = "继续"
}

' 关系定义
model1 ||--o{ agent1 : "被使用"
model1 ||--o{ agent2 : "被使用"

planRecord1 ||--o{ agentExecRecord1 : "包含"
planRecord1 ||--o{ agentExecRecord2 : "包含"
planRecord1 ||--o| userInputState1 : "关联"

agentExecRecord1 ||--o{ thinkActRecord1 : "包含"
agentExecRecord1 ||--o{ thinkActRecord2 : "包含"
agentExecRecord2 ||--o{ thinkActRecord3 : "包含"

thinkActRecord1 ||--o{ actToolInfo1 : "调用"
thinkActRecord2 ||--o{ actToolInfo2 : "调用"

' 可能的子计划关系（递归）
thinkActRecord1 ||--o| planRecord1 : "可能触发子计划"

' 配置关系
agent1 ..> mcpConfig1 : "可能使用MCP工具"
agent2 ..> mcpConfig1 : "可能使用MCP工具"

note top of model1 : 动态模型配置\n包含AI模型的基本信息
note top of planRecord1 : 计划执行记录\n跟踪整个计划的执行过程
note right of thinkActRecord1 : 思考-行动记录\n记录智能体的思考和行动过程
note bottom of mcpConfig1 : MCP配置\n外部服务集成配置

@enduml
```

## 核心对象说明

### 1. 配置管理对象

#### 1.1 DynamicModelEntity (动态模型实体)
- **作用**: 存储AI模型配置信息
- **关键属性**: modelName, baseUrl, apiKey, type, modelDescription
- **关系**: 与 DynamicAgentEntity 一对多关系

#### 1.2 DynamicAgentEntity (动态智能体实体)
- **作用**: 存储智能体配置信息
- **关键属性**: agentName, agentDescription, nextStepPrompt, className, availableToolKeys
- **关系**: 与 DynamicModelEntity 多对一关系

#### 1.3 McpConfigEntity (MCP配置实体)
- **作用**: 存储外部MCP服务器配置
- **关键属性**: mcpServerName, connectionType, connectionConfig
- **关系**: 被智能体间接使用

### 2. 执行记录对象

#### 2.1 PlanExecutionRecord (计划执行记录)
- **作用**: 跟踪计划执行的完整生命周期
- **关键属性**: currentPlanId, rootPlanId, title, userRequest, steps, agentExecutionSequence
- **关系**: 
  - 与 AgentExecutionRecord 一对多关系
  - 与 UserInputWaitState 一对一关系
  - 支持递归关系（子计划）

#### 2.2 AgentExecutionRecord (智能体执行记录)
- **作用**: 记录单个智能体的执行过程
- **关键属性**: agentName, startTime, endTime, maxSteps, currentStep, status, thinkActSteps
- **关系**: 与 ThinkActRecord 一对多关系

#### 2.3 ThinkActRecord (思考-行动记录)
- **作用**: 记录智能体单次思考-行动循环
- **关键属性**: 
  - 思考阶段: thinkInput, thinkOutput, thinkStartTime, thinkEndTime
  - 行动阶段: actionDescription, actionResult, actStartTime, actEndTime
  - 工具信息: toolName, toolParameters, actToolInfoList
- **关系**: 
  - 与 AgentExecutionRecord 多对一关系
  - 与 ActToolInfo 一对多关系
  - 与 PlanExecutionRecord 一对一关系（子计划）

### 3. 辅助对象

#### 3.1 ActToolInfo (行动工具信息)
- **作用**: 记录工具调用的详细信息
- **关键属性**: name, parameters, result, id
- **关系**: 被 ThinkActRecord 包含

#### 3.2 UserInputWaitState (用户输入等待状态)
- **作用**: 管理需要用户输入的等待状态
- **关键属性**: waitingForInput, inputPrompt, inputReceived, userResponse
- **关系**: 被 PlanExecutionRecord 关联

## 对象关系特点

### 1. 层次化结构
```
PlanExecutionRecord (计划)
  └── AgentExecutionRecord[] (智能体执行序列)
      └── ThinkActRecord[] (思考-行动步骤)
          └── ActToolInfo[] (工具调用信息)
```

### 2. 递归关系
- ThinkActRecord 可以通过 subPlanExecutionRecord 触发新的 PlanExecutionRecord
- 支持无限嵌套的子计划执行

### 3. 配置与执行分离
- 配置对象 (DynamicAgentEntity, DynamicModelEntity, McpConfigEntity) 相对静态
- 执行对象 (PlanExecutionRecord, AgentExecutionRecord, ThinkActRecord) 动态创建

### 4. 时间序列特性
- 所有执行记录都包含时间戳信息
- 支持执行过程的完整时间线重建

## 使用场景

### 1. 计划执行跟踪
通过 PlanExecutionRecord 跟踪整个计划的执行状态和进度。

### 2. 智能体行为分析
通过 AgentExecutionRecord 和 ThinkActRecord 分析智能体的思考和决策过程。

### 3. 工具使用统计
通过 ActToolInfo 统计和分析工具的使用情况和效果。

### 4. 错误诊断和调试
通过完整的执行记录链进行问题定位和性能优化。

---

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**参考**: JManus 核心实体类源码