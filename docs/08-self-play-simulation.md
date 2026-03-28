---
title: "Self Play：用模拟对话替代真实用户的上线前验证方法"
date: "2026-03-26"
tags: ["技术探索"]
slug: "self-play-simulation"
---

# Self Play：用模拟对话替代真实用户的上线前验证方法

一个新的企业 AI Agent 在正式接待真实客户之前，应该被如何测试？

大多数团队的回答是：内部几个人手动试一试，觉得看起来没问题，就推上线。上线之后再看真实对话、发现问题、修改、再上线。

这个方式在低风险场景勉强可以接受。但在销售外呼、患者预约、法律 Intake 这类直接影响收入或客户关系的场景中，这是不负责任的——因为每一次「上线后发现问题」的代价，都可能是一个丢失的客户或一次糟糕的品牌体验。

iMorph.ai 的 Self Play 功能提供了另一种路径：**让系统在沙盒环境中自行生成大规模的模拟对话，在真实用户出现之前完成系统化验证。**

## Self Play 的核心机制

Self Play 的基本思路来自强化学习领域——让系统通过「自我博弈」来发现弱点。在会话 AI 的语境下，它的含义是：系统扮演两个角色，一个是 Agent（被测试的对象），另一个是模拟客户（测试的驱动者），两者之间进行完整的多轮对话。

**模拟客户（Simulated User）的生成：**

系统从 Lead Profile 或客户画像数据中提取关键维度，为每个测试场景生成一个合成的「模拟用户消息序列」：

```python
simulated_user_config = {
    "persona": {
        "name": "Michael Chen",
        "intent": "existing_coverage_review",   # 现有保障评估
        "coverage_status": "partial",            # 已有部分保险
        "objection_style": "time_pressed",       # 倾向于「我没时间」异议
        "language": "english",
        "emotional_state": "neutral"
    },
    "opening_message": "Hi, I got a call from your office earlier?",
    "response_strategy": "answer_if_asked",      # 被追问才提供信息
    "max_turns": 12
}
```

这个配置驱动一个「用户模拟器」——它不是在随机生成文本，而是基于 persona 约束，在每一轮 Agent 回复后，生成符合该用户特征的下一条消息。

**完整对话的沙盒运行：**

沙盒中的对话与真实对话使用完全相同的 Agent 定义（Journey + Guideline + Tool + Retriever），只是工具调用结果被 Mock（避免写入真实数据库），语音合成被跳过（只处理文本层）。这确保测试结果与生产行为的高度一致。

**评分与分类：**

每一轮对话结束后，系统对完整的对话记录进行多维度评分：

- **任务完成率**：是否走完了预期的 Journey 阶段
- **Guideline 遵守率**：关键行为规则是否在应触发时被遵守
- **信息收集完整性**：Handoff 包中必须包含的字段是否齐全
- **异议处理质量**：遭遇预设异议时，Agent 的响应是否符合预期
- **终止条件合理性**：对话是如何结束的（转接/follow-up/拒接），是否与意图判断一致

评分结果分为 Good / Acceptable / Bad 三级，Bad 的对话自动进入 Insights 队列，触发问题分析和 Suggestion 生成。

## 为什么 Self Play 能发现手动测试发现不了的问题

手动测试的最大局限是**覆盖面**。一个测试者在 30 分钟内能够尝试 5-10 个场景，而这 5-10 个场景往往是测试者认为「最重要的」或「最容易出问题的」——这天然带有认知偏差。

真实客户的分布是长尾的：大量客户会以测试者没想到的方式提问、以奇怪的顺序提供信息、使用不标准的表达方式。这些「边缘情况」在手动测试中几乎不会出现，但在真实用户量下，它们加起来可能占到 20-30% 的对话。

Self Play 通过两种机制覆盖这些边缘情况：

**一，批量场景矩阵生成。** 系统可以基于 Lead Profile 的多维度属性（intent × objection_style × coverage_status × language）自动生成场景矩阵：

```
intent: [new_coverage, existing_review, complaint, price_sensitive]
× objection_style: [time_pressed, skeptical, cooperative, defensive]
× coverage_status: [none, partial, full, confused]
= 48 个场景组合
```

每个场景运行 3-5 次（引入轻微随机性），一次 Self Play 批次可以覆盖 150-200 个对话，在 10 分钟内完成。

**二，已知 Bad 场景的回归测试。** 每一个在历史真实对话或上一轮 Self Play 中发现的 Bad 场景，都可以被转化为固定的回归测试用例。每次修改 Agent 配置后，这些用例自动重跑，确保问题真正被修复而不是被其他改动引入新问题。

## Self Play 在 POC 阶段的战略价值

对于一个正在向客户展示产品的 AI Agent 供应商来说，Self Play 有超出工程的战略意义。

传统的 POC 展示方式是：让客户现场或远程「试用」Agent，看看感觉怎么样。这种方式存在几个问题：

- 客户的试用是随机的，可能恰好触发 Bad 场景，留下不好印象
- 客户无法量化「这个 Agent 好到什么程度」，决策依赖感觉
- 如果 POC 失败，供应商不知道问题出在哪里，优化缺乏方向

Self Play 改变了这个局面：**在 POC 展示前，先用 Self Play 跑 100+ 个场景，把结果报告交给客户看。**

一份 Self Play 报告包含：
- 任务完成率：85%（目标 Journey 完整走完的比例）
- 关键规则遵守率：91%（HIPAA / 免责声明 / 价格保护等必须规则）
- Handoff 包完整率：78%（包含所有必需字段的 Handoff 数量）
- P90 对话轮数：8.3 轮（完成完整资格筛选的中位数轮次）
- Bad 场景分布：12% 来自异议处理，5% 来自意外话题

这比「感觉上不错」更有说服力。客户看到的是结构化证据，而不是销售演示的主观印象。

## 与 Test 模块的关系

Self Play 是 iMorph.ai Test 模块的核心功能，与 Test 模块中的其他能力形成互补：

- **单轮测试（Unit Test）**：验证特定输入→输出的确定性行为（如：输入「我要取消预约」，Agent 必须转接）
- **场景测试（Scenario Test）**：预设完整的多轮对话脚本，验证 Agent 按预期路径运行
- **Self Play**：不预设用户消息，让模拟器根据 persona 自由生成，发现意想不到的边缘情况

三者组合使用：Unit Test 覆盖确定性规则，Scenario Test 覆盖已知流程，Self Play 覆盖未知的长尾场景。

## Self Play 的当前限制与演化方向

Self Play 的有效性取决于模拟用户的「真实感」。当前阶段，模拟用户的局限包括：

- 语言风格偏标准化，缺少真实用户的口语化表达、语法错误、信息跳跃
- 情绪模拟较浅层，难以真实还原「焦虑患者」「愤怒客户」等强情绪状态
- 多模态缺失：Self Play 目前在文本层运行，无法测试语音质量、打断处理等语音特有问题

后续演化方向包括：基于真实历史对话的 few-shot 模拟（让模拟用户更接近真实用户的说话方式），以及在 Self Play 结果和真实对话结果之间做分布比对（持续校准模拟准确度）。

---

Self Play 的本质，是把「上线后发现」的问题变成「上线前发现」。在企业级 AI 产品中，这个时序差异的价值远超工程本身——每一个在沙盒中被发现并修复的问题，都是一个没有发生在真实客户身上的坏体验。
