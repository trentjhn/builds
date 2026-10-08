# Builds

I'm Trenton (TJ) Johnson: Yale CS, nearly two years as a technical PM at PayPal, now co-founder of SignalWorks, an AI consultancy that deploys into the customer's environment and stays through adoption. I build with Claude Code every day, and most of what I build is agent infrastructure: giving agents the right context, hard limits, and an honest definition of done.

Public repos are linked. Most of my best work is private (client work, active ventures, personal infrastructure), so those entries are marked and described here instead.

---

## Agent infrastructure

### the-lobby
**Private** · TypeScript, Node, SQLite

A task queue that runs headless Claude Code agents against real open work across my repos, with a local web UI where I veto tasks, answer agents' questions, and review results. 126 commits between Aug 8 and Aug 16, 2026.

- **One writer.** A daemon is the only process that writes the SQLite queue. The scanner, the agent runs, and the UI send it intents. On restart it adopts runs left orphaned and kills provable crash survivors.
- **Trust lives in credentials, not prompts.** Each agent works in its own git worktree whose remote is a local mirror, so it has nothing to push with. Only the daemon holds GitHub credentials, and it pushes only when the repo's trust tier allows (11 repos, 4 tiers from autonomous push to propose only). Text scanned from repos fills a fixed task template and never chooses the actions.
- **Budgets are enforced, not reported.** 150 turns and 45 minutes per task by default, one active task per repo. The turn default started at 50 and was raised after the first real run showed 50 starved real work. Over budget: graceful stop, a work-in-progress commit, then a hard kill, with the worktree kept for review.
- **Done means three outside signals agree.** The run reached a terminal state, the deliverable exists, and the task's own check passes. Never the exit code, never the agent's word, and no LLM judge anywhere in that path.

### My Claude Code harness: a hand-built context layer for one person
**Private** · Markdown skills, Python hooks

Everything I know about how I work, what I decided, and what happened in past sessions, kept in files and fed back to the agent at the right moment.

- **64 skills.** Reusable workflows the agent loads by trigger phrase: premortem, fresh-context review, compliance check, deploy gate, a voice guide for anything a person will read.
- **10 hooks across 5 lifecycle events.** One dispatcher fans each prompt out to six checks in a single process. A guard blocks irreversible shell commands on unattended runs. A stop hook writes build state to disk on each new commit, so the next session starts where this one ended.
- **Memory that surfaces itself.** 274 memory notes across 76 projects. A per-prompt hook matches the prompt against them (a word index plus local embeddings) and injects the relevant note unasked. A recall skill searches every past session transcript and returns the sessions, dates, and snippets where a topic came up.
- **/cook and /serve.** `/cook` scaffolds a project from my knowledge base: spec, design decisions, threat model, agent roles, step plan. `/serve` executes that plan autonomously, with review depth set by each step's risk, a pause list for irreversible actions (force push, production deploy, sending email), and hard caps on wall time and iterations. the-lobby, GridVision, and Hermes all started from `/cook`.

### Field deployment: an agent platform for state government
**Client work, no public code** · Aug 31 to Sep 4, 2026

A five-day contract deployment at an AI agent platform for state government. I worked in the field with agency customers and in the product codebase.

- **Found the skills layer was close to undiscoverable.** 52% of a 256-skill library had never been invoked in 90 days. The likely cause, which I handed forward as a hypothesis: catalog skills failed to resolve in the loader while personal skills loaded fine, so users got "unknown skill" and stopped trying.
- **Took it from data to merged code.** Six tickets, then 24 pull requests, 16 merged that week: the loader fix, a daily resolver health check that monitors catalog and loader parity, and a repair detector that flags failing skills for review behind a kill switch.
- **Corrected the adoption metric.** Machine-generated sessions were counted as human usage. Filtering them out moved the one-turn rate from 57.6% to 42.7%, so every number built on the old figure had understated real engagement.

