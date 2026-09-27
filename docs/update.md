我完整看了这本《企业级 Agent 开发手册》的结构和核心内容，并把它与 **2026 年最新的 Agent Runtime、MCP、Agent 安全、长周期 Agent、Evals、Observability，以及社区真实生产问题**做了交叉比较。

我的判断是：

> **这本手册已经不是“Agent 入门手册”，而是一个比较完整的 Agent Runtime / Platform 工程基础版。**
>
> 但如果目标是做到真正的 **企业级 Agent 开发手册 V3 / Production Agent Engineering Handbook**，目前最大的短板不是再增加几个 Agent 范式，而是要把 **Agent 当成一种“分布式、概率性、可执行的软件系统”**来设计。

当前手册已经覆盖了 Agent 建模、ReAct / Planning / Multi-Agent / Workflow、Tool、MCP、Model Gateway、Context / Memory、Policy、Runtime、Task、Quality、Observability、Evaluation 和 Intent 等核心模块。

但是，和 2026 年的生产实践相比，我认为至少还需要补 **12 个大的领域**。

---

# 一、先给你一个总体判断

我把目前这本书和真正生产级 Agent 所需要的能力放在一起看：

| 能力                     | 当前手册 | V3 建议                              |
| ---------------------- | ---- | ---------------------------------- |
| Agent 基础模型             | ✅    | 保留                                 |
| Agent 建模               | ✅    | 深化                                 |
| ReAct / Planning       | ✅    | 保留                                 |
| Multi-Agent            | ✅    | **大幅深化**                           |
| Workflow               | ✅    | **深化为 Durable Workflow**           |
| Tool                   | ✅    | **深化 Tool Runtime**                |
| MCP                    | ✅    | **跟进 2026 MCP**                    |
| Skills                 | ✅    | **增加 Skill Runtime / Marketplace** |
| Model Gateway          | ✅    | **增加 Model Capability Registry**   |
| Context                | ✅    | **升级为 Context Engineering**        |
| Memory                 | ✅    | **增加 Memory Governance**           |
| Policy                 | ✅    | **升级为 Runtime Policy Enforcement** |
| Runtime                | ✅    | **升级为 Durable Agent Runtime**      |
| Checkpoint             | ✅    | **升级为 Event + Snapshot + Replay**  |
| Task                   | ✅    | **升级为 Distributed Task Runtime**   |
| Quality                | ✅    | **增加 Quality Gate**                |
| Evals                  | ✅    | **升级为 Continuous Evaluation**      |
| Observability          | ✅    | **升级为 Agent SRE / Reliability**    |
| Security               | ⚠️   | **需要大幅补强**                         |
| Sandbox                | ❌    | **必须新增**                           |
| Browser / Computer Use | ❌    | **建议新增**                           |
| Agent-to-Agent         | ❌    | **必须新增**                           |
| Agent Protocol         | ⚠️   | MCP 之外增加 A2A                       |
| Agent Deployment       | ❌    | **必须新增**                           |
| Agent Release          | ❌    | **必须新增**                           |
| Agent Versioning       | ⚠️   | **需要系统化**                          |
| Prompt Lifecycle       | ⚠️   | **必须新增**                           |
| Business Transaction   | ❌    | **必须新增**                           |
| Agent SRE              | ❌    | **必须新增**                           |
| Agent Governance       | ⚠️   | **必须新增**                           |
| Cost Engineering       | ⚠️   | **需要升级**                           |
| Knowledge Engineering  | ⚠️   | **需要升级**                           |
| Human Operations       | ⚠️   | **需要升级**                           |

所以这本书现在大概是：

> **Agent Engineering 1.0 → 75%**

距离：

> **Enterprise Agent Engineering 2.0 → 还缺一层真正的 Production Platform Engineering**

---

# 二、最大的结构性问题：现在还是“Agent 技术栈”，不是完整的“Agent 生命周期”

这是我认为最重要的问题。

目前书中的总体架构已经非常完整：

> Reasoning / Execution
> Runtime
> Capability
> Model
> Quality
> Governance
> Observability / Evaluation

这个分层本身是合理的。

但是它缺少一个非常重要的横向维度：

```text
                    Agent Lifecycle
 ┌───────────────────────────────────────────┐
 │ Design → Develop → Test → Evaluate       │
 │      → Release → Deploy → Operate        │
 │      → Observe → Improve → Retire        │
 └───────────────────────────────────────────┘
```

也就是说：

现在的手册回答的是：

> **“一个 Agent 运行的时候需要什么？”**

但企业真正还需要回答：

> **“一个 Agent 从创建到退役，整个生命周期怎么管理？”**

---

