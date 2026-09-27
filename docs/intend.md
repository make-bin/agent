如果你要做的是**企业级 Agent 的生产落地**，我建议不要把“意图识别”理解成简单的 `LLM → intent` 分类，而应该把它设计成一个独立的 **Intent & Routing Layer（意图理解与路由层）**。

核心目标不是“把用户话术分类得多准”，而是：

> **准确理解用户想完成什么 → 判断是否足够确定 → 补充缺失信息 → 决定执行模式 → 路由到正确 Agent / Workflow / Tool。**

AWS 对 Agent Routing 的定义也基本是这个思路：由 classifier/router 理解用户意图，再把请求分发给专门的 Agent、Workflow 或 Tool。([AWS 文档][1])

---

# 一、生产级意图识别应该长什么样

我建议整体架构：

```text
                         User Request
                              │
                              ▼
                    ┌──────────────────┐
                    │ Input Gateway    │
                    │ 鉴权 / 限流 / 安全 │
                    └────────┬─────────┘
                             │
                             ▼
                 ┌────────────────────────┐
                 │ Context Builder         │
                 │                        │
                 │ 当前Query               │
                 │ Session                 │
                 │ Conversation Summary    │
                 │ User Context            │
                 │ 当前Task State          │
                 └───────────┬────────────┘
                             │
                             ▼
                 ┌────────────────────────┐
                 │ Intent Understanding   │
                 │ 意图理解层              │
                 └───────────┬────────────┘
                             │
            ┌────────────────┼────────────────┐
            ▼                ▼                ▼
      Rule Classifier   Semantic Matcher   LLM Classifier
            │                │                │
            └────────────────┼────────────────┘
                             ▼
                 ┌────────────────────────┐
                 │ Intent Decision Engine │
                 │                        │
                 │ Top-K                  │
                 │ Confidence             │
                 │ Ambiguity              │
                 │ Multi-Intent           │
                 │ OOD                    │
                 └───────────┬────────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
           Direct          Clarify        Fallback
           Route           Question       / Human
              │
              ▼
        ┌─────────────────┐
        │ Routing Engine  │
        └────────┬────────┘
                 │
       ┌─────────┼─────────────┐
       ▼         ▼             ▼
     Agent    Workflow        Tool
```

**非常重要的一点：**

> **Intent Classification 和 Routing 不应该是一个概念。**

Intent 是：

```text
用户想做什么？
```

Routing 是：

```text
谁来做？
怎么做？
用哪个 Workflow？
是否需要多个 Agent？
```

这样后面 Agent 平台扩展才不会被意图分类模型绑死。

---

# 二、不要只设计一个 Intent

生产系统建议把用户意图建模成一个 **Intent Object**。

例如：

用户：

> “帮我查一下订单 12345 为什么还没发货，如果超过承诺时间帮我申请退款。”

不要只返回：

```json
{
  "intent": "refund"
}
```

而应该：

```json
{
  "intent": {
    "domain": "order",
    "action": "delivery_inquiry",
    "sub_action": "late_delivery"
  },

  "secondary_intents": [
    {
      "domain": "refund",
      "action": "apply_refund"
    }
  ],

  "entities": {
    "order_id": "12345"
  },

  "constraints": {
    "condition": "delivery_overdue"
  },

  "execution_mode": "workflow",

  "confidence": 0.94,

  "ambiguity": 0.06,

  "requires_clarification": false
}
```

这就从简单的：

```text
Intent Classification
```

升级成：

```text
Intent Understanding
```

---

# 三、Intent Schema 怎么设计

我建议企业 Agent 平台统一定义：

```json
{
  "intent_id": "order.delivery.late",
  "domain": "order",
  "action": "delivery_inquiry",
  "operation": "query",

  "entities": {
    "order_id": "12345"
  },

  "constraints": {
    "status": "overdue"
  },

  "goal": "查询订单延迟原因",

  "execution_mode": "workflow",

  "required_capabilities": [
    "order.query",
    "shipping.query"
  ],

  "confidence": 0.94,

  "ambiguity": 0.03,

  "ood": false,

  "multi_intent": false,

  "requires_clarification": false
}
```

其中几个字段非常关键。

### 1. intent_id

不要只使用：

```text
退款
```

而应该有层次：

```text
order
 ├── query
 │    ├── order_status
 │    └── delivery_status
 │
 └── refund
      ├── refund_policy
      ├── refund_apply
      └── refund_cancel
```

也就是：

```text
Domain
   ↓
Capability
   ↓
Action
   ↓
Operation
```

这样才能支持企业级 Agent 数量不断增加。

---

