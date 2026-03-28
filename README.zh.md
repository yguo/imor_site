# iMorph.ai

iMorph.ai 是一个面向销售与外呼场景的 AgentOps 产品。它的目标不是做一个“能聊天的机器人”，而是做一套真正可以构建、测试、监控、部署、分析并持续优化的智能体工作系统。它在产品形态上与 [Fin](https://fin.ai) 有相似之处，都会强调围绕智能体建立持续改进闭环，而不只是提供一个提示词编辑器或聊天界面；但 iMorph.ai 的重心并不放在典型的 inbound support，而是放在 outbound、sales、lead qualification、follow-up orchestration 这些更接近收入生成和运营执行的领域。

这背后的判断很简单。真正进入商业一线后，智能体面对的往往不是静态问答，而是多轮、带状态、带目标、带风险的商业对话。一个销售外呼 Agent 需要理解什么时候应该继续推进，什么时候应该收集信息，什么时候应该解释知识，什么时候应该克制，什么时候应该 handoff，什么时候应该把客户送入 follow-up queue。它还必须在对话中持续感知：这一轮最相关的知识是什么、当前客户状态是什么、哪些术语需要统一、哪些规则必须生效、哪些动作可以执行、哪些信息必须被记住。换句话说，这类产品最核心的问题不是“模型会不会说话”，而是“上下文在每一轮是如何被正确装配出来的”。

这也是 iMorph.ai 选择借鉴 Parlant 的根本原因。Parlant 提供的不是一个简单的 prompt 封装层，而是一种围绕会话运行时上下文进行建模和装配的方法。它把智能体的行为拆成多个显式建模单元，例如 [Guidelines](https://www.parlant.io/docs/concepts/customization/guidelines/)、[Journeys](https://www.parlant.io/docs/concepts/customization/journeys/)、[Retrievers](https://www.parlant.io/docs/concepts/customization/retrievers/)、[Glossary](https://www.parlant.io/docs/concepts/customization/glossary/) 和 [Variables](https://www.parlant.io/docs/concepts/customization/variables/)。这些元素并不是为了让建模变复杂，而是为了让系统只在正确的时候把正确的信息交给模型，而不是把所有说明、所有 SOP、所有知识、所有例外一次性塞进一个超长系统提示词里。对于外呼和销售场景，这种动态上下文加载与条件判断尤其关键，因为客户行为、客户意图、话题路径和业务动作会在每一轮都变化。

## 一、产品主张：把销售与外呼智能体做成“可运营系统”，而不是“Prompt Demo”

如果把今天绝大多数 AI agent 产品分成两类，一类更像“会聊天的前端”，另一类更像“真正的运营系统”。iMorph.ai 明确想做后者。它的产品主张可以概括成一句话：**让企业能够像运营销售流程一样运营智能体。**

这意味着产品必须同时满足四类需求。第一类是建模需求，也就是业务团队如何把一个智能体定义出来。第二类是验证需求，也就是在真正面对客户前，如何知道这个 Agent 的效果能否达到要求。第三类是运行需求，也就是智能体上线后如何被持续监控和调优。第四类是分析需求，也就是系统能否从大量真实对话中反推出知识缺口、规则缺口、流程缺口和运营动作建议。

因此，iMorph.ai 的核心并不是一个聊天窗口，而是一整套围绕智能体生命周期展开的模块：Build、Test、Monitor、Deploy、Insights、Suggestions。Build 负责构建和编辑 Agent Definition；Test 负责在沙盒中与不同 lead 测试对话；Monitor 负责观察每一次回复背后的上下文装配过程；Deploy 负责统一 runtime 与 channel gateway；Insights 负责保存和分析会话；Suggestions 则负责把离线分析结果转化成下一轮调优建议。这套结构背后的理念，与 Fin 对 Train / Test / Deploy / Analyze 的强调非常接近，只不过 iMorph.ai 将它进一步聚焦到 outbound 和 sales 场景。

## 二、为什么要深耕 Outbound 和 Sales

很多 AI 产品会自然从客服支持切入，因为 support 的场景更标准化、目标更清晰、用户意图也更明确。但 outbound 和 sales 才是更能体现智能体“上下文工程”价值的场景。原因在于，销售与外呼不是一个单轮完成的任务，它是一个会随着对话推进不断变化的过程。

一个外呼 Agent 可能面对的是“曾经留资但沉默的 lead”，也可能是“活动现场扫码后等待跟进的人”，还可能是“已有基础保险但对补充保障感兴趣的人”。这些人虽然都可以被称为潜在客户，但他们的状态、关注点、抗拒点、决策节奏完全不同。模型必须根据这些差异决定：应该用怎样的 opening、先问什么、什么时候解释、什么时候收敛、什么时候转人工、什么时候创建 follow-up。这种复杂度远高于简单的 FAQ 问答。

同时，销售场景非常重视“下一步动作”。客户没有成交并不意味着任务结束，可能意味着稍后跟进、换渠道触达、转入销售队列、或者更新 lead_outcome。Agent 如果不能和这些后续动作结合，就只是一个会说话的机器人，而不是一个能参与业务流程的运营节点。也正因如此，iMorph.ai 的最终产品边界不会停留在聊天本身，而是会延伸到队列、后续动作、线索分层、运营建议这些更贴近 GTM 的环节。

从商业版图来看，iMorph.ai 最适合先从几个高结构化、高复用、强 ROI 的外呼场景切入，例如保险外呼资格筛选、汽车试驾邀约、金融预筛选、服务回访与预约、教育咨询邀约等。这些场景的共同点是：有明确的话术边界、有稳定的流程结构、有较多可复用知识，同时对客服/销售人效提升非常敏感。只要在一个垂类里把 Build/Test/Monitor/Insights 这套闭环跑通，就有机会扩展到更多相邻的 revenue workflows。

## 三、技术护城河：会话领域的上下文工程

iMorph.ai 最重要的技术护城河，不在于“也能接一个大模型”，而在于它把会话系统理解成一个上下文装配问题。对销售与外呼智能体而言，这一点比生成质量本身还要重要。

在真实对话里，模型不应该一开始就看到所有知识、所有 SOP、所有规则、所有例外、所有工具说明。这样做的结果往往是：提示词越来越长，注意力越来越稀释，输出越来越不稳定。Parlant 对这一点解释得很清楚：Guidelines 的价值就在于只把“此刻相关的指令”加载进当前上下文，而不是让模型在一堆永远都在的规则中自己猜哪些最重要。Journeys 的价值在于把业务流程建模为一个可推进、可分支、可恢复的多轮结构，而不是让每一轮都从零开始。Retrievers 的价值在于把“应该知道什么”与“应该做什么”区分开来，前者是知识 grounding，后者是动作执行。Glossary 和 Variables 则分别保证术语一致性与会话状态连续性。

这套思路本质上是一种动态上下文加载机制。Journey 先确定大致处于什么业务阶段；Guideline 再决定当前应该遵守哪些行为规则；Retriever 决定这轮需要拉哪些知识；Tool 决定能否执行动作；Variable 决定过去收集的信息是否还能被复用；Canned Responses 决定在高敏感度语境下是否应该采用更稳定的表达。模型最终不是“从全量规则中自由发挥”，而是在一组经过条件判断后装配出来的上下文中生成回复。

这种做法对于 outbound 和 sales 有天然优势。因为销售场景最怕两件事：一是说多了不该说的话，二是没说该说的话。动态上下文装配同时降低了这两种风险。它让系统更容易克制、更容易一致、更容易解释、更容易调优。换句话说，iMorph.ai 的护城河不只是“大模型回答得不错”，而是“在复杂商业对话中，能持续把正确的上下文送到模型前面”。

## 四、iMorph.ai 当前的主要功能

这个仓库虽然还处在 POC 阶段，但功能主线已经成型，而且已经能表达未来产品的核心形态。

Build 是定义层。Agent Definition 会持久化保存，当前已经支持至少两个 agent 的定义管理，并能区分 `definition-only` 与 `runtime-ready` 的状态。Build 不只是展示名字和描述，而是在逐步走向一个真正的 Agent Studio：把 Journey、Guidelines、Glossary、Tools、Retrievers、Variables、Canned Responses 作为显式建模对象展示出来，并允许围绕它们进行编辑、保存和未来的自动草稿生成。

Test 是沙盒层。这里可以把某个 runtime-ready 的 Agent 与某个 lead 组合起来，发起真实 session，也可以使用 Self Play 让系统根据 lead profile 自动生成用户消息。这一点非常重要，因为智能体的价值不能等到上线之后再验证。POC 阶段就应该能测试 opening、语言切换、FAQ 回答、异议处理和后续动作是否达到标准。对外呼 Agent 来说，Test 不是“可有可无的 playground”，而是上线前必要的质量门槛。

Monitor 是运行观察层。它已经不只是传统日志，而是 request-based 的 phase debugger，可以按请求查看 Guideline Match、Tool Calls、Generation 和 Analysis。这种设计不是为了“好看”，而是为了让调优真正有抓手。团队不仅能看到模型最后说了什么，还能看到这一轮上下文是如何被装配出来的，哪些 guideline 被命中、哪些工具被调用、哪些知识被带入、哪一步最慢、哪一步判断最重。对于 AgentOps 来说，这种可观察性是系统化调优的前提。

Deploy 是运行绑定与渠道暴露层。现在的实现仍然偏轻量，但产品方向已经清晰：一个 Agent Definition 可以绑定到共享 runtime gateway，再决定通过哪些 channel 对外暴露。这里借鉴了 OpenClaw 一类产品将 gateway 作为统一入口层的思路，不过 iMorph.ai 会把它和 Agent Definition、Build/Test/Monitor 紧密绑定，而不是把部署做成孤立的技术页面。

Insights 是对话资产层。对话一旦发生，就不应该只是测试记录或临时日志。它们应该被持久化、过滤、标记、分类，并成为未来优质语料、测试集和 Suggestion 生成的基础。目前系统已经支持 Conversation 的持久化、人工评分、Self Play 标记，以及在 Insights 中进行回看。

Suggestions 是离线优化层。它的目标不是只生成“分析报告”，而是直接给出针对 Agent 构建元素的下一步建议。系统当前已经按 `Knowledge Gap`、`Guideline Gap` 和 `Ops Suggestion` 这三类组织建议，并为将来的 evidence-first reviewer 奠定了结构基础。也就是说，未来 Suggestion 不会只是“AI 觉得这里有问题”，而会是“根据 conversation、trace、queue、rating 和 diagnostic rules 提取出的证据，再由模型总结出的建议”。

## 五、产品战略：从构建、测试、部署到持续优化的闭环

如果用一句话概括 iMorph.ai 的产品逻辑，那就是：**Build 定义智能体，Test 验证智能体，Monitor 解释智能体，Insights 积累数据资产，Suggestions 推动下一轮优化，Deploy 让同一智能体可被多个渠道复用。**

这套闭环特别适合销售与外呼团队。因为这类团队不会满足于一个“会说话”的 agent，他们最终关心的是更高的接通质量、更好的转化、更少的无效会话、更准确的后续动作，以及更快的调优周期。也就是说，智能体只有进入一个闭环，才能产生复利。一次 conversation 会进入 Insights；Insights 会影响 Suggestion；Suggestion 会影响 Build；Build 的变更会回到 Test 和 Monitor 验证；通过后再进入 Deploy。这才是一个可持续迭代的产品，而不是一个一次性交付的机器人项目。

Fin 的公开产品叙事也强调类似的闭环，这正是它给 iMorph.ai 最大的启发之一：不是把 AI agent 当成一个单独产品，而是把它当成一个不断训练、不断模拟、不断部署、不断分析的系统。iMorph.ai 在这个方向上会进一步强化“会话上下文工程”的重要性，并将其应用在 outbound 与 sales 上。

## 六、如何在本地运行这个仓库

这个仓库由一个 Python 后端和一个 React + Vite 前端组成。后端负责 Parlant runtime、Agent Definition 持久化、会话与评分存储、Monitor、Suggestions 等功能；前端负责 Build、Test、Monitor、Deploy、Insights 以及文档页。

要运行后端，首先需要在环境中提供 `OPENAI_API_KEY`。如果你希望使用非默认的 OpenAI-compatible provider，也可以同时设置 `OPENAI_BASE_URL`。项目根目录下的 `.env` 会在启动脚本中自动读取，而后端默认把 `OPENAI_BASE_URL` 设为 `https://openrouter.ai/api/v1`。完成这些准备后，直接在仓库根目录执行：

```bash
./start_insurance_demo.sh
```

这个脚本会启动本地服务，并使用 `insurance_outbound_demo.scripts.run_local_server` 作为入口。服务默认运行在 `http://127.0.0.1:8800`。启动过程中，系统会准备本地持久化目录，加载 lead-backed customers，确保保险外呼 demo agent 已经 provisioned，并同步 runtime binding。当前 demo 已经启用了本地 session 持久化、监控 trace 聚合和 deterministic opening fast path。

要运行前端，进入 `web-app` 目录，安装依赖并启动 Vite：

```bash
cd web-app
npm install
npm run dev -- --host 127.0.0.1
```

前端默认运行在 `http://127.0.0.1:5173`。如果没有额外配置，前端会自动把 API 请求指向 `http://127.0.0.1:8800`。如果你要切换后端地址，可以通过设置 `VITE_PARLANT_BASE_URL` 来完成。

启动完成后，可以在浏览器里打开前端页面。你可以在 Build 中查看 Agent Definition，在 Test 中创建对话，在 Monitor 中看 request phases，在 Deploy 中绑定 runtime 和切换 gateway，在 Insights 中查看持久化对话，在 Suggestions 中查看诊断规则与建议。

## 七、开发者应该如何理解这个仓库

从开发者角度看，这个仓库不是一个干净的开源 SDK 项目，而是一个快速演化中的产品试验场。它既包含可运行的 demo，又包含逐步逼近产品化的结构设计。后端主逻辑位于 `insurance_outbound_demo` 目录下，里面包括 API 组合、Agent Definition 持久化、conversation 聚合、diagnostic rules、runtime bindings、suggestions 生成等模块。前端则位于 `web-app`，实现了 Agent Studio、Test、Monitor、Deploy、Insights、Suggestions 以及博客渲染系统。

当前代码仍然保留明显的 POC 特征。例如，有些运行时逻辑仍集中在 API 层，有些存储仍采用本地 JSON 文件，有些构建/绑定/部署语义仍在产品演进中，测试覆盖率也还不足。但这并不妨碍它成为一个很好的实验底座。事实上，这个仓库最大的价值就在于：它已经把“产品思路”和“运行原型”拉通了。接下来真正要做的，是把这些思路从 POC 结构逐步收敛成更稳健、更模块化、更可部署的系统。

## 八、下一步应该做什么

下一步最重要的不是再堆更多功能，而是继续把系统做扎实。首先，后端需要从 POC 结构走向更清晰的分层。Definition、Runtime、Deploy、Insight、Suggestion 之间的边界已经在产品上出现了，但代码上仍需要进一步解耦。特别是 Agent Definition、Runtime Binding、Suggestion Evidence Pipeline 这些对象，最终都应该被做成更正式的 domain model 和 service layer，而不是继续以 ad hoc 的方式扩展。

其次，Build 需要一条更低门槛的 authoring 路径。当前的 Agent Studio 已经能很好地暴露 Parlant 的建模元素，但对非技术运营来说，直接面对 Journey、Guidelines、Retrievers、Variables 等对象仍然偏重。未来需要一个更友好的 onboarding / scenario composer / wizard，将自然语言输入自动拆解成 Agent Draft，再交给 Build 进行审阅和编辑。这件事在产品上极其重要，但也非常难，因为它必须在降低门槛的同时保留控制力。

再次，Deploy 还只是共享 runtime gateway 的起点，距离真正的多渠道发布控制台还有距离。未来需要增加 channel-specific 配置、凭证管理、发布状态、rollout 与 health 机制，并与 Monitor 形成更完整的 runtime 与 channel 观察能力。

最后，Suggestions 需要从“诊断规则 + 启发式建议”走向更完整的 evidence-first reviewer。也就是说，系统应该先通过 conversations、ratings、monitor traces、queues 和 outcomes 找到证据，再把这些结构化证据包交给模型，由模型负责总结与建议，而不是直接让模型自由发挥。这是 Suggestions 未来能否成为可信优化系统的关键。

## 九、参考资料

iMorph.ai 的方向受到了两个主要来源的启发。第一个是 Parlant，它为会话智能体提供了非常清晰的建模与动态上下文工程思路，尤其是 Guidelines、Journeys、Retrievers、Glossary 和 Variables 这些概念。第二个是 Fin，它在产品层面强调 Train / Test / Deploy / Analyze 的连续闭环，让智能体不再只是“一个聊天窗口”，而是成为一个可以被系统化运营和优化的产品对象。

如果你想理解 iMorph.ai 背后的理念，建议从这些资料开始阅读：

- [Fin](https://fin.ai)
- [Parlant Guidelines](https://www.parlant.io/docs/concepts/customization/guidelines/)
- [Parlant Journeys](https://www.parlant.io/docs/concepts/customization/journeys/)
- [Parlant Retrievers](https://www.parlant.io/docs/concepts/customization/retrievers/)
- [Parlant Glossary](https://www.parlant.io/docs/concepts/customization/glossary/)
- [Parlant Variables](https://www.parlant.io/docs/concepts/customization/variables/)

这份中文 README 并不是一份最终对外文案，而更像一份当前阶段的产品说明书。它既解释 iMorph.ai 现在是什么，也解释它将来为什么应该成为一个更完整的 AgentOps 系统。