# 三、必须新增：Agent Lifecycle Engineering

这是 V3 我最建议增加的一章。

建议直接新增：

# 第十八章 Agent Lifecycle Engineering

完整生命周期：

```text
Idea
 ↓
Design
 ↓
Agent Definition
 ↓
Prompt / Tool / Skill
 ↓
Development
 ↓
Unit Test
 ↓
Evaluation
 ↓
Security Test
 ↓
Staging
 ↓
Canary
 ↓
Production
 ↓
Observe
 ↓
Continuous Evaluation
 ↓
Optimization
 ↓
Rollback
 ↓
Retire
```

尤其需要解决：

### 1. Agent Version

不是只有：

```text
Agent v1
Agent v2
```

而应该是：

```text
Agent Version
├── Prompt Version
├── Model Version
├── Tool Version
├── Skill Version
├── Workflow Version
├── Policy Version
├── Knowledge Version
├── Memory Schema Version
└── Runtime Version
```

否则线上出现问题以后，你根本不知道：

> **“这个 Agent 到底是什么版本？”**

---

# 四、这是目前手册非常明显的短板：没有真正的 Agent Release Engineering

传统微服务有：

```text
Git
→ Build
→ Test
→ CI
→ CD
→ Canary
→ Rollback
```

Agent 必须增加：

```text
Prompt
Model
Tool
Skill
Policy
Knowledge
Evaluation Dataset
```

因此应该形成：

```text
Agent CI/CD
```

例如：

```text
Agent Change
      ↓
Unit Test
      ↓
Tool Test
      ↓
Prompt Test
      ↓
Security Test
      ↓
Golden Dataset
      ↓
Trajectory Eval
      ↓
Regression Eval
      ↓
Cost Eval
      ↓
Canary
      ↓
Production
```

然后设置：

```text
Quality Gate
```

例如：

```text
Task Success Rate >= 95%

Tool Selection Accuracy >= 98%

Hallucination Rate <= 1%

Policy Violation = 0

Critical Security Failure = 0

Cost / Task <= Budget

P95 Latency <= SLA
```

这才是真正的 **Agent CI/CD**。

OpenAI 目前的生产 Agent 实践也明显把 tracing、trace grading、datasets、eval runs 和持续优化连接起来，而不是把 Evaluation 当成上线前的一次测试。([OpenAI Developers][1])

---

# 五、第二个巨大短板：Runtime 还需要升级成 Durable Execution

你这本书已经有：

```text
Checkpoint
Recovery
Failure Model
Cancellation
Resource Control
```

这是正确方向。

而且手册已经明确意识到：

> Checkpoint 能恢复 Agent 状态，但不能撤销已经发生的副作用，因此需要 Idempotency + Side Effect Record + Business State Verification。

这个认识非常重要。

但是还不够。

---

## 真正生产级 Runtime 应该变成：

```text
                Agent Runtime
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
   Event Log     State Store   Snapshot
        │            │            │
        └────────────┼────────────┘
                     ↓
                  Replay
                     ↓
              Deterministic Resume
```

也就是说需要增加：

### Event Sourcing

```text
TaskCreated
StepStarted
LLMCalled
ToolRequested
ToolStarted
ToolCompleted
StateUpdated
CheckpointCreated
HumanApprovalRequested
HumanApproved
StepCompleted
TaskCompleted
```

### Replay

出现问题：

```text
Task 123
```

能够：

```text
Load Events
 ↓
Replay
 ↓
Reconstruct State
 ↓
Find Failure
```

### Deterministic Resume

不能简单：

```text
Pod Crash
 ↓
Load Checkpoint
 ↓
继续
```

而应该：

```text
Pod Crash
 ↓
Load Snapshot
 ↓
Replay Events
 ↓
Check Side Effects
 ↓
Determine Safe Point
 ↓
Resume
```

Google 在 2026 年推出 Agent Executor 时，也把 **durable execution、event log、snapshot、resume**作为长周期 Agent Runtime 的核心能力。([Google Cloud][2])

---

# 六、第三个短板：没有真正解决“副作用事务”

这是企业 Agent 最容易出事故的地方之一。

例如：

```text
Agent
 ↓
退款 Tool
 ↓
退款成功
 ↓
Network Timeout
 ↓
Agent 不知道成功
 ↓
Retry
 ↓
再次退款
```

你当前手册已经提到了 Idempotency，但建议 V3 把它提升为一个独立章节：

# Agent Transaction Engineering

核心：

```text
Agent
 ↓
Business Transaction
 ↓
Action
 ↓
Side Effect
 ↓
Verification
 ↓
Commit
```

增加：

