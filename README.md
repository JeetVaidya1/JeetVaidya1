### Jeet Vaidya

CS at UBC (Kelowna), graduating 2027. I design, build, and ship production web systems with AI as a first-class primitive — from zero-knowledge auth architectures to long-running agent runtimes. Most of my work is solo, end to end: system design, implementation, tests, deployment, billing, and ongoing support.

---

## Selected Projects

### [iClicker Poll Notifier](https://github.com/JeetVaidya1/iclicker-notifier) — Published Chrome Extension · 220+ active users
Manifest V3 Chrome extension that fires instant desktop, audio, and Telegram push notifications the moment a professor opens an iClicker poll. Published to the Chrome Web Store. Used live in lecture halls by 220+ students.

- **Four independent poll-detection layers** running in parallel for fault tolerance — no single method covers every edge case, so every detector runs concurrently and the first to fire wins:
  1. **WebSocket frame interception** — hooks into iClicker's real-time socket protocol to catch server-pushed poll events as they arrive
  2. **URL change monitoring** — watches for `/poll` and `/question/` path transitions via the History API
  3. **DOM `MutationObserver`** — watches for poll-related elements (timers, answer options, submit buttons) being mounted into the page
  4. **Text-pattern matching** — scans for phrases like "answer now" and "time remaining" as a last-resort detector
- **Collaborative peer detection network** — one student's successful detection fans out as a Telegram push to every other student in the same class session. Course and activity IDs are parsed from the iClicker URL at page load, the extension auto-registers the user in that session, and when any peer's detection fires the backend broadcasts to everyone in the session. Turns each user's weakness into the network's strength: if your internet lags, a classmate's detector still saves you.
- **Manifest V3 service worker architecture** — background logic runs as an ephemeral service worker (no persistent background page under MV3), with a `chrome.offscreen` document spawned on demand just for audio playback to work around the MV3 audio restriction. Service worker wake-up is handled via session heartbeat messages so the broadcast pipeline stays connected.
- **Cloudflare Workers + KV backend** — the broadcast layer is fully serverless: session registry in KV, heartbeat keepalive, Telegram Bot API for push delivery, HMAC-verified webhooks. Sessions auto-expire on a 6–10 hour TTL so stale data self-cleans without a cron.
- **Telegram account linking** — 6-digit OTP flow with a 10-minute expiry. User messages `@IclickerNotificationBot`, receives a code, pastes it into the extension popup, and the `chat_id` is bound to their session for future pushes.
- **Chrome Web Store listing** — passed Chrome's MV3 review with scoped host permissions (iClicker domains only) and a public privacy policy explaining every data field captured.
- **Privacy-first by design** — no credentials captured, no poll answers read, no grades accessed, no personal info collected. The only data that leaves the device is the Telegram chat ID and the course/activity IDs needed to route notifications. Uninstalling the extension clears all local data.
- **State tracking** — still detects new polls while the user is reviewing a previous question's results (common case where naive detectors get confused by the "active poll" UI state already being visible).
- **Debug surface** — "Scan Page for Poll Elements" tool in the popup UI exposes what every detector sees on the current page, so support/debug can isolate which layer is firing and which isn't.
- **"LIVE" badge** — red badge overlay on the extension icon when a poll is active, providing at-a-glance status without opening the popup.
- **Chrome autoplay workaround** — audio alerts route through the offscreen document with fallback to visual notification when Chrome's autoplay policy blocks sound before any user interaction with the page.

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

### [git-gud](https://github.com/tysonLivadney/COSC310-Project-Git_gud) — Food Delivery Platform · Team Project (COSC 310)
Full-stack food ordering and delivery platform built with a **9-person team** over a semester-long Software Engineering course under an agile / scrum SDLC. Deployed via Docker Compose with separated backend and frontend services.

