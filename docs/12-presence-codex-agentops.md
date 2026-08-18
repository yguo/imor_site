# What OpenAI Presence's Codex Improvement Loop Reveals About the Next Generation of AgentOps

The hardest part of an enterprise agent begins after launch.

Users introduce situations the design team did not anticipate. Knowledge becomes outdated. Business policies change. Tools fail. Customer language evolves. An agent that performs well today will not automatically maintain the same quality tomorrow.

OpenAI Presence offers a notable response to this problem.

According to OpenAI's public materials, production sessions, human escalations, and quality signals can reveal gaps in a Presence agent. Codex, using the Presence plugin, investigates those signals and suggests updates. Teams can test proposed changes against the production version before approving a controlled rollout.

This loop exposes a central question for the next generation of AgentOps:

> How can agents discover and repair problems faster while enterprises retain control over evidence, judgment, and release?

iMorph's perspective can be summarized as an evidence-driven improvement chain:

![The evidence-driven agent improvement chain](asssets/presence-agentops-loop.png)

This is not a feature checklist. It is a product principle for next-generation AgentOps: automation should expand observation and shorten improvement cycles, but it should not bypass evidence, evaluation, or accountability.

## 1. Presence describes controlled improvement, not uncontrolled rewriting

OpenAI's public description of Presence is careful and important.

It states that:

- production sessions, escalations, and quality signals can reveal agent gaps;
- Codex, using the Presence plugin, investigates signals and suggests updates;
- teams can test proposed changes against the production version;
- approved changes move through controlled rollout;
- new versions can be monitored and rolled back.

It is easy to misread this as an agent continuously rewriting itself in production.

The process OpenAI describes is closer to:

`Detect signals -> Investigate -> Propose a change -> Test and compare -> Team approval -> Controlled rollout`

Codex accelerates investigation and proposal creation. The enterprise still decides what may change, how it should be validated, and when it should reach production.

The objective of next-generation AgentOps is therefore not simply to remove people. It is to redistribute work:

- systems handle scale, pattern detection, and evidence preparation;
- people evaluate business impact, accept risk, and approve consequential changes.

## 2. Why agents require continuous operations more than traditional software

Most traditional software behavior is explicitly defined in code. Similar inputs tend to follow relatively stable paths, and failures can often be traced through logs, tests, and exceptions.

Agent behavior is produced by multiple dynamic components:

- how the user expresses a need;
- how the model interprets context;
- which knowledge is loaded;
- which policies and rules are triggered;
- the current stage of the task;
- whether a tool is called and what it returns;
- whether the agent should clarify, refuse, or escalate;
- whether the business environment has changed.

“The agent performed poorly” is therefore not a useful diagnosis.

The same visible symptom can have very different causes:

| Visible problem | Possible root causes |
| --- | --- |
| Incorrect answer | Missing knowledge, retrieval failure, conflict, or reasoning error |
| Policy not followed | Missing rule, unclear condition, conflict, or execution failure |
| Tool call failed | Bad parameters, missing permissions, API failure, or inconsistent state |
| Too many handoffs | Poor task boundary, confidence threshold, or escalation policy |
| Missing escalation | Risk-detection failure, policy gap, or insufficient context |
| Overall metric decline | A shift in product, policy, users, or channel distribution |

AgentOps cannot stop at telling a team that an error rate increased. It must help explain where the problem occurred, why it occurred, what should change, and whether the change actually worked.

## 3. The tension between automatic discovery and enterprise control

From an efficiency perspective, an ideal system appears able to:

1. read production conversations automatically;
2. detect anomalies and repeated failures;
3. identify root causes;
4. propose changes to knowledge, rules, or workflows;
5. run evaluations;
6. publish a new version.

The earlier steps are highly valuable. As the number of agents, channels, and sessions grows, people cannot review every interaction manually. Automation is better suited to:

- identifying repeated failures across large volumes;
- detecting rare but high-risk issues;
- spotting metric changes after a release;
- turning production failures into reproducible cases;
- comparing multiple candidate changes.

