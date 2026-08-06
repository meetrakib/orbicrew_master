# Cost & Pricing Analysis

Part of the AI Employee Office Platform documentation set. See `00_INDEX.md` for the full document list.

---

## 1. Purpose of this document

You asked three concrete questions:

1. What will maintenance cost at the very beginning, when you're the only user?
2. How much should you charge customers, given competitors like HyperAgent (Airtable's product) and others charge anywhere from ~$20 to $500+/mo, sometimes "$5000 or more"?
3. How do you make sure the system is a genuine **cost saver with better output at low usage** — not just another credit-metered black box?

This document answers all three with real numbers pulled from current provider pricing (verified via web search, see citations), not guesses.

---

## 2. Competitor pricing reality (what "$20 to $5000" actually looks like)

You mentioned a platform charging $20–$5000+. Research turned up a naming collision worth clearing up first: there is a crypto trading bot called "HyperAgent" (Hyperliquid-focused, CHF 49/149/499 tiers) which is unrelated to general AI agent platforms, and there's also Hyperagent by Airtable (built by Howie Liu), which is a persistent AI-agent-as-employee platform, priced per-task rather than in fixed tiers — <cite index="21-1">their homepage shows real output cards with cost and duration metadata printed inline, like "23 minutes, $8.82."</cite> Neither publishes a clean $20–$5000 ladder, so the number you saw was likely from a different competitor or a blended reading of several. Either way, here's the real competitive landscape as of mid-2026:

| Platform | Model | Entry price | Top self-serve tier | Notes |
|---|---|---|---|---|
| Lindy AI | Subscription + opaque credit quota | $49.99/mo (Plus) | $199.99/mo (Max) | <cite index="34-1">Billing is monthly only, no annual discount, and the old free plan is gone, replaced by a 7-day trial; "standard usage" is a quota Lindy no longer publishes a number for, and your assistant simply pauses when you hit it.</cite> Enterprise is custom above that. |
| Manus AI | Credit-based | $20/mo (Pro) | $200/mo (Extended Monthly Usage) | <cite index="30-1">All plans use a credit-based system, and usage scales quickly depending on task complexity, with a free plan giving 1,000 starter credits plus 300 refreshed daily.</cite> |
| Hyperagent (Airtable) | Pure pay-per-task, no published tiers | Task-priced (~single dollars to double digits per task shown in examples) | N/A — usage-based | Early-access positioning, no fixed subscription ladder found. |
| HyperAgent (trading bot, unrelated) | Subscription | CHF 49/mo | CHF 499/mo (Enterprise) | Different product category entirely (crypto trading), included here only to avoid confusion. |

**The pattern across all of them:** entry tier ~$20–50/mo, mid-tier ~$100–200/mo, and anything past that is either Enterprise-custom or so usage-dependent it's effectively uncapped. The complaints in every review are the same: **opaque credit consumption**, "model tax" surcharges for using better models, and bills that don't match expectations. <cite index="32-1">One breakdown found a single lead-gen automation — search, email, follow-up call — consuming 275 credits in one run, enough to blow through an entire monthly Pro allotment in about 18 leads.</cite>

This is your opening. Competitors compete on *feature breadth*; almost none of them compete on **cost transparency and cost efficiency**. That's your wedge, and it's the one you already told me matters most to you.

---

## 3. Initial maintenance cost — you as the only user (Phase 0)

Assumption: you're running this for your own agency work, no paying customers yet, moderate daily usage (research, writing, some coding-agent tasks, a handful of overnight jobs per week).

### 3.1 Infrastructure — genuinely minimal at this stage

| Item | Provider / option | Cost | Notes |
|---|---|---|---|
| App server (orchestrator + web UI) | Hetzner CX22 (2 vCPU / 4GB) | <cite index="39-1">~$4.59/mo</cite> | <cite index="41-1">Hourly billing rounds up with a monthly price cap — no minimum contract.</cite> Enough for Phase 0 traffic. |
| Postgres + pgvector | Self-hosted on the same Hetzner box (or a second small instance once it needs to scale independently) | $0/mo beyond the VPS cost above | No managed-database vendor — avoids vendor lock-in per your explicit requirement. A CX22 comfortably runs Postgres + the app for Phase 0 volume; see `08_devops_and_deployment.md` Section 9 for the self-hosted setup, backup tooling, and when to split Postgres onto its own instance. |
| Auth | Self-hosted (e.g., a lightweight open-source auth library/service — see `08_devops_and_deployment.md`) | $0/mo | No managed auth vendor; standard JWT session auth is not complex enough to justify one at this stage |
| Redis (task queue) | Self-hosted on the same Hetzner box | $0/mo | At solo-user volume, co-locating Redis on the app VPS is fine; no need for a managed Redis service yet |
| Object storage (generated files) | Self-hosted MinIO (S3-compatible, open source) on the same box initially, or Cloudflare R2 if you'd rather not manage storage yourself | $0/mo (MinIO) or free tier (R2) | MinIO keeps you fully self-hosted; R2 is a pragmatic non-lock-in alternative since it's S3-API-compatible and easy to migrate away from — your call based on how much ops you want to own |
| Domain + DNS | Any registrar + Cloudflare | ~$1–2/mo amortized | One-time annual cost spread monthly |
| **Infra subtotal** | | **~$6–8/mo** | Essentially just the VPS cost, since Postgres/Redis/auth/storage are all self-hosted on it at this stage |

**Note on the vendor-lock-in decision:** the original version of this document suggested Supabase for managed Postgres convenience. Given your explicit preference to avoid vendor lock-in, the recommended path is now **self-hosted Postgres + pgvector from day one** — see `08_devops_and_deployment.md` Section 9 for the concrete setup (backup tooling, migration path to a dedicated DB instance as you scale). The cost difference is negligible at this stage (a few dollars either way); the real tradeoff is a small amount of extra ops work (you manage backups yourself) in exchange for zero platform dependency — worth it given how strongly you've stated this preference.

### 3.2 Model/LLM costs — the variable that actually matters


This is where your spend will really live, and it's usage-driven, not fixed. With the tiered routing approach from the main requirements doc (cheap classifier model triages everything, only genuinely hard tasks reach frontier models, prompt caching on repeated context, batch API for overnight non-realtime jobs):

| Usage pattern | Estimated monthly LLM spend |
|---|---|
| Light personal use (a few tasks/day, mostly routed to cheap/mid-tier models, occasional frontier-model call for hard reasoning) | **$15–40/mo** |
| Moderate personal use (daily research + writing + some coding-agent work + a few overnight batch jobs/week) | **$40–100/mo** |
| Heavy personal use (frontier-tier model used often, large context windows, many overnight autonomous runs) | **$100–250/mo** |

Without routing/caching discipline, the same workload on frontier models alone could easily run **3–8x higher** — this is exactly why the router/budget-guard components in the main requirements doc are not optional, they're the difference between these numbers and a runaway bill.

### 3.3 Voice/avatar costs (only if you're using the office-avatar UI regularly)

| Component | Cost basis | Estimate at light personal use |
|---|---|---|
| STT (Whisper self-hosted on your existing VPS, or Deepgram pay-per-use) | Free if self-hosted; ~$0.0043–0.0125/min if hosted API | $0–10/mo |
| TTS (ElevenLabs or Google Cloud TTS) | Per-character/per-request | $5–20/mo at light daily use |
| Avatar rendering (Live2D sprite-based) | One-time asset cost, no per-minute fee | $0/mo recurring |

Skip Simli/Tavus (per-minute talking-head APIs) at this stage — they add real recurring cost for a feature that's nice-to-have, not core, until you're using the avatar UI heavily or showing it to prospects.

### 3.4 Total realistic Phase 0 monthly cost

| Scenario | Total/month |
|---|---|
| **Bare minimum** (free-tier infra, light LLM use, self-hosted voice) | **~$20–30/mo** |
| **Comfortable daily-driver** (small paid VPS, moderate LLM use, hosted TTS) | **~$60–120/mo** |
| **Heavy personal use with avatar UI active daily** | **~$150–300/mo** |

For context: this is already cheaper than a single competitor Pro-tier seat ($50–200/mo) — and you get the whole platform, not a seat.

---

## 4. What changes when you get customers (Phase 2 scaling)

You don't need to rearchitect anything to scale — the same components scale up:

- **Hetzner CX22 → CPX/CCX tiers** as concurrent load grows (still cheap relative to AWS/GCP equivalents).
- **Postgres moves to its own dedicated Hetzner instance** (separate from the app server) once concurrent load justifies isolating database I/O — still self-hosted, just split onto its own box, with `pgbackrest` or `WAL-G` handling continuous backup to object storage (see `08_devops_and_deployment.md` Section 9). Realistically an extra ~$10-20/mo for a dedicated CPX-tier DB instance at this stage, not a jump to a managed-database bill.
- **Redis**: move to a dedicated small VPS once multiple tenants depend on queue uptime — still self-hosted, avoiding a managed Redis vendor.
- **LLM cost becomes per-tenant metered**, not a personal expense — this is where the router/budget-guard earns its keep as your gross margin protector, not just a personal cost saver.

Rough infra cost at ~20-50 paying customers, moderate usage each: **$150–400/mo infra**, scaling roughly linearly with tenant count until you're firmly in "hire a part-time DevOps" territory (hundreds of active tenants).

---

## 5. Pricing recommendation for your customers

### 5.1 Design principles (from your stated requirements)

- Two tracks: **managed platform** (we host, monthly subscription with usage tiers) and **BYO-API** (customer's own keys, smaller flat platform fee).
- Must be a genuine **cost saver vs. competitors**, and give **better output at low usage** — meaning even your cheapest tier should feel generous, not crippled, because your actual marginal cost per task is lower thanks to routing.
- Pricing should be simpler and more transparent than the "opaque credit" model competitors get criticized for.

### 5.2 Suggested tier structure — Managed Platform track

| Tier | Price | Positioning | What's included |
|---|---|---|---|
| **Starter** | **$19/mo** | Undercuts every competitor's entry tier ($49.99 Lindy, $20 Manus) while giving more real usage because of cost-routing efficiency | 1 office manager + up to 3 specialist agents, ~500 tasks/mo fair-use cap, standard GUI + one messaging channel (Telegram or Discord), email support |
| **Growth** | **$59/mo** | Direct undercut of Lindy Pro ($99.99) and Manus mid-tier | Up to 8 specialist agents, ~2,500 tasks/mo, all messaging channels (Telegram + Discord + WhatsApp), overnight autonomous task runs, priority routing to faster models when needed |
| **Studio** | **$149/mo** | Positioned against Lindy Max ($199.99) but with more transparent limits | Unlimited specialist agents, ~10,000 tasks/mo, animated office UI unlocked, multi-user seats (small team), audit log + usage dashboard, priority support |
| **Enterprise** | **Custom (~$400–2,000+/mo)** | Matches where competitors' "custom/contact sales" tiers sit | Dedicated tenant isolation, SSO, custom integrations, SLA, dedicated onboarding — priced per real infra + support cost, not a marketing number |

Note deliberately: no "$5,000/mo" tier as a self-serve option. If you ever see genuine demand at that price point, it should come from a real enterprise negotiation (custom integrations, SLA, dedicated infra) — not a shelf price nobody trusts.

### 5.3 Suggested pricing — BYO-API track

Since the customer pays their own model costs directly to Anthropic/OpenAI/etc., your fee is purely for orchestration, UI, memory/storage, and multi-channel access:

| Tier | Price | What's included |
|---|---|---|
| **BYO Starter** | **$9/mo** | Core orchestration + up to 3 agents + standard GUI, customer's own API key |
| **BYO Growth** | **$25/mo** | Up to 8 agents, all messaging channels, overnight autonomy, usage dashboard |
| **BYO Studio** | **$59/mo** | Unlimited agents, animated office UI, multi-user seats, audit log |

This track is deliberately cheap — it's a low-margin, high-goodwill acquisition channel for technical users (like your peer freelancers/agencies) who already have API access and just want your orchestration layer. It also builds trust: technical buyers can see exactly what they're paying for since it's not bundled with opaque model markup.

### 5.4 Why this pricing is defensible, not just "cheap"

- Your **actual marginal cost per task is lower** than competitors because of tiered routing + caching + batch APIs — so a $19 tier can include meaningfully more usage than a competitor's $49.99 tier without losing margin.
- **Transparent task/usage counts instead of opaque "credits"** directly answers the #1 complaint found in every competitor review researched above.
- **BYO track removes your LLM cost risk entirely** for price-sensitive technical customers, while still monetizing the orchestration/UX value you built.

---

## 6. Cost-saver mechanisms — how "better output on low usage" is actually delivered

This is the technical promise behind the pricing, restated as concrete mechanisms (fully detailed in `03_system_design.md`):

1. **Router-first execution** — every task is classified before any model call; simple tasks never touch expensive models.
2. **Prompt caching** — brand guidelines, system prompts, and repo context are cached, not resent.
3. **Context compression** — long histories are summarized by a cheap model before being handed to an expensive one.
4. **Batch APIs for overnight/non-realtime work** — typically ~50% cheaper than synchronous calls.
5. **Hard per-task and per-tenant budget ceilings** — protects both your margin and the customer from surprise bills.
6. **Retry-then-escalate, not infinite retry** — a stuck cheap-tier agent escalates once, rather than looping and burning spend.

Together, these are what let you legitimately advertise "frontier-model quality, budget pricing" without it being a marketing lie — the routing logic is doing real work to make that true.

---

## 7. Summary table — your two original questions, answered directly

| Question | Answer |
|---|---|
| **What will maintenance cost at the beginning?** | Realistically **$20–120/mo** depending on how heavily you personally use it, almost entirely LLM-usage-driven rather than infra-driven. Infra alone can be under $10/mo on free tiers. |
| **How much should I charge customers?** | Managed: **$19 / $59 / $149/mo** tiers, undercutting Lindy/Manus at every level. BYO: **$9 / $25 / $59/mo**, since you're not carrying their model cost. Enterprise stays custom-quoted, not a fixed shelf price. |
