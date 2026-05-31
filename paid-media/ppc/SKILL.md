---
name: ppc
description: >
  Use this skill whenever the user wants help with paid search, Google Ads, PPC campaigns, or
  performance marketing. Triggers include: planning a new Google Ads campaign, setting up Smart
  Bidding or Performance Max, writing ad copy and asset groups, optimizing existing campaigns,
  troubleshooting poor ROAS or lead quality, setting up conversion tracking, and any question
  about PPC strategy or Google Ads features. Also use for questions about AI Max, campaign
  structure, negative keywords, asset performance, High-Value Mode, Enhanced Conversions, or
  the 2026 agentic Google Ads approach. If the user mentions "Google Ads", "PPC", "paid search",
  "campaign", "ROAS", "CPA", "ad copy", "keywords", "Performance Max", or anything related to
  running ads, use this skill.
---

# PPC / Google Ads – 2026 Strategy Framework

## The Core Mental Model

Google Ads in 2026 runs on agentic, AI-driven systems. Your role has shifted from manual operator to **Strategic Director**: define the business objective, feed the algorithm pristine data, set guardrails, and let the machine optimize. Think of Google Ads as a highly capable intern — your job is to give it the right data, the rules of engagement, and a crystal-clear definition of success.

---

## Phase 1: Planning — Data & Guardrail Blueprint

Do this before touching campaign settings. Planning is now about preparing inputs and boundaries for Google's algorithms.

### 1. Establish Advanced Tracking First

Basic pixel tracking is insufficient. Set up:

- **Enhanced Conversions**: Pass hashed first-party data (email, phone) with conversion events so Google can match offline signals.
- **Data Manager API**: Feed real-time offline data into Google Ads — CRM milestones, qualified leads, actual closed revenue. This is what tells the algorithm what a *real* conversion looks like vs. a form fill.

Never launch a campaign without offline conversion data flowing in. The algorithm optimizes for whatever signal you give it; if you only give it form fills, it will find cheap form fills regardless of quality.

### 2. Define Brand Guidelines & AI Constraints

Google's generative tools produce ad copy on the fly. Before launch, input:

- Brand colors and fonts (for visual assets)
- Tone of voice guidelines
- **Off-limit phrases**: explicit list of words/claims the AI must never use (competitor names, unverifiable superlatives, regulated claims, etc.)

Without these guardrails, the AI will generate copy that is technically on-topic but potentially off-brand or legally risky.

### 3. Map Customer Intent Themes (Not Keywords)

Don't build a granular keyword spreadsheet. Instead, group strategy into broad **Intent Themes** — clusters of user problems/goals your product solves. Examples:
- "Small business looking for accounting software"
- "Enterprise team needs HR automation"

AI Max features will match your ads to queries beyond your literal keyword list anyway. Trying to control this at the keyword level is fighting the algorithm; steering it with intent themes is working with it.

---

## Phase 2: Creation — Building the AI Engine

### Campaign Structure by Objective

| Goal | Campaign Type |
|------|---------------|
| High-intent search traffic | **Search Campaign + AI Max** (replaced Dynamic Search Ads) |
| Cross-channel dominance (Search, YouTube, Shopping, Maps, Gmail) | **Performance Max (PMax)** |

You will typically run both: Search+AI Max for bottom-funnel capture, PMax for full-funnel growth.

### Smart Bidding — Start Here on Day One

Never launch with manual bidding. Choose value-based bidding from the start:

- **Maximize Conversions + Target CPA constraint** — for lead generation where all leads have similar value
- **Maximize Conversion Value + Target ROAS constraint** — for e-commerce or when lead value varies

Set the CPA/ROAS target based on historical data or a conservative estimate; you can tighten it as data accumulates. Starting too aggressive starves the algorithm of learning volume.

### Asset Groups — Multi-Modal Inputs

