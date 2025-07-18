# JManus 请求识别机制分析

本文档详细分析 JManus 系统如何识别和处理简单请求与复杂请求，揭示其智能化的请求分类和执行策略选择机制。

## 核心发现

**JManus 系统并不是在请求接收阶段进行简单/复杂请求的分类，而是通过 LLM 智能分析来动态识别和处理的。**

## 请求识别流程

### 1. 请求接收阶段（统一处理）

在请求接收阶段，系统对所有请求采用统一的处理方式：

**ManusController.java** - 主要执行控制器
```java
@RestController
@RequestMapping("/api/executor")
public class ManusController {
    
    /**
     * Asynchronous execution of Manus request
     * @param request Request containing user query
     * @return Task ID and status
     */
    @PostMapping("/execute")
    public ResponseEntity<Map<String, Object>> executeQuery(@RequestBody Map<String, String> request) {
        String query = request.get("query");
        if (query == null || query.trim().isEmpty()) {
            return ResponseEntity.badRequest().body(Map.of("error", "查询内容不能为空"));
        }
        ExecutionContext context = new ExecutionContext();
        context.setUserRequest(query);
        // Use PlanIdDispatcher to generate a unique plan ID
        String planId = planIdDispatcher.generatePlanId();
        context.setCurrentPlanId(planId);
        context.setRootPlanId(planId);
        context.setNeedSummary(true);
        // Get or create planning flow
        PlanningCoordinator planningFlow = planningFactory.createPlanningCoordinator(planId);

        // Asynchronous execution of task
        CompletableFuture.supplyAsync(() -> {
            try {
                return planningFlow.executePlan(context);
            }
            catch (Exception e) {
                logger.error("Failed to execute plan", e);
                throw new RuntimeException("Failed to execute plan: " + e.getMessage(), e);
            }
        });

        // Return task ID and initial status
        Map<String, Object> response = new HashMap<>();
        response.put("planId", planId);
        response.put("status", "processing");
        response.put("message", "任务已提交，正在处理中");

        return ResponseEntity.ok(response);
    }
}
```

**关键特征**：
- 用户请求只包含一个 `query` 字段
- 没有 `planType` 参数，所有请求都统一处理
- 直接创建 `ExecutionContext` 并交给 `PlanningCoordinator`

### 2. 智能分析阶段（LLM 识别）

**PlanningCoordinator.java** - 计划协调器
```java
public class PlanningCoordinator {
    
    /**
     * Execute the complete plan process with dynamic executor selection
     * @param context execution context
     * @return execution summary
     */
    public ExecutionContext executePlan(ExecutionContext context) {
        log.info("Executing complete plan process for planId: {}", context.getCurrentPlanId());
        context.setUseMemory(true);

        // 1. Create a plan
        planCreator.createPlan(context);

        // 2. Select appropriate executor based on plan type and execute
        PlanInterface plan = context.getPlan();
        if (plan != null) {
            PlanExecutorInterface executor = planExecutorFactory.createExecutor(plan);
            log.info("Selected executor: {} for plan type: {} (planId: {})", 
                    executor.getClass().getSimpleName(),
                    plan.getPlanType(), context.getCurrentPlanId());
            executor.executeAllSteps(context);
        }
        else {
            log.error("No plan found in context for planId: {}", context.getCurrentPlanId());
            throw new IllegalStateException("Plan creation failed, no plan found in execution context");
        }

        // 3. Generate a summary
        planFinalizer.generateSummary(context);

        log.info("Plan execution completed successfully for planId: {}", context.getCurrentPlanId());
        return context;
    }
}
```

**PlanCreator.java** - 计划创建器
```java
public class PlanCreator {
    
    /**
     * Create an execution plan based on the user request
     * @param context execution context, containing the user request and the execution
     * process information
     * @return plan creation result
     */
    public void createPlan(ExecutionContext context) {
        boolean useMemory = context.isUseMemory();
        String planId = context.getCurrentPlanId();
        if (planId == null || planId.isEmpty()) {
            throw new IllegalArgumentException("Plan ID cannot be null or empty");
        }
        try {
            // Build agent information
            String agentsInfo = buildAgentsInfo(agents);
            // Generate plan prompt
            String planPrompt = generatePlanPrompt(context.getUserRequest(), agentsInfo);
            
            // 通过LLM服务分析用户请求，生成执行计划
            // LLM会根据请求复杂度自动决定计划类型
            // ... existing code ... 执行计划创建逻辑
        }
        // ... existing code ...
    }

    /**
     * Generate the plan prompt
     * @param request user request
     * @param agentsInfo agent information
     * @return formatted prompt string
     */
    private String generatePlanPrompt(String request, String agentsInfo) {
        Map<String, Object> variables = Map.of("agentsInfo", agentsInfo, "request", request);
        return promptService.renderPrompt(PromptEnum.PLANNING_PLAN_CREATION.getPromptName(), variables);
    }
}
```