The risk increases as automation approaches production changes.

A reasonable knowledge update may conflict with a policy. A change that reduces handoffs may increase compliance risk. An optimization that improves the aggregate score may cause regression in a rare but critical scenario.

A local metric improvement is not always an enterprise improvement.

Yet keeping every step manual creates another form of risk. Problems are detected late, repairs take too long, and approvals are made with incomplete context.

Effective enterprise control is not the same as adding more manual steps. It means placing automation and human judgment where each is strongest.

## 4. Evidence: every recommendation should begin with production reality

iMorph believes agent improvement should not begin with “ask the model to rewrite the prompt.” It should begin with Evidence.

Useful evidence is more than a low score or negative comment. It should reconstruct the conditions under which the agent operated:

- user goal and full conversation;
- task outcome;
- context assembled for the request;
- knowledge and sources used;
- rules triggered or missed;
- tool parameters, results, and errors;
- current task stage;
- escalation event and reason;
- configuration and version.

Evidence makes a problem inspectable, replayable, and comparable.

If a recommendation cannot point to the production conversations, traces, or business outcomes that support it, the team cannot distinguish a real diagnosis from a plausible-sounding explanation.

The first responsibility of next-generation AgentOps is not to generate recommendations faster. It is to establish a verifiable factual basis for every recommendation.

## 5. Root Cause: map poor performance to a changeable system object

Evidence explains what happened. Root Cause should explain why.

A low-scoring conversation can involve different categories of failure:

- **Knowledge:** missing, outdated, conflicting, or poorly retrieved information;
- **Policy / Guideline:** a missing, unclear, conflicting, or unexecuted rule;
- **Journey / Workflow:** an incorrect stage, branch, or termination condition;
- **Tool:** a parameter, permission, API, or state issue;
- **Context:** missing, excessive, misordered, or lost context;
- **Model Behavior:** an understanding, reasoning, generation, or instruction-following issue;
- **Handoff:** a poor escalation threshold, timing, or handoff package;
- **Business Change:** a shift in products, policies, or user behavior.

Root Cause prevents every failure from being assigned to “the model” or “the prompt.”

Automated analysis can help produce candidate causes, associate traces, and organize supporting evidence. High-risk cases still require business, technical, or compliance judgment. Models can assist investigation; enterprises must decide which explanations are strong enough to justify production change.

## 6. Change: turn a recommendation into something reviewable

“Improve answer quality” is not an approvable change.

An actionable Change should explain:

- what should be modified;
- the before-and-after difference;
- which production evidence supports the change;
- which scenarios and metrics should improve;
- which other tasks may be affected;
- which risks and uncertainties remain;
- the appropriate rollout scope.

The change object should not be limited to a prompt. It may be:

- a knowledge item;
- a policy or Guideline;
- a Journey branch;
- a tool configuration;
- a permission boundary;
- an escalation rule;
- a model or context strategy.

Systems can help prepare a change, but “generated by a model” does not mean “correct.” Eval must determine whether the change deserves to reach production.

## 7. Eval: test every change against both the target problem and historical failures

Eval is the most important isolation layer between automation and enterprise control.

Every candidate change should answer:

- did it fix the original problem;
- did it pass existing task and policy evaluations;
- did it reintroduce a historical failure;
- did the target metric improve;
- did another important scenario regress;
- did high-risk outcomes remain within policy;
- was the improvement stable rather than accidental?

Evaluation should cover more than response text:

- task completion;
- policy adherence;
- correct tool use;
- information completeness;
- escalation judgment;
- risk events;
- business outcomes.

The most valuable evaluation assets often come from real failures.

When a production issue is diagnosed and resolved, it should become a regression case for future releases. The enterprise is then not merely fixing one problem; it is building a stronger business evaluation system over time.

## 8. Approval: enterprises approve risk and benefit, not text

Approval should be more than a button.

Approvers need to understand:

- why the change is necessary;
- the evidence behind the root-cause judgment;
- what changed;
- which evaluations passed or failed;
- which business outcomes may improve;
- which risks remain;
- how much traffic, which channels, or which users are affected;
- when the release should stop or roll back.