* Idempotency
* Transaction Boundary
* Saga
* Compensation
* Two-phase business confirmation
* Side-effect ledger
* Business-state reconciliation
* Exactly-once illusion
* At-least-once execution
* Safe retry

尤其要明确：

> Agent Runtime 很难真正提供 exactly-once。

工程上通常只能做到：

```text
At-least-once execution
+
Idempotent Tool
+
Side Effect Ledger
+
Business Reconciliation
```

这应该成为企业 Agent 开发手册的核心原则。

---

# 七、第四个巨大短板：Sandbox / Computer Use / Browser Runtime

现在 Agent 已经不只是：

```text
LLM
 ↓
API Tool
```

生产 Agent 越来越需要：

```text
Browser
Code
Shell
File
Computer
Container
Sandbox
```

OpenAI 当前 Agents SDK 已经把 sandbox execution 作为独立能力，并强调运行时与计算资源解耦、安全性、持久性和可扩展性。([OpenAI][3])

所以建议新增：

# Agent Execution Sandbox

架构：

```text
Agent Runtime
       ↓
Execution Sandbox
 ┌───────────────┐
 │ File System   │
 │ Shell         │
 │ Browser       │
 │ Code Runtime  │
 │ Network       │
 │ Package       │
 └───────────────┘
       ↓
External System
```

需要解决：

### Isolation

```text
Tenant
Agent
Task
Sandbox
```

隔离。

### Network Policy

```text
Allowlist
Denylist
Egress Control
DNS Control
Private Network
```

### Credential Isolation

Agent：

```text
不能看到 API Key
不能看到 DB Password
不能看到 OAuth Token
```

而是：

```text
Agent
 ↓
Credential Broker
 ↓
Tool / Sandbox
```

AWS AgentCore 当前也把 Runtime、Identity、Gateway、Memory、Policy、Browser、Code Interpreter 等作为独立的生产资源进行治理，而不是简单视为 Tool。([AWS 文档][4])

---

# 八、第五个短板：Security 还不够深

当前手册已经有：

```text
Prompt Injection
Tool Result Trust Boundary
Least Privilege
Secret Management
Audit
```

这是非常好的基础。

但现在生产环境的安全问题已经从：

> “Prompt Injection 怎么防？”

变成：

> **“整个 Agent 是否形成一个可信计算边界？”**

应该增加：

# Agent Security Engineering

至少包含：

```text
1. Agent Identity
2. User Identity
3. Delegated Identity
4. Tool Identity
5. Workload Identity
6. Credential Broker
7. OAuth
8. Token Exchange
9. Permission Boundary
10. Network Isolation
11. Sandbox
12. Data Boundary
13. Memory Boundary
14. Tool Trust
15. MCP Trust
16. Agent-to-Agent Trust
```

---

## 特别需要增加几个 2026 年的新问题

### Tool Poisoning

MCP Tool 自己的 description 可能成为攻击面。

### Tool Shadowing

两个 Tool：

```text
get_user()
```

和：

```text
get_user_safe()
```

模型可能选错。

### Indirect Prompt Injection

```text
User
 ↓
Agent
 ↓
Web
 ↓
恶意网页
 ↓
Prompt Injection
 ↓
Agent
 ↓
Tool
```

### Memory Poisoning

```text
恶意信息
 ↓
Memory
 ↓
未来 Session
 ↓
Agent
 ↓
错误行动
```

社区生产实践中已经反复出现 memory leakage、tool misuse、auth expiry、retry cascade 等问题；这些问题往往发生在 Agent 的“基础设施层”而不是模型本身。([Reddit][5])

---

# 九、第六个短板：MCP 章节已经需要更新

现在手册中的 MCP：

```text
MCP
Tool
Resource
Prompt
Discovery
Registration
Authentication
Authorization
Audit
Rate Limit
```

基础没问题。

但是已经不能按照早期 MCP 理解。

2026-07-28 MCP 规范已经引入：

```text
Stateless Core
Multi-Round-Trip Requests
Tasks Extension
Authorization Hardening
Extensions Framework
Cacheable List Results
Enterprise Managed Authorization
```

并正式将 Tasks 放入 extension 模型。([Model Context Protocol Blog][6])

所以建议改成：

# MCP Production Engineering

包括：

```text
MCP Client
MCP Gateway
MCP Server
MCP Registry
MCP Authorization
MCP Task
MCP Session
MCP Cache
MCP Observability
MCP Security
MCP Version Compatibility
```

特别是：

```text
Agent
 ↓
MCP Gateway
 ↓
MCP Server
```

应该成为企业架构中的标准边界。

---

# 十、第七个短板：缺少 A2A / Agent-to-Agent

目前书中有：

