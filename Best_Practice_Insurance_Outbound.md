# Best Practice: Parlant Insurance Outbound Demo

这份文档面向第一次接手本项目的同事，目标不是解释全部 SDK 细节，而是讲清楚这套保险外呼 Demo 在 Parlant 里的核心建模方式，以及后端是如何把业务 `SOP` 映射成可运行的 `Journey`、`Guideline`、`Glossary` 和 `Tool` 的。

## 1. 先讲结论

这套 Demo 采用的是一套很典型的 Parlant 建模方式：

- `SOP` 不是 Parlant 的原生对象
- `Journey` 承载“流程骨架”
- `Guideline` 承载“行为规则、合规要求、异议处理、长尾恢复”
- `Tool` 承载“读写外部事实”
- `Glossary` 承载“领域词汇和业务口径”
- `lead metadata + tool result` 承载“每个会话的动态上下文”

所以，这套设计的关键不是“把 SOP 当对象直接加载”，而是：

1. 先把业务 SOP 写成一个清晰的源文件
2. 启动时把 SOP 的步骤映射到 Journey 状态和 scoped Guidelines
3. 运行时通过当前 session 的 `lead_id` 动态读取不同客户画像
4. 让同一个 Journey/Guideline 结构，在不同 lead 上跑出不同的话术和不同结果

## 2. Parlant 里几个最重要的概念

### 2.1 Journey

`Journey` 是流程编排对象，适合表达：

- 一个任务从哪里开始
- 中间会经过哪些状态
- 在什么条件下转向哪个分支
- 哪一步要调用工具
- 哪一步要结束或交接

在本项目里，`Journey` 就是“保险外呼资格筛选 SOP”的主流程骨架。

对应代码：
[qualification_journey.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/journeys/qualification_journey.py)

核心状态顺序是：

1. `load_lead_profile`
2. `opening`
3. `current_coverage`
4. `need_discovery`
5. `intent outcome branch`
6. `handoff` 或 `follow-up` 或 `close`

一句话理解：
`Journey` 负责回答“流程现在走到哪一步了”。

### 2.2 Guideline

`Guideline` 是条件驱动的行为规则，适合表达：

- 什么时候该怎么说
- 哪些话不能说
- 遇到异议时怎么处理
- 调用某个工具前需要补齐哪些字段
- 某个 Journey 阶段里必须遵守什么顺序

在这个 Demo 里，Guideline 分为 4 类：

1. 全局规则
对应文件：
[global_rules.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/guidelines/global_rules.py)

2. 异议处理
对应文件：
[objection_handling.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/guidelines/objection_handling.py)

3. 长尾问题恢复
对应文件：
[long_tail_guidelines.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/guidelines/long_tail_guidelines.py)

4. Journey scoped 的 SOP enforcement
对应文件：
[sop_enforcement.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/guidelines/sop_enforcement.py)

一句话理解：
`Guideline` 负责回答“在当前上下文下，Agent 应该如何行动”。

### 2.3 Glossary

`Glossary` 是领域词汇表，用来给模型稳定业务语义。

在本项目里，目前词汇很少，但角色很明确：

- `Insurance Lead`
- `Coverage Gap`
- `Do Not Call`

对应文件：
[glossary.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/glossary.py)

一句话理解：
`Glossary` 负责回答“这个业务词在本系统里是什么意思”。

它不是知识库，也不是 FAQ，更像一层统一术语定义。

### 2.4 Tool

`Tool` 是 Agent 和外部事实世界打交道的能力。

本项目的工具全部在：
[lead_tools.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/tools/lead_tools.py)

现在主要有 4 个工具：

- `load_lead_profile`
- `record_lead_outcome`
- `record_follow_up_plan`
- `record_handoff_packet`

它们分别负责：

- 读取当前 lead 画像
- 记录意愿分层结果
- 记录后续回访计划
- 记录人工顾问交接包

一句话理解：
`Tool` 负责回答“Agent 需要读取什么，或者把结果写到哪里”。

### 2.4.1 什么时候该用 Tool，什么时候该用 Retriever

这是 Parlant 里一个非常重要、也非常容易混淆的区别。

可以用一句话先记住：

- `Retriever` 解决“Agent 应该知道什么”
- `Tool` 解决“Agent 需要做什么”

