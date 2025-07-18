# JManus 用户故事设计文档

## 1. 概述

本文档基于JManus系统的Vue3前端代码，整理了系统已支持的用户故事，并提供完整的流程图分析。JManus是一个多智能体协作系统，提供Web界面供用户配置和使用智能体。

## 2. 已支持的用户故事清单

### 2.1 核心用户故事

| 序号 | 用户故事 | 页面/组件 | 优先级 | 状态 |
|------|----------|-----------|--------|------|
| 1 | 作为用户，我希望在首页输入任务并快速执行 | home/index.vue | 高 | ✅ 已实现 |
| 2 | 作为用户，我希望与智能体进行对话交互 | direct/index.vue | 高 | ✅ 已实现 |
| 3 | 作为管理员，我希望配置和管理智能体 | configs/agentConfig.vue | 高 | ✅ 已实现 |
| 4 | 作为管理员，我希望配置AI模型 | configs/modelConfig.vue | 高 | ✅ 已实现 |
| 5 | 作为管理员，我希望配置MCP服务器 | configs/mcpConfig.vue | 中 | ✅ 已实现 |
| 6 | 作为管理员，我希望管理提示词模板 | configs/dynamicPromptConfig.vue | 中 | ✅ 已实现 |
| 7 | 作为管理员，我希望配置系统基础设置 | configs/basicConfig.vue | 中 | ✅ 已实现 |
| 8 | 作为用户，我希望选择智能体可用的工具 | tool-selection-modal/index.vue | 中 | ✅ 已实现 |
| 9 | 作为用户，我希望切换界面语言 | language-switcher/index.vue | 低 | ✅ 已实现 |
| 10 | 作为用户，我希望看到友好的错误页面 | error/notFound.vue | 低 | ✅ 已实现 |

### 2.2 详细用户故事描述

#### 故事1：首页任务输入与执行
- **用户角色**：普通用户
- **需求描述**：作为用户，我希望能在首页直接输入任务描述，系统能理解我的需求并快速启动执行
- **验收标准**：
  - 提供清晰的输入框和示例提示
  - 支持快捷键（Enter）发送
  - 能自动跳转到执行页面
  - 保存用户输入的任务内容

#### 故事2：智能体对话交互
- **用户角色**：普通用户
- **需求描述**：作为用户，我希望能与智能体进行连续对话，实时查看执行进度和结果
- **验收标准**：
  - 支持多轮对话
  - 实时显示智能体状态
  - 可以查看执行计划和步骤
  - 支持停止执行操作

#### 故事3：智能体配置管理
- **用户角色**：系统管理员
- **需求描述**：作为管理员，我希望能创建、编辑、删除智能体，配置其工具和模型
- **验收标准**：
  - 完整的CRUD操作界面
  - 工具选择和配置功能
  - 模型关联配置
  - 批量操作支持

#### 故事4：AI模型配置
- **用户角色**：系统管理员
- **需求描述**：作为管理员，我希望能配置不同的AI模型提供商和参数
- **验收标准**：
  - 支持多种模型提供商
  - 模型参数配置
  - 连接测试功能
  - 模型使用统计

#### 故事5：MCP服务器配置
- **用户角色**：系统管理员
- **需求描述**：作为管理员，我希望能配置MCP服务器以扩展系统工具能力
- **验收标准**：
  - 支持多种连接类型（SSE、STUDIO、STREAMING）
  - JSON配置编辑器
  - 连接状态监控
  - 配置验证功能

## 3. 系统架构流程图

### 3.1 整体用户流程

```mermaid
graph TB
    Start([用户访问系统]) --> Home{首页}
    Home --> |输入任务| TaskInput[任务输入]
    Home --> |配置管理| Config[配置中心]
    
    TaskInput --> Direct[对话页面]
    Direct --> Chat[智能体对话]
    Chat --> Execute[执行计划]
    Execute --> Result[查看结果]
    
    Config --> AgentConfig[智能体配置]
    Config --> ModelConfig[模型配置]
    Config --> McpConfig[MCP配置]
    Config --> PromptConfig[提示词配置]
    Config --> BasicConfig[基础配置]
    
    AgentConfig --> ToolSelection[工具选择]
    
    Result --> |继续对话| Chat
    Result --> |新任务| Home
    
    style Home fill:#e1f5fe
    style Direct fill:#f3e5f5
    style Config fill:#e8f5e8
```

