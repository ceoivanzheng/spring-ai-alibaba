# JManus 复合计划执行流程 - PlantUML时序图

## 概述

本文档使用PlantUML格式描述JManus系统处理复合计划"用浏览器基于百度，查询今天阿里巴巴的股价，并返回最新股价"的完整流程时序图。

## PlantUML时序图

```plantuml
@startuml JManus复合计划执行流程-股价查询用例
!theme plain
skinparam backgroundColor #FFFFFF
skinparam sequenceArrowThickness 2
skinparam roundcorner 20
skinparam maxmessagesize 60
skinparam ParticipantPadding 20

participant "用户" as User
participant "Vue3前端" as Frontend
participant "REST API层" as API
participant "PlanningCoordinator" as Coordinator
participant "PlanCreator" as Creator
participant "PlanExecutorFactory" as Factory
participant "PlanExecutor" as Executor
participant "DynamicAgent" as Agent
participant "BrowserUseTool" as Browser
participant "Chrome浏览器" as ChromeDriver
participant "百度搜索" as BaiduSearch
participant "LLM服务" as LLM
participant "数据库" as DB
participant "执行记录器" as Recorder

== 1. 用户输入与需求分析阶段 ==
User -> Frontend: 输入复合查询请求
note right of User
"用浏览器基于百度，查询今天
阿里巴巴的股价，并返回最新股价"
end note

Frontend -> API: POST /api/executor/execute
note right of API
ExecutionRequest {
  userRequest: "用浏览器基于百度，查询今天阿里巴巴的股价，并返回最新股价",
  agentName: "browser-agent",
  planType: "COMPLEX",
  maxSteps: 10
}
end note

== 2. 智能计划创建阶段 ==
API -> Coordinator: 接收执行请求
Coordinator -> Creator: 创建复合执行计划
Creator -> LLM: 分析复合任务需求
note right of LLM
任务分解：
1. 启动浏览器
2. 导航到百度
3. 搜索"阿里巴巴股价"
4. 解析搜索结果
5. 提取股价信息
6. 验证数据准确性
7. 整理返回结果
end note

LLM --> Creator: 返回多步骤计划
note right of Creator
ExecutionPlan {
  steps: 7个子任务,
  requiredTools: ["BrowserUseTool"],
  planType: COMPLEX
}
end note
Creator --> Coordinator: 返回ExecutionPlan

Coordinator -> Recorder: 记录计划创建
Recorder -> DB: 保存PlanExecutionRecord

== 3. 执行器初始化阶段 ==
Coordinator -> Factory: 创建复合任务执行器
Factory -> DB: 查询智能体配置
DB --> Factory: 返回带BrowserTool的AgentEntity
Factory -> Agent: 创建DynamicAgent实例
note right of Agent
配置BrowserUseTool
设置Chrome驱动路径
初始化浏览器参数
end note
Factory --> Coordinator: 返回PlanExecutor

== 4. 计划执行阶段 ==
Coordinator -> Executor: 开始执行复合计划

group Step 1: 浏览器启动与导航
    Executor -> Agent: 执行步骤1-浏览器启动
    Agent -> Agent: Think: 需要启动浏览器并导航到百度
    Agent -> Recorder: 记录思考过程
    Recorder -> DB: 保存ThinkActRecord(THINKING)
    
    Agent -> Browser: Act: 启动浏览器工具
    Browser -> ChromeDriver: 启动Chrome实例
    ChromeDriver --> Browser: 浏览器就绪
    Browser -> ChromeDriver: 导航到百度首页
    ChromeDriver -> BaiduSearch: GET https://www.baidu.com
    BaiduSearch --> ChromeDriver: 返回百度首页
    ChromeDriver --> Browser: 页面加载完成
    Browser --> Agent: 浏览器启动成功
    
    Agent -> Recorder: 记录行动结果
    Recorder -> DB: 更新ThinkActRecord(COMPLETED)
    Agent --> Executor: 步骤1完成
end

group Step 2-3: 搜索执行
    Executor -> Agent: 执行步骤2-搜索阿里巴巴股价
    Agent -> Agent: Think: 在百度搜索框输入关键词
    Agent -> Browser: Act: 执行搜索操作
    note right of Browser
    搜索参数 {
      query: "阿里巴巴股价 今日",
      waitForResults: true
    }
    end note
    Browser -> ChromeDriver: 定位搜索框元素
    ChromeDriver -> BaiduSearch: 输入搜索关键词
    Browser -> ChromeDriver: 点击搜索按钮
    ChromeDriver -> BaiduSearch: 提交搜索请求
    BaiduSearch --> ChromeDriver: 返回搜索结果页面
    ChromeDriver --> Browser: 搜索结果加载完成
    Browser --> Agent: 搜索执行成功
    Agent --> Executor: 步骤2完成
end

group Step 4-5: 结果解析与数据提取
    Executor -> Agent: 执行步骤3-解析搜索结果
    Agent -> Agent: Think: 需要分析页面内容，提取股价信息
    Agent -> Browser: Act: 获取页面内容
    Browser -> ChromeDriver: 提取页面HTML内容
    ChromeDriver --> Browser: 返回页面内容
    note right of Browser
    页面内容包含：
    - 股价数字
    - 涨跌幅
    - 更新时间
    - 相关财经信息
    end note
    Browser --> Agent: 返回结构化页面数据
    
    Agent -> LLM: 请求解析股价信息
    note right of LLM
    从HTML内容中识别：
    当前股价: $XXX.XX
    涨跌: +/-X.XX (+/-X.XX%)
    更新时间: YYYY-MM-DD HH:mm
    end note
    LLM --> Agent: 返回提取的股价数据
    note right of Agent
    StockPriceInfo {
      symbol: "BABA",
      currentPrice: "75.23",
      change: "+1.45",
      changePercent: "+1.96%",
      updateTime: "2025-01-15 15:30",
      currency: "USD"
    }
    end note
    
    Agent -> Recorder: 记录数据提取结果
    Recorder -> DB: 保存提取的股价数据
    Agent --> Executor: 步骤3完成
end

group Step 6-7: 数据验证与结果整理
    Executor -> Agent: 执行步骤4-验证数据准确性
    Agent -> Agent: Think: 验证股价数据的时效性和准确性
    Agent -> Browser: Act: 检查数据更新时间
    Browser --> Agent: 确认数据为最新
    
    Agent -> Agent: Think: 整理最终回答
    note right of Agent
    整理回答内容：
    "根据百度搜索结果，阿里巴巴(BABA)今日最新股价为$75.23，
    较昨日上涨$1.45(+1.96%)，数据更新时间：2025-01-15 15:30"
    end note
    
    Agent -> Recorder: 记录最终结果
    Recorder -> DB: 更新AgentExecutionRecord
    Agent --> Executor: 所有步骤完成
end

== 5. 清理与结果返回阶段 ==
Executor -> Browser: 清理浏览器资源
Browser -> ChromeDriver: 关闭浏览器实例
ChromeDriver --> Browser: 资源清理完成

Executor --> Coordinator: 执行完成
Coordinator -> Recorder: 最终化处理
Recorder -> DB: 更新PlanExecutionRecord状态
note right of Recorder
ExecutionResult {
  planId: "plan-20250115-002",
  status: "COMPLETED",
  result: "阿里巴巴(BABA)今日最新股价...",
  executionTime: "45秒",
  stepsCompleted: 7
}
end note

Coordinator --> API: 返回执行结果
API --> Frontend: HTTP 200 + ExecutionResult
Frontend --> User: 显示股价查询结果

@enduml
```