---

## Client systems (SignalWorks)

### Employee handbook compliance system
**Client work, private** · JavaScript, Cloudflare Workers, D1 (SQLite)

For an HR consultancy that maintains employee handbooks for its client companies (79 on the live roster). It watches employment law, matches each handbook section against the firm's approved policy library, flags where a handbook has drifted, and drafts a redline that cites the law that changed. New language comes from the firm's approved library, never from the model's reading of the law.

- **Live today.** An alert pipeline polls 24 free law sources (22 enabled) across California, city, and federal law, and a dashboard sits behind Cloudflare Access with default-deny roles. A comparison engine makes one pinned model call per section to separate material from cosmetic differences. Every call is journaled, so a replay rebuilds the same rows with zero provider calls. 687 tests, no network calls.
- **In build: the review and release workflow.** Phase 4 is 23 of 103 plan steps complete as of Oct 8, 2026. The schema adds document versions and sections, review items, approvals that only a named principal can sign, releases, and a compliance event log. States only move forward. History tables refuse updates and deletes at the database layer, and the only delete path is a per-client deletion that leaves a certificate. Triggers refuse links between one client's records and another's.
- **Verification is mechanical.** Every step gets a fresh-context review, and high-risk steps get two: one against the spec, one on side effects and failure paths. A mutation gate plants faults to prove the tests catch them. Tests also pin platform limits the local engine doesn't enforce, such as D1 rejecting LIKE patterns over 50 bytes.

### Commercial mortgage refinance leads
**Client work, private** · Python, SEC loan data, Claude Code agents

For a commercial mortgage broker: find California commercial loans maturing in 2027 that are likely refinance candidates, then find a way to reach each owner.

- **Screening from SEC filings.** Securitized commercial loans report loan-level data to the SEC every month, including maturity, payment, and property income. The first pass read 311 bond trusts and 23,022 loans. The current list is 85 loans in the broker's deal band maturing 6 to 12 months out, and 30 of them can't refinance their full balance at 7.5% on reported income.
- **Owners from the filings, private loans from county records.** Guarantor and sponsor fields name the owner on most securitized loans. Private and bridge loans never reach the SEC, so they come from county recorder deeds and indexes: 9 more in band and due in the window, from free sources only. Agents search only indexes open without a click; accepting a site's terms is left to a person.
- **A contact counts only after it's tested.** Each email goes through two delivery-test services, needs a source tying it to the property, and is checked against people who have left. A ledger keeps every result, so no owner is researched or tested twice. The owner is reachable on 66 of the 85 so far (33 by email, 33 by phone).
- **Every number is recomputed.** Rebuilding from the September filings caught a field-selection bug that understated payments on 28 loans whose interest-only period had ended. A fresh-context pass rechecked every figure before the client saw it.

---

### Parking lead generation
**Client work, private** · Python CLI · delivered to a marketing client

Turns a parking-garage address into a ranked, contact-enriched list of nearby businesses likely to advertise there.

- **Staged and replayable.** Google Places discovery, then website scraping and Hunter.io enrichment, then scoring and export. Every stage saves its output, so any stage can be rerun on its own.
- **Safe at the edges.** A spend budget flags rows it couldn't enrich instead of silently dropping them, and the CSV export guards against spreadsheet formula injection.

## Flagship builds

### GridVision
**Private (active venture)** · Python, PyTorch, RF-DETR

Computer vision that turns raw football film into a charted play: backfield, personnel, receiver distribution, and strength at the snap. A coach corrects the output with a few taps, and the corrections become training data.

