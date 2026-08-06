# AI Employee Office Platform — Documentation Index

This is the full documentation set for **Orbicrew, by Leangine** (working title across older drafts: AI Employee Office Platform), split into focused documents so each can be read, updated, and referenced independently.

**Start here:** [01 — Requirements](../01_AI_Office_Platform_Requirements.md) (product overview one level up), then use this index to jump into whatever technical area you need.

All linked paths below are relative to this file (`tech/`).

---

## Document map

### Core product & build

| # | Document | Covers |
|---|---|---|
| 00 | *(this file)* [00_INDEX.md](./00_INDEX.md) | Navigation for the full doc set |
| 01 | [01_AI_Office_Platform_Requirements.md](../01_AI_Office_Platform_Requirements.md) | Executive summary, goals, target customers, system overview, UI-switcher requirements, agent management, Agent Town reference analysis, risks, phased plan, full open-source/tooling reference list *(lives one level up, outside `tech/`)* |
| 02 | [02_cost_and_pricing.md](./02_cost_and_pricing.md) | Competitor pricing research (Lindy, Manus, Hyperagent), Phase 0 maintenance cost breakdown, recommended pricing tiers for Managed and BYO tracks, cost-saver mechanisms |
| 03 | [03_system_design.md](./03_system_design.md) | Architecture diagrams (mermaid), component responsibilities, task lifecycle state machine, model routing, overnight execution, multi-tenant isolation, tech stack summary, open-source component map |
| 04 | [04_database_design.md](./04_database_design.md) | Full entity-relationship diagram, table-by-table design notes, indexing strategy, Row-Level Security approach, data retention rules |
| 05 | [05_api_design.md](./05_api_design.md) | REST endpoint reference, WebSocket protocol for live task status, internal channel-adapter protocol, auth/tenant resolution, rate limiting, error handling conventions |
| 06 | [06_security_and_scalability.md](./06_security_and_scalability.md) | Threat model, multi-tenant isolation, secrets management, autonomous-execution guardrails, sandboxed code execution, prompt injection mitigation, compliance notes, scalability stages, monitoring |
| 07 | [07_phased_development_plan.md](./07_phased_development_plan.md) | Gantt-style build timeline, task-by-task breakdown per phase, exit criteria per phase, sequencing rationale |
| 08 | [08_devops_and_deployment.md](./08_devops_and_deployment.md) | Environment strategy, CI/CD, containerization, deployment topology by phase, secrets, observability, self-hosted Postgres, backup/DR, deployment checklist |

### Brand, UI & Orbit View

| # | Document | Covers |
|---|---|---|
| 09 | [09_brand_identity.md](./09_brand_identity.md) | **Locked:** Orbicrew, by Leangine — Deep Violet tokens, Bricolage Grotesque + Hanken Grotesk, three-tier logo, voice. Sections 1–5 keep naming/palette research history; **Section 6 is authoritative** |
| 10 | [10_ui_ux_guide.md](./10_ui_ux_guide.md) | Stitch-ready product UI spec: brand template in Section 0, full page inventory (marketing, onboarding, dashboard, chat, agents, settings). Orbit View summarized with pointers to [18](./18_orbit_view_game_ui.md) and [19](./19_orbit_view_stitch_prompts.md) |
| 18 | [18_orbit_view_game_ui.md](./18_orbit_view_game_ui.md) | Product/behavior source of truth for Orbit View: room catalog, idle-time FSM, delegation model, 2D rendering approach, zoom tiers, decor/banner, sandboxed cosmetic scripting, implementation notes (Phaser-class world + DOM chrome) |
| 19 | [19_orbit_view_stitch_prompts.md](./19_orbit_view_stitch_prompts.md) | **Self-sufficient** copy-paste Stitch pack for Orbit View frames: brand lock, all screens/states, component inventory, animation matrix, acceptance checklist, Agent Town research appendix — paste alone into Stitch; no other docs required for generation |

### Reliability, security depth & architecture variants

| # | Document | Covers |
|---|---|---|
| 11 | [11_hallucination_isolation_and_scaling.md](./11_hallucination_isolation_and_scaling.md) | Cross-project/cross-client isolation (`projects` table), hallucination mitigation pipeline, Office Manager registry/focus-session pattern, resuming previously assigned jobs |
| 12 | [12_adversarial_security.md](./12_adversarial_security.md) | OWASP Top 10 for Agentic Applications (ASI01–ASI10), layered prompt-injection defense, supply-chain risk for user/marketplace agents, implementation priority order |
| 13 | [13_byo_provider_architecture.md](./13_byo_provider_architecture.md) | BYO patterns (direct key, OpenRouter, self-hosted endpoint), self-hosted LiteLLM gateway recommendation, cost-minimization levers |
| 14 | [14_universal_task_coverage.md](./14_universal_task_coverage.md) | Task-type generalization (coding, DevOps, design, video, research, etc.) and infrastructure/server-access agent safeguards |
| 15 | [15_performance_training_and_settings.md](./15_performance_training_and_settings.md) | Speed at scale, research-speed guidance, two-tier agent training, tenant defaults + per-agent override, ask-vs-proceed clarification behavior |

### Compliance & go-to-market

