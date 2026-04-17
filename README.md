### Jeet Vaidya

CS at UBC (Kelowna), graduating 2027. I design, build, and ship production web systems with AI as a first-class primitive — from zero-knowledge auth architectures to long-running agent runtimes. Most of my work is solo, end to end: system design, implementation, tests, deployment, billing, and ongoing support.

---

## Selected Projects

### [Vindexa](https://github.com/JeetVaidya1/vindexa) — AI Assessment Platform
Live at [vindexa.org](https://vindexa.org). AI assessment generator for course creators and L&D teams: upload content, auto-generate graded assessments, publish shareable links, get per-topic analytics.

- **API**: FastAPI with pydantic-settings, Supabase JWT bearer auth, per-resource ownership checks, rate limiting (per-user, per-endpoint)
- **Database**: Supabase (PostgreSQL + pgvector) with Row Level Security on every table; ownership checks as defense-in-depth alongside RLS
- **AI**: OpenAI GPT-5 / GPT-5-mini routed per task complexity, `text-embedding-3-small` for retrieval, contextual chunking on ingestion
- **Billing**: Stripe subscriptions (monthly / semester / yearly) + credit top-ups, signature-verified webhooks, tier detection, usage tracking
- **Scale**: 42 API routes, 113 tests, 6 router modules, 9 service modules, multimodal ingestion (PDF / DOCX / PPTX / images)

### [Phantom Defender](https://github.com/JeetVaidya1/PhantomDefender) — Privacy SaaS
Privacy command center that creates disposable email aliases and burner phone numbers, then actively monitors for leaks, strips trackers, plants honeypot aliases, and automates GDPR deletion requests.

- **Zero-knowledge architecture**: client-side AES-256-GCM encryption, PBKDF2 key derivation with 600K iterations, auth key and encryption key derived on separate paths. Server stores encrypted blobs and never sees plaintext
- **Stack**: Next.js 14 (App Router), React 18, Tailwind, Supabase (PostgreSQL + RLS), Stripe, Cloudflare Email Routing, Twilio (receive-only SMS), OpenAI `gpt-4o-mini`
- **Infrastructure**: 15 database tables, 20 RLS policies, 42 API routes, 280 passing tests, rate limits on every endpoint, append-only audit log with 90-day retention, emergency-nuke account destruction with 30-day recovery window
- **Development**: built solo on a 3-agent CI pipeline — a feature-builder writes code, a security-auditor reviews it, a test-runner gates merges before they land on main
- **Unit economics**: ~99% gross margin on the base $9.99 plan

### [Jarvis](https://github.com/JeetVaidya1/jarvis) — 24/7 Personal AI Agent
Self-hosted autonomous agent running on a MacBook Pro under launchd. Controlled via Telegram, a real-time web dashboard, or direct WebSocket clients. Built on Claude.

- **Embedded agent runtime**: Anthropic SDK `messages.stream()` in-process, not a CLI subprocess. Token-by-token streaming, real-time tool visibility, mid-run cancellation via AbortController propagated to the SDK, automatic retries with exponential backoff for 429 / 5xx, and tool-loop detection to break infinite tool-calling cycles
- **57 tools** across shell execution, file I/O, Playwright browser automation, GitHub (`gh` CLI), Polymarket CLOB trading, macOS automation (iMessage / Contacts / Reminders / system monitor), Google Calendar + Gmail, web research (Brave, Firecrawl, Perplexity), real-time data (NewsAPI, CoinGecko, Alpha Vantage), and sub-agent spawning
- **WebSocket gateway**: central control plane on port 18789 with a typed discriminated-union protocol (`chat.send`, `agent.cancel`, `agent.status`, `session.list`), session registry, channel-to-session mapping, optional token auth, heartbeat-based dead-connection detection, and event broadcasting to all connected clients
- **Semantic memory**: `sqlite-vec` + `@huggingface/transformers` running `all-MiniLM-L6-v2` (384-dim ONNX model) in-process for vector search over Markdown memory files; zero external vector DB
- **MCP server**: exposes the 57-tool surface to Claude Code as an MCP endpoint, plus a `claude_code` tool inside Jarvis for delegating to sub-sessions
- **Dynamic skill loading**: new tools can be installed from `~/.jarvis/skills/` at runtime and hot-reloaded without restart
- **Trading engine**: Polymarket CLOB integration, parallel forecasting via `Promise.allSettled()` (~30s per cycle regardless of candidate count, vs ~5min sequential), in-process DistilBERT SST-2 for pre-signal sentiment (~30ms latency), Claude Opus for edge calculation, devil's-advocate bear-case eval before committing capital, outcome logging with LLM-as-judge calibration review
- **Dashboard**: Express + SSE command surface on port 4242 with live agent stream, activity feed, system stats (CPU / memory / disk / network), portfolio widget, iMessage feed, drag-and-drop persistent widget layout
- **Operations**: launchd daemon with auto-start on boot and crash recovery, HMAC-verified webhook ingress, scheduled autonomous programs (morning briefing, portfolio reviews, end-of-day summary, weekly outcome review), structured logging to daily log files

### [Canvas AI Assistant](https://github.com/JeetVaidya1/canvas-ai-assistant) — Multimodal RAG for Canvas LMS
Personal study assistant that ingests Canvas course materials and grounds generated quizzes, notes, practice problems, and full exam simulations in citable source chunks.

- **Ingestion pipeline**: PDFs via pdfplumber and PyMuPDF, PPTX via python-pptx, DOCX via python-docx, images via Tesseract OCR, with contextual retrieval on top
- **Vector store**: FAISS, scoped per course so retrieval never crosses course boundaries
- **Generation**: OpenAI GPT-4 with citations tracing each generated question back to the source chunk it was derived from
- **Stack**: Python, FastAPI, uvicorn, TypeScript frontend

### [iClicker Poll Notifier](https://github.com/JeetVaidya1/iclicker-notifier) — Chrome Extension · 220+ active users
Manifest V3 extension that fires instant desktop + Telegram notifications the moment a professor opens a poll.

- **Four independent detection layers** for fault tolerance: WebSocket frame interception, DOM mutation observation, URL change monitoring, and text-pattern matching. If any single method fails, the others still fire
- **Peer broadcast network**: one student's poll capture triggers Telegram alerts for all connected peers via a Cloudflare Workers + KV backend, eliminating single-point-of-failure detection
- Published to the Chrome Web Store

### [IntakeOS](https://github.com/JeetVaidya1/IntakeOS) — AI Customer Intake Agent
Embedded conversational agent for business websites with an explicit 5-layer reasoning architecture separating the language model from the control logic.

- **Layer 1 — Understanding**: intent detection, entity extraction, sentiment classification
- **Layer 2 — Reasoning**: explicit state machine (`greeting → collecting → confirming → completed`) and decision tree, testable independently of the LLM
- **Layer 3 — Knowledge**: RAG over the business's KB so questions are answered before info collection starts
- **Layer 4 — Action**: persistence, third-party integrations, escalation to human agents
- **Layer 5 — Communication**: response composition adapted to detected sentiment
- **Stack**: Next.js 14 (App Router), Supabase, Sentry, Vitest, OpenAI GPT-4o

---

## Engineering Capabilities

**Full-Stack Web Applications**
Shipped production SaaS end to end: Supabase-backed PostgreSQL with Row Level Security + per-user ownership checks, Stripe subscription billing with signature-verified webhooks, client-side AES-256-GCM zero-knowledge encryption, and 280+ tests covering auth, payments, encryption, and email-routing paths (Phantom Defender).

**Python Backend Engineering (FastAPI + PostgreSQL)**
Built Vindexa as a multi-tenant FastAPI application: pydantic-settings configuration, JWT bearer auth with API-key fallback, per-resource ownership checks as defense-in-depth alongside RLS, rate limiting, pgvector for embeddings, Stripe billing with signature-verified webhooks, 42 routes and 113 tests.

**AI Agent Architecture**
Built Jarvis as an always-on agent runtime with the Anthropic SDK embedded directly in-process. Implemented streaming with token-by-token tool visibility, cancellable execution via propagated `AbortController`, tool-loop detection, automatic exponential-backoff retries for 429/5xx, and a tool registry routing across 57 tools. Exposed the entire tool surface over MCP for Claude Code interoperability.

**Retrieval-Augmented Generation**
Designed per-tenant RAG pipelines in Vindexa and Canvas AI Assistant with contextual chunking, FAISS and pgvector for retrieval, model routing that selects GPT-5 or GPT-5-mini per task complexity, and citation-tracing so every generated answer links back to the source chunks it was derived from. Implemented in-process embeddings for Jarvis using `@huggingface/transformers` with `all-MiniLM-L6-v2` ONNX for zero-infra semantic search.

**Service Architecture & Async Coordination**
Jarvis runs a WebSocket gateway with a typed discriminated-union protocol, an embedded agent runtime, a trading engine, a scheduled-job system (node-cron), a webhook ingress server (HMAC-verified), and a real-time dashboard (Express + SSE) — all coordinated through a shared event bus with cross-module pub/sub. Designed the `channel → gateway → runtime → tools` layering so Telegram, the dashboard, voice, and external clients are all equivalent adapters.

**Production Operations**
Shipped a Chrome extension to 220+ active users with four independent detection layers for fault tolerance. Operate Jarvis via launchd with auto-start on boot and crash recovery, HMAC-verified webhooks, structured logging to daily log files, and a 30-minute heartbeat health check. Phantom Defender includes append-only audit logging with 90-day retention, rate limiting on every endpoint, and an emergency-nuke account-destruction flow with a 30-day recovery window.

**AI-Native Development Workflow**
I treat the Anthropic SDK, Claude Code, and MCP as first-class engineering primitives. I build agent pipelines with scoped context, subprocess tools wrapped in structured JSON schemas, and explicit review gates before merge. Phantom Defender is built on a 3-agent CI pipeline (feature-builder / security-auditor / test-runner). Jarvis ships with an MCP server exposing its tool surface to Claude Code and a `claude_code` tool for delegating to sub-sessions. My development loop is designing the context, the tools, and the failure modes — the model handles the line-by-line.

---

## Stack

**Languages** · TypeScript, Python, JavaScript, Java, SQL, C/C++, Bash

**Backend** · FastAPI, Next.js API Routes, Express, Node.js 22, Spring Boot

**Frontend** · Next.js 14 (App Router), React 18, Tailwind, Vite, Chrome Extensions (Manifest V3)

**Databases** · PostgreSQL (Supabase + RLS), pgvector, FAISS, SQLite (+ sqlite-vec), Redis, MongoDB

**AI / ML** · Anthropic SDK (streaming, tool use, vision, MCP), OpenAI SDK (GPT-4o / GPT-5, embeddings), `@huggingface/transformers` (in-process ONNX), LangChain, PyTorch, TensorFlow

**Cloud & Infra** · AWS, Docker, Cloudflare (Workers, KV, Email Routing), Vercel, Supabase, GitHub Actions CI/CD, launchd, `gh` CLI

**Testing** · Vitest, Pytest, Playwright, JUnit

**Payments** · Stripe (Checkout, Subscriptions, webhooks, credit systems)

---

Currently looking for **Summer 2026 internships**. Reach me at **vaidyajeet4@gmail.com**.
