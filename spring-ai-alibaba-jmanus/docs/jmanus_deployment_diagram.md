# JManus AI 智能助手平台 - 部署图 (Deployment Diagram)

本文档展示 JManus AI 智能助手平台的部署图，描述物理节点与软件部署拓扑，用于架构设计与运维。

## 文档说明

**使用场景**: 描述物理节点与软件部署拓扑  
**应用阶段**: 架构设计与运维  
**关键优势**: 展示基础设施和网络布局  

## 生产环境部署图

### 1. 微服务集群部署

```plantuml
@startuml
!theme plain
skinparam node {
  BackgroundColor lightblue
  BorderColor darkblue
}
skinparam artifact {
  BackgroundColor lightyellow
  BorderColor orange
}
skinparam cloud {
  BackgroundColor lightgreen
  BorderColor darkgreen
}

title JManus 生产环境微服务部署图

cloud "Internet" {
  [用户] as Users
  [外部API客户端] as APIClients
}

node "Load Balancer Cluster" as LBCluster {
  artifact "Nginx 1" as Nginx1
  artifact "Nginx 2" as Nginx2
  artifact "HAProxy" as HAProxy
}

node "Web Server Cluster" as WebCluster {
  artifact "Vue3 App 1" as Vue1
  artifact "Vue3 App 2" as Vue2
  artifact "Static CDN" as CDN
}

node "API Gateway Cluster" as GatewayCluster {
  artifact "Gateway Instance 1" as GW1
  artifact "Gateway Instance 2" as GW2
  artifact "Gateway Instance 3" as GW3
}

node "Application Server Cluster" as AppCluster {
  node "JManus Core Nodes" as CoreNodes {
    artifact "Core Service 1" as Core1
    artifact "Core Service 2" as Core2
    artifact "Core Service 3" as Core3
  }
  
  node "Microservice Nodes" as MicroNodes {
    artifact "Agent Service" as AgentSvc
    artifact "Model Service" as ModelSvc
    artifact "MCP Service" as McpSvc
    artifact "Tool Service" as ToolSvc
  }
}

node "Cache Cluster" as CacheCluster {
  artifact "Redis Master" as RedisMaster
  artifact "Redis Slave 1" as RedisSlave1
  artifact "Redis Slave 2" as RedisSlave2
  artifact "Redis Sentinel" as RedisSentinel
}

node "Database Cluster" as DBCluster {
  artifact "PostgreSQL Primary" as PGPrimary
  artifact "PostgreSQL Standby 1" as PGStandby1
  artifact "PostgreSQL Standby 2" as PGStandby2
  artifact "PgPool-II" as PgPool
}

node "Message Queue Cluster" as MQCluster {
  artifact "RabbitMQ Node 1" as MQ1
  artifact "RabbitMQ Node 2" as MQ2
  artifact "RabbitMQ Node 3" as MQ3
}

node "Monitoring & Logging" as MonitorCluster {
  artifact "Prometheus" as Prometheus
  artifact "Grafana" as Grafana
  artifact "ELK Stack" as ELK
  artifact "Jaeger" as Jaeger
}

cloud "External Services" as ExtServices {
  artifact "DashScope API" as DashScope
  artifact "OpenAI API" as OpenAI
  artifact "MCP Servers" as McpServers
}

cloud "Tool Infrastructure" as ToolInfra {
  artifact "Browser Grid" as BrowserGrid
  artifact "Python Cluster" as PythonCluster
  artifact "Container Registry" as Registry
}

' 网络连接
Users --> LBCluster : HTTPS/443
APIClients --> LBCluster : HTTPS/443

LBCluster --> WebCluster : HTTP/80
LBCluster --> GatewayCluster : HTTP/8080

GatewayCluster --> AppCluster : HTTP/8080
AppCluster --> CacheCluster : TCP/6379
AppCluster --> DBCluster : TCP/5432
AppCluster --> MQCluster : TCP/5672

AppCluster --> ExtServices : HTTPS/443
AppCluster --> ToolInfra : HTTP/Various

MonitorCluster --> AppCluster : Metrics Collection
MonitorCluster --> CacheCluster : Monitoring
MonitorCluster --> DBCluster : Monitoring

@enduml
```

