# Spring AI Alibaba JManus 系统设计文档

## 1. 项目概述

### 1.1 项目简介

Spring AI Alibaba JManus 是一个基于 Java 的多智能体协作系统，专门设计用于处理需要高度确定性的探索性任务。该系统采用 Plan-Act 模式，提供完整的 HTTP 调用接口，适合 Java 开发者进行二次集成。

### 1.2 核心特性

- **🤖 纯Java多智能体协作实现**：完整的多智能体协作系统，提供HTTP调用接口
- **🛠️ Plan-Act模式**：精确控制每个执行细节，提供极高的执行确定性
- **🔗 MCP集成**：原生支持模型上下文协议(MCP)，无缝集成外部服务和工具
- **📜 Web界面配置**：通过直观的Web管理界面轻松配置智能体
- **🌊 无限上下文处理**：支持从海量内容中精确提取目标信息

### 1.3 技术栈

- **后端**: Java 17+, Spring Boot 3.x, Spring AI
- **前端**: Vue 3, TypeScript, Vite
- **数据库**: H2/MySQL/PostgreSQL
- **AI模型**: 阿里云DashScope API（可配置其他AI模型提供商）
- **容器**: Docker支持

## 2. 系统架构

### 2.1 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        Web 前端 (Vue3)                          │
├─────────────────────────────────────────────────────────────────┤
│                        REST API 层                             │
├─────────────────────────────────────────────────────────────────┤
│  规划协调层  │  智能体管理层  │  工具系统层  │  MCP集成层      │
├─────────────────────────────────────────────────────────────────┤
│                        核心业务层                              │
│  Plan-Act执行引擎  │  动态智能体  │  执行记录器  │  状态管理    │
├─────────────────────────────────────────────────────────────────┤
│                        数据访问层                              │
│        JPA/Hibernate        │       数据库 (H2/MySQL/PG)      │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 核心模块

#### 2.2.1 Plan-Act 执行架构

Plan-Act模式是系统的核心执行机制，包含以下组件：

- **PlanType**: 定义计划类型（SIMPLE、MAPREDUCE）
- **PlanCreator**: 基于用户需求和LLM创建执行计划
- **PlanExecutor**: 执行计划的具体实现
- **PlanningCoordinator**: 协调整个计划生命周期

#### 2.2.2 智能体系统

- **BaseAgent**: 智能体抽象基类，定义基本行为
- **DynamicAgent**: 动态可配置的智能体实现
- **AgentService**: 智能体管理服务
- **DynamicAgentLoader**: 智能体加载器

#### 2.2.3 工具系统

- **ToolCallBiFunctionDef**: 工具调用接口定义
- **内置工具**: 浏览器工具、Bash工具、Python执行器、文件操作等
- **MCP工具**: 通过MCP协议集成的外部工具
- **工具回调管理**: 统一的工具调用和状态管理

## 3. 核心组件详细设计

### 3.1 Plan-Act 执行引擎

#### 3.1.1 计划类型 (PlanType)

```java
public enum PlanType {
    SIMPLE("简单计划", "适用于基本的任务执行，步骤按顺序进行"),
    MAPREDUCE("MapReduce计划", "适用于复杂的分布式任务，支持并行处理和结果聚合");
}
```

**设计说明**:
- SIMPLE模式：顺序执行，适用于线性任务流程
- MAPREDUCE模式：支持并行处理和结果聚合，适用于大数据处理场景

#### 3.1.2 计划执行器 (PlanExecutor)

**接口定义**:
```java
public interface PlanExecutorInterface {
    void executeAllSteps(ExecutionContext context);
}
```

**实现类**:
- `PlanExecutor`: 简单计划执行器
- `MapReducePlanExecutor`: MapReduce计划执行器

**核心功能**:
- 执行上下文管理
- 步骤状态跟踪
- 错误处理和恢复
- 执行记录和日志

#### 3.1.3 规划协调器 (PlanningCoordinator)

**职责**:
- 协调计划创建、执行、最终化的完整生命周期
- 管理执行上下文和状态
- 处理异常和错误恢复

### 3.2 智能体系统

#### 3.2.1 智能体基类 (BaseAgent)

**核心属性**:
```java
public abstract class BaseAgent {
    private String currentPlanId;
    private String rootPlanId;
    private AgentState state;
    protected ILlmService llmService;
    protected PlanExecutionRecorder planExecutionRecorder;
    private Map<String, Object> envData;
}
```

