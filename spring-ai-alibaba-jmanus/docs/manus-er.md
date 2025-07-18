# JManus 数据库 ER 图

本文档使用 PlantUML 生成 JManus 项目的数据库实体关系图（ER图），展示了系统中各个数据表的结构和关系。

## 数据库表概览

JManus 系统包含以下核心数据表：

1. **dynamic_agent** - 动态智能体表
2. **dynamic_agent_tools** - 智能体工具关联表
3. **dynamic_models** - 动态模型表
4. **plan_template** - 计划模板表
5. **plan_execution_record** - 计划执行记录表
6. **agent_execution_record** - 智能体执行记录表
7. **think_act_record** - 思考-行动记录表
8. **act_tool_info** - 行动工具信息表
9. **prompt** - 提示词表
10. **mcp_config** - MCP配置表
11. **system_config** - 系统配置表

## PlantUML ER 图

```plantuml
@startuml JManus数据库ER图

!define primary_key(x) <b><color:#b8861b><&key></color> x</b>
!define foreign_key(x) <color:#aaaaaa><&key></color> x
!define column(x) <color:#efefef><&media-record></color> x
!define table(x) entity x << (T, #ffaaaa) >>

skinparam linetype ortho
skinparam ranksep 50
skinparam nodesep 40

' 配置表
table(dynamic_models) {
  primary_key(id): BIGINT <<generated>>
  --
  column(type): VARCHAR(50) <<not null>>
  column(model_name): VARCHAR(255) <<not null>>
  column(provider): VARCHAR(100) <<not null>>
  column(configuration): TEXT
  column(is_active): BOOLEAN <<default: true>>
  column(created_at): TIMESTAMP <<default: now>>
  column(updated_at): TIMESTAMP <<default: now>>
}

table(dynamic_agent) {
  primary_key(id): BIGINT <<generated>>
  --
  column(agent_name): VARCHAR(255) <<not null, unique>>
  column(agent_description): VARCHAR(1000) <<not null>>
  column(system_prompt): TEXT <<deprecated>>
  column(next_step_prompt): TEXT <<not null>>
  column(class_name): VARCHAR(255) <<not null>>
  foreign_key(model_id): BIGINT
  column(namespace): VARCHAR(255)
  column(created_at): TIMESTAMP <<default: now>>
  column(updated_at): TIMESTAMP <<default: now>>
}

table(dynamic_agent_tools) {
  foreign_key(agent_id): BIGINT <<not null>>
  --
  column(tool_key): VARCHAR(255) <<not null>>
}

table(plan_template) {
  primary_key(id): BIGINT <<generated>>
  --
  column(template_name): VARCHAR(255) <<not null, unique>>
  column(template_description): TEXT
  column(template_content): TEXT <<not null>>
  column(template_type): VARCHAR(50) <<not null>>
  column(is_active): BOOLEAN <<default: true>>
  column(created_at): TIMESTAMP <<default: now>>
  column(updated_at): TIMESTAMP <<default: now>>
}

table(prompt) {
  primary_key(id): BIGINT <<generated>>
  --
  column(prompt_name): VARCHAR(255) <<not null>>
  column(namespace): VARCHAR(255) <<not null>>
  column(prompt_description): TEXT
  column(message_type): VARCHAR(50) <<not null>>
  column(type): VARCHAR(50) <<not null>>
  column(built_in): BOOLEAN <<default: false>>
  column(prompt_content): TEXT <<not null>>
  column(is_active): BOOLEAN <<default: true>>
  column(created_at): TIMESTAMP <<default: now>>
  column(updated_at): TIMESTAMP <<default: now>>
}

table(mcp_config) {
  primary_key(id): BIGINT <<generated>>
  --
  column(mcp_server_name): VARCHAR(255) <<not null, unique>>
  column(connection_type): VARCHAR(20) <<not null>>
  column(connection_config): VARCHAR(4000) <<not null>>
  column(created_at): TIMESTAMP <<default: now>>
  column(updated_at): TIMESTAMP <<default: now>>
}

table(system_config) {
  primary_key(id): BIGINT <<generated>>
  --
  column(config_group): VARCHAR(255) <<not null>>
  column(config_sub_group): VARCHAR(255) <<not null>>
  column(config_key): VARCHAR(255) <<not null>>
  column(config_path): VARCHAR(255) <<not null, unique>>
  column(config_value): TEXT
  column(default_value): TEXT
  column(description): TEXT
  column(input_type): VARCHAR(50) <<not null>>
  column(options_json): TEXT
  column(created_at): TIMESTAMP <<default: now>>
  column(updated_at): TIMESTAMP <<default: now>>
}

' 执行记录表
table(plan_execution_record) {
  primary_key(id): BIGINT <<generated>>
  --
  column(current_plan_id): VARCHAR(255) <<not null, unique>>
  column(root_plan_id): VARCHAR(255)
  foreign_key(think_act_record_id): BIGINT
  column(title): VARCHAR(500)
  column(user_request): TEXT
  column(start_time): TIMESTAMP <<default: now>>
  column(end_time): TIMESTAMP
  column(steps): TEXT <<JSON>>
  column(current_step_index): INTEGER <<default: 0>>
  column(completed): BOOLEAN <<default: false>>
  column(summary): TEXT
}

table(agent_execution_record) {
  primary_key(id): BIGINT <<generated>>
  --
  column(conversation_id): VARCHAR(255)
  column(agent_name): VARCHAR(255) <<not null>>
  column(agent_description): VARCHAR(1000)
  column(start_time): TIMESTAMP <<default: now>>
  column(end_time): TIMESTAMP
  column(max_steps): INTEGER <<default: 10>>
  column(current_step): INTEGER <<default: 0>>
  column(status): VARCHAR(20) <<not null>>
  column(agent_request): TEXT
  column(result): TEXT
  column(error_message): TEXT
  column(model_name): VARCHAR(255)
  column(is_completed): BOOLEAN <<default: false>>
  column(is_stuck): BOOLEAN <<default: false>>
}

table(think_act_record) {
  primary_key(id): BIGINT <<generated>>
  --
  foreign_key(plan_execution_record_id): BIGINT <<not null>>
  foreign_key(agent_execution_record_id): BIGINT <<not null>>
  column(step_name): VARCHAR(255)
  column(think_start_time): TIMESTAMP
  column(think_end_time): TIMESTAMP
  column(act_start_time): TIMESTAMP
  column(act_end_time): TIMESTAMP
  column(think_input): TEXT
  column(think_output): TEXT
  column(action_needed): BOOLEAN <<default: false>>
  column(action_description): TEXT
  column(action_result): TEXT
  column(status): VARCHAR(50) <<not null>>
  column(error_message): TEXT
  column(tool_name): VARCHAR(255)
  column(tool_parameters): TEXT
  column(step_index): INTEGER <<not null>>
  column(retry_count): INTEGER <<default: 0>>
}

table(act_tool_info) {
  primary_key(id): BIGINT <<generated>>
  --
  foreign_key(think_act_record_id): BIGINT <<not null>>
  column(name): VARCHAR(255) <<not null>>
  column(parameters): TEXT
  column(result): TEXT
  column(tool_id): VARCHAR(255)
  column(service_group): VARCHAR(100)
  column(execution_time_ms): INTEGER
  column(status): VARCHAR(20) <<not null>>
  column(created_at): TIMESTAMP <<default: now>>
}

' 关系定义
dynamic_agent ||--o{ dynamic_agent_tools : "agent_id"
dynamic_agent }o--|| dynamic_models : "model_id"

plan_execution_record ||--o{ think_act_record : "plan_execution_record_id"
agent_execution_record ||--o{ think_act_record : "agent_execution_record_id"
think_act_record ||--o{ act_tool_info : "think_act_record_id"

' 自引用关系
plan_execution_record }o--o| think_act_record : "think_act_record_id"
plan_execution_record }o--o{ agent_execution_record : "conversation_id"

note top of dynamic_agent : 动态智能体配置\n支持运行时创建和修改
note top of plan_execution_record : 计划执行的完整记录\n包含执行状态和步骤
note top of think_act_record : 智能体思考-行动过程\nReAct模式的核心记录
note top of mcp_config : MCP服务器配置\n支持多种连接类型

@enduml
```