#### 例子 A：适合用 Retriever

需求：

客户问：

> “医疗险和重疾险有什么区别？我这种有孩子的家庭应该先看哪个？”

这里更适合 `Retriever`，因为 Agent 需要从企业知识库里读取相关知识，再组织回答。

例如它可能需要读取：

- 医疗险介绍
- 重疾险介绍
- 两类险种的区别
- 有家庭责任客户的常见优先级建议

这类信息通常来自：

- 企业产品文档
- FAQ
- 销售知识库
- 内部培训材料

为什么这是 `Retriever` 而不是 `Tool`：

- Agent 不是在执行业务动作
- Agent 是在补充知识上下文
- 返回的是“知识片段”，不是“系统操作结果”

所以这类场景更适合建模成：

`customer asks product-detail question -> retriever returns relevant knowledge -> agent explains using that knowledge`

#### 例子 B：适合用 Tool

需求：

客户说：

> “可以，下周三晚上联系我。”

或者：

> “不要再联系我了。”

这里更适合 `Tool`，因为 Agent 需要执行一个明确的业务动作。

例如它需要：

- 写入 follow-up 计划
- 更新 lead outcome
- 打上 `do_not_call` 标签
- 生成 handoff packet

为什么这是 `Tool` 而不是 `Retriever`：

- 这里不是要补知识
- 而是要把对话结果写入业务系统
- 输出应当是结构化、可落盘、可审计的

所以这类场景更适合建模成：

`customer gives operational intent -> agent calls tool -> business result is persisted`

#### 在保险外呼项目里的判断方法

如果你在写需求时发现问题更像下面这些词，优先想 `Retriever`：

- 解释
- 介绍
- 区别
- 适用人群
- 保障范围
- 常见限制

如果更像下面这些词，优先想 `Tool`：

- 记录
- 更新
- 打标
- 查询精确状态
- 安排回访
- 发起交接

#### 对这个项目的直接映射

在当前保险 Demo 里：

- `load_lead_profile` 是 `Tool`
- `record_lead_outcome` 是 `Tool`
- `record_follow_up_plan` 是 `Tool`
- `record_handoff_packet` 是 `Tool`

而未来如果我们接入产品知识库，下面这种能力就更适合做成 `Retriever`：

- 检索医疗险/重疾险/定寿/意外险说明
- 检索不同家庭结构对应的保障建议
- 检索常见理赔边界和免责范围说明

所以可以把它们理解成：

- `Tool`：让 Agent 和业务系统交互
- `Retriever`：让 Agent 和知识系统交互

### 2.5 SOP

`SOP` 在 Parlant 里不是原生实体，而是我们自己定义的业务源文件。

对应文件：
[qualification_sop.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/sop/qualification_sop.py)

这里定义的不是 prompt，而是业务步骤：

- `opening`
- `current_coverage`
- `need_discovery`
- `intent_scoring`
- `handoff_or_close`

每一步都包含：

- `objective`
- `success_signal`
- `mapped_component`

一句话理解：
`SOP` 负责回答“业务流程应该怎么设计”，但真正执行它的是 Journey 和 Guideline。

## 3. 这套 Demo 的真实调用链路

这里是最重要的部分。

### 3.1 启动阶段：先把 Agent 装配出来

入口在：
[run_local_server.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/scripts/run_local_server.py)

启动时做了三件事：

1. 启动 Parlant `Server`
2. `ensure_lead_customers(server)`
3. `ensure_insurance_outbound_agent(server)`

其中 agent 装配函数在：
[app.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/app.py)

装配顺序是：

1. `create_agent`
2. `add_domain_glossary`
3. `add_global_guidelines`
4. `add_objection_guidelines`
5. `add_long_tail_guidelines`
6. `create_qualification_journey`

这意味着：

- 词汇先注册
- 通用规则再注册
- 最后把主流程挂上去

这是推荐做法，因为 Journey 依赖的语言环境和规则环境应当先准备好。

### 3.2 SOP 是怎么“动态加载”进系统的

这里要说清楚：这不是“运行中热加载 markdown 文件”的那种动态加载。

本项目里的动态加载，准确说分两层。

#### 第一层：启动时映射式加载

`qualification_sop.py` 作为业务真源，定义了每个 SOP step。

