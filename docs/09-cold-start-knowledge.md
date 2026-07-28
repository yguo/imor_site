---
title: "Cold-Start Knowledge Engineering: How to Make a New Agent Useful Quickly without Historical Data"
date: "2026-02-03"
tags: ["Product Thinking"]
slug: "cold-start-knowledge"
---

# Cold-Start Knowledge Engineering: How to Make a New Agent Useful Quickly without Historical Data

Enterprise AI products face a problem that nearly every team encounters but few products take seriously: **a new customer's agent has no historical data, so what makes it perform well?**

A classic story about "learning AI" is that the system continuously improves by accumulating real conversation data and gets better with use. That story is true for a customer that has been operating for six months. But it is not true on day one for a newly signed customer. They have no historical conversations, no Bad-case samples, no validated Guideline library, and no tested knowledge system.

This is the **cold-start problem**: how can a brand-new agent reach a "useful enough" level quickly without any historical data?

## Three Sources of Cold-Start Gaps

The knowledge cold-start problem for an enterprise agent actually comes from three different gaps:

**First, a process knowledge gap.** The system does not know the customer's business process. How many steps are in the appointment flow? Which calls require an immediate human handoff? How much advance notice is required to cancel an appointment?

**Second, a domain knowledge gap.** The system does not know the customer's specific business content. Which services do they offer? What are the prices? Which regions do they cover? Which insurance plans do they accept?

**Third, a boundary knowledge gap.** The system does not know what the customer "must not say." Which topics require careful handling? Which questions must be handed off to a human? Which expressions do not fit the brand voice?

Traditional AI products respond by asking customers to configure everything themselves: fill out questionnaires, upload documents, and write rules manually. This is the fundamental reason the cold-start experience is poor. Customers must define a system they have never operated before, starting from zero without any feedback.

## iMorph.ai's Cold-Start Strategy: Industry Knowledge before Customer Knowledge

iMorph.ai's core approach to cold start is: **pre-build general industry knowledge into reusable knowledge bases, so a new customer's agent stands on accumulated knowledge instead of starting from zero.**

This approach splits the cold-start problem into two parts:

**Industry knowledge:** This knowledge applies to every customer in the same industry. HIPAA requirements for dental practices, common insurance questions, and the general framework for appointment scheduling do not need to be defined again for every new customer. They should be built into industry templates in advance.

**Customer knowledge:** This knowledge is unique to each customer. The practice name, specific services, prices, opening hours, and accepted insurance plans must come from the customer, but they can be entered through structured fill-in-the-blank forms rather than free-form configuration.

When a new dental practice signs up, it does not load a blank page. It loads an **agent template preloaded with industry knowledge** and only needs to enter its own specific information.

## The Structure of an Industry Template: Three Knowledge Layers

A complete industry template contains three layers of knowledge:

**Layer 1: General industry Guidelines, or prebuilt behavior rules**

These are explicit models of business rules common to the industry, validated by the iMorph.ai team across multiple customer deployments:

```yaml
# General dental practice Guidelines (example)
- id: "dental_hipaa_identity"
  trigger: "User requests appointment information"
  rule: "Verify name and date of birth before providing any appointment details"

- id: "dental_emergency_escalation"
  trigger: "User mentions severe toothache, facial swelling, or bleeding"
  rule: "Identify as an emergency immediately, provide the practice emergency number, and recommend care"

- id: "dental_insurance_boundary"
  trigger: "User asks whether insurance covers a treatment"
  rule: "Explain that human verification is required, provide the front-desk contact, and make no specific promise"
```

These Guidelines come from real deployment experience. They solve problems that every dental practice encounters rather than the special needs of one particular practice.

**Layer 2: General industry Journeys, or workflow skeletons**

The industry template includes prebuilt Journey state machines for the primary business workflows:

```
New patient appointment:
  Opening → Collect identity → Select service → Check calendar → Confirm time → Confirm contact details → Send confirmation

Existing patient rescheduling:
  Opening → Verify identity → Find original appointment → Select new time → Confirm update

Emergency call:
  Opening → Rapid symptom assessment → [Urgent] Immediate handoff / [Non-urgent] Schedule earliest available time
```