### 2. 单体应用部署

```plantuml
@startuml
!theme plain
title JManus 单体应用部署图

node "Application Server" as AppServer {
  artifact "JManus Spring Boot App" as JManusApp {
    component "Web Layer" as WebLayer
    component "API Layer" as APILayer
    component "Service Layer" as ServiceLayer
    component "Core Engine" as CoreEngine
    component "Tool Integrations" as ToolIntegrations
  }
  
  artifact "Embedded H2 Database" as H2DB
  artifact "Local Cache" as LocalCache
}

node "External Dependencies" as ExtDeps {
  artifact "Chrome Browser" as Chrome
  artifact "Python Runtime" as Python
  artifact "Node.js Runtime" as NodeJS
}

cloud "AI Services" as AIServices {
  artifact "DashScope" as DashScope
  artifact "OpenAI" as OpenAI
  artifact "Claude API" as Claude
}

cloud "MCP Ecosystem" as MCPEco {
  artifact "MCP Server 1" as MCP1
  artifact "MCP Server 2" as MCP2
  artifact "Custom MCP Tools" as CustomMCP
}

node "Monitoring" as Monitoring {
  artifact "Application Logs" as Logs
  artifact "JVM Metrics" as JVMMetrics
  artifact "Health Check" as HealthCheck
}

' 部署关系
JManusApp --> H2DB : JDBC
JManusApp --> LocalCache : Direct Access
JManusApp --> Chrome : WebDriver Protocol
JManusApp --> Python : Process Execution
JManusApp --> NodeJS : Process Execution

JManusApp --> AIServices : HTTPS
JManusApp --> MCPEco : MCP Protocol

JManusApp --> Monitoring : Logging/Metrics

@enduml
```

### 3. Docker容器化部署

```plantuml
@startuml
!theme plain
title JManus Docker容器化部署图

node "Docker Host 1" as Host1 {
  artifact "Nginx Container" as NginxContainer {
    component "nginx:alpine" as Nginx
    component "SSL Certificates" as SSL
  }
  
  artifact "JManus App Container" as AppContainer {
    component "openjdk:17-jre" as JDK
    component "JManus Application" as App
    component "Configuration Files" as Config
  }
}

node "Docker Host 2" as Host2 {
  artifact "Database Container" as DBContainer {
    component "postgres:15" as PostgreSQL
    component "Data Volume" as DataVol
  }
  
  artifact "Redis Container" as RedisContainer {
    component "redis:7-alpine" as Redis
    component "Redis Config" as RedisConfig
  }
}

node "Tool Containers Host" as ToolHost {
  artifact "Browser Container" as BrowserContainer {
    component "selenium/standalone-chrome" as Chrome
    component "Browser Drivers" as Drivers
  }
  
  artifact "Python Container" as PythonContainer {
    component "python:3.11-slim" as Python
    component "Python Packages" as PyPackages
  }
}

node "Docker Registry" as Registry {
  artifact "JManus Images" as Images
  artifact "Tool Images" as ToolImages
}

cloud "Container Orchestration" as Orchestration {
  artifact "Docker Compose" as DockerCompose
  artifact "Docker Swarm" as DockerSwarm
  artifact "Kubernetes" as K8s
}

' 容器网络
NginxContainer --> AppContainer : HTTP
AppContainer --> DatabaseContainer : TCP/5432
AppContainer --> RedisContainer : TCP/6379
AppContainer --> BrowserContainer : HTTP/4444
AppContainer --> PythonContainer : HTTP/8000

Registry --> Host1 : Image Pull
Registry --> Host2 : Image Pull
Registry --> ToolHost : Image Pull

Orchestration --> Host1 : Container Management
Orchestration --> Host2 : Container Management
Orchestration --> ToolHost : Container Management

@enduml
```

### 4. Kubernetes集群部署

