# Building AI Agents — The Practical Guide

> A practical, production-oriented guide to architecting, building, and operating AI agents.
> Available in **English** and **Deutsch**.

**Version 1.2 (April 2026)** — updated for the 2026 model generation (Claude 4.7 Opus, GPT-5, Gemini 3.0).

By **Fabian Bäumler**, [DeepThink AI](https://thnkdeep.ai).

---

## Read the Guide

| Language | File |
| --- | --- |
| English | [Building AI Agents — The Practical Guide](v1.2/Building-AI-Agents-Practical-Guide-EN.md) |
| Deutsch | [KI-Agenten entwickeln — Der Praxisleitfaden](v1.2/KI-Agenten-entwickeln-Praxisleitfaden-DE.md) |

Older versions are kept under [`archive/`](archive/).

## What's inside

Six parts, twelve chapters, four appendices:

- **Part I** — Fundamentals and the 11 fundamental agentic patterns
- **Part II** — Agent architecture: the four critical gaps, skills layer, memory architecture
- **Part III** — Performance and speed optimization for production
- **Part IV** — Information retrieval and production-ready RAG systems
- **Part V** — Self-improving multi-agent RAG systems
- **Part VI** — From prototype to production: decision framework, security, deployment
- **Appendices** — Architecture checklists, benchmarking templates, troubleshooting, further resources

## What's new in v1.2

- All model references updated to the 2026 frontier set: **Claude 4.7 Opus** (1M-token context, extended thinking), **Claude 4.6 Sonnet**, **GPT-5**, **Gemini 3.0**
- New 2026 patterns: 1-hour prompt-cache TTL, agentic memory at 200k+ context, native parallel tool calls, native citations, MCP server ecosystem, cross-provider swarm
- Modernized deployment chapter: Vercel AI SDK 5, AI Gateway, Workflow DevKit, Cloudflare Workers AI, Modal, Inngest, Temporal
- Appendix D refreshed with the 2026 SDK and tooling ecosystem
- Style: em-dashes removed, German umlauts verified

## Repository layout

```
.
├── v1.2/
│   ├── Building-AI-Agents-Practical-Guide-EN.md
│   ├── KI-Agenten-entwickeln-Praxisleitfaden-DE.md
│   ├── de/                       # split source files (chapters 1-6, 7-12)
│   └── en/                       # split source files (chapters 1-6, 7-12)
├── archive/
│   └── v1.1/                     # previous edition (February 2026)
├── assets/
│   └── diagrams/                 # rendered SVG/PNG diagrams
└── LICENSE
```

## License

Content is licensed under [CC BY 4.0](LICENSE). You are free to share and adapt with attribution.

## Citation

```
Bäumler, F. (2026). Building AI Agents — The Practical Guide (v1.2). DeepThink AI.
```

---

# KI-Agenten entwickeln — Der Praxisleitfaden

Ein praxisorientierter Leitfaden für Architektur, Aufbau und Betrieb produktionsreifer KI-Agenten.

**Version 1.2 (April 2026)**, aktualisiert für die Modellgeneration 2026 (Claude 4.7 Opus, GPT-5, Gemini 3.0).

Von **Fabian Bäumler**, [DeepThink AI](https://thnkdeep.ai).

## Was ist drin

Sechs Teile, zwölf Kapitel, vier Anhänge: Grundlagen und die elf agentischen Muster, Architekturlücken, Skills- und Gedächtnis-Layer, Performance, RAG-Systeme, selbstverbessernde Multi-Agent-Systeme, Sicherheits- und Deployment-Strategien.

## Was ist neu in v1.2

- Alle Modellreferenzen auf den 2026er Stand: Claude 4.7 Opus, Claude 4.6 Sonnet, GPT-5, Gemini 3.0
- Neue Muster für 2026: Prompt-Caching mit 1-Stunden-TTL, agentic memory mit 200k+ Kontext, native Parallel-Tool-Calls, native Citations, MCP-Server-Ökosystem, Cross-Provider-Swarm
- Modernisiertes Deployment-Kapitel: Vercel AI SDK 5, AI Gateway, Workflow DevKit, Cloudflare Workers AI, Modal, Inngest, Temporal
- Anhang D mit aktualisiertem SDK- und Werkzeug-Ökosystem
- Stil: Em-Dashes entfernt, Umlaute geprüft

## Lizenz

Inhalte stehen unter [CC BY 4.0](LICENSE) — frei nutzbar mit Quellenangabe.
