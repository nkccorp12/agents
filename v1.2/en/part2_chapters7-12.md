# Part IV: Information Retrieval and RAG Systems

Retrieval-Augmented Generation (RAG) forms the backbone of many agent systems. In this part we cover the optimization of search queries and the five decisive corrections that turn a flawed RAG prototype into a production-ready system.

---

## Chapter 7: Hybrid Query Optimization

### 7.1 The Problem with Pure Semantic Search

Pure semantic search produces noisy results in practice. Complex questions lead to a kind of similarity search that returns superficially matching but factually imprecise hits. The problem is especially acute for queries with hard constraints: a search for a black dress that is not made of polyester and has at least four stars cannot be reliably answered by pure semantic similarity.

### 7.2 Filter-first Strategy

The solution lies in a two-step approach: first, hard constraints are applied as structured metadata filters. Only then is semantic search applied to the already pre-filtered result set. This order is critical, because reversed, the semantic results would circumvent the filters.

### 7.3 Hard Constraints vs. Fuzzy Requirements

The key to the hybrid strategy lies in distinguishing between hard and soft requirements. Hard constraints are objectively measurable criteria: color, price, rating, availability. These are implemented as structured metadata filters. Fuzzy requirements, by contrast, are subjective or context-dependent criteria: elegance, perceived quality, style. These remain in the domain of semantic search.

### 7.4 Structured Filters Before Semantic Search

In practical terms this means: the search query is first decomposed into structured filters and semantic components. The filters drastically reduce the result set, from thousands to a manageable number. Semantic search then ranks this pre-selection by relevance. The outcome: instead of thousands of noisy hits, only a few precisely matching results.

As of 2026, the dominant query-decomposition pattern uses a small, fast model (such as Claude 4.6 Sonnet or Gemini 3.0 Flash) to extract structured filters from natural-language input, while reserving the larger reasoning model for the final ranking and answer-synthesis step. Prompt caching with the new 1-hour TTL keeps the decomposition prompt warm across an entire user session at near-zero marginal cost.

### 7.5 MindsDB Implementation

MindsDB, as an open-source solution, provides an ideal platform for implementing structured query workflows. The platform supports both classical SQL filters and semantic search operations and allows the seamless combination of both approaches in a unified query language. This greatly simplifies the implementation of the filter-first strategy.

### 7.6 Domain-Specific Collection Structuring

The filter-first strategy gains additional power when combined with domain-specific collection structuring. Rather than storing all documents in a single collection and relying solely on metadata filters, professional systems organize documents into separate collections by type before any search begins. In a legal context, for example, this means maintaining distinct collections for sales agreements, corporate and IP agreements, and operational contracts.

This structural separation provides an immediate advantage: the system knows which collection to query before retrieval begins, eliminating an entire category of irrelevant results. A query about termination clauses no longer retrieves maintenance agreements simply because they share similar vocabulary. The collection structure acts as the coarsest and most effective filter, reducing the search space before metadata filters and semantic search even engage.

> **Key Takeaways Chapter 7**
> - Pure semantic search is insufficient for complex queries with hard criteria.
> - The filter-first strategy separates hard constraints from soft requirements.
> - Structured metadata filters reduce the result set before semantic search.
> - Domain-specific collection structuring eliminates irrelevant results at the structural level.
> - Result: from thousands of noisy hits to a few precisely matching results.

---

## Chapter 8: Production-Ready RAG Systems

Insights from production deployments at large technology companies show: standard RAG systems frequently deliver an error rate of 60 percent or more. Five targeted corrections can reduce this rate to a production-viable level.

### 8.1 The 5 Key RAG Corrections from Practice

The five corrections each address a specific weakness: the processing of complex documents, the quality of metadata, the search strategy, the quality of search queries, and the gap between mathematical similarity and domain relevance. Together they transform a flawed system into a reliable production tool.

### 8.2 PDF Processing Overhaul

Standard PDF loaders fail with complex documents containing tables, lists, and nested structures. Formatting is lost, table contents become unstructured text, and important contextual information disappears. Two complementary approaches address this problem.

The first approach uses conversion via an intermediate format such as Google Docs, employing specialized loaders that preserve document structure including tables, enumerations, and hierarchies. Layout-aware document understanding libraries such as DuckLink take this further by using AI-powered layout analysis to extract text while maintaining the original structural relationships, converting complex documents into well-structured Markdown that preserves tables, clause hierarchies, and formatting.