These Journey templates cover 80% of dental practice calls. New customers do not need to describe the workflows again; they only need to confirm or fine-tune them.

**Layer 3: Industry Glossary, or terminology library**

A prebuilt industry terminology library ensures that the agent understands industry-specific terms in the same way as the customer:

```yaml
# Dental Glossary (example)
- term: "Teeth cleaning"
  canonical: "Prophylaxis"
  note: "A standard service for a six-month checkup, usually covered by insurance"

- term: "Root canal"
  canonical: "Root Canal Therapy"
  note: "Common in emergency scenarios; confirm whether acute pain is present"

- term: "PPO/HMO"
  canonical: "Dental insurance plan type"
  note: "Common abbreviations when users mention insurance; distinguish in-network and out-of-network coverage"
```

## SOP-to-Agent Compilation: Turning Existing Documents into Structured Knowledge

Many customers already have an SOP document or talk-track manual, but it is an unstructured Word document or PDF that the system cannot use directly.

iMorph.ai's SOP compilation feature automates this process:

**Input:** The customer's existing SOP documents, FAQ lists, training manuals, and other unstructured materials.

**Processing:**

1. Extract business-stage descriptions from the documents and map them to Journey states.
2. Extract behavior rules and cautions and convert them into Guidelines.
3. Extract domain terms and definitions and add them to the Glossary.
4. Flag knowledge gaps, such as "The document does not mention X; the customer needs to provide it."

**Output:** An initial agent configuration ready for deployment, accompanied by a list of information that still needs to be added.

This turns "converting an SOP into an agent" from a multi-week project requiring an AI engineer into work an operations professional can complete in a few hours.

## The First Self-Play Round: Use Simulation Instead of Waiting

After a new agent is configured, the next cold-start challenge is that there is no way to know which configurations are wrong before real customers arrive.

The traditional approach is to wait: wait for real conversations to accumulate, wait for problems to surface, and wait for operations staff to discover and fix them. This cycle often takes 2–4 weeks.

iMorph.ai's Self-Play breaks this waiting cycle during cold start:

**Run a Self-Play batch covering the scenario matrix immediately after configuration is complete, with 100–200 conversations, to find problems before real users arrive:**

- Missing Guideline: A type of call has no corresponding handling rule.
- Journey dead end: A user intent causes the conversation to get stuck and prevents progress.
- Knowledge gap: The agent gives an uncertain or incorrect answer to a type of question.

These problems are discovered, flagged, and prioritized in the sandbox, allowing operations teams to fix the most important issues before launch.

## Product Design Goal for Cold-Start Speed

iMorph.ai treats cold-start speed as one of its core product constraints:

**Goal: from registration to the first inbound call successfully handled by the agent in less than 24 hours.**

The corresponding engineering decisions are:

- Preloaded industry templates: usable immediately after registration without configuration from scratch.
- Fill-in-the-blank onboarding: enter customer-specific information in 15 minutes.
- Automated Self-Play validation: run a baseline scenario validation immediately after configuration, completing it in 30 minutes.
- Progressive optimization: use the industry template as the foundation on day one, then let real conversation data from the first week drive Suggestions.

The design is based on one insight: **customers will not choose you because an agent is perfect on day one, but they will abandon you if launch is too complicated.**

The quality of the cold-start experience directly determines trial conversion. An agent that successfully answers its first real call on the day of registration creates the psychological anchor "it actually works," which no sales pitch can replace.

---

Cold-start knowledge engineering is not an isolated technical problem. It sits at the intersection of product design, industry knowledge, and user experience. The quality of industry templates determines the new customer's starting point. The smoothness of SOP compilation determines configuration friction. The breadth of Self-Play coverage determines the possibility that an agent is already broadly reliable before launch.

Together, these three elements form a true cold-start solution. The system does not wait for data to accumulate; it starts with enough knowledge and confidence to handle the first real business call from day one.

---
