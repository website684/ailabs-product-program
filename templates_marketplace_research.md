# BetterPlace AI Labs — Agent Templates & Marketplace: PRD-Ready Synthesis

## 1. TEMPLATE ANATOMY

**Fixed (versioned, in template artifact) vs. org-bound (resolved at install), mapped to our config model:**

| Field | Status | Mechanism |
|---|---|---|
| `chat_workflow_steps`/`workflow_steps` | FIXED | Immutable per template_version. May contain marked "NL-editable" prompt sub-slots (see below). |
| `tool_ids` | FIXED capability list + BOUND instance | Template declares required tool *capability types*; actual `tool_id`/credential resolved at install — no secrets ship in the artifact. |
| `mcp_servers` | FIXED capability list + BOUND endpoint | Same pattern as `tool_ids`: template names required MCP server *kinds*, org supplies endpoint+auth at install. |
| `document_ids` | BOUND, always | Never in the versioned artifact. Template defines a "document slot" schema (required doc types, resync cadence) — org attaches its own docs at install. |
| `guardrail_policy` | FIXED floor + org-extensible | Vendor-owned non-negotiable floor ships with template; org may *tighten*, never loosen. Field-level "manageability" flag decides what upgrades can auto-propagate. |
| `UiConfig` | BOUND, always | Org branding/theme/logo. |
| `VoiceConfig` | BOUND, always | Voice ID, telephony number; template only flags voice-channel support as fixed capability. |
| `language[]` | FIXED certified set → BOUND subset | Template ships a *certified* language list (tested prompts/voice assets); org activates a subset at install. Adding an uncertified language requires re-certification, not a config edit. |

**Binding/slots mechanism (recommend):** Split every template package into
`{fixed: workflow_steps, prompts, guardrail_floor, certified_languages[], required_tool_capabilities[], required_mcp_capabilities[]}` and
`{bound_at_install: tool_ids, mcp_servers, document_ids, UiConfig, VoiceConfig, active_language_subset}`.
Steals from: n8n's "no credential values travel with the export" rule (tool_ids/mcp_servers), Lindy's per-install Knowledge Base object w/ resync (document_ids), and Custom GPT's four-block separation (instructions/knowledge/actions/capabilities) as the conceptual template.

Add a **third category** — "NL-customizable at runtime": bounded prompt sub-slots within `workflow_steps` that org admins edit via natural language without cutting a new template version (steals from Fountain's Cue / Sierra's Ghostwriter), constrained to stay inside the guardrail floor.

