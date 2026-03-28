# iMorph.ai

iMorph.ai is an AgentOps product for building, testing, observing, and improving outbound and sales agents. The ambition is similar in shape to [Fin](https://fin.ai): one system that helps teams train, test, deploy, and analyze agents as part of a continuous operating loop. The difference is the center of gravity. iMorph.ai is not being shaped primarily around inbound support. It is being shaped around outbound sales, proactive qualification, follow-up orchestration, and the peculiar demands of multi-turn commercial conversations where timing, tone, segmentation, and next-best action matter as much as factual correctness.

This repository is the current product and engineering workspace for that idea. It combines a Parlant-powered backend, a React workspace for Build/Test/Monitor/Deploy/Insights, a persistent local demo environment, and a set of product experiments around agent drafting, runtime binding, offline conversation analysis, and suggestion generation. It is still a POC in code quality, but the product thesis is already clear: if outbound AI is going to work in the real world, it cannot be a single static prompt with a thin chat shell around it. It needs context engineering, runtime control, observability, and a feedback system that can continuously tune the agent as the business changes.

## Product Thesis

The core belief behind iMorph.ai is that outbound and sales conversations are a special category of AI application. They are not just question-answer exchanges. They are stateful commercial interactions with changing stakes across the conversation. A lead may start skeptical, become curious, ask about terms, reveal constraints, require handoff, or disappear into a follow-up queue. The agent has to decide what matters now, what must be remembered, what should be asked next, what should be loaded into context, and what should remain out of the prompt. In practice, this means the hardest problem is not merely language generation. The hardest problem is conversation-specific context assembly.

This is where Parlant is especially relevant. Parlant’s documentation frames the problem as one of dynamic context delivery rather than one large, static system prompt. Its guidelines model exists to deliver the “right” instruction set into the model only when the condition is relevant, because excessive prompt accumulation causes attention dilution and degraded compliance. Parlant’s journeys model similarly treats multi-turn flows as adaptive SOPs rather than rigid flowcharts, allowing the agent to follow structured process without becoming robotic. That perspective is foundational to iMorph.ai. The product is being built around the idea that higher agent quality comes from dynamically composing the smallest useful units of context at runtime instead of overloading the model with everything all at once. See the Parlant docs on [Guidelines](https://www.parlant.io/docs/concepts/customization/guidelines/), [Journeys](https://www.parlant.io/docs/concepts/customization/journeys/), [Retrievers](https://www.parlant.io/docs/concepts/customization/retrievers/), [Glossary](https://www.parlant.io/docs/concepts/customization/glossary/), and [Variables](https://www.parlant.io/docs/concepts/customization/variables/).

The product architecture therefore centers on a small set of explicit modeling elements. Journeys encode structured multi-turn intent progression. Guidelines encode conditional behavioral rules. Glossary terms define business vocabulary so both customers and builders can refer to domain-specific concepts consistently. Retrievers ground the model with knowledge it should know in this turn, while tools execute operations it should do. Variables preserve state that should persist across turns or refresh on a schedule. Canned responses provide stable phrasing for high-sensitivity moments such as opening lines, handoff, or compliance language. Taken together, these elements form a runtime context system rather than a monolithic prompt. That is the core moat: better context engineering for commercial conversations.

## Why Outbound and Sales Are the Focus

The commercial opportunity for AI agents is enormous, but much of the category still concentrates on support automation. Outbound and sales deserve their own operating model. In outbound motion, the system is not merely reacting to an incoming issue. It is proactively contacting a lead, qualifying interest, understanding objections, selecting tone based on region and channel, deciding when to push and when to back off, and orchestrating downstream actions such as follow-up, appointment booking, handoff, or disqualification. The cost of getting this wrong is high: damaged trust, poor conversion, wasted SDR cycles, and unusable transcript data.

That makes outbound an unusually fertile domain for an AgentOps product. The workflows are repetitive enough to benefit from systemization, but nuanced enough that naive prompt-only designs fail quickly. Teams need a place to define how an agent should behave, test that behavior before rollout, inspect the real runtime reasoning path, and learn from conversations after the fact. Fin’s public product language captures this well through a Train/Test/Deploy/Analyze loop, where the product is not just the agent itself but the full continuous improvement cycle around it. iMorph.ai adopts the same macro-loop, but orients it toward sales and outbound pipeline building rather than support deflection alone. Fin’s framing is visible on its homepage, where it describes a “continuous improvement loop” of training on procedures and policies, simulating behavior before launch, deploying across channels, and analyzing performance afterward. See [Fin](https://fin.ai).

## What iMorph.ai Includes Today

Today’s repository already expresses that loop in product form. Build is where agent definitions live. Test is where the selected runtime-ready agent can be exercised against leads, with manual input or self-play. Monitor is where the request path is inspected phase by phase, including guideline matching, tool calling, generation, and response analysis. Deploy is where a selected agent owns the shared runtime gateway before channels are exposed. Insights persists conversations and supports review, filtering, and operator evaluation. Suggestions sits on top of offline evidence and diagnostic rules to propose where the agent or operating process should be improved next.

Even at the POC stage, that structure matters. It means the product is not treating the model as the only artifact. The real artifact is the operating system around the model: the definition, the runtime binding, the channel exposure, the monitoring path, the conversation corpus, the suggestions loop, and the eventual business outcome.

## The Commercial Map

The likely commercial shape of iMorph.ai is not “another chatbot builder.” It is closer to a system of record and control plane for conversational revenue operations. The natural buyer is a team that already runs SDR, inside sales, lead qualification, or appointment-setting workflows and wants AI to handle a larger share of first contact and structured follow-up without surrendering control over messaging or process. In that sense, the product can grow from an outbound sales wedge into a broader conversational operations suite.

The first commercial surface is outbound qualification and invitation flows, such as insurance outreach, EV test-drive invitations, financing prequalification, or service reminder campaigns. The second surface is multi-channel deployment, where the same agent logic can be exposed through web entry points, internal terminal tooling, and enterprise messaging channels such as Feishu, DingTalk, and WhatsApp. The third surface is post-conversation intelligence: identifying what the agent did well, what it missed, which knowledge should be added, which guidelines are failing, and which leads should be routed differently. Over time, this creates a product that spans build-time authoring, runtime orchestration, and post-run optimization.

## Why the Technical Moat Matters

The strongest moat here is not simply having an LLM in the loop. The moat comes from treating conversation quality as a context-engineering problem with operational controls. Parlant is useful precisely because it breaks the problem into conditionally active components. Its documentation distinguishes between data the agent should “know” via retrievers and data or operations it should “load” or “do” via tools. It also makes journeys adaptive instead of rigid, which is important for sales conversations because the customer does not advance in a perfect flowchart. That design leads naturally to a product where different elements are loaded or excluded based on context, instead of every rule living in a giant system prompt.

This matters commercially because outbound agents live or die on edge cases. A support bot that is slightly verbose is annoying. A sales or outbound bot that is slightly wrong can quietly destroy conversion. Dynamic context loading, scoped guidelines, journey-aware behavior, glossary-backed terminology, and runtime variables are not implementation details. They are the difference between a demo and a production-worthy commercial agent.

## Go-To-Market

The practical GTM motion for iMorph.ai should start narrow and opinionated. The product is strongest when the target workflow is structured, repetitive, and revenue-adjacent. A wedge such as insurance outbound qualification, automotive test-drive invitation, or financial pre-screening allows the team to demonstrate concrete value quickly: more qualified conversations, faster operator follow-up, better transcript quality, and a measurable reduction in workflow inconsistency. That creates the right path to expansion. Once the Build/Test/Monitor/Insights loop proves itself in one outbound motion, the same operating model can extend to adjacent sales and lifecycle use cases.

The messaging should therefore emphasize three things. First, iMorph.ai is a way to launch AI agents that can be governed, not just prompted. Second, it is designed for sales and outbound teams that care about structured commercial outcomes, not only support resolution. Third, it gets better through its own data exhaust: monitored requests, rated conversations, offline clustering, and suggestion generation. The product story should feel less like “chat with your data” and more like “build a revenue-grade conversational system that you can tune over time.”

## Running This Repository Locally

This repository is designed to run locally with a Python backend and a Vite frontend. The backend uses a local Python virtual environment that already includes Parlant, and the frontend is a standalone React workspace in `web-app`. By default, the local launcher reads environment variables from `.env`, stores demo data under `parlant-data-insurance-demo`, and points the NLP layer to an OpenAI-compatible endpoint. The demo currently defaults `OPENAI_BASE_URL` to OpenRouter unless you override it.

To run the backend, make sure your environment exposes an `OPENAI_API_KEY`. If you want to use a different compatible provider, you can also set `OPENAI_BASE_URL`. Then start the demo server from the repository root:

```bash
./start_insurance_demo.sh
```

The backend will start on `http://127.0.0.1:8800`. It provisions local lead-backed customers, binds the insurance outbound runtime agent, enables local persistence for sessions and variables, and turns on the monitor span pipeline. The API layer also exposes demo-specific endpoints for agent definitions, conversations, suggestions, monitor traces, and runtime bindings.

To run the frontend, install the web dependencies and start Vite from the `web-app` directory:

```bash
cd web-app
npm install
npm run dev -- --host 127.0.0.1
```

The frontend runs on `http://127.0.0.1:5173`. It defaults `VITE_PARLANT_BASE_URL` to `http://127.0.0.1:8800`, so the local frontend and backend work together without extra configuration. If you want to point the frontend at another backend, set `VITE_PARLANT_BASE_URL` before launching the dev server.

Once both processes are running, you can open the application, select the runtime-ready agent, start a lead session in Test, inspect the request path in Monitor, review persisted sessions in Conversations, and view definition-driven configuration in Build and Deploy.

## How the Codebase Is Organized

The backend lives primarily under `insurance_outbound_demo`. The runtime server entry point is `insurance_outbound_demo/scripts/run_local_server.py`. The API composition and demo-only routes live in `insurance_outbound_demo/api.py`. Persisted definitions, conversations, diagnostic rules, suggestions, and runtime bindings each have their own modules and local JSON-backed storage under `parlant-data-insurance-demo`. The frontend lives under `web-app`, where the React application implements the Build, Test, Monitor, Deploy, Insights, and blog-style docs surfaces.

The codebase is still very much a product lab. Some parts are already converging on a cleaner architecture, especially persisted agent definitions and runtime binding. Other parts still have clear POC characteristics, such as tactical glue in the API layer, file-based storage instead of a database, and limited test coverage. That is acceptable for this stage, but it is also the obvious place to improve next.

## What Should Happen Next

The next step is to make the backend more robust without losing product speed. The most important structural move is to separate definition management, runtime services, analysis pipelines, and infrastructure concerns more clearly. Agent definitions are now persisted, but the boundary between definition, runtime binding, deploy state, and suggestion evidence still needs to become more explicit. The repository should evolve toward a more formal internal model where the build-time artifact and the runtime artifact are clearly distinct, and where suggestions are produced from a repeatable evidence pipeline instead of growing as ad hoc heuristics.

The second major step is to improve the authoring experience. The current Build workflow already exposes the right Parlant concepts, but a production-grade product will need a better path for non-technical operators. That likely means a wizard or scenario composer that converts plain business requirements into draft journeys, guidelines, glossary terms, retrievers, tools, variables, and canned responses. The challenge is to lower the modeling burden without hiding the mechanics so deeply that teams lose control. This remains one of the central open design questions.

The third major step is deployment maturity. The current shared-runtime gateway logic is enough for local experimentation, but production deployment will require a stronger control plane around channel credentials, rollout state, gateway ownership, and channel-specific configuration. Over time, Deploy should feel less like a placeholder and more like a proper multi-channel release surface.

The fourth major step is better offline analysis. Conversations and ratings are already being persisted, and Suggestions now has a diagnostic-rules concept. The natural progression is to formalize an evidence-first reviewer pipeline that classifies conversation issues, collects structured evidence, and only then asks an LLM to summarize and recommend changes. That is how Suggestions becomes a trustworthy optimization tool instead of a stream of generic AI advice.

## References

The product and technical direction in this repository is heavily informed by Parlant’s context engineering model and by the Train/Test/Deploy/Analyze loop visible in Fin’s public product narrative. If you want the conceptual background behind iMorph.ai, start with Parlant’s documentation on [Guidelines](https://www.parlant.io/docs/concepts/customization/guidelines/), [Journeys](https://www.parlant.io/docs/concepts/customization/journeys/), [Retrievers](https://www.parlant.io/docs/concepts/customization/retrievers/), [Variables](https://www.parlant.io/docs/concepts/customization/variables/), and [Glossary](https://www.parlant.io/docs/concepts/customization/glossary/), then compare that with the way [Fin](https://fin.ai) frames training, simulation, deployment, and AI-powered analysis as a continuous improvement loop.
