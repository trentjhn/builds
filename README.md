# Builds

Production AI systems I've designed and shipped, the architecture decisions behind them, and the through-line that connects them.

I'm a product builder (ex-PayPal technical PM) who builds AI-native systems fast. The speed isn't luck: most of these run on a personal harness I built, a config-as-code pattern where the agent's behavior, state, and tools all live in files, plus a project scaffolder (`/cook`) that encodes the architecture decisions from every prior build. New systems start from accumulated judgment instead of from scratch. That's why "built it in a day" shows up a lot below.

Repos are linked where public. A few are private (client work or live products); those are marked.

---

## The pattern running through all of it

```
CLAUDE.md (behavioral contract)
  + context/ (live state as files)
  + scripts/ or templates/ (tools)
  + git (persistence + history)
  = a self-contained, stateless-by-design AI system
```

The agent's behavior lives in files, not chat memory. Sessions reload state from disk, so nothing depends on a conversation staying alive. The same pattern carries across trading, government intelligence, lead-gen, and project scaffolding. A second recurring move: **grounding gates**, the model is structurally forbidden from inventing the facts that matter (dates, dollar figures, algorithm names), so a slick output can't ship a wrong number.

---

## Shipped and live

The ones you can click and watch work, or that run unattended in production.

### PQC Deal Engine
**Live:** https://pqc-deal-engine.vercel.app/ · **Repo:** [`pqc-deal-engine`](https://github.com/trentjhn/pqc-deal-engine) (public) · Next.js + TypeScript on Vercel

Point it at a company and it generates a board-ready post-quantum-cryptography deal readout: their "harvest now, decrypt later" risk (quantified with the Mosca inequality), the exact regulatory deadlines their vertical faces, a NIST-suite migration sketch, and a CISO one-pager. It's the go-to-market translation layer on top of a crypto scan, not a scanner.

- **Facts can't be hallucinated.** Every regulatory date and algorithm name comes from a versioned data file, never the model's memory. A grounding gate scans each generated readout and rejects any year or standard not in that file, failing closed rather than shipping an invented fact. Even a tampered share link rebuilds the facts server-side.
- **The risk math is deterministic code, not the LLM.** The Mosca score is a pure, tested function; the model only writes prose around a precomputed verdict.
- 27 passing tests, rate-limited generation endpoint, server-side-only key. Built and deployed in about a day.

### GitRecap
**Live (access-gated):** https://gitrecap-gamma.vercel.app · **Repo:** [`gitrecap`](https://github.com/trentjhn/gitrecap) (public) · TypeScript

A phone-first web app that reconstructs what you actually did each day from your GitHub commits and turns it into a readable narrative, so you can see your own work instead of forgetting it.

- **Cost-near-zero caching:** the past is frozen and cached, only the live present is recomputed, so the app runs at almost no API spend.
- Idempotent commit syncing, auth-gated, security headers verified, 71 tests. A real shipped product, not a demo.