```text
Multi-Agent
Sub-Agent
Handoff
```

但是没有真正解决：

> **不同 Agent 如何跨系统通信？**

这已经成为独立问题。

现在协议栈越来越明显：

```text
                   Agent Application
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
       MCP              A2A              A2UI
        │                │                │
   Tool / Data       Agent ↔ Agent    Agent ↔ UI
```

Google 2026 年的 Agent Protocol 指南也把 MCP、A2A、A2UI、AG-UI 等放在同一个 Agent 协议生态里讨论。([Google 开发者博客][7])

所以建议增加：

# Agent-to-Agent Architecture

包括：

```text
Agent Discovery
Agent Card
Agent Capability
Agent Identity
Agent Authentication
Agent Authorization
Task Delegation
Handoff
Context Transfer
Task Status
Streaming
Async Task
Agent Trust
```

---

# 十一、第八个短板：Context Engineering 需要从 Memory 中独立出来

现在手册已经很好地定义了：

```text
Context
Memory
Working Memory
Session Memory
Long-term Memory
Episodic Memory
Semantic Memory
```

并且已经提出：

```text
Context Selection
 ↓
Retrieval
 ↓
Ranking
 ↓
Compression
 ↓
Context Assembly
```

这是正确的。

但是 2026 年以后，Context 已经不应该只是 Memory 的一个子问题。

应该独立成：

# Context Engineering

完整模型：

```text
                    Context
                       │
        ┌──────────────┼───────────────┐
        ↓              ↓               ↓
 Conversation      Memory          Knowledge
        ↓              ↓               ↓
       State          Tools          User
        └──────────────┼───────────────┘
                       ↓
                  Retrieval
                       ↓
                    Ranking
                       ↓
                 Compression
                       ↓
                  Assembly
                       ↓
                 Context Budget
                       ↓
                     LLM
```

尤其要增加：

### Context Provenance

每一段 Context：

```text
来自哪里？
什么时候产生？
谁产生？
是否可信？
是否过期？
```

### Context Freshness

```text
Fresh
Stale
Expired
Superseded
Contradicted
```

### Context Trust

```text
System
Trusted Tool
Verified DB
Retrieved Knowledge
User
Web
Memory
```

不同 Trust Level。

---

# 十二、第九个短板：Knowledge Engineering 还不够

目前书里提到了：

```text
RAG
Grounding
Knowledge
```

但企业真正落地 RAG，最难的往往不是 Vector DB。

而是：

```text
Data Ingestion
 ↓
Document Parsing
 ↓
OCR
 ↓
Layout Understanding
 ↓
Chunking
 ↓
Metadata
 ↓
ACL
 ↓
Embedding
 ↓
Index
 ↓
Hybrid Retrieval
 ↓
Rerank
 ↓
Grounding
 ↓
Citation
```

尤其企业知识库：

```text
用户 A 能看到什么？
用户 B 能看到什么？
部门 A 能看到什么？
```

所以必须增加：

# Enterprise Knowledge Engineering

包括：

* Document ingestion
* OCR
* Layout parsing
* Chunking
* Metadata
* ACL-aware retrieval
* Hybrid search
* Reranking
* Knowledge freshness
* Citation
* Provenance
* Knowledge version
* Knowledge deletion
* Knowledge rollback

---

# 十三、第十个短板：Evaluation 需要升级成“持续质量工程”

你现在的 Evaluation 已经比较完整：

```text
Unit
Component
E2E
Trajectory
LLM-as-Judge
Golden Dataset
Observability vs Evaluation
```



但是企业真正需要：

```text
Offline Eval
       ↓
Pre-Production
       ↓
Canary
       ↓
Online Eval
       ↓
Production Trace
       ↓
Failure Mining
       ↓
Dataset
       ↓
Regression Test
       ↓
New Version
```

也就是：

# Continuous Agent Evaluation

而不是：

```text
测试一次
→ 上线
```

而是：

```text
Production
 ↓
Trace
 ↓
Failure Detection
 ↓
Dataset
 ↓
Eval
 ↓
Improvement
 ↓
Release
```

OpenAI 当前的 Agent eval 文档也明确把 **Trace → Grader → Dataset → Eval Run**作为持续改进链路。([OpenAI Developers][1])

社区实际反馈也很一致：很多团队一开始把 Evals 当成事后补救；真正能规模化的团队才会把 eval 当成每次变更都运行的测试体系。([Reddit][8])

---

# 十四、第十一个短板：Observability 要升级为 Agent SRE

目前书里已经有：

```text
Trace
Metrics
Logs
Cost
Debugging
```

这是基础。

但企业级还缺：

# Agent SRE

传统：