然后在：
[qualification_journey.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/journeys/qualification_journey.py)

通过 `sop_step("opening")`、`sop_step("current_coverage")`、`sop_step("need_discovery")` 等调用，把 SOP 的 `objective` 注入到 Journey 的 `chat_state` 文案里。

例如这类逻辑：

```python
f"{sop_step('opening').objective} Use the loaded lead profile..."
```

也就是说：

- SOP 文件定义业务步骤
- Journey 在创建状态时读取这些步骤
- 最终让每个状态携带业务目标

这就是第一层动态：  
`SOP step -> Journey state text`

#### 第二层：Journey scoped guideline 加载

在同一个 `create_qualification_journey()` 里，最后还会调用：

```python
await add_qualification_sop_guidelines(journey)
```

对应文件：
[sop_enforcement.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/guidelines/sop_enforcement.py)

这里把 SOP 变成了一组 Journey-scoped Guideline，例如：

- Journey 刚开始时，遵循 SOP 顺序
- opening 阶段必须先拿到继续沟通许可
- discovery 阶段至少收集一个 budget signal 和 timing signal
- handoff 前必须先创建 handoff packet

这就是第二层动态：  
`SOP step -> Journey-scoped rules`

所以更准确地说：

这套系统不是“直接执行 SOP”，而是“把 SOP 编译成 Journey + Guideline”。

## 4. 运行时为什么会对不同 Lead 表现不同

这是这套 Demo 的第二个关键点。

### 4.1 Lead 在 session 里被绑定

lead 目录逻辑在：
[lead_directory.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/lead_directory.py)

启动时会执行：

- 读取 `data/leads.json`
- 为每个 lead 创建一个同 ID 的 customer

这样在这个 Demo 里：

`Lead == Customer`

同时，前端创建 session 时会把 `lead_id` 放进 session metadata。

API 层入口在：
[api.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/api.py)

这个文件做了两件和运行时密切相关的事：

1. 提供 `/demo/leads` 给前端选 lead
2. 强制 `POST /sessions` 使用 `allow_greeting=true`

### 4.2 Journey 一开始先调用 `load_lead_profile`

主 Journey 的第一步不是开场，而是：

```python
t0 = await journey.initial_state.transition_to(tool_state=load_lead_profile)
```

也就是说，每次会话进入这条 qualification journey，都会先读当前 lead。

### 4.3 `load_lead_profile` 会按当前 session 的 metadata 取 lead

核心逻辑在：
[lead_tools.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/tools/lead_tools.py)

关键函数是：

- `_resolve_current_lead()`
- `load_lead_profile()`

逻辑顺序如下：

1. 读取 `p.Session.current.metadata`
2. 如果 metadata 里有 `lead_id`
3. 根据 `lead_id` 去 `leads.json` 找对应 lead
4. 返回该 lead 的完整画像
5. 如果没有 session lead，再退回 active lead 兜底

所以同一套 Journey/Guideline，不需要写死多个版本，就能因为 `lead_id` 不同而得到不同上下文。

这就是运行时动态：

`session.metadata.lead_id -> load_lead_profile() -> different lead context`

## 5. 这套 Demo 里，各个能力分别解决什么问题

### 5.1 Journey 解决“主线流程一致”

保险外呼的主线必须稳定：

- 先开场
- 再问现有保障
- 再做需求发现
- 再给出结果
- 再 handoff 或 close

这些不能交给自由生成，否则每轮都可能乱。

所以主线必须放在 Journey。

### 5.2 Guideline 解决“遇到不同情况怎么处理”

例如：

- 客户说自己很忙
- 客户说已经有保险
- 客户问为什么联系他
- 客户离题
- 客户预算不足
- 客户问折扣
- 客户只接受 WeChat 跟进

这些不是主流程节点，而是流程中的行为约束和恢复逻辑。

所以应该放在 Guideline，而不是把 Journey 画成一个巨大的分叉图。

### 5.3 Tool 解决“把事实拿进来，把结果写出去”

如果没有 Tool，Agent 只能“说过了”，但不会真的留下业务结果。

这个 Demo 用 Tool 把业务事实持久化到本地 JSON：

- `lead_outcomes.json`
- `follow_up_queue.json`
- `handoff_queue.json`