## 表关系说明

### 主要关系

1. **智能体配置关系**
   - `dynamic_agent` → `dynamic_models` (多对一)：智能体使用的AI模型
   - `dynamic_agent` → `dynamic_agent_tools` (一对多)：智能体可用的工具列表

2. **执行记录关系**
   - `plan_execution_record` → `think_act_record` (一对多)：计划包含多个思考-行动步骤
   - `agent_execution_record` → `think_act_record` (一对多)：智能体执行包含多个思考-行动步骤
   - `think_act_record` → `act_tool_info` (一对多)：每个思考-行动可调用多个工具

3. **自引用关系**
   - `plan_execution_record` → `think_act_record` (多对一)：子计划由某个思考-行动步骤触发
   - `plan_execution_record` ↔ `agent_execution_record` (多对多)：通过conversation_id关联

### 约束说明

- **唯一性约束**：
  - `dynamic_agent.agent_name` - 智能体名称唯一
  - `plan_execution_record.current_plan_id` - 计划ID唯一
  - `prompt.namespace + prompt_name` - 命名空间内提示词名称唯一
  - `mcp_config.mcp_server_name` - MCP服务器名称唯一

- **状态约束**：
  - `agent_execution_record.status` ∈ ('IDLE', 'RUNNING', 'FINISHED', 'ERROR')
  - `think_act_record.status` ∈ ('PENDING', 'THINKING', 'ACTING', 'COMPLETED', 'FAILED')
  - `act_tool_info.status` ∈ ('SUCCESS', 'FAILED', 'TIMEOUT')
  - `mcp_config.connection_type` ∈ ('SSE', 'STUDIO', 'STREAMING')

## 数据库支持

JManus 支持多种数据库：

- **H2** - 开发和测试环境
- **MySQL** - 生产环境
- **PostgreSQL** - 生产环境

所有表结构通过 JPA/Hibernate 自动生成，支持跨数据库兼容。

## 索引建议

为提高查询性能，建议在以下字段上创建索引：

```sql
-- 智能体相关索引
CREATE INDEX idx_agent_namespace ON dynamic_agent(namespace);
CREATE INDEX idx_agent_model ON dynamic_agent(model_id);

-- 执行记录相关索引
CREATE INDEX idx_plan_root_id ON plan_execution_record(root_plan_id);
CREATE INDEX idx_agent_conversation ON agent_execution_record(conversation_id);
CREATE INDEX idx_think_act_plan ON think_act_record(plan_execution_record_id);
CREATE INDEX idx_think_act_agent ON think_act_record(agent_execution_record_id);

-- 提示词相关索引
CREATE INDEX idx_prompt_namespace ON prompt(namespace);
CREATE INDEX idx_prompt_type ON prompt(type);

-- 时间相关索引
CREATE INDEX idx_plan_start_time ON plan_execution_record(start_time);
CREATE INDEX idx_agent_start_time ON agent_execution_record(start_time);
```

---

**文档版本**: 1.0  
**生成时间**: 2025年1月  
**维护者**: Spring AI Alibaba Team