```plantuml
@startuml
!theme plain
title JManus Kubernetes集群部署图

cloud "Kubernetes Cluster" {
  
  package "Ingress Layer" {
    artifact "Nginx Ingress Controller" as IngressCtrl
    artifact "SSL Termination" as SSL
    artifact "Load Balancer" as LB
  }
  
  package "Application Namespace" {
    node "JManus Deployment" as JManusDeployment {
      artifact "JManus Pod 1" as Pod1
      artifact "JManus Pod 2" as Pod2
      artifact "JManus Pod 3" as Pod3
    }
    
    artifact "JManus Service" as JManusService
    artifact "ConfigMap" as ConfigMap
    artifact "Secret" as Secret
  }
  
  package "Database Namespace" {
    node "PostgreSQL StatefulSet" as PGStatefulSet {
      artifact "PostgreSQL Master" as PGMaster
      artifact "PostgreSQL Replica" as PGReplica
    }
    
    artifact "PostgreSQL Service" as PGService
    artifact "Persistent Volume Claims" as PVC
  }
  
  package "Cache Namespace" {
    node "Redis Deployment" as RedisDeployment {
      artifact "Redis Master" as RedisMaster
      artifact "Redis Sentinels" as RedisSentinels
    }
    
    artifact "Redis Service" as RedisService
  }
  
  package "Tool Namespace" {
    node "Browser Grid" as BrowserGrid {
      artifact "Chrome Nodes" as ChromeNodes
      artifact "Firefox Nodes" as FirefoxNodes
    }
    
    node "Python Executor" as PythonExec {
      artifact "Python Workers" as PythonWorkers
    }
  }
  
  package "Monitoring Namespace" {
    artifact "Prometheus" as Prometheus
    artifact "Grafana" as Grafana
    artifact "Jaeger" as Jaeger
  }
}

cloud "External Services" {
  artifact "AI APIs" as AIAPIs
  artifact "MCP Servers" as MCPServers
}

' 网络关系
IngressCtrl --> JManusService : Route Traffic
JManusService --> JManusDeployment : Load Balance
JManusDeployment --> PGService : Database Access
JManusDeployment --> RedisService : Cache Access
JManusDeployment --> BrowserGrid : Tool Execution
JManusDeployment --> PythonExec : Code Execution

JManusDeployment --> AIAPIs : External API Calls
JManusDeployment --> MCPServers : MCP Protocol

Prometheus --> JManusDeployment : Metrics Collection
Grafana --> Prometheus : Visualization
Jaeger --> JManusDeployment : Tracing

@enduml
```

## 云平台部署图

### 1. AWS云部署架构

```plantuml
@startuml
!theme plain
title JManus AWS云部署架构

cloud "AWS Cloud" {
  
  package "VPC" {
    
    package "Public Subnets" {
      node "Availability Zone A" as AZA_Public {
        artifact "ALB" as ALB
        artifact "NAT Gateway A" as NATA
      }
      
      node "Availability Zone B" as AZB_Public {
        artifact "NAT Gateway B" as NATB
      }
    }
    
    package "Private Subnets" {
      node "Availability Zone A" as AZA_Private {
        artifact "ECS Tasks A" as ECSA
        artifact "Lambda Functions A" as LambdaA
      }
      
      node "Availability Zone B" as AZB_Private {
        artifact "ECS Tasks B" as ECSB
        artifact "Lambda Functions B" as LambdaB
      }
    }
    
    package "Database Subnets" {
      node "RDS Multi-AZ" as RDS {
        artifact "PostgreSQL Primary" as RDSPrimary
        artifact "PostgreSQL Standby" as RDSStandby
      }
      
      node "ElastiCache" as ElastiCache {
        artifact "Redis Cluster" as RedisCluster
      }
    }
  }
  
  package "Managed Services" {
    artifact "ECS Fargate" as ECSFargate
    artifact "Lambda" as Lambda
    artifact "API Gateway" as APIGateway
    artifact "CloudFront" as CloudFront
    artifact "S3" as S3
    artifact "Secrets Manager" as SecretsManager
  }
  
  package "Monitoring & Security" {
    artifact "CloudWatch" as CloudWatch
    artifact "X-Ray" as XRay
    artifact "WAF" as WAF
    artifact "IAM" as IAM
  }
}

cloud "External Services" {
  artifact "AI Service APIs" as AIAPIs
  artifact "MCP Providers" as MCPProviders
}

' 网络流
CloudFront --> ALB : Content Delivery
ALB --> ECSFargate : Load Balance
ECSFargate --> RDS : Database
ECSFargate --> ElastiCache : Cache
ECSFargate --> Lambda : Serverless Functions

ECSFargate --> AIAPIs : External Calls
Lambda --> MCPProviders : MCP Integration

CloudWatch --> ECSFargate : Monitoring
XRay --> ECSFargate : Tracing
WAF --> ALB : Security

@enduml
```

