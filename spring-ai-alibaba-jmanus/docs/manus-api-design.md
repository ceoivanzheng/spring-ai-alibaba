# JManus API 设计文档

## 1. 概述

Spring AI Alibaba JManus 系统提供了完整的REST API接口，供前端界面和第三方系统集成使用。所有API都遵循RESTful设计规范，支持跨域访问(CORS)。

### 1.1 基本信息

- **基础URL**: `http://localhost:18080`
- **API版本**: v1
- **数据格式**: JSON
- **字符编码**: UTF-8
- **跨域支持**: 已启用 (`@CrossOrigin(origins = "*")`)

### 1.2 API接口清单汇总

| 模块 | 方法 | 路径 | 描述 |
|------|------|------|------|
| **智能体管理** | GET | `/api/agents` | 获取所有智能体 |
| | GET | `/api/agents/namespace/{namespace}` | 按命名空间获取智能体 |
| | GET | `/api/agents/{id}` | 获取智能体详情 |
| | POST | `/api/agents` | 创建智能体 |
| | PUT | `/api/agents/{id}` | 更新智能体 |
| | DELETE | `/api/agents/{id}` | 删除智能体 |
| | GET | `/api/agents/tools` | 获取可用工具列表 |
| **MCP配置管理** | GET | `/api/mcp/list` | 获取所有MCP服务器配置 |
| | POST | `/api/mcp/add` | 添加MCP服务器配置 |
| **AI模型管理** | GET | `/api/models` | 获取所有AI模型 |
| | GET | `/api/models/{id}` | 获取模型详情 |
| | POST | `/api/models` | 创建模型配置 |
| | PUT | `/api/models/{id}` | 更新模型配置 |
| | DELETE | `/api/models/{id}` | 删除模型配置 |
| | GET | `/api/models/types` | 获取所有模型类型 |
| **提示词管理** | GET | `/api/prompt` | 获取所有提示词 |
| | GET | `/api/prompt/namespace/{namespace}` | 按命名空间获取提示词 |
| | GET | `/api/prompt/{id}` | 获取提示词详情 |
| | POST | `/api/prompt` | 创建提示词 |
| | PUT | `/api/prompt/{id}` | 更新提示词 |
| | DELETE | `/api/prompt/{id}` | 删除提示词 |
| **计划模板管理** | POST | `/api/plan-template/save` | 保存计划模板 |
| | POST | `/api/plan-template/versions` | 获取计划版本历史 |
| | POST | `/api/plan-template/get-version` | 获取特定版本的计划 |
| **计划执行** | POST | `/api/executor/execute` | 执行计划 |
| | GET | `/api/executor/details/{planId}` | 获取执行详情 |
| | POST | `/api/executor/stop` | 停止执行 |
| | GET | `/api/executor/status/{planId}` | 查询执行状态 |
| **系统配置管理** | GET | `/api/config/group/{groupName}` | 按组获取配置 |
| | POST | `/api/config/batch-update` | 批量更新配置 |

### 1.3 通用响应格式

#### 成功响应
```json
{
  "data": {},
  "timestamp": "2025-01-XX 10:00:00"
}
```

#### 错误响应
```json
{
  "error": "错误描述信息",
  "status": 400,
  "timestamp": "2025-01-XX 10:00:00"
}
```

## 2. 智能体管理 API