```text
Availability
Latency
Error Rate
```

Agent 应该增加：

```text
Task Success Rate
Task Completion Rate
Wrong Tool Rate
Wrong Route Rate
Hallucination Rate
Grounding Rate
Verification Failure Rate
Human Escalation Rate
Policy Deny Rate
Retry Rate
Loop Rate
Token Cost
Cost / Task
Cost / Success
```

---

## 更重要的是 Agent SLO

例如：

```text
Task Success SLO >= 99%

Tool Success >= 99.9%

Policy Violation = 0

P95 Task Latency <= 30s

P99 Task Latency <= 120s

Hallucination <= 1%

Human Escalation <= 5%
```

然后形成：

```text
Agent SLO
 ↓
SLI
 ↓
Alert
 ↓
Incident
 ↓
Root Cause
 ↓
Remediation
```

AWS 当前 AgentCore 的生产观测已经开始覆盖 Agent、Memory、Gateway、Tool、Policy、Identity 等不同资源，并提供 session、resource usage、trace、log 等维度，这说明 Agent observability 正在从“LLM 日志”向完整运行平台演进。([AWS 文档][9])

---

# 十五、第十二个短板：没有“Agent Incident Management”

这个我非常建议加。

传统服务出现：

```text
500
 ↓
Alert
 ↓
Incident
 ↓
Oncall
 ↓
Root Cause
 ↓
Fix
```

Agent 出问题可能是：

```text
没有报错
但是结果错了
```

例如：

```text
HTTP 200
LLM 200
Tool 200
Trace 正常

但是：

业务结果错误
```

这是 Agent 最大的特殊性之一。

所以必须增加：

# Agent Incident Management

分类：

```text
Infrastructure Failure
Model Failure
Tool Failure
Context Failure
Knowledge Failure
Reasoning Failure
Policy Failure
Security Failure
Memory Failure
Routing Failure
Quality Failure
Business Failure
```

然后：

```text
Incident
 ↓
Trace
 ↓
Task
 ↓
Step
 ↓
Decision
 ↓
Root Cause
```

---

# 十六、还有一个非常关键的问题：Silent Failure

传统：

```text
Exception
```

很容易发现。

Agent：

```text
Tool Call = 200
LLM = 200
HTTP = 200
```

但：

```text
业务结果 = 错
```

例如：

```text
应该调用 OrderLookup
Agent 没调用

应该走 RefundWorkflow
Agent 走了普通 Answer

应该询问用户
Agent 自己猜了

应该拒绝
Agent 执行了
```

所以应该增加：

# Behavioral Correctness

不仅检查：

```text
Output Correctness
```

还检查：

```text
Behavior Correctness
```

例如：

```text
Expected:

User
 ↓
Intent
 ↓
OrderLookup
 ↓
PolicyLookup
 ↓
RefundDecision
 ↓
HumanApproval
 ↓
Refund
```

实际：

```text
User
 ↓
LLM
 ↓
Refund
```

最终回答可能看起来完全正常。

但是：

> **Trajectory 是错的。**

这也是社区近期反复讨论的问题：最终输出正确并不能证明 Agent 的工具选择、执行路径和真实业务动作正确。([Reddit][10])

---

# 十七、建议新增一个非常重要的概念：Agent Behavior Graph

我认为这是你这本书 V3 可以做出特色的地方。

传统 Trace：

```text
Task
 ↓
LLM
 ↓
Tool
 ↓
LLM
```

进一步升级：

```text
                Agent Behavior Graph

Task
 │
 ├── Intent
 │
 ├── Decision
 │
 ├── Context
 │
 ├── Model
 │
 ├── Tool
 │
 ├── State
 │
 ├── Policy
 │
 ├── Human
 │
 └── Outcome
```

然后可以检查：

```text
Wrong Route
Missing Tool
Unexpected Tool
Tool Loop
Agent Loop
Policy Bypass
Unexpected Handoff
Missing Verification
Unauthorized Action
```

这会比传统 Observability 更接近真正的 **Agent Debugging**。

---

# 十八、Multi-Agent 也需要重新定义

现在手册里：

```text
Supervisor
 ├── ResearchAgent
 ├── OrderAgent
 ├── PolicyAgent
 └── FinanceAgent
```

已经有了。

但是生产级 Multi-Agent 需要解决：

```text
Agent A
 ↓
Agent B
 ↓
Agent C
```

到底：

* 谁拥有 Task？
* 谁拥有 State？
* 谁拥有 Context？
* 谁负责最终结果？
* 谁负责权限？
* Agent B 能否调用 Agent A 的 Tool？
* Agent B 的输出是否可信？
* Agent A 如何验证 Agent B？
* 出错怎么回滚？

