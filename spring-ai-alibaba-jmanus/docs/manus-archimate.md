# JManus Architecture - ArchiMate Diagram

本文档使用 PlantUML ArchiMate 图表展示 JManus 系统的整体架构设计。

## 系统架构概述

JManus 是一个基于 Spring Boot 的多智能体协作系统，采用分层架构设计，支持 Plan-Act 模式执行和 MCP 协议集成。

## ArchiMate 架构图

```plantuml
@startuml JManus_Architecture
skinparam rectangle<<behavior>> {
	roundCorner 25
}

' 定义 Archimate sprites
sprite $bProcess jar:archimate/business-process
sprite $bService jar:archimate/business-service
sprite $aService jar:archimate/application-service
sprite $aComponent jar:archimate/application-component
sprite $tService jar:archimate/technology-service
sprite $tInterface jar:archimate/technology-interface
sprite $node jar:archimate/node
sprite $businessObject jar:archimate/business-object

title JManus 系统架构图 (ArchiMate)

' ===== 应用层 =====
package "应用层 (Application Layer)" {
  rectangle "Vue3 Web UI" as web_ui <<$aComponent>><<behavior>> #Application
  rectangle "Agent API" as agent_api <<$aComponent>><<behavior>> #Application
  rectangle "MCP API" as mcp_api <<$aComponent>><<behavior>> #Application
  rectangle "Plan API" as plan_api <<$aComponent>><<behavior>> #Application
}

' ===== 业务层 =====  
package "业务层 (Business Layer)" {
  rectangle "Agent Management" as agent_mgmt <<$bService>><<behavior>> #Business
  rectangle "Plan Execution" as plan_execution <<$bService>><<behavior>> #Business
  rectangle "MCP Integration" as mcp_integration <<$bService>><<behavior>> #Business
  rectangle "Tool Management" as tool_mgmt <<$bService>><<behavior>> #Business
  
  rectangle "Think-Act Process" as think_act <<$bProcess>><<behavior>> #Business
  rectangle "Plan-Act Process" as plan_act <<$bProcess>><<behavior>> #Business
}

' ===== 应用组件层 =====
package "应用组件层 (Application Components)" {
  rectangle "Dynamic Agent" as dynamic_agent <<$aComponent>><<behavior>> #Application
  rectangle "ReAct Agent" as react_agent <<$aComponent>><<behavior>> #Application
  rectangle "Plan Agent" as plan_agent <<$aComponent>><<behavior>> #Application
  rectangle "Planning Coordinator" as coordinator <<$aComponent>><<behavior>> #Application
  
  rectangle "LLM Service" as llm_service <<$aService>><<behavior>> #Application
  rectangle "Config Service" as config_service <<$aService>><<behavior>> #Application
}

' ===== 技术层 =====
package "技术层 (Technology Layer)" {
  rectangle "Spring Boot" as spring_boot <<$tService>><<behavior>> #Technology
  rectangle "Spring AI" as spring_ai <<$tService>><<behavior>> #Technology
  rectangle "JPA/Hibernate" as jpa <<$tService>><<behavior>> #Technology
  
  rectangle "Database Interface" as db_interface <<$tInterface>><<behavior>> #Technology
  rectangle "AI Model Interface" as ai_interface <<$tInterface>><<behavior>> #Technology
}

' ===== 基础设施层 =====
package "基础设施层 (Infrastructure)" {
  rectangle "H2 Database" as h2_db <<$node>><<behavior>> #Technology
  rectangle "MySQL Database" as mysql_db <<$node>><<behavior>> #Technology
  rectangle "DashScope API" as dashscope <<$node>><<behavior>> #Technology
  rectangle "MCP Servers" as mcp_servers <<$node>><<behavior>> #Technology
  rectangle "Chrome Driver" as chrome <<$node>><<behavior>> #Technology
  rectangle "Python Runtime" as python_env <<$node>><<behavior>> #Technology
}

' ===== 工具生态 =====
package "工具生态 (Tool Ecosystem)" {
  rectangle "Browser Tool" as browser_tool <<$aComponent>><<behavior>> #Application
  rectangle "Python Tool" as python_tool <<$aComponent>><<behavior>> #Application
  rectangle "Bash Tool" as bash_tool <<$aComponent>><<behavior>> #Application
  rectangle "File Tool" as file_tool <<$aComponent>><<behavior>> #Application
  rectangle "MCP Tool" as mcp_tool <<$aComponent>><<behavior>> #Application
}

' ===== 数据对象 =====
package "数据层 (Data Layer)" {
  rectangle "Agent Config" as agent_config <<$businessObject>><<behavior>> #Business
  rectangle "Plan Data" as plan_data <<$businessObject>><<behavior>> #Business
  rectangle "Execution Record" as exec_record <<$businessObject>><<behavior>> #Business
  rectangle "MCP Config" as mcp_config <<$businessObject>><<behavior>> #Business
}

' ===== 关系定义 =====
web_ui -down-> agent_api : "调用"
web_ui -down-> mcp_api : "调用"
web_ui -down-> plan_api : "调用"

agent_api -down-> agent_mgmt : "使用"
mcp_api -down-> mcp_integration : "使用"  
plan_api -down-> plan_execution : "使用"

agent_mgmt -down-> dynamic_agent : "管理"
plan_execution -down-> coordinator : "协调"
plan_execution -right-> think_act : "执行"
plan_execution -right-> plan_act : "执行"

dynamic_agent .down.|> react_agent : "包含"
dynamic_agent .down.|> plan_agent : "包含"
coordinator -down-> llm_service : "使用"

llm_service -down-> spring_ai : "基于"
config_service -down-> jpa : "使用"
spring_ai -down-> ai_interface : "通过"
jpa -down-> db_interface : "通过"

db_interface .down.|> h2_db : "连接"
db_interface .down.|> mysql_db : "连接"
ai_interface -down-> dashscope : "调用"

tool_mgmt .down.|> browser_tool : "管理"
tool_mgmt .down.|> python_tool : "管理"
tool_mgmt .down.|> bash_tool : "管理"
tool_mgmt .down.|> file_tool : "管理"
tool_mgmt .down.|> mcp_tool : "管理"

browser_tool -down-> chrome : "使用"
python_tool -down-> python_env : "使用"
mcp_tool -down-> mcp_servers : "连接"

agent_mgmt -right-> agent_config : "管理"
plan_execution -right-> plan_data : "处理"
plan_execution -right-> exec_record : "记录"
mcp_integration -right-> mcp_config : "配置"

legend left
JManus 多智能体协作系统架构
====
<$bProcess> : 业务流程
====
<$bService> : 业务服务
====
<$aService> : 应用服务
====
<$aComponent> : 应用组件
====
<$tService> : 技术服务
====
<$tInterface> : 技术接口
====
<$node> : 基础设施节点
====
<$businessObject> : 业务对象
endlegend

@enduml
```



