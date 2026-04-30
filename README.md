# Building AI Agents — The Practical Guide

> A practical, production-oriented guide to architecting, building, and operating AI agents.
> Available in **English** and **Deutsch**.

**Version 1.3 (April 2026)** — sharpened with verified model capabilities, full reference implementations, and an expanded production chapter (auth, secrets, multi-tenancy, audit, SLOs, cost control).

By **Fabian Bäumler**, [DeepThink AI](https://thnkdeep.ai).

---

## Read the Guide

| Language | File |
| --- | --- |
| English | [Building AI Agents — The Practical Guide (v1.3)](v1.3/Building-AI-Agents-Practical-Guide-EN.md) |
| Deutsch | [KI-Agenten entwickeln — Der Praxisleitfaden (v1.3)](v1.3/KI-Agenten-entwickeln-Praxisleitfaden-DE.md) |

Reference implementations under [`examples/`](examples/) (tool-agent + RAG-agent, Python + TypeScript, ~1,144 LOC).
Older versions: [`v1.2/`](v1.2/) (April 2026 first edition), [`archive/v1.1/`](archive/v1.1/) (February 2026).
Reviewer notes and research outputs: [`_meta/`](_meta/).

## What's inside

Six parts, twelve chapters, seven appendices:

- **Part I** — Fundamentals and the 11 fundamental agentic patterns
- **Part II** — Agent architecture: the four critical gaps, skills layer (incl. Anthropic Skill format), memory architecture
- **Part III** — Performance and speed optimization for production
- **Part IV** — Information retrieval and production-ready RAG systems
- **Part V** — Self-improving multi-agent RAG systems (eval-harness, approval gates, anti-drift, canary rollouts, veto zones)
- **Part VI** — From prototype to production: decision framework, expanded security (identity, secrets, tenancy, PII), expanded deployment (SLOs, audit, rollback, cost control)
- **Appendices A–G** — Checklists, benchmarking templates, troubleshooting, further resources, model capability matrix, reference implementations, skill format spec

## What's new in v1.3

- **Verified model capabilities** in Appendix E: Claude 4.7 Opus (1M context, **Adaptive Thinking**), Claude 4.6 Sonnet, Claude Haiku 4.5, **GPT-5 (400k context, not 1M)**, Gemini 3 Pro / Flash / 3.1, with caching TTLs, pricing, rate limits, and provider-specific caveats — every value sourced.
- **SDK naming corrected** to **Claude Agent SDK** (`@anthropic-ai/claude-agent-sdk`, `claude-agent-sdk`).
- **Full reference implementations** in Appendix F + [`examples/`](examples/): tool-agent (customer-support with approval gate, RBAC, tenant isolation, audit log) and RAG-agent (hybrid BM25 + pgvector, Cohere Rerank v3.5, native citations, 1h prompt cache) — Python and TypeScript, with goldens, Promptfoo configs, Modal deployment.
- **Chapter 9 rewritten** with eval-harness (DeepEval + Promptfoo + GrowthBook), approval gates, frozen-baseline anti-drift, canary rollouts, and explicit veto zones (money movement, auth, medical/legal, destructive ops).
- **Chapter 11 expanded** (11.5–11.11): Output guardrails, security monitoring, integrating security with patterns, **Identity & Auth (OAuth On-Behalf-Of, RFC 8693)**, **Secret handling (Vault/KMS, session-scoped tokens)**, **Multi-tenancy (KV-cache-bleed prevention, vLLM cache_salt, Postgres RLS)**, **PII & data classification (Presidio, GDPR Art. 17)**.
- **Chapter 12 expanded** (12.4–12.7): **SLOs & rate limits** (P50/P95/P99, tenant quotas, provider failover via AI Gateway), **Audit logs** (OpenTelemetry GenAI, WORM storage), **Rollback & incident response** (canary, runbooks), **Cost control** (per-tenant budgets, progressive throttling, Haiku→Sonnet→Opus tiering for 60–90% savings).
- **Section 4.6** added: Anthropic Skill format with `SKILL.md` + `skill.yaml` overlay.
- **Appendix G** added: complete skill format specification with versioning, runtime contract, registry pattern.
- **DE lectorate**: TOC anchors fixed, missing code example from EN ported, translation artefacts cleaned, terminology unified.

## Repository layout

```
.
├── v1.3/                            # current edition (April 2026)
│   ├── Building-AI-Agents-Practical-Guide-EN.md
│   ├── KI-Agenten-entwickeln-Praxisleitfaden-DE.md
│   ├── sections/                    # appendices E, F, G as separate files
│   └── snippets/                    # chapter extension fragments
├── v1.2/                            # previous edition
├── examples/                        # reference implementations (~1,144 LOC code + SQL/YAML/JSON)
│   ├── tool-agent/                  # customer-support agent with approval gate
│   └── rag-agent/                   # hybrid-search RAG with citations
├── archive/v1.1/                    # February 2026 edition
├── assets/diagrams/                 # 11 architecture diagrams (SVG)
├── _meta/                           # codex review, roadmap, research outputs
├── CITATION.cff
└── LICENSE                          # CC BY 4.0
```

## License

Content is licensed under [CC BY 4.0](LICENSE). You are free to share and adapt with attribution.

## Citation

```
Bäumler, F. (2026). Building AI Agents — The Practical Guide (v1.3). DeepThink AI.
```

---

# KI-Agenten entwickeln — Der Praxisleitfaden

Praxisorientierter Leitfaden für Architektur, Aufbau und Betrieb produktionsreifer KI-Agenten.

**Version 1.3 (April 2026)**, geschärft durch verifizierte Modell-Capabilities, vollständige Referenz-Implementierungen und ein ausgebautes Production-Kapitel (Auth, Secrets, Mandantentrennung, Audit, SLOs, Kostensteuerung).

Von **Fabian Bäumler**, [DeepThink AI](https://thnkdeep.ai).

## Was ist drin

Sechs Teile, zwölf Kapitel, sieben Anhänge: Grundlagen + 11 agentische Pattern, Architekturlücken, Skills-Layer (mit Anthropic Skill-Format), Memory, Performance, RAG, selbstverbessernde Multi-Agent-Systeme (mit Eval-Harness, Approval Gates, Verbotszonen), Sicherheit (Identity, Secrets, Mandantentrennung, PII), Deployment (SLOs, Audit, Rollback, Kosten), Anhänge A–G inkl. Capability-Matrix, Referenz-Implementierungen und Skill-Format-Spezifikation.

## Was ist neu in v1.3

- Verifizierte Modell-Capabilities (GPT-5 = **400k**, nicht 1M; Adaptive Thinking statt Extended Thinking bei Opus 4.7; Claude Agent SDK statt Code SDK)
- Vollständige Referenz-Implementierungen (Tool-Agent + RAG-Agent, Python + TypeScript, lauffähig)
- Kapitel 9 komplett neu mit Eval-Harness, Approval Gates und Verbotszonen
- Kapitel 11 erweitert um OAuth On-Behalf-Of, Secret-Handling, Mandantentrennung, PII
- Kapitel 12 erweitert um SLOs, Audit-Logs, Rollback, Kostensteuerung
- DE-Lektorat: TOC-Anchors korrigiert, fehlendes Code-Beispiel ergänzt, Terminologie vereinheitlicht

## Lizenz

Inhalte stehen unter [CC BY 4.0](LICENSE) — frei nutzbar mit Quellenangabe.
