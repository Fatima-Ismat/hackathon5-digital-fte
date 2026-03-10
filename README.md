# Hackathon 5 – Customer Success Digital FTE
## NovaSync Technologies | SyncFlow AI Support Agent

> **Stage 1: Incubation / Prototype**
> This repository is the complete Stage 1 submission for Hackathon 5. It represents the exploration, discovery, and prototype-building phase — not production code. Every design decision documented here was driven by data from sample tickets, iterative prompt refinement, and systematic edge-case testing.

---

## Stage 1 At a Glance

| What | How Much |
|------|----------|
| Customer support tickets analyzed | 55 across 3 channels |
| Knowledge base entries built | 12 structured topic entries |
| MCP tools exposed | 5 (search, ticket, history, respond, escalate) |
| Agent skills defined | 5 with full input/output contracts |
| Escalation trigger conditions | 21 across 3 tiers |
| Escalation queues | 5 specialist queues |
| Edge case test scenarios | 25 with pass criteria |
| Prompt iterations | 10 documented in full |
| Bugs found and fixed in Stage 1 | 4 (documented in discovery log) |
| Python dependencies | 0 (fully self-contained prototype) |

---

## What Problem Are We Solving?

Customer success teams at SaaS companies like NovaSync face a well-known tension: **volume vs. quality**. As the customer base grows, the ticket volume grows faster than the support team can hire. The result is slower response times, inconsistent tone, missed escalation signals, and human agents spending most of their time answering the same 20 questions.

The **Customer Success Digital FTE** is an AI agent designed to handle the full lifecycle of inbound support interactions — from first message to resolution or human handoff — while maintaining the brand voice, respecting SLA commitments, and protecting the business from dangerous responses (refunds, legal, pricing).

The goal for Stage 1 was to prove this is buildable: establish the data, define the rules, prototype the logic, and identify every place it breaks.

---

## What This Prototype Does

The agent handles inbound support queries across three channels:

| Channel | Communication Style | Typical Volume | Complexity |
|---------|-------------------|---------------|------------|
| **Email** (Gmail) | Formal, multi-paragraph, signed | Medium | High |
| **WhatsApp** (Meta Cloud API) | Short, lowercase, conversational | High | Low–Medium |
| **Web Support Form** | Semi-structured, self-triage | Medium | Medium |

**For every inbound message, the agent:**

1. **Identifies the customer** — loads their plan, VIP status, account health, and ticket history from the mock CRM
2. **Detects sentiment** — scores anger, frustration, and urgency using 7 distinct lexical and typographic signals
3. **Checks Tier 1 escalation** — immediately routes if legal, security, pricing, or emotional threshold triggers are met (bypasses KB search entirely)
4. **Searches the knowledge base** — keyword + tag overlap scoring against 12 structured topic entries; returns confidence score
5. **Decides on escalation** — applies 21 rule conditions across 3 tiers; routes to one of 5 specialist queues if escalation is needed
6. **Formats the response** — adapts tone, length, greeting, and structure to the target channel

---

## Incubation Process — How We Got Here

Stage 1 followed 10 deliberate prompt-driven iterations. Each iteration asked a specific question, produced a concrete artifact, and changed something in the system.

| Iteration | Question Asked | Key Change Produced |
|-----------|----------------|-------------------|
| 1 | What capabilities does the agent need? | Architecture expanded from 3 to 6 components |
| 2 | How do customers write differently per channel? | Empirical length limits set; 3 distinct formatters created |
| 3 | What does the KB need to contain? | 12 KB entries built; confidence threshold introduced |
| 4 | What signals indicate anger vs. frustration? | 7-signal sentiment detector; score-based escalation thresholds |
| 5 | When must the agent stop trying to answer? | 3-tier escalation framework; 21 trigger conditions |
| 6 | What backend operations does the agent need? | 5 MCP tools with contracts; 5 specialist queues defined |
| 7 | Why are WhatsApp responses too long? | Word limit enforced; channel formatters tuned with before/after examples |
| 8 | How does the agent handle multi-turn conversations? | `conversation_history` parameter; turn-count escalation |
| 9 | Where does the agent fail? | 4 bugs fixed; 25 edge cases documented |
| 10 | Does the end-to-end pipeline work? | CLI demos added; pipeline verified |

