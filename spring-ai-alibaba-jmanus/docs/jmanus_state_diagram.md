# JManus AI 智能助手平台 - 状态图 (State Diagram)

本文档展示 JManus AI 智能助手平台的状态图，表示对象生命周期状态及转换，用于复杂状态管理。

## 文档说明

**使用场景**: 表示对象生命周期状态及转换  
**应用阶段**: 复杂状态管理  
**关键优势**: 帮助跟踪状态变化、调试异常行为  

## 核心对象状态图

### 1. 智能体状态图

```plantuml
@startuml
!theme plain
skinparam state {
  BackgroundColor lightblue
  BorderColor darkblue
}

title JManus 智能体状态图

[*] --> NOT_STARTED : 创建智能体

state NOT_STARTED {
  NOT_STARTED : 状态：未开始
  NOT_STARTED : 描述：智能体已创建但未开始执行
  NOT_STARTED : 操作：可配置参数、工具绑定
}

state IN_PROGRESS {
  IN_PROGRESS : 状态：执行中
  IN_PROGRESS : 描述：智能体正在执行任务
  IN_PROGRESS : 操作：步骤执行、工具调用、状态更新
  
  state THINKING {
    THINKING : 子状态：思考中
    THINKING : 描述：分析任务需求
  }
  
  state ACTING {
    ACTING : 子状态：执行中
    ACTING : 描述：调用工具执行操作
  }
  
  state WAITING_INPUT {
    WAITING_INPUT : 子状态：等待输入
    WAITING_INPUT : 描述：等待用户提供信息
  }
  
  [*] --> THINKING
  THINKING --> ACTING : 制定行动计划
  ACTING --> THINKING : 需要重新思考
  THINKING --> WAITING_INPUT : 需要用户输入
  WAITING_INPUT --> THINKING : 收到用户输入
  ACTING --> WAITING_INPUT : 执行过程中需要输入
}

state COMPLETED {
  COMPLETED : 状态：已完成
  COMPLETED : 描述：任务成功执行完成
  COMPLETED : 操作：结果输出、资源清理
}

state BLOCKED {
  BLOCKED : 状态：阻塞
  BLOCKED : 描述：等待外部条件或用户干预
  BLOCKED : 操作：问题诊断、条件等待
}

state FAILED {
  FAILED : 状态：失败
  FAILED : 描述：执行过程中发生不可恢复错误
  FAILED : 操作：错误记录、资源清理
}

' 状态转换
NOT_STARTED --> IN_PROGRESS : start() / 初始化执行环境
IN_PROGRESS --> COMPLETED : 任务完成 / 生成结果
IN_PROGRESS --> BLOCKED : 遇到阻塞条件 / 暂停执行
IN_PROGRESS --> FAILED : 发生错误 / 记录错误信息

BLOCKED --> IN_PROGRESS : 条件满足 / 恢复执行
BLOCKED --> FAILED : 超时或无法恢复 / 终止执行

COMPLETED --> [*] : cleanup() / 清理资源
FAILED --> [*] : cleanup() / 清理资源

' 自转换
IN_PROGRESS --> IN_PROGRESS : stepExecution() / 执行下一步
BLOCKED --> BLOCKED : waitCondition() / 继续等待

note right of NOT_STARTED
  智能体创建后的初始状态
  可进行配置和参数设置
end note

note top of IN_PROGRESS
  智能体执行的核心状态
  包含思考、执行、等待输入等子状态
end note

note bottom of COMPLETED
  任务成功完成状态
  系统自动进行清理工作
end note

@enduml
```

### 2. 任务执行状态图

