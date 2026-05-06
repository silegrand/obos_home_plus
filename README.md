# OBOS Home+

A voice-first resident wellbeing and home reflection tool, built for deployment under partnership with OBOS, Scandinavia's largest member-owned housing cooperative. Developed by Assistiv Systems Ltd.

---

## What this is

OBOS Home+ is a twenty-minute, voluntary, voice-first conversation that gives OBOS residents a private space to reflect on how their home is working for them — and, at their choice, to produce two personal reports and route them to the right people.

It is not a care product, a diagnostic system, or a monitoring tool. It is a housing-experience instrument: a structured, consent-gated conversation that helps residents stay in their homes longer by surfacing friction early, on their own terms.

The tool is described internally as the **Warm Porch** — the place where a neighbour might stop to ask how you are, without being invited in, and without the conversation being a transaction.

The working specification for the product is `OBOS_Care_Tool_V17.docx`. The single-file application is `OBOS_Home_Plus_V17.html`.

---

## Quick start

The application is a single self-contained HTML file. No build step. No dependencies to install.

**Deploy via GitHub Pages (recommended):**

1. Go to your repository Settings → Pages
2. Set source to your main branch, root directory
3. Save — GitHub will provide a URL: `https://yourusername.github.io/repo-name/OBOS_Home_Plus_V17.html`

Share that URL with anyone accessing the demo or pilot. It opens directly in the browser — no setup, no downloads.

> Chrome is required for voice input (Web Speech API). The application works without voice in any modern browser, but voice is a first-class input and the experience is designed around it.

**You will need an Anthropic API key.** The key is entered on the first screen, used only in your browser tab, and never stored or transmitted anywhere other than directly to the Anthropic API. When you close the tab, everything is gone.

Each person running a session will need access to an API key. A full Level 2 session at high engagement costs approximately four to five dollars in Opus API usage. This is deliberate — the specification requires the current best Opus model and the build is never downgraded for cost (§10.2).

> Note for repo visibility: GitHub Pages works on public repositories without a paid plan. For a private repo, GitHub Pro or above is required for Pages hosting.

---

## The conversation

Twenty-two questions across four sections. The section order reflects the findings of a Norwegian cultural researcher: starting with home and surroundings earns the trust needed to ask personal questions. Leading with health or wellbeing questions triggers polite disengagement in Norwegian users within sixty seconds.

| Section | Name | Questions | Feeds |
|---------|------|-----------|-------|
| 1 | About Your Home | 4 | Your Home Report |
| — | WILL/WILL NOT screen | — | — |
| 2 | About You | 5 | Your Wellness Report |
| 3 | How Things Are Going | 6 | Your Wellness Report |
| 4 | What Matters About the Future | 4 | Your Wellness Report |

Section 3 contains three scale questions (three-point, with optional say-more): Staying Steady (physical), Staying Sharp (cognitive), and Staying Safe (independence at home).

---

## Architecture

### The Iron Curtain

The single most important architectural commitment. No data ever crosses between the Your Wellness Report and the Your Home Report, with one bounded exception (the inferred-link supplementary question).

This is enforced at the API call level, not at the prompt level. Prompt instructions are not enforcement; context scoping is enforcement.

In practice:

- `homeAnswers` and `wellnessAnswers` are stored as separate state objects in Alpine.js
- The Wellness Report generator receives only `wellnessAnswers`
- The Home Report generator receives only `homeAnswers`
- The GP Summary and Community Note receive only Wellness Report content
- The Home Note receives only Home Report content, plus the linkage statement if the resident has approved it
- No API call in the architecture has access to both bodies of content simultaneously

This protects against the specific failure mode of an OBOS Housing Officer — who is not a clinician and not bound by clinical confidentiality — learning anything about a resident's health, mood, cognition, or personal life.

### The four parallel classifiers

Every answered question triggers four concurrent Anthropic API calls:

1. **Reflection** — generates a warm, MI-aligned response to what was shared
2. **Crisis classifier** — fires on explicit, present-tense, self-directed expressions of risk
3. **Safeguarding classifier** — fires on explicit disclosures of abuse, coercion, or harm from another person
4. **Hostility classifier** — fires on clear withdrawal of consent to continue

Strict precedence: crisis beats safeguarding beats hostility beats reflection. Each classifier fails closed — if the API call errors, the safest classification is assumed.

### The WILL/WILL NOT screen

Positioned between Section 1 and Section 2. The resident has already engaged with home-territory questions; the tool has already listened. Before the conversation moves into personal territory, the screen makes an explicit promise about what will and will not happen with what is shared.

This is a deliberate departure from the earlier CAN/CANNOT framing. "Cannot" implies a technical limitation. "Will not" implies an ethical commitment. For a Norwegian resident weighing whether to trust the tool with anything personal, that distinction matters.

### The inferred-link supplementary question

After all four sections complete, the tool runs a candidate-link detection call. If the conversation contains both a concrete home issue (Section 1) and a specific wellbeing observation (Sections 2-4) that plausibly connect, the resident is offered a single MI-shaped supplementary question: *Would you say the [home issue] is affecting how you feel or how you manage day to day?*

If the resident answers yes, a fixed-grammar linkage statement is generated and shown to the resident for explicit consent before anything is sent. The statement can only say: *[Name] has reported [home issue] and has indicated this is affecting their [wellbeing / day-to-day life / comfort at home].* It cannot expand beyond that grammar. The Housing Officer learns that the resident has a problem with the lift and that the resident says it is affecting them. The Housing Officer learns nothing about the resident's clinical or psychological life.

Three separate consents are required before this statement reaches anyone.

### The Consented Click routing architecture

After both reports are shown, the resident is offered up to three routing options. Each is an offer, not a default. None pre-fill. None chain.

- **Share Wellness Report with GP or kommune** — generates a GP Summary from Wellness Report content only
- **Share Home Report with OBOS Housing Officer** — generates a Home Note from Home Report content only
- **Share with Community Captain or Social Worker** — generates a Connection Note from Wellness Report content only

Pressing none of these and printing the reports is always an equally valid outcome.

### State management

Built on Alpine.js (CDN, no build step). All state lives in the browser tab. When the tab closes, everything is destroyed. There is no database, no server, no transcript store.

```
screen          → current UI screen (17 states)
homeAnswers     → Section 1 answers only (Iron Curtain)
wellnessAnswers → Sections 2-4 answers only (Iron Curtain)
qIndex          → current question (0-21)
safetyEventFired → suppresses Sovereign Pulse if true
```

---

## Model

All API calls use the current best Opus model from Anthropic. The model constant is defined at the top of the script block:

```javascript
const MODEL = 'claude-opus-4-6';
```

Update this string when Anthropic releases a newer Opus model. The build is never downgraded for cost — this is a hardline rule in the specification (§10.2).

---

## What is implemented in V17

- All 22 questions across 4 sections, with correct types (open, scale, scale + say-more)
- Full section reorder: home first, wellness earned
- WILL/WILL NOT screen as trust-earning hinge
- All four parallel classifiers on every answer
- Crisis screen with UK/Norway/Sweden resources
- Safeguarding screen with elder-abuse resources
- Sovereignty Pause (hostility detected)
- Skip mechanism with 3-skip note and 6-skip off-ramp
- Voice input via Web Speech API with 2.5-second silence auto-stop
- Iron Curtain enforcement at API context level throughout
- Candidate-link detection and inferred-link supplementary question
- Linkage statement generation and three-screen consent flow
- Wellness Report generation (Section 2-4 content only)
- Home Report generation (Section 1 content only)
- Both reports displayed with tab toggle and print
- GP Summary generation (Wellness content only)
- Home Note generation (Home content only, linkage statement if approved)
- Community Note generation (Wellness content only)
- Consented Click routing UI
- Print functionality for reports and routed notes
- Clean early-exit at any screen with partial report generation