### 2. 阿里云部署架构

```plantuml
@startuml
!theme plain
title JManus 阿里云部署架构

cloud "阿里云" {
  
  package "专有网络VPC" {
    
    package "公网子网" {
      artifact "负载均衡SLB" as SLB
      artifact "NAT网关" as NATGateway
    }
    
    package "应用子网" {
      node "可用区A" as AZA {
        artifact "ECS实例A1" as ECSA1
        artifact "ECS实例A2" as ECSA2
      }
      
      node "可用区B" as AZB {
        artifact "ECS实例B1" as ECSB1
        artifact "ECS实例B2" as ECSB2
      }
    }
    
    package "数据库子网" {
      artifact "RDS PostgreSQL" as RDS
      artifact "Redis企业版" as Redis
    }
  }
  
  package "托管服务" {
    artifact "容器服务ACK" as ACK
    artifact "函数计算FC" as FC
    artifact "API网关" as APIGateway
    artifact "CDN" as CDN
    artifact "对象存储OSS" as OSS
  }
  
  package "安全与监控" {
    artifact "云监控CMS" as CMS
    artifact "链路追踪" as Tracing
    artifact "Web应用防火墙" as WAF
    artifact "访问控制RAM" as RAM
  }
}

cloud "DashScope" {
  artifact "通义千问" as Qwen
  artifact "通义万相" as Vision
}

cloud "外部MCP服务" {
  artifact "MCP Server 1" as MCP1
  artifact "MCP Server 2" as MCP2
}

' 连接关系
CDN --> SLB : 内容分发
SLB --> AZA : 负载均衡
SLB --> AZB : 负载均衡
AZA --> RDS : 数据库连接
AZB --> RDS : 数据库连接
AZA --> Redis : 缓存连接
AZB --> Redis : 缓存连接

AZA --> Qwen : AI API调用
AZB --> Vision : AI API调用
AZA --> MCP1 : MCP协议
AZB --> MCP2 : MCP协议

CMS --> AZA : 监控采集
CMS --> AZB : 监控采集
Tracing --> AZA : 链路追踪
WAF --> SLB : 安全防护

@enduml
```

## 开发环境部署图

### 1. 本地开发环境

```plantuml
@startuml
!theme plain
title JManus 本地开发环境部署图

node "开发者工作站" as DevWorkstation {
  
  artifact "IDE环境" as IDE {
    component "IntelliJ IDEA" as IntelliJ
    component "VS Code" as VSCode
  }
  
  artifact "JManus应用" as JManusApp {
    component "Spring Boot DevTools" as DevTools
    component "Hot Reload" as HotReload
  }
  
  artifact "本地数据库" as LocalDB {
    component "H2内存数据库" as H2
    component "测试数据" as TestData
  }
  
  artifact "工具环境" as ToolEnv {
    component "Chrome浏览器" as Chrome
    component "Python 3.11" as Python
    component "Node.js" as NodeJS
  }
}

node "Docker Desktop" as DockerDesktop {
  artifact "开发容器" as DevContainers {
    component "Redis容器" as RedisContainer
    component "PostgreSQL容器" as PostgreSQLContainer
    component "MCP模拟器" as MCPSimulator
  }
}

cloud "开发工具服务" {
  artifact "GitHub" as GitHub
  artifact "Maven Central" as MavenCentral
  artifact "Docker Hub" as DockerHub
}

cloud "测试AI服务" {
  artifact "OpenAI API" as OpenAI
  artifact "本地LLM" as LocalLLM
}

' 连接关系
IDE --> JManusApp : 开发调试
JManusApp --> LocalDB : 数据存储
JManusApp --> ToolEnv : 工具调用
JManusApp --> DevContainers : 外部服务

DevContainers --> DockerDesktop : 容器运行

IDE --> GitHub : 代码管理
JManusApp --> MavenCentral : 依赖下载
DevContainers --> DockerHub : 镜像拉取

JManusApp --> OpenAI : AI服务测试
JManusApp --> LocalLLM : 本地AI测试

@enduml
```