The second approach is more radical: eliminate text extraction entirely and convert PDF pages into images that are embedded directly into the database. Multimodal models such as Claude 4.7 Opus, GPT-5, and Gemini 3.0 then read the visual layout, tables, and clause structures as complete visual units, preserving all formatting and structural information that any text extraction process inevitably destroys. This visual processing approach is particularly effective in domains where document layout carries meaning, such as legal contracts, financial statements, and regulatory filings. As of 2026, native PDF input is supported across all major model families, making the visual approach the default for high-stakes document RAG.

### 8.3 Enhanced Metadata: LLM-Generated Summaries

Raw document chunks with only title and URL as metadata are insufficient for nuanced distinctions. The solution: a language model automatically enriches each chunk with generated summaries, FAQ sentences, relevant keywords, and access-control metadata. This enriched metadata substantially improves both filtering and semantic search.

In production pipelines as of 2026, this enrichment step is typically run with a fast Sonnet- or Flash-class model behind prompt caching, so that a corpus refresh of millions of chunks remains economically feasible.

### 8.4 Hybrid Search: Vector + BM25

Vector search alone misses critical documents, especially when semantic similarity does not correlate with actual relevance. Combining vector search with BM25 keyword search closes this gap. BM25 finds exact term matches while vector search captures contextual similarity. Combining both result sets delivers substantially higher retrieval precision.

### 8.5 Multi-Agent Query Processing Pipeline

Poorly formulated search queries yield poor results, regardless of the quality of the search system. The solution is a three-stage agent pipeline: a query optimizer reformulates vague questions into precise search queries. A query classifier determines which document categories should be searched. A post-processor deduplicates and sorts results by their original document position.

```python
# Example: 3-stage query pipeline using Claude 4.6 Sonnet (2026)
from anthropic import Anthropic

client = Anthropic()
MODEL = "claude-4-6-sonnet-latest"

def optimize_query(raw_query: str) -> str:
    resp = client.messages.create(
        model=MODEL,
        max_tokens=512,
        system="Rewrite the user query as a precise retrieval query.",
        messages=[{"role": "user", "content": raw_query}],
    )
    return resp.content[0].text

def classify_query(query: str) -> list[str]:
    resp = client.messages.create(
        model=MODEL,
        max_tokens=256,
        system="Return a JSON list of relevant collection names.",
        messages=[{"role": "user", "content": query}],
    )
    return resp.content[0].text
```

### 8.6 Re-Ranking for Domain Relevance

Mathematical similarity does not equal domain relevance. A document that scores highest in vector similarity may be tangentially related at best, while a critically important document scores lower because it uses different terminology for the same concept. This gap between semantic similarity and actual relevance is the fifth and often overlooked weakness in standard RAG systems.

The solution is a dedicated re-ranking stage after initial retrieval. A specialized re-ranker model receives the initial result set and reorders it based on actual relevance to the specific question rather than abstract vector distance. In domain-specific contexts the impact is dramatic: legal queries surface the most legally pertinent clauses, medical queries prioritize clinically relevant findings, and financial queries highlight the most material disclosures, regardless of whether they scored highest in raw similarity.

Re-ranking operates as a distinct pipeline step between retrieval and response generation. The initial retrieval casts a wide net using hybrid search (vector plus BM25), and the re-ranker then applies domain-aware judgment to surface the most relevant results. This two-stage approach combines the recall advantage of broad retrieval with the precision advantage of focused re-ranking.

### 8.7 From 60% Error Rate to Production Quality

The combination of all five corrections transforms an unreliable system into a production-viable tool. The key lies in systematic application: each individual correction improves the system, but only their interplay overcomes the 60-percent error-rate threshold and delivers reliable, traceable results.

### 8.8 Mandatory Source Attribution

In professional domains, a RAG system that delivers correct answers without traceable sources is nearly as useless as one that delivers wrong answers. Legal teams, medical professionals, and financial analysts cannot act on information they cannot verify. Source attribution is not an optional convenience feature, it is a mandatory production requirement.

Every response from a production RAG system must include detailed citations: the specific document, the page number, the relevant section or clause. This enables human professionals to verify claims, maintain audit trails, and meet regulatory compliance standards. Systems that deliver "black box" responses, correct but unverifiable, fail to meet the professional standards of any high-stakes domain.