# 四、Intent Catalog 是整个系统的核心

实际上生产级 Intent Recognition 最大的问题，不是模型，而是：

> **Intent Catalog 怎么设计。**

建议平台维护一个：

```text
Intent Catalog
```

例如：

| Intent           | 描述   | Required Entity | Agent          | Workflow          |
| ---------------- | ---- | --------------- | -------------- | ----------------- |
| order.query      | 查询订单 | order_id        | OrderAgent     | OrderQuery        |
| order.cancel     | 取消订单 | order_id        | OrderAgent     | CancelOrder       |
| refund.query     | 查询退款 | order_id        | RefundAgent    | RefundQuery       |
| refund.apply     | 申请退款 | order_id/reason | RefundAgent    | RefundWorkflow    |
| logistics.query  | 查询物流 | tracking_no     | LogisticsAgent | Tracking          |
| complaint.create | 发起投诉 | reason          | ComplaintAgent | ComplaintWorkflow |

**Intent 本质上应该成为 Agent 平台的一个业务契约。**

这个思路与当前一些 Agent Routing 架构的做法一致：先定义 route/capability contract，再由 router 实现路由，而不是让分类模型自行“发明”路由。([Jitender Sharma][2])

---

# 五、生产环境不要“一上来就调用大模型”

这是非常重要的一点。

推荐采用：

```text
                User Query
                    │
                    ▼
             ┌──────────────┐
             │ Rule Engine  │
             └──────┬───────┘
                    │
              能确定？
              /       \
            Yes        No
             │          │
             ▼          ▼
           Route    Semantic Search
                         │
                    高置信度？
                    /      \
                  Yes       No
                   │         │
                   ▼         ▼
                 Route      LLM
                             │
                             ▼
                     Decision Engine
```

也就是：

### 第一层：Deterministic Rules

例如：

```text
“订单 12345”
“退款”
“取消订单”
```

某些明显关键词、API 参数、业务上下文可以直接确定。

这样可以降低：

* LLM 成本
* 延迟
* 不确定性

Meta 最近公开的一个生产分类案例也采用类似的 decision funnel：先用确定性规则处理稳定模式，再把新颖/模糊输入交给 LLM。([Engineering at Meta][3])

---

# 六、第二层：Semantic Intent Matching

Intent Catalog 中每一个 Intent 都维护：

```text
Intent Definition
+
Description
+
Examples
+
Negative Examples
+
Required Entities
+
Capabilities
```

例如：

```text
Intent:
refund.apply

Description:
用户希望针对订单申请退款。

Examples:
- 我要退这个订单
- 这个商品不要了，帮我退款
- 我要申请退款

Negative Examples:
- 我的退款什么时候到账
- 退款规则是什么
- 为什么退款失败
```

然后做：

```text
Query
 ↓
Embedding
 ↓
Intent Vector Search
 ↓
Top-K Intent
```

得到：

```text
refund.apply       0.91
refund.query       0.78
refund.policy      0.52
```

这里不要只看 Top1。

应该同时看：

```text
Top1 Score
Top2 Score
Margin = Top1 - Top2
```

例如：

```text
Top1 = 0.82
Top2 = 0.81
```

虽然 Top1 看起来很高，但实际上：

> **高度不确定。**

---

# 七、第三层才是 LLM Intent Classifier

当：

```text
Rule
 ↓
Semantic
 ↓
仍然无法确定
```

再调用 LLM。

LLM 不应该直接返回：

```text
refund
```

而应该返回严格结构化结果：

```json
{
  "intent_id": "refund.apply",
  "confidence": 0.91,
  "ambiguity": 0.08,
  "multi_intent": false,
  "entities": {
    "order_id": "12345"
  },
  "missing_entities": [],
  "requires_clarification": false
}
```

结构化输出对于生产系统非常重要。NVIDIA 的 Intent Classifier 也是将意图、响应/路由等决策通过结构化 JSON 输出，而不是依赖自然语言解析。([NVIDIA Docs][4])

---

# 八、最关键：不要相信 LLM 自己说的 confidence

例如：

```json
{
    "intent": "refund",
    "confidence": 0.98
}
```

**这个 0.98 并不天然代表 98% 正确。**

LLM 自己生成的 confidence 需要通过离线标注数据进行校准。

生产系统真正应该关注：

```text
Predicted Intent
        +
Historical Accuracy
        +
Confidence Calibration
```

而不是：

```text
LLM 自报 confidence
```

生产 Intent Router 应该建立 calibration 数据，尤其是每个 Intent 分别校准，而不是简单设置一个全局阈值。([Future AGI][5])

---

# 九、真正生产级的 Decision Engine