**关键方法**:
- `getName()`: 获取智能体名称
- `clearUp(String planId)`: 清理资源

#### 3.2.2 动态智能体 (DynamicAgent)

**特性**:
- 继承自ReActAgent，支持思考-行动循环
- 动态配置名称、描述、提示词
- 可配置的工具集合
- 支持用户输入交互

**数据库实体 (DynamicAgentEntity)**:
```java
@Entity
@Table(name = "dynamic_agent")
public class DynamicAgentEntity {
    private String agentName;
    private String agentDescription;
    private String nextStepPrompt;
    private List<String> availableToolKeys;
    private DynamicModelEntity model;
    private String namespace;
}
```

#### 3.2.3 智能体服务 (AgentService)

**API接口**:
- `getAllAgents()`: 获取所有智能体
- `createAgent(AgentConfig)`: 创建智能体
- `updateAgent(AgentConfig)`: 更新智能体
- `deleteAgent(String id)`: 删除智能体
- `getAvailableTools()`: 获取可用工具列表

### 3.3 工具系统

#### 3.3.1 工具接口定义

```java
public interface ToolCallBiFunctionDef<T> {
    String getName();
    String getDescription();
    String getParameters();
    Class<T> getInputType();
    ToolExecuteResult run(T input);
    String getCurrentToolStateString();
    void cleanup(String planId);
}
```

#### 3.3.2 内置工具

| 工具名称 | 功能描述 | 实现类 |
|---------|---------|--------|
| BrowserUseTool | 浏览器自动化工具 | `BrowserUseTool` |
| Bash | 命令行执行工具 | `Bash` |
| PythonExecute | Python代码执行 | `PythonExecute` |
| TextFileOperator | 文件操作工具 | `TextFileOperator` |
| GoogleSearch | 网络搜索工具 | `GoogleSearch` |
| MapReduceTool | MapReduce处理工具 | `MapReduceTool` |
| TerminateTool | 任务终止工具 | `TerminateTool` |

#### 3.3.3 工具回调管理

**PlanningFactory**负责创建和管理工具回调映射：
```java
public Map<String, ToolCallBackContext> toolCallbackMap(String planId, String rootPlanId, List<String> terminateColumns)
```

**工具注册流程**:
1. 创建工具定义列表
2. 为每个工具创建FunctionToolCallback
3. 设置工具元数据和参数
4. 建立工具名称到回调的映射关系

### 3.4 MCP集成系统

#### 3.4.1 MCP服务接口

```java
public interface IMcpService {
    void addMcpServer(McpConfigRequestVO mcpConfig);
    void removeMcpServer(long id);
    List<McpConfigEntity> getMcpServers();
    List<McpServiceEntity> getFunctionCallbacks(String planId);
    void close(String planId);
}
```

#### 3.4.2 MCP工具封装

**McpTool实现**:
- 封装MCP协议的工具调用
- JSON参数序列化和反序列化
- 状态管理和清理
- 错误处理和重试机制

#### 3.4.3 连接类型支持

- **SSE**: Server-Sent Events连接
- **STUDIO**: 本地STDIO连接
- **STREAMING**: 流式连接

#### 3.4.4 重试机制

- 最大3次重试
- 递增等待时间（1s, 2s, 3s）
- 超时配置（60秒）
- 连接状态监控

## 4. 数据模型设计

### 4.1 智能体相关

#### 4.1.1 DynamicAgentEntity
```sql
CREATE TABLE dynamic_agent (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    agent_name VARCHAR(255) NOT NULL UNIQUE,
    agent_description VARCHAR(1000) NOT NULL,
    system_prompt TEXT,  -- 已废弃
    next_step_prompt TEXT NOT NULL,
    class_name VARCHAR(255) NOT NULL,
    model_id BIGINT,
    namespace VARCHAR(255)
);

CREATE TABLE dynamic_agent_tools (
    agent_id BIGINT,
    tool_key VARCHAR(255),
    FOREIGN KEY (agent_id) REFERENCES dynamic_agent(id)
);
```

### 4.2 MCP配置相关

