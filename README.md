# Dro Marin

**AI Systems Engineer · Construction Tech**

8 years in residential and commercial construction — drywall, framing, $4.5M+ in project value. Now I build the AI infrastructure I wished existed when I was in the field.

[![200+ Automation Workflows](https://img.shields.io/badge/Automation_Workflows-200%2B-FF6D5A?style=flat-square)](#ai-agent-infrastructure)
[![5 Production AI Agents](https://img.shields.io/badge/AI_Agents-5_in_Production-8B5CF6?style=flat-square)](#ai-agent-infrastructure)
[![141K+ Contractor Profiles](https://img.shields.io/badge/Profiles-141K%2B-10B981?style=flat-square)](https://contractorsnearme.ai)
[![2 MCP Servers](https://img.shields.io/badge/MCP_Servers-2_Production-F59E0B?style=flat-square)](#ai-agent-infrastructure)
[![9 Specialist Agents](https://img.shields.io/badge/Agent_Orchestration-9_Specialists-3B82F6?style=flat-square)](#multi-agent-orchestration)

---

## Blueprint Takeoff Engine

Digital takeoff system for construction blueprints — contractors upload PDFs, calibrate scale, and measure areas/lengths/counts directly in the browser.

**Spatial indexing** — R-tree (`RBush`) for O(log n) nearest-neighbor snap point queries. Search threshold adjusts dynamically with viewport zoom level so snap behavior feels consistent at any magnification.

**Computational geometry** — Shoelace formula for polygon area calculation. Ray-casting algorithm for point-in-polygon detection. Centroid computation with signed-area weighting and degenerate geometry handling (collinear vertices, self-intersecting polygons).

**Coordinate transforms** — Multi-layer pipeline converting between viewport, image, physical, and PDF coordinate spaces with rotation matrix support for non-axis-aligned blueprints.

**Undo system** — Command pattern with time-based coalescing. Edits within a 100ms window merge into a single undo step. Separate redo stack. Configurable history cap. Designed for rapid-fire measurement adjustments where each drag shouldn't be its own undo entry.

**Tile rendering** — Custom `TileSource` for large blueprint images. Tiles served through authenticated edge function proxies with token injection and automatic refresh on expiry.

**Scale detection** — OCR-based scale calibration reads notation from blueprint title blocks. Manual override via two-point calibration for non-standard drawings.

50 files. Pure TypeScript. All computation runs client-side; server handles authenticated tile access and PDF processing.

---

## Python Data Engineering

40+ production Python scripts handling data pipelines, platform migrations, batch processing, and infrastructure automation.

**Batch processing at scale** — `ThreadPoolExecutor` with configurable concurrency (2 threads local, 16 threads cloud) for parallel API calls across 141K+ records. Provider abstraction layer swaps between local inference (Ollama) and cloud LLM APIs with a single environment variable. Exponential backoff retry on rate limits with configurable ceiling.

**Data migration pipelines** — Three-mode execution pattern across all migration scripts: `--audit` (read-only inventory, zero API calls), `--dry-run` (fetch and diff without writes), `--execute` (apply changes with backup). Used for migrating 200+ automation workflows between platforms, rewriting URL mappings, and rebinding credential references across production systems.

**Quality gate pipelines** — Multi-pass data integrity system: LLM generation → regex-based filler detection (15+ patterns) → domain-specific validation (SEO rules, character limits, placeholder detection) → integrity verification pass → spot-check sampling. Each phase is independently re-runnable against the full dataset.

**ETL and data sync** — Import/export pipelines for leads, service areas, portfolio photos, trade taxonomies, and content posts. Schema analysis scripts that diff source and target data models before migration. Deduplication analysis with configurable match thresholds.

**Infrastructure automation** — Multi-zone DNS record management across 24 Cloudflare zones (DKIM, SPF, MX provisioning). Execution monitoring scripts that paginate workflow run histories, aggregate success/failure rates, and flag anomalies. Notion workspace utilities for orphan detection, quality auditing, and bulk cleanup.

All scripts use standard library (`urllib`, `json`, `argparse`, `concurrent.futures`) with minimal external dependencies. Virtual environments per project, git-ignored.

---

## Multi-Agent Orchestration

I built a system the way a Renaissance workshop operates: named specialists, each with a defined role, assembled by a master coordinator who knows which hand to call for which stroke.

Nine named agents form the core. **Sisyphus** carries the long plans — multi-step implementation sessions that persist across interruptions, tracking progress through structured markdown checklists, rolling the boulder forward until the work is done. **Oracle** handles deep reasoning — architecture tradeoffs, security analysis, the problems that require sitting with the question before answering. **Metis** reviews plans before execution begins, finding the gaps and unstated assumptions the builder would discover only after the foundation was poured. **Momus** is the critic — examines finished work against rigorous standards, catches what the craftsman's pride would rather ignore. **Explore** and **Librarian** are the research arms: one searches the internal codebase for patterns and conventions, the other reaches outward to documentation, open-source implementations, and external references. Both run as background agents, fired in parallel, returning results asynchronously while the primary agent continues non-overlapping work. **Atlas** bears infrastructure. **Prometheus** extends capabilities into unfamiliar territory.

**Task categories route work to the right instrument.** `visual-engineering` sends frontend and design work to a model optimized for UI craft. `ultrabrain` handles genuinely hard logic — architecture, algorithms, complex debugging. `deep` runs autonomous research-then-implement cycles for problems requiring thorough investigation before the first line of code. `artistry` takes unconventional approaches to problems that resist standard patterns. `quick` handles the single-file changes where speed matters more than deliberation.

**A skill system composes capability on demand.** 80+ modular skills — each a self-contained instruction set with mode detection, scope guards, and anti-patterns. Skills load into any agent at delegation time. A frontend task might load `frontend-ui-ux` and `tailwind-css-patterns`. A deployment loads `ship-it` — a deterministic release pipeline with security gates, visual verification, and automatic rollback on failure. Skills are additive: the agent's base reasoning plus domain-specific craft knowledge, composed at the moment of need.

**A 15-persona strategic council handles decisions that require more than one perspective.** Each persona carries a fixed decision-making framework and known blind spots with explicit deference rules — the inventor defers to the operator on shipping discipline, the operator defers to the systems thinker on architecture, the systems thinker defers to the field practitioner on whether a contractor would actually use the thing. Two voice modes: some personas speak as internal monologue, others speak directly as advisors. The council convenes on strategic questions, debates with the human in the loop driving each exchange, and converges to a directive. No simulated consensus — every round requires human engagement.

**Background execution and session continuity** make the system feel less like calling functions and more like directing a team. Agents fire in parallel with `run_in_background=true`, return task IDs, and send completion notifications. The primary agent continues with non-overlapping work or ends its response and waits. Every agent session carries a persistent ID — follow-up work resumes with full context preserved, no repeated file reads, no re-exploration. Plans persist as structured markdown with checkbox tasks, session summaries, and completion states. A runtime health monitor watches for plugin mismatches and self-heals.

The whole system is open-source, runs locally, composes with any MCP-compatible client, and costs nothing beyond the model API calls.

---

## AI Agent Infrastructure

Five production agents orchestrated through n8n pipelines with a composable safety layer. 200+ workflows managed through the orchestration system described above.

### Guardrails

Composable safety layer with 7 detection types — including prompt injection defense, PII sanitization (input and output), content filtering, secret leak prevention, and domain-scoped topical alignment. Each agent gets a risk-appropriate subset composed at deployment time.

### Agents

| Agent | What It Does |
|-------|-------------|
| **Estimator** | Project photos → structured material takeoff → PDF estimate with labor, overhead, profit |
| **Receptionist** | Inbound call → intent classification → urgency scoring → lead creation |
| **Project Manager** | Active jobs + weather forecast → prioritized daily crew schedule |
| **Copywriter** | Brand-voice marketing content for contractor profiles |
| **Lead Dispatch** | Incoming lead → trade + geo matching → contractor notification |

### MCP Servers

Two production Model Context Protocol servers that let any MCP-compatible client (Claude Desktop, IDE extensions, external AI agents) interact with the full platform through natural language.

Any MCP-compatible client can search the contractor directory, trigger AI agents, manage content, and route leads through natural language. The full platform surface is accessible conversationally.

Authenticated transports with rate limiting, per-operation analytics, and LLM-native documentation routes for AI agent discoverability.

---

## Webhook Cryptography

All inbound webhooks validated at the edge before reaching application logic.

**HMAC validation** — SHA-1 and SHA-256 signature verification for telephony webhooks, SSO tokens, signed links, and auth callbacks. All via Web Crypto API with timing-safe comparison. Zero dependency on Node.js `crypto` module.

**Input sanitization** — Strict allowlists on all user-facing input fields. Character class restrictions, length caps, numeric clamping, and URL path validation to prevent open redirects.

---

## Directory Platform — 141K+ Profiles

Contractor directory serving 141K+ profiles across all 50 US states with programmatic SEO.

**ISR architecture** — On-demand incremental static regeneration. Profile pages revalidate hourly, state/city pages every 24 hours. No bulk static generation — the entire corpus is served through ISR with canonical URL enforcement and 50-state slug canonicalization in edge middleware.

**Search** — Hybrid system combining full-text ranking, geo-radius proximity, and service area containment into a single query pipeline. Faceted filtering (trade, service, rating, verified) with cursor-based pagination.

**Structured data** — Type-safe JSON-LD builder generating `LocalBusiness` (with `OfferCatalog`, `GeoCircle` service areas, `aggregateRating`), `BreadcrumbList`, `FAQPage`, and `WebSite` with `SearchAction` for Google sitelinks.

**Sitemaps** — 150+ XML sitemaps in a sitemap index: per-state profile sitemaps, per-state city sitemaps, trade sitemaps, blog sitemaps. Each capped at 50K URLs per spec. CDN-cached at 6-hour TTL.

**Bio pipeline** — Multi-threaded batch processor generating SEO-optimized bios for 141K+ profiles. Multi-pass quality gates strip AI filler, enforce SEO constraints, and validate domain-specific rules before any bio goes live. Rate-limit-aware with automatic retry and backoff.

---

## Self-Healing Pipeline

Autonomous error detection → resolution → PR creation. Production errors trigger an AI agent that classifies severity, selects the appropriate model tier, isolates the fix in its own branch, and opens a pull request for human review. Storm suppression prevents redundant fix attempts during incident spikes. Parallel fixes run in isolation so they never contaminate each other or main. The agent never merges — every fix awaits human approval.

Runs on a dedicated VPS with process supervision, its own error monitoring, and a full autonomous debugging toolkit.

---

## Offline-First Mobile App

Construction site companion — photo documentation, daily logs, safety checklists. Built for job sites with unreliable connectivity.

- **13 native plugins** via Capacitor — camera with haptic feedback, geolocation, push notifications, network state detection
- **IndexedDB queue** — 6 typed object stores (pending photos, checklist updates, activity logs, cached jobs, cached checklists, sync status). Versioned schema migrations.
- **Sync service** — Auto-syncs pending queue on reconnect. Exponential backoff retry (3 attempts). 30-second background polling when online.
- **EXIF GPS extraction** — Reads geolocation from mobile photo metadata before upload for automatic location tagging on the job map
- **PDF reports** — Server-side generation with embedded images, checklists, and pagination
- **Dual push** — Web Push (VAPID) for browser + FCM for native mobile

Rails 8 backend with Solid Queue for background job processing (webhook delivery, report generation, notification dispatch).

---

## More Projects

**AI Drawing Analysis** — Floor plan analysis via Gemini: room detection, opening extraction, confidence scoring. Canvas measurement tools (area, linear, count) on Konva with Shoelace area calculation and scale conversion. Redline detection that identifies delta symbols and revision cloud bubbles on construction drawings.

**Programmatic Video** — Remotion compositions with spring-based scene transitions and WebGL shader backgrounds. Multi-aspect output (1:1, 16:9, 4:5) from a single composition tree.

**Real-time Dashboard** — WebSocket server ingesting IDE hook events for session lifecycle tracking. In-memory event store (10K cap) with metrics aggregation. Client reconnect with exponential backoff (1.5x multiplier, 30s ceiling).

**Agent Command Dashboard** — The craftsman who works blind works twice. This dashboard was built so I never have to. A self-healing operations center that monitors every running agent, every model's health, and every error that enters the system. When a production error arrives, the dashboard doesn't just display it. It classifies severity, dispatches an autonomous agent to investigate and fix, tracks the heal rate over time, and streams the entire execution log in real-time via SSE. I watch agents think. Three-panel layout: active sessions on the left, live execution trace in the center, PR context on the right. One-click merge when the fix looks correct. An MCP server health grid shows which tool servers are responding and which have gone silent. AI model health monitors track provider availability across multiple vendors. A tools inventory catalogs every capability by type: triggers, actions, utilities, controls. Hours-saved metrics aggregate from workflow execution data. Voice integration lets me issue commands by speaking. No database. The entire surface is stateless — all data flows from the VPS agent API in real-time. React 19, Vite, TanStack Query for cache coordination, Framer Motion for state transitions. Built because the alternative was switching between six terminal windows and hoping I noticed the failure before the user did.

**macOS Menu Bar App** — A single-binary Swift app compiled with `swiftc`, no package manager, no Xcode project, zero external dependencies. It lives in the menu bar and answers one question all day long: how much have I burned and how much remains. Seven AI providers tracked simultaneously — token consumption by model, estimated USD cost with built-in price tables, daily and all-time totals, 7-day trend charts. Real-time quota monitoring pulls remaining allowances from each provider, tracks reset countdowns, and calculates pace — whether I'm running ahead of or behind my expected daily burn rate. Model health indicators show which providers are healthy, degraded, or broken. An agent configuration view displays which specialist agents are assigned to which models, what MCP servers are connected, and what task categories are active. Global hotkey summons the panel instantly. File watchers auto-refresh when configuration files change on disk. I built it because I was checking three different provider dashboards manually, doing mental arithmetic on remaining quotas, and losing 15 minutes every time I needed to decide whether to route a heavy task to a provider running low. Now I glance at the menu bar.

---

## Stack

| Layer | What I Use |
|-------|------------|
| **Languages** | TypeScript, Python, SQL, Ruby, Swift |
| **Python** | `concurrent.futures`, `argparse`, `requests`, `tqdm`, `openai` SDK, provider abstraction (Ollama / OpenRouter / GPT), data pipelines, batch ETL |
| **LLMs** | Claude (Opus · Sonnet · Haiku) + Codex, GPT, Gemini, MiniMax, GLM, Kimi |
| **Orchestration** | 9 named agents · 5 task categories · 80+ composable skills · 15-persona strategic council · n8n pipelines · 200+ workflows |
| **Frontend** | React 19, Next.js 16, Vite, Bun, TanStack Query, Tailwind |
| **Mobile** | Capacitor · 13 native plugins · IndexedDB offline-first sync |
| **Database** | PostgreSQL — 170 migrations, row-level security, realtime subscriptions, 34 edge functions |
| **Backend** | Serverless edge runtime, Rails 8, background job queues |
| **Infra** | Edge compute (serverless workers, 24 DNS zones, tunnels), process supervision, CI/CD |
| **AI Safety** | 7-type composable guardrails · HMAC webhook crypto · PII sanitization pipelines |
| **Voice** | Conversational AI + telephony integration (multi-tenant, per-contractor) |
| **Protocols** | MCP (SSE + JWT), OpenAPI 3.1, `llms.txt`, RSS 2.0 |
| **Monitoring** | Error tracking with auto-heal pipeline · product analytics |

---

## Background

8 years in residential and commercial construction. Managed drywall and framing projects — K-12 schools, hospitals, multi-family housing, custom homes, and a 6-year mixed-use landmark development. $4.5M+ in total project value.

Built my first production tool to automate bid estimation — cut prep time by 60%. That turned into a full-time pivot into AI engineering for the industry I came from.

AWS Solutions Architect + Developer Associate certified.

---

<p align="center">
  <a href="https://buildx.pro">buildx.pro</a> · <a href="https://contractorsnearme.ai">contractorsnearme.ai</a> · <a href="https://mycontractor.ai">mycontractor.ai</a> · <a href="mailto:alexandro@mybuildx.com">alexandro@mybuildx.com</a>
</p>