### 2. CI/CD流水线部署

```plantuml
@startuml
!theme plain
title JManus CI/CD流水线部署图

package "源代码管理" {
  artifact "GitHub Repository" as GitHub
  artifact "Feature Branches" as FeatureBranches
  artifact "Main Branch" as MainBranch
}

package "CI/CD平台" {
  node "GitHub Actions" as GitHubActions {
    artifact "Build Job" as BuildJob
    artifact "Test Job" as TestJob
    artifact "Security Scan Job" as SecurityJob
    artifact "Package Job" as PackageJob
  }
  
  artifact "Workflow Triggers" as Triggers
}

package "构建环境" {
  node "Build Agents" as BuildAgents {
    artifact "Maven Build" as MavenBuild
    artifact "Docker Build" as DockerBuild
    artifact "Test Execution" as TestExec
  }
}

package "制品仓库" {
  artifact "Docker Registry" as DockerRegistry
  artifact "Maven Repository" as MavenRepo
  artifact "Helm Charts" as HelmCharts
}

package "部署环境" {
  node "Staging Environment" as Staging {
    artifact "Staging Cluster" as StagingCluster
    artifact "Integration Tests" as IntegrationTests
  }
  
  node "Production Environment" as Production {
    artifact "Production Cluster" as ProdCluster
    artifact "Blue-Green Deployment" as BlueGreen
  }
}

package "监控反馈" {
  artifact "Deployment Monitoring" as DeployMonitor
  artifact "Performance Testing" as PerfTest
  artifact "Health Checks" as HealthChecks
}

' 流水线流程
GitHub --> Triggers : Push/PR Events
Triggers --> GitHubActions : Trigger Workflows
GitHubActions --> BuildAgents : Execute Jobs

BuildJob --> MavenBuild : Compile & Package
TestJob --> TestExec : Unit & Integration Tests
SecurityJob --> TestExec : Security Scanning
PackageJob --> DockerBuild : Container Building

BuildAgents --> DockerRegistry : Push Images
BuildAgents --> MavenRepo : Publish Artifacts
BuildAgents --> HelmCharts : Update Charts

DockerRegistry --> Staging : Deploy to Staging
IntegrationTests --> Production : Approval Gate
Production --> BlueGreen : Production Deployment

DeployMonitor --> ProdCluster : Monitor Deployment
PerfTest --> ProdCluster : Performance Validation
HealthChecks --> GitHub : Feedback Loop

@enduml
```

## 网络架构与安全部署

### 1. 网络安全架构

```plantuml
@startuml
!theme plain
title JManus 网络安全架构部署图

package "外部网络" {
  cloud "Internet" {
    [用户流量] as UserTraffic
    [恶意攻击] as Attacks
  }
}

package "边界安全" {
  node "DDoS防护" as DDoSProtection
  node "Web应用防火墙" as WAF
  node "SSL终端" as SSLTermination
}

package "负载均衡层" {
  node "负载均衡器" as LoadBalancer {
    artifact "健康检查" as HealthCheck
    artifact "会话保持" as SessionAffinity
  }
}

package "应用网络" {
  node "应用集群" as AppCluster {
    artifact "JManus实例" as JManusInstances
    artifact "内部API" as InternalAPI
  }
  
  node "服务网格" as ServiceMesh {
    artifact "Istio" as Istio
    artifact "mTLS" as mTLS
    artifact "流量策略" as TrafficPolicy
  }
}

package "数据网络" {
  node "数据库集群" as DBCluster {
    artifact "加密连接" as EncryptedConn
    artifact "访问控制" as AccessControl
  }
  
  node "缓存集群" as CacheCluster {
    artifact "Redis认证" as RedisAuth
    artifact "数据加密" as DataEncryption
  }
}

package "监控安全" {
  node "安全监控" as SecurityMonitoring {
    artifact "入侵检测" as IDS
    artifact "日志审计" as LogAudit
    artifact "威胁检测" as ThreatDetection
  }
}

' 安全流量路径
UserTraffic --> DDoSProtection : 流量清洗
DDoSProtection --> WAF : 应用层防护
WAF --> SSLTermination : SSL卸载
SSLTermination --> LoadBalancer : 负载分发

LoadBalancer --> ServiceMesh : 服务路由
ServiceMesh --> AppCluster : 加密通信
AppCluster --> DBCluster : 安全连接
AppCluster --> CacheCluster : 认证访问

SecurityMonitoring --> AppCluster : 监控
SecurityMonitoring --> DBCluster : 审计
SecurityMonitoring --> CacheCluster : 检测

Attacks -.-> DDoSProtection : 攻击阻断
Attacks -.-> WAF : 恶意请求拦截

@enduml
```