#### 4.2.1 McpConfigEntity
```sql
CREATE TABLE mcp_config (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    mcp_server_name VARCHAR(255) NOT NULL,
    connection_type ENUM('SSE', 'STUDIO', 'STREAMING'),
    config_json TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 4.3 执行记录相关

#### 4.3.1 PlanExecutionRecord
```sql
CREATE TABLE plan_execution_record (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    plan_id VARCHAR(255) NOT NULL,
    conversation_id VARCHAR(255),
    user_request TEXT,
    status VARCHAR(50),
    start_time TIMESTAMP,
    end_time TIMESTAMP,
    result TEXT,
    error_message TEXT
);
```

## 5. API设计

### 5.1 智能体管理API

#### 5.1.1 获取所有智能体
```http
GET /api/agents
Response: List<AgentConfig>
```

#### 5.1.2 创建智能体
```http
POST /api/agents
Body: AgentConfig
Response: AgentConfig
```

#### 5.1.3 更新智能体
```http
PUT /api/agents/{id}
Body: AgentConfig
Response: AgentConfig
```

#### 5.1.4 删除智能体
```http
DELETE /api/agents/{id}
Response: 204 No Content
```

#### 5.1.5 获取可用工具
```http
GET /api/agents/tools
Response: List<Tool>
```

### 5.2 MCP管理API

#### 5.2.1 添加MCP服务器
```http
POST /api/mcp/servers
Body: McpConfigRequestVO
Response: McpConfigResponseVO
```

#### 5.2.2 删除MCP服务器
```http
DELETE /api/mcp/servers/{id}
Response: 204 No Content
```

#### 5.2.3 获取MCP服务器列表
```http
GET /api/mcp/servers
Response: List<McpConfigEntity>
```

### 5.3 计划执行API

#### 5.3.1 执行计划
```http
POST /api/plans/execute
Body: ExecutionRequest
Response: ExecutionResult
```

#### 5.3.2 查询执行状态
```http
GET /api/plans/{planId}/status
Response: ExecutionStatus
```

## 6. 前端设计

### 6.1 技术栈

- **Vue 3**: 响应式框架
- **TypeScript**: 类型安全
- **Vite**: 构建工具
- **Element Plus**: UI组件库

### 6.2 主要页面

#### 6.2.1 智能体配置页面 (agentConfig.vue)

**功能特性**:
- 智能体列表展示
- 创建/编辑智能体
- 工具分配管理
- 模型配置

**组件结构**:
```
AgentConfig
├── AgentList (智能体列表)
├── AgentDetail (智能体详情)
├── ToolSelectionModal (工具选择弹窗)
└── ModelConfig (模型配置)
```

#### 6.2.2 MCP配置页面 (mcpConfig.vue)

**功能特性**:
- MCP服务器管理
- 连接类型配置
- JSON配置编辑
- 连接状态监控

#### 6.2.3 工具选择组件 (tool-selection-modal)

**功能特性**:
- 按服务组分类显示工具
- 搜索和过滤功能
- 批量选择和取消
- 工具状态管理

### 6.3 状态管理

**智能体状态**:
```typescript
interface AgentState {
  agents: Agent[]
  currentAgent: Agent | null
  availableTools: Tool[]
  loading: boolean
}
```

**MCP状态**:
```typescript
interface McpState {
  servers: McpServer[]
  connectionStates: Map<string, ConnectionState>
  loading: boolean
}
```

## 7. 部署和配置

### 7.1 环境要求

- Java 17+
- Node.js 16+
- 数据库：H2/MySQL 8.0+/PostgreSQL 12+

### 7.2 配置文件

#### 7.2.1 应用配置 (application.yml)
```yaml
spring:
  profiles:
    active: h2  # h2/mysql/postgres
  
  ai:
    dashscope:
      api-key: ${DASHSCOPE_API_KEY}
      chat:
        model: qwen-max

  datasource:
    url: jdbc:h2:file:./h2-data/jmanus
    username: sa
    password:

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: false

server:
  port: 18080

manus:
  max-steps: 10
  enable-browser: true
  chrome-driver-path: /usr/bin/chromedriver
```

#### 7.2.2 数据库配置

**MySQL配置 (application-mysql.yml)**:
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/jmanus
    username: root
    password: password
  jpa:
    database-platform: org.hibernate.dialect.MySQLDialect
```

**PostgreSQL配置 (application-postgres.yml)**:
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/jmanus
    username: postgres
    password: password
  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
```

### 7.3 Docker部署

#### 7.3.1 Dockerfile
```dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app
COPY target/jmanus.jar app.jar

EXPOSE 18080