**Versioning + upgrade model:**
- **Version/alias split**: immutable, numbered `template_version` + a stable per-tenant `installed_agent_alias` pointer. Upgrade = repoint alias, never mutate the live tenant agent. Steals from AWS Bedrock Agent Alias (`UpdateAgentAlias`) and Salesforce Agentforce (20 versions, one active, instant zero-downtime swap).
- **Draft → Commit → Activate** authoring lifecycle before a version is install/upgrade-eligible (Salesforce Agentforce lifecycle) — the commit gate doubles as the certification checkpoint (see §3).
- **Per-field manageability flags** (vendor-managed / org-locked / org-overridable) on template fields decide what an upgrade can auto-propagate vs. must preserve — steals from Salesforce 2GP "subscriber editable" metadata.
- **Publisher-initiated, selectively-targeted push upgrades** (choose which tenant aliases, which version, when) — steals from Salesforce push-upgrade targeting; never silent auto-push-to-all.
- **Fork escape hatch**: orgs needing changes beyond slots can fork, which explicitly severs the upgrade-propagation link (Zapier's fork-not-subscribe model) — surface that tradeoff at fork time so the org knows it now owns patching.

## 2. INSTALL EXPERIENCE

**Flow:** Browse → Preview (workflow diagram, sample conversation, declared slots) → Install wizard configures **only bound slots** (tool credentials/MCP endpoints, document_ids ingest, UiConfig branding, VoiceConfig number + active `language[]` subset) → Preflight → Sandboxed test conversation → Publish/Activate.

- The installer touches bound slots only — never `workflow_steps`, prompts, or the guardrail floor directly (that requires the NL-customization sub-slot or a fork).
- **Time-to-value target**: same-day activation, <60 min for a standard template against existing org docs, <1 day if new tool/MCP connections are required. (Sierra's 4–10 week sales-led onboarding is the anti-pattern that forced them to build Ghostwriter — don't replicate a discovery-call-led install.)
- **Preflight checks**: required `tool_ids`/`mcp_servers` reachable+authenticated; `document_ids` ingested and indexed; `guardrail_policy` floor intact and unmodified; `language[]` subset has certified prompt/voice assets; `UiConfig` renders; scripted eval set passes in sandbox (lighter-weight analog of ServiceNow's continuous build-time security/perf checks).
- **Test-before-publish**: mandatory sandbox conversation(s) against the org's real bound documents/tools before the alias flips live — mirrors Agentforce's Commit-before-Activate gate; block publish on preflight failure.
- Default to a **guided one-click install** that auto-provisions required infra (Microsoft Copilot Studio's marketplace-install pattern); reserve manual/raw-config install for IT-authorized advanced use only.
- **Install once, surface everywhere**: one install binds the agent across configured channels (web widget/WhatsApp/voice) rather than per-channel reinstall — steals from Microsoft 365 Agent Store.

## 3. MARKETPLACE SHAPE

- **Certification**: staged, not single-pass — (1) architecture/scope review of declared slots, (2) continuous automated security + guardrail-floor scan during build, (3) formal manual cert (prompt-injection/policy review) before listing, steals from ServiceNow Store's 3-stage path. Any change to fixed template logic forces a new version + **full** re-cert, no partial re-review — this is the direct fix for the GPT Store's failure where 100+ policy-violating GPTs stayed live because review was automated-only.
- **First-party vs. partner**: clear "Official/Verified" badge for BetterPlace-authored templates; partner templates pass the identical cert gate plus a signed revenue-share agreement.
- **Discovery**: designed categories by industry (frontline/BFSI/retail/logistics) and role (HR/ops/recruiting) plus use-case search — never a flat generic grid. GPT Store's "static, generic Explore page" is the explicit anti-pattern; support browsing by vertical "skills library" (Salesforce Agentforce for Retail/Manufacturing pattern) instead of one generic template reskinned per industry.
- **Ratings/quality signals**: surface usage-based signals (live-install count, resolution rate) on the listing card, not star ratings alone — the GPT Store's real failure wasn't its 5% publish rate, it was that published GPTs still went undiscovered and unused; optimize for post-publish usage metrics, not publish-count vanity metrics.
- **What to explicitly avoid from GPT Store**: (a) publish-then-invisible — no dedicated discovery UX for 95%+ of listings; (b) monetization promised then delayed/unclear, eroding builder trust; (c) light/automated-only moderation missing live violations at scale.

## 4. COMMERCIALS

Recommend **hybrid metering**, not a single axis — every vendor surveyed that launched single-axis (Agentforce's flat $2/conversation, Sierra's pure per-outcome) was later forced to add a second pricing dimension.

1. **Per-outcome, tiered** for high-value conversations — define "outcome" directly in the template schema (tag which `workflow_steps` node = billable outcome), tiered like Intercom Fin ($0.99 routine resolution vs. $9.99 qualified/high-value outcome) rather than Sierra's flat $1–2.50 — fits frontline-HR use cases where "document verified" and "candidate fully onboarded" carry very different value.
2. **Flat/predictable tier** for SMB/mid-market frontline buyers — steals from Paradox ($1,000–1,500/mo + location scaling) or apna's job-post-embedded bundled model. Findings indicate India/SEA frontline HR buyers want predictable cost, not usage risk — prioritize this over pure per-outcome billing for that segment.
3. **Credit/action-based fallback** for granular, agentic multi-tool-call templates — steals from Agentforce Flex Credits (~$0.005/credit).

**Partner revenue share**: adopt Salesforce AgentExchange's transparent tiered model — ~15% platform share on paid partner installs (higher for OEM-style listings), lower marginal rate past a volume/AOV benchmark, no setup/monthly listing fee, only payment-processing pass-through. Publish exact %, cadence, and floor terms up front — avoid the GPT Store's mistake of an announced-but-undelivered/opaque revenue-share program.

## 5. TOP 8 MECHANICS TO STEAL

1. **Version/Alias split** for zero-downtime upgrades — AWS Bedrock Agent Alias (`UpdateAgentAlias`) / Salesforce Agentforce (20 versions, 1 active, instant swap).
2. **Draft → Commit → Activate** lifecycle with channels bound to a committed version — Salesforce Agentforce.
3. **Per-field "manageability" flags** (vendor-managed vs. subscriber-editable) to prevent upgrades clobbering org customizations — Salesforce 2GP managed packages.
4. **Publisher-initiated, selectively-targeted push upgrades** (choose tenant/version/timing) — Salesforce push upgrade.
5. **No-secrets-in-artifact credential re-resolution** at import — n8n workflow import.
6. **Per-install Knowledge Base object with live resync**, addable/removable independent of workflow logic — Lindy.
7. **Staged certification** (architecture review → continuous build-time security/perf → formal cert, full resubmission on fixed-logic change) — ServiceNow Store.
8. **Guided one-click install that auto-provisions infra**, manual path reserved for IT-authorized advanced use — Microsoft Copilot Studio Agent Library.

*(9th, worth including): "Install once, surface everywhere" multi-channel binding — Microsoft 365 Agent Store.*


---

# Appendix: findings by dimension


## marketplace-patterns

- **[mechanic]** Salesforce's Summer '25 release lets Agent Templates be shipped inside First- and Second-Generation Managed Packages — partners package a fully defined agent template (topics, actions, instructions, prompts) and the installing org's customer creates a live agent from that template in minutes, inheriting the org's own security settings, field-level security, and sharing rules automatically at install time.  
  Source: https://aquivalabs.com/blog/agent-templates-are-now-packageable-heres-why-that-matters-for-your-agentforce-strategy/
- **[mechanic]** AgentExchange (launched March 2025) requires listed components — agent actions, topics, prompt templates, full agent templates — to pass 'rigorous security and customer reviews' before listing, and launched with 200+ partners and hundreds of pre-vetted components; installed components inherit the org's existing security/sharing model rather than bringing their own.  
  Source: https://vantagepoint.io/blog/sf/salesforce-agentexchange-buyers-guide-ai-agents
- **[number]** AgentExchange Checkout revenue share is 15% of net revenue for ISVforce partners (25% for OEM) on paid listings, plus Stripe's $0.30/transaction fee on card payments; free apps carry no Salesforce revenue share, and no setup/monthly/card-storage fees are charged — with a 'Marginal PNR' model giving partners lower marginal royalty rates once they cross an Average Order Value benchmark.  
  Source: https://developer.salesforce.com/docs/platform/isvforce/guide/appexchange-checkout-rev-share.html
- **[pattern]** Best practice for customizing installed AgentExchange components is to NOT modify the original installed component directly — modifications risk being overwritten on package updates — instead partners/customers are told to create custom clones or extend components, i.e. the platform doesn't offer safe in-place override of fixed template logic.  
  Source: https://vantagepoint.io/blog/sf/salesforce-agentexchange-buyers-guide-ai-agents
- **[requirement]** ServiceNow Store certification separates 'security/performance review during build' from a final formal certification pass: a typical path is discovery/architecture (1-2 wks) defining scope and certification requirements, build/test (2-3 wks) with security and performance reviewed continuously, then a certification review (3-5 wks, sometimes 5-8 wks total to listing) where any code change forces a new version number and a fresh full resubmission — no partial re-review.  
  Source: https://xpertappdev.com/services/certification-guide.html
- **[mechanic]** Microsoft's Agent Library/Agent Store gives two install paths with different admin burden: clicking Install on a template imports the solution directly and automatically enables required code components in the environment, whereas a manual GitHub-based install requires an admin to separately enable code components in environment settings — the guided marketplace path is explicitly recommended over the raw-files path.  
  Source: https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-install-agent
- **[pattern]** Microsoft 365's curated Agent Store (70+ agents from Microsoft, partners, and customers) is explicitly positioned as 'install once, use in multiple places across the M365 ecosystem' — i.e. the binding unit is a single install action that then surfaces the agent across Teams/Outlook/Copilot Chat surfaces rather than per-surface reinstallation.  
  Source: https://learn.microsoft.com/en-us/microsoft-365/copilot/copilot-agent-store
- **[mechanic]** Lindy's template install flow attaches a Knowledge Base object per install that supports live-resyncing sources (files/docs/websites/audio up to 20MB, Google Drive/OneDrive/Dropbox) on a 24-hour resync cycle, and the KB can be freely added-to/reconfigured/removed post-install without touching the rest of the template's automation logic — a clean example of separating fixed workflow from bound-at-install data.  
  Source: https://www.lindy.ai/blog/knowledge-base-and-lindy-embed
- **[number]** The GPT Store's core discovery/monetization failures at scale: of 3M+ custom GPTs created, only ~159,000 (about 5%) were ever published to the public store (95% attrition before publish), the promised usage-based revenue-share program slated for Q1 2024 was delayed/never clearly delivered on schedule, and the Explore page was criticized as 'static and generic' making post-publish discovery hard even for listed GPTs — publishing is not the bottleneck, discoverability and unclear monetization are.  
  Source: https://en.wikipedia.org/wiki/GPT_Store
- **[pattern]** A Gizmodo-cited investigation found 100+ GPTs in the public GPT Store violating OpenAI's own usage policies (explicit content, academic-cheating aids, unvetted medical/legal advice) still live post-launch, showing that OpenAI's pre-publish review/moderation for the store did not reliably catch policy violations — a caution against a lightweight/automated-only certification gate for a template marketplace.  
  Source: https://winbuzzer.com/2024/09/05/openai-struggles-with-gpt-store-policy-enforcement-xcxwbn/
- **[number]** GPT Store per-conversation payouts reportedly average around $0.03/conversation, requiring roughly 33,000+ qualifying conversations to reach $1,000/month, with a minimum-engagement bar (around 25 conversations/week) below which a listed GPT earns nothing at all — most builders fall under this floor and see zero payout despite being published.  
  Source: https://www.digitalapplied.com/blog/gpt-store-custom-gpts-business-guide-2026

## multitenant-templates

- **[number]** Salesforce Agentforce lets you create up to 20 versions of an agent, but only one version is 'active' at a time; activating a new version replaces the old one instantaneously with zero downtime for end users.  
  Source: https://help.salesforce.com/s/articleView?id=ai.agent_versions_lifecycle.htm&language=en_US&type=5
- **[mechanic]** Agentforce's Winter '26 release restructures agent metadata so each agent version is fully isolated — changes to one version cannot leak side effects into another — which is the mechanism that lets orgs adopt versioning without deactivating live agents during deploys.  
  Source: https://docs.gearset.com/en/articles/12550109-agentforce-changes-in-the-winter-26-release
- **[mechanic]** Agent lifecycle is explicitly Draft → Commit → Activate/Deactivate: drafts are working copies isolated from production, 'commit' finalizes a version for deployment, and channel connections (Lightning, Slack, email, voice) bind to a specific committed version rather than 'the agent' generically.  
  Source: https://help.salesforce.com/s/articleView?id=ai.agent_versions_lifecycle.htm&language=en_US&type=5
- **[mechanic]** AWS Bedrock Agents separate the agent definition (versions, numbered like a build) from an 'alias', which is a stable pointer with a routing configuration; UpdateAgentAlias repoints the alias to a new underlying agent version while the alias itself stays live and invokable, giving zero-downtime cutover — the alias is what client applications actually call (InvokeAgent), never the raw version.  
  Source: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_agent_UpdateAgentAlias.html
- **[mechanic]** Salesforce second-generation managed packages define per-component 'manageability' rules (e.g., 'subscriber editable') that determine whether the platform vendor's push upgrade is allowed to overwrite a given field/component or must leave subscriber customizations (like a customized page layout) untouched — this is the core mechanism for upgrading a shared template without clobbering tenant-specific overrides.  
  Source: https://trailhead.salesforce.com/content/learn/modules/second-generation-managed-packages/explore-subscriber-experience-of-packages
- **[mechanic]** Salesforce push upgrades let the publisher choose exactly which subscriber orgs receive an upgrade, to which version, and when — i.e., propagation to installed instances is publisher-initiated and selectively targeted, not automatic-for-all-on-save.  
  Source: https://developer.salesforce.com/docs/atlas.en-us.pkg2_dev.meta/pkg2_dev/push_upgrade_intro_2GP.htm
- **[pattern]** Zapier's public template gallery uses a fork, not a subscribe, model: templates are 'living resources' where the original creator's edits update the template listing, but every user who already clicked 'Use this template' gets an independently owned Zap copy that does NOT auto-update — they must re-import to get the new version, so Zapier explicitly recommends the publisher run their own versioning/notification process for shared templates.  
  Source: https://growwstacks.com/blog/how-to-share-zapier-zaps-template-guide
- **[mechanic]** n8n workflow templates never carry credential values in the exported JSON — every credential reference must be re-resolved against the target instance at import time (via explicit binding UI, name-matching, or empty placeholders the importer fills in), which is the concrete 'binding at install' pattern for the tool-credential slot of a template.  
  Source: https://docs.n8n.io/build/manage-workflows/n8n-packages/how-import-works
- **[pattern]** Custom GPTs decompose into four separable building blocks — system instructions (~8,000 char persona/behavior), Knowledge Files (RAG grounding, up to 20 files/512MB each), Actions (OpenAPI schema for external tool calls), and toggleable built-in capabilities (code interpreter, DALL-E, browsing) — mirroring the fixed-workflow-vs-bound-data split a template/instance model needs (instructions=fixed template logic, Knowledge/Actions=per-deployment bound data).  
  Source: https://blog.apify.com/custom-gpts-knowledge/

## vertical-packaging

- **[number]** Sierra prices per successful outcome only (roughly $1–2.50 per resolved conversation, unresolved/escalated conversations are typically free); for low-value interactions like routing it falls back to a flat per-conversation fee instead. No public rate card — negotiated per enterprise contract, with year-one costs (platform + implementation) estimated at $200k–$350k+.  
  Source: https://valueaddvc.com/blog/how-does-sierra-ai-make-money-outcome-based-pricing-enterprise-agents-and-the-business-model-breakdown
- **[number]** Intercom Fin bills $0.99 per billable 'outcome' (resolution, procedure handoff to a human, or disqualification), but $9.99 for a qualified-lead outcome — i.e. it price-discriminates by outcome value, not a flat per-resolution rate. Only one outcome is charged per conversation even if Fin performs multiple actions. Standalone 'Fin for platforms' deployment has a $49/month base fee bundling 50 resolutions, then $0.99/each beyond.  
  Source: https://www.gleap.io/blog/intercom-fin-ai-pricing-2026
- **[number]** Salesforce Agentforce launched at $2/conversation (a conversation = first message to resolution/escalation/24h inactivity timeout) in Fall 2024, but backlash over unpredictability forced a pivot: it now also offers 'Flex Credits' billed per discrete agent action ($0.005/credit; a standard 20-credit action ≈ $0.10, a 30-credit voice action ≈ $0.15) plus a flat per-user license starting at $125/user/month as an alternative to consumption pricing.  
  Source: https://www.getmonetizely.com/blogs/the-doomed-evolution-of-salesforces-agentforce-pricing
- **[number]** Paradox (Olivia), a frontline/high-volume recruiting agent (McDonald's, Chipotle, 7-Eleven, GM, CVS), starts at $1,000–1,500/month for basic conversational screening + scheduling, requires 12-month minimum contracts, and scales with locations/hiring volume/integration scope rather than per-seat; it was acquired by Workday in Oct 2025 and folded into that platform.  
  Source: https://www.hiretruffle.com/blog/paradox-ai-pricing
- **[mechanic]** Apna's AI Calling Agent (India frontline/blue-collar hiring) is embedded directly into every job post rather than sold as a separate configurable product: it auto-generates role-specific interview questions from the job post, conducts live multilingual (English/Hindi/regional) voice interviews at up to 10,000 simultaneous calls, scores responses in real time, and follows up via call/WhatsApp/email — achieving 80%+ candidate connect rate vs an industry average of ~30%.  
  Source: https://mediabrief.com/apna-co-launches-indias-first-job-posting-integrated-multilingual-ai-calling-agent-cutting-hiring-time-by-50/
- **[mechanic]** Fountain (frontline hiring OS used by UPS, Amazon DSP, Sweetgreen; 1.2M hires/year) publishes no rate card for its core enterprise product — fully custom quoted — but does sell industry-oriented configurations (retail, manufacturing, logistics, healthcare, hospitality) and an AI orchestration layer called 'Cue' that lets managers automate hiring workflow steps via natural-language commands rather than pre-set templates alone.  
  Source: https://www.fountain.com/posts/high-volume-recruiting-tools
- **[number]** 11x sells its AI SDR as a fixed-price 'digital worker with a quota' ($36k–68k/year, annual contract, ~$5k/month, guided onboarding, no self-serve) — explicitly positioned against seat-based and pure-usage pricing as more budget-predictable; Artisan by contrast uses usage-based credits with no platform fee (plus a $67/seat/month add-on for its dialer), moving toward quote-scoped enterprise pricing ($280–$5,000/month reported range).  
  Source: https://www.11x.ai/guides/artisan-vs-11x
- **[pattern]** Salesforce packages vertical agent capability as a 'skills library,' not a single template: Agentforce for Retail ships pre-built agent skills for order management, guided shopping, appointment scheduling, and loyalty-promotion creation; parallel libraries exist for Manufacturing (prebuilt role-based agent templates) and Consumer Goods (prebuilt templates + data connectors) — each addressable industry gets its own composable skill set rather than one generic agent reskinned.  
  Source: https://www.cxtoday.com/crm/agentforce-goes-industry-specific-as-salesforce-releases-an-agent-skills-library-for-retailers/
- **[requirement]** Sierra's real-world time-to-value: standard onboarding is sales-led and CSM-guided, taking 4–10 weeks for initial deployment (some enterprise rollouts 3–7 months to full production) via a Learn More form → demo → discovery session → scoped pilot → 90-day onboarding with dedicated support; in March 2026 Sierra launched 'Ghostwriter,' a conversational builder where a business user describes the agent in plain English and points it at existing docs/materials, and the AI extracts the logic and generates a deployable agent — aimed at collapsing that weeks-long build cycle.  
  Source: https://sierra.ai/blog/fast-to-build-faster-to-impact