As of 2026, all major model families expose first-class citation support: Anthropic's Citations API, OpenAI's structured citations in GPT-5, and Google's grounding metadata in Gemini 3.0. Use these native primitives instead of post-hoc citation extraction.

### 8.9 Case Study: Legal Document RAG

Legal document processing illustrates how all five corrections and additional domain requirements work together in practice. Legal RAG implementations fail at disproportionately high rates when treated as general-purpose document retrieval, because legal documents demand precision, structural awareness, and auditability that generic approaches cannot deliver.

A production-ready legal RAG system applies the full correction stack in sequence. Document collections are structured by contract type (Chapter 7.6) so the system queries sales agreements separately from operational contracts. PDF processing uses visual document understanding to preserve clause structures, table formatting, and section hierarchies that carry legal meaning. An agentic query pipeline implements "think-before-search" reasoning that mirrors how legal teams actually work: first determine which collections are relevant, then break complex legal questions into targeted sub-queries, then apply filters by contract type and date before executing search.

After retrieval, a domain-trained re-ranker reorders results by legal relevance rather than mathematical similarity. Every response includes precise citations, specific contract, page, clause, enabling the legal team to verify and audit. This case study demonstrates a transferable principle: high-stakes professional domains (medical, financial, regulatory) require the same layered approach where each correction addresses a specific failure mode that generic RAG cannot handle.

> **Key Takeaways Chapter 8**
> - Standard RAG systems frequently have an error rate of 60% or more.
> - Five targeted corrections address PDF processing, metadata, search, queries, and re-ranking.
> - Re-ranking bridges the gap between mathematical similarity and domain relevance.
> - Hybrid search (vector + BM25) closes the gaps of pure vector search.
> - A three-stage query pipeline optimizes requests before the actual search.
> - Mandatory source attribution is a production requirement, not an optional feature.
> - Domain-specific RAG (legal, medical, financial) requires the full correction stack plus auditability.

---

# Part V: Self-Improving Systems

The highest level of agent architecture consists of systems that improve themselves. Rather than static performance, these systems learn from their own mistakes and automatically optimize their workflows.

---

## Chapter 9: Self-Improving Multi-Agent RAG Systems

### 9.1 5-Dimensional Evaluation Frameworks

Effective self-improvement requires a differentiated evaluation system. A 5-dimensional framework assesses agent results along multiple axes: precedent correctness, compliance adherence, risk identification, evidence support, and clarity. Each dimension is evaluated separately, enabling targeted improvements.

| Dimension | Evaluation Criterion | Goal |
|---|---|---|
| Precedent | Correct application of relevant foundations | Domain accuracy |
| Compliance | Adherence to professional standards | Regulatory conformity |
| Risk | Identification of all potential hazard areas | Complete risk coverage |
| Evidence | Reasoning supported by real cases | Verifiable justification |
| Clarity | Comprehensibility for human experts | Practical usability |

### 9.2 Specialized Agent Teams

In a self-improving system, specialized agents take on clearly defined roles. A research agent gathers relevant information. A compliance checker scans regulatory requirements. A risk assessor identifies potential problems. A precedent analyst searches for comparable cases. A synthesis agent brings all findings together. Each agent is optimized for its specific task.

As of 2026, a common cost-quality split is to use Claude 4.7 Opus (1M-token context) for the synthesis agent and weakness detector, and Claude 4.6 Sonnet or GPT-5 mini for the specialized worker agents. Extended thinking is enabled selectively on the synthesis and critique steps where deliberate reasoning has the highest leverage.

### 9.3 The Inner Loop: Execution and Feedback

The inner loop encompasses the actual task execution and immediate feedback. The specialized agents carry out their tasks, the results are assessed along the five dimensions, and if ratings are insufficient the execution is repeated with adjusted parameters. This cycle optimizes quality within a single task.

### 9.4 The Outer Loop: Learning and Protocol Editing

The outer loop goes beyond individual tasks. A weakness detector analyzes the 5-dimensional ratings across many executions and identifies systematic weaknesses. A protocol editor then rewrites the agents' working instructions to address the identified weaknesses. The new protocols are tested, re-evaluated, and iteratively improved.

