# AI Architect Roadmap — Akhil Mandala

**Written:** June 2026
**Goal:** Senior Platform Engineer → Solution Architect → AI Platform Architect
**Timeline anchor:** Interview-ready by July 31, 2026. Architect trajectory: 2-3 years.

---

## The Fundamental Mindset Shift

Engineers ask: *"How do I build this?"*
Architects ask: *"Should we build this, why this way, and what's the cost of being wrong?"*

The transition isn't about learning more technologies. It's about changing the question you ask first.

Akhil is already making this shift — enabling work (sync workflow, Sonar report, MCP tools, Beauty story breakdown) is architect behavior. The gap is naming and articulating it in architect vocabulary.

---

## The Unfair Advantage (Use It Now)

Built two production MCP servers in a year when most engineers haven't touched one. MCP is where REST was in 2010 — the window where this is a differentiator is **12-18 months**.

Combined with: enterprise Kafka on an 80+-consumer system, API gateway migration at scale, platform engineering instincts. This is a rare profile in 2026. Don't dilute it with generic prep.

---

## Three Parallel Tracks

---

### Track 1 — Platform Engineering
**Gets you:** Senior Platform Engineer title (near-term)

Platform Engineering is the formalization of what Akhil already does. The job: build the "paved road" — the golden path that makes other engineers faster and less likely to make mistakes.

**GAP work reframed in platform language:**

| What was built | Platform Engineering name |
|---|---|
| `sync-main-into-feature-branch` workflow | Developer workflow automation |
| Weekly Sonar portfolio report | Developer experience feedback loop |
| Sonar MCP server | AI-native developer tooling |
| JWT rollout across 6+ apps | Security golden path |
| Beauty story breakdown | Platform contract authoring |

**What to learn:**
- Internal Developer Platform (IDP) thinking — CNCF Platform Engineering whitepaper
- Treat developers as users, the platform as the product
- Backstage / developer portal concepts (Gap already uses this)
- Developer experience metrics: DORA (deployment frequency, lead time, MTTR, change failure rate)

**Interview framing:**
> "I didn't just fix the APIGEE migration for my team — I built the substrate that 8 other teams migrated on top of."

---

### Track 2 — Distributed Systems Architecture
**Gets you:** The vocabulary and first-principles depth to design systems at architect level

You have production experience. What's needed is the **language and frameworks** to articulate it in interviews and design reviews.

#### Domain-Driven Design (DDD) — Learn this first
The most important non-technology skill for an architect. Maps directly to your GAP work.

Core vocabulary:
- **Bounded Context** — a domain boundary with its own model and language. PF is one bounded context; BF is another.
- **Domain Event** — something that happened in the domain (`StyleCreated`, `HazmatAttributeUpdated`). Kafka topics are domain event streams.
- **Aggregate** — a consistency boundary. The SKU aggregate owns its invariants.
- **Anti-Corruption Layer** — translation between bounded contexts. The BF→PF contract is an ACL.

Exercise: Map the PF/BF system to these concepts. This is a `source/08-SystemDesign/` note.

#### Event-Driven Architecture Patterns
You've done this at GAP. Now learn the names so you can design from scratch.

| Pattern | What it solves | Where you've touched it |
|---|---|---|
| Event Sourcing | Store state as events, not current state | Kafka PLM events |
| CQRS | Separate read/write models for scale | PF read vs write paths |
| Saga | Distributed transactions without 2PC | Cross-service consistency |
| Outbox Pattern | Guaranteed event delivery | Kafka producer reliability |
| Consumer Group rebalancing | Partition assignment on membership change | Kafka consumer groups |

#### Operational Architecture (what architects own that engineers don't)
- **OpenTelemetry** — unified traces, metrics, logs. Now the industry standard. Learn the data model.
- **Capacity planning** — "how many Kafka partitions for 10x load, and why?" is an architect question.
- **Cost architecture** — architects justify spend. What does the Kafka cluster cost per million messages?

---

### Track 3 — AI Architecture
**Gets you:** Solution/Platform Architect role in 2-3 years. The emerging specialty.

#### The 6 Design Concerns Unique to AI Systems

These are what separates an AI Architect from a traditional architect. Traditional architects don't know these yet — that's the window.

**1. Non-determinism as a first-class design concern**
The same input can produce different outputs. Traditional testing breaks. Design implications: evaluation pipelines, output validation, confidence thresholds, fallback paths.

