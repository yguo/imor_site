# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **documentation-only** repository containing product narrative, technical best practices, and architecture notes for iMorph.ai. There is no executable code here.

The actual implementation lives at `/Users/yguo/Documents/Dev/parlant_w/`.

## Running the Implementation

All commands apply to the `parlant_w` repo, not this one.

**Backend** (starts on `http://127.0.0.1:8800`):
```bash
./start_insurance_demo.sh
```
Requires `OPENAI_API_KEY` in environment. Defaults `OPENAI_BASE_URL` to OpenRouter unless overridden.

**Frontend** (starts on `http://127.0.0.1:5173`):
```bash
cd web-app
npm install
npm run dev -- --host 127.0.0.1
```
Frontend defaults `VITE_PARLANT_BASE_URL` to `http://127.0.0.1:8800`.

## Product Architecture

iMorph.ai is an AgentOps platform built around Parlant for **outbound sales and qualification agents**. The core thesis: agent quality comes from dynamically assembling the smallest useful context units at runtime, not from a large static system prompt.

### Product Loop
**Build → Test → Monitor → Deploy → Insights → Suggestions** (continuous improvement cycle)

### Parlant Modeling Elements

The backend decomposes every agent into these runtime-assembled units:

| Element | Role |
|---|---|
| **Journey** | Multi-turn flow progression — answers "where is the agent in the process now" |
| **Guideline** | Conditional behavioral rules — answers "what should the agent do in this context" |
| **Glossary** | Domain vocabulary — stabilizes terminology across turns and languages |
| **Tool** | External system actions — "what the agent needs to do" (write outcomes, trigger handoffs) |
| **Retriever** | Knowledge grounding — "what the agent needs to know" (product FAQs, policy docs) |
| **Variable** | Cross-turn state — language preference, budget signals, intent status |
| **Canned Response** | Stable phrasing for high-sensitivity moments (opening lines, handoff, compliance) |

Key distinction: **Retriever = knowledge context** (agent should know X); **Tool = business action** (agent should do X).

### Insurance Demo Backend Structure (`parlant_w/insurance_outbound_demo/`)

Agent assembly order at startup (`app.py`):
1. `add_domain_glossary(agent)`
2. `add_global_guidelines(agent)`
3. `add_objection_guidelines(agent)`
4. `add_long_tail_guidelines(agent)`
5. `create_qualification_journey(agent)` — reads SOP steps, creates Journey states + scoped guidelines

Runtime flow:
- Frontend passes `lead_id` in session metadata
- Journey first step loads lead profile via `load_lead_profile` tool
- Same Journey/Guideline structure produces different behavior per lead

Key files:
- `scripts/run_local_server.py` — entry point
- `api.py` — demo routes + session creation with `allow_greeting=true`
- `journeys/qualification_journey.py` — main flow (opening → coverage → discovery → intent → handoff/close)
- `guidelines/` — global_rules, objection_handling, long_tail_guidelines, sop_enforcement
- `tools/lead_tools.py` — load_lead_profile, record_lead_outcome, record_follow_up_plan, record_handoff_packet
- `sop/qualification_sop.py` — business SOP source of truth (not a Parlant object; compiled into Journey + Guidelines)
- `lead_directory.py` — loads `data/leads.json`, creates one Parlant customer per lead

### Monitor / Observability

Monitor uses OpenTelemetry spans. The `duration_ms` shown is a sum of nested spans (not wall-clock time — parent spans include child spans, so the sum overcounts). Use `ended_at - started_at` for true latency.

Four observable phases per request: **Guideline Matching → Tool Calls → Message Generation → Response Analysis**

Key tuning insight: guideline scope matters more than count. Guidelines that participate in every turn (including opening) make the system heavy. Scope guidelines to the appropriate Journey state to reduce matching cost.

### Insights Architecture

**Conversations** — persisted sessions with transcript, metadata, self-play flag, ratings, outcomes

**Suggestions** use an evidence-first approach (not raw LLM reflection):
1. Diagnostic rules scan conversations, ratings, monitor traces, queues
2. Structured evidence package assembled
3. LLM generates suggestion from evidence

Three suggestion categories: **Knowledge Gap** (retriever/glossary gaps) → **Guideline Gap** (journey/rule/tool gaps) → **Ops Suggestion** (follow-up, handoff, routing issues)

## Document Index

| File | Purpose |
|---|---|
| `README.md` | Product thesis, GTM, setup instructions, architecture overview |
| `README.zh.md` / `iMorph Intro.md` | Chinese product narrative (website copy) |
| `Why iMorph.md` | Product positioning vs. Fin, Sierra, Cresta, Palantir |
| `Build iMorph.md` | AgentOps methodology, Build/Test/Monitor/Deploy/Insights design rationale |
| `Best_Practice_Insurance_Outbound.md` | Parlant modeling deep dive for the insurance demo |
| `Vibe POC Best Practice.md` | POC methodology and tuning lessons |
| `call_sequence.md` | Trace analysis for a slow Chinese insurance reply |