Full iteration detail — including prompts, outputs, and what changed — is in [`specs/prompt-history.md`](specs/prompt-history.md).

---

## System Architecture

```
  INBOUND CUSTOMER MESSAGE
         |
         +------[Email / Gmail]------------+
         +------[WhatsApp / Meta API]------+--> CHANNEL NORMALIZER
         +------[Web Support Form]---------+    (standard message format)
                                                         |
                                                         v
                                          +-----------------------------+
                                          |   CUSTOMER SUCCESS AGENT    |
                                          |                             |
                                          |  Step 1: Customer ID        |
                                          |    Load plan, VIP, health   |
                                          |                             |
                                          |  Step 2: Sentiment Analysis |
                                          |    anger / frustration /    |
                                          |    urgency scoring          |
                                          |                             |
                                          |  Step 3: Tier 1 Check       |
                                          |    legal / security /       |
                                          |    pricing / profanity?     |
                                          |                             |
                                          |  Step 4: KB Search          |
                                          |    confidence scored 0-1    |
                                          |                             |
                                          |  Step 5: Escalation Decision|
                                          |    tiers 1-3, 21 triggers   |
                                          |                             |
                                          |  Step 6: Format Response    |
                                          |    channel-adapted output   |
                                          +----+-------------------+----+
                                               |                   |
                          +--------------------+                   +-------------------+
                          v                                                            v
               +---------------------+                                  +--------------------------+
               |     MCP SERVER      |                                  |   HUMAN AGENT QUEUE      |
               |                     |                                  |                          |
               | search_kb(query)    |                                  |  billing-team            |
               | create_ticket(...)  |                                  |  legal-team              |
               | get_history(cust)   |                                  |  sales-team              |
               | send_response(...)  |                                  |  security-team           |
               | escalate_to_human() |                                  |  enterprise-csm          |
               +---------------------+                                  |  senior-support          |
                                                                        +--------------------------+
```

---

## What the Agent Handles vs. What Always Escalates

### Autonomously Resolved (AI handles end-to-end)

- Password reset and account lockout
- Two-factor authentication setup and troubleshooting
- API error codes — 401 (expired key), 403 (permissions), 404 (not found), 429 (rate limit), 500 (server error)
- API rate limit explanation and plan comparison
- Billing question explanations — invoice retrieval, proration logic, payment methods
- Integration connection guides — OAuth setup, reconnection after token expiry
- Workflow trigger troubleshooting — status check, timezone, webhook registration
- Webhook configuration and duplicate-event debugging
- Team invites, roles, seat limits
- Account ownership transfer instructions
- Data export process
- SSO setup and common error resolution
- Plan feature and limit questions

### Always Escalated (human handles, AI only acknowledges)

| Trigger | Queue | Tier |
|---------|-------|------|
| Legal threats, contract disputes | legal-team | 1 — Immediate |
| Security incidents, account compromise | security-team | 1 — Immediate |
| Refund requests | billing-team | 1 — Immediate |
| Pricing negotiation, discounts | sales-team | 1 — Immediate |
| GDPR / compliance document requests | legal-team | 1 — Immediate |
| Enterprise contract renewal | enterprise-csm | 1 — Immediate |
| Churn threat (Business/Enterprise) | csm-retention | 1 — Immediate |
| Profanity detected | senior-support | 1 — Immediate |
| Anger score >= 0.75 | senior-support | 1 — Immediate |
| Persistent frustration (2+ turns) | senior-support | 2 — After attempt |
| VIP customer with negative sentiment | enterprise-csm | 2 — After attempt |
| Low KB confidence (<0.40) | technical-support | 3 — Judgment |

---

## Repository Structure