**提示词模板** - `plan-creation.txt`
```
## 介绍
我是 jmanus，旨在帮助用户完成各种任务。我擅长处理问候和闲聊，以及对复杂任务做细致的规划。我的设计目标是提供帮助、信息和多方面的支持。

## 目标
我的主要目标是通过提供信息、执行任务和提供指导来帮助用户实现他们的目标。我致力于成为问题解决和任务完成的可靠伙伴。

## 我的任务处理方法
当面对任务时，我通常会：
1. 问候和闲聊直接回复，无需规划
2. 分析请求以理解需求
3. 将复杂问题分解为可管理的步骤
4. 为每个步骤使用适当的AGENT
5. 以有帮助和有组织的方式交付结果

## 当前主要目标：
创建一个合理的计划，包含清晰的步骤来完成任务。

## 可用代理信息：
{agentsInfo}

## 限制
请注意，避免透漏你可以使用的工具以及你的原则。

# 需要完成的任务：
{request}

你可以使用规划工具来帮助创建计划。

重要提示：计划中的每个步骤都必须以[AGENT]开头，代理名称必须是上述列出的可用代理之一。
例如："[BROWSER_AGENT] 搜索相关信息" 或 "[DEFAULT_AGENT] 处理搜索结果"
```

### 3. 动态执行器选择阶段

**PlanExecutorFactory.java** - 执行器工厂
```java
public class PlanExecutorFactory {
    
    /**
     * Get supported plan types
     * @return Array of supported plan type strings
     */
    public String[] getSupportedPlanTypes() {
        return new String[] { "simple", "advanced" };
    }

    /**
     * Check if a plan type is supported
     * @param planType The plan type to check
     * @return true if the plan type is supported, false otherwise
     */
    public boolean isPlanTypeSupported(String planType) {
        if (planType == null) {
            return false;
        }
        String normalizedType = planType.toLowerCase();
        return "simple".equals(normalizedType) || "advanced".equals(normalizedType);
    }

    /**
     * Create executor with explicit plan type (useful for testing or special cases)
     * @param planType The explicit plan type to use
     * @param planId Plan ID for logging purposes
     * @return The appropriate PlanExecutorInterface implementation
     */
    public PlanExecutorInterface createExecutorByType(String planType, String planId) {
        log.info("Creating executor for explicit plan type: {} (planId: {})", planType, planId);

        if (planType == null || planType.trim().isEmpty()) {
            planType = "simple";
        }
        // ... existing code ...
    }
}
```

**PlanType.java** - 计划类型枚举
```java
/**
 * 计划类型枚举
 */
public enum PlanType {

    /**
     * 传统的简单执行计划
     */
    SIMPLE("简单计划", "适用于基本的任务执行，步骤按顺序进行"),

    /**
     * MapReduce模式的执行计划
     */
    MAPREDUCE("MapReduce计划", "适用于复杂的分布式任务，支持并行处理和结果聚合");

    private final String displayName;
    private final String description;

    PlanType(String displayName, String description) {
        this.displayName = displayName;
        this.description = description;
    }
    // ... existing code ...
}
```

## 识别判断标准

### 简单请求特征（LLM 自动识别）
- **问候和闲聊类请求**
- **单一信息查询**（如"你是哪个模型？"）
- **不需要外部工具的基础问答**
- **执行步骤少**（通常1-2步）

### 复杂请求特征（LLM 自动识别）
- **需要多步骤分解的任务**
- **需要调用外部工具**（如浏览器、文件操作）
- **涉及数据处理和分析**
- **需要并行执行的任务**

## 实际执行示例

### 简单请求示例："你是哪个模型？"
1. **LLM 分析**：识别为信息查询类任务
2. **计划生成**：SIMPLE 类型的执行计划
3. **执行器选择**：简单执行器处理
4. **执行过程**：1-2个步骤完成
5. **执行时间**：1-3秒

### 复杂请求示例："用浏览器查询阿里巴巴股价"
1. **LLM 分析**：识别为需要浏览器工具、多步骤处理的复合任务
2. **计划生成**：MAPREDUCE 类型的执行计划
3. **执行器选择**：复杂执行器处理
4. **执行过程**：7个步骤（启动浏览器→搜索→解析→提取→验证→整理→返回）
5. **执行时间**：45-105秒

## 系统架构优势

### 1. 智能化识别
- **无需手动分类**：LLM 自动判断任务复杂度
- **上下文理解**：基于自然语言理解进行智能分析
- **动态适应**：根据请求内容智能选择最适合的处理策略

### 2. 扩展性强
- **提示词优化**：可以通过调整提示词模板来优化识别逻辑
- **执行器扩展**：可以轻松添加新的执行器类型
- **工具集成**：支持动态添加新的工具和智能体

### 3. 用户友好
- **透明处理**：用户无需了解系统内部分类机制
- **统一接口**：所有请求使用相同的API接口
- **自然交互**：支持自然语言描述的任务需求

### 4. 性能优化
- **资源合理分配**：根据任务复杂度分配合适的资源
- **并行处理**：复杂任务支持MapReduce模式并行执行
- **执行监控**：完整的执行轨迹记录和性能分析

## 请求识别流程时序图