### 2.1 接口概览

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/api/agents` | 获取所有智能体 |
| GET | `/api/agents/namespace/{namespace}` | 按命名空间获取智能体 |
| GET | `/api/agents/{id}` | 获取智能体详情 |
| POST | `/api/agents` | 创建智能体 |
| PUT | `/api/agents/{id}` | 更新智能体 |
| DELETE | `/api/agents/{id}` | 删除智能体 |
| GET | `/api/agents/tools` | 获取可用工具列表 |

### 2.2 数据模型

#### AgentConfig
```json
{
  "id": "string",
  "name": "string",
  "description": "string",
  "nextStepPrompt": "string",
  "availableTools": ["string"],
  "model": {
    "id": "string",
    "modelName": "string",
    "provider": "string"
  },
  "namespace": "string"
}
```

#### Tool
```json
{
  "key": "string",
  "name": "string", 
  "description": "string",
  "enabled": true,
  "serviceGroup": "string"
}
```

### 2.3 接口详情

#### 2.3.1 获取所有智能体
```http
GET /api/agents
Accept: application/json
```

**响应示例**:
```json
[
  {
    "id": "1",
    "name": "数据分析智能体",
    "description": "专门用于数据分析和处理的智能体",
    "nextStepPrompt": "请根据用户需求进行数据分析...",
    "availableTools": ["google_search", "python_execute", "data_analyzer"],
    "model": {
      "id": "1",
      "modelName": "qwen-max",
      "provider": "dashscope"
    },
    "namespace": "default"
  }
]
```

#### 2.3.2 创建智能体
```http
POST /api/agents
Content-Type: application/json

{
  "name": "新智能体",
  "description": "智能体描述",
  "nextStepPrompt": "执行提示词",
  "availableTools": ["tool1", "tool2"],
  "model": {
    "id": "1"
  },
  "namespace": "default"
}
```

#### 2.3.3 更新智能体
```http
PUT /api/agents/{id}
Content-Type: application/json

{
  "name": "更新后的智能体名称",
  "description": "更新后的描述",
  "nextStepPrompt": "更新后的提示词",
  "availableTools": ["updated_tool1", "updated_tool2"]
}
```

#### 2.3.4 删除智能体
```http
DELETE /api/agents/{id}
```

**响应**:
- 成功: `200 OK`
- 参数错误: `400 Bad Request`

#### 2.3.5 获取可用工具列表
```http
GET /api/agents/tools
```

**响应示例**:
```json
[
  {
    "key": "google_search",
    "name": "谷歌搜索",
    "description": "通过谷歌搜索获取信息",
    "enabled": true,
    "serviceGroup": "search"
  },
  {
    "key": "python_execute",
    "name": "Python执行器",
    "description": "执行Python代码",
    "enabled": true,
    "serviceGroup": "code"
  }
]
```

## 3. MCP配置管理 API

### 3.1 接口概览

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/api/mcp/list` | 获取所有MCP服务器配置 |
| POST | `/api/mcp/add` | 添加MCP服务器配置 |

### 3.2 数据模型

#### McpConfigVO
```json
{
  "id": "long",
  "mcpServerName": "string",
  "connectionType": "SSE|STUDIO|STREAMING",
  "configJson": "string",
  "isActive": true,
  "createdAt": "2025-01-XX 10:00:00"
}
```

#### McpConfigRequestVO
```json
{
  "mcpServerName": "string",
  "connectionType": "SSE|STUDIO|STREAMING", 
  "configJson": "string"
}
```

### 3.3 接口详情

#### 3.3.1 获取所有MCP服务器配置
```http
GET /api/mcp/list
Accept: application/json
```

**响应示例**:
```json
[
  {
    "id": 1,
    "mcpServerName": "file-system",
    "connectionType": "STUDIO",
    "configJson": "{\"command\": \"node\", \"args\": [\"/path/to/mcp-server.js\"]}",
    "isActive": true,
    "createdAt": "2025-01-15 10:00:00"
  }
]
```

#### 3.3.2 添加MCP服务器配置
```http
POST /api/mcp/add
Content-Type: application/json

{
  "mcpServerName": "weather-service",
  "connectionType": "SSE",
  "configJson": "{\"url\": \"https://weather.api.com/mcp\", \"headers\": {\"Authorization\": \"Bearer token\"}}"
}
```

**连接类型说明**:
- **SSE**: Server-Sent Events连接，适用于实时数据流
- **STUDIO**: 本地STDIO连接，适用于本地MCP服务器
- **STREAMING**: 流式连接，适用于长连接场景