建议形成：

```text
Agent
 ├── Identity
 ├── Capability
 ├── Trust Level
 ├── Context Contract
 ├── Input Contract
 ├── Output Contract
 ├── Permission
 └── SLA
```

也就是：

# Agent Contract

这是现在书里比较缺的。

---

# 十九、建议新增“Agent Contract”

每一个 Agent 都应该像微服务一样拥有：

```yaml
Agent:
  identity:
  owner:
  purpose:

  input:
  output:

  capabilities:
  tools:
  skills:

  policies:
  permissions:

  runtime:
    timeout:
    retry:
    checkpoint:

  quality:
    success_slo:
    hallucination_threshold:

  cost:
    budget:

  security:
    trust_level:

  dependencies:
```

这样：

> Agent 才真正成为企业平台中的“一等软件实体”。

---

# 二十、Model Gateway 也需要升级

目前已经有：

```text
Model Routing
Fallback
Token
Cost
Observability
```

但建议增加：

# Model Capability Registry

因为：

```text
Model A
```

不只是“价格 $X”。

它应该有：

```text
Model
├── Context Window
├── Input Cost
├── Output Cost
├── Reasoning
├── Tool Calling
├── Structured Output
├── Vision
├── Audio
├── MCP
├── Computer Use
├── Latency
├── Reliability
└── Eval Profile
```

于是 Router 才能真正做到：

```text
Task
 ↓
Required Capability
 ↓
Model Capability Matching
 ↓
Cost / Latency / Quality
 ↓
Model
```

而不是简单：

```text
复杂任务 → 大模型
简单任务 → 小模型
```

---

# 二十一、成本控制也需要升级

现在手册已经有：

```text
Cost / Task
Cost / User
Cost / Agent
Cost / Tenant
Cost / Tool
Cost / Model
```

这个基础很好。

但是企业需要：

# Agent FinOps

核心指标：

```text
Cost / Task
Cost / Successful Task
Cost / User
Cost / Tenant
Cost / Agent
Cost / Model
Cost / Tool
Cost / Workflow
```

然后：

```text
Budget
 ↓
Quota
 ↓
Runtime Enforcement
 ↓
Cost Alert
 ↓
Degrade
 ↓
Stop
```

甚至：

```text
Task Budget = $0.5

已经用了 $0.47
 ↓
剩余预算不足
 ↓
切换小模型
 ↓
限制 Tool
 ↓
要求人工
```

---

# 二十二、Human-in-the-loop 要从“审批”升级为“Human Operations”

现在手册：

```text
Approve
Reject
Modify
```

不够。

真正企业需要：

```text
Human Queue
 ↓
Risk
 ↓
Priority
 ↓
SLA
 ↓
Assignment
 ↓
Approval
 ↓
Resume
```

例如：

```text
High Risk
→ 5 min SLA

Medium Risk
→ 30 min SLA

Low Risk
→ Batch Review
```

同时支持：

```text
Takeover
Pause
Modify
Resume
Cancel
Escalate
```

---

# 二十三、还有一个非常容易被忽略的：Agent Data Governance

现在有：

```text
Memory Security
PII
Retention
Deletion
Audit
```

但是企业真正需要：

```text
Data Classification
 ↓
Data Access
 ↓
Data Residency
 ↓
Data Retention
 ↓
Data Deletion
 ↓
Data Lineage
 ↓
Audit
```

尤其：

```text
LLM Input
LLM Output
Tool Input
Tool Output
Memory
Trace
Prompt
Embedding
Knowledge
```

都可能包含企业数据。

所以必须定义：

> **Agent Data Plane**

---

# 二十四、建议重新调整整本书的架构

如果让我基于你现在这本手册继续做 V3，我不会简单在后面增加几章。

我会把它重构成：

```text
企业级 Agent Engineering Handbook V3
```

---

# Part I：Agent Fundamentals

### 1. Agent 基础

### 2. Agent Mental Model

### 3. Agent Object Model

### 4. Agent Contract

---

# Part II：Agent Development

### 5. Agent Development Paradigms

* ReAct
* Planning
* Workflow
* Reflection
* Multi-Agent
* Hybrid

### 6. Prompt Engineering

### 7. Tool Engineering

### 8. Skill Engineering

### 9. Knowledge Engineering

### 10. Context Engineering

### 11. Memory Engineering

---

# Part III：Agent Runtime

### 12. Agent Runtime

### 13. Durable Execution

### 14. State & Event Sourcing

### 15. Checkpoint / Replay / Recovery

### 16. Task & Scheduling

### 17. Transaction & Side Effect

### 18. Sandbox / Browser / Computer Use

---