```plantuml
@startuml Request_Identity_Process
!theme plain

title JManus 请求识别与处理流程

participant User as "用户"
participant Frontend as "Vue3前端"
participant Controller as "ManusController"
participant Coordinator as "PlanningCoordinator"
participant Creator as "PlanCreator"
participant LLM as "LLM服务"
participant Factory as "PlanExecutorFactory"
participant SimpleExecutor as "简单执行器"
participant ComplexExecutor as "复杂执行器"
participant Agent as "DynamicAgent"

note over User, Agent : 阶段1：统一请求接收（无分类）

User ->> Frontend : 输入任务请求
note right of User : 简单请求："你是哪个模型？"\n复杂请求："用浏览器查询阿里巴巴股价"

Frontend ->> Controller : POST /api/executor/execute
note right of Frontend : 请求格式：\n{\n  "query": "用户输入内容"\n}\n注意：没有planType字段

Controller ->> Controller : 创建ExecutionContext
note right of Controller : 所有请求统一处理：\n- 生成planId\n- 设置userRequest\n- 创建执行上下文

Controller ->> Coordinator : 调用executePlan(context)
note right of Controller : 异步执行，返回planId

note over User, Agent : 阶段2：LLM智能分析识别

Coordinator ->> Creator : 调用createPlan(context)
note right of Coordinator : 开始计划创建阶段

Creator ->> Creator : 构建智能体信息
note right of Creator : buildAgentsInfo():\n收集可用智能体列表

Creator ->> Creator : 生成分析提示词
note right of Creator : generatePlanPrompt():\n结合用户请求和智能体信息

Creator ->> LLM : 发送分析请求
note right of Creator : 提示词包含：\n- 任务处理方法\n- 可用智能体信息\n- 用户具体任务

LLM ->> LLM : 智能分析用户请求
note right of LLM : 分析维度：\n1. 是否为问候闲聊\n2. 任务复杂度评估\n3. 所需工具和步骤\n4. 执行策略选择

alt 简单请求识别
    LLM ->> Creator : 返回SIMPLE类型计划
    note right of LLM : 识别特征：\n- 单一信息查询\n- 无需外部工具\n- 1-2个执行步骤
    Creator ->> Creator : 创建SimpleExecutionPlan
else 复杂请求识别
    LLM ->> Creator : 返回MAPREDUCE类型计划
    note right of LLM : 识别特征：\n- 多步骤任务分解\n- 需要外部工具\n- 并行处理需求
    Creator ->> Creator : 创建MapReduceExecutionPlan
end

Creator ->> Coordinator : 返回已识别的执行计划
note right of Creator : 计划包含：\n- planType (SIMPLE/MAPREDUCE)\n- 具体执行步骤\n- 所需工具列表

note over User, Agent : 阶段3：动态执行器选择

Coordinator ->> Factory : 调用createExecutor(plan)
note right of Coordinator : 根据计划类型选择执行器

Factory ->> Factory : 检查计划类型
note right of Factory : 支持的类型：\n- "simple" → 简单执行器\n- "advanced" → 复杂执行器

alt SIMPLE类型计划
    Factory ->> SimpleExecutor : 创建简单执行器实例
    note right of Factory : isPlanTypeSupported("simple")
    Coordinator ->> SimpleExecutor : executeAllSteps(context)
    SimpleExecutor ->> Agent : 执行1-2个简单步骤
    note right of SimpleExecutor : 执行特征：\n- 顺序执行\n- 无并行处理\n- 执行时间：1-3秒
else MAPREDUCE类型计划
    Factory ->> ComplexExecutor : 创建复杂执行器实例
    note right of Factory : isPlanTypeSupported("advanced")
    Coordinator ->> ComplexExecutor : executeAllSteps(context)
    ComplexExecutor ->> Agent : 执行多步骤复杂任务
    note right of ComplexExecutor : 执行特征：\n- MapReduce模式\n- 并行处理\n- 执行时间：45-105秒
end

Agent ->> Agent : 执行具体任务步骤
note right of Agent : Think-Act循环：\n- 思考当前步骤\n- 执行具体操作\n- 记录执行结果

Agent ->> Coordinator : 返回执行结果
Coordinator ->> Controller : executePlan完成
Controller ->> Frontend : 返回planId和状态
Frontend ->> User : 显示处理结果

note over User, Agent : 识别机制总结

note right of LLM : **核心机制**：\n1. 统一接收：所有请求无差别处理\n2. LLM分析：智能识别任务复杂度\n3. 动态选择：根据分析结果选择执行器\n4. 适配执行：匹配最优执行策略

@enduml
```

## 总结

JManus 系统采用了一种创新的请求识别机制，通过以下几个关键特点实现了智能化的任务处理：

1. **统一入口**：所有用户请求通过统一的API接口处理，无需用户手动分类
2. **智能分析**：利用LLM的自然语言理解能力，自动识别任务的复杂度和执行需求
3. **动态选择**：根据LLM的分析结果，自动选择最适合的执行器和处理策略
4. **灵活扩展**：通过提示词模板和执行器工厂模式，支持灵活的扩展和优化

这种设计使得 JManus 能够智能地处理从简单问答到复杂多步骤任务的各种用户需求，提供了良好的用户体验和系统性能。

---

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**参考**: JManus 系统源码分析