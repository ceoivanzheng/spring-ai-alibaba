# JManus 简单对话处理流程 - PlantUML时序图

## 概述

本文档使用PlantUML格式描述JManus系统处理简单对话"你是哪个模型？"的完整流程时序图。

## PlantUML时序图

```plantuml
@startuml JManus简单对话处理流程
!theme plain
skinparam backgroundColor #FFFFFF
skinparam sequenceArrowThickness 2
skinparam roundcorner 20
skinparam maxmessagesize 60

participant "用户" as User
participant "Vue3前端" as Frontend
participant "REST API层" as API
participant "PlanningCoordinator" as Coordinator
participant "PlanCreator" as Creator
participant "PlanExecutorFactory" as Factory
participant "PlanExecutor" as Executor
participant "DynamicAgent" as Agent
participant "LLM服务" as LLM
participant "数据库" as DB
participant "执行记录器" as Recorder

== 用户输入处理阶段 ==
User -> Frontend: 输入"你是哪个模型？"
Frontend -> API: POST /api/executor/execute
note right of API
ExecutionRequest {
  userRequest: "你是哪个模型？",
  agentName: "default-agent",
  planType: "SIMPLE",
  maxSteps: 10
}
end note

== 计划创建阶段 ==
API -> Coordinator: 接收执行请求
Coordinator -> Creator: 创建执行计划
Creator -> LLM: 分析用户需求
note right of LLM
理解"你是哪个模型？"
确定为信息查询类任务
end note
LLM --> Creator: 返回计划步骤
note right of Creator
生成ExecutionPlan {
  steps: ["查询当前模型信息"],
  planType: SIMPLE
}
end note
Creator --> Coordinator: 返回ExecutionPlan

Coordinator -> Recorder: 记录计划创建
Recorder -> DB: 保存PlanExecutionRecord

== 执行器创建阶段 ==
Coordinator -> Factory: 创建执行器
Factory -> DB: 查询智能体配置
DB --> Factory: 返回DynamicAgentEntity
Factory -> Agent: 创建DynamicAgent实例
Factory --> Coordinator: 返回PlanExecutor

== 计划执行阶段 ==
Coordinator -> Executor: 开始执行计划

loop 执行步骤循环
    Executor -> Agent: 执行当前步骤
    Agent -> Agent: Think阶段：分析任务
    note right of Agent
    思考："用户询问我的模型信息，
    我需要获取当前配置的模型"
    end note
    
    Agent -> Recorder: 记录思考过程
    Recorder -> DB: 保存ThinkActRecord(THINKING)
    
    Agent -> Agent: Act阶段：执行行动
    note right of Agent
    行动：查询自身模型配置
    end note
    Agent -> DB: 查询关联的DynamicModelEntity
    DB --> Agent: 返回模型信息
    note right of DB
    ModelInfo {
      modelName: "qwen-max",
      provider: "dashscope",
      description: "阿里云通义千问大模型"
    }
    end note
    
    Agent -> Recorder: 记录行动结果
    Recorder -> DB: 更新ThinkActRecord(COMPLETED)
    
    Agent --> Executor: 返回步骤结果
    note right of Agent
    StepResult {
      content: "我是阿里云通义千问大模型(qwen-max)",
      completed: true
    }
    end note
    
    Executor -> Recorder: 记录执行状态
    Recorder -> DB: 更新AgentExecutionRecord
end

== 结果返回阶段 ==
Executor --> Coordinator: 执行完成
Coordinator -> Recorder: 最终化处理
Recorder -> DB: 更新PlanExecutionRecord状态
note right of Recorder
ExecutionResult {
  planId: "plan-20250115-001",
  status: "COMPLETED",
  result: "我是阿里云通义千问大模型(qwen-max)，专门用于自然语言理解和生成任务。"
}
end note

Coordinator --> API: 返回执行结果
API --> Frontend: HTTP 200 + ExecutionResult
Frontend --> User: 显示回答

== 后续查询阶段（可选） ==
Frontend -> API: GET /api/executor/details/{planId}
API -> Recorder: 查询执行详情
Recorder -> DB: 获取完整执行记录
DB --> Recorder: 返回详细记录
Recorder --> API: 返回执行详情
API --> Frontend: 返回详细信息

@enduml
```

## 时序图说明

### 主要阶段
1. **用户输入处理阶段**：用户通过前端界面输入问题，前端构造API请求
2. **计划创建阶段**：系统分析用户需求，创建执行计划
3. **执行器创建阶段**：加载智能体配置，创建执行实例
4. **计划执行阶段**：智能体执行Think-Act循环，处理用户请求
5. **结果返回阶段**：返回处理结果给用户
6. **后续查询阶段**：可选的执行详情查询

### 关键交互
- **Think-Act循环**：智能体的核心处理模式，先思考再行动
- **数据库记录**：完整记录每个执行步骤，便于审计和分析
- **模型信息查询**：通过查询DynamicModelEntity获取当前AI模型配置

### 预期输出
用户最终收到的回答：
> "我是阿里云通义千问大模型(qwen-max)，专门用于自然语言理解和生成任务。我可以帮助您处理各种文本相关的工作，包括问答、分析、创作等。"

## 使用说明

1. 将PlantUML代码复制到支持PlantUML的编辑器中（如VSCode with PlantUML插件）
2. 或者使用在线PlantUML编辑器：http://www.plantuml.com/plantuml/
3. 生成的时序图可以导出为PNG、SVG等格式用于文档展示

## 相关文档

- [JManus简单对话处理流程分析](manus-simple-dialog.md)
- [JManus架构设计](manus-architecture.md)
- [JManus组件设计](manus-component.md)