A 2026 best practice is to persist outer-loop learnings as agentic memory: the protocol editor writes structured updates into a long-term memory store that the agents read at the start of every run. With 200k+ context windows now widely available and 1M tokens on Claude 4.7 Opus, the full protocol history can be carried across sessions without lossy summarization.

### 9.5 Automatic System Optimization

The interplay of the inner and outer loops creates a self-optimizing system. The inner loop ensures quality in individual cases; the outer loop improves the systemic foundations. With each iteration the overall system becomes more capable without requiring manual intervention.

### 9.6 Weakness Detection and Protocol Evolution

The system produces not a single supposedly perfect solution but a range of optimized variants with different emphases. This reflects reality: in complex domains there is rarely a single correct answer, but rather different optimal trade-offs. Protocol evolution ensures that the system understands and navigates these trade-offs ever more effectively over time.

> **Key Takeaways Chapter 9**
> - 5-dimensional evaluation enables targeted, differentiated improvements.
> - Specialized agent teams with clearly defined roles maximize result quality.
> - The inner loop optimizes individual tasks; the outer loop improves the overall system.
> - Protocol evolution enables continuous improvement without manual intervention.
> - The system learns from systematic weaknesses, not only from individual errors.

---

# Part VI: From Prototype to Production

The final part brings all findings together and delivers practical frameworks for architectural decision-making, security design, and the productive operation of agent systems.

---

## Chapter 10: Architecture Decision Framework

The right architectural decision is the single most important factor for the success of an agent system. This chapter provides a structured framework that translates the insights from the preceding chapters into a systematic decision process.

### Decision Level 1: Pattern Selection

Start with the simplest architecture that satisfies your requirements. A single agent with the ReAct pattern is sufficient for a surprisingly large number of use cases. Increase complexity only when a measurable quality gain justifies it. The pattern selection matrix from Chapter 2.5 serves as the starting point.

### Decision Level 2: Closing Architecture Gaps

Systematically verify whether your system addresses the four critical architecture gaps from Chapter 3. Planning, sub-agents, file-system access, and the detailed prompter must function as a coherent system. If any component is missing, the overall system suffers.

### Decision Level 3: Integrating the Skills Layer

Identify recurring task patterns and encapsulate them as skills. Begin with the three to five most common task types and expand the skills library incrementally. Measure the quality difference between ad-hoc and skill-based execution.

### Decision Level 4: Prioritizing Optimization

Choose speed optimizations based on your specific bottleneck. If total wait time is the problem, parallelization and predict-and-prefetch help. If result quality is the problem, branching and multi-critic review help. If reliability is the problem, backup agents and human-in-the-loop help.

As of 2026, a fifth lever has become standard: aggressive prompt caching with the 1-hour TTL on Claude 4.7 Opus and 4.6 Sonnet. Long, stable system prompts and skill libraries now cost a fraction of their original token price on cache hits, which often outperforms shrinking the prompt manually.

### Decision Level 5: Designing the Security Layer

Before any agent reaches production, define the security architecture. Identify which agent actions are irreversible or affect external systems, and implement guardrails at all three levels: input validation, plan and tool control, and output verification. The three-layer guardrail system from Chapter 11 provides the foundation. Security is not a feature to be added later, it must be designed in from the start.

> **Architecture Decision Principles**
> - Start simple and increase complexity only when demonstrably needed.
> - The four architecture gaps must be addressed as a coherent system.
> - Skills transform ad-hoc behavior into consistent, reusable processes.
> - Optimization requires measurement: optimize the actual bottleneck, not the assumed one.
> - Human oversight is not a stopgap but a quality feature.
> - Security guardrails at input, planning, and output level are mandatory for production.

---

## Chapter 11: Agent Security Architecture

AI agents in production do not fail loudly with stack traces and error messages. They fail silently, processing fraudulent requests as legitimate, leaking sensitive data in polished responses, or executing actions that a human operator would immediately reject. This silent failure mode makes agent security fundamentally different from conventional software security and demands a dedicated architectural approach.

### 11.1 The Silent Failure Problem

The most dangerous agent failures are the ones nobody notices. Consider a customer support agent that receives a message from a scammer claiming to be a traveling customer, requesting an address change and a refund using a stolen order number. Without proper guardrails, the agent processes this as a legitimate request: it changes the address, initiates the refund, and responds politely, all while facilitating fraud.