- **Stack (matches the common production FastAPI + React + Docker + GitHub Actions stack)**: FastAPI + Pydantic + Uvicorn + pytest on the backend, React 19 + Vite + React Router + axios on the frontend, Dockerized for local dev, GitHub Actions CI with Ruff linting and pytest-cov coverage reporting on every push and PR
- **Architecture**: 15 API router modules (`admin`, `auth`, `delivery`, `drivers`, `menus`, `menu_items`, `orders`, `order_total`, `payment`, `promo_codes`, `restaurants`, `reviews`, `location`, `notifications`) layered into `routers → services → repositories → schemas` with separated unit and integration test suites
- **Authentication**: salted PBKDF2-HMAC password hashing, bearer tokens with server-side session expiry, role-based authorization across `user / owner / manager / driver`
- **13 authored PRs** — my core contributions:
  - Order creation and management (`#58`) — the foundation of the ordering flow
  - Admin subsystem and endpoint tests (`#71`, `#81`)
  - Admin analytics report (`#92`) — average delivery time per driver and highest-rated restaurants, implemented with Pandas aggregations
  - Driver accounts and profile system (`#101`)
  - Auto-assign available driver algorithm (`#111`)
  - Promo code / discount system (`#139`, `#143`) — backend pricing logic and frontend display across checkout, orders list, and order detail
  - Order endpoint test coverage (`#104`)
  - Structural refactors: split payment from pricing (`#119`), extracted address lookup into its own module for single-responsibility (`#117`), introduced a `DriverStatus` enum replacing magic strings (`#112`), reorganized orders and admin services (`#105`)
- **Maintainer role**: merged 11 teammate PRs into `main` as a code reviewer, responsible for gating changes through CI and team review before they land
- **CI pipeline** (`.github/workflows/ci.yml`): runs Ruff on critical error patterns (`E9, F63, F7, F82`), pytest with `--cov` generating `coverage.xml`, artifact upload on every build

### [Canvas AI Assistant](https://github.com/JeetVaidya1/canvas-ai-assistant) — Multimodal RAG for Canvas LMS
Personal study assistant that ingests Canvas course materials and grounds generated quizzes, notes, practice problems, and full exam simulations in citable source chunks.

- **Ingestion pipeline**: PDFs via pdfplumber and PyMuPDF, PPTX via python-pptx, DOCX via python-docx, images via Tesseract OCR, with contextual retrieval on top
- **Vector store**: FAISS, scoped per course so retrieval never crosses course boundaries
- **Generation**: OpenAI GPT-4 with citations tracing each generated question back to the source chunk it was derived from
- **Stack**: Python, FastAPI, uvicorn, TypeScript frontend

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

**Production Shipping & User-Facing Products**
Published a Manifest V3 Chrome extension through the Chrome Web Store review process and operate it in production for 220+ active users, including the peer broadcast network that keeps their detection reliable even when individual browsers miss the poll event. Shipped Vindexa to production at vindexa.org with Stripe billing and tiered plans. Phantom Defender's 280-test suite covers auth, payments, encryption, and email-routing paths.

**Full-Stack Web Applications**
Designed and shipped production SaaS end to end: Supabase-backed PostgreSQL with Row Level Security + per-user ownership checks, Stripe subscription billing with signature-verified webhooks, client-side AES-256-GCM zero-knowledge encryption, and comprehensive test coverage across auth, payments, and data paths.

**Python Backend Engineering (FastAPI + PostgreSQL)**
Built Vindexa as a multi-tenant FastAPI application: pydantic-settings configuration, JWT bearer auth with API-key fallback, per-resource ownership checks as defense-in-depth alongside RLS, rate limiting, pgvector for embeddings, Stripe billing with signature-verified webhooks, 42 routes and 113 tests. Same backend stack (FastAPI + Pydantic + Uvicorn + pytest + Ruff) shipped on a 9-person team in COSC 310 with a production-style GitHub Actions CI pipeline.

**AI Agent Architecture**
Built Jarvis as an always-on agent runtime with the Anthropic SDK embedded directly in-process. Implemented streaming with token-by-token tool visibility, cancellable execution via propagated `AbortController`, tool-loop detection, automatic exponential-backoff retries for 429/5xx, and a tool registry routing across 57 tools. Exposed the entire tool surface over MCP for Claude Code interoperability.

**Retrieval-Augmented Generation**
Designed per-tenant RAG pipelines in Vindexa and Canvas AI Assistant with contextual chunking, FAISS and pgvector for retrieval, model routing that selects GPT-5 or GPT-5-mini per task complexity, and citation-tracing so every generated answer links back to the source chunks it was derived from. Implemented in-process embeddings for Jarvis using `@huggingface/transformers` with `all-MiniLM-L6-v2` ONNX for zero-infra semantic search.