**2. Context window as a resource**
Like memory or CPU — finite, expensive, must be managed. RAG is fundamentally a context management strategy. Architects design for it: what goes in the context, in what order, at what cost?

**3. Token economics (cost architecture)**
LLM calls have a per-token cost. Decisions: prompt caching, model tiering (use Haiku for classification, Opus for reasoning), output length constraints, batching. This is infrastructure cost architecture.

**4. Prompt injection — the new SQL injection**
Any system that takes user input and passes it to an LLM is a new attack surface. Architects must design trust boundaries, input validation, and output sandboxing.

**5. Human-in-the-loop design**
Knowing *when* to escalate to a human is an architectural decision, not a product one. Confidence thresholds, audit trails, rollback patterns.

**6. Agent orchestration patterns**
Multi-agent systems have coordination, failure, and consistency problems just like microservices. The patterns are new names for old distributed systems problems:

| Agent pattern | Distributed systems analogue |
|---|---|
| ReAct (Reason + Act loop) | Event loop with side effects |
| Planning agent | Orchestrator / workflow engine |
| Memory (short/long-term) | Cache + persistent store |
| Tool use | RPC / API call |
| Multi-agent handoff | Service mesh routing |
| Reflection | Circuit breaker + retry with audit |

#### What to Build

**July anchor project:** Team-shared SSE MCP server deployed to GAP's CDP platform.
- SSE transport (not STDIO) so teammates connect via URL, no jar needed
- Deployed to CDP using existing Chartis CI/CD
- This is a *platform engineering* artifact — infrastructure that other engineers' agents run on
- Document the architecture decisions (auth, rate limiting, observability)

**After July:** Build one RAG system end-to-end.
- Doesn't need to be complex — "ask questions about our runbooks" is enough
- Learn: chunking strategies, embedding models, vector stores (pgvector, Pinecone), retrieval (similarity vs hybrid), evaluation
- The failure modes are the learning: hallucination, retrieval misses, context stuffing

---

## Architecture Decision Records (ADRs)

Start writing one per week. This is the highest-leverage habit for an architect in training.

Format for every ADR in `source/08-SystemDesign/ADR-XXX-topic.md`:
```
## Problem
## Options Considered
## Decision
## Tradeoffs Accepted
## What Would Change This Decision
```

Start with decisions already made at GAP:
- ADR-001: Why Kafka over RabbitMQ
- ADR-002: Why STDIO over SSE for the first MCP server
- ADR-003: Why hazmat fields at global Item level, not per market-channel
- ADR-004: Why a sync GHA workflow instead of manual merges

---

## Weekly Habit Stack

| Day | Track | Focus |
|---|---|---|
| Monday | Java | Concurrency internals (maps to platform reliability) |
| Tuesday | Kafka | Deep internals + event-driven patterns |
| Wednesday | Spring | Internals + DDD application |
| Thursday | React | Production usage from GAP codebase |
| Friday | AI Architecture | MCP, RAG, agents, token economics |
| Saturday | System Design | One ADR or full system design doc |
| Sunday | Synthesis | Weekly master note + what can I explain cold? |

---

## The Interview Narrative to Build Toward

By July 31, be able to say this cold:

> "I work on a platform team that owns the product data domain for a global retailer — 80+ consumers, multi-brand, multi-channel. I've driven a major API gateway migration, built AI tooling that integrates with our developer workflow, and shipped platform automation that the whole team runs on. My next step is formalizing that into a Platform Engineer role, and my long-term direction is AI Platform Architecture — designing the infrastructure that agentic systems run on inside enterprises."

That's not a crafted story. That's just true. The prep is learning the vocabulary to say it in architect language.

---

## The Honest Gap (What to Close)

| Gap | Why it matters | How to close it |
|---|---|---|
| DDD vocabulary | Architects talk in bounded contexts and domain events | One deep note in career-os + apply to PF/BF |
| "What breaks at 10x" thinking | Architects design for scale they haven't hit yet | Add this question to every engineering note |
| Business case framing | Architects justify cost, not just correctness | Add "Cost / ROI" section to ADRs |
| Cross-team influence | Akhil specs for PT-PF; architects influence org-wide | Volunteer for cross-team design reviews |
| AI security | Prompt injection is a new attack class | Learn OWASP Top 10 for LLMs |