## 时序图说明

### 主要阶段
1. **用户输入与需求分析阶段**：复合任务输入，系统识别为复杂任务类型
2. **智能计划创建阶段**：LLM分解复合任务为7个子步骤
3. **执行器初始化阶段**：创建带浏览器工具的智能体实例
4. **计划执行阶段**：分组执行浏览器自动化操作
5. **清理与结果返回阶段**：资源清理和结果返回

### 关键特性
- **任务分解**：复合任务智能分解为可执行的子任务
- **工具集成**：BrowserUseTool与Chrome浏览器的深度集成
- **数据提取**：从网页内容中智能提取结构化股价信息
- **资源管理**：完整的浏览器生命周期管理

### 执行步骤详解
1. **浏览器启动与导航**：初始化Chrome实例并导航到百度
2. **搜索执行**：自动化搜索操作，输入关键词并提交
3. **结果解析与数据提取**：解析搜索结果，提取股价信息
4. **数据验证与结果整理**：验证数据准确性并格式化回答

### 预期输出
```
阿里巴巴股价查询结果

根据百度搜索的最新数据：
- 股票代码: BABA (纽约证券交易所)
- 当前股价: $75.23 USD
- 涨跌情况: +$1.45 (+1.96%)
- 数据更新时间: 2025年1月15日 15:30 (美东时间)

*注：股价数据来源于百度财经，实际交易请以官方数据为准。*
```

## 性能指标

- **总执行时间**：45-60秒
- **成功率**：>95% (网络条件良好时)
- **步骤数量**：7个主要执行步骤
- **工具使用**：BrowserUseTool + Chrome WebDriver

## 异常处理机制

### 网络异常
- 3次重试机制，指数退避策略
- 30秒超时自动终止
- 降级到其他搜索引擎

### 页面解析异常
- 多套CSS选择器备用策略
- 部分信息缺失时的容错处理
- 数据格式验证和范围检查

### 浏览器异常
- 自动重启浏览器实例
- WebDriver异常恢复机制
- 资源泄露防护

## 扩展能力

1. **多股票批量查询**：支持同时查询多只股票
2. **历史数据查询**：扩展支持历史股价和走势
3. **智能分析**：结合AI提供投资建议
4. **实时监控**：股价提醒和监控功能

## 使用说明

1. 将PlantUML代码复制到支持PlantUML的编辑器中
2. 或者使用在线PlantUML编辑器：http://www.plantuml.com/plantuml/
3. 可导出为PNG、SVG等格式用于文档展示

## 相关文档

- [JManus复合计划执行流程分析](manus-plan-dialog.md)
- [JManus简单对话处理流程](manus-simple-dialog.md)
- [JManus架构设计](manus-architecture.md)
- [JManus组件设计](manus-component.md)