---

## What is deferred

These features are specified in `OBOS_Care_Tool_V17.docx` and require backend endpoints to implement.

- **Vinter-Trygg** (§6.11) — Yr.no weather API integration for seasonal home check-in prompting
- **Sovereign Pulse** (§7.9) — zero-content proof-of-life signal to a family contact; requires a thin relay endpoint
- **Styreleder Resilience Dashboard** (§8.2) — anonymised, k-anonymised building-level ESG signal; requires a write-only counter endpoint
- **ESG counter** (§8.1) — the anonymised aggregate that feeds the dashboard
- **Asset-Based Offer screen** (§7.4) — requires asset detection logic and a post-report offer screen; foundation is in place
- **Direct maintenance ticketing integration** (§11.2) — OBOS internal system integration
- **Norwegian Bokmål locale** (§6.9) — voice input lang and crisis/safeguarding resource strings

---

## Files

| File | Description |
|------|-------------|
| `OBOS_Home_Plus_V17.html` | The application — single file, no build step |
| `OBOS_Care_Tool_V17.docx` | Working specification — all design decisions, architectural commitments, hardline rules |
| `README.md` | This file |

---

## Version history

| Version | Date | Summary |
|---------|------|---------|
| V17 | 6 May 2026 | Full section reorder (home first); WILL/WILL NOT replaces CAN/CANNOT; both reports generated at end; two-layer domain naming formalised. First software build. |
| V16.1 | 27 April 2026 | Product renamed OBOS Home+; commercial-value sentence added; concrete resident scenario added |
| V16 | 27 April 2026 | Two-report architecture; Iron Curtain; inferred-link supplementary question; Opus 4.7 hardline rule; Norwegian GDPR architectural-by-design; Section 4 redesigned as OBOS home-and-environs questions |
| V13 | — | Asset-Based Naboskap; Vinter-Trygg; Styreleder Resilience Dashboard; Sovereign Pulse |
| V12 | — | OBOS pivot; Maintenance layer; Naboskap layer; Consented Click routing |
| V10 | — | Canonical build reference; 18-question structure; single Staying Well Plan |

Full version history and design rationale in `OBOS_Care_Tool_V17.docx` §12.

---

## Principles that are not negotiable

These are hardline rules in the specification. They are recorded here so that anyone working on this codebase knows they are not implementation preferences.

**The Iron Curtain must never be undone.** No data crosses between the Wellness Report and the Home Report except through the bounded, consent-gated inferred-link exception. Removing the API-level context scoping to simplify a refactor is not an option. Prompt instructions are not enforcement; context scoping is enforcement.

**The model must never be downgraded for cost.** Every API call runs on the current best Opus model. The build is upgraded as Anthropic releases better models. It is never downgraded.

**A member who says no is not a failed user.** The Sovereignty Pivot is structural. Every decline — of any feature, at any point — is a successful outcome. No decline triggers a follow-up, a flag, or a note in any system.

**The tool is not a care product.** It cannot be marketed, configured, or operated as a diagnostic system, a monitoring system, a medical device, or a triage instrument. A feature request that makes it any of these is rejected before scoping.

**Nothing is sent to anyone without a resident's positive press of a button.** There is no auto-route, no escalation by silence, no override by concern.

---

## About

OBOS Home+ is built by **Assistiv Systems Ltd**, a UK health technology company focused on the Missing Middle — older adults who are too at-risk to be fully safe at home but not yet eligible for formal NHS or social care support.

The OBOS Home+ tool is the onboarding layer of the wider Assistiv platform, deployed in partnership with OBOS for the Norwegian and Swedish market. The wider platform (passive mmWave sensing, FHIR-aligned care wallet, Triple Tap Logic) is outside the scope of this repository.

---

*Specification: OBOS_Care_Tool_V17.docx · Assistiv Systems Ltd · 6 May 2026*