## 4. AI模型管理 API

### 4.1 接口概览

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/api/models` | 获取所有AI模型 |
| GET | `/api/models/{id}` | 获取模型详情 |
| POST | `/api/models` | 创建模型配置 |
| PUT | `/api/models/{id}` | 更新模型配置 |
| DELETE | `/api/models/{id}` | 删除模型配置 |
| GET | `/api/models/types` | 获取所有模型类型 |

### 4.2 数据模型

#### ModelConfig
```json
{
  "id": "string",
  "modelName": "string",
  "provider": "string",
  "type": "string",
  "baseUrl": "string",
  "apiKey": "string",
  "modelDescription": "string",
  "configuration": "string"
}
```

### 4.3 接口详情

#### 4.3.1 获取所有AI模型
```http
GET /api/models
Accept: application/json
```

**响应示例**:
```json
[
  {
    "id": "1",
    "modelName": "qwen-max",
    "provider": "dashscope",
    "type": "CHAT",
    "baseUrl": "https://dashscope.aliyuncs.com/api/v1",
    "apiKey": "sk-xxx...",
    "modelDescription": "阿里云通义千问大模型",
    "configuration": "{\"temperature\": 0.7, \"maxTokens\": 2000}"
  }
]
```

#### 4.3.2 创建模型配置
```http
POST /api/models
Content-Type: application/json

{
  "modelName": "gpt-4",
  "provider": "openai",
  "type": "CHAT",
  "baseUrl": "https://api.openai.com/v1",
  "apiKey": "sk-xxx...",
  "modelDescription": "OpenAI GPT-4模型"
}
```

#### 4.3.3 获取所有模型类型
```http
GET /api/models/types
```

**响应示例**:
```json
["CHAT", "EMBEDDING", "IMAGE", "AUDIO"]
```

## 5. 提示词管理 API

### 5.1 接口概览

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/api/prompt` | 获取所有提示词 |
| GET | `/api/prompt/namespace/{namespace}` | 按命名空间获取提示词 |
| GET | `/api/prompt/{id}` | 获取提示词详情 |
| POST | `/api/prompt` | 创建提示词 |
| PUT | `/api/prompt/{id}` | 更新提示词 |
| DELETE | `/api/prompt/{id}` | 删除提示词 |

### 5.2 数据模型

#### PromptVO
```json
{
  "id": "string",
  "promptName": "string",
  "promptDescription": "string",
  "promptContent": "string",
  "messageType": "string",
  "type": "string",
  "builtIn": false,
  "namespace": "string"
}
```

### 5.3 接口详情

#### 5.3.1 获取所有提示词
```http
GET /api/prompt
Accept: application/json
```

**响应示例**:
```json
[
  {
    "id": "1",
    "promptName": "数据分析提示词",
    "promptDescription": "用于数据分析任务的提示词模板",
    "promptContent": "请分析以下数据并给出结论：{data}",
    "messageType": "USER",
    "type": "ANALYSIS",
    "builtIn": false,
    "namespace": "default"
  }
]
```

#### 5.3.2 创建提示词
```http
POST /api/prompt
Content-Type: application/json

{
  "promptName": "新提示词",
  "promptDescription": "提示词描述",
  "promptContent": "提示词内容模板",
  "messageType": "USER",
  "type": "CUSTOM",
  "namespace": "default"
}
```

## 6. 计划模板管理 API

### 6.1 接口概览

| 方法 | 路径 | 描述 |
|------|------|------|
| POST | `/api/plan-template/save` | 保存计划模板 |
| POST | `/api/plan-template/versions` | 获取计划版本历史 |
| POST | `/api/plan-template/get-version` | 获取特定版本的计划 |

### 6.2 数据模型

#### 保存计划请求
```json
{
  "planJson": "string"
}
```

#### 版本查询请求
```json
{
  "planId": "string"
}
```