### 3.2 首页任务输入流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant Home as 首页组件
    participant Store as 任务存储
    participant Router as 路由
    participant Direct as 对话页面
    
    User->>Home: 访问首页
    Home->>User: 显示输入界面和示例
    
    User->>Home: 输入任务内容
    Note over User,Home: 支持Enter快捷键
    
    Home->>Store: 保存任务到store
    Store-->>Home: 确认保存
    
    Home->>Router: 导航到对话页面
    Router->>Direct: 加载对话组件
    
    Direct->>Store: 获取任务内容
    Store-->>Direct: 返回任务数据
    
    Direct->>User: 显示对话界面
    Note over Direct,User: 自动开始执行任务
```

### 3.3 智能体配置流程

```mermaid
sequenceDiagram
    participant Admin as 管理员
    participant Config as 配置页面
    participant AgentAPI as 智能体API
    participant ToolModal as 工具选择弹窗
    participant ModelAPI as 模型API
    
    Admin->>Config: 进入智能体配置
    Config->>AgentAPI: 获取智能体列表
    AgentAPI-->>Config: 返回智能体数据
    
    Admin->>Config: 创建/编辑智能体
    Config->>Admin: 显示编辑表单
    
    Admin->>Config: 选择工具
    Config->>ToolModal: 打开工具选择弹窗
    ToolModal->>AgentAPI: 获取可用工具
    AgentAPI-->>ToolModal: 返回工具列表
    Admin->>ToolModal: 选择工具
    ToolModal-->>Config: 返回选中工具
    
    Admin->>Config: 选择模型
    Config->>ModelAPI: 获取模型列表
    ModelAPI-->>Config: 返回模型数据
    
    Admin->>Config: 保存配置
    Config->>AgentAPI: 提交智能体配置
    AgentAPI-->>Config: 确认保存成功
    Config-->>Admin: 显示成功消息
```

### 3.4 对话执行流程

```mermaid
stateDiagram-v2
    [*] --> 初始化
    初始化 --> 准备就绪: 加载组件
    准备就绪 --> 执行中: 发送任务
    执行中 --> 思考阶段: 智能体分析
    思考阶段 --> 行动阶段: 制定计划
    行动阶段 --> 工具调用: 执行操作
    工具调用 --> 结果处理: 获取结果
    结果处理 --> 执行中: 继续执行
    结果处理 --> 完成: 任务完成
    执行中 --> 暂停: 用户暂停
    暂停 --> 执行中: 用户恢复
    暂停 --> 停止: 用户停止
    完成 --> 准备就绪: 新任务
    停止 --> 准备就绪: 重新开始
    
    执行中: 显示进度
    思考阶段: 显示思考过程
    行动阶段: 显示计划步骤
    工具调用: 显示工具使用
    结果处理: 显示执行结果
```

## 4. 页面组件详细说明

### 4.1 首页组件 (home/index.vue)

**功能特性**：
- 欢迎界面和系统介绍
- 任务输入框（支持Enter快捷键）
- 示例任务展示卡片
- 语言切换功能
- 任务存储和路由导航

**交互流程**：
```mermaid
graph LR
    A[用户输入] --> B{验证输入}
    B --> |有效| C[保存到Store]
    B --> |无效| A
    C --> D[导航到对话页]
    D --> E[开始执行]
```

### 4.2 对话页面 (direct/index.vue)

**功能特性**：
- 侧边栏（计划执行状态）
- 对话容器（消息展示）
- 输入区域（支持多种输入模式）
- 右侧面板（执行详情）
- 语言切换和配置入口

**布局结构**：
```mermaid
graph TB
    Direct[对话页面] --> Sidebar[侧边栏]
    Direct --> LeftPanel[左侧面板]
    Direct --> RightPanel[右侧面板]
    
    LeftPanel --> ChatHeader[聊天头部]
    LeftPanel --> ChatContainer[对话容器]
    LeftPanel --> InputArea[输入区域]
    
    RightPanel --> ExecutionDetails[执行详情]
    RightPanel --> PlanSteps[计划步骤]
    
    Sidebar --> PlanStatus[计划状态]
    Sidebar --> AgentInfo[智能体信息]
