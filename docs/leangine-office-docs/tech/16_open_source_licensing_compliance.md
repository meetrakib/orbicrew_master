# Open Source Licensing & Commercial Compliance

Part of the AI Employee Office Platform documentation set. See `00_INDEX.md` for the full document list.

You asked directly: for every open-source tool this project plans to use or customize, do we actually have permission to do that, given we're selling this commercially (SaaS or otherwise)? This document answers that honestly, tool by tool, based on real license research — not an assumption that "open source" automatically means "free to build a commercial product on."

**This is not legal advice.** License terms and their interpretation can be nuanced, and terms of service change over time. Treat this as a solid starting map, and have an actual lawyer review the final stack before launch, especially the SaaS-resale-specific clauses flagged below.

---

## 1. The core distinction that matters for a commercial SaaS

Two different questions get conflated constantly, and this document keeps them separate for every tool:

1. **Can I use this code/model in my product and sell my product?** (Usually yes, for permissively-licensed tools.)
2. **Can I resell or sublicense the tool/service itself as part of my offering?** (Often no, even when #1 is yes — this is the ElevenLabs case below, and it's a real trap.)

Most "is X open source" questions online only answer #1. A commercial SaaS reselling access to these tools (e.g., a customer's BYO OpenRouter key routed through your gateway, or your own pooled ElevenLabs account powering many customers' voice features) needs both answered.

---

## 2. Tool-by-tool audit

### 2.1 LangGraph — orchestration core

| | |
|---|---|
| Core library (`langgraph`, `langgraph-checkpoint`, `langgraph-sdk`, `langgraph-prebuilt`) | **MIT** — <cite index="162-1">MIT-licensed open-source library, free to use</cite>, <cite index="163-1">no licensing fees or royalties even for commercial products, no obligation to open-source your own application code — suitable for integration into commercial SaaS products.</cite> |
| **`langgraph-api` (the server runtime component)** | **NOT MIT — Elastic License 2.0.** <cite index="160-1">To run this specific server runtime in production, you need a commercial license key, typically tied to an Enterprise plan.</cite> |

**What this means for you**: the core LangGraph library (the graph/state-machine primitives the Office Manager and specialist agents are built on, per `03_system_design.md`) is genuinely free for commercial use. **But if you use LangGraph's own hosted server runtime (`langgraph-api`) rather than building your own API layer around the core library, that specific component requires a paid Enterprise license.** Our architecture already calls for a custom API Gateway and orchestration backend (`03_system_design.md` Section 2, `05_api_design.md`) rather than depending on LangGraph's own server product — **this is the correct choice already, and this finding confirms it's not just an architectural preference but a licensing necessity.** Action item: explicitly confirm during implementation that no `langgraph-api` package or LangGraph Platform hosted service is pulled in — only the core `langgraph` library.

### 2.2 Agent Town (UI reference) — fork candidate

| | |
|---|---|
| License | **MIT** (per `01_AI_Office_Platform_Requirements.md` Section 6, confirmed from the repo's LICENSE file) |
| Commercial use | Yes — MIT permits commercial forking, modification, and resale with attribution preserved |

No new concern here beyond what's already documented — MIT is the most permissive common license, forking this for the visual/office-UI layer (per the existing "fork the UI layer only" decision) is fully compliant. Preserve the copyright notice per MIT's one real requirement.

### 2.3 OpenClaw — evaluated, not depended on long-term (per existing recommendation)

| | |
|---|---|
| Core Gateway | **MIT** — <cite index="169-1">commercial use allowed</cite> |
| Official AgentSkills | **MIT** — <cite index="169-1">commercial use allowed</cite> |
| Third-party ClawHub skills | **Varies** — <cite index="169-1">mixed licenses (MIT, Apache, and others) depending on the individual skill author</cite> |

This confirms the existing recommendation in `03_system_design.md` Section 12 (don't depend on OpenClaw as the core runtime long-term) is doubly correct: even setting aside the "someone else's roadmap" risk already documented, **any third-party skill from its marketplace carries its own, unaudited license** — a real risk if your own system ever integrates specific OpenClaw skills rather than just taking UI inspiration. Since our recommendation is to replace OpenClaw with our own LangGraph-based runtime entirely, this risk doesn't carry forward into the actual build — flagged here only for completeness and in case the fork temporarily depends on it during early prototyping.

### 2.4 LiteLLM — BYO gateway (per `13_byo_provider_architecture.md`)

| | |
|---|---|
| Core gateway/proxy | **MIT** — <cite index="179-1">content outside the `enterprise/` directory is MIT-licensed</cite> |
| Enterprise features (SSO, audit logs, RBAC, semantic caching) | **Separate commercial license**, <cite index="182-1">requiring a paid license key, roughly starting at $250/month for the Basic enterprise tier</cite> |
| **Caveat worth knowing** | <cite index="178-1">There's an active, unresolved GitHub issue about the license boundary not being cleanly/consistently enforced in the codebase — some documented "Enterprise" features have their gating logic or entire implementation sitting in nominally MIT-licensed files, making the actual free/paid line genuinely unclear in places.</cite> |

**What this means for you**: the core routing/gateway functionality this project actually needs (unified provider access, cost tracking, basic budgets/rate limits per `13_byo_provider_architecture.md` Section 4.1) is in the free MIT-licensed core. **The specific enterprise-tier features mentioned in that document — SSO, fine-grained RBAC, audit-log retention policies — are NOT free** and would need either a paid LiteLLM Enterprise license or a custom-built equivalent using your own Postgres audit tables (which `04_database_design.md`'s `task_steps` table already substantially covers, making a custom build the more likely path anyway). Action item: `06_security_and_scalability.md` and `13_byo_provider_architecture.md` should be read with this distinction in mind — build audit logging and RBAC in your own application layer (already mostly planned) rather than assuming LiteLLM's enterprise tier is free.

### 2.5 Whisper (STT)

| | |
|---|---|
| License | **MIT** — <cite index="188-1">confirmed via the official OpenAI repository LICENSE file</cite>, <cite index="187-1">allowing for commercial and non-commercial use</cite> |

Clean — no gotchas. Self-hosted Whisper (per `03_system_design.md` and `13_byo_provider_architecture.md`) is fully compliant for commercial use, including reselling a product built on it.

### 2.6 Pipecat / LiveKit Agents (voice pipeline)

| | |
|---|---|
| Pipecat | Open source (framework itself; verify the specific license file at implementation time — not fully confirmed in this research pass) |
| LiveKit Agents SDK | **Apache-2.0** — <cite index="197-1">confirmed directly from the official repository</cite> |
| LiveKit server | **Apache-2.0** — <cite index="204-1">confirmed directly from the official repository</cite> |
| **LiveKit turn-detection models specifically** | **Separate "LiveKit Model License"** — <cite index="197-1">not Apache-2.0 like the rest of the framework</cite> |

**What this means for you**: Apache-2.0 is permissive and commercial-use-friendly (similar practical effect to MIT, plus an explicit patent grant). **The one flagged exception is LiveKit's turn-detection models specifically** — if the voice pipeline design ends up using LiveKit's own turn-detection feature (deciding when a user has finished speaking), check the actual "LiveKit Model License" terms at implementation time rather than assuming Apache-2.0 blanket coverage extends to it.

### 2.7 Phaser (game engine, for the office UI)

| | |
|---|---|
| License | **MIT** — <cite index="200-1">confirmed via the official Phaser license page</cite> |

Clean, no gotchas — <cite index="205-1">MIT allows commercial closed-source use with essentially no obligation beyond including the copyright notice somewhere in the shipped product, such as a credits screen or a NOTICES file.</cite>

### 2.8 Postgres, pgvector, Redis, MinIO (self-hosted data layer)

All four are long-established, widely-commercially-deployed open-source projects (PostgreSQL License — a permissive MIT-style license; pgvector — PostgreSQL License; Redis — dual-licensed, core is source-available under RSALv2/SSPLv1 as of recent versions, worth a specific check depending on which Redis version/fork is used, since Redis's licensing has changed more than once in recent years; MinIO — AGPLv3 for the server, which is copyleft and has specific implications). **Flagging Redis and MinIO specifically as needing a fresh license check at implementation time**, since both have had real, publicized licensing changes in the last few years that a general "it's open source" assumption could miss.

- **Redis**: if using an older Redis version genuinely under the original BSD-3-Clause, that's fully permissive. If using a newer version under RSALv2/SSPLv1 (Redis Inc.'s post-2024 licensing), commercial *use* internally is generally fine but *offering Redis itself as a hosted service to third parties* has real restrictions — this matters because your product isn't reselling Redis directly, it's using Redis as internal infrastructure, which is the permitted case, but worth confirming which exact Redis distribution/version is deployed, and consider a fully-open fork (e.g., Valkey, the Linux Foundation's BSD-licensed Redis fork created specifically in response to this licensing change) as a safer default.
- **MinIO**: the MinIO *server* is **AGPLv3**, a copyleft license — this is meaningfully different from the permissive MIT/Apache pattern elsewhere in this stack. AGPL's copyleft obligations are triggered by *modifying and distributing* the software or *offering it as a network service*; using MinIO as unmodified internal infrastructure (object storage your own backend talks to, not something you expose or resell directly to customers) is the common, generally-accepted use case, but this deserves specific legal review before launch given AGPL is the one genuinely restrictive license in this whole stack. **Cloudflare R2 (the alternative already offered in `08_devops_and_deployment.md`) sidesteps this entirely**, since it's a paid managed service you're a customer of, not open-source code you're distributing — worth weighing this AGPL nuance against the self-hosting-purity goal from `02_cost_and_pricing.md`'s vendor-lock-in discussion.

### 2.9 Google Fonts (Bricolage Grotesque, Hanken Grotesk, and all other options in `09_brand_identity.md`)

<cite index="55-1">All fonts on Google Fonts are open-source and free for both personal and commercial use — usable on client projects, SaaS products, e-commerce stores, and any other commercial website without licensing fees.</cite> Clean across the board, no action needed.

### 2.10 ElevenLabs (TTS) — the real "resell vs. use" trap

This is the most important finding in this entire document, precisely because it's *not* an open-source tool and the mistake here is a different kind of mistake — a terms-of-service one, not a license one.

| | |
|---|---|
| Model | Paid API service, not open source |
| Commercial use of *output* | <cite index="207-1">Yes, on paid plans — you can integrate AI voices into apps, games, software, and other commercial products, and the audio output is yours to distribute.</cite> |
| **Reselling the *service* itself** | <cite index="207-1">Explicitly NOT allowed — "you cannot resell the ElevenLabs service itself, only the output content."</cite> |
| Reseller/OEM programs | <cite index="212-1">ElevenLabs does have a formal OEM/Reseller program with its own defined terms ("Bundled Service" — the combined offering of ElevenLabs' Services and your own Customer Solution), suggesting a proper path exists for exactly this kind of embedded commercial use, but it requires a specific agreement, not just a standard paid API key.</cite> |

**What this means for your two pricing tracks specifically**:

- **Managed platform tenants** (using your pooled ElevenLabs account, per `02_cost_and_pricing.md`): this is fine under standard paid-plan terms *if structured correctly* — you're using ElevenLabs to generate voice output for tasks your own agents perform, which is normal commercial API usage, not reselling ElevenLabs access. But if the product surface ever looks like "customers get a raw ElevenLabs API key/quota through us," that crosses into the reseller territory needing the formal OEM/Reseller agreement — worth being deliberate that customers interact with *your product's* voice feature, never with something that looks like direct ElevenLabs account access.
- **BYO tenants** who bring their own ElevenLabs key: this is unambiguously fine — they have their own direct relationship with ElevenLabs under their own account, your platform is just calling their key on their behalf, same pattern already established for BYO LLM keys in `13_byo_provider_architecture.md`.
- **Action item**: before launch, have a direct conversation with ElevenLabs (or review their current Reseller/OEM terms directly) about whether the managed-tier voice feature, as actually built, needs a formal Reseller/OEM agreement rather than a standard paid plan — this is exactly the kind of judgment call worth a real legal/business conversation, not a doc-based guess.

### 2.11 Google Cloud TTS (alternative to ElevenLabs)

Standard Google Cloud Platform terms of service apply — using it as a backend for your own product's voice feature (not reselling GCP access directly) is the normal, well-established commercial use case for any GCP API, with no unusual resale restriction found in this research pass. Lower research priority than ElevenLabs here since GCP's standard terms are well-trodden ground for exactly this pattern.

---

## 3. Summary table

| Tool | License/terms | Commercial use in your product | Reselling the tool/service itself |
|---|---|---|---|
| LangGraph (core) | MIT | ✅ Yes | N/A (it's a library, not a service) |
| LangGraph API/Platform | Elastic License 2.0 | ⚠️ Requires paid Enterprise license for production server use | N/A — don't use this component |
| Agent Town | MIT | ✅ Yes | ✅ Yes (with attribution) |
| OpenClaw core | MIT | ✅ Yes (not depended on long-term per existing architecture decision) | ✅ Yes, but third-party skills vary |
| LiteLLM (core) | MIT | ✅ Yes | ✅ Yes |
| LiteLLM Enterprise features | Commercial license | ⚠️ Paid tier (~$250+/mo) or build your own equivalent | N/A |
| Whisper | MIT | ✅ Yes | ✅ Yes |
| LiveKit Agents / server | Apache-2.0 | ✅ Yes | ✅ Yes |
| LiveKit turn-detection models | Separate Model License | ⚠️ Check specific terms if used | Unclear — check terms |
| Phaser | MIT | ✅ Yes | ✅ Yes |
| Postgres / pgvector | PostgreSQL License (permissive) | ✅ Yes | ✅ Yes |
| Redis | Version-dependent (BSD-3 or RSALv2/SSPLv1) | ✅ Yes as internal infra | ⚠️ Confirm version; consider Valkey fork |
| MinIO | AGPLv3 (server) | ⚠️ Generally fine as unmodified internal infra; get real legal review | ❌ Copyleft triggers on distribution/network service — avoid modifying and redistributing |
| Google Fonts | Open Font License | ✅ Yes | ✅ Yes |
| ElevenLabs | Paid API, ToS-governed | ✅ Yes, output is yours | ❌ Cannot resell the service itself — may need formal OEM/Reseller agreement for managed-tier voice feature |
| Google Cloud TTS | Standard GCP ToS | ✅ Yes | Standard GCP terms, no unusual restriction found |

---

## 4. What to do with this before launch

1. **Confirm at implementation time**: no `langgraph-api`/LangGraph Platform package gets pulled in — core `langgraph` only.
2. **Confirm at implementation time**: LiteLLM enterprise-tier features aren't silently relied upon — build audit/RBAC in your own Postgres-backed application layer (already substantially the plan).
3. **Get a real legal opinion, not just this document**, on: MinIO's AGPL implications for your specific deployment shape, and whether your managed-tier ElevenLabs usage needs a formal Reseller/OEM agreement.
4. **Confirm Redis version/distribution** being deployed and its current license, or default to Valkey (BSD-licensed fork) to sidestep the question entirely.
5. **Keep a running open-source attribution file** (a `NOTICES.md` or in-app "open source licenses" page) listing every dependency and its license — standard practice for a commercial product built substantially on open source, and something investors/enterprise customers will eventually ask for.