```plantuml
@startuml
!theme plain
title JManus 任务执行状态图

[*] --> SUBMITTED : 用户提交任务

state SUBMITTED {
  SUBMITTED : 状态：已提交
  SUBMITTED : 描述：任务已提交等待处理
  SUBMITTED : 数据：任务ID、用户需求、时间戳
}

state PLANNING {
  PLANNING : 状态：规划中
  PLANNING : 描述：AI正在生成执行计划
  PLANNING : 操作：需求分析、计划生成、智能体选择
}

state EXECUTING {
  EXECUTING : 状态：执行中
  EXECUTING : 描述：按计划执行任务步骤
  
  state SEQUENTIAL_EXECUTION {
    SEQUENTIAL_EXECUTION : 顺序执行模式
  }
  
  state PARALLEL_EXECUTION {
    PARALLEL_EXECUTION : 并行执行模式
    
    state MAP_PHASE {
      MAP_PHASE : Map阶段
    }
    
    state REDUCE_PHASE {
      REDUCE_PHASE : Reduce阶段
    }
    
    [*] --> MAP_PHASE
    MAP_PHASE --> REDUCE_PHASE : Map完成
    REDUCE_PHASE --> [*]
  }
  
  [*] --> SEQUENTIAL_EXECUTION
  SEQUENTIAL_EXECUTION --> PARALLEL_EXECUTION : 切换到并行模式
  PARALLEL_EXECUTION --> SEQUENTIAL_EXECUTION : 切换到顺序模式
}

state PAUSED {
  PAUSED : 状态：暂停
  PAUSED : 描述：执行暂停等待恢复
  PAUSED : 原因：用户暂停、系统维护、资源不足
}

state COMPLETED {
  COMPLETED : 状态：已完成
  COMPLETED : 描述：任务成功执行完成
  COMPLETED : 结果：执行结果、耗时统计、资源使用
}

state FAILED {
  FAILED : 状态：执行失败
  FAILED : 描述：任务执行过程中失败
  FAILED : 信息：错误原因、失败步骤、诊断信息
}

state CANCELLED {
  CANCELLED : 状态：已取消
  CANCELLED : 描述：用户主动取消任务
  CANCELLED : 操作：资源清理、状态回滚
}

' 主要状态转换
SUBMITTED --> PLANNING : validate() / 验证任务有效性
PLANNING --> EXECUTING : planGenerated() / 计划生成完成
EXECUTING --> COMPLETED : allStepsCompleted() / 所有步骤完成
EXECUTING --> FAILED : executionError() / 执行错误
EXECUTING --> PAUSED : pause() / 暂停请求
PAUSED --> EXECUTING : resume() / 恢复执行
PAUSED --> CANCELLED : cancel() / 取消任务

' 异常转换
PLANNING --> FAILED : planningError() / 计划生成失败
SUBMITTED --> CANCELLED : cancel() / 提交后取消
ANY_STATE --> CANCELLED : forceCancel() / 强制取消

COMPLETED --> [*] : archive() / 归档任务
FAILED --> [*] : cleanup() / 清理资源
CANCELLED --> [*] : cleanup() / 清理资源

@enduml
```

### 3. MCP连接状态图

```plantuml
@startuml
!theme plain
title JManus MCP连接状态图

[*] --> DISCONNECTED : 系统启动

state DISCONNECTED {
  DISCONNECTED : 状态：断开连接
  DISCONNECTED : 描述：与MCP服务器无连接
  DISCONNECTED : 操作：可配置连接参数
}

state CONNECTING {
  CONNECTING : 状态：连接中
  CONNECTING : 描述：正在建立MCP连接
  CONNECTING : 超时：30秒连接超时
}

state CONNECTED {
  CONNECTED : 状态：已连接
  CONNECTED : 描述：MCP连接已建立
  
  state IDLE {
    IDLE : 空闲状态
    IDLE : 等待工具调用
  }
  
  state CALLING {
    CALLING : 调用中
    CALLING : 正在执行MCP工具
  }
  
  state CACHING {
    CACHING : 缓存中
    CACHING : 正在缓存调用结果
  }
  
  [*] --> IDLE
  IDLE --> CALLING : toolCall() / 调用工具
  CALLING --> IDLE : callCompleted() / 调用完成
  CALLING --> CACHING : enableCache() / 启用缓存
  CACHING --> IDLE : cacheStored() / 缓存保存
}

state RECONNECTING {
  RECONNECTING : 状态：重连中
  RECONNECTING : 描述：连接断开后尝试重连
  RECONNECTING : 策略：指数退避重试
}

state ERROR {
  ERROR : 状态：错误
  ERROR : 描述：连接或调用发生错误
  ERROR : 操作：错误诊断、日志记录
}

' 状态转换
DISCONNECTED --> CONNECTING : connect() / 开始连接
CONNECTING --> CONNECTED : connectionEstablished() / 连接成功
CONNECTING --> ERROR : connectionFailed() / 连接失败
CONNECTING --> DISCONNECTED : timeout() / 连接超时

CONNECTED --> DISCONNECTED : disconnect() / 主动断开
CONNECTED --> RECONNECTING : connectionLost() / 连接丢失
CONNECTED --> ERROR : protocolError() / 协议错误

RECONNECTING --> CONNECTED : reconnectSuccess() / 重连成功
RECONNECTING --> ERROR : reconnectFailed() / 重连失败
RECONNECTING --> DISCONNECTED : giveUp() / 放弃重连

ERROR --> DISCONNECTED : reset() / 重置连接
ERROR --> RECONNECTING : retry() / 重试连接

note right of CONNECTED
  MCP连接的正常工作状态
  支持工具调用和结果缓存
end note

note left of RECONNECTING
  自动重连机制
  采用指数退避策略
  最多重试5次
end note

@enduml
```

### 4. 配置管理状态图