- **Assisted first, because the data says so.** Detection works off the shelf. Separating players from the sideline crowd does not. With a person drawing boundary lines and tapping the QB, player admission scores 0.95 to 0.98 precision and 0.88 to 0.93 recall on two film sources, against a bar of 0.70 and 0.85. The fully automatic path still fails on sideline crowd, and that is the open problem.
- **Confident and wrong is the fatal error.** Every chart read is scored by a pre-registered criterion against 47 hand-charted reps (188 cells across 4 videos). The system may abstain or flag a likely miss, but confident-wrong cells must stay at zero, and they do.
- **A number decides promotion.** A TrackEval gate (HOTA against the current champion) promotes a model only on strict improvement, with separate exit codes for regression, missing ground truth, and infrastructure failure. The tracking and fine-tune stages it guards are designed but not yet the active layer. 678 tests.
- **Coach-labeled film is next.** A junior college staff handed me 13 games of archive film, cut into one clip per play in two angles, with their own per-play breakdown sheets (formation, front, blitz, coverage). Their tags become the ground truth the chart read is scored against.

### Hermes
**Private (personal infrastructure)** · Python

A framework-free agent stack I wrote to understand tool use from primitives. It runs as two read-only Telegram bots on a small server: one drafts posts from my notes, one challenges my thinking with live research.

- **Client-side tool-use loop** driven by the model's stop reason, with a spend guard on three axes: per call, per run, and per day.
- **SSRF gate** that rejects private addresses even when an IPv4 address hides inside an IPv6 one (mapped, 6to4, IPv4-compatible, NAT64).
- **Injection defenses as code.** Untrusted content gets taint tracking and a fence with a one-time random nonce, so injected text cannot forge the closing tag and pass as instructions.
- 292 tests. A companion plugin, `hermes-operator`, retrofits the same safety floor onto a third-party agent framework.

---

## Edge Lab
**Private** · Python, Claude Code

My personal trading analyst, used daily. Claude reads my written framework, current context, and journal, then stress-tests an idea against my own rules instead of agreeing with it. 715 commits since March 2026, 395 tests.

- **Numbers never come from the model.** All math runs in tested scripts. Price levels are computed from two independent data sources and labeled agree, single-source, or disagree. A disagreeing level is never quoted.
- **Every statistic carries its evidence.** A deterministic analysis script returns each measured figure with its sample size, effective sample size, and confidence interval. When the evidence is too thin, the value is absent rather than shown with a warning.
- **Gates run before any analysis.** Three checks: the written plan matches the broker (read-only connection), every open position has a written plan, and the append-only journal matches reality. A gate that cannot judge exits differently from one that finds problems.
- **Predictions are scored, not remembered.** The agent drafts each call and I ratify it, with a mechanical condition that settles it. Calls are scored separately on level, calibration, timing, and direction, never averaged into one hit rate.

---

## Shipped

### PQC Deal Engine
**Live:** https://pqc-deal-engine.vercel.app/ · **Repo:** [`pqc-deal-engine`](https://github.com/trentjhn/pqc-deal-engine) · Next.js, TypeScript

Point it at a company and it generates a board-ready post-quantum cryptography readout: "harvest now, decrypt later" risk, the regulatory deadlines its industry faces, a migration sketch, and a CISO one-pager.

- **Facts can't be invented.** Every regulatory date and algorithm name comes from a versioned data file. A grounding gate rejects any year or standard in the output that isn't in that file, and fails closed.
- **The risk math is code.** The Mosca score is a pure, tested function. The model only writes prose around a verdict that was already computed.
- 27 tests, rate-limited generation, server-side key. Built and deployed in about a day.