这也是为什么我们能把 Parlant 对话和运营动作连接起来。

### 5.4 Glossary 解决“口径统一”

Glossary 在这个 Demo 里还很轻，但它是值得保留的层。

原因是保险这种领域有很多容易漂移的词：

- `coverage gap`
- `do not call`
- `policy review`
- `qualified lead`

如果不固定定义，随着 guideline 和 journey 增多，模型理解会越来越散。

## 6. 我们这套设计为什么是合理的

### 6.1 把 SOP 当“源文件”，不是当 prompt

这是本项目最好的实践之一。

优点：

- 业务流程有独立真源
- Journey 和 Guideline 可以从 SOP 派生
- 以后改流程，不必直接去翻所有规则文件
- 便于未来把 SOP 做成 GUI 可编辑对象

### 6.2 把主流程和长尾恢复分开

主流程在 Journey，长尾在 Guideline，这样：

- 流程图不会爆炸
- 运营人员能分清“主线”和“异常处理”
- 更适合后续做 GUI 观察命中情况

### 6.3 让 Tool 成为结构化输出边界

`record_lead_outcome`、`record_follow_up_plan`、`record_handoff_packet` 这些工具，是这套 Demo 从“聊天”走向“业务系统”的关键。

如果未来接 CRM、拨号系统、工单系统，这一层最容易替换。

## 7. 这份 Demo 里最值得同事记住的调用逻辑

可以记成下面这条链：

```text
run_local_server.py
  -> ensure_lead_customers(server)
  -> ensure_insurance_outbound_agent(server)
      -> add_domain_glossary(agent)
      -> add_global_guidelines(agent)
      -> add_objection_guidelines(agent)
      -> add_long_tail_guidelines(agent)
      -> create_qualification_journey(agent)
          -> sop_step(...) 读取 SOP 步骤目标
          -> 定义 Journey 主状态和条件分支
          -> 定义工具调用前的 Journey-scoped guidelines
          -> add_qualification_sop_guidelines(journey)
```

运行时再接上：

```text
front-end selects lead
  -> session.metadata.lead_id
  -> journey starts
  -> load_lead_profile()
  -> current lead context loaded
  -> opening / coverage / discovery / outcome
  -> record_* tools persist business results
```

## 8. 给同事的实操建议

### 8.1 想改流程顺序，优先看 SOP 和 Journey

先看：
[qualification_sop.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/sop/qualification_sop.py)

再看：
[qualification_journey.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/journeys/qualification_journey.py)

不要先去改长尾 guideline。

### 8.2 想改异议处理，优先看 Guideline

先看：
[objection_handling.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/guidelines/objection_handling.py)

再看：
[long_tail_guidelines.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/guidelines/long_tail_guidelines.py)

### 8.3 想改“不同 lead 的表现”，优先看 lead 数据和工具解析逻辑

先看：
[lead_directory.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/lead_directory.py)

再看：
[lead_tools.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/tools/lead_tools.py)

### 8.4 想改合规口径或统一术语，优先看 Glossary 和 Global Guidelines

先看：
[glossary.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/glossary.py)

再看：
[global_rules.py](/Users/yguo/Documents/Dev/parlant_w/insurance_outbound_demo/guidelines/global_rules.py)

## 9. 未来更好的演进方向

如果未来要让运营同学在 GUI 里配置，而不是改代码，建议按下面方向演进：

- 把 `qualification_sop.py` 升级成可视化 Procedure
- 把 `Guideline` 变成自然语言可编辑的 Guidance Cards
- 把 Journey scoped guideline 和全局 guideline 明确分层展示
- 给每轮对话增加 trace，展示命中了哪些 guideline、处于哪个 journey state、调用了哪个 tool

这样做时，这份 Demo 现在的代码结构其实已经是一个很好的起点，因为：

- SOP 已经独立出来了
- 主流程和长尾规则已经拆开了
- 工具边界已经明确了
- 动态上下文已经通过 session metadata 打通了

## 10. 一句话总结

这套保险外呼 Demo 的本质不是“把所有规则写成 prompt”，而是：

把 `SOP` 作为业务真源，  
把它映射成 `Journey + Guideline`，  
再通过 `Tool + lead metadata` 把它变成对不同客户动态生效的可运行系统。