```plantuml
@startuml
!theme plain
title JManus 配置管理状态图

[*] --> DEFAULT : 系统初始化

state DEFAULT {
  DEFAULT : 状态：默认配置
  DEFAULT : 描述：使用系统默认配置
  DEFAULT : 来源：配置文件、环境变量
}

state LOADING {
  LOADING : 状态：加载中
  LOADING : 描述：正在加载配置
  LOADING : 操作：文件读取、格式验证
}

state ACTIVE {
  ACTIVE : 状态：配置生效
  ACTIVE : 描述：配置已加载并生效
  
  state MONITORING {
    MONITORING : 监控配置变化
  }
  
  state APPLYING {
    APPLYING : 应用配置更改
  }
  
  [*] --> MONITORING
  MONITORING --> APPLYING : configChanged() / 检测到变化
  APPLYING --> MONITORING : applyCompleted() / 应用完成
}

state VALIDATING {
  VALIDATING : 状态：验证中
  VALIDATING : 描述：验证配置有效性
  VALIDATING : 检查：格式、依赖、约束
}

state INVALID {
  INVALID : 状态：配置无效
  INVALID : 描述：配置验证失败
  INVALID : 操作：错误报告、回滚准备
}

state BACKING_UP {
  BACKING_UP : 状态：备份中
  BACKING_UP : 描述：备份当前配置
  BACKING_UP : 目的：为回滚做准备
}

state ROLLBACK {
  ROLLBACK : 状态：回滚中
  ROLLBACK : 描述：恢复到之前配置
  ROLLBACK : 触发：配置错误、用户请求
}

' 配置加载流程
DEFAULT --> LOADING : loadConfig() / 加载自定义配置
LOADING --> VALIDATING : configLoaded() / 配置读取完成
VALIDATING --> ACTIVE : validationPassed() / 验证通过
VALIDATING --> INVALID : validationFailed() / 验证失败

' 配置更新流程
ACTIVE --> BACKING_UP : updateRequest() / 更新请求
BACKING_UP --> LOADING : backupCompleted() / 备份完成
INVALID --> ROLLBACK : rollbackTriggered() / 触发回滚
ROLLBACK --> ACTIVE : rollbackCompleted() / 回滚完成

' 错误处理
LOADING --> DEFAULT : loadingError() / 加载失败
INVALID --> DEFAULT : useDefault() / 使用默认配置

@enduml
```

### 5. 用户会话状态图

```plantuml
@startuml
!theme plain
title JManus 用户会话状态图

[*] --> ANONYMOUS : 用户访问

state ANONYMOUS {
  ANONYMOUS : 状态：匿名用户
  ANONYMOUS : 描述：未认证用户
  ANONYMOUS : 权限：只读访问、基本功能
}

state AUTHENTICATING {
  AUTHENTICATING : 状态：认证中
  AUTHENTICATING : 描述：正在验证用户身份
  AUTHENTICATING : 方式：API密钥、Token验证
}

state AUTHENTICATED {
  AUTHENTICATED : 状态：已认证
  AUTHENTICATED : 描述：用户身份已验证
  
  state NORMAL_USER {
    NORMAL_USER : 普通用户权限
    NORMAL_USER : 任务提交、状态查看
  }
  
  state ADMIN_USER {
    ADMIN_USER : 管理员权限
    ADMIN_USER : 系统配置、用户管理
  }
  
  [*] --> NORMAL_USER
  NORMAL_USER --> ADMIN_USER : promoteToAdmin() / 提升权限
  ADMIN_USER --> NORMAL_USER : demoteToUser() / 降低权限
}

state ACTIVE_SESSION {
  ACTIVE_SESSION : 状态：活跃会话
  ACTIVE_SESSION : 描述：用户正在使用系统
  ACTIVE_SESSION : 监控：操作日志、活跃时间
}

state IDLE {
  IDLE : 状态：空闲
  IDLE : 描述：用户暂时无操作
  IDLE : 超时：30分钟无操作
}

state EXPIRED {
  EXPIRED : 状态：会话过期
  EXPIRED : 描述：会话超时失效
  EXPIRED : 清理：会话数据、临时文件
}

state LOCKED {
  LOCKED : 状态：账户锁定
  LOCKED : 描述：因安全原因锁定
  LOCKED : 原因：多次失败、异常行为
}

' 认证流程
ANONYMOUS --> AUTHENTICATING : login() / 开始认证
AUTHENTICATING --> AUTHENTICATED : authSuccess() / 认证成功
AUTHENTICATING --> ANONYMOUS : authFailed() / 认证失败

' 会话管理
AUTHENTICATED --> ACTIVE_SESSION : startSession() / 开始会话
ACTIVE_SESSION --> IDLE : noActivity() / 无活动
IDLE --> ACTIVE_SESSION : userAction() / 用户操作
IDLE --> EXPIRED : timeout() / 超时

' 安全控制
AUTHENTICATING --> LOCKED : tooManyAttempts() / 尝试次数过多
ACTIVE_SESSION --> LOCKED : suspiciousActivity() / 可疑活动
LOCKED --> ANONYMOUS : unlock() / 解锁账户

' 会话结束
ACTIVE_SESSION --> ANONYMOUS : logout() / 用户登出
EXPIRED --> ANONYMOUS : cleanup() / 清理过期会话
LOCKED --> ANONYMOUS : sessionTerminated() / 会话终止

@enduml
```