### Government Relations Intelligence Dashboard
**Live:** [gov.signalworks.live](https://gov.signalworks.live) · private (client work) · Python, GitHub Actions

A governance briefing for a public-affairs operator. Eleven public government sources are scraped four times a day on weekdays, diffed against the last run, summarized through a hallucination gate, and published as a static dashboard.

- **Two-layer hallucination gate.** Displayed text must be a verbatim span from the source, and every named entity in the headline must appear in that span. The model cannot author a bill number or a dollar figure.
- **State lives in a separate repo from the pipeline**, so the published brief never reveals who is being monitored, and a rollback is a `git revert`.
- **Three source states** (active, quiet, degraded) tell "nothing happened today" apart from "a scraper broke."

### MLB All-Star Aggregator
**Repo:** [`blitz-mlb-allstar`](https://github.com/trentjhn/blitz-mlb-allstar) · Python, static site

Scrapes Baseball Reference, builds a validated dataset of the 2024 to 2026 MLB All-Stars (260 rows for 177 players), joins it to a video game's top-100 player ratings, and serves it as a local site.

- **The build refuses to write bad data.** Unique keys, 30 teams per season, allowed values, and a reconciliation of every season against the full All-Star rosters. Any failure means nothing is written. A name that fits two players stops the build rather than guess.
- **Polite, reproducible scraping.** At most one request every 4 seconds, a hard stop and a block marker on a 403 or 429, and all 274 fetched pages cached with SHA-256 checks, so a fresh clone builds with no network.
- **Disagreements are documented, not smoothed over.** Where the reference output and the source pages differ, the data follows the pages and the README lists each case. 211 test functions run offline against the cached pages.

### AI Search Visibility Tracker
**In use on SignalWorks engagements** · private · Next.js, Postgres

Measures how a brand shows up inside AI answer engines (ChatGPT, Claude, Perplexity, Gemini, Google AI Overviews) by running each prompt many times instead of trusting one answer. Results carry Wilson 95% confidence intervals, which stay correct near 0% and 100% on small samples, and extraction is decoupled from querying so a prompt change doesn't re-pay for engine calls.

### Viridian
**Repo:** [`viridian`](https://github.com/trentjhn/viridian) · Go, single binary

A terminal UI that watches Claude Code sessions in real time (tool calls, token spend, session memory, file diffs) through the harness's hook system. Every hook is an append-only logger that always exits 0, because a failing pre-tool hook would block every tool call in the host.

### GitRecap
**Live (access-gated):** https://gitrecap-gamma.vercel.app · **Repo:** [`gitrecap`](https://github.com/trentjhn/gitrecap) · TypeScript

A phone-first app that rebuilds what you did each day from your GitHub commits and writes it up as a readable narrative. Past days are cached once and never recomputed, so it runs at almost no API cost.

### YouTube Summarizer Premium
**Private** · React + Vite, Python + Flask

Paste a YouTube link and get a structured breakdown of the video instead of watching all of it.

- **Two depths.** Quick Read gives five focused sections on the core message; Deep Analysis gives an eight-section breakdown with exact quotes, arguments, and claims.
- **Every claim links to its timestamp,** so you can jump to the moment in the video that backs it.
- Gemini with an OpenAI fallback, a Redis cache, and live progress streamed to the browser over WebSockets.

**Smaller public tools:** [`quantum-arxiv-digest`](https://github.com/trentjhn/quantum-arxiv-digest) (pulls and ranks quantum and post-quantum papers, runs with no API key), [`configkit`](https://github.com/trentjhn/configkit), [`promptarena`](https://github.com/trentjhn/promptarena), [`zenkai`](https://github.com/trentjhn/zenkai).

---

## How I build

Most of this runs on **AI-Knowledgebase** (private): a practitioner reference distilled from 100+ sources across prompting, context engineering, agent systems, evaluation, and security. `/cook` reads it to scaffold every new project. A few patterns recur on purpose:

- **Context as files.** Behavior, state, and decisions live on disk, so no session depends on a conversation staying alive.
- **Grounding gates.** The model never authors a load-bearing fact (a date, a dollar figure, a bill number). Facts are injected and checked.
- **Evals decide.** A model or feature ships only when a measured gate says it beat what it replaces.
- **Exact math in code.** Risk scores, statistics, and sizing run in tested functions, never in the model.
- **Trust in credentials, not instructions.** What an agent cannot do is enforced by what it holds, not by what it was told.