Different changes also require different decision-makers:

| Change type | Relevant decision-makers |
| --- | --- |
| General knowledge correction | Business or knowledge owner |
| Language or workflow adjustment | Product or operations owner |
| Tool write or permission change | Technical and system owner |
| Compliance-policy change | Legal, compliance, or risk team |
| High-risk action or major version change | Multi-role approval |

Approval should not make releases heavier. It should make every consequential decision better informed.

The system prepares the context. People accept the risk.

## 9. Where should the automation boundary sit?

Evidence, Root Cause, Change, Eval, and Approval do not require the same level of automation.

A practical division of responsibility is:

| Stage | Work suited to automation | Human judgment to retain |
| --- | --- | --- |
| Evidence | Collection, clustering, anomaly detection, evidence preparation | Business impact |
| Root Cause | Candidate causes, trace association, confidence ranking | High-risk causal confirmation |
| Change | Draft edits, diffs, impact prompts | Business intent and policy correctness |
| Eval | Batch execution, version comparison, regression detection | Evaluation standards and risk thresholds |
| Approval | Approval package and rollout recommendation | Risk acceptance and final release |

The closer automation gets to production action, the stricter its evidence, permissions, and approval requirements should become.

This leads to a simple principle:

> The system may expand observation, propose hypotheses, prepare changes, and run evaluations automatically; the enterprise must retain explicit control over high-impact changes and production release.

## 10. Next-generation AgentOps is a change-control system, not merely a dashboard

Many AgentOps products still center on dashboards:

- conversation volume;
- success rate;
- latency;
- token usage and cost;
- user ratings;
- handoff rate.

These metrics matter, but they mainly explain what happened.

Next-generation AgentOps must also explain:

- why it happened;
- what should change;
- whether the change worked;
- whether it introduced new risk;
- who approved it;
- which version is live;
- whether post-release results matched expectations.

The central object of AgentOps will gradually move from Metric to Change:

1. Move from a metric anomaly to production evidence.
2. Investigate the evidence and prepare a change set.
3. Use an evaluation report to validate benefit and risk.
4. Create an approval record before controlled release.
5. Return release outcomes to the next evidence cycle.

The dashboard remains an entry point. Enterprise control comes from the complete chain connecting production evidence to release outcomes.

## 11. The iMorph perspective: improve with evidence, build trust through control

iMorph is not focused on making an agent appear to “evolve by itself.” The more important goal is to make every improvement transparent, testable, and controllable by the enterprise.

We believe next-generation AgentOps should have three basic characteristics:

### 1. Every recommendation leads back to evidence

Teams can see which real problems support the recommendation and why it deserves priority.

### 2. Every change faces evaluation

A change must not only fix the target issue; it must also demonstrate that it does not damage other important scenarios.

### 3. Every release preserves enterprise control

The enterprise can decide who approves the change, how broadly it is released, and when it should stop or roll back.

This approach does not try to remove people from every step. It focuses human attention on the moments that require business judgment and accountability.

## Conclusion: trustworthy automated improvement does not bypass the enterprise

The Codex improvement loop in OpenAI Presence sends a clear signal: production agent operations will move beyond monitoring and reporting. Systems will become more active in investigating problems, preparing changes, and organizing validation.

More active does not have to mean less controlled.

The core of next-generation AgentOps is not an agent that continuously rewrites itself. It is a trusted improvement chain:

**Evidence -> Root Cause -> Change -> Eval -> Approval**

Evidence keeps recommendations grounded in reality. Root Cause separates symptoms from problems. Change makes modifications reviewable. Eval prevents local improvements from damaging the wider system. Approval ensures that the enterprise retains final authority.

Models can accelerate discovery and repair. AgentOps must ensure that speed does not come at the expense of evidence, accountability, and governance.

Trustworthy automated improvement does not bypass the enterprise. It helps the enterprise gain clearer, faster, and more reliable control with less manual effort.