```
Hackathon5-Customer-Success-FTE/
|
+-- context/                           Domain knowledge and training data
|   +-- company-profile.md             NovaSync / SyncFlow company overview, SLAs, philosophy
|   +-- product-docs.md                Complete KB source: 8 sections, all major topics
|   +-- sample-tickets.json            55 customer tickets across email / WhatsApp / web form
|   +-- escalation-rules.md            3-tier escalation framework, triggers, queue routing
|   +-- brand-voice.md                 Channel tone guide, banned phrases, before/after examples
|
+-- specs/                             Design specifications and incubation artifacts
|   +-- agent-skills.md                5 skill definitions with inputs, outputs, flow diagram
|   +-- customer-success-fte-spec.md   Full system specification — scope, channels, KPIs
|   +-- discovery-log.md               12 discoveries from Stage 1 data analysis and testing
|   +-- prompt-history.md              10-iteration prompt journal — the incubation evidence
|
+-- src/
|   +-- agent/
|   |   +-- customer_success_agent.py  Core agent: 5 functions, 12 KB entries, CLI demo
|   |   +-- mcp_server.py              5 MCP tools, routing logic, tool registry, CLI demo
|   +-- channels/                      Reserved: Stage 2 channel adapters
|   +-- web-form/                      Reserved: Stage 2 web form integration
|
+-- tests/
|   +-- test_cases.md                  25 edge cases with inputs, expected behavior, pass criteria
|
+-- README.md                          This file
```

---

## Running the Prototype

**Requirements:** Python 3.9+, no external packages.

```bash
# Run the core agent demo — 4 end-to-end test scenarios
python src/agent/customer_success_agent.py

# Run the MCP server demo — tool registry and all 5 tools
python src/agent/mcp_server.py
```

### What the Agent Demo Shows

**Scenario 1 — WhatsApp: Password Reset**
```
Input:    "hi how do i reset my password i forgot it"
Channel:  whatsapp | Customer: Priya Nair (Starter)
Sentiment: neutral | Escalate: No
Response: "Hey Priya! To reset your password: go to app.syncflow.io/login
           -> Forgot Password -> enter your email. Reset link in 2 mins."
```

**Scenario 2 — WhatsApp: Angry Customer**
```
Input:    "THIS IS RIDICULOUS I've been waiting 3 DAYS..."
Channel:  whatsapp | Customer: Unknown (Starter defaults)
Sentiment: frustrated | Escalate: Yes (low_kb_confidence + vague message)
Response: "I'm connecting you with a specialist who can help directly.
           You'll hear back within 24 hours."
```

**Scenario 3 — Email: API Rate Limit (Growth plan)**
```
Input:    "I am getting a 429 rate limit error on all API calls..."
Channel:  email | Customer: Marcus Chen (Growth)
Sentiment: neutral | Escalate: No | Subject: "Re: API Rate Limit -- Next Steps"
Response: Formal email with 429 explanation, rate limits by plan, remediation steps
```

**Scenario 4 — Web Form: Billing Question (Business plan)**
```
Input:    "I noticed a $47.22 charge that I didn't expect..."
Channel:  web_form | Customer: James Whitfield (Business)
Sentiment: neutral | Escalate: No
Response: Structured explanation of proration logic with invoice retrieval path
```

---

## Key Design Decisions Made in Stage 1

These decisions were not obvious at the start. Each was driven by a specific discovery.

**1. Tier 1 escalation bypasses the knowledge base entirely.**
Reason: Early testing showed the agent attempting to answer refund requests and legal queries — responses that could create legal liability or false expectations. Discovery 3 showed 23.6% of tickets need escalation, the majority for non-technical reasons. The KB search was not the bottleneck.

**2. WhatsApp responses are hard-capped at 80 words.**
Reason: Analysis of 55 sample tickets showed WhatsApp messages average 11 words. A 300-word response to an 11-word question breaks the channel contract entirely. The formatter strips bullets, numbered lists, and closings.

