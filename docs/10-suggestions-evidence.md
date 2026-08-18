# A Request Failed. What Should You Fix?

When a customer request fails to produce the right answer or trigger the right next action, the easiest response is to reopen the prompt and start editing. But in enterprise conversations, failure is an end-to-end business problem. The cause may be missing knowledge, an incorrect conversation step, or a follow-up that never happens. iMorph Suggestions helps teams identify what actually needs to change.

## Start with a familiar business scenario

An insurance customer asks an Agent:

> "I'm moving to another state next month. Can I keep my current policy? Will my premium change?"

The Agent gives a polished response: insurance requirements may vary by location, so the customer should speak with an advisor.

Technically, the answer is not wrong. In practice, it does not help the customer move forward.

Should the team add another instruction telling the Agent to be more specific? Maybe. But before changing anything, they need to understand where the failure came from.

## Knowledge Gap: The Agent does not have the answer

The first possibility is simple: the Agent does not have the information it needs.

The knowledge base may contain only general policy information, with nothing about moving between states. The right content may exist, but the Retriever may not surface it. Or the customer may have used a common industry abbreviation that is missing from the Glossary.

In any of these cases, telling the Agent to "give an accurate answer" will not help. A prompt cannot supply facts the Agent was never given.

A Knowledge Gap points the team to the knowledge layer. The right fix may be to add or update an FAQ, revise a business document, define a missing Glossary term, or change what the Retriever can access and how it retrieves it. The original question should then be tested again, along with alternative wording and follow-up questions.

The goal is not to make the knowledge base bigger. It is to make sure the knowledge base covers what customers are actually asking.

## Guideline Gap: The Agent has the knowledge but takes the wrong next step

The second possibility is that the information already exists, but the Agent does not follow the right process.

To determine whether the policy can continue, the Agent may first need to confirm the customer's destination state, intended effective date, and current coverage. It may then need to call an eligibility or quoting tool. Instead, the Agent asks nothing, triggers no tool, and falls back to a safe but unhelpful response.

This is not missing knowledge. It is a Guideline Gap.

Here, the team needs to inspect the Agent's behavior: Does the Guideline define the right trigger and expected action? Is the Journey missing a required step? Is the Tool called at the right moment? Is information already provided by the customer stored as a Variable and reused later in the conversation?

After the change, it is not enough to check whether the final response sounds better. The team should confirm that the Agent asks the right questions, follows the intended Journey, calls the correct Tool, and does not disrupt scenarios that were already working.

## Ops Suggestion: The conversation ended, but the business did not pick it up

There is a third possibility: the Agent answers correctly and identifies clear interest in continuing coverage or receiving a quote, but nothing happens after the conversation.

Perhaps the lead never reaches the service queue for the customer's new location. Perhaps the handoff packet is missing the destination state or desired effective date, forcing the next representative to ask the same questions again. Or perhaps the customer requests a call from a person, but no owner or follow-up deadline is assigned.

Improving the Agent's wording will not solve any of these problems. An Ops Suggestion focuses on what should happen after the conversation: whether the lead needs a higher follow-up priority, whether it should be routed to a different team, whether the handoff contains enough context, and whether an outcome requires human attention.

It is a reminder that an Agent's job does not end when the conversation ends. Much of the business value is realized only when the follow-up, handoff, or operational action is completed.

## Every suggestion should answer one question: Why?

If the Suggestions module did nothing more than ask a model to read a few conversations and produce plausible recommendations, it would quickly become another report that nobody trusts.

The principle behind iMorph Suggestions is evidence first. A recommendation worth acting on should be traceable to the underlying conversations, ratings, Monitor traces, queues, and outcomes. Teams should be able to see where the problem occurred, whether it forms a repeated pattern, what happened at runtime, which Agent component may be involved, and how a change can be verified.

A useful Suggestion does more than say, "This could be improved." It gives the team a clear path: find the problem, identify the right target, make a focused change, and test the result against the original failure and the broader regression tests.

## Do not start with the prompt. Start with the actual problem.

Enterprise Agents do not become reliable through one round of configuration. Reliability comes from finding, fixing, and testing issues as real conversations reveal them.

If the Agent does not know, address the Knowledge Gap. If it knows but does not act correctly, address the Guideline Gap. If the conversation succeeds but the business fails to follow through, act on the Ops Suggestion.

That is the product logic behind iMorph Suggestions. A failure should not end with the vague conclusion that "the model performed poorly." It should become a specific, credible, and verifiable next action. Each resolved issue should leave the Agent with more complete knowledge, more reliable behavior, or a stronger operational handoff.