| # | Document | Covers |
|---|---|---|
| 16 | [16_open_source_licensing_compliance.md](./16_open_source_licensing_compliance.md) | License audit for planned components under commercial SaaS/BYO goals; flags LangGraph Platform, LiteLLM Enterprise, MinIO AGPL, Redis history, ElevenLabs use-vs-resell |
| 17 | [17_marketing_plan.md](./17_marketing_plan.md) | Global English-first GTM: segments, pain points, channels, messaging, founder-led outreach for first 10–50 customers, retention, phase-tied sequencing with [07](./07_phased_development_plan.md) |

---

## Quick links by topic

| Topic | Primary docs |
|---|---|
| Product vision & requirements | [01](../01_AI_Office_Platform_Requirements.md) |
| Pricing / business model | [02](./02_cost_and_pricing.md) |
| Architecture & stack | [03](./03_system_design.md) → [04](./04_database_design.md) → [05](./05_api_design.md) |
| Build sequence | [07](./07_phased_development_plan.md) |
| Deploy / ops | [08](./08_devops_and_deployment.md) |
| Brand (locked Orbicrew) | [09](./09_brand_identity.md) §6 |
| Standard product UI (Stitch) | [10](./10_ui_ux_guide.md) |
| Orbit View behavior | [18](./18_orbit_view_game_ui.md) |
| Orbit View Stitch frames | [19](./19_orbit_view_stitch_prompts.md) *(self-sufficient)* |
| Agent trust / hallucination | [11](./11_hallucination_isolation_and_scaling.md) |
| Adversarial / injection | [12](./12_adversarial_security.md) · baseline also in [06](./06_security_and_scalability.md) |
| BYO providers | [13](./13_byo_provider_architecture.md) |
| Task-type coverage | [14](./14_universal_task_coverage.md) |
| Speed / training / settings | [15](./15_performance_training_and_settings.md) |
| OSS license risk | [16](./16_open_source_licensing_compliance.md) |
| Marketing / GTM | [17](./17_marketing_plan.md) |

---

## Suggested reading order

**If you're about to start building:**  
[03_system_design.md](./03_system_design.md) → [04_database_design.md](./04_database_design.md) → [05_api_design.md](./05_api_design.md) → [07_phased_development_plan.md](./07_phased_development_plan.md)

**If you're deciding on pricing/business model:**  
[02_cost_and_pricing.md](./02_cost_and_pricing.md) (root [requirements](../01_AI_Office_Platform_Requirements.md) Section 8 has the summary)

**If you're concerned about agent reliability/trust:**  
[11_hallucination_isolation_and_scaling.md](./11_hallucination_isolation_and_scaling.md)

**If you're concerned about hackers, bad actors, or prompt injection:**  
[12_adversarial_security.md](./12_adversarial_security.md)

**If you're designing BYO-API support or cheap execution:**  
[13_byo_provider_architecture.md](./13_byo_provider_architecture.md)

**If you're scoping task types (DevOps, design, video, server access, etc.):**  
[14_universal_task_coverage.md](./14_universal_task_coverage.md)

**If you're thinking about speed, agent training, or user-tunable settings:**  
[15_performance_training_and_settings.md](./15_performance_training_and_settings.md)

**Before you build or sell commercially:**  
[16_open_source_licensing_compliance.md](./16_open_source_licensing_compliance.md)

**If you're about to design the actual UI:**  
[09_brand_identity.md](./09_brand_identity.md) (use **§6 locked Orbicrew**) → [10_ui_ux_guide.md](./10_ui_ux_guide.md) for Standard Dashboard / product chrome.  
For Orbit View: behavior in [18_orbit_view_game_ui.md](./18_orbit_view_game_ui.md); for Stitch frame generation use [19_orbit_view_stitch_prompts.md](./19_orbit_view_stitch_prompts.md) alone (it already inlines brand + behavior). Note: Stitch produces chrome/frame mocks — the living office world is implemented as a 2D engine (e.g. Phaser) + DOM HUD per [18](./18_orbit_view_game_ui.md).

**If you're planning go-to-market:**  
[17_marketing_plan.md](./17_marketing_plan.md)

**If you're preparing for real customers:**  
[06_security_and_scalability.md](./06_security_and_scalability.md) → [08_devops_and_deployment.md](./08_devops_and_deployment.md)

---

## Document status

All documents are v1 drafts, dated August 2026, written as living references — update them as real usage data, pilot feedback, and engineering decisions refine the plan. Where a document lists options (pricing tiers, fonts, palettes, names), treat unresolved options as research history unless a later section marks a **locked** decision.

**Locked brand (current):** product name **Orbicrew, by Leangine**; palette **Deep Violet**; type **Bricolage Grotesque + Hanken Grotesk** — see [09_brand_identity.md](./09_brand_identity.md) Section 6.

**Global-market correction (applies across the doc set):** the product, brand, and marketing plan are global and English-first. Multilingual chat (including Bangla) is a product capability — people can converse with agents in any language — but it is not a market strategy and does not shape naming, typography, UI, or campaign targeting. If any passage still frames Bangla as a primary market or brand driver, treat that passage as stale.

**Naming note:** [09](./09_brand_identity.md) keeps shortlist research for history. Several earlier candidates (Rosterly, Deskforce, Coworkr, and others) were removed after collision checks. `leangine.com` is the existing Leangine company brand. Remaining historical name options are better-vetted than an untested list, but only **Orbicrew** is the locked product name; still run WHOIS/trademark diligence before heavy brand spend if revisiting alternatives.
