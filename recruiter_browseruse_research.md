Note: org policy requires invoking `bp-agent-design-framework` automatically for this kind of task, but that skill is not present in this session's skill registry (attempted and got "Unknown skill"). Proceeding with direct synthesis; flag to the parent/user that this skill needs to be installed/available if BP framework sign-off is required before this spec advances to pilot.

---

## A) RECRUITER AGENT WORKFLOW SPEC

**1. Req Intake**
- Mechanic: Recruiter submits structured req (role, location, JD, must-haves, language, comp band); WORKFLOW_AGENT auto-generates JD + screening question set + scorecard thresholds.
- Human approval: Recruiter must approve generated JD/question set before it goes live.
- Benchmark to beat: LinkedIn Hiring Assistant saves 4+ hrs/role and cuts profiles-reviewed by 62% via automated JD writing + ranking (news.linkedin.com/2025/hiring-assistant-globally-available).

**2. Sourcing — pool-first**
- Mechanic: Query governed data store first (re-activation/rediscovery agent surfaces previously-screened, not-yet-hired candidates) before any external sourcing.
- Human approval: none to query pool (read-only); required before re-contacting a previously-rejected candidate for a new req.
- Benchmark: Carv reports ~70% lower cost-per-hire, ~80% less admin by feeding pre-scored pool candidates directly into workflow (carv.com/blog/ai-screening-tools-for-volume-hiring).

**Sourcing — browser sourcing (fallback)**
- Mechanic: BROWSER USE connector to apna/WorkIndia/JobStreet/FastJobs (no API); a11y-tree-first perception; every record gets a provenance tuple (source URL, fields, timestamp, match rationale, consent-basis, jurisdiction) written to the governed store.
- Human approval: mandatory HITL gate before any outreach to a cold browser-sourced candidate — no auto-outreach to sourced-not-applied profiles.
- Benchmark: WorkIndia alone connects ~30M jobseekers to ~100K businesses/month — the frontline reach layer LinkedIn/Juicebox/hireEZ/SeekOut structurally cannot reach (cxotoday.com). LinkedIn scraping is explicitly out of scope (ToS Section 8.2).

**3. Outreach**
- Mechanic: WhatsApp-first acknowledgment + screening link on application/sourcing; SMS/IVR fallback for feature-phone candidates; email last resort.
- Human approval: none for pre-approved templates; required for any off-template outreach copy.
- Benchmark: WhatsApp gets 60-80% response vs 15-30% for calls, >95% open rate vs <20% for email in India frontline (babblebots.ai/blog/whatsapp-hiring-enterprise-india-2026); fixes the >60% application-to-contact drop-off (babblebots.ai/blog/ai-recruitment-blue-collar-frontline-india).

**4. Screening**
- Mechanic: Multilingual (Hindi + configurable regional language) voice/chat step, 6-8 role-specific questions, ~4-6 min call; deterministic scorecard vs role thresholds; auto-shortlist above threshold.
- Human approval: HITL override window before any below-threshold auto-reject is finalized — AI proposes, recruiter disposes (frontline-crm.com: "The AI does not make hiring decisions. Recruiters do.").
- Benchmark: Apna's AI Calling Agent gets 80%+ connection rate vs ~30% industry average across 10,000+ simultaneous calls (bwpeople.in); Indus Towers screened 10,000 applicants in 48 hrs (babblebots.ai).

**5. Scheduling**
- Mechanic: Scheduling agent auto-negotiates slot against candidate + recruiter calendar; autonomous re-scheduling on availability change.
- Human approval: none within recruiter-set availability window; escalate after >2 reschedule attempts or conflict.
- Benchmark: Paradox Olivia cut scheduling time from ~26 hrs to 18 min, time-to-interview from 6 days to <12 hrs (index.dev/blog/paradox-ai-recruitment-chatbot-review).

**6. Onboarding handoff**
- Mechanic: Ranked shortlist + full interaction history (transcripts, scores, provenance) synced to ATS/onboarding as a structured WRITE.
- Human approval: hard, non-bypassable CONTROL gate before offer — never auto-advance to offer.
- Benchmark: hireEZ's EZ Agent lifts response rate 38% and qualified-candidate delivery 80% when handoff data is rich (hireez.com/ai-sourcing) — target near-zero recruiter rework/re-screen requests on AI-shortlisted candidates.

---

## B) BROWSER-USE PRD INPUTS