```

### 4.3 配置中心 (configs/index.vue)

**功能模块**：
- 基础配置：系统参数设置
- 智能体配置：CRUD操作和工具选择
- 模型配置：AI模型参数设置
- MCP配置：外部服务集成
- 提示词配置：模板管理

**导航结构**：
```mermaid
graph LR
    ConfigCenter[配置中心] --> BasicConfig[基础配置]
    ConfigCenter --> AgentConfig[智能体配置]
    ConfigCenter --> ModelConfig[模型配置]
    ConfigCenter --> McpConfig[MCP配置]
    ConfigCenter --> PromptConfig[提示词配置]
    
    AgentConfig --> ToolSelection[工具选择]
    AgentConfig --> ModelBinding[模型绑定]
    
    McpConfig --> ConnectionTest[连接测试]
    McpConfig --> ConfigEditor[配置编辑]
```

### 4.4 工具选择弹窗 (tool-selection-modal/index.vue)

**功能特性**：
- 按服务组分类显示工具
- 搜索和过滤功能
- 批量选择和取消选择
- 工具状态管理（启用/禁用）
- 组级别的全选功能

**交互逻辑**：
```mermaid
graph TB
    OpenModal[打开工具选择] --> LoadTools[加载工具列表]
    LoadTools --> GroupTools[按服务组分组]
    GroupTools --> DisplayTools[显示工具列表]
    
    DisplayTools --> Search[搜索过滤]
    DisplayTools --> Sort[排序]
    DisplayTools --> Select[选择工具]
    
    Select --> Individual[单个选择]
    Select --> GroupSelect[组级选择]
    
    Individual --> UpdateState[更新状态]
    GroupSelect --> UpdateState
    
    UpdateState --> Confirm[确认选择]
    Confirm --> SaveSelection[保存选择]
```

## 5. 技术实现要点

### 5.1 状态管理

**任务存储 (task store)**：
```typescript
interface TaskState {
  currentTask: string | null
  homeVisited: boolean
}
```

**侧边栏存储 (sidebar store)**：
```typescript
interface SidebarState {
  collapsed: boolean
  activePanel: string
}
```

### 5.2 国际化支持

系统支持多语言切换，包括：
- 中文（简体）
- 英文
- 动态语言切换组件
- 完整的翻译键值对

### 5.3 响应式设计

- 自适应布局
- 移动端友好
- 灵活的面板尺寸调整
- 可折叠的侧边栏

### 5.4 API集成

主要API服务包括：
- `PlanActApiService`: 计划执行相关API
- `AgentApiService`: 智能体管理API
- `ModelApiService`: 模型配置API
- `McpApiService`: MCP服务器API
- `PromptApiService`: 提示词管理API
- `AdminApiService`: 系统配置API

## 6. 用户旅程图

### 6.1 新用户首次使用

```mermaid
journey
    title 新用户首次使用JManus系统
    section 初次访问
      访问首页: 5: 用户
      查看示例: 4: 用户
      了解功能: 4: 用户
    section 尝试使用
      输入简单任务: 5: 用户
      查看执行过程: 5: 用户
      获得结果: 5: 用户
    section 深入使用
      进入配置中心: 3: 用户
      配置智能体: 2: 用户
      选择工具: 3: 用户
    section 熟练使用
      创建复杂任务: 5: 用户
      监控执行: 5: 用户
      优化配置: 4: 用户
```

### 6.2 管理员配置系统

```mermaid
journey
    title 管理员配置JManus系统
    section 系统设置
      配置基础参数: 4: 管理员
      设置AI模型: 5: 管理员
      配置MCP服务: 3: 管理员
    section 智能体管理
      创建智能体: 5: 管理员
      配置工具: 4: 管理员
      测试功能: 5: 管理员
    section 维护优化
      监控性能: 4: 管理员
      更新配置: 4: 管理员
      用户培训: 3: 管理员
```

## 7. 未来扩展方向

### 7.1 短期优化

1. **用户体验改进**
   - 添加更多交互提示
   - 优化加载状态显示
   - 增强错误处理

2. **功能增强**
   - 支持批量任务执行
   - 添加任务历史记录
   - 增加更多配置选项

### 7.2 长期规划

1. **高级功能**
   - 用户权限管理
   - 多租户支持
   - API监控面板

2. **集成扩展**
   - 更多MCP服务器集成
   - 第三方工具支持
   - 自定义插件系统

---

**文档版本**: 1.0  
**最后更新**: 2025年1月  
**维护者**: Spring AI Alibaba Team