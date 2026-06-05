# AI Builds Log

> Personal record of AI systems built — architecture, key decisions, what made each solution elegant.
> Updated as I build. Not exhaustive — focused on things worth remembering.

---

## Systems Overview

Quick look at what's been built and why each matters:

| System | Status | Type | Problem Solved |
|--------|--------|------|---|
| **[Magnum Opus / /cook](#1-magnum-opus--the-cook-project-scaffold-system)** | Live | Meta-Infrastructure | Building systems requires making ~40 architecture decisions ad hoc (topology, context strategy, model selection, eval baseline) and without a workflow they get made implicitly or skipped |
| **[Viridian](#2-viridian--session-intelligence-tui-for-claude-code)** | Live (Phases 0-2, 4 shipped; 3, 3.5 functional) | Meta-Infrastructure | Building AI systems generates a fast-moving stream of tool calls, token spend, session memory, and file diffs — but the host harness shows none of it in real time |
| **[YouTube Summarizer Premium](#3-youtube-summarizer-premium--full-stack-ai-video-intelligence)** | Production-deployed | Full-Stack AI | Long videos require watching in full to extract information; chunking-based tools lose narrative coherence |
| **[AI Search Visibility Tracker](#4-ai-search-visibility-tracker)** | Live (1 pilot baseline complete) | Production AEO Measurement | Brands are increasingly discovered through AI answer engines, but no commercial AEO tool publishes its math — the credibility ceiling is whoever builds the rigorous version first |
| **[Government Relations Intelligence Dashboard](#5-government-relations-intelligence-dashboard)** | Live (autonomous daily cron) | Production Intelligence Pipeline | A multi-role public-affairs operator can't manually read every relevant agenda + bill calendar + board packet + PDF every morning, and off-the-shelf LLM summarization fabricates dollar figures and bill numbers |
| **[edge_lab](#6-edge_lab--trading-analyst-system)** | Live | Trading Automation | Trading frameworks break down under real-time pressure — steps skipped, math approximated, thesis drifted |
| **[Zenkai](#7-zenkai--personalized-ai-learning-app)** | Functional | Learning Platform | Good reference material doesn't create retention; needed active recall and spaced repetition for AI content |
| **[AI-Knowledgebase](#8-ai-knowledgebase--personal-knowledge-library)** | Continuously growing | Knowledge Library | AI engineering knowledge scattered across 100+ sources with no practitioner-depth synthesis that ages well |
| **[interview-prep](#9-interview-prep--job-search-os)** | Live | Job Search OS | 10+ concurrent applications across memory-less sessions; needed live-state CRM with company-specific context |
| **[Domain-Specialized PRD System](#10-domain-specialized-prd-generation-system)** | Complete | PRD Generation | Generic PM templates fail in industrial domains: wrong personas, wrong success metrics, missing physical constraints |
| **[Parking Lead-Gen Agent](#11-parking-lead-gen-agent)** | Functional | Lead Generation CLI | Out-of-home advertising sales sources local advertisers manually — Yelp + spreadsheet + phone book — for hours per asset with no audit trail of why a prospect was contacted |
| **[security-var-agent](#12-security-var-agent--value-added-reseller-recommendation-engine)** | Functional | Recommendation Engine | VAR workflows require market-real vendor analysis, ROI modeling, and confidence scoring; manual comparison is error-prone |

---

## The Pattern Running Through All of This

Before diving into individual systems, there's a meta-pattern worth naming:

```
CLAUDE.md (behavioral contract)
  + context/ (live state as files)
  + scripts/ or templates/ (tools)
  + git (persistence + history)
  = a self-contained AI-native system
```

Every system below uses some version of this. The agent behavior lives in files, not in chat memory. Sessions are stateless by design — the files ARE the memory. This is the context-as-files pattern applied consistently across domains: trading, job search, interview prep, knowledge management.

The second pattern: **dual AI portability**. Three systems now have both CLAUDE.md and GEMINI.md (same behavioral contract, different tool dialects). When one AI hits limits, swap to the other without losing context.

---

## Systems

### 1. Magnum Opus — The /cook Project Scaffold System

**Status:** Live (Claude Code skill, system-level)
**Location:** `~/.claude/skills/cook/` (local) + `AI-Knowledgebase/future-reference/skills-catalog/meta/cook/` (shareable)
**Hub document:** `AI-Knowledgebase/future-reference/playbooks/magnum-opus.md`

#### The Problem
Every other system in this log was built by hand — architecture decided ad hoc, agents assembled from memory, context strategy figured out mid-build. After seven systems, a pattern emerged: every new AI project requires ~40 architectural decisions before writing a line of code (topology, context strategy, model selection, tool design, eval baseline, security threat model), and without a workflow those decisions get made implicitly. The gaps surface in production.

#### What It Is
A meta-system: a system whose output is other systems. Invoked with `/cook` in Claude Code, it runs a 9-phase interactive workflow — intake → domain research → project classification → spec + pre-flight → harness design → build methodology → capability selection → scaffold output → eval baseline — and writes a complete, opinionated project structure to disk. Every architectural decision learned across seven prior builds is encoded in a single structured workflow. New projects start from that accumulated knowledge instead of from scratch.

Output: a project directory with CLAUDE.md, SOUL.md, AGENTS.md, design doc, implementation plan, and pre-selected agents and skills from the catalog.

#### Architecture

```
/cook skill (SKILL.md)             ← CLI entrypoint; triggers on "new project" / "/cook"
    ↓ reads
magnum-opus.md (hub document)     ← 9-phase decision workflow; routes to KB, never contains KB content
    ↓ routes to
KB-INDEX.md                       ← flat navigation catalog (line ranges, read times, descriptions)
    ↓ routes to
LEARNING/ docs                    ← agentic-engineering.md, context-engineering.md, evaluation.md, etc.
    ↓ combined with
agent-catalog/CATALOG.md          ← 22 agents across 6 categories
skills-catalog/CATALOG.md         ← 34 skills across 5 categories
prompt-catalog/CATALOG.md         ← 16 prompt patterns
    ↓ produces
Project scaffold on disk          ← CLAUDE.md + SOUL.md + AGENTS.md + design doc + implementation plan
```

#### Key Decisions and Why They're Elegant

**1. Hub document as manual RAG**
`magnum-opus.md` routes to KB sections by file path and line range but never copies KB content. The hub knows *where* knowledge lives; the KB docs contain the actual knowledge. When KB docs get updated, the hub doesn't need updating (unless the file path changes). This is the same principle as an index page — routing layer vs. content layer — applied to an agentic workflow.

**2. Three-layer separation: workflow / knowledge / capabilities**
The /cook skill handles workflow state (what phase are we in). The KB docs handle knowledge (what does context engineering actually mean). The CATALOG.md files handle capability listing (what agents and skills exist). No layer bleeds into another. You can update one without touching the others. This made the system significantly cheaper to maintain than a monolithic doc.

**3. Three-way topology decision (not just "multi-agent")**
Early draft collapsed all multi-agent patterns into a binary: single vs. multi. Phase 1 now forces three distinct choices: single agent / hierarchical multi-agent (orchestrator + workers) / agent teams (peer agents with shared task list and direct messaging). These have meaningfully different context budgets, coordination overhead, and cost profiles. Collapsing them produces wrong architecture decisions.

**4. Prohibited Patterns with "Do not..." framing**
Early draft had an "Anti-Patterns" section with prohibited behaviors listed as topic sentences. This creates a priming problem: naming bad behavior as a heading makes the model consider it as a candidate. Renamed to "Prohibited Patterns" with explicit "Do not..." framing throughout. The fix is about framing: prohibitions should be unambiguous, not implied.

**5. SOUL.md as a functional personality file**
Instead of embedding agent character in CLAUDE.md (which holds structural rules), SOUL.md is a separate file loaded before every session. It holds non-negotiable behavioral traits (considered output, cite sources, YAGNI, reversible actions, verify before claiming complete) and a `[PROJECT CHARACTER]` section for project-specific customization. Separating personality from rules reduces behavioral variance across sessions.

**6. Catalog-first convention**
CATALOG.md entries are written before the agent/skill/prompt file is created — inverts the natural instinct (write the thing, update the index later) and prevents stale catalogs structurally. The catalog is always authoritative because it was written first. Now codified in CLAUDE.md so future additions follow the same rule.

**7. Portability via KB_ROOT**
All KB paths in the skill are anchored to a single path at the top of SKILL.md. To install on any machine: one path substitution. The portable version in `skills-catalog/meta/cook/` uses `/path/to/your/AI-Knowledgebase` as a placeholder — anyone who clones the KB can make the skill live with a single find/replace.

---

### 2. Viridian — Session Intelligence TUI for Claude Code

**Status:** Live (Phases 0-2, 4 shipped; Phase 3 + 3.5 functional; Phases 5-6 ahead)
**Location:** `/Users/t-rawww/Projects/viridian/`
**Repo:** `trentjhn/viridian` (public)
**Binary:** `vir` (single static Go binary)

#### The Problem
Building AI systems with Claude Code generates a fast-moving stream of tool calls, token spend, session memory, and file diffs — but the host harness shows none of this in real time. Meaningful introspection requires after-the-fact log spelunking. The result: you build *with* Claude Code without ever watching how it actually works on your project, and lose the chance to catch tool-routing inefficiencies, runaway token spend, or stale context being injected into the next session.

#### What It Is
A Go TUI that watches Claude Code sessions in real time via the host harness's hook system. Three Python hook scripts (PreToolUse / Stop / SessionStart) write to a shared SQLite session DB; a BubbleTea TUI reads from the same DB and renders live panels — Activity (every tool call, color-coded), Stats (tool counts, real token spend), Memory (what was carried into the current session), Diffs (before/after for Edit/Write), and an embedded PTY shell with scrollback. Sub-100ms latency from hook write to TUI render via fsnotify.

#### Architecture

```
Claude Code session (any project)
   │
   ├─ PreToolUse hook   ──► pre_tool_use.py    ──► tool_events table
   ├─ Stop hook         ──► stop.py            ──► session_memory table
   └─ SessionStart hook ──► session_start.py   ──► reads memory → additionalContext JSON
                                  │
                          ~/.local/share/viridian/session.db (SQLite)
                                  │
                          fsnotify (parent dir, 30ms debounce)
                                  │
                          vir watch (BubbleTea TUI)
                          ├─ Activity / Stats / Memory / Diffs panels
                          └─ Embedded PTY shell (vt10x) with scrollback
```

Hooks write; TUI reads. The DB is the contract between them. Live updates flow over fsnotify → channel → BubbleTea Cmd → re-render. Transcript JSONL is a parallel data source for accurate per-turn token counts.

#### Stack
- Go 1.26.2, single static binary
- `charmbracelet/bubbletea` v1.3 — TUI framework
- `charmbracelet/lipgloss` v1.1 — styling, borders, layout
- `charmbracelet/harmonica` — spring physics for intro animation
- `spf13/cobra` v1.10 — CLI subcommands
- `modernc.org/sqlite` v1.48 — pure-Go SQLite (no CGO)
- `fsnotify/fsnotify` v1.9 — sub-100ms DB change detection
- `creack/pty` + `hinshun/vt10x` — embedded shell + terminal emulator
- `hexops/gotextdiff` + `alecthomas/chroma` — diff parsing + syntax highlighting
- Python 3 stdlib (no deps) for the three hook scripts

#### Key Decisions and Why They're Elegant

**1. Three-hook capture surface (PreToolUse / Stop / SessionStart)**
Each hook writes a different *kind* of fact. PreToolUse captures fine-grain intent (the tool call, before it runs). Stop captures session shape (turns, files, summary). SessionStart consumes — it doesn't write — closing the loop by injecting prior memory as `additionalContext`. Three orthogonal cuts of a session, one shared SQLite file.

**2. Hooks must never raise, ever**
The CLAUDE.md rule and the script structure both enforce it: every hook wraps `main()` in try/except, prints to stderr, exits 0. Reason: a non-zero exit on PreToolUse blocks *all* Claude Code tool execution. Treating the hook as a strictly side-effecting append-only logger is the only safe contract when you don't own the host harness.

**3. fsnotify on the parent directory, filtered by filename**
Watching the DB file directly breaks because SQLite atomically replaces files during journal cleanup, invalidating the inode. Watching the parent dir survives recreation; the loop filters `ev.Name != dbPath`. A 30ms debounce coalesces SQLite's burst (data write + journal cleanup) into a single signal. Sub-100ms perceived latency, zero idle DB reads.

**4. Hook-side line-number enrichment**
`pre_tool_use.py` finds `old_string` in the file at write time and stores `_viridian_line` in the JSON before SQLite insert. The Go diff viewer uses it to auto-center the viewport on the edit. The line is computed *once*, at the only moment the file is guaranteed to still match — not re-derived on every render.

**5. SGR reset boundary fix**
Lipgloss emits `\x1b[0m` (full SGR reset) between every styled span. That clears the background to the terminal's native color, producing a striped appearance over the styled panels. Failed approaches: per-component `BorderBackground()`, BG prefixes per span. Working fix: `fillBackground()` runs once at the output boundary and replaces every `\x1b[0m` with `\x1b[0;48;2;R;G;Bm` (reset + re-apply BG). Fix the leak at the seam, not at every span.

**6. PTY scrollback via capture-and-replay**
vt10x is fixed-size, no scrollback. Rather than swap to a different terminal lib, the `Terminal` struct keeps a raw byte capture buffer alongside the live VT. `ScrollRender` allocates a temporary larger vt10x, replays the entire capture, and slices the requested viewport. Scrollback as a pure function of the byte stream — the live VT is untouched.

**7. `Update()` mutates, `View()` is pure**
BubbleTea's `View()` runs on a value-copy of the model; mutations vanish. CLAUDE.md captures this as a gotcha after a bug where a "do once then stop" centering flag never stuck. Solution: scroll clamping (`clampShellScroll`) and one-shot anchor centering (`applyCenterScroll`) live in `Update()` only. Mutations belong on the side that's allowed to mutate.

**8. Project-scoped Activity and Diffs**
Filters `tool_events` by current working directory before rendering. Without this, Activity showed tool calls from every concurrent Claude Code session globally — useless noise. The DB stays global (one source of truth across all projects); the TUI scopes the *view*. The right cut is at read time, not write time.

**9. Heuristic memory extraction first, AI extraction later**
After studying claude-mem (59k stars), chose user-message extraction with no AI calls — messages ≥30 chars, capped at 15, truncated to 500. Storage and injection architecture are AI-ready: Stage 2 can swap in summarization without touching the schema or the hook contract. Ship the boring version first; preserve the upgrade seam.

**10. Two parallel data sources: hooks (live) + JSONL transcripts (truth)**
Hooks miss usage events and assistant text (PreToolUse only fires on tool calls). The JSONL transcript at `~/.claude/projects/*.jsonl` carries per-turn `usage` blocks with real token counts. Viridian reads both — hooks for live activity, transcripts for accurate stats / copy mode / historical diffs. Estimation stays as fallback for sessions that predate the install.

**11. `vir init` as merge, not overwrite**
Writes to `~/.claude/settings.json` by reading existing JSON, walking the `hooks` map, idempotently appending only the three hook entries it owns. Other tools' hooks survive. Re-running prints "already registered" instead of duplicating. The right primitive for a shared config file you don't own.

**12. Why this matters — meta-infrastructure framing**
Viridian sits in the same tier as Magnum Opus / `/cook`: it's a tool *about* Claude Code, not a tool *built by* Claude Code. The artifact and the harness are the same surface — you watch your AI coding sessions through a TUI you wrote in Go, using hooks the host harness exposes. Most builds in this log produce systems that do work for the user; Viridian produces visibility into the work itself, which is the layer above.

---

### 3. YouTube Summarizer Premium — Full-Stack AI Video Intelligence

**Status:** Production-deployed and live
**Location:** `/Users/t-rawww/Projects/youtube-summarizer-complete/`
**Repo:** `trentjhn/youtube-summarizer-premium` (private)
**Live URL:** `https://summarizeyt.app`

#### The Problem
Long videos require watching in full to extract information. Most AI summarization tools hit context limits and chunk the video — losing the narrative arc and connection between ideas. The result is bullet points that look comprehensive but miss the through-line.

#### What It Is
Full-stack AI SaaS that transforms YouTube videos into structured intelligence. Dual-mode summarization (Quick: concise 3-paragraph summary, Deep: comprehensive narrative) with context-aware agentic chat and timestamp navigation back to source material. Auth, Stripe payments, and a tiered free/pro model.

#### Stack
- **Frontend:** React + Vite + Tailwind CSS v4 + Framer Motion (Bento Grid layout, parallax hero)
- **Backend:** Flask (Python) + SQLAlchemy + PostgreSQL (Supabase) + Flask-Limiter + SocketIO
- **LLM Models:** Google Gemini 2.5 Flash-Lite (primary), OpenAI GPT-4o-mini (chat fallback)
- **Infrastructure:** Vercel (frontend), Render free tier (backend) + UptimeRobot keep-warm pings
- **Proxy:** ProxyJet rotating residential proxies (`PROXY_URL` env var) — bypasses YouTube IP blocks

#### Key Decisions and Why They're Elegant

**1. Model migration for economics + context**
Switched from OpenAI GPT-4o-mini → Google Gemini 2.5 Flash-Lite. Economics: 33% cost reduction. Context: 8x larger window (1M vs 128K tokens). Practical impact: eliminates chunking for 99%+ of videos. Documented migration logic in `ai_summarizer.py` shows deliberate model economics optimization beyond just "newer is better."

**2. Dual-mode summarization architecture**
Two completely different prompting strategies with identical core principles but different output depth:
- **Quick Mode:** 3-paragraph full summary (~250 words) + 5–7 key points + timestamps — substantive standalone value
- **Deep Mode:** 8-module breakdown with detailed analysis, key quotes, arguments sections
The separation lets users choose summarization depth without branching prompt logic — same prompt engine, two different output targets. Prompt versioning (v5.1) auto-invalidates cache on prompt changes — no stale summaries.

**3. Production-grade prompt engineering**
Sophistication rarely seen in deployed systems. Core principles baked into prompts:
- **Comprehensiveness Principle:** Content dictates output length, not arbitrary constraints.
- **Faithful Representation:** Preserve tone/intent without sanitizing controversial content.
- **Attribution Preservation:** Clear sourcing when speaker quotes others or references studies.
- **Tone Matching:** 5 configurable tones (Objective, Academic, Casual, Skeptical, Provocative).
- **Few-shot learning embedded:** BAD vs GOOD output examples in the prompt itself.

**4. Residential proxy architecture for YouTube extraction**
YouTube blocks all cloud provider and datacenter IPs from transcript/data APIs. Solution: rotating residential proxies via `PROXY_URL` env var, applied to both youtube-transcript-api (via requests Session) and yt-dlp (via native proxy option). Proxy test runs in a background daemon thread at startup — non-blocking, so cold starts aren't delayed. Key insight: datacenter proxies also blocked (different error: "Sign in to confirm you're not a bot" vs "IP belonging to a cloud provider"). Residential IPs are the only reliable path from a cloud backend.

**5. Three-method extraction with graceful fallback**
Transcript extraction tries three methods in order: youtube-transcript-api → yt-dlp → Selenium. Each has a hard timeout. `ignoreerrors: False` on yt-dlp — returning None on failure caused a downstream NoneType crash that was harder to diagnose than an explicit exception. All method failures accumulate into a single structured error string for debugging.

**6. Async processing with polling fallback and retry tolerance**
Background thread processing + `/api/video-status/<id>` polling every 3s. In-memory ProgressTracker is wiped on Render restart — solved with DB fallback in the status endpoint (queries Video table if tracker is empty). Frontend tolerates 5 consecutive poll failures before surfacing an error. Polling endpoint explicitly exempt from Flask-Limiter global rate limits via `@limiter.request_filter` — "50 per hour" global limit was killing polling on active sessions.

**7. Render free tier keep-warm strategy**
Render free tier spins down after 15 min idle; cold start is 30–60s. `/api/health` endpoint added for UptimeRobot to ping every 5 minutes — keeps server warm at zero cost. Flask-Limiter exemption on the health endpoint prevents pings from counting against rate limits.

**8. Context-aware chat with layered context management**
Chat service builds conversation context from three layers: video title → structured summary → transcript (truncated to 5K chars). Conversation history limited to 10 messages to prevent context bloat. Context engineering: careful composition prevents token waste while preserving relevance.

**9. Mobile-first parallax architecture**
Parallax hero uses framer-motion scroll-driven transforms (opacity, scale, blur, y). Critical insight: CSS `filter: blur()` triggers GPU compositing on every frame — on mobile this causes jank. Solution: `isMobile` state via `window.matchMedia`, disables blur entirely on mobile and reduces parallax magnitude. `touch-action: manipulation` on all interactive elements eliminates the 300ms double-tap detection delay. Snap scrolling (`snap-mandatory`) retained but `snap-stop-always` removed — allows natural momentum-based arrival at snap points instead of hard braking.

**10. Token accounting and context strategy**
Conservative max input (900K/1M tokens) leaves explicit buffer for output. Detailed token-per-word calculation (1.3 tokens/word English, 150 words/minute of speech) → ~195 tokens/min of video, ~11,700 tokens/hour. Strategic math prevents runtime surprises.

**11. Deployment infrastructure-as-code**
Frontend (Vercel): `VITE_API_URL` env var points to backend. Backend (Render): `render.yaml` IaC blueprint. Critical decision: Web Service instead of serverless — Lambda timeouts before LLM processing finishes. Deployment logic reflects real constraints.

**12. Cache invalidation strategy**
PostgreSQL persistence + cache key versioning. Prompt version bump (e.g. v5.0 → v5.1) auto-invalidates all cached summaries without data loss. `/api/admin/clear-cache` endpoint for manual purge. Admin endpoints protected by `X-Admin-Token` header.

**Lessons & pre-flight framework:** `future-reference/playbooks/building-ai-saas.md`

---

### 4. AI Search Visibility Tracker

**Status:** Live for one founder-led consumer-brand pilot — 30+ prompts seeded, baseline run complete across 3 active engines, methodology page + Gap Report rendering against real data. Pending: T+30 re-baseline (~30 days post-T0), Perplexity + Google AI Overview re-enable when client revenue justifies the spend.
**Repo:** Private

#### The Problem
Brands are increasingly discovered through AI answer engines (ChatGPT, Claude, Perplexity, Gemini, Google AI Overviews) rather than classical search — but no commercial AEO tool publishes its math. Existing tools (Profound, Otterly, AthenaHQ, CrowdReply) report a single point estimate per prompt with no sample size, no confidence interval, no transparent scoring formula. The result: a category in early stages with no credibility-grounding methodology. The opportunity: build the credible measurement substrate first.

#### What It Is
A multi-engine AEO measurement tool that reports brand visibility with statistical rigor instead of vibes. Same prompt fans out to multiple AI answer engines, each engine runs N=5 (Lite) or N=10 (Standard) times, structured fields (mention rate, position, sentiment, citations, competitor mentions) are extracted via Zod-enforced LLM extraction, and a daily aggregator produces per-prompt metrics with **Wilson 95% confidence intervals** and an **AutoGEO impression-weighted GEO Score** ported from the ICLR'26 paper. The methodology page is published — the moat is the math everyone else treats as proprietary.

#### Architecture

```
seed scripts ──► Postgres (Client / Keyword / Prompt + domains[] + competitor split)
                                     │
                                     ▼
                       enqueueBaseline(promptId, engines, N)
                                     │  fan-out at enqueue, not in worker
                                     ▼
                pg-boss queue: engine-query  (one job per promptId × engine × runNumber)
                                     │
                                     ▼
                engineRegistry[engine].query(prompt)  ──► OpenAI / Anthropic / Google
                                     │
                                     ▼
              Run row upsert  (responseText, engineCitations[], responseRaw Json)
                                     │
                                     ▼
                   pg-boss queue: result-extraction  (separate downstream job)
                                     │
                                     ▼
              extractResults() → generateObject + Zod (Claude, temperature 0)
                                     │
                                     ▼
                      Result row (brandMention, position, sentiment, citations, competitors)
                                     │
                                     ▼
                    Daily aggregator → Analysis row (Wilson CI, GEO, SoV) per (promptId, date)
                                     │
                                     ▼
                Next.js dashboard: /clients/[slug]/{gaps, comparison, …} + /methodology
```

State is concentrated in Postgres (queue, raw responses, extractions, aggregates all in one DB). Engine adapters are interchangeable behind a single `EngineAdapter` interface; extraction is decoupled from querying so prompt revisions don't re-pay engine API costs.

#### Stack
- Next.js 16 (App Router) + React 19 + TypeScript 5
- Prisma 7 with `@prisma/adapter-pg` driver-adapter pattern
- PostgreSQL (one DB serves data + queue + Studio)
- pg-boss v11 (`batchSize: 2` to stay under provider per-minute ceilings)
- Vercel AI SDK v6 (`ai@^6`) + `@ai-sdk/{openai,anthropic,google,perplexity}@^3`
- Models: `gpt-5` + `openai.tools.webSearch`, `claude-opus-4-7` + `anthropic.tools.webSearch_20250305`, Gemini via `@ai-sdk/google`; extraction on `claude-sonnet-4-6`
- 3 engines active in v1; Perplexity + Google AI Overview (via `serpapi@^2`) scaffolded but disabled in `ALL_ENGINES`
- Zod 3 for extraction schema + env validation
- Tailwind 3 + shadcn/ui + Recharts 2 + lucide-react

#### Key Decisions and Why They're Elegant

**1. Wilson 95% CI over normal approximation**
In the N=5 / N=10 regime with proportions often near 0% or 100%, the textbook normal approximation produces intervals like `[-0.12, 0.32]` — lower bound below zero is mathematically nonsense. Wilson stays bounded in `[0,1]` by construction and has better small-sample coverage (Brown/Cai/DasGupta 2001). 15-line hand port — no dependency. Prevents reporting nonsense lower bounds in the exact regime the tool runs in.

**2. Hand-port of AutoGEO impression formula**
`lib/analysis/geo-score.ts` is a faithful port of `cxcscmu/AutoGEO/.../geo_score.py` — word count × position decay (`exp(-i/(T-1))`) × split-by-citations-per-sentence. The first scaffold contained an invented heuristic; verification against the published source caught it. Citing AutoGEO (ICLR'26) grounds the math in literature instead of internal heuristic. Prevents the "looks rigorous but isn't" failure mode that kills credibility on first pushback.

**3. N=5 Lite / N=10 Standard sample-size discipline**
Same prompt to the same engine produces different responses; multi-run sampling quantifies the variance instead of pretending it doesn't exist. Even at N=5 the methodology is 5x more rigorous than competitors who report a single point estimate, and the ladder gives a clean upgrade path for retainer work.

**4. `temperature: 0` for extraction**
Engine queries are non-deterministic by nature; the extraction layer that pulls structured fields out of those responses must not add more variance on top. This is what makes "re-run extraction after a prompt tweak" safe. Prevents drift on identical inputs across re-extractions.

**5. Zod-enforced extraction schema via `generateObject`**
Structural enforcement at the LLM boundary; the model cannot return a shape it wasn't asked for. Hallucinated fields drop on the floor instead of polluting the `Result` table. The schema doubles as documentation of every field the dashboard expects.

**6. 3 engines active in v1, 2 deferred (cost discipline)**
Perplexity ($50 prepaid minimum) and Google AI Overview via SerpAPI ($75/month subscription) are scaffolded fully in `lib/engines/` but disabled at the orchestration layer. Methodology page tells the truth about it. Re-enable = remove a comment, not a refactor. Prevents pre-revenue burn while keeping the v2 path one line away.

**7. `Client.domains String[]` for AutoGEO brand-share aggregation**
Brand-level GEO Score requires knowing which cited URLs belong to the client. `domains` is a substring-match allow-list per client (subdomains, retailer pages, social profiles). Schema add was operator-gated and shipped with `@default([])` so prior seeds stayed compatible. Without this column, brand-level GEO is ungrounded.

**8. `peerCompetitors` vs `incumbents` split**
Two-bucket competitor model. Incumbents (the dominators in the category) saturate every prompt and tell you nothing about positioning; peers are the actual share-of-voice comparison group. The Gap Report ranks by `peerMentionRate × (1 − clientMentionRate)` — high peer activity with low client presence means the gap is closeable. A flat competitor list would surface only "the giants always win" non-insights.

**9. One pg-boss job per `(promptId, engine, runNumber)` — fan-out at enqueue, not in worker**
Failed runs retry independently; one slow engine doesn't block the others. `@@unique([promptId, engine, runNumber])` paired with worker upsert makes pg-boss batch-retry semantics safe (a re-delivered job becomes an idempotent no-op).

**10. Extraction as a separate downstream job**
A failed extraction must never lose the raw response. Decoupling extraction from the engine call means a prompt-engineering iteration re-runs extraction without re-paying the API call. `Run.responseRaw Json` is the forensic substrate that makes the escape hatch real.

**11. Engine citations persisted to a dedicated `engineCitations String[]` column**
Mid-flight bug surfaced: `JSON.stringify` drops Vercel AI SDK's getter-based `result.sources`, so reading citations off `responseRaw` returns nothing. Fix: extract `citations` at adapter access time and persist to its own column. Schema-level discipline catching SDK serialization quirks.

**12. `result.sources` shape pre-flight verified**
The integration field is `sourceType: 'url'`, not `type: 'url'` — caught before it cost a baseline. Same reflex caught `gpt-5 + openai.tools.webSearch({})` as the correct setup vs falling back to `gpt-4o`. Pre-flight verification on every external integration is a standing reflex captured in the project's CLAUDE.md.

**13. `Prompt.category` flexible string vs strict enum**
Vertical-specific extensions (consumer brand: "occasion / pairing / anti-celebrity"; legal: "compliance / intake / discovery") get added at seed time without a migration. `Engine` and `Sentiment` stay strict because they're universal across clients. Right thing strict, right thing flexible.

**14. Methodology page as the credibility wedge**
Every competitor treats their math as proprietary. Publishing it at `/methodology` (Wilson formula, AutoGEO citation, sample-size rationale, what's deferred to v2 and why) inverts the trust dynamic. The page is shippable in front of clients during engagements — the moat is the thing you'd think you should hide.

**15. Premortem applied → adjustments visible in commit log**
A formal premortem on the pilot client meeting surfaced four high-stakes-low-likelihood failure modes (connector misalignment, existing-vendor block, win-meeting-lose-engagement, methodology defense). Output rewrote the deliverable around the moat instead of around the metrics. Premortem isn't decoration; it changed the artifact.

**16. Cost ceiling enforced via env var**
`COST_CEILING_USD` validated at boot in `lib/env.ts` (default $20, baseline back-of-envelope $7.50). Scheduler aborts if projected spend exceeds. A single bad prompt-bank can't burn the budget silently.

---

### 5. Government Relations Intelligence Dashboard

**Status:** Live and autonomous. GitHub Actions cron fires at 6:30am PT daily. Pipeline is fully unattended; the operator is a passive reviewer behind a Cloudflare Access whitelist. Two repos: a public pipeline and a private dashboard repo (which also holds state). Cloudflare Pages auto-deploys on push, ~90s lag.
**Use case archetype:** A daily, pre-7am governance briefing for a multi-role public-affairs operator (government relations + community-college trusteeship + traffic commission). Five public meeting and legislation sources are scraped, diffed against yesterday, summarized through a verbatim + NER grounding gate, and published as a static dashboard before the operator opens their laptop — no inbox, no copy-paste, no manual triage.

#### The Problem
A working public-affairs operator with three concurrent roles can't manually read every relevant agenda, bill calendar, board packet, and PDF every morning. Off-the-shelf LLM summarization is too risky — fabricated dollar figures or invented bill numbers break trust with internal stakeholders the moment they're caught once. The need: a daily intelligence pipeline that's both comprehensive (multi-source) and grounded (every fact provably extracted from primary text).

#### What It Is
An autonomous overnight pipeline that scrapes five government and legislative sources, diffs against yesterday's snapshot, summarizes new items through a two-layer hallucination gate (verbatim substring + spaCy NER), and renders a static HTML dashboard for the operator to skim before 7am. Five-source coverage: Playwright/Chromium for JS-rendered transit and city government sites, `pdfplumber` for PDF agendas with page-marker citation deep-links, `pyopenstates` for the state legislature API.

#### Architecture

```
GitHub Actions cron (6:30am PT)
  → pull_state (clone dashboard repo, read last_seen.json + history.json + items_by_day.json)
  → 5 scrapers in parallel:
       Playwright/Chromium  (3 JS-rendered sources)
       requests + pdfplumber (1 source)
       pyopenstates API     (state legislature)
  → content_hash diff vs last_seen.json → only NEW items survive
  → Gemini Flash 2.5 summarizer (one call per source, prompt-injection isolated)
  → verbatim-substring gate + spaCy NER gate → drops hallucinated items
  → Jinja2 renderer (two-pane unified feed; pin/save via localStorage)
  → publisher: atomic single-commit push (HTML + state) to dashboard repo
  → Cloudflare Pages auto-deploys
  → SMTP alerts on quiet-everything OR push-fail OR multi-source degraded
```

#### Stack
- Python 3.12 (CI parity) / 3.14 (local). `requirements-lock.txt` for reproducible CI.
- GitHub Actions for cron + concurrency control (`cancel-in-progress: false`)
- Playwright + headless Chromium for JS-rendered sources
- pdfplumber (10MB cap, 30s streaming timeout, alnum + stopword readability heuristic)
- pyopenstates >=2.3.1 for state legislature (free tier 10 req/min)
- google-generativeai (Gemini 2.5 Flash primary, Gemini 1.5 Flash fallback, raw-excerpt fallback below that)
- spaCy 3.8 + en_core_web_sm for NER grounding
- Jinja2 with autoescape; vanilla JS pin/save module (no frontend framework)
- GitPython for commit/push operations
- Cloudflare Pages + Cloudflare Access (Google OAuth) for delivery + auth
- SMTP alerts via Zoho (app-specific password)

#### Key Decisions and Why They're Elegant

**1. State lives in a separate repo, not the pipeline repo**
`last_seen.json`, `history.json`, `items_by_day.json` and the rendered `index.html` all live in the private dashboard repo. The pipeline repo is stateless. This means the pipeline can be made public for portfolio purposes without leaking who the operator monitors, and dashboard rollback is `git revert` not state surgery.

**2. Two-layer hallucination gate (verbatim substring + spaCy NER)**
The model is forbidden from authoring the fact sentence. `display_text` must be a verbatim 80–500-char span from `raw_text`, AND every named entity in the model's `headline` must appear in that span (with ±1% tolerance for `MONEY` because rounding is legitimate). Bill numbers and contract numbers are extracted via regex on top of NER because spaCy mis-labels `LAW`. Fabricated dollar figures and bogus bill numbers are structurally impossible.

**3. Allow-list for the operator's own role-set in `why_relevant`**
The NER gate would reject "this matters to a transit-agency GRM" because role names don't appear in the verbatim excerpt. A small allow-list of role/jurisdiction entities (the operator's three professional hats) lets relevance commentary mention them without faking new facts. Separates "fact must be grounded" from "context can reference role."

**4. Gemini safety filters set to BLOCK_NONE for governance**
Default consumer-safety filters BLOCK on crime, drugs, housing, public-health mentions, which kills governance summaries (police contracts, homeless services, opioid settlement). All four categories explicitly set to `BLOCK_NONE`. Without this the system would silently false-positive-block half its real content.

**5. HTML body as a content floor under PDFs**
Source PDFs are growing past the 10MB cap. The scraper always extracts the detail page body text first (capped at 30KB) so the summarizer has *something* to ground against even when every PDF fails. PDF text layers on top when extraction succeeds. The system degrades gracefully from rich → metadata-only instead of empty.

**6. PDF page markers for citation deep-links**
Scrapers inject `=== [PDF N p.P] ===` into raw_text before each page. The summarizer prompt reads them to emit `source_page`. The renderer stitches `url#page=N`. Markers are stripped from display copy AFTER the grounding substring check so verification sees the original characters on both sides. The reader clicks the citation and lands on the exact PDF page.

**7. Per-source one-call-per-summary prompt-injection isolation**
Five sources, five separate Gemini calls. A malicious PDF in one source can't contaminate items in another. The renderer also rejects model-emitted `item_url` values with non-http schemes (`javascript:`, `data:`) — so even if injection succeeds, the link can't escape.

**8. Same-day idempotence with `--force` escape hatch**
`_already_ran_today` reads `history[0]` and exits 0 if today is already covered. A failed-then-re-run pattern bypasses with `--force`. `save_state` upserts (drops any existing same-date entry before prepending) so multiple force-runs don't pile up. The dashboard is repeatable without losing yesterday.

**9. Rolling 14-day item store with most-recent-anchor preservation**
`items_by_day.json` keeps 14 days, renderer uses last 7. When a source has no in-window content (some sources meet monthly), the truncator preserves the single most-recent older content-bearing entry as an anchor, so empty-state UX shows "Most recent: [date] — [brief]" instead of "No items yet." Solves a real UX failure where quarterly-cadence sources looked broken.

**10. PAT scrubbing on every git error path**
Every `git.GitCommandError` echoes the full argv on failure, including the `https://<token>@github.com/...` URL. State manager and publisher both wrap git calls in `_redact()` that scrubs both the raw token AND its base64 form (because Basic-auth headers also leak to logs). After clone, `set_url()` rewrites `.git/config` to remove the token — auth rides per-command `http.extraheader` instead of being persisted.

**11. `_pre_summarized` items bypass the diff**
Some sources publish meeting *schedule skeletons* with no agenda yet. Their `content_hash` is stable across runs; normal diff logic would mark them "already seen" forever. The diff explicitly passes them through regardless of `last_seen` membership, with renderer-side dedup via `items_by_day.json`. Calendar metadata stays visible without polluting the news diff.

**12. Three-state source semantics on the dashboard**
`active` (scraped + survived NER), `quiet` (scraped ok, zero new), `degraded` (scrape exception OR all items dropped by gate OR fallback used). The operator can tell "nothing happened" from "the system is broken" at a glance. Multi-source degraded triggers an SMTP alert even though the dashboard still ships — quality off, not down.

---

### 6. edge_lab — Trading Analyst System

**Status:** Live
**Location:** `/Users/t-rawww/edge_lab/`
**Config:** `CLAUDE.md` + `GEMINI.md`

#### The Problem
Trading frameworks are easy to write and hard to follow under real-time pressure. Without a system that enforces your own rules at every decision, steps get skipped — macro alignment unchecked, position sizing eyeballed, past trade history ignored. The failure mode isn't a bad framework. It's a good framework that doesn't get used.

#### What It Is
A session-aware swing trading analyst grounded in a pre-built framework. Not a generic chatbot — it reasons strictly within my macro bias, key levels, portfolio state, and trade journal. It stress-tests my thesis, never generates one from thin air.

#### Architecture

```
edge_lab/
├── CLAUDE.md / GEMINI.md     ← Behavioral contract (dual AI)
├── context/
│   ├── macro-outlook.md      ← Current HTF bias (agent updates in-session)
│   ├── positions.md          ← Current positions, risk snapshot
│   ├── market-snapshot.md    ← Generated indicator data (make refresh)
│   ├── macro-snapshot.md     ← Macro conditions
│   └── pending-setups.md     ← Watchlist — auto-price-checked at session start
├── levels/
│   ├── macro-levels.md       ← Yearly/quarterly opens per symbol
│   └── watchlist.md          ← Monthly/weekly opens + key levels
├── journal/
│   ├── INDEX.md              ← Built by scripts/build-index.py
│   ├── template.md
│   └── YYYY-MM-DD-[SYM]-[TF].md  ← Trade entries
├── playbook/
│   ├── README.md             ← Index + matching guide (always read first)
│   └── [one file per setup type]
├── framework/                ← Personal trading framework rules
├── reviews/                  ← Weekly/periodic review outputs
├── scripts/
│   ├── calc.py               ← 12-command calculator — all math offloaded here
│   ├── position-sizer.py     ← --calc mode: shares, % at risk, buffer impact
│   ├── live-prices.py        ← yfinance multi-ticker fetch
│   ├── build-index.py        ← Rebuilds journal/INDEX.md
│   ├── check_alerts.py       ← Watchlist alert logic
│   ├── fetch_market_state.py ← Market condition fetch
│   ├── weekly-summary.py     ← Weekly review generation
│   └── [full test suite]     ← test_*.py for every script
└── Makefile                  ← `make refresh` etc.
```

#### Key Decisions and Why They're Elegant

**1. Context as files, not conversation memory**
All state lives in structured markdown — macro outlook, open positions, watchlist. Agent reads fresh each session. No hallucinated continuity from chat history.

**2. Dual AI portability**
CLAUDE.md and GEMINI.md are the same behavioral contract with different tool dialects. When Claude usage hits limits, Gemini picks up with zero behavior drift. Tool mapping:
```
Read → read_file  |  Write → write_file  |  Edit → replace
Bash → run_shell_command  |  (+) save_memory  |  (+) ask_user
```

**3. Auto-stale detection**
On every chart upload: checks `market-snapshot.md` timestamp. >24h → flags immediately. Macro outlook >7 days → flags. Prevents silent stale-data analysis.

**4. Watchlist price monitoring at session start**
`pending-setups.md` stores symbol + target level + condition. At session start, agent fetches live prices via yfinance and flags any ticker within 3% of its target. Zero manual checking required.

**5. Journal-driven similarity matching**
Before any analysis, reads `journal/INDEX.md` → finds 3 structurally similar past setups → reads full entries. Analysis grounded in personal trade history, not generic TA patterns.

**6. Non-interactive position sizer**
`position-sizer.py --calc` runs inline during trade discussion (not after). Pulls live position sizing data from context. Outputs shares, % at risk, buffer impact, binding constraint.

**7. Session close = git push**
End-of-session protocol: list changed files → rebuild journal index → `git add -A && git commit && git push`. Framework versioned automatically.

**8. Two types of index files**
edge_lab has two distinct index patterns serving different purposes:

- `journal/INDEX.md` — **dynamically generated** by `scripts/build-index.py` every time a trade entry is added. The agent never manually maintains it. Run the script, it rebuilds from scratch. Always accurate.
- `playbook/README.md` — **mandatory-first-read** index. The CLAUDE.md explicitly enforces: *"ALWAYS read playbook/README.md first — it is the index. Use the Matching Guide to identify the correct setup file, then open only that file. Never open individual playbook files without reading the README index first."*

The distinction matters: the journal index is a catalog (what exists), the playbook index is a router (which one to open). Different jobs, different maintenance models.

**9. `calc.py` — math fully offloaded to Python**
12-command calculator covering every calculation type in swing trading: R:R, R multiple, week R, P&L, exposure, drawdown, avg cost, $ at risk, spot conversion, price vs levels. CLAUDE.md has a hard rule: agent never computes numbers in-context, always calls `calc.py` and shows output directly. Solves a real failure mode — LLM arithmetic compounds errors in long-context sessions; Python doesn't.

**10. Tested scripts**
Every script in `scripts/` has a corresponding `test_*.py`. Production-grade practices applied to a personal trading tool.

**11. Two-mode news system**
Two distinct protocols with different jobs — not one "news" command:

- **Morning Brief** (`/morning-brief` or "morning brief"): Runs last30days on 5 X accounts + Polymarket, filters output to facts only (strips sentiment, predictions, opinions), writes to `context/news-snapshot.md`. Curated, persistent, 4-6 bullets. Feeds the analysis context.
- **Daily Digest** (`/daily-digest` or "daily digest"): Runs last30days 5 times — one targeted call per account with account-matched topic keywords. Synthesizes into thematic narratives + notable events + upcoming items. Session only, never writes to a file. Comprehensive, informational, not tied to any analysis workflow.

The separation matters: morning brief is a context-preparation tool (facts → file → analysis). Daily digest is a reading tool (synthesis → understand what happened today). Same accounts, completely different intent and output.

**12. Global slash command skills**
`/morning-brief` and `/daily-digest` installed at `~/.claude/skills/` — available in any Claude Code session globally, not just edge_lab. Each skill is self-contained with the full protocol so it works standalone. The auto-triggers in CLAUDE.md also remain, so natural language and slash commands both work.

**13. `fetch-digest.py` — parked custom X scraper**
Built a direct X API scraper using `requests` + session cookies (AUTH_TOKEN + ct0) that bypasses last30days entirely. Pulls all original posts from all 5 accounts with recency-boosted engagement scoring and ID caching to avoid rate limits. Parked pending a valid Bearer token (last30days's internal bird-search client handles this; raw requests cannot use a stale public token). Lives at `scripts/fetch-digest.py` — activate by adding `X_BEARER_TOKEN` to `.env`.

---

### 7. Zenkai — Personalized AI Learning App

**Status:** Functional — end-to-end working for Module 2. 8 modules remaining for full content coverage.
**Repo:** `trentjhn/zenkai`
**Design doc:** `/Users/t-rawww/AI-Knowledgebase/docs/plans/2026-02-28-zenkai-design.md`

#### The Problem
Reading well-written reference material doesn't create retention. The AI-Knowledgebase had hundreds of pages of synthesized AI engineering content — but without active recall and spaced repetition, it stayed reference-only: useful to look up, not useful to apply from memory. The vision: something like Duolingo for AI, where learning is interactive, gated by mastery, and actually engaging.

#### What It Is
A local web app that turns this knowledge base into an interactive learning experience. Named after the DBZ Zenkai boost. Spaced repetition, scenario-based quizzes with AI PM framing, module gating (≥70% to unlock next).

#### Stack
- Frontend: React + Tailwind + shadcn/ui + Framer Motion
- Backend: FastAPI (Python)
- DB: SQLite (local)
- AI: Claude claude-sonnet-4-6 via Anthropic API

#### Key Design Decision
**Delta sync via git commit hashes** — compares KB file hash against last-generated hash. Only regenerates quiz content for files that actually changed. Avoids burning API credits on unchanged content every launch. The KB is the source of truth; Zenkai derives from it.

---

### 8. AI-Knowledgebase — Personal Knowledge Library

**Status:** Live (continuously growing)
**Location:** `/Users/t-rawww/AI-Knowledgebase/`
**Repo:** `trentjhn/AI-Knowledgebase` (private)

#### The Problem
AI engineering knowledge is scattered across 100+ papers, documentation sites, and blog posts of wildly varying depth. Most tutorials are either too shallow for practitioners or benchmark-focused — which decays fast as models improve. No single resource synthesized core concepts, frameworks, and operational playbooks at practitioner depth in a form that aged well.

#### What It Is
A personal AI knowledge library built for genuine depth — distilled, practitioner-level reference docs on core AI engineering topics. Built for personal use and as a methodology foundation for future consulting/audit work.

#### Architecture

```
AI-Knowledgebase/
├── CLAUDE.md                ← Behavioral contract + last30days research trigger
├── KB-INDEX.md              ← Navigation layer (line counts, read times, one-liners)
├── builds-log.md            ← This file
├── LEARNING/
│   ├── FOUNDATIONS/
│   │   ├── reasoning-llms/
│   │   ├── multimodal/
│   │   └── emerging-architectures/   ← Frontier monitoring (not how-to)
│   ├── AGENTS_AND_SYSTEMS/
│   │   ├── agentic-engineering/
│   │   ├── context-engineering/
│   │   ├── skills/
│   │   └── mcp/
│   ├── PRODUCTION/
│   │   ├── evaluation/
│   │   ├── fine-tuning/
│   │   ├── inference-optimization/
│   │   └── rl-alignment/
│   ├── SECURITY/
│   │   └── ai-security/
│   └── ai-governance/
├── future-reference/
│   ├── playbooks/           ← How-to protocols (building agents, RAG, websites...)
│   ├── prompt-catalog/
│   └── specs/
└── CAREER/
```

#### What's Elegant Here

**1. Distillation as the primary operation**
Each doc synthesizes 10–20+ primary sources (papers, docs, blog posts) into a single readable reference. Not a bookmark list, not raw notes — a genuine synthesis with narrative explanation before bullets, analogies for abstraction, "why this matters" for every concept. The writing standard was set by explicit feedback: educational first, reference second.

**2. Two-tier structure: LEARNING vs future-reference**
- `LEARNING/` is evergreen conceptual knowledge (what things are, how they work)
- `future-reference/` is operational protocols (how to build things)
These are intentionally separate. Mixing them creates reference docs that are neither good to learn from nor good to execute against.

**3. Layered README index hierarchy**
This is the most architecturally deliberate thing in the KB — a 4-level README cascade that lets any reader (or AI agent) orient at any depth:

```
README.md                           ← Root: "what is this repo, what's where"
└── LEARNING/README.md              ← Section: learning path guide (Foundation→Build→Ship)
    ├── FOUNDATIONS/README.md       ← Subsection: what's in FOUNDATIONS, how long, prereqs
    ├── AGENTS_AND_SYSTEMS/README.md
    └── PRODUCTION/README.md
└── future-reference/README.md     ← Section: practical tools vs. study material
└── KB-INDEX.md                    ← Flat catalog: file paths, line counts, read times
```

Each level answers a different question:
- Root README: "Where do I start?"
- Section READMEs: "What's in this section, how long will it take, what should I read first?"
- KB-INDEX.md: "I know what I want — where exactly is it?"

This hierarchy means an AI agent dropped into this repo can navigate without being told anything. It reads README.md, gets oriented, follows the links to the right section README, then hits KB-INDEX.md for the specific file. **Progressive disclosure for navigation.**

**4. Frontier monitoring as its own document type**
`emerging-architectures.md` is intentionally not a tutorial. It's a Signal vs. Noise framework for evaluating new research — SSMs/Mamba, MoE, byte-level models, continuous/latent space. Designed to age well. The evaluation framework matters more than specific benchmarks, which decay fast.

**5. CLAUDE.md with last30days research trigger**
The KB now has a behavioral contract. The key rule: when asked to research or update any topic, automatically run `/last30days [topic] --sources=hackernews,youtube` before synthesizing. HN surfaces what engineers have actually discovered in the field; YouTube pulls transcript content from builders shipping real products. Output is session-only context — it supplements primary sources, never replaces them.

**6. builds-log.md (this file) as the meta-layer**
A record of what was built, why, and what patterns are running across systems. Turns building into deliberate practice — not just shipping and forgetting.

**7. ArXiv Research Digest — Automated KB-aligned paper sourcing**
Infrastructure that feeds the KB. Problem: finding high-signal papers manually is inefficient (Twitter ~2/week, ArXiv raw ~50/week at 90% noise). Solution: automated weekly digest that queries ArXiv across 9 KB topics (past 7 days), scores with Claude for KB relevance (not citations), outputs 8 papers/week directly updatable into KB.

*Stack:* ArXiv API (9 topics) → Claude Opus batch scoring → GitHub Actions workflow (Friday 8am UTC) → `/raw/arxiv-papers/YYYY-MM-DD.md` digest.

*Key insights:* (1) Citation lag makes citations a poor signal; Claude scores immediate relevance. (2) KB-aligned criteria prioritize mechanisms over metrics, failure modes over successes, reusable patterns over domain applications. (3) Past 7 days beats 1-4 weeks — high-signal papers surface immediately. (4) Single batch call is cheaper and gives Claude full context for relative comparison. (5) Version-aware ID normalization prevents silent failures. (6) 0.8+ relevance threshold yields ~8 papers/week (vs 50 with citation-only filtering) — quality over volume.

---

### 9. interview-prep — Job Search OS

**Status:** Live
**Location:** `/Users/t-rawww/interview-prep/`
**Config:** `CLAUDE.md` + `GEMINI.md`

#### The Problem
Running 10+ concurrent job applications means managing unique context for every company, hiring manager, interview format, and STAR story alignment — across weeks of sessions that share no memory. Notes apps stay static. Chat history resets. There was no system built for the actual complexity of a multi-track job search campaign.

#### What It Is
A full job search operating system — not a notes folder. Tracks 10+ active/passive opportunities, stores STAR stories, maintains interview schedules, logs every session, and holds all core materials. The CLAUDE.md doubles as a live CRM.

#### Architecture

```
interview-prep/
├── CLAUDE.md / GEMINI.md    ← Behavioral contract + live pipeline state
├── core-materials/
│   ├── paypal-narrative.md  ← PayPal PM story (the origin story)
│   ├── paypal-metrics.md    ← Verified numbers (140M logins, $2.4M margin, 52% lift)
│   ├── behavioral-stories.md ← STAR story beats (4 stories, deploy-ready)
│   ├── master-pitch.md      ← Pitch versions (60s, 2min, modular)
│   ├── pm-playbook.md       ← Reusable frameworks
│   └── pm-frameworks-refresher.md ← RICE, CIRCLES, metrics
├── companies/               ← 27 folders, one per company tracked
│   ├── mariana-minerals/
│   ├── bridgestone/
│   ├── ziff-davis/
│   └── [24 more...]
├── sessions/                ← Date-stamped session logs (YYYY-MM-DD.md)
├── session-recaps/          ← Condensed recaps for review
├── templates/               ← Reusable interview templates
├── docs/                    ← Supporting documents
└── side-hustle/             ← Parallel tracks, consulting exploration
```

#### Key Decisions and Why They're Elegant

**1. CLAUDE.md as live CRM**
The CLAUDE.md holds the full pipeline: 10 ranked opportunities, interview schedules, key contacts with context, STAR stories, what's working/not working. Every session starts with this loaded — no re-explaining. It gets updated in-session as new information comes in.

**2. Company folders as atomic context units**
Each of the 27 companies has its own folder. Company-specific prep, call notes, and research are isolated — never bleeds between opportunities. When prepping for a specific call, load only that company's folder.

**3. Session log as institutional memory**
Daily sessions logged at `sessions/YYYY-MM-DD.md`. Recaps live in `session-recaps/`. Can look back and trace exactly when a belief changed, when an opportunity shifted, what worked in a call.

**4. human-voice skill integration**
All outreach drafts (emails, thank-yous, LinkedIn messages) run through the `human-voice` skill first. Enforces direct, Oakland-rooted voice — not corporate filler. The behavioral rule is in the CLAUDE.md itself: no em dashes, no motivational poster BS.

**5. Dual AI portability**
GEMINI.md exists for the same reason as edge_lab — continuity across AI tool switches mid-job-search.

**6. Communication style baked in as constraints**
The CLAUDE.md has explicit language rules: no em dashes, no filler phrases, don't over-rehearse, simplify when brain fog hits. These override default AI communication tendencies at the system level.

---

### 10. Domain-Specialized PRD Generation System

**Status:** Complete
**Type:** Local Claude Code project, config-as-code via `CLAUDE.md`

#### The Problem
Generic PM templates solve the wrong problem in industrial domains. A standard PRD template optimizes for DAU/MAU, but in mining software you measure recovery rate and cost per ton. It assumes users are individual product managers, but plant operators and process engineers have different pain points. It ignores physical constraints (ore composition, flotation chemistry, equipment specs) that drive every technical decision. Using a generic template in a domain-specific case study produces a PRD that *looks* professional but solves the wrong problem at every section.

#### What It Is
A purpose-built PRD generation environment for a specialized industrial domain (mining and minerals-processing software). Not a generic PM template: it enforces domain-correct framing, terminology, and physical constraints so the output solves the right problem at every section instead of looking professional while missing the domain.

#### Architecture

```
prd-system/
├── CLAUDE.md               ← Behavioral contract + language rules
├── QUICKREF.md             ← Fast reference for a working session
├── context/
│   ├── company.md          ← product lines, personas, metrics, tensions
│   └── glossary.md         ← Industrial terminology (SX-EW, flotation, RFI, EPCC, P&ID...)
├── ai-reference/
│   └── ai-pm-framing.md    ← Load only when prompt involves AI/ML explicitly
├── templates/
│   └── prd-template.md     ← Custom domain PRD template (NOT generic PM template)
└── output/
    ├── PRD-AutomatedBidEvaluation.md    ← Produced during a session
    └── PreMortem-AutomatedBidEvaluation.md
```

#### Key Decisions and Why They're Elegant

**1. Specialized language rules as hard constraints**
The CLAUDE.md bans generic PM vocabulary and forces industrial-specific equivalents:
```
"users"     → plant operators / process engineers / mine planners
DAU/MAU     → recovery rate / cost per ton / schedule variance
"edge case" → failure mode (treat as first-class requirement)
"validate"  → "test this assumption before V2 by [specific method]"
```
This forces the agent to sound like an industrial PM, not a consumer tech PM. The domain framing is enforced at the system level.

**2. 3-phase working session: frame, build, stress-test**
The CLAUDE.md drives a timeboxed PRD session:
- **Phase 1, Frame:** which product, primary persona, core tension, biggest constraint
- **Phase 2, Build:** `/prd` command, reasons first, asks clarifying questions, builds using the domain template
- **Phase 3, Stress-test:** `/premortem` command, Tigers (real risks), Paper Tigers (overblown), Elephants (unspoken)

**3. Conditional context loading**
`ai-pm-framing.md` is loaded **only** when the prompt involves AI/ML explicitly. Domain-specific context on demand — not always loaded. Keeps the context window efficient.

**4. Output folder**
Session outputs saved to `output/`. The PRD and PreMortem are persistent artifacts, reviewable and reusable after the session.

**5. "Fill structure, not words" principle**
The CLAUDE.md frames the agent role explicitly: surface structure and risks, not replace PM judgment. After generating any section, it offers 1-2 questions for the human to take back to stakeholders. Keeps the human's reasoning primary.

**6. Thinking framework baked in**
Every feature analysis forces: systems first (inputs→process→outputs→feedback), costs always (capital/operating/time/risk), three different people (who approves, who uses, who gets fired). Physical constraints named — never ignored.

---

### 11. Parking Lead-Gen Agent

**Status:** Functional and actively iterated. Real cost data: 14 runs in `state/spend-ledger.jsonl` with measured costs ranging $0.024–$0.20 per run depending on radius and category breadth.
**Repo:** Private

#### The Problem
Out-of-home advertising sales teams source local advertisers manually — Yelp, Google Maps, a spreadsheet, a phone book. The work is geographic search + quality filtering + contact enrichment + fit scoring, repeated for every parking garage or transit hub on the asset list. Hours per asset, low yield, no audit trail of why a given prospect was contacted. The opportunity: collapse the loop into a single CLI that returns a contact-enriched ranked list for ten cents.

#### What It Is
A Python CLI that takes a parking-garage address and returns a ranked, contact-enriched list of nearby businesses who are strong candidates to place ads in that garage. One run = one CSV in `output/{YYYY-MM-DD_HHMMSS}_{address-slug}/leads.csv`. Includes fit-score rationale, contact (name + title + quality) where available, and a fallback LinkedIn search URL when email enrichment fails. Audit-friendly by design: every stage persists JSON to disk, every run appends to a spend ledger, every output CSV is timestamped and addressable for diff.

#### Architecture

```
address
  → prospector.py     Google Places API (New) + Geocoding + haversine    → data/raw_businesses.json
  → filters.py        rating floor + chain blocklist + airport-terminal reject
  → enricher.py       website scrape (BeautifulSoup) + Hunter.io domain search → data/enriched_businesses.json
  → qualifier.py      Claude Haiku 4.5 tool_use scoring (5-worker pool, rate-limit-aware retry) → data/qualified_leads.json
  → exporter.py       contact-forward CSV + run summary (cost, runtime, Hunter quota, top-3 preview)
                      → output/{YYYY-MM-DD_HHMMSS}_{address-slug}/leads.csv
  → ledger.py         append per-run row to state/spend-ledger.jsonl
```

Single-threaded Python orchestration with parallelism only inside the qualifier stage. Every stage persists JSON to `data/` so any stage can be replayed via `--resume-from {prospect,enrich,qualify}` without re-hitting upstream APIs. State (Hunter cache, Hunter usage counter, spend ledger) lives in a separate `state/` directory that survives `make clean`.

#### Stack
- Python 3.11+ (Makefile enforces via `check-python` target)
- **APIs:** Google Places API (New) `places:searchNearby` (legacy endpoint banned in CLAUDE.md), Google Geocoding, Hunter.io domain search, Anthropic Claude Haiku 4.5 with native `tool_use` structured output and ephemeral prompt caching
- **Libraries:** `anthropic==0.96.0`, `requests==2.33.1`, `beautifulsoup4==4.14.3`, `python-dotenv==1.2.2`; `pytest==9.0.3` + `pytest-mock` + `responses` for HTTP mocking
- **Makefile targets:** `install`, `setup` (interactive `.env` wizard), `run ADDRESS=...`, `dry`, `test`, `clean` (preserves `state/`), `budget-report` (last-7-day spend from JSONL), `ship` (git archive zip in `dist/`)
- 13 test modules including `test_eval.py` (offline eval fixture) and `test_integration.py` (full pipeline with mocked Places + tool_use)

#### Key Decisions and Why They're Elegant

**1. Anti-hallucination contract as Rule 1**
Claude never recalls business facts from training data, every fact is injected as structured JSON from Places or Hunter. The qualifier user message is `json.dumps({garage:..., business:...})` with only name, address, distance, category, rating — no review text, no description that would invite the model to fill in the blank. Tool_use schema enforces output shape (`tier`, `fit_score`, `score_rationale`) so there's nothing to parse, nothing to hallucinate.

**2. Interim JSON as source of truth between stages**
Every stage reads from disk and writes to disk. `--resume-from` is therefore a real product feature, not a debug toggle. Tuning a min-score? Don't re-pay Places + Hunter; replay from `qualified_leads.json`. Same context-as-files pattern from edge_lab applied to a one-shot pipeline.

**3. `state/` separated from `data/`**
`data/` holds replayable interim JSON (gitignored, wipeable). `state/` holds cross-run persistent assets the operator should never lose: Hunter monthly-quota counter, Hunter domain cache, spend ledger. `make clean` wipes the first and protects the second by directory boundary, not by an `.gitignore` exception that's easy to forget.

**4. Dry-run fixture mode (`make dry`)**
Demos and onboarding cost zero dollars and zero network. The dry path loads `tests/fixtures/dry_run_businesses.json`, skips Hunter and Anthropic entirely, and still exercises the filter → checkpoint → export → summary path so a new operator sees the real CSV shape before they pay for keys. `--dry-run` takes precedence over `--resume-from` with a logged warning instead of a silent override.

**5. Three-method email extraction with explicit fallbacks**
Enricher tries: HTML scrape → JS-unicode-unescape pass (catches `>owner@…` from React/Next.js sites) → Hunter domain search. Junk filter rejects `noreply@`, blocklisted demo domains, and DIY hosting platforms (`wixsite.com`, `squarespace.com`) where Hunter would query the platform instead of the business. When all three fail, a Google `linkedin_search` URL pre-baked from business name + city ships in the CSV — a free fallback that costs nothing and the operator clicks once.

**6. Three contact columns collapsed to one cell**
`_format_contact` renders `Jane Doe — Director of Marketing (senior_manager)` in a single `contact` cell. The dataclass still carries `contact_name`, `contact_title`, `contact_quality` separately so the sort tiebreaker (reachable leads rank higher on equal fit_score) and `--resume-from` replay still work. Collapse happens at the export boundary, not the model.

**7. Scoring cluster surfaced together in the CSV**
`rating`, `review_count`, `fit_score`, `score_rationale` sit adjacent in `CSV_FIELDS`. Operator scanning the list can sanity-check Claude's score against the underlying signal without flipping between sheets. Came from TDD (RED test first, then GREEN feat), not afterthought.

**8. Timestamped + slugged output directory**
`output/{YYYY-MM-DD_HHMMSS}_{address-slug}/leads.csv`. Earlier date-only naming silently overwrote prior CSVs on same-day re-runs, defeating the whole point of parameter-tuning. The `HHMMSS` suffix makes every run addressable for diff. Slug uses NFKD + combining-mark strip so accented Latin (`Café`) maps to ASCII (`cafe`) instead of collapsing to `unnamed`.

**9. Rate-limit-aware retry with two regimes**
`RateLimitError` honors the server's `retry-after` header (clamped to [1, 60]s, defaults to 30s when absent), up to 3 retries. Non-rate-limit exceptions get one retry at 2s. The split came from a real failure where a fixed 2s backoff landed both retry attempts in the same throttle window. Failed leads surface as `tier=ERROR` rows rather than silent drops.

**10. Two-layer budget guard**
Pre-flight: project tokens × leads, abort before any Claude call if over `--max-cost`. Mid-run: each worker checks `cost_tracker.total_usd >= max_cost` before its own call, and over-budget leads exit as `tier=BUDGET` rows. Wrapped by safety caps (`--max-cost` ≤ $5, `--radius` ≤ 5mi without `--allow-expensive-run`) and an interactive confirm for permissive-everything footguns. This is the spend discipline that lets the README advertise "$0.05–$0.15/run" with a straight face.

**11. `--top-n 50` default with full pool retained in checkpoint**
Operator-facing CSV is capped at 50 rows so the downstream marketing operator opens a tight list, not a long tail. Truncation happens *after* the qualified-leads checkpoint write, so re-exporting with a different cap from the same `data/qualified_leads.json` is free. Negative values rejected explicitly — Python's negative-slice semantics would silently mean "all but the last N".

**12. CSV formula-injection guard at export boundary**
`_csv_safe` prefixes any cell starting with `=`, `+`, `-`, `@`, `\t`, `\r` with a single quote so Excel/Numbers/Sheets render literal text instead of executing a formula. RFC 4180 is silent on this; it's a spreadsheet UX hazard, so the fix lives where CSV meets spreadsheet, not in the model layer. UTF-8 BOM (`utf-8-sig`) added so Windows Excel auto-detects encoding.

---

### 12. security-var-agent — Value-Added Reseller Recommendation Engine

**Status:** Functional (reached solid state, business case closed)
**Location:** `/Users/t-rawww/AI-Agent-Project/`
**Repo:** `trentjhn/security-var-agent` (GitHub public)

#### The Problem
VAR workflows require evaluating dozens of vendors across multiple dimensions (market position, technical fit, ROI, implementation complexity). Manual comparison is error-prone and time-consuming. Spreadsheet-based scoring misses market context and ROI implications. The result: recommendations based on incomplete analysis or vendor familiarity rather than systematic evaluation.

#### What It Is
A modular service-oriented architecture for comprehensive software solution recommendations. Analyzes client requirements, market conditions, and technical constraints to produce evidence-based vendor recommendations with ROI projections, implementation timelines, and confidence scoring.

#### Stack
- **Frontend:** React + TypeScript (AI Advisor chat interface, side-by-side recommendation comparison)
- **Backend:** Node.js + TypeScript (modular service architecture)
- **Services:** Context Understanding, Analysis, Recommendation Engine, ROI Calculator, Implementation Planning
- **Data:** Market Data Cache Service (99.9% hit rate), Confidence Scoring System
- **Infrastructure:** Supabase (data persistence), Jest (comprehensive test coverage)

#### Key Decisions and Why They're Elegant

**1. Modular service-oriented architecture**
Six independent services (Context, Analysis, Recommendation, ROI, Implementation, Data Freshness) composed by a central orchestrator. Each service is independently testable and replaceable. Changes to vendor scoring logic don't affect ROI calculation or confidence scoring.

**2. Market data cache with 99.9% hit rate**
Market data freshness is critical — recommendations should reflect current vendor positions and pricing. Caching strategy avoids redundant API calls while maintaining real-time context. Cache invalidation rules ensure stale data is caught before reaching recommendations.

**3. Confidence scoring system**
Not all recommendations are equally strong. The system quantifies confidence across four dimensions: market validation (vendor track record), technical fit (architecture compatibility), financial viability (ROI > threshold), and implementation feasibility (timeline realistic). Scoring is transparent to users — they see which recommendations are high-confidence vs. cautious.

**4. ROI calculator with detailed financial modeling**
Beyond "this vendor is cheaper" — the system models total cost of ownership (licensing + implementation + training + maintenance), payback period, and break-even timelines. Financial output is evidence-based and defensible.

**5. Implementation timeline visualization**
Recommendations include detailed phased timelines — pre-implementation (discovery, vendor selection), implementation (deployment, integration, testing), and post-implementation (training, optimization). Timelines account for parallel vs. sequential work, vendor constraints, and client readiness.

**6. TypeScript for type safety across service boundaries**
Services communicate via strict interfaces — a change to the Recommendation output format is caught at compile time across all consumers. Prevents silent failures where one service produces data another service doesn't expect.

**7. Comprehensive test suite**
Jest coverage for service logic, recommendation scoring, ROI calculations, and confidence algorithms. Ensures scoring logic stays consistent as vendor datasets and market conditions change.

**Project Status:** Built to specification for a VAR workflow automation opportunity. Reached functional state with core services operational. Client opportunity closed, but the system demonstrates patterns (modular scoring, financial modeling, confidence transparency) applicable to other recommendation engines (cloud provider selection, SaaS stack optimization, etc.).

---

## What These Systems Demonstrate

Collectively, these twelve systems (plus ArXiv sourcing infrastructure) demonstrate depth across the full AI engineering stack:

**Full-Stack Capability**: From production deployment (YouTube Summarizer), full-stack architecture (Zenkai), to real-time automation (edge_lab). Not just backend or frontend — end-to-end product thinking.

**Sophisticated Prompt Engineering**: Production-grade techniques (dual-mode prompting, comprehensiveness principle, tone matching, few-shot learning embedded in prompts). Not generic "do better" instructions.

**Context Engineering at Scale**: Systems that manage complex context strategically — layered composition, conditional loading, token accounting, persistent state as files instead of chat memory.

**Economic Optimization**: Model selection driven by cost/context tradeoffs (33% cheaper, 8x larger window). Strategic math (token-per-word, cache invalidation) to prevent runtime surprises.

**State Management**: Systems that maintain real-world state (trading positions, job opportunities, KB content) and make decisions grounded in that state, not hallucinated context.

**Behavioral Contracts (CLAUDE.md/GEMINI.md)**: Explicit, persistent system instructions that enforce domain-specific vocabulary, constraints, and workflows. AI agents that respect boundaries.

**Dual AI Portability**: Same behavioral contract across different AI tools (Claude↔Gemini), proving that agent behavior can be decoupled from implementation.

**Meta-Infrastructure**: Systems that build other systems — the /cook skill + magnum-opus workflow generates complete project scaffolds. A 9-phase decision workflow for every new AI project, capturing architecture decisions that would otherwise be made implicitly.

These patterns are replicable and applicable to other AI systems beyond these eight primary systems.

---

## Protocols and Playbooks

### Website Build Protocol

**Location:** `/Users/t-rawww/AI-Knowledgebase/future-reference/playbooks/building-professional-websites.md`

A comprehensive protocol for building professional, non-AI-slop websites. Synthesized from: Impeccable (7-domain design skill), interface-design, ui-skills, spec-kit (Spec-Driven Development), DESIGN.md (Google Stitch), Emil Kowalski, Paper.design, Pencil.dev, FontofWeb.

**Phase structure:**
```
Phase 0    → PROJECT.md — what you're building, constraints, aesthetic direction
Phase 0.5  → CLAUDE.md for the web project — persistent rules for the build
Phase 1    → DESIGN.md — typography, color, spacing, motion, component rules
Phase 2    → Tech stack decision (Astro/Next.js/Vite + animation/3D choices)
Phase 3    → spec-kit scaffolding (specify init → specify feature → /speckit.tasks → /speckit.implement)
Phase 4    → Build order: tokens → typography → nav → layout → hero → body → footer → micro-animations → scroll
Phase 4.5  → Copy strategy (copy brief, fingerprint test, section-specific rules)
Phase 4.6  → Image protocol (decision tree, optimization, AI image prompts)
Phase 5    → Quality gates (Impeccable commands, baseline-ui, accessibility, AI fingerprint checklist)
Phase 6    → Metadata + SEO
```

**What's elegant:** DESIGN.md is treated like AGENTS.md or CLAUDE.md — a plain-text contract that tells AI tools what the design intent is. The entire build is spec-driven before a single component is written.

---

## Cross-System Patterns

These patterns appear across multiple systems. Worth recognizing as a personal methodology.

| Pattern | Systems | What It Solves |
|---|---|---|
| **Context as files** | edge_lab, interview-prep, PRD system, KB | Session continuity without chat memory dependency |
| **Behavioral contract (CLAUDE.md)** | 7 of 8 systems (all except VAR Agent) | Consistent agent behavior across sessions, no re-explaining |
| **Dual AI portability** | edge_lab, interview-prep | Claude↔Gemini handoff with zero behavior drift |
| **Conditional context loading** | YouTube Summarizer, PRD system, edge_lab | Load domain context on-demand — keep context window efficient |
| **Custom language constraints** | PRD system, interview-prep | Force domain-appropriate vocabulary at the system level |
| **Structured output folders** | PRD system, edge_lab (journal/), YouTube Summarizer | Persistent artifacts from sessions, not lost to chat history |
| **Non-interactive scripts** | edge_lab, YouTube Summarizer | Inline calculation/processing during workflow, not as a separate step |
| **Git as persistence layer** | edge_lab, interview-prep, KB, Zenkai | Version-controlled state — history is automatic |
| **Distillation over aggregation** | KB | Reference-able depth vs. bookmark pile |
| **Frontier monitoring layer** | KB (emerging-architectures) | Evaluate new research without chasing benchmarks |
| **DESIGN.md as design contract** | Website builds, Zenkai | AI tools know design intent before writing code |
| **Delta sync** | Zenkai, YouTube Summarizer (cache versioning) | Cost-efficient content/cache regeneration |
| **Session close = commit** | edge_lab | Automatic versioning without manual git discipline |
| **Phase-based workflows** | PRD system, website builds | Structured execution that mirrors real-world constraints |
| **Layered README index hierarchy** | AI-Knowledgebase | Agent (or human) orients at any depth without being told what to read |
| **Dynamic index generation** | edge_lab (journal/INDEX.md) | Index always accurate — rebuilt from source, never manually maintained |
| **Mandatory-first-read index** | edge_lab (playbook/README.md) | Enforced routing — never open sub-files without reading the index first |
| **Math offloading to scripts** | edge_lab (calc.py, position-sizer.py) | Eliminates LLM arithmetic errors entirely — deterministic by design |
| **Token accounting strategy** | YouTube Summarizer | Conservative limits, explicit buffer math, prevent runtime surprises |
| **Production-grade prompt engineering** | YouTube Summarizer | Comprehensiveness principle, faithful representation, tone matching, few-shot examples embedded |
| **Residential proxy for cloud extraction** | YouTube Summarizer | Cloud IPs blocked by YouTube; datacenter proxies also blocked; residential rotating proxies are the only reliable path |
| **Background thread for blocking startup ops** | YouTube Summarizer | Proxy test at import time blocked Gunicorn cold start; moved to daemon thread — server ready immediately |
| **Rate limiter exemption for polling** | YouTube Summarizer | Global rate limits apply to all routes by default; polling endpoints must be explicitly exempted or they count against per-hour caps |
| **DB fallback for in-memory state** | YouTube Summarizer | In-memory trackers wiped on restart; DB query as fallback prevents false "not found" errors on server bounce |
| **Keep-warm via health endpoint + UptimeRobot** | YouTube Summarizer | Free tier spin-down is a user-facing latency problem; lightweight health ping every 5 min prevents idle at zero cost |
| **Mobile GPU compositing awareness** | YouTube Summarizer | CSS blur() triggers compositing layers on every animated frame; disable on mobile, keep on desktop |
| **Trusted sources constraint** | edge_lab (6 X accounts in CLAUDE.md) | Agent only pulls from named, curated accounts — not the open web |
| **Middle layer / news filter** | edge_lab (news-snapshot.md) | Raw news filtered to facts before reaching analysis layer; thesis ownership stays with human |
| **Modular service architecture** | security-var-agent | Services independently testable/replaceable; changes in one domain don't cascade |
| **Confidence scoring transparency** | security-var-agent | Quantified confidence across multiple dimensions; recommendations include explicit strength assessment |
| **Financial modeling + ROI clarity** | security-var-agent | Detailed cost modeling prevents false economies; breaks down total cost of ownership |
| **Hub document (routing layer)** | Magnum Opus, KB-INDEX | Separates where knowledge lives from what the knowledge is — update KB docs without touching the routing layer |
| **Catalog-first convention** | Magnum Opus (agent/skill/prompt catalogs) | Forces index accuracy before file creation — structurally prevents stale catalogs |
| **Phase-gated workflow as a CLI skill** | /cook, website build protocol | Workflow becomes an invocable tool — same phases every time, no steps skipped under time pressure |
| **Prohibited Patterns framing** | Magnum Opus | "Do not..." framing prevents anti-pattern priming — topic sentences that name bad behavior can surface it as a candidate |
| **SOUL.md personality contract** | Magnum Opus scaffold output | Separates agent character from structural rules — CLAUDE.md holds constraints, SOUL.md holds values |