### 2. 多数据中心部署

```plantuml
@startuml
!theme plain
title JManus 多数据中心部署架构

package "主数据中心 (北京)" {
  node "Primary DC" as PrimaryDC {
    artifact "JManus集群 A" as ClusterA
    artifact "PostgreSQL主库" as PGPrimary
    artifact "Redis主集群" as RedisPrimary
  }
  
  artifact "全局负载均衡" as GSLB
}

package "灾备数据中心 (上海)" {
  node "Disaster Recovery DC" as DRDC {
    artifact "JManus集群 B" as ClusterB
    artifact "PostgreSQL从库" as PGSecondary
    artifact "Redis从集群" as RedisSecondary
  }
}

package "边缘节点 (广州)" {
  node "Edge DC" as EdgeDC {
    artifact "缓存层" as EdgeCache
    artifact "静态资源" as StaticFiles
  }
}

cloud "CDN网络" {
  artifact "全球CDN节点" as CDNNodes
}

package "数据同步" {
  artifact "数据库复制" as DBReplication
  artifact "文件同步" as FileSync
  artifact "配置同步" as ConfigSync
}

' 数据中心连接
GSLB --> ClusterA : 主要流量
GSLB --> ClusterB : 故障切换
GSLB --> EdgeDC : 边缘加速

ClusterA --> PGPrimary : 读写
ClusterB --> PGSecondary : 只读
PGPrimary --> PGSecondary : 数据复制

ClusterA --> RedisPrimary : 缓存
ClusterB --> RedisSecondary : 备用缓存
RedisPrimary --> RedisSecondary : 数据同步

CDNNodes --> EdgeDC : 边缘分发
EdgeDC --> ClusterA : 回源

DBReplication --> PGSecondary : 异步复制
FileSync --> ClusterB : 文件同步
ConfigSync --> ClusterB : 配置同步

@enduml
```

## 部署配置与运维

### 关键部署参数

1. **资源配置**
   - CPU: 4-8核心（生产环境）
   - 内存: 8-16GB（生产环境）
   - 存储: SSD 100GB+（数据库）
   - 网络: 千兆带宽

2. **JVM参数**
   ```bash
   -Xms4g -Xmx8g
   -XX:+UseG1GC
   -XX:MaxGCPauseMillis=200
   -XX:+HeapDumpOnOutOfMemoryError
   ```

3. **数据库配置**
   - 连接池: HikariCP
   - 最大连接数: 20-50
   - 连接超时: 30s
   - 读写分离: 支持

4. **缓存配置**
   - Redis集群模式
   - 内存: 4-8GB
   - 持久化: RDB+AOF
   - 过期策略: LRU

### 监控指标

1. **应用指标**
   - QPS/TPS
   - 响应时间
   - 错误率
   - JVM指标

2. **基础设施指标**
   - CPU使用率
   - 内存使用率
   - 磁盘I/O
   - 网络流量

3. **业务指标**
   - 任务执行成功率
   - 智能体响应时间
   - AI API调用延迟
   - 工具执行成功率

---

**文档版本**: 1.0  
**创建日期**: 2025年1月  
**部署图数量**: 10个核心部署图  
**涵盖环境**: 开发、测试、生产、云平台等  
**建模工具**: PlantUML