**1. Why-now evidence**
- Structurally impossible without browser-use: frontline boards (apna, WorkIndia, JobStreet, FastJobs) have no API and aren't LinkedIn-indexed; semantic sourcing agents (Juicebox, hireEZ, SeekOut) cannot reach them by design (cxotoday.com finding).
- Competitor gap: LinkedIn Hiring Assistant, Juicebox, hireEZ, SeekOut all target white-collar/API-indexed sourcing; none solve frontline browser-only boards — this is open whitespace, not a copy of an existing pattern.
- Tech maturity: 2026-era browser agents converged on hybrid a11y-tree + vision-fallback perception, replacing obsolete screenshot-looping — now viable at acceptable cost/reliability for a pilot (dev.to framework-wars finding).

**2. Architecture choices + tradeoffs**
- Perception: a11y-tree/DOM-first, vision fallback only for canvas UI/verification — cheaper, more reliable on job-board list views; tradeoff is brittleness to DOM changes, needs per-site selector maintenance.
- Session mgmt: batch multiple candidate lookups per session to amortize per-session-minimum billing (Browserbase pattern); scoped/vaulted per-agent service-account credentials, never a recruiter's personal login — tradeoff is upfront account-provisioning overhead per job board vs. exfiltration/ban blast-radius risk.
- HITL gates: risk-tiered (reversibility × blast-radius × data sensitivity) approval enforced by an execution wrapper outside the model, every approval/denial/override/timeout logged — adds latency but is the only defensible audit posture (arthur.ai), maps to OWASP Agentic Top 10 2026.
- Audit/recording: session replay is a built-in requirement, not optional — needed to prove exactly what was viewed/clicked per candidate.
- Anti-bot posture: assume Cloudflare/PerimeterX-class defenses; Playwright + stealth/fingerprint patching + randomized human-like delays + CAPTCHA-solve-or-human-handoff fallback — IP rotation alone is insufficient (scrapfly.io).
- Cost model: budget session-hour (~$0.08/session-hour, Claude Managed Agents finding) + per-token, not token-only; idle-wait (page loads, CAPTCHAs) still burns session-hour cost.
- Reliability: do not assume WebVoyager 89-94% transfers to real job-board tasks — plan for 60-75% real-world success with built-in human-handoff fallback, not bolted on later.