### Government Relations Intelligence Dashboard
**Live, fully autonomous** (GitHub Actions cron, 6:30am PT daily) · [gov.signalworks.live](https://gov.signalworks.live) · private (client work)

A pre-7am governance briefing for a public-affairs operator with three concurrent roles. Five public meeting and legislation sources are scraped, diffed against yesterday, summarized through a hallucination gate, and published as a static dashboard before they open their laptop. No inbox, no triage.

- **Two-layer hallucination gate (verbatim substring + spaCy NER).** The model can't author a fact sentence; `display_text` must be a verbatim span from the source, and every named entity in the headline must appear in that span (with tolerance for rounded dollar figures). Fabricated bill numbers and dollar amounts are structurally impossible, which is the whole reason a stakeholder trusts it.
- **State lives in a separate repo from the pipeline,** so the live brief never leaks who's being monitored, and a rollback is `git revert`, not state surgery.
- **Three-state source semantics** (`active` / `quiet` / `degraded`) let the operator tell "nothing happened today" from "the system is broken" at a glance.

### AI Search Visibility Tracker
**Live** (one founder-led brand pilot, baseline complete) · private · Next.js 16 + Postgres + pg-boss

Measures how brands surface inside AI answer engines (ChatGPT, Claude, Gemini, etc.) with statistical rigor instead of a single guess. Same prompt fans out across engines, runs N times, and reports per-prompt metrics the competitors don't publish.

- **The math is the moat, and it's published.** Wilson 95% confidence intervals (correct for the near-0%/near-100% small-sample regime where the textbook formula returns nonsense), plus an AutoGEO impression-weighted GEO score hand-ported from the ICLR'26 paper and cited. Every competitor (Profound, Otterly, AthenaHQ) treats their scoring as proprietary; the `/methodology` page inverts the trust dynamic by showing the work.
- **Extraction decoupled from querying** (separate pg-boss jobs, `temperature: 0`, Zod-enforced schema), so a prompt revision re-runs extraction without re-paying for engine calls.
- **Cost ceiling enforced at boot** via env var; a bad prompt bank can't silently burn the budget.

### YouTube Summarizer Premium
**Production-deployed** · `youtube-summarizer-premium` (private) · React/Vite + Flask + Postgres

Full-stack AI SaaS that turns long videos into structured intelligence with dual-depth summaries, context-aware chat, auth, and Stripe billing.

- **Model migration as economics, not novelty:** moved GPT-4o-mini to Gemini 2.5 Flash-Lite for a 33% cost cut and an 8x larger context window, which eliminates video chunking (and the lost-narrative problem) for 99%+ of videos.
- **Residential-proxy extraction:** YouTube blocks all datacenter and cloud IPs; the documented insight is that datacenter proxies are *also* blocked, so rotating residential IPs are the only reliable path from a cloud backend.
- Three-method extraction with graceful fallback, keep-warm health pings, prompt-version cache invalidation.

### Viridian
**Live** (core phases shipped) · **Repo:** [`viridian`](https://github.com/trentjhn/viridian) (public) · Go, single static binary (`vir`)

A terminal UI that watches Claude Code sessions in real time, tool calls, token spend, session memory, and file diffs, via the host harness's hook system. It's a tool *about* the AI coding harness, not one built by it.

- **fsnotify on the parent directory, not the DB file:** SQLite atomically replaces files during journal cleanup, invalidating the inode, so watching the file directly breaks; watching the parent dir and filtering by name survives it, with a 30ms debounce for sub-100ms latency.
- **Hooks must never raise:** a non-zero exit on a PreToolUse hook blocks *all* Claude Code tool execution, so every hook is a strictly append-only logger wrapped to always exit 0. The only safe contract when you don't own the host.
- **`vir init` merges into `~/.claude/settings.json`** idempotently instead of overwriting, so other tools' hooks survive.

---

## Infrastructure and tooling

The harness and the reusable tools the rest is built on.

### Magnum Opus — the `/cook` project scaffolder
**Live** (Claude Code skill) · part of the AI-Knowledgebase

A meta-system whose output is other systems. `/cook` runs a 9-phase interactive workflow (intake, spec, harness design, capability selection, scaffold, eval baseline) and writes a complete, opinionated project structure to disk, encoding the ~40 architecture decisions every new AI project needs before the first line of code.

- **Hub document as manual RAG:** the workflow routes to knowledge-base sections by path and line range but never copies their content, so updating the knowledge doesn't mean updating the workflow. Routing layer vs. content layer.
- **Three-way topology choice** (single / hierarchical / agent-team) instead of a binary, because those have genuinely different context budgets and failure modes; collapsing them produces wrong architectures.
- **Catalog-first convention:** the index entry is written before the thing it indexes, which structurally prevents stale catalogs.

### AI-Knowledgebase
**Live, continuously growing** · `AI-Knowledgebase` (private)

A practitioner-depth reference library: 14+ synthesis docs distilling 100+ primary sources across AI engineering (prompting, context engineering, agentic systems, evaluation, fine-tuning, security), plus the playbooks and catalogs `/cook` draws from. Distillation, not aggregation: each doc synthesizes 10-20 sources into one readable reference with a four-level README cascade so an agent dropped into the repo can orient itself without being told anything.

### quantum-arxiv-digest
**Public, clone-and-run** · **Repo:** [`quantum-arxiv-digest`](https://github.com/trentjhn/quantum-arxiv-digest) · Python CLI

Pulls the latest quantum, post-quantum-cryptography, and cryptography papers off arxiv into clean, browsable folders. Runs with no API key out of the box (fetch + organize); add a key and it also ranks every paper by relevance without discarding any. A sample run is committed under `examples/` so you can see the output without cloning.

### configkit
**Public** · **Repo:** [`configkit`](https://github.com/trentjhn/configkit) · JavaScript

Generates expert-level LLM config files and skill packs from six plain-English questions. No account required.

### promptarena
**Public** · **Repo:** [`promptarena`](https://github.com/trentjhn/promptarena) · TypeScript

Hands-on prompt-engineering practice with AI feedback, 15 scenarios from beginner (email tone) to advanced (meta-prompting, PRD generation).

---

## Personal systems and experiments

Smaller or single-user builds. Useful, but not the headline.

- **edge_lab** — a session-aware swing-trading analyst that enforces my own framework at every decision (macro alignment, position sizing, journal-similarity matching) and offloads all math to tested Python scripts (`calc.py`), because LLM arithmetic compounds errors in long sessions. Dual-AI portable (CLAUDE.md + GEMINI.md). Private.
- **Domain-Specialized PRD System** — a config-as-code PRD generator for specialized industrial domains. Enforces domain-correct vocabulary as hard constraints (no DAU/MAU in mining software; recovery rate and cost per ton instead) so the output solves the right problem instead of looking professional while missing the domain. Runs a frame, build, stress-test (premortem) session.
- **interview-prep — Job Search OS** — a file-based CRM for a multi-track job search: 27 company folders as atomic context units, STAR stories, session logs as institutional memory, all outreach run through a voice skill. The CLAUDE.md doubles as live pipeline state. Private.
- **Zenkai** — a local web app that turns the knowledge base into a spaced-repetition learning experience (React + FastAPI). Delta-syncs against KB file hashes so it only regenerates quiz content for files that changed. [`zenkai`](https://github.com/trentjhn/zenkai), functional.
- **Parking Lead-Gen Agent** — a Python CLI that turns a parking-garage address into a ranked, contact-enriched advertiser list for about ten cents a run. Replayable stage-by-stage from disk, two-layer budget guard, CSV formula-injection guard at the export boundary. Real measured cost data in a spend ledger. Private.

---

## The methodology

The patterns worth naming, because they recur on purpose:

- **Context as files** — state lives on disk, sessions are stateless by design. The files are the memory.
- **Grounding gates** — the model is forbidden from authoring load-bearing facts (dates, money, bill numbers, algorithm names); they're injected and verified, so a confident output can't be a wrong one.
- **Math offloaded to deterministic code** — anything that has to be exact (risk scores, position sizing) runs in tested Python, never in the model.
- **Decouple expensive stages** — separate the API call from the processing so a downstream tweak doesn't re-pay the upstream cost.
- **Cost discipline as a first-class feature** — boot-time cost ceilings, budget guards, keep-warm strategies, model choice driven by economics.
- **Dual-AI portability** — the same behavioral contract across Claude and Gemini, so a tool-limit on one doesn't stall the work.
- **Routing layer vs. content layer** — indexes and hub documents point to where knowledge lives instead of copying it, so the knowledge can change without breaking the navigation.