最终不是：

```python
if confidence > 0.8:
    route()
```

而应该综合：

```text
                    Intent Decision
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
    Confidence          Margin            OOD
        │                 │                 │
        └─────────────────┼─────────────────┘
                          │
                    Risk Assessment
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
           Execute     Clarify     Fallback
```

例如：

### Case 1

```text
Top1 = refund.apply
confidence = 0.95
margin = 0.24
OOD = false
risk = low
```

直接执行。

---

### Case 2

```text
Top1 = refund.apply
confidence = 0.78
margin = 0.03
```

不要执行。

问：

> “您是想申请退款，还是查询已经提交的退款进度？”

---

### Case 3

```text
confidence = 0.35
OOD = true
```

进入：

```text
General Agent
```

或者：

```text
Human
```

---

# 十、生产级 Intent 必须支持多意图

这是普通 Intent Classifier 很容易遗漏的。

例如：

> “帮我查一下订单什么时候到，如果今天还不到就帮我退款。”

实际上：

```text
Intent 1:
logistics.query

Intent 2:
refund.apply
```

并且存在：

```text
dependency
```

应该变成：

```text
              User Request
                   │
          ┌────────┴────────┐
          ▼                 ▼
    LogisticsQuery      RefundCheck
          │                 │
          └────────┬────────┘
                   ▼
             Decision
                   │
          overdue = true
                   │
                   ▼
             RefundApply
```

所以 Intent Layer 最终需要输出：

```text
Intent Graph
```

而不仅仅是：

```text
Intent Label
```

---

# 十一、还要解决“上下文意图”

用户：

> 用户：帮我查订单 12345

Agent：

> 订单 12345 当前正在运输中。

用户：

> 那什么时候能到？

第二句话单独看：

```text
什么时候能到？
```

几乎没有完整意图。

但是结合：

```text
Conversation Context
+
Current Task
+
Previous Intent
```

应该识别：

```text
logistics.estimated_arrival
```

所以生产 Intent Recognition 的输入应该是：

```text
Intent Input =
Current Query
+
Conversation Summary
+
Previous Intent
+
Current Task State
+
User Context
```

而不是：

```text
Current Query
```

---

# 十二、意图识别和 Agent Runtime 怎么连接

结合你之前设计的 **Agent Runtime + Agent 多范式架构**，我建议：

```text
                    A2UI Gateway
                         │
                         ▼
                  Agent Runtime
                         │
                         ▼
               Intent & Routing Layer
                         │
             ┌───────────┼────────────┐
             ▼           ▼            ▼
          Simple       Workflow      Agent
          Task          Task          Task
             │           │            │
             ▼           ▼            ▼
          Tool Call   Workflow     Agent Graph
```

这里非常关键：

> **Intent Layer 不负责执行任务。**

它只负责：

```text
Understand
     ↓
Decide
     ↓
Route
```

真正执行交给：

```text
Agent Runtime
Workflow Runtime
Tool Runtime
```

---

# 十三、甚至可以把“执行范式”作为意图识别结果

这与你前面研究的 Agent 多范式设计可以直接结合。

例如：

```json
{
  "intent": "order.status.query",

  "execution": {
    "mode": "tool_call",
    "agent": null,
    "workflow": null
  }
}
```

简单任务：

```text
查订单
```

→ Tool Call。

---

复杂一点：

```text
帮我分析订单为什么延迟
```

变成：

```json
{
  "intent": "order.delay.analysis",
  "execution": {
    "mode": "workflow",
    "workflow": "OrderDelayAnalysis"
  }
}
```

再复杂：

> “分析最近一个月订单延迟原因，并给出优化方案。”

可以：

```json
{
  "intent": "order.delay.analysis",
  "execution": {
    "mode": "agent",
    "agent": "OrderAnalysisAgent"
  }
}
```

因此：

> **Intent Recognition 不只是决定“哪个 Agent”，还可以决定“采用什么执行范式”。**

这会让你的 Agent 平台架构非常清晰。

---

# 十四、生产级 Intent Service 微服务设计

我建议单独做成：

```text
Intent Service
```

内部拆：

```text
Intent Gateway
      │
      ├── Context Builder
      │
      ├── Rule Engine
      │
      ├── Semantic Matcher
      │
      ├── LLM Classifier
      │
      ├── Decision Engine
      │
      ├── Clarification Engine
      │
      ├── OOD Detector
      │
      └── Intent Evaluator
```

旁边配：

```text
Intent Registry
```

管理：

```text
Intent Definition
Examples
Negative Examples
Entity Schema
Agent Mapping
Workflow Mapping
Risk Level
Confidence Threshold
Version
```