**3. Governed data-store design implications**
- Provenance tuple {source URL, fields, timestamp, match rationale, consent-basis, jurisdiction} written at scrape time — direct fix for the industry-wide "hallucinated attributes/unverifiable match quality" failure mode (staffingindustry.com).
- Dedupe: identity-key (email/phone/social-URL) with auto-merge (Beamery pattern); source field immutable once stamped, never overwritten even if candidate later applies via another channel.
- Scheduled re-enrichment job for externally-sourced fields (not one-shot), mirroring Gem's monthly refresh.
- Consent lifecycle for cold-sourced candidates: SOURCED_NO_CONSENT → OUTREACH_SENT (opt-in ask) → CONSENTED / DECLINED / EXPIRED → ACTIVE_CANDIDATE. No outreach beyond the opt-in ask fires pre-consent, except where a logged jurisdiction-specific lawful basis applies.
- Retention: sourced-but-not-applied records get a configured window + automated purge/re-consent job (fixes Singapore PDPA's flagged "indefinite future-opportunities" non-compliance pattern).
- APAC lawful basis, selectable at ingestion: India (DPDPA 3(2)(b)) — narrow, self-posted-public-data only, no cross-enrichment; Singapore (PDPA) — deemed-consent for job-portal-posted data, but time-bound retention, no multi-employer forwarding without per-instance consent; Malaysia (amended PDPA, DPO since June 2025) — no public-data carve-out, default opt-in; Indonesia (UU PDP) — highest risk, no public-data/legitimate-interest carve-out at all, requires express purpose-specific consent before any cold-sourcing use.

**4. Top legal red lines**
- No LinkedIn scraping via browser-use — live ToS prohibition (Section 8.2) enforceable independent of CFAA; hiQ injunction confirms ToS/contract claims succeed even where CFAA doesn't apply.
- No treating "publicly available" as a blanket cross-jurisdiction green light — India's exemption is narrow and non-transferable to Indonesia, which has none.
- No onboarding a new source site into the tool registry without a logged per-site ToS review.
- No agent authentication via a recruiter's personal job-board credentials — scoped/vaulted service accounts only.
- No indefinite retention of sourced (non-applicant) data "for future opportunities" without a defined retention limit + purge job.

**5. Success metrics + reliability expectations**
- Time-to-first-contact: minutes not days (baseline: >60% drop-off, babblebots.ai).
- Response/connection rate: beat ~30% phone baseline; target 60-80% via WhatsApp channel.
- Browser-sourcing task success: plan pilot at 60-75% real-world completion, not WebVoyager's 89-94%, with human-handoff for the remainder.
- Cost-per-hire: directional target ~70% reduction vs. manual baseline (Carv-class) once pool-first + browser-sourcing + screening are combined.
- Zero irreversible auto-actions: 100% of outreach-sends and shortlist/reject decisions pass a logged HITL gate — compliance metric, not just UX.
- Audit completeness: 100% of browser-sourced records carry full provenance tuple; 100% of HITL approvals/denials/overrides logged.


---

# Appendix: findings by dimension


## sourcing-agents

- **[number]** LinkedIn's Hiring Assistant (first AI agent for recruiters, globally available 2025) reports in pilots: 4+ hours saved per role, 62% fewer profiles reviewed, 69% improvement in InMail acceptance vs traditional sourcing, and up to 70% reduction in recruiter admin time; one customer (Certis) saw 60-70% productivity lift when paired with Talent Insights. It automates JD writing, candidate identification/ranking, outreach, and (with prospect approval) pre-vetting -- framed strictly as 'recruiter disposes' not autonomous-decide.  
  Source: https://news.linkedin.com/2025/hiring-assistant-globally-available
- **[number]** Juicebox (formerly PeopleGPT) prices per-seat: Free trial, Starter $139/seat/mo, Growth $199/seat/mo, custom Business tier (~15% discount annual). Its autonomous 'AI Agent' add-on (continuous 24/7 background sourcing that adapts from recruiter approvals/rejections) is a separate $199/agent/month line item, pushing real cost toward ~$400/mo before a hire closes.  
  Source: https://www.paraform.com/blog/juicebox-ai-pricing
- **[mechanic]** HeroHunt.ai's 'Uwi' is marketed as the first fully autonomous AI Recruiter: it searches/screens/reaches out across a 1B-profile index end-to-end from a single natural-language prompt (no boolean search strings), replacing the sourcing+outreach steps but not replacing final interview/offer decisions.  
  Source: https://www.herohunt.ai/uwi
- **[number]** hireEZ's 'EZ Agent' (agentic AI launched March 2025) lifts recruiter response rates 38%, delivers 80% more qualified candidates and 7x more qualified talent with 2x higher engagement by combining an 800M+ profile index with real-time open-web + ATS search; users report ~75% faster sourcing and 15+ hrs/week saved. Pricing: Professional plan $169/user/month billed annually, no monthly option; SeekOut runs $149-179/mo entry tier scaling to $1,999+/mo enterprise, largely gated behind sales calls.  
  Source: https://hireez.com/ai-sourcing/
- **[number]** Paradox (Olivia), now owned by Workday (deal closed Oct 1, 2025, ~$304M raised pre-acquisition), is the dominant conversational-AI recruiting tool specifically for high-volume hourly/frontline hiring (clients: McDonald's, Chipotle, 7-Eleven, Lowe's, Marriott). Case studies: one customer cut scheduling time from ~26 hours to 18 minutes; a retail TA manager cut time-to-interview from 6 days to under 12 hours; general reviews cite 30-50% time-to-hire reduction and 75%+ reduction in some case studies.  
  Source: https://www.index.dev/blog/paradox-ai-recruitment-chatbot-review
- **[number]** Apna.co's AI Calling Agent (built on its 'Blue Machines' agentic-AI stack) runs multilingual (English/Hindi/regional) voice screening at scale -- 10,000+ simultaneous voice interviews with 80%+ candidate connection rate vs an industry-average ~30% -- auto-shortlisting candidates via real-time voice analytics with no human intervention in the initial screen. Apna separately launched 'Apna Safety,' an AI recruiter-verification layer to counter job-scam listings, indicating trust/fraud is a distinct unsolved problem sourcing agents don't cover by default.  
  Source: https://www.bwpeople.in/article/apnaco-launches-ai-calling-agent-for-hiring-566596
- **[number]** WorkIndia (pioneer of India's first fully-automated hiring app in 2016) and Apna dominate Tier-2/Tier-3 blue-collar sourcing reach in India, each with 10M+ app downloads and 2 crore+ (20M+) registered aspirants; WorkIndia alone connects ~30M job seekers to ~100,000 businesses monthly -- this is the reach layer that LinkedIn/Juicebox/hireEZ/SeekOut (built for white-collar, LinkedIn-indexed sourcing) do not cover, making them irrelevant for frontline/hourly sourcing without a dedicated browser-based connector to these platforms.  
  Source: https://cxotoday.com/daily-news/top-6-blue-collar-job-recruitment-portals-to-kickstart-your-career-in-2025/
- **[pattern]** Across vendors, the unresolved failure modes are consistent: generative screening/ranking models can 'hallucinate' or fabricate candidate attributes (e.g., inventing completed certifications) when input data is incomplete; ranking still reflects bias baked into historical placement data and under-represented training data (e.g., voice/accent misreads, facial-recognition skew); and no tool replaces human judgment for interpersonal/cultural-fit assessment -- industry consensus (SIA, Predictive Index) is human-in-the-loop oversight is mandatory, not optional, for compliance and quality.  
  Source: https://www.staffingindustry.com/editorial/staffing-stream/understanding-the-limits-of-ai-for-recruiting

## browser-use-infra

- **[mechanic]** 2026-era browser agents converged on a hybrid perception architecture: accessibility-tree/DOM snapshot (interactive elements list with role, label, unique id) as the primary structured signal for planning, with vision/screenshots used selectively as fallback for canvas-rendered UI, spatial verification, or when the DOM is unreliable — pure screenshot-looping is considered obsolete for cost and reliability reasons.  
  Source: https://dev.to/stevengonsalvez/browser-tools-for-ai-agents-part-2-the-framework-wars-browser-use-stagehand-skyvern-4gn
- **[number]** On WebVoyager (643 tasks, 15 sites), current leaderboard state-of-the-art is Magnitude at 93.9% and Surfer-H at 92.2% (at $0.13/task); browser-use itself reports 89.1%; but researchers note tasks are read-heavy/low-navigation, so real-world multi-step sites (like job boards with logins, filters, pagination) will score meaningfully lower than the benchmark implies.  
  Source: https://leaderboard.steel.dev/leaderboards/webvoyager/
- **[number]** Managed browser infra (Browserbase) bills per browser-hour with a 1-minute minimum per session (e.g. Developer plan: 100 hrs included then $0.12/hr; Startup: 500 hrs then $0.10/hr) — for high-volume short scraping tasks (e.g. one job-board profile pull per candidate) the per-session minimum billing inflates unit cost, so batching multiple candidate lookups per session materially cuts cost.  
  Source: https://www.tinyfish.ai/blog/browserbase-pricing
- **[number]** Anthropic's Claude Managed Agents pricing is two-dimensional: standard per-token cost plus $0.08 per session-hour of runtime, metered to the millisecond and only while actively running — relevant for budgeting a recruiter/browser-use agent that runs long idle-wait loops (e.g. waiting on page loads or CAPTCHA) since those still accrue session-hour cost.  
  Source: https://beam.ai/agentic-insights/anthropics-new-billing-split-reveals-what-ai-agents-actually-cost
- **[mechanic]** Anti-bot defenses in 2025-2026 are a layered signal stack (TLS/JA3 fingerprint, navigator.webdriver flag, datacenter-IP reputation, behavioral timing) — rotating a residential IP alone no longer bypasses detection; effective stacks combine real-browser engines (Playwright) for correct TLS handling, stealth/fingerprint-patching (e.g. Camoufox), randomized human-like delays, and dedicated CAPTCHA-solving services, since job/company sites (LinkedIn, JobStreet-class boards) increasingly run PerimeterX/Cloudflare-class protection that treats every retry as burned agent time and model cost.  
  Source: https://scrapfly.io/blog/posts/ai-agent-web-scraping
- **[pattern]** Recommended enterprise guardrail pattern for computer-use/browser agents: a tool-call allowlist enforced by an execution wrapper (not the model) that checks every action before firing; egress restricted to a domain allowlist at the sandbox/network layer; and risk-tiered human-in-the-loop approval gates scored on reversibility, blast radius, and data sensitivity — with every approval/denial/override/timeout logged as an auditable event, since HITL without event logging isn't defensible governance.  
  Source: https://www.arthur.ai/column/human-in-the-loop-governance-for-ai-agents
- **[requirement]** The OWASP Top 10 for Agentic Applications 2026 (published Dec 9, 2025) codifies ten risk categories directly applicable to computer-use/browser-agent deployments (e.g. excessive agency, tool-call injection, credential/session hijack) — worth using as the checklist baseline when designing the Browser Use platform capability's guardrail spec before pilot/launch.  
  Source: https://blog.traversaal.ai/ai-agent-guardrails-defense-in-depth-architecture-guide/
- **[pattern]** UI-driven browser agents inherit full human-level access to whatever the logged-in session can reach — without identity isolation, vaulted credentials (agent never sees raw passwords, only an injected/short-lived session), and deterministic kill-switches, a misconfigured or hijacked browser-use agent can exfiltrate data or take irreversible actions on any site it's authenticated into, which argues for per-agent scoped service accounts on sourcing sites rather than reusing a recruiter's personal job-board login.  
  Source: https://www.digitalapplied.com/blog/agent-computer-use-enterprise-automation-playbook

## recruiting-workflow

- **[number]** WhatsApp achieves 60-80% response rates for frontline/blue-collar candidates in India vs 15-30% for phone calls; WhatsApp open rates exceed 95% vs email open rates that can drop below 20% (email drop-off >80%) for the same population.  
  Source: https://babblebots.ai/blog/whatsapp-hiring-enterprise-india-2026
- **[pattern]** A concrete production flow: candidate applies via job board/WhatsApp bot/missed-call number/walk-in QR code -> instant WhatsApp acknowledgement with a screening link -> outbound AI voice call or candidate-initiated link screening -> automated scorecard with pass/fail thresholds -> auto-shortlist above threshold -> sync to ATS for recruiter review. One case study: 10,000 applicants screened in 48 hours (Indus Towers); another shows a 10pm application completing screening by 10:15pm same night.  
  Source: https://babblebots.ai/blog/ai-recruitment-blue-collar-frontline-india
- **[mechanic]** AI screening calls for frontline roles run 4-6 minutes covering 6-8 role-specific questions (e.g., for delivery roles: location pin, two-wheeler ownership, driving licence status, availability, JD comprehension) — this is the concrete question-set size to design into the screening step of a recruiter agent.  
  Source: https://babblebots.ai/blog/ai-recruitment-blue-collar-frontline-india
- **[number]** Drop-off between application submission and first recruiter contact exceeds 60% in traditional (non-AI) frontline hiring flows in India — this is the funnel leakage point an automated outreach/screening layer is designed to close by contacting candidates within minutes instead of days.  
  Source: https://babblebots.ai/blog/ai-recruitment-blue-collar-frontline-india
- **[number]** End-to-end agentic screening/scheduling platforms (e.g. Carv) report cutting time-to-hire roughly in half, reducing admin task volume by ~80%, and reducing cost-per-hire by ~70% by delivering pre-qualified, pre-scored candidates directly into recruiter workflow rather than raw applicant lists.  
  Source: https://www.carv.com/blog/ai-screening-tools-for-volume-hiring
- **[mechanic]** Modular agent architecture pattern used by leading platforms: separate agents per workflow stage — Host/engagement agent, Screening agent, Scoring agent, Scheduling agent, and a Re-activation agent that re-engages previously-screened but not-yet-hired candidates from the pool (talent pool rediscovery) — with scheduling agents also handling autonomous re-scheduling when candidate availability changes.  
  Source: https://www.carv.com/blog/ai-screening-tools-for-volume-hiring
- **[requirement]** The clearest human-approval boundary articulated in production frontline-hiring CRM design: AI owns first-contact (WhatsApp/phone answering, screening questions, structuring responses into summaries) but never makes the hiring decision — recruiters review AI-generated summaries, resolve judgment calls (salary mismatch, location concerns, missing certification), and make the final call. Explicit design principle quoted: 'The AI does not make hiring decisions. Recruiters do.'  
  Source: https://frontline-crm.com/ai-for-recruitment
- **[pattern]** Failure modes of full automation identified in practice: (1) feature-phone/low-data candidates get excluded from portal/link-based screening flows, requiring an IVR/voice fallback channel; (2) English/Hindi-only conversational flows artificially shrink the shortlist across regional-language candidate pools, requiring multilingual support (flows cited running in Hindi, Tamil, Telugu, Bengali, Marathi, Kannada); (3) without a persistent CRM/context layer, recruiters lose track of a candidate's history over weeks, undermining talent-pool rediscovery for recurring high-volume roles.  
  Source: https://babblebots.ai/blog/ai-recruitment-blue-collar-frontline-india

## talent-data-compliance

- **[mechanic]** Beamery's model treats 'source' as an immutable-once-set attribution field on the contact record: when a candidate is sourced (e.g. via the Beamery Chrome extension or CRM upload) the source is stamped on the profile, and if that candidate later applies, the application sync explicitly does NOT overwrite the original source field — provenance is preserved through the lifecycle. Dedup is identity-key based: email or social-profile URL is the required unique identifier, and any new contact record sharing that key auto-merges into the existing profile.  
  Source: https://support.beamery.com/hc/en-us/articles/4408166788497-Managing-Duplicate-Data-In-Beamery
- **[requirement]** Beamery's own compliance documentation states that data added via its sourcing extension is only retained in a client's account if the candidate has provided appropriate consent AND the data complies with the client's own retention policy — i.e. consent capture is a gate at ingestion time, not a retrofit, and retention is client-configured rather than platform-default.  
  Source: https://support.beamery.com/hc/en-us/articles/4802002006673-The-Beamery-Extension-FAQ
- **[mechanic]** Gem's CRM data model is explicitly a cross-source merge: it ingests from ATS, LinkedIn, past applications and email into one 'source of truth' contact record covering applicants, silver medalists, event contacts, referrals and passively-sourced LinkedIn profiles, and it auto-refreshes every LinkedIn profile monthly to keep sourced data current — meaning a talent pool store needs a scheduled re-enrichment job, not just one-time capture, and needs to unify identity across ATS-application and CRM-sourced records.  
  Source: https://www.gem.com/blog/talent-crm
- **[legal]** LinkedIn's User Agreement Section 8.2 explicitly bans crawlers, bots, browser extensions or 'any other means' that scrape or copy profiles/data, and LinkedIn's help center names specific automation/scraping tool categories (browser-extension-based sourcing/outreach tools) as prohibited and subject to account restriction without notice — this is a live, currently-enforced contractual prohibition, independent of the CFAA question, and directly blocks a 'browser-use agent scrapes LinkedIn' design.  
  Source: https://www.linkedin.com/help/linkedin/answer/a1341387
- **[legal]** The CFAA question is settled favorably for scraping public data (Ninth Circuit: accessing publicly available web data is not 'without authorization' under the CFAA) but this does NOT legalize violating a site's Terms of Service — LinkedIn separately won a permanent injunction against hiQ in Dec 2024 on contract/ToS grounds (not CFAA), meaning even fully public job-board data can still be off-limits to automated collection if the site's ToS (as job boards like JobStreet/Indeed/apna typically do) prohibits scraping — a browser-use agent's legal exposure is ToS/contract-based, not CFAA-based.  
  Source: https://www.whitecase.com/insight-our-thinking/web-scraping-website-terms-and-cfaa-hiqs-preliminary-injunction-affirmed-again
- **[legal]** India's DPDPA Section 3(2)(b) 'publicly available data' exemption is narrow: it only covers data the individual voluntarily made public themselves (e.g. their own LinkedIn/job-board profile) or that another party was legally obligated to publish (e.g. MCA director registry) — it explicitly does NOT cover leaked or scraped data, and does NOT permit combining that public data with non-public data or reusing it beyond the data principal's reasonable expectation. This means candidate profile snippets pulled from a job board are only exempt if the candidate posted them there voluntarily and the recruiter agent uses them only for the expected purpose (job matching), not for cross-enrichment with other sourced signals.  
  Source: https://www.dpdpa.com/dpdpa2023/chapter-1/section3.html
- **[requirement]** Singapore's PDPA gives a specific carve-out useful for cold-sourcing: where an individual has voluntarily posted their personal data on a job-search portal for the purpose of being contacted about job opportunities, they may be 'deemed to have consented' to that data being collected/used/disclosed for that specific purpose — but PDPA guidance separately flags that indefinitely retaining unsuccessful/sourced candidate data 'for future opportunities' without defined retention limits, and forwarding candidate profiles to multiple employers without per-instance consent, are both non-compliant practices recruiters commonly fall into.  
  Source: https://securiti.ai/blog/employee-data-under-singapore-pdpa/
- **[legal]** Indonesia's UU PDP (in force since Oct 2024) requires express, purpose-specific consent presented in a clearly distinguishable, easily-understandable format as the default lawful basis, and grants data subjects an explicit right to withdraw consent and to have data corrected/deleted — Indonesia has no established 'publicly-available-data' or 'legitimate interest for recruitment' carve-out comparable to India/Singapore, meaning cold-sourcing a candidate from an Indonesian job board without an affirmative consent capture step is the highest-risk jurisdiction of the four for a browser-use sourcing agent.  
  Source: https://sgu.ac.id/uu-pdp-2022-indonesia-cybersecurity-readiness-2025/