#### 特定版本查询请求
```json
{
  "planId": "string",
  "versionIndex": "string"
}
```

### 6.3 接口详情

#### 6.3.1 保存计划模板
```http
POST /api/plan-template/save
Content-Type: application/json

{
  "planJson": "{\"currentPlanId\": \"plan-001\", \"title\": \"数据分析计划\", \"steps\": [\"step1\", \"step2\"]}"
}
```

**响应示例**:
```json
{
  "planId": "plan-001",
  "saved": true,
  "message": "计划保存成功"
}
```

#### 6.3.2 获取计划版本历史
```http
POST /api/plan-template/versions
Content-Type: application/json

{
  "planId": "plan-001"
}
```

**响应示例**:
```json
{
  "planId": "plan-001",
  "versionCount": 3,
  "versions": ["v1.0", "v1.1", "v1.2"]
}
```

## 7. 计划执行 API

### 7.1 接口概览

| 方法 | 路径 | 描述 |
|------|------|------|
| POST | `/api/executor/execute` | 执行计划 |
| GET | `/api/executor/details/{planId}` | 获取执行详情 |
| POST | `/api/executor/stop` | 停止执行 |
| GET | `/api/executor/status/{planId}` | 查询执行状态 |

### 7.2 数据模型

#### ExecutionRequest
```json
{
  "userRequest": "string",
  "agentName": "string",
  "planType": "SIMPLE|MAPREDUCE",
  "maxSteps": 10,
  "terminateColumns": ["string"]
}
```

#### ExecutionResult
```json
{
  "planId": "string",
  "status": "RUNNING|COMPLETED|FAILED|CANCELLED",
  "result": "string",
  "errorMessage": "string",
  "startTime": "2025-01-XX 10:00:00",
  "endTime": "2025-01-XX 10:30:00"
}
```

### 7.3 接口详情

#### 7.3.1 执行计划
```http
POST /api/executor/execute
Content-Type: application/json

{
  "userRequest": "分析销售数据并生成报告",
  "agentName": "数据分析智能体",
  "planType": "SIMPLE",
  "maxSteps": 10,
  "terminateColumns": ["销售额", "增长率", "趋势"]
}
```

**响应示例**:
```json
{
  "planId": "plan-20250115-001",
  "status": "RUNNING",
  "message": "计划执行已启动",
  "startTime": "2025-01-15 10:00:00"
}
```

#### 7.3.2 获取执行详情
```http
GET /api/executor/details/{planId}
```

**响应示例**:
```json
{
  "currentPlanId": "plan-20250115-001",
  "userRequest": "分析销售数据并生成报告",
  "status": "COMPLETED",
  "startTime": "2025-01-15 10:00:00",
  "endTime": "2025-01-15 10:30:00",
  "agentExecutionSequence": [
    {
      "agentName": "数据分析智能体",
      "status": "FINISHED",
      "currentStep": 5,
      "maxSteps": 10,
      "thinkActSteps": [
        {
          "stepName": "数据收集",
          "thinkingOutput": "需要收集最近一个月的销售数据",
          "actionOutput": "成功获取销售数据",
          "status": "COMPLETED"
        }
      ]
    }
  ],
  "summary": "数据分析完成，生成了详细的销售报告"
}
```

## 8. 系统配置管理 API

### 8.1 接口概览

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | `/api/config/group/{groupName}` | 按组获取配置 |
| POST | `/api/config/batch-update` | 批量更新配置 |

### 8.2 数据模型

#### ConfigEntity
```json
{
  "id": "long",
  "groupName": "string",
  "configKey": "string",
  "configValue": "string",
  "description": "string",
  "type": "string"
}
```

### 8.3 接口详情

#### 8.3.1 按组获取配置
```http
GET /api/config/group/llm
```

