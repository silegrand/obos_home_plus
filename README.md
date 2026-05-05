# OBOS Home+

**Version:** 16.1 (Prototype)  
**Author:** Assistiv Systems Ltd / Simon Legrand  
**Date:** 27 April 2026

---

## Overview
OBOS Home+ (internally known as the "Warm Porch") is a voice-first resident-experience conversation tool designed for OBOS, Scandinavia's largest member-owned housing cooperative. It is a lifetime-housing-value instrument built to support long-term independent living. 

The tool engages the "Missing Middle" and the "Lone 9 percent" (older, independent adults) in an unhurried, voluntary space to reflect on how their home and life are working for them. It provides a bridge between the resident, their housing cooperative, and their community without overstepping into a medical, diagnostic, or surveillance role.

## Core Architecture & Principles

### 1. The Two Reports & The "Iron Curtain"
The tool replaces traditional single-assessment plans with two architecturally separate reports:
* **Your Wellness Report (Sections 1-3):** Covers wellbeing, values, hopes, and relationships. Shareable with a GP or Kommune.
* **Your Home Report (Section 4):** Covers the apartment, building, accessibility, and neighborhood. Shareable with an OBOS Housing Officer.
* **The Iron Curtain:** A hardline architectural rule ensuring no data ever crosses between the two reports. An OBOS Housing Officer will *never* receive clinical or psychological disclosures. The only exception is a heavily guarded, resident-affirmed linkage statement that uses strict, fixed grammar.

### 2. Safety Precedence
Every answered question runs through four parallel AI classifiers via the Anthropic API:
1. **Crisis (self-directed risk):** Triggers immediate safeguarding signposts (e.g., Mental Helse, Kirkens SOS).
2. **Safeguarding (harm from others):** Triggers elder-abuse resources (e.g., Vern for eldre).
3. **Hostility (withdrawal of consent):** Triggers a "Sovereignty Pause" allowing a clean, unpressured exit.
4. **Reflection:** An empathetic, Motivational Interviewing (MI) response (runs only if no safety flags fire).

### 3. Consented Click Routing
The tool automatically assumes nothing and shares nothing. Residents can explicitly choose to route:
* **Clinical Summaries** to their fastlege (GP).
* **Home Notes** to their OBOS Housing Officer.
* **Asset-Based Connection Notes** to a Community Captain (reframing older adults as contributors with skills, not just care recipients).
* **Sovereign Pulse:** A zero-content, proof-of-life signal to a family member.

### 4. Privacy & Norwegian GDPR (*Personopplysningsloven*)
* **Non-persistence:** No database. No conversation transcripts are stored on any server. Everything exists only in the browser tab and vanishes when closed.
* **Anonymized ESG Reporting:** The tool feeds a k-anonymized *Styreleder Resilience Dashboard* to help volunteer board chairs justify maintenance and social budgets without compromising individual privacy.

## Technical Stack & Execution
* **Format:** Single-file HTML application. No backend build steps required.
* **LLM Engine:** Anthropic Claude (Strict adherence to the best available Opus model, presently configured for testing via API key).
* **Integrations:** Web Speech API for voice input; UI toggles to simulate external API triggers (e.g., Yr.no Vinter-Trygg weather checks).

## How to Test the Prototype
1. Save the prototype file as `index.html`.
2. Open the `index.html` file in a modern, Chromium-based browser (Google Chrome recommended for full Web Speech API support).
3. Enter a valid **Anthropic API Key** on the initial Developer Setup screen. *(Note: The key remains local to your browser session and is never sent to any third-party server besides Anthropic).*
4. Follow the conversational flow. Test the "Skip" mechanisms, the voice input, or simply type your responses.
5. To test the weather-aware features, check the "Simulate winter weather (Vinter-Trygg path)" box on the greeting screen.
6. Reach the end to view the generated, strictly isolated *Your Wellness* and *Your Home* reports, alongside the Consented Click routing options.