This class of failure, social engineering, prompt injection, data exfiltration through conversational manipulation, does not trigger conventional error monitoring. The agent completes its task successfully from a technical perspective. No exceptions, no timeouts, no retries. The attack surface expands significantly as agents gain autonomy and handle higher-stakes decisions: financial transactions, personal data access, account modifications.

### 11.2 The Three-Layer Guardrail System

Effective agent security follows the defense-in-depth principle adapted for AI-specific vulnerabilities. Three guardrail layers, each operating at a different stage of the agent's execution cycle, create overlapping zones of protection. No single layer is sufficient on its own, each catches threats that the others miss.

| Layer | When It Operates | Primary Function |
|---|---|---|
| Input Guardrails | Before the agent thinks | Detect and neutralize malicious inputs |
| Plan & Tool Guardrails | Before the agent acts | Constrain what the agent is allowed to do |
| Output Guardrails | Before the user sees the response | Verify and sanitize what the agent returns |

### 11.3 Input Guardrails: Before the Agent Thinks

The first line of defense intercepts threats before they reach the agent's reasoning process. Input guardrails perform four critical functions. First, prompt injection detection: identifying inputs that attempt to override the agent's instructions, alter its persona, or extract system prompts. Second, social engineering detection: flagging requests that follow known scam patterns such as urgency claims, identity impersonation, or emotional manipulation.

Third, sensitive data redaction: automatically removing or masking personally identifiable information, credentials, or other sensitive data from inputs before processing. Fourth, high-risk request flagging: immediately escalating requests involving address changes, account access modifications, payment redirections, or refund processing above defined thresholds. These flags do not necessarily block the request, they trigger additional verification steps.

### 11.4 Plan and Tool Guardrails: Before the Agent Acts

The second layer controls what the agent is permitted to do, regardless of what it decides it wants to do. This layer enforces structural constraints that cannot be overridden by conversational manipulation. The agent is required to produce a brief execution plan before taking any action, and this plan is validated against business rules before execution proceeds.

Tool allowlists restrict the agent to explicitly permitted tools only: no tool discovery, no dynamic tool creation. Hard business rules define absolute boundaries: no address changes without OTP verification, no refunds above a defined amount without manager approval, never request passwords or full credit card numbers, and mandatory confirmation for all irreversible actions. These rules operate as deterministic checks, not probabilistic assessments. They cannot be talked around regardless of how convincing the input appears.

In 2026, the MCP (Model Context Protocol) server ecosystem has matured into the de-facto distribution channel for tools. Treat every external MCP server as untrusted: enforce the same allowlist, scoping, and audit-logging discipline you would apply to a third-party API, and prefer signed, vendor-published servers over arbitrary community ones.

### 11.5 Output Guardrails: Before the User Sees

The third layer verifies and sanitizes the agent's response before it reaches the user. Output guardrails ensure factual grounding: every claim in the response must be traceable to actual data, not hallucinated. Sensitive information that may have entered the processing pipeline is stripped from the output: internal system identifiers, other customers' data, or confidential business logic.

When the agent encounters uncertainty, output guardrails enforce explicit acknowledgment rather than confident fabrication. The agent must say "I don't know" or "I need to escalate this" rather than generating a plausible-sounding but potentially harmful response. Unclear or ambiguous situations are automatically escalated to human operators. This layer serves as the final safety net before the agent's output enters the real world.

### 11.6 Security Monitoring and Continuous Improvement

Guardrails are not a static deployment, they require continuous refinement based on real-world attack patterns. A comprehensive monitoring layer logs all agent attempts, actions, and guardrail interventions. This data serves two purposes: forensic analysis of incidents and proactive identification of emerging attack patterns.

Failure pattern tracking identifies systematic vulnerabilities: are certain prompt structures consistently bypassing input guardrails? Are specific tool combinations being exploited? Is there a category of social engineering that the system consistently fails to detect? Each identified pattern translates into updated guardrail rules. The security monitoring cycle mirrors the self-improving approach from Chapter 9, applied specifically to the security domain.

### 11.7 Integrating Security with Agent Patterns

Agent security is not an isolated concern, it connects directly to the control patterns from Chapter 2. The Human-in-the-Loop pattern (Pattern 10) provides the escalation mechanism for cases that guardrails flag but cannot resolve autonomously. The Custom Logic pattern (Pattern 11) provides the architectural foundation for implementing hard business rules as deterministic guardrails.