ENV DASHSCOPE_API_KEY=""
ENV SPRING_PROFILES_ACTIVE=h2

CMD ["java", "-jar", "app.jar"]
```

#### 7.3.2 docker-compose.yml
```yaml
version: '3.8'
services:
  jmanus:
    build: .
    ports:
      - "18080:18080"
    environment:
      - DASHSCOPE_API_KEY=${DASHSCOPE_API_KEY}
      - SPRING_PROFILES_ACTIVE=mysql
    depends_on:
      - mysql
  
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: jmanus
    ports:
      - "3306:3306"
```

## 8. 扩展和定制

### 8.1 自定义智能体

#### 8.1.1 继承BaseAgent
```java
@Component
public class CustomAgent extends BaseAgent {
    @Override
    public String getName() {
        return "CustomAgent";
    }
    
    @Override
    public void clearUp(String planId) {
        // 清理逻辑
    }
}
```

#### 8.1.2 注册智能体
```java
@DynamicAgentDefinition(
    name = "CustomAgent",
    description = "自定义智能体",
    availableTools = {"tool1", "tool2"}
)
public class CustomAgentConfig {
    // 配置逻辑
}
```

### 8.2 自定义工具

#### 8.2.1 实现工具接口
```java
public class CustomTool implements ToolCallBiFunctionDef<CustomInput> {
    @Override
    public String getName() {
        return "custom-tool";
    }
    
    @Override
    public ToolExecuteResult run(CustomInput input) {
        // 工具执行逻辑
        return new ToolExecuteResult("result");
    }
    
    // 其他必需方法...
}
```

#### 8.2.2 注册工具
在PlanningFactory中添加工具注册：
```java
toolDefinitions.add(new CustomTool());
```

### 8.3 MCP集成

#### 8.3.1 配置MCP服务器

**本地MCP服务器 (STUDIO模式)**:
```json
{
  "command": "node",
  "args": ["/path/to/mcp-server.js"],
  "env": {
    "API_KEY": "your-api-key"
  }
}
```

**远程MCP服务器 (SSE/STREAMING模式)**:
```json
{
  "url": "https://mcp.example.com/api/v1",
  "headers": {
    "Authorization": "Bearer token"
  }
}
```

## 9. 监控和日志

### 9.1 执行记录

系统自动记录所有执行过程：
- 计划创建和执行状态
- 智能体思考和行动记录
- 工具调用和结果
- 错误和异常信息

### 9.2 日志配置 (logback-spring.xml)

```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/jmanus.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/jmanus.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <logger name="com.alibaba.cloud.ai.example.manus" level="INFO"/>
    
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

## 10. 安全考虑

### 10.1 API安全

- 跨域配置 (CORS)
- 请求验证和参数校验
- 错误信息脱敏

### 10.2 工具安全

- 工具权限控制
- 命令执行限制
- 文件访问控制

### 10.3 数据安全

- 敏感数据加密存储
- API密钥安全管理
- 执行日志脱敏

## 11. 性能优化

### 11.1 缓存策略

- MCP工具回调缓存 (10分钟过期)
- 智能体配置缓存
- 执行结果缓存

### 11.2 异步处理

- 长时间执行任务异步化
- 工具调用超时控制
- 连接池管理

### 11.3 资源管理

- 智能体实例生命周期管理
- 工具资源清理
- 内存使用监控

## 12. 故障排除

### 12.1 常见问题

#### 12.1.1 MCP连接失败
- 检查网络连接
- 验证配置JSON格式
- 查看服务器日志

#### 12.1.2 智能体执行异常
- 检查工具配置
- 验证提示词格式
- 查看执行记录

#### 12.1.3 数据库连接问题
- 检查数据库配置
- 验证连接参数
- 查看数据库日志

### 12.2 日志分析

重要日志模式：
```
INFO  PlanningCoordinator - Starting plan execution: {planId}
ERROR DynamicAgent - Agent execution failed: {error}
WARN  McpService - MCP connection timeout: {serverName}
```

## 13. 未来路线图

### 13.1 短期计划

- 增强MCP协议支持
- 优化执行性能
- 完善监控面板

### 13.2 长期规划

- 支持更多AI模型提供商
- 集成更多内置工具
- 提供插件系统
- 支持分布式部署

---

**文档版本**: 1.0  
**最后更新**: 2025年1月  
**维护者**: Spring AI Alibaba Team