# Part IV：Agent Infrastructure

### 19. Model Gateway

### 20. Model Capability Registry

### 21. Tool Gateway

### 22. MCP

### 23. A2A

### 24. Agent Gateway

### 25. Agent Communication

---

# Part V：Agent Security

### 26. Agent Identity

### 27. Authorization

### 28. Credential Management

### 29. Prompt Injection

### 30. Tool Poisoning

### 31. Memory Poisoning

### 32. Data Exfiltration

### 33. Sandbox Security

### 34. Agent-to-Agent Trust

---

# Part VI：Agent Quality

### 35. Hallucination

### 36. Grounding

### 37. Validation

### 38. Verification

### 39. Behavioral Correctness

### 40. Agent Evaluation

### 41. Trajectory Evaluation

### 42. Continuous Evaluation

---

# Part VII：Agent Observability & SRE

### 43. Agent Trace

### 44. Agent Metrics

### 45. Agent Logs

### 46. Agent Behavior Graph

### 47. Agent SLI / SLO

### 48. Agent Incident Management

### 49. Agent Debugging

### 50. Agent Reliability Engineering

---

# Part VIII：Agent Lifecycle

### 51. Agent Development Lifecycle

### 52. Agent Versioning

### 53. Prompt Versioning

### 54. Skill / Tool Versioning

### 55. Agent CI/CD

### 56. Canary

### 57. Rollback

### 58. Online Optimization

---

# Part IX：Enterprise Governance

### 59. Multi-Tenancy

### 60. Cost Governance

### 61. Data Governance

### 62. Compliance

### 63. Audit

### 64. Human Operations

### 65. Agent Catalog

### 66. Agent Registry

---

# Part X：Production Patterns

### 67. Customer Service Agent

### 68. Research Agent

### 69. Coding Agent

### 70. Data Analyst Agent

### 71. SRE Agent

### 72. Workflow Agent

### 73. Multi-Agent System

### 74. Long-Running Agent

---

# 二十五、最终我认为最应该补的不是“章节”，而是这张总架构

你现在的架构：

```text
Reasoning
Runtime
Capability
Model
Quality
Governance
Observability
```

建议 V3 升级为：

```text
                         Enterprise Agent
                                │
        ┌───────────────────────┼────────────────────────┐
        │                       │                        │
        ↓                       ↓                        ↓
  Experience Layer        Agent Gateway            Agent Protocol
        │                       │                  MCP / A2A / A2UI
        ↓                       ↓
  Intent / Session        Auth / Routing
                                │
                                ↓
                     ┌─────────────────────┐
                     │   Agent Runtime     │
                     │                     │
                     │ Planner             │
                     │ Executor            │
                     │ State               │
                     │ Scheduler           │
                     │ Durable Execution   │
                     │ Checkpoint          │
                     │ Replay              │
                     └──────────┬──────────┘
                                │
            ┌───────────────────┼────────────────────┐
            ↓                   ↓                    ↓
       Context Engine      Capability Engine    Knowledge Engine
            │                   │                    │
      Memory/RAG          Tool/Skill/MCP       RAG/Search/KB
            │                   │                    │
            └───────────────────┼────────────────────┘
                                ↓
                         Model Gateway
                                │
                ┌───────────────┼───────────────┐
                ↓               ↓               ↓
             Model A         Model B         Model C
                                │
                                ↓
                        Execution Sandbox
                     ┌──────────┼───────────┐
                     ↓          ↓           ↓
                  Browser     Code        Computer
                                │
                                ↓
                        Enterprise Systems


════════════════════════════════════════════════════════════

              横向治理 / 控制平面

 Identity
 Authorization
 Policy
 Security
 Credential
 Multi-Tenant
 Cost
 Audit
 Data Governance


════════════════════════════════════════════════════════════

              横向质量 / SRE 平面

 Trace
 Metrics
 Logs
 Behavior Graph
 Evaluation
 Quality Gate
 SLO
 Incident
 Continuous Improvement


════════════════════════════════════════════════════════════

              Agent Lifecycle

 Design
   ↓
 Develop
   ↓
 Test
   ↓
 Evaluate
   ↓
 Release
   ↓
 Canary
   ↓
 Production
   ↓
 Observe
   ↓
 Improve
   ↓
 Rollback / Retire
```

这个架构比现在的版本更接近 **2026 年企业 Agent Platform 的完整形态**。

---

# 二十六、如果只允许你优先补 10 个，我建议是这 10 个

按照“对企业生产落地影响”来划分，而不是对某个技术栈做评价：