## 架构层次说明

### 1. 应用层 (Application Layer)
- **Vue3 Web UI**: 提供用户友好的图形界面，支持智能体配置、MCP管理和执行监控
- **REST API**: 提供标准化的HTTP接口，包括智能体管理、MCP服务管理和计划执行

### 2. 业务层 (Business Layer)  
- **Agent Management**: 智能体的创建、配置、更新和删除管理
- **Plan Execution**: 支持Plan-Act和Think-Act两种执行模式
- **MCP Integration**: 集成外部服务，支持SSE、STUDIO、STREAMING连接类型
- **Tool Management**: 工具注册、调用和生命周期管理

### 3. 应用组件层 (Application Components)
- **Dynamic Agent**: 可配置的智能体，支持运行时修改
- **ReAct Agent**: 基于思考-行动循环的反应式智能体  
- **Plan Agent**: 先制定计划再执行的计划式智能体
- **Planning Coordinator**: 协调整个执行流程

### 4. 技术层 (Technology Layer)
- **Spring Boot**: 应用框架和依赖注入容器
- **JPA/Hibernate**: 对象关系映射和数据持久化
- **Spring AI**: AI模型集成和抽象层

### 5. 基础设施层 (Infrastructure)
- **数据库**: 支持H2、MySQL、PostgreSQL多种数据库
- **AI服务**: 集成DashScope、OpenAI等大语言模型服务
- **外部工具**: Chrome浏览器、Python运行时、MCP服务器等

### 6. 工具生态系统 (Tool Ecosystem)
- **Browser Tool**: 网页浏览和操作
- **Python Tool**: Python代码执行
- **Bash Tool**: Shell命令执行
- **File Tool**: 文件读写操作
- **MCP Tool**: 外部服务集成

## 关键特性

### 🎯 Plan-Act 模式
系统支持两种执行模式：
- **Think-Act**: 思考-行动循环，适合探索性任务
- **Plan-Act**: 先制定详细计划再执行，提供更高的确定性

### 🔗 MCP 协议支持
原生支持Model Context Protocol (MCP)：
- **SSE连接**: Server-Sent Events流式连接
- **STUDIO连接**: 本地STDIO连接  
- **STREAMING连接**: 流式数据传输

### 🛠️ 动态配置
- 智能体可通过Web界面动态配置
- 工具可热插拔，支持运行时添加/移除
- 支持多命名空间隔离

### 📊 监控和日志
- 完整的执行记录和审计日志
- 实时性能监控和告警
- 分层的错误处理和恢复机制

## 部署选项

### 数据库支持
- **H2**: 嵌入式数据库，适合开发和测试
- **MySQL**: 生产环境推荐
- **PostgreSQL**: 企业级应用支持

### AI模型集成
- **DashScope**: 阿里云大模型服务
- **OpenAI**: GPT系列模型
- **可扩展**: 支持其他兼容的AI服务

---

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**参考**: [PlantUML ArchiMate Documentation](https://plantuml.com/en/archimate-diagram)