# Dro Marin

**AI Systems Engineer · Construction Tech**

8 years in residential and commercial construction — drywall, framing, $4.5M+ in project value. Now I build the AI infrastructure I wished existed when I was in the field.

[![200+ Automation Workflows](https://img.shields.io/badge/Automation_Workflows-200%2B-FF6D5A?style=flat-square)](#ai-agent-infrastructure)
[![5 Production AI Agents](https://img.shields.io/badge/AI_Agents-5_in_Production-8B5CF6?style=flat-square)](#ai-agent-infrastructure)
[![141K+ Contractor Profiles](https://img.shields.io/badge/Profiles-141K%2B-10B981?style=flat-square)](https://contractorsnearme.ai)
[![2 MCP Servers · 11 Tools](https://img.shields.io/badge/MCP_Servers-2_(11_tools)-F59E0B?style=flat-square)](#ai-agent-infrastructure)

---

## Blueprint Takeoff Engine

Digital takeoff system for construction blueprints — contractors upload PDFs, calibrate scale, and measure areas/lengths/counts directly in the browser.

**Spatial indexing** — R-tree (`RBush`) for O(log n) nearest-neighbor snap point queries. Search threshold adjusts dynamically with viewport zoom level so snap behavior feels consistent at any magnification.

**Computational geometry** — Shoelace formula for polygon area calculation. Ray-casting algorithm for point-in-polygon detection. Centroid computation with signed-area weighting and degenerate geometry handling (collinear vertices, self-intersecting polygons).

**Coordinate transforms** — 5-layer pipeline: viewport (screen px) → image (source px) → physical (real-world units via calibrated scale) → PDF (72 DPI, Y-axis inversion). Rotation matrix support for non-axis-aligned blueprints.

**Undo system** — Command pattern with time-based coalescing. Edits within a 100ms window merge into a single undo step. Separate redo stack. Configurable history cap. Designed for rapid-fire measurement adjustments where each drag shouldn't be its own undo entry.

**Tile rendering** — Custom `TileSource` for large blueprint images. Tiles served through authenticated edge function proxies with token injection and automatic refresh on expiry.

**Scale detection** — OCR-based scale calibration reads notation from blueprint title blocks. Manual override via two-point calibration for non-standard drawings.

50 files. Pure TypeScript. All computation runs client-side; server handles authenticated tile access and PDF processing.

---

## AI Agent Infrastructure

Five production agents orchestrated through n8n pipelines with a composable safety layer. 200+ workflows managed via a custom multi-agent orchestration framework.

### Guardrails

7 detection types, composed per-agent based on risk profile:

| Type | What It Catches |
|------|----------------|
| Jailbreak detection | Prompt injection, role-play exploits, instruction override attempts |
| NSFW filtering | Inappropriate content in generation outputs |
| PII sanitization | Names, phone numbers, addresses, SSNs — scrubbed from both input and output |
| Secret key detection | API keys, tokens, credentials leaking into responses |
| Topical alignment | Off-domain queries — agent stays in construction context |
| URL whitelist | Restricts generated links to approved domains only |
| Keyword blocking | Catches specific disallowed terms per deployment |

### Agents

| Agent | Model | Pipeline | Safety |
|-------|-------|----------|--------|
| **Estimator** | Claude Opus + Codex (vision) | Project photos → structured material takeoff → PDF estimate with labor, overhead, profit | PII in + out |
| **Receptionist** | Claude Haiku + Codex + Voice AI | Inbound call → HMAC-validated webhook → intent classification (booking / quote / inquiry / complaint) → urgency scoring → lead creation | Jailbreak + PII |
| **Project Manager** | Claude Haiku + Codex | Active jobs + weather forecast → prioritized daily crew schedule with weather-adjusted task reordering | PII output |
| **Copywriter** | Claude Sonnet + Codex | Brand-voice marketing content for contractor profiles | Jailbreak + NSFW + PII |
| **Lead Dispatch** | Rule engine | Incoming lead → trade + geo matching → contractor notification via branded email | — |

### MCP Servers

Two Model Context Protocol servers — 11 tools total:

- **Ops server** — SSE transport, JWT-authenticated. Exposes all 5 agents as callable tools. Any MCP-compatible client (Claude Code, IDE extensions, external agents) can programmatically trigger estimation, schedule generation, or lead capture.
- **Directory API** — 6 tools: contractor search, profile retrieval, review management, trade listing, lead submission. Streamable HTTP transport with sliding-window rate limiting and per-tool analytics.

OpenAPI 3.1.0 spec with `x-ai-hint` annotations for AI agent discoverability. `llms.txt` and `llms-full.txt` routes providing LLM-native documentation.

---

## Webhook Cryptography

All inbound webhooks validated at the edge before reaching application logic.

**HMAC-SHA1** — Telephony signature validation per Twilio spec. Builds signing string from URL + alphabetically sorted POST params. Computes HMAC via Web Crypto API (`crypto.subtle.importKey` → `crypto.subtle.sign`). **Constant-time XOR comparison** — iterates full signature length regardless of mismatch position to prevent timing-based extraction.

**HMAC-SHA256** — SSO token generation and verification, shared photo link signing, auth callback validation. All via `crypto.subtle` — zero dependency on Node.js `crypto` module. Tokens carry expiry timestamps with server-side enforcement.

**Input sanitization** — Hazard types restricted to `[a-z0-9\s_-]` (40 char max). Descriptions normalized and capped at 280 chars. Confidence scores clamped to `[0, 1]` at 3 decimal precision. Return-to URLs reject `//`, `://`, and non-`/`-prefixed paths to prevent open redirects.

---

## Directory Platform — 141K+ Profiles

Contractor directory serving 141K+ profiles across all 50 US states with programmatic SEO.

**ISR architecture** — On-demand incremental static regeneration. Profile pages revalidate hourly, state/city pages every 24 hours. No bulk static generation — the entire corpus is served through ISR with canonical URL enforcement and 50-state slug canonicalization in edge middleware.

**Search** — Hybrid system combining three signals:
- **Full-text**: PostgreSQL `tsvector` with GIN index. Weighted ranking — company name (A), location (B), bio text (C). Queries via `websearch_to_tsquery` with `ts_rank` scoring.
- **Geo-radius**: Bounding box pre-filter → Haversine distance calculation for precise radius matching.
- **Service area**: Polygon-based containment check against contractor-defined service boundaries.

Faceted filtering (trade, service, rating, verified) with cursor-based pagination.

**Structured data** — Type-safe JSON-LD builder generating `LocalBusiness` (with `OfferCatalog`, `GeoCircle` service areas, `aggregateRating`), `BreadcrumbList`, `FAQPage`, and `WebSite` with `SearchAction` for Google sitelinks.

**Sitemaps** — 150+ XML sitemaps in a sitemap index: per-state profile sitemaps, per-state city sitemaps, trade sitemaps, blog sitemaps. Each capped at 50K URLs per spec. CDN-cached at 6-hour TTL.

**Bio pipeline** — 16-thread parallel batch processor generating SEO bios for 141K+ profiles. LLM generates → regex quality gate strips 15+ AI filler patterns ("Here is the bio", "As an expert", `{Company}` placeholders) → SEO validator enforces business name + city in first 150 characters. Exponential backoff retry on rate limits (2s → 32s). Two-phase: generation then cleanup.

---

## Self-Healing Pipeline

Autonomous error detection → resolution → PR creation.

1. **Sentry alert** fires on unhandled exception
2. **Edge function** applies sliding-window rate limiter (10 events/min, database-backed) — prevents alert storms from triggering redundant fix attempts
3. **VPS agent** receives task. Classifies complexity → routes to appropriate model:
   - **Opus + Codex** — architectural issues, multi-file refactors
   - **Sonnet + Codex** — runtime errors, logic bugs
   - **Haiku + Codex** — type errors, lint fixes
   - Automatic fallback chain if primary model unavailable
4. **Git worktree isolation** — each fix gets its own `git worktree` and branch. Parallel fixes can't contaminate each other or main
5. **Automated PR** — agent commits fix, creates pull request via CLI, emits SSE event to monitoring dashboard
6. **Human review** — PR awaits approval. Agent never merges to main

VPS runs behind a permanent reverse-proxy tunnel with process supervision. Has its own error monitoring DSN and full tool suite for autonomous debugging.

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

**AI Drawing Analysis** — Floor plan analysis via Gemini 2.5 Flash: room detection, opening extraction, confidence scoring. Canvas measurement tools (area, linear, count) on Konva with Shoelace area calculation and scale conversion. Redline detection that identifies delta symbols and revision cloud bubbles on construction drawings.

**Programmatic Video** — Remotion compositions with spring-based scene transitions and WebGL shader backgrounds. Multi-aspect output (1:1, 16:9, 4:5) from a single composition tree.

**Real-time Dashboard** — WebSocket server ingesting IDE hook events for session lifecycle tracking. In-memory event store (10K cap) with metrics aggregation. Client reconnect with exponential backoff (1.5x multiplier, 30s ceiling).

**Construction AI Plugin** — Custom multi-agent orchestration layer with 7 domain-specific agents (estimation, code inspection, procurement, scheduling, customer service, ad optimization, content generation) + blueprint OCR via Mistral vision. Runs inside Claude Code as a first-party plugin.

---

## Stack

| Layer | What I Use |
|-------|------------|
| **Languages** | TypeScript, Python, SQL, Ruby |
| **LLMs** | Claude (Opus · Sonnet · Haiku) + Codex, GPT-4o, Gemini 2.5 Flash |
| **Orchestration** | n8n pipelines · custom multi-agent framework · 200+ workflows |
| **Frontend** | React 19, Next.js 16, Vite, Bun, TanStack Query, Tailwind |
| **Mobile** | Capacitor · 13 native plugins · IndexedDB offline-first sync |
| **Database** | PostgreSQL — 170 migrations, row-level security, realtime subscriptions, 34 edge functions |
| **Backend** | Serverless edge runtime, Rails 8, background job queues |
| **Infra** | Edge compute (serverless workers, static hosting, tunnels), process supervision, CI/CD |
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
