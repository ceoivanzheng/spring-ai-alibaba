# JManus 核心组件分解结构

本文档使用 PlantUML WBS（工作分解结构图）展示 JManus 系统的核心组件层次结构，按照分层架构进行组件分解。

## 组件分解结构图

```plantuml
@startwbs
* JManus AI 智能助手平台
** 用户接口层 (Presentation Layer)
*** Vue3 Web UI
**** 用户界面组件
**** 路由管理
**** 状态管理
*** REST API Gateway
**** 请求路由
**** 认证授权
**** 响应处理

** API控制器层 (Controller Layer)
*** Agent Controller
**** 智能体CRUD操作
**** 工具列表获取
*** Plan Template Controller
**** 计划生成
**** 计划保存
**** 版本管理
*** MCP Controller
**** MCP服务器管理
**** 连接配置
*** Model Controller
**** 模型配置管理
**** 模型类型获取
*** Prompt Controller
**** 提示词CRUD
**** 命名空间管理
*** Config Controller
**** 系统配置管理
**** 批量更新
*** Manus Controller
**** 执行器调用
**** 用户输入处理

** 业务服务层 (Business Service Layer)
*** Agent Service
**** 智能体管理
**** 工具回调获取
*** Plan Template Service
**** 计划模板管理
**** 版本控制
*** MCP Service
**** MCP客户端管理
**** 工具回调缓存
*** Model Service
**** 模型配置服务
**** 模型验证
*** Prompt Service
**** 提示词管理
**** 模板渲染
*** User Input Service
**** 表单输入管理
**** 等待状态处理
*** Config Service
**** 配置项管理
**** 分组操作

** 核心组件层 (Core Components)
*** Planning Coordinator
**** 计划协调
**** 执行流程管理
*** Plan Creator
**** LLM驱动计划生成
**** 智能体信息集成
*** Plan Executor
**** 计划步骤执行
**** 智能体调度
*** Dynamic Agent
**** 可配置智能体
**** 工具调用
*** Dynamic Agent Loader
**** 智能体动态加载
**** 配置管理
*** LLM Service
**** 大语言模型调用
**** 多模型支持
*** Plan Execution Recorder
**** 执行记录
**** 状态跟踪

** 工具生态层 (Tool Ecosystem)
*** Browser Tool
**** 网页浏览
**** 页面操作
**** 元素交互
*** Python Execute Tool
**** Python代码执行
**** 结果处理
*** Bash Tool
**** Shell命令执行
**** 目录管理
*** Planning Tool
**** 计划生成工具
**** 执行计划解析
*** Form Input Tool
**** 表单输入处理
**** 用户交互
*** MCP Tool Integration
**** MCP协议集成
**** 外部服务连接

** 技术支撑层 (Technology Layer)
*** Spring Boot Framework
**** 依赖注入
**** 自动配置
**** Web服务支持
*** Spring AI Framework
**** AI模型抽象
**** 工具调用管理
*** JPA/Hibernate
**** 对象关系映射
**** 事务管理
*** Vite Dev Server
**** 前端开发服务
**** 热重载
*** Database Interface
**** 数据库抽象
**** 连接管理
*** AI Model Interface
**** 模型调用接口
**** 响应处理

** 基础设施层 (Infrastructure)
*** 数据库系统
**** H2 Database
***** 嵌入式数据库
***** 开发测试环境
**** MySQL Database
***** 生产环境
***** 高性能存储
**** PostgreSQL Database
***** 企业级应用
***** 高级特性支持
*** AI服务接口
**** DashScope API
***** 阿里云通义千问
***** 模型调用
**** OpenAI API
***** GPT系列模型
***** 兼容接口
*** 外部服务
**** MCP Servers
***** 外部工具服务
***** 协议通信
**** Chrome Browser
***** 浏览器自动化
***** 页面渲染
**** Python Runtime
***** Python代码执行
***** 包管理

** 数据层 (Data Layer)
*** 配置数据
**** Agent Config
***** 智能体配置
***** 工具绑定
**** Model Config
***** 模型参数
***** API配置
**** MCP Config
***** MCP服务器配置
***** 连接参数
**** Prompt Config
***** 提示词模板
***** 版本管理
*** 执行数据
**** Plan Template
***** 执行计划模板
***** 步骤定义
**** Execution Context
***** 执行上下文
***** 状态数据
@endwbs
```

## 组件层次说明

### 🎯 架构分层概述

JManus 采用8层分层架构设计，每层职责明确，组件边界清晰：

1. **用户接口层** - 提供用户交互界面和API网关
2. **API控制器层** - 处理HTTP请求，实现RESTful接口
3. **业务服务层** - 封装业务逻辑，提供服务接口
4. **核心组件层** - 实现核心功能，包括规划、执行、智能体等
5. **工具生态层** - 提供各种工具集成，支持多模态操作
6. **技术支撑层** - 提供技术框架和接口抽象
7. **基础设施层** - 提供底层资源和外部服务
8. **数据层** - 管理系统数据和配置信息

### 🔧 核心组件特点

#### **智能体系统**
- **Dynamic Agent**: 支持运行时配置的智能体
- **Agent Loader**: 动态加载和管理智能体实例
- **Agent Service**: 提供智能体生命周期管理

#### **规划执行系统**
- **Plan Creator**: 基于LLM生成智能执行计划
- **Plan Executor**: 协调执行计划中的各个步骤
- **Planning Coordinator**: 统一协调规划和执行流程

#### **工具集成生态**
- **多工具支持**: 浏览器、Python、Bash、MCP等
- **统一接口**: 标准化工具调用和结果处理
- **可扩展性**: 支持新工具类型的快速集成

#### **数据管理体系**
- **配置管理**: 智能体、模型、提示词等配置
- **执行记录**: 完整的执行历史和状态跟踪
- **版本控制**: 支持配置和计划的版本管理

### 🚀 技术优势

1. **分层解耦**: 清晰的层次边界，便于维护和扩展
2. **组件化设计**: 高内聚低耦合的组件结构
3. **标准接口**: 统一的服务接口和数据格式
4. **可扩展性**: 支持新功能和工具的快速集成
5. **可观测性**: 完整的执行记录和状态监控

---

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**更新日期**: 2025年1月  
**建模工具**: PlantUML WBS