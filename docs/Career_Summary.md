# Akhil Mandala — Career Summary

**Role:** Senior Software Engineer, PT — Product Foundation, GAP Inc.
**Stack:** Java 21, Spring Boot, Apache Kafka, Kubernetes (AKS), GCP, React + Redux Toolkit
**Email:** akhil_mandala@gap.com

---

## What I Do at GAP

I'm a senior engineer on the Product Foundation team — the team that owns GAP's product and brand data services. Product Foundation (PF) is the canonical source of truth for Style/CC/SKU attributes across brand × market × channel. Brand Foundation (BF) owns merchandise hierarchy structures and cross-app translation. These services sit at the core of GAP's commerce platform — 80+ consumers depend on them.

My work pattern is both hands-on and enabling: I execute tickets on my own, and I spec and drive cross-cutting infrastructure tracks that teammates build on top of.

---

## Shipped Work

### APIGEE → CDP API Gateway Migration
Drove JWT auth rollout across multiple internal apps (Annual Purge, Sharepoint, internal EP/BF) as GAP exited APIGEE in favour of the CDP API Gateway. Soft-decommissioned APIGEE proxies. Built a dual-instance sync GHA workflow (`sync-main-into-feature-branch`) to keep the `api-gateway-version` branch in sync with `main` automatically after each deploy. This workflow now runs continuously and removed a recurring manual step for the whole team.

**Relevant for:** API gateway design, JWT auth, traffic migration, automation

---

### 503 Probe Analysis + AKS CPU Optimization
Investigated and documented a recurring 503 pattern across Product Foundation and Brand Foundation apps running on AKS. Traced root causes to probe misconfiguration and resource limit tuning. Authored the analysis page and drove AKS CPU optimization changes in production (Sep 2025).

**Relevant for:** Kubernetes liveness/readiness probes, resource requests/limits, HPA, reliability engineering

---

### Sonar MCP Server (Interim Tool)
Built a 7-tool Python MCP server that queries GAP's internal SonarQube instance, allowing AI agents (Claude Desktop) to retrieve code quality metrics, quality gate status, issues, and hotspots without leaving the IDE. Designed as an interim solution while the official SonarSource MCP server is blocked on a company-wide security review (PSEC-9238).

**Relevant for:** MCP architecture, tool design, developer productivity, AI tooling

---

### Gateway Monitor MCP Server
Built a Java/Spring Boot MCP server connected to GCP Cloud Logging. Tools: `analyzeApp` (gateway errors + app exceptions + caller breakdown), `getTopCallers` (request volume by user), `detectIncident` (gateway-wide incident detection), `getGatewayHealth` (error summary). Used live in on-call triage via Claude Desktop.

**Relevant for:** MCP server architecture (Java/Spring AI), GCP logging, observability, agentic incident triage

---

### Weather MCP Server (Reference Implementation)
Built a public reference Java/Spring Boot MCP server using Spring AI. Documented the full pattern — STDIO transport, tool definition, testing with MockRestServiceServer, Claude Desktop wiring. Wrote an internal Confluence guide on building MCP servers at GAP. Next step: deploy a team-shared SSE-transport MCP server to Gap's CDP platform.

**Relevant for:** MCP internals, STDIO vs SSE, Spring AI, developer productivity

---

### Weekly Sonar Portfolio Report Automation
Built a GitHub Actions workflow + Python script that runs every Monday at 08:00 IST, queries SonarQube for all PT-PF apps, and emails an HTML portfolio health report to the team DG. Report is reviewed in Monday standup — new-code regressions are attributed to the checkin author.

**Relevant for:** GHA automation, SonarQube integration, team accountability loops

---

### Artifactory Modernization
Migrated 8 GHA workflows from static ARTIFACTORY_USERNAME/PASSWORD secrets to OIDC-based token minting (`mint-jfrog-token`). Drove the same migration for the `product-foundation-avro-schema-modelling` publishing repo.

**Relevant for:** OIDC, secrets management, CI/CD security, JFrog Artifactory

---

### Beauty Enablement (Story Breakdown Author)
Authored the full story breakdown for PETEPF-8849 — adding Beauty-category hazmat, lot, and inventory control attributes to Product Foundation. Key design decision: hazmat fields live at global Item level only, never duplicated per market-channel. Wrote the validation rules and patch API contract.

**Relevant for:** API contract design, domain modelling, cross-functional feature delivery

---

### Release Engineering
Author of most "Foundation - iN release tracker" Confluence pages (i564, i566, i568, ...). Own the DevOps weekly task tickets. Serve as PagerDuty on-call for brand-foundation and product-foundation namespaces.

**Relevant for:** Release processes, on-call engineering, operational ownership

---

## Technology Depth

| Area | What I've done |
|---|---|
| **Kafka** | RabbitMQ → Kafka migration; PLM attributes on BF Publisher; consumer group and offset management; Kafka 7.3.2 → 7.9.2 migration in progress |
| **Kubernetes / AKS** | Probe tuning, CPU optimization, HPA, namespace management (`nms-pt-productfoundation`, `nms-pt-brandfoundation`) |
| **Java / Spring Boot** | Java 21; Spring Security JWT; `@Transactional`; Spring AI for MCP; DI internals |
| **GCP** | Cloud Logging queries; log-based alerting; multi-project setup (prod/non-prod per namespace) |
| **MCP / AI** | Two working MCP servers (Python + Java); STDIO and SSE transport; tool + resource design; agentic on-call triage |
| **GHA / CI-CD** | 8+ workflows authored; OIDC secrets; scheduled reports; auto-merge workflows |
| **SonarQube** | Portfolio reporting; quality gate analysis; Sonar MCP tooling |
| **React / Redux** | Production usage via GAP codebase; Redux Toolkit, createSlice, createAsyncThunk |

---

## Leadership Pattern

I tend to build infrastructure that multiplies the team's output rather than one-off fixes:
- The `sync-main-into-feature-branch` workflow eliminated a recurring manual merge step for everyone.
- The weekly Sonar report created a team accountability loop that didn't exist before.
- The Sonar MCP server gave the team AI-accessible code quality data before the official tool was approved.
- The Beauty story breakdown gave the team a clear contract before implementation started.

---

## Career Direction

**Current:** Senior Software Engineer (GAP Inc.)
**Near-term target:** Senior Platform Engineer
**Long-term:** Solution Architect / Platform Architect
**Emerging focus:** AI Platform Engineering — agentic systems, MCP, developer productivity tooling

**Target companies:** ServiceNow, Salesforce, T-Mobile, Walmart Global Tech, JPMorgan Chase, Microsoft, Adobe, Optum, Confluent
**Timeline:** Interview-ready by July 31, 2026