## 复合状态和并发状态

### 系统整体状态图

```plantuml
@startuml
!theme plain
title JManus 系统整体状态图

[*] --> INITIALIZING : 系统启动

state INITIALIZING {
  INITIALIZING : 状态：初始化中
  INITIALIZING : 描述：系统组件启动
  INITIALIZING : 操作：配置加载、服务启动、数据库连接
}

state RUNNING {
  RUNNING : 状态：运行中
  RUNNING : 描述：系统正常运行
  
  -- 并发区域：核心服务 --
  state TASK_SERVICE {
    TASK_SERVICE : 任务服务
    [*] --> IDLE_TASK
    IDLE_TASK --> PROCESSING_TASK : 接收任务
    PROCESSING_TASK --> IDLE_TASK : 任务完成
  }
  
  state AGENT_SERVICE {
    AGENT_SERVICE : 智能体服务
    [*] --> READY
    READY --> EXECUTING : 智能体执行
    EXECUTING --> READY : 执行完成
  }
  
  state MCP_SERVICE {
    MCP_SERVICE : MCP服务
    [*] --> CONNECTED_MCP
    CONNECTED_MCP --> CALLING_MCP : 工具调用
    CALLING_MCP --> CONNECTED_MCP : 调用完成
  }
  --
}

state MAINTENANCE {
  MAINTENANCE : 状态：维护模式
  MAINTENANCE : 描述：系统维护中
  MAINTENANCE : 操作：数据备份、系统更新、配置调整
}

state DEGRADED {
  DEGRADED : 状态：降级运行
  DEGRADED : 描述：部分功能不可用
  DEGRADED : 原因：服务故障、资源不足
}

state ERROR {
  ERROR : 状态：系统错误
  ERROR : 描述：系统发生严重错误
  ERROR : 操作：错误诊断、紧急修复
}

' 状态转换
INITIALIZING --> RUNNING : initCompleted() / 初始化完成
RUNNING --> MAINTENANCE : maintenanceMode() / 进入维护
RUNNING --> DEGRADED : serviceFailure() / 服务故障
RUNNING --> ERROR : criticalError() / 严重错误

MAINTENANCE --> RUNNING : maintenanceCompleted() / 维护完成
DEGRADED --> RUNNING : serviceRestored() / 服务恢复
ERROR --> RUNNING : errorResolved() / 错误解决

DEGRADED --> ERROR : cascadingFailure() / 级联故障
ERROR --> DEGRADED : partialRecovery() / 部分恢复

@enduml
```

## 状态转换触发条件

### 关键状态转换说明

1. **智能体状态转换**
   - `NOT_STARTED → IN_PROGRESS`: 调用 `start()` 方法
   - `IN_PROGRESS → COMPLETED`: 所有任务步骤完成
   - `IN_PROGRESS → FAILED`: 遇到不可恢复的错误
   - `BLOCKED → IN_PROGRESS`: 阻塞条件解除

2. **任务执行状态转换**
   - `SUBMITTED → PLANNING`: 任务验证通过
   - `PLANNING → EXECUTING`: 执行计划生成完成
   - `EXECUTING → PAUSED`: 用户暂停或系统需要
   - `PAUSED → EXECUTING`: 用户恢复或条件满足

3. **MCP连接状态转换**
   - `DISCONNECTED → CONNECTING`: 调用连接方法
   - `CONNECTING → CONNECTED`: 握手成功
   - `CONNECTED → RECONNECTING`: 连接丢失
   - `RECONNECTING → CONNECTED`: 重连成功

## 状态监控和调试

### 状态变化日志记录
- 每次状态转换都会记录详细日志
- 包含转换时间、触发条件、上下文信息
- 支持状态变化的回放和分析

### 异常状态检测
- 监控状态转换的合法性
- 检测长时间停留在某状态的情况
- 自动触发状态恢复机制

### 性能指标统计
- 各状态的平均停留时间
- 状态转换的频率统计
- 异常状态的发生率分析

---

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**状态图数量**: 6个核心状态图  
**涵盖对象**: 智能体、任务、MCP连接、配置、用户会话、系统整体  
**建模工具**: PlantUML