**Browser Extensions & Edge Runtimes**
Ship-worthy Manifest V3 Chrome extension with service worker architecture, `chrome.offscreen` document for audio playback within MV3 constraints, multi-layered detection running in parallel, and a Cloudflare Workers + KV serverless backend with HMAC-verified webhooks and TTL-based session cleanup.

**Team Engineering & Agile Collaboration**
Shipped git-gud (COSC 310 food delivery platform) with a 9-person team under an agile / scrum SDLC: 13 authored PRs spanning order creation, the admin subsystem with Pandas-driven analytics, driver accounts with an auto-assignment algorithm, promo code pricing across backend and frontend, and four structural refactors (splitting responsibilities, extracting modules, introducing enums, reorganizing services). Served as a maintainer: merged 11 teammate PRs into `main`, gating changes through a CI pipeline (Ruff + pytest + coverage artifact) before they landed.

**Service Architecture & Async Coordination**
Jarvis runs a WebSocket gateway with a typed discriminated-union protocol, an embedded agent runtime, a trading engine, a scheduled-job system (node-cron), a webhook ingress server (HMAC-verified), and a real-time dashboard (Express + SSE) — all coordinated through a shared event bus with cross-module pub/sub. Designed the `channel → gateway → runtime → tools` layering so Telegram, the dashboard, voice, and external clients are all equivalent adapters.

**Production Operations**
Operate Jarvis under launchd with auto-start on boot and crash recovery, HMAC-verified webhooks, structured logging to daily log files, and a 30-minute heartbeat health check. Phantom Defender includes append-only audit logging with 90-day retention, rate limiting on every endpoint, and an emergency-nuke account-destruction flow with a 30-day recovery window.

**AI-Native Development Workflow**
I treat the Anthropic SDK, Claude Code, and MCP as first-class engineering primitives. I build agent pipelines with scoped context, subprocess tools wrapped in structured JSON schemas, and explicit review gates before merge. Phantom Defender is built on a 3-agent CI pipeline (feature-builder / security-auditor / test-runner). Jarvis ships with an MCP server exposing its tool surface to Claude Code and a `claude_code` tool for delegating to sub-sessions. My development loop is designing the context, the tools, and the failure modes — the model handles the line-by-line.

---

## Stack

**Languages** · TypeScript, Python, JavaScript, Java, SQL, C/C++, Bash, R

**Backend** · FastAPI, Pydantic (v2) + pydantic-settings, Uvicorn, Starlette, SlowAPI (rate limiting), Next.js API Routes, Express, Node.js 22, Spring Boot

**Frontend** · React 18 & 19, Next.js 14 (App Router), Vite, React Router, Tailwind, axios, react-hot-toast, Chrome Extensions (Manifest V3)

**Databases & Storage** · PostgreSQL (Supabase + Row Level Security), pgvector, Redis, FAISS, SQLite (+ sqlite-vec), MongoDB, JSON-file repositories, Cloudflare KV

**AI / ML** · Anthropic SDK (streaming, tool use, vision, MCP server), OpenAI SDK (GPT-4o / GPT-5, embeddings), `@huggingface/transformers` (in-process ONNX), LangChain, PyTorch, TensorFlow, scikit-learn, Pandas, NumPy

**Cloud & Infra** · AWS, Docker, Docker Compose, Cloudflare (Workers, KV, Email Routing), Vercel, Supabase, GitHub Actions CI/CD, launchd, `gh` CLI

**Testing & Code Quality** · Pytest + pytest-cov, Vitest, Playwright, JUnit, Ruff, ESLint (flat config), isort

**Payments** · Stripe (Checkout, Subscriptions, webhooks, credit systems)

**Collaboration & Process** · Git, GitHub, PR-based review workflows, CI gates (lint + test + coverage), Agile / Scrum SDLC, code review as a maintainer

> The backbone across my projects — FastAPI + PostgreSQL + React + TypeScript + Docker + GitHub Actions + Anthropic / OpenAI — is the common production stack for modern AI-native SaaS. I'm comfortable adding pieces I haven't shipped yet (SQLAlchemy, AWS ECS / SQS / EventBridge, Prometheus / Grafana / PostHog) given the adjacency to what I've already built.

---

Currently looking for **Summer 2026 internships**. Reach me at **vaidyajeet4@gmail.com**.