The ad unit is now an **Asset Group**, not individual ads. Google dynamically assembles the right combination for each user/context. Provide the maximum allowed inputs:

**Text:**
- Up to 15 headlines and 4 descriptions
- Write in varied tones (urgent, benefit-focused, question-based, social proof)
- Use natural language editing prompts inside the asset editor to refine: *"Make headline 3 sound more urgent"* or *"Rewrite description 2 to emphasize the free trial"*

**Visuals:**
- High-res images (multiple aspect ratios)
- 4K video — both vertical (9:16) and horizontal (16:9)
- If you lack video: use Google's integrated generative tools to create them from your existing images and copy

**Promotions & Commerce:**
- Upload discount codes and promotional offers → Google serves **Promotion Bundles** automatically
- Connect product feeds → enables **Native Checkout** via the Universal Commerce Protocol (users buy without leaving the Google surface)

---

## Phase 3: Optimization — Steering the Machine

Optimization is no longer bid adjustments and keyword mining. It is data analysis and input refinement.

### 1. Audit Asset Performance (Not Ad Performance)

The old A/B test between two ads is gone. Instead, audit at the asset level:

- Open **Responsive Search Ad Asset Details** — Google grades each headline and description individually
- **Immediately replace anything rated "Low"** — low-rated assets drag the entire asset group
- Use natural language prompts to give Google creative direction for replacements, don't just swap in a similar line

### 2. Police AI Max Reach Weekly

AI Max casts a wide net, especially within AI Overview / Conversational Discovery ad placements. Budget bleed from irrelevant matches is a real risk.

**Weekly routine:**
1. Check the **Search Terms Report** — look for irrelevant queries getting clicks
2. Apply **negative keywords** aggressively for clear mismatches
3. Use **Brand List Exclusions** to block competitor brand terms (prevents your budget from going to users searching for a competitor who won't convert)

### 3. Improve Lead Quality with High-Value Mode

Symptom: Google is hitting your CPA target but sales says the leads are cold/unqualified.

Cause: The algorithm found the cheapest path to a conversion signal — it optimized correctly for the wrong thing.

Fix: Switch PMax campaigns to **High-Value Mode**. This tells the algorithm to stop chasing volume and instead replicate only the customer profiles matching your highest-value CRM cohorts. You must have offline conversion data flowing in (see Phase 1) for this to work.

### 4. Run Asset Experiments Before Scaling

Before increasing budget, validate your creative direction with data:

- Use **Performance Max Asset Experiments** to split-test fundamentally different creative angles
- Example: "Emotional/Benefits" asset group vs. "Discounts/Features" asset group
- Measure incremental ROI — not just click-through rate — before committing budget to the winner

---

## Common Situations & What to Do

**"My campaign just launched and has no conversions"**
→ Check tracking first (Enhanced Conversions firing? Data Manager connected?). Then ensure the bid strategy isn't set too aggressively — the algorithm needs volume to learn. Consider temporarily using Maximize Conversions without a CPA cap for the first 2–4 weeks.

**"My CPA keeps rising"**
→ Check Search Terms Report for irrelevant spend. Audit for low-rated assets. Verify that the conversion signal is still clean (no duplicate conversions, no test traffic contaminating data).

**"Leads are cheap but terrible quality"**
→ Activate High-Value Mode + ensure CRM closed-revenue data is flowing via Data Manager API. The algorithm is optimizing for the signal you gave it; upgrade the signal.

**"I don't know what ad copy to write"**
→ Start with intent themes. For each theme, write headlines that address: the problem, the solution, the differentiator, social proof, and urgency. Use the asset editor's natural language prompts to generate variations. Supply all 15 headline slots — more inputs = better dynamic assembly.

**"Should I use PMax or Search?"**
→ Both, typically. Search+AI Max for people actively searching your product terms. PMax for everything else and for cross-channel reach. PMax alone can cannibalize brand traffic; run them together with brand exclusions on PMax.