---

# 十五、Intent Registry 最终可以成为平台能力

例如：

```yaml
intent:
  id: refund.apply
  version: v3

  description: >
    用户希望针对指定订单申请退款

  examples:
    - 我要退款
    - 帮我退掉这个订单
    - 我不想要了，申请退款

  negative_examples:
    - 退款什么时候到账
    - 退款规则是什么

  entities:
    - order_id
    - refund_reason

  execution:
    mode: workflow
    workflow: RefundWorkflow

  risk:
    level: high

  clarification:
    required_entities:
      - order_id

  routing:
    agent: RefundAgent
```

这样新增一个业务 Agent 时，不需要重新修改整个 Router。

只需要：

```text
注册 Intent
      ↓
注册 Capability
      ↓
绑定 Agent / Workflow
      ↓
增加 Examples
      ↓
Eval
      ↓
上线
```

---

# 十六、最后一定要建立 Intent Evaluation

这是生产系统和 Demo 最大的区别之一。

不能只看：

```text
Intent Accuracy = 95%
```

至少要看：

| 指标                    | 含义               |
| --------------------- | ---------------- |
| Intent Accuracy       | 意图是否正确           |
| Precision             | 路由到某 Intent 是否准确 |
| Recall                | 该 Intent 是否都能识别  |
| Top-K Accuracy        | 正确 Intent 是否进入候选 |
| OOD Recall            | 陌生意图能否识别         |
| Clarification Rate    | 需要澄清的比例          |
| Wrong Route Rate      | 错误路由比例           |
| Multi-Intent Accuracy | 多意图识别能力          |
| Entity Accuracy       | 参数抽取准确率          |
| Routing Success       | 路由后任务成功率         |
| End-to-End Success    | 最终业务成功率          |

尤其要关注：

```text
Wrong Route Rate
```

因为：

> **意图识别错一次，可能导致整个 Agent 执行错。**

而不是简单回答错一句话。

---

# 十七、我建议你最终采用这套生产级模型

可以浓缩成：

```text
                 ┌─────────────────────┐
                 │    User Request     │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │ Context Builder     │
                 └──────────┬──────────┘
                            │
                            ▼
              ┌────────────────────────────┐
              │ Intent Understanding       │
              │                            │
              │ Rule                       │
              │ Semantic                   │
              │ LLM                        │
              └─────────────┬──────────────┘
                            │
                            ▼
              ┌────────────────────────────┐
              │ Intent Decision Engine     │
              │                            │
              │ Intent                     │
              │ Entities                   │
              │ Multi Intent               │
              │ Confidence                 │
              │ OOD                        │
              │ Risk                       │
              └─────────────┬──────────────┘
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Execute        Clarify        Fallback
             │
             ▼
       ┌───────────────┐
       │ Routing Engine │
       └───────┬───────┘
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
     Tool   Workflow   Agent
```

最终形成一个完整闭环：

```text
Intent Catalog
      ↓
Intent Recognition
      ↓
Intent Decision
      ↓
Routing
      ↓
Agent / Workflow / Tool
      ↓
Execution Result
      ↓
Outcome Evaluation
      ↓
Intent Evaluation Dataset
      ↓
优化 Intent Catalog / Examples / Model
      ↓
重新评估
```

**这个闭环才是“生产级意图识别”。**

而且它和你之前在做的 **Agent Runtime + 多种 Agent 执行范式 + Workflow + Tool + A2UI** 是可以直接衔接起来的：**Intent Layer 是 Agent Runtime 上游的“任务理解与执行模式选择层”，而不是 Agent 本身。**

[1]: https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/workflow-for-routing.html?utm_source=chatgpt.com "Workflow for routing - AWS Prescriptive Guidance"
[2]: https://jitendersharma.dev/insights/design-intent-router?utm_source=chatgpt.com "How to Design an Intent Router for Agentic AI | Jitender Sharma"
[3]: https://engineering.fb.com/2026/06/25/security/privacy-aware-infrastructure-in-the-ai-native-era-an-asset-classification-case-study/?utm_source=chatgpt.com "Privacy-Aware Infrastructure in the AI-Native Era: An Asset Classification Case Study - Engineering at Meta"
[4]: https://docs.nvidia.com/aiq-blueprint/2.0.0/architecture/agents/intent-classifier.html?utm_source=chatgpt.com "Intent Classifier — NVIDIA AI-Q Blueprint"
[5]: https://futureagi.com/blog/intent-classification-evaluation-pipeline-2026/?utm_source=chatgpt.com "Intent Classification Evaluation Pipeline 2026"