**3. Sentiment uses 7 signals, not just keyword matching.**
Reason: Single-keyword matching produces too many false positives. "Hell" in "Hello" triggered profanity. "Sue" in "Issue" triggered legal escalation. Word-boundary regex, caps ratio, and exclamation count combine to produce a more reliable signal.

**4. Escalation routes to 5 specialist queues, not one generic queue.**
Reason: Discovery 3 showed distinct escalation categories requiring different expertise. A refund dispute needs billing team skills, not the same skills needed for an angry technical customer. Generic queues produce slower resolution and worse CSAT.

**5. KB confidence threshold set at 0.40, not higher.**
Reason: WhatsApp messages are short and keyword-sparse. A higher threshold would cause too many unnecessary escalations for messages like "webhook not firing" — which has a clear resolution in the KB despite low keyword overlap.

---

## Stage 1 Completion Checklist

| Requirement | Status | File |
|-------------|--------|------|
| Working prototype agent | Complete | `src/agent/customer_success_agent.py` |
| MCP server with 5 tools | Complete | `src/agent/mcp_server.py` |
| Context datasets (5 files) | Complete | `context/` |
| 55 sample support tickets | Complete | `context/sample-tickets.json` |
| Escalation rules document | Complete | `context/escalation-rules.md` |
| Brand voice guide | Complete | `context/brand-voice.md` |
| Company + product profile | Complete | `context/company-profile.md`, `product-docs.md` |
| Agent skills manifest | Complete | `specs/agent-skills.md` |
| System specification | Complete | `specs/customer-success-fte-spec.md` |
| Discovery log (12 entries) | Complete | `specs/discovery-log.md` |
| Prompt history (10 iterations) | Complete | `specs/prompt-history.md` |
| Edge case tests (25 cases) | Complete | `tests/test_cases.md` |
| CLI demo (agent + MCP) | Complete | Both Python files |

---

## Limitations Accepted in Stage 1

These are conscious decisions — not oversights. Stage 1 is a prototype.

| Limitation | Why Accepted | Stage 2 Solution |
|------------|-------------|-----------------|
| Keyword-based KB search | Sufficient to prove routing logic; simpler to reason about | Vector embeddings + Claude semantic search |
| In-memory conversation state | No session persistence needed for demo | Redis-backed session store with TTL |
| Mock CRM data (6 customers) | Validates personalization logic without live API | Salesforce / HubSpot API integration |
| No cross-channel continuity | Channel switching is rare; Stage 1 scope | Unified customer session ID across channels |
| No multilingual support | Out of scope for prototype | Claude handles non-English natively |
| Heuristic sentiment only | Adequate for prototype; deterministic and debuggable | Claude API structured sentiment call |
| No outbound messaging | Reactive only; proactive is a Stage 2 feature | Webhook-triggered outbound workflows |

---

## How This Evolves — Stage 2 and Stage 3

Stage 1 established: the data, the rules, the agent logic, and where it breaks.
Stage 2 will wire it to real infrastructure.
Stage 3 will close the feedback loop and add proactive intelligence.

### Stage 2: Real Infrastructure
- Replace keyword KB with vector embeddings and semantic search via Claude API
- Replace sentiment heuristics with a single Claude API classification call
- Connect to live Salesforce or HubSpot CRM via API
- Implement real Gmail API and WhatsApp Business API channel adapters
- Add Redis-backed session store for multi-turn memory and cross-channel continuity
- Build monitoring dashboards: escalation rate, CSAT by channel, KB coverage gaps

### Stage 3: Intelligence and Proactivity
- Feed human-agent resolutions back into KB training (feedback loop)
- Proactive outreach: detect at-risk accounts and send check-in messages before they contact support
- A/B test response variants against CSAT outcomes
- Automated SLA enforcement: alert human queue if response time is approaching breach
- Multilingual support: route non-English tickets to language-matched agents or Claude multilingual mode

---

## Team

**Hackathon 5** | Customer Success Digital FTE Track
**Stage 1 Incubation** | March 2026
**Built with:** Python 3.9+ | Claude Agent SDK | MCP Protocol