**响应示例**:
```json
[
  {
    "id": 1,
    "groupName": "llm",
    "configKey": "max_tokens",
    "configValue": "2000",
    "description": "最大生成令牌数",
    "type": "INTEGER"
  },
  {
    "id": 2,
    "groupName": "llm",
    "configKey": "temperature",
    "configValue": "0.7",
    "description": "生成温度",
    "type": "FLOAT"
  }
]
```

#### 8.3.2 批量更新配置
```http
POST /api/config/batch-update
Content-Type: application/json

[
  {
    "id": 1,
    "configValue": "3000"
  },
  {
    "id": 2,
    "configValue": "0.8"
  }
]
```

## 9. API 错误处理

### 9.1 HTTP状态码

| 状态码 | 描述 | 使用场景 |
|--------|------|----------|
| 200 | OK | 请求成功 |
| 400 | Bad Request | 请求参数错误 |
| 404 | Not Found | 资源不存在 |
| 500 | Internal Server Error | 服务器内部错误 |

### 9.2 错误响应格式

```json
{
  "error": "错误描述信息",
  "status": 400,
  "timestamp": "2025-01-15 10:00:00",
  "path": "/api/agents"
}
```

## 10. API 认证和安全

### 10.1 跨域配置
- 所有API接口都支持CORS跨域访问
- 允许的源：`*`（开发环境）
- 生产环境建议配置具体的前端域名

### 10.2 请求头要求
```http
Content-Type: application/json
Accept: application/json
```

### 10.3 安全建议
- 敏感配置（如API密钥）通过环境变量管理
- 生产环境启用HTTPS
- 添加请求频率限制
- 实现API访问日志记录

## 11. API 使用示例

### 11.1 创建并执行智能体任务完整流程

#### 步骤1：创建AI模型配置
```http
POST /api/models
Content-Type: application/json

{
  "modelName": "qwen-max",
  "provider": "dashscope",
  "type": "CHAT",
  "baseUrl": "https://dashscope.aliyuncs.com/api/v1",
  "apiKey": "sk-xxx...",
  "modelDescription": "阿里云通义千问大模型"
}
```

#### 步骤2：创建智能体
```http
POST /api/agents
Content-Type: application/json

{
  "name": "数据分析专家",
  "description": "专门用于数据分析和可视化的智能体",
  "nextStepPrompt": "基于用户需求，分析数据并生成专业报告",
  "availableTools": ["python_execute", "google_search", "text_file_operator"],
  "model": {
    "id": "1"
  },
  "namespace": "default"
}
```

#### 步骤3：执行任务
```http
POST /api/executor/execute
Content-Type: application/json

{
  "userRequest": "分析最近三个月的销售数据，找出增长趋势和关键影响因素",
  "agentName": "数据分析专家",
  "planType": "SIMPLE",
  "maxSteps": 15,
  "terminateColumns": ["月份", "销售额", "增长率", "主要因素"]
}
```

#### 步骤4：查询执行状态
```http
GET /api/executor/details/plan-20250115-001
```

### 11.2 配置MCP服务器示例

#### 添加本地文件系统MCP服务器
```http
POST /api/mcp/add
Content-Type: application/json

{
  "mcpServerName": "filesystem",
  "connectionType": "STUDIO",
  "configJson": "{\"command\": \"node\", \"args\": [\"/path/to/filesystem-mcp-server.js\"], \"env\": {\"HOME\": \"/Users/username\"}}"
}
```

#### 添加远程API MCP服务器
```http
POST /api/mcp/add
Content-Type: application/json

{
  "mcpServerName": "weather-api",
  "connectionType": "SSE",
  "configJson": "{\"url\": \"https://weather-mcp.example.com/api\", \"headers\": {\"Authorization\": \"Bearer your-token\"}}"
}
```

## 12. 前端集成指南

### 12.1 TypeScript类型定义

#### Agent相关类型
```typescript
export interface Agent {
  id: string
  name: string
  description: string
  availableTools: string[]
  nextStepPrompt?: string
  model?: Model | null
}

export interface Tool {
  key: string
  name: string
  description: string
  enabled: boolean
  serviceGroup?: string
}
```