The key insight is that security must be designed as an integral layer, not bolted on as an afterthought. The three-layer guardrail system should be considered a fifth critical architecture component alongside the four gaps identified in Chapter 3: planning, sub-agents, file-system access, detailed prompter, and now security guardrails.

> **Key Takeaways Chapter 11**
> - Agent failures in production are silent, not loud: social engineering beats stack traces.
> - Three guardrail layers (input, plan/tool, output) create overlapping defense-in-depth.
> - Hard business rules as deterministic checks cannot be overridden by conversational manipulation.
> - Output guardrails must enforce "I don't know" over confident fabrication.
> - Security monitoring requires continuous rule updates based on real-world attack patterns.
> - Security is a fifth critical architecture component, not an optional add-on.

---

## Chapter 12: Deployment and Operations

Operating an agent system in production places different demands than operating conventional software. The non-deterministic nature of language models requires adapted strategies for monitoring, error handling, and continuous improvement.

### Monitoring Strategy

Monitor not only technical metrics such as response time and error rate, but also quality metrics of agent results. Implement automated quality checks that validate agent outputs against defined standards. Use the 5-dimensional evaluation frameworks from Chapter 9 as the foundation for your monitoring.

### Error Handling

Agent systems require multi-stage fallback strategies. When a specialized agent fails, a more general agent takes over. When the language model does not respond, backup agents step in. When result quality falls below a threshold, a human-in-the-loop process is automatically triggered.

### Continuous Improvement

Establish an outer improvement cycle that systematically evaluates production data. Identify recurring error types and translate them into improved skills, adjusted prompts, or additional validation rules. Use the insights from the self-improving approach (Chapter 9) to automate this process as far as possible.

### Modern Deployment Platforms (2026)

The deployment landscape for agent systems has consolidated around a handful of platforms, each with distinct strengths. Vercel AI SDK 5 with the AI Gateway has become a default choice for full-stack TypeScript teams, offering unified provider routing, built-in failover between Claude 4.7 Opus, GPT-5, and Gemini 3.0, and tight integration with Next.js Server Actions and the Workflow DevKit for durable, pause-and-resume agents. Cloudflare Workers AI runs inference at the edge with sub-100ms cold starts, well suited to latency-sensitive guardrail and routing layers. Modal targets Python-first teams that need GPU-backed components (custom rerankers, embeddings, multimodal pre-processing) alongside their agent loops, with serverless scaling and per-second billing. For long-running, crash-safe orchestration, Vercel Workflow and Inngest have largely replaced hand-rolled queue systems. The architectural principle has not changed: pick the platform that matches your team's primary language and your agent's longest-running step, not the one with the loudest marketing.

> **Key Takeaways Chapter 12**
> - Agent systems require monitoring at both the technical and content level.
> - Multi-stage fallback strategies ensure reliability in production operations.
> - Continuous improvement uses production data for systematic optimization.
> - The transition from prototype to production is an iterative, not a one-time, process.
> - Pick a deployment platform that matches your language stack and longest-running step.

---

# Appendices

## Appendix A: Architecture Checklists

| Checkpoint | Status | Priority |
|---|---|---|
| Pattern selection documented and justified | [ ] | High |
| Planning tool implemented | [ ] | High |
| Sub-agents with isolated context | [ ] | High |
| File-system access for context management | [ ] | High |
| Detailed prompter as orchestration layer | [ ] | High |
| Memory architecture with three layers defined | [ ] | High |
| Memory pipeline (extract, consolidate, retrieve) implemented | [ ] | Medium |
| Context window budget strategy defined | [ ] | Medium |
| Skills layer defined and documented | [ ] | Medium |
| Speed optimization identified | [ ] | Medium |
| Prompt caching with 1h TTL configured for stable prompts | [ ] | Medium |
| Three-layer guardrail system (input, plan/tool, output) | [ ] | High |
| Hard business rules for irreversible actions defined | [ ] | High |
| MCP server allowlist and audit logging | [ ] | High |
| Security monitoring and logging implemented | [ ] | High |
| Human-in-the-loop for critical actions | [ ] | High |
| Monitoring strategy defined | [ ] | Medium |
| Fallback mechanisms implemented | [ ] | High |
| Quality metrics defined and measured | [ ] | Medium |
| Self-improvement cycle planned | [ ] | Low |