| 优先级 | 必须补齐的能力                             | 原因                            |
| --- | ----------------------------------- | ----------------------------- |
| P0  | **Durable Agent Runtime**           | 长任务、Pod 崩溃、节点故障               |
| P0  | **Agent Transaction / Side Effect** | 防止重复扣款、退款、写入                  |
| P0  | **Agent Security Architecture**     | Prompt Injection 已经不是唯一风险     |
| P0  | **Agent Sandbox**                   | Code / Browser / Computer Use |
| P0  | **Continuous Evaluation**           | 防止线上质量退化                      |
| P0  | **Agent SRE / SLO**                 | 从“能运行”到“可运营”                  |
| P0  | **Agent Lifecycle / CI-CD**         | Agent 版本、灰度、回滚                |
| P1  | **Context Engineering**             | 长上下文和多轮任务的核心                  |
| P1  | **Agent-to-Agent / A2A**            | Multi-Agent 跨系统协作             |
| P1  | **Behavioral Correctness**          | 解决“HTTP 200 但业务做错”            |

---

# 二十七、最值得注意的一个结论

你现在这本书其实已经完成了一个很重要的转变：

```text
Prompt + LLM
        ↓
Agent
        ↓
Agent Runtime
```

但 V3 应该继续完成第二次转变：

```text
Agent Runtime
        ↓
Production Agent System
        ↓
Enterprise Agent Platform
        ↓
Agent SRE + Governance + Lifecycle
```

也就是说：

> **下一阶段不要继续围绕“Agent 有哪些能力”扩章节，而应该围绕“Agent 作为生产系统会怎么坏，以及企业如何控制、恢复、验证、发布和运营它”来组织。**

这一点和 2026 年业界的演进方向比较一致：Google 强调 deterministic control + durable execution，OpenAI 把 guardrails、human review、tracing、evals 和 sandbox 纳入生产 Agent 栈，MCP 本身也已经开始向 stateless、tasks、authorization hardening 演进。([Google 开发者博客][11])

**所以，如果这本书的最终目标真的是“企业级 Agent 开发手册，可以直接指导公司研发实现”，我建议不要做一个简单的 V3 增补版，而是直接做成《企业级 Agent 开发手册 V3：Production Agent Engineering》——在你现有 17 章基础上，重点把上面这些“生产闭环”补齐。**

如果按你现在这本书继续往下做，我下一步最建议直接产出的是一份**“V3 完整目录 + 每章具体要补什么 + 哪些现有章节修改/合并/删除 + 对应的微服务架构 + 技术选型 + 核心数据模型 + API + 执行流程”**，这样就可以直接作为企业 Agent 平台的研发蓝本。

[1]: https://developers.openai.com/api/docs/guides/agent-evals?utm_source=chatgpt.com "Evaluate agent workflows | OpenAI API"
[2]: https://cloud.google.com/blog/products/ai-machine-learning/agent-executor-googles-distributed-agent-runtime?utm_source=chatgpt.com "Agent Executor, Google’s distributed Agent Runtime | Google Cloud Blog"
[3]: https://openai.com/zh-Hans-CN/index/the-next-evolution-of-the-agents-sdk/?utm_source=chatgpt.com "Agents SDK 的全新演进 | OpenAI"
[4]: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-security-best-practices.html?utm_source=chatgpt.com "Security best practices for AgentCore Runtime - Amazon Bedrock AgentCore"
[5]: https://www.reddit.com/r/LLMDevs/comments/1tpui4q/the_hardest_part_of_production_llm_systems_turned/?utm_source=chatgpt.com "The hardest part of production LLM systems turned out to be infrastructure, not prompts"
[6]: https://blog.modelcontextprotocol.io/posts/2026-07-28/?utm_source=chatgpt.com "The 2026-07-28 Specification | Model Context Protocol Blog"
[7]: https://developers.googleblog.com/developers-guide-to-ai-agent-protocols/?utm_source=chatgpt.com "Developer’s Guide to AI Agent Protocols - Google Developers Blog"
[8]: https://www.reddit.com/r/LLMDevs/comments/1rv6kah/main_observability_and_evals_issues_when_shipping/?utm_source=chatgpt.com "Main observability and evals issues when shipping AI agents."
[9]: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/observability-service-provided.html?utm_source=chatgpt.com "Amazon Bedrock AgentCore generated observability data - Amazon Bedrock AgentCore"
[10]: https://www.reddit.com/r/AI_Agents/comments/1u9vkg4/four_things_that_silently_break_in_production_ai/?utm_source=chatgpt.com "Four things that silently break in production AI agents and how to catch them before users do"
[11]: https://developers.googleblog.com/why-we-built-adk-20/?utm_source=chatgpt.com "Why we built ADK 2.0 - Google Developers Blog"