#### 执行相关类型
```typescript
export interface ExecutionRequest {
  userRequest: string
  agentName: string
  planType: 'SIMPLE' | 'MAPREDUCE'
  maxSteps?: number
  terminateColumns?: string[]
}

export interface ExecutionResult {
  planId: string
  status: 'RUNNING' | 'COMPLETED' | 'FAILED' | 'CANCELLED'
  result?: string
  errorMessage?: string
  startTime: string
  endTime?: string
}
```

### 12.2 API调用示例（JavaScript/TypeScript）

#### 基础API服务类
```typescript
class JManusApiService {
  private baseUrl = 'http://localhost:18080'

  // 通用请求方法
  private async request<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      headers: {
        'Content-Type': 'application/json',
        ...options.headers,
      },
      ...options,
    })

    if (!response.ok) {
      throw new Error(`API请求失败: ${response.status} ${response.statusText}`)
    }

    return response.json()
  }

  // 智能体相关API
  async getAllAgents(): Promise<Agent[]> {
    return this.request<Agent[]>('/api/agents')
  }

  async createAgent(agent: Omit<Agent, 'id'>): Promise<Agent> {
    return this.request<Agent>('/api/agents', {
      method: 'POST',
      body: JSON.stringify(agent),
    })
  }

  // 执行相关API
  async executeTask(request: ExecutionRequest): Promise<ExecutionResult> {
    return this.request<ExecutionResult>('/api/executor/execute', {
      method: 'POST',
      body: JSON.stringify(request),
    })
  }

  async getExecutionDetails(planId: string): Promise<any> {
    return this.request(`/api/executor/details/${planId}`)
  }
}
```

## 13. API版本控制和兼容性

### 13.1 版本策略
- 当前版本：v1
- API路径不包含版本号，通过Header指定版本
- 向后兼容性保证：至少支持前一个主版本

### 13.2 版本指定（可选）
```http
API-Version: v1
```

### 13.3 弃用策略
- 新功能优先在最新版本中实现
- 弃用的API至少提前3个月通知
- 提供迁移指南和工具

## 14. 性能和限制

### 14.1 请求限制
- 单次请求大小：最大10MB
- 并发连接数：每IP最大100个
- 长时间执行任务：支持异步处理

### 14.2 响应时间
- 一般API响应：< 1秒
- 复杂查询：< 5秒
- 任务执行：异步处理，通过状态查询获取结果

### 14.3 数据限制
- 智能体名称：最大255字符
- 提示词内容：最大40000字符
- 执行步骤数：最大100步

---

**文档版本**: 1.0  
**最后更新**: 2025年1月  
**维护者**: Spring AI Alibaba Team

## 附录

### A. 常用API端点快速参考

#### 智能体管理
```
GET    /api/agents           - 获取所有智能体
POST   /api/agents           - 创建智能体
GET    /api/agents/{id}      - 获取智能体详情
PUT    /api/agents/{id}      - 更新智能体
DELETE /api/agents/{id}      - 删除智能体
GET    /api/agents/tools     - 获取可用工具
```

#### 计划执行
```
POST /api/executor/execute      - 执行计划
GET  /api/executor/details/{id} - 获取执行详情
```

#### MCP配置
```
GET  /api/mcp/list  - 获取MCP服务器列表
POST /api/mcp/add   - 添加MCP服务器
```

### B. 错误代码参考

| 错误代码 | 描述 | 解决方案 |
|----------|------|----------|
| AGENT_NOT_FOUND | 智能体不存在 | 检查智能体ID是否正确 |
| INVALID_PLAN_TYPE | 无效的计划类型 | 使用SIMPLE或MAPREDUCE |
| MCP_CONNECTION_FAILED | MCP连接失败 | 检查MCP服务器配置 |
| EXECUTION_TIMEOUT | 执行超时 | 增加超时时间或优化任务 |