## Appendix B: Benchmarking Templates

For systematic benchmarking we recommend the following metrics per agent system:

| Metric | Measurement Method | Target Value |
|---|---|---|
| Response time (P50) | End-to-end timing | Depends on use case |
| Response time (P95) | End-to-end timing | Max. 3x the P50 value |
| Success rate | Automated quality checks | At least 95% |
| Error rate | Automated monitoring | Below 5% |
| Hallucination rate | Sampling review by experts | Below 2% |
| Cost per request | Token consumption and API costs | Budget-dependent |
| Cache hit rate (prompt caching) | Provider telemetry | At least 70% on stable prompts |
| User satisfaction | Feedback collection | At least 4.0 out of 5.0 |

## Appendix C: Troubleshooting Guide

| Symptom | Likely Cause | Solution Approach |
|---|---|---|
| Agent hallucinates facts | Context window overloaded | Use file-system access, reduce context |
| Inconsistent result quality | Missing standardization | Introduce skills layer |
| Long response times | Sequential execution | Multi-tool speed-up, parallelization |
| Agent ignores instructions | Insufficient system prompt | Revise detailed prompter |
| Faulty search results | Pure semantic search | Implement hybrid search |
| Quality drops at scale | Monolithic system prompt | Introduce skills layer and sub-agents |
| Same errors repeated | Missing learning capability | Implement outer loop self-improvement |
| Performance degrades over time | Memory bloat / context rot | Audit memory, prune stale entries, budget context window |
| Agent forgets prior context | No structured memory | Implement three-layer memory architecture |
| Agent processes fraudulent requests | Missing input guardrails | Implement prompt injection and social engineering detection |
| Agent executes unauthorized actions | No plan/tool guardrails | Enforce tool allowlists and hard business rules |
| Sensitive data in agent responses | Missing output guardrails | Add output sanitization and grounding verification |
| High token bill on stable prompts | Prompt caching not used | Enable 1h-TTL caching for system prompt and skill library |
| Untrusted MCP tools in production | Missing MCP allowlist | Restrict to signed, vendor-published servers |

## Appendix D: Further Resources

The following frameworks and tools support the implementation of the architectures described in this book (current as of 2026):

- **Anthropic Claude SDK** -- Official SDK for Claude 4.7 Opus (1M-token context) and Claude 4.6 Sonnet, with native support for extended thinking, prompt caching (5-minute and 1-hour TTL), the Citations API, and agentic memory primitives.
- **OpenAI Agents SDK** -- Framework for developing multi-modal agent systems on top of GPT-5, with structured outputs, parallel tool calls, and built-in handoff between agents.
- **Google Gemini SDK** -- Access to Gemini 3.0 with native multimodal input, grounding metadata, and code execution as a first-class tool.
- **Vercel AI SDK 5** -- Provider-neutral TypeScript SDK with the AI Gateway, streaming UI primitives, MCP client support, and tight integration with Vercel Workflow for durable agents.
- **LangGraph** -- Framework for state-based agent workflows with support for all 11 patterns and first-class checkpointing for long-running processes.
- **LangChain DeepAgents** -- Reference implementation of the Skills Layer Architecture approach.
- **MindsDB** -- Open-source platform for structured query workflows and hybrid search.
- **DASH** -- Open-source data agent with memory and continuous learning capability.
- **CrewAI** -- Platform for multi-agent systems with role assignment and task delegation.
- **Mem0 (MemZero)** -- Open-source memory layer for AI agents with extraction, consolidation, and retrieval pipelines, now with 200k+ context-window support.
- **DuckLink** -- Document understanding library with AI-powered layout analysis for structure-preserving extraction and Markdown conversion.
- **MCP Server Registry** -- Curated catalog of vendor-published Model Context Protocol servers for tool distribution across Claude, GPT-5, and Gemini agents.
- **Cloudflare Workers AI** -- Edge-native inference and agent runtime with sub-100ms cold starts, suitable for guardrail and routing layers.
- **Modal** -- Python-first serverless platform with GPU-backed components for custom rerankers, embeddings, and multimodal pre-processing.
- **Inngest / Vercel Workflow** -- Durable execution engines for crash-safe, pause-and-resume agent orchestration.

---

*Building AI Agents -- The Practical Guide*
*Version 1.2 (April 2026)*
*© Fabian Bäumler, DeepThink AI*
