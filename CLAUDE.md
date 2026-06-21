# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A personal engineering knowledge base and interview readiness system — not a software project. There is no build system, no tests, and no deployable code. All content is Markdown.

**Goal:** Become interview-ready by July 31, 2026 for Senior/Platform Engineer and future Architect roles (ServiceNow, Salesforce, Microsoft, Confluent, etc.), while continuing full-time work at GAP.

**Core principle:** Every work item at GAP should produce at least one asset here — a note, a behavioral story, a system design, or a DSA entry.

---

## Repository Structure

```
source/          # All knowledge notes, organized by topic
  01-DSA/        # LeetCode problems (NeetCode 150 focus)
  02-Java/       # Java internals: concurrency, JVM, GC
  03-Spring/     # Spring internals: beans, DI, AOP, transactions
  04-Kafka/      # Kafka deep dives: partitions, consumers, ISR, EOS
  05-Kubernetes/ # K8s/AKS: pods, HPA, probes, scheduling
  06-React/      # React + Redux Toolkit from GAP production
  07-AI/         # MCP, RAG, Agents, Spring AI, LangChain concepts
  08-SystemDesign/ # Full system design docs
  09-Behavioral/ # STAR stories from GAP experience
  10-Applications/ # Job application tracker

templates/       # Starting point for every new note (use these)
docs/            # Reference docs and interview readiness plan
DASHBOARD.md     # Current focus, daily checklist, July goals
DECISIONS_AND_CONTEXT.md  # Target companies, career direction
```

---

## Templates

Every new note must start from the appropriate template in `templates/`:

| Template | Use for |
|---|---|
| `dsa-template.md` | `source/01-DSA/` — one file per problem |
| `engineering-template.md` | `source/02-09/` — Java, Spring, Kafka, K8s, React, AI topics |
| `system-design-template.md` | `source/08-SystemDesign/` |
| `behavioral-template.md` | `source/09-Behavioral/` — STAR format |
| `ai-template.md` | `source/07-AI/` — MCP, RAG, agents |
| `architecture-template.md` | Architecture decision comparisons |

**Never create a note without a template.** The templates enforce the "What / Why / How / Tradeoffs / GAP Example / Interview Questions" structure that makes notes interview-ready.

**One file per topic.** Kafka Consumer Groups and Kafka ISR are two separate files, not sections in one file. Keep notes atomic so they're independently searchable and revisable.

---

## Note-Writing Conventions

- Every engineering note must include a **Real World Example (GAP)** section — connect the concept to actual GAP production work (CDP migration, AKS, Kafka, Sonar MCP, etc.).
- Every note must include **Interview Questions** — 3+ questions the note should make you able to answer cold.
- DSA notes require both brute-force and optimized approaches with explicit time/space complexity.
- Behavioral stories follow STAR (Situation, Task, Actions, Results) and must include **Metrics** and which interview questions they answer.
- The **Architecture Thinking** section in engineering notes asks: Why does this exist? What alternatives exist? What breaks at scale? Would you still choose it at 10x?

---

## Key GAP Context

Production experience to draw from when writing notes:

- **CDP API Gateway Migration** — API gateway design, routing, traffic management
- **RabbitMQ → Kafka Migration** — messaging system tradeoffs, consumer groups, exactly-once semantics
- **AKS Optimization** — Kubernetes requests/limits, HPA, readiness/liveness probes, node scheduling
- **Sonar + MCP Automation** — MCP tools/resources, STDIO vs SSE, tool calling patterns
- **503 Probe Analysis** — Kubernetes probe debugging, pod lifecycle

---

## Daily Cadence

- 1 DSA problem → `source/01-DSA/`
- 1 Engineering note → relevant `source/` subfolder
- Weekly rotation: Java (Mon) → Kafka (Tue) → Spring (Wed) → React (Thu) → AI (Fri) → System Design (Sat) → Review (Sun)
- Sunday: create one master deep-dive note for the week's topic

---

## July 2026 Targets

- 40 DSA problems
- 30 engineering notes
- 5 system design docs
- 5 behavioral stories
- 1 MCP project
- 20 job applications

---

## How to Assist in This Repo

See `AGENT_INSTRUCTIONS.md` at the repo root — that file is the authoritative, model-agnostic guide on persona, note quality bar, vocabulary, and what not to do. It can be used with any AI assistant.

Key reference files for context:
- `docs/Career_Summary.md` — shipped work and GAP context
- `docs/AI_Architect_Roadmap.md` — architect evolution roadmap
- `AGENT_INSTRUCTIONS.md` — full assistant behavior guide
