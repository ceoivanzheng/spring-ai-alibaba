# JManus 项目 Claude 对话提示词记录

本文档记录了在 JManus 项目开发过程中使用的 Claude 对话提示词。

## 架构图相关提示词

### 1. 架构图修改和细化
```
修改文档，还是参考下面代码，生成jmanus的详细架构图
@startuml
[详细的 PlantUML 架构图代码示例]
```

### 2. 架构图布局优化
```
很好，这个图元素和逻辑比较清晰了，布局看起来不太美观，调整一下布局，整体对称布局，有呼吸感
```

## 代码理解和组件分析

### 3. 核心组件理解
```
理解工程代码，增加核心组件
```

## 文档生成相关提示词

### 4. 组件分解结构图生成
```
生成manus-component.md
理解工程中用到的核心组件，用plantuml的wbs工作分解结构图来描述Jmanus的核心组件，分层分解关系；如
@startwbs
* Business Process Modelling WBS
** Launch the project
*** Complete Stakeholder Research
*** Initial Implementation Plan
** Design phase
*** Model of AsIs Processes Completed
**** Model of AsIs Processes Completed1
**** Model of AsIs Processes Completed2
*** Measure AsIs performance metrics
*** Identify Quick Wins
** Complete innovate phase
@endwbs
```

### 5. 对话记录整理
```
回顾我所有对话，将我输入的提示词保存到manus-mine-claude-prompt.md
```

## 提示词特点分析

### 架构图相关
- **参考代码示例**：提供具体的 PlantUML 代码作为参考模板
- **布局美化**：关注图表的视觉效果，要求对称布局和呼吸感
- **细化需求**：从简单到详细的逐步完善

### 代码分析相关
- **简洁明确**：直接表达需要理解工程代码的核心组件
- **实际驱动**：基于真实代码结构进行分析

### 文档生成相关
- **格式规范**：明确指定使用的图表类型（WBS、PlantUML等）
- **示例引导**：提供具体的格式示例帮助理解需求
- **结构化思维**：要求按分层分解关系组织内容

---

**记录时间**: 2025年1月  
**项目**: JManus AI 智能助手平台  
**用途**: Claude 对话提示词参考和复用