---
name: competitor-analysis
description: Guides a user through analyzing market competitors, ingesting the business-model-canvas output for value proposition and target market context. Profiles 3–5 competitors, builds a comparison matrix, and outputs a summary SWOT analysis in Markdown and HTML.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - business-model-canvas
argument-hint: "[path/to/bmc-markdown.md]  # optional; defaults to docs/business-model-canvas/"
user-invocable: true
---
Cross-skill integration: load BMC context before prompting. NEVER re-ask what the BMC already provided. NEVER fabricate competitor data — unevidenced fields stay `Unknown — requires verification`.

1. PHASE 0 (Load BMC Context): Resolve BMC output from `docs/business-model-canvas/` or provided path. Extract **Value Propositions** and **Customer Segments**. If no BMC output found: report `BMC output not found — run business-model-canvas first` and halt. Display extracted context as baseline.

2. PHASE 1 (Identify Competitors): Ask the user to list 3–5 key competitors (name + website). If a web-search tool is available, offer to discover competitors automatically using the BMC Value Propositions as search context. If no tool available or user declines, rely on user-provided list. Store as `competitors[]` array.

3. PHASE 2 (Profile Competitors): For each competitor, elicit three fields one competitor at a time. Prompt: "Tell me about {competitor}." Support `/skip` to leave a field empty. Do not re-prompt once answered.
   - **Strengths** — What do they excel at? Core unfair advantages?
   - **Weaknesses** — Gaps in their offering, poor reviews, technical limitations?
   - **Pricing Model** — How do they monetize? (freemium, flat subscription, enterprise-only, etc.)

4. PHASE 3 (Comparison Matrix): Elicit 3–5 comparison dimensions from the user (e.g., "price", "features", "ease of use", "support"). For each dimension, ask the user to rate each competitor and the user's own product on a simple scale (Low/Medium/High or 1–5). Reference the BMC Value Propositions as the baseline for the user's own product scores.

5. PHASE 4 (Render): Compile the completed analysis into two formats:
   - **Markdown** — structured document with: BMC Baseline preamble, Competitor Profiles (Strengths/Weaknesses/Pricing per competitor), Comparison Matrix table, SWOT summary.
   - **HTML** — self-contained page with inline styles: profiles section, comparison table with alternating row colors, SWOT grid. Tag `[Scope: CA]`.

6. PHASE 5 (Output): Present both outputs. Offer to write to `docs/competitor-analysis/`. Do not write without explicit confirmation. Chat-only fallback: display inline.

Directives:
- BMC-First: Always establish baseline before the first competitor prompt.
- No Redundancy: Never ask the user to re-state content already captured in the BMC. Reference it.
- Competitor-at-a-Time: Profile exactly one competitor per interaction. Never ask about multiple competitors in a single message.
- Web Search Optional: Attempt web search if tool available; user always has final say on competitor list. Never fabricate search results.
- Anti-Fabrication: Never invent competitor strengths, weaknesses, or pricing. Unevidenced = `Unknown — requires verification`.
- Empty Cells: Unanswered fields render as `—` (em dash) in output tables.
- Comparison Dimensions: User-defined, not hardcoded. Default if user skips: `Features`, `Pricing`, `Ease of Use`, `Support`, `Market Reach`.

Schema `[Scope: CA]`:

```
Input Context (from BMC):
  bmc.value-propositions  <string>
  bmc.customer-segments   <string>

State (in-memory):
  competitors[]: [
    name          <string>
    website       <string>
    strengths     <string>
    weaknesses    <string>
    pricing-model <string>
  ]
  comparison-dimensions[]: [<string>]
  comparison-scores[]: [
    dimension  <string>
    self-score <string>
    competitor-scores: {<name>: <string>}
  ]
  current-phase  <int 0-5>
  current-index  <int>
```

```
Markdown Output:

# Competitor Analysis

## BMC Baseline
### Value Propositions
{bmc content}

### Customer Segments
{bmc content}

## Competitor Profiles
### {Competitor 1}
- **Strengths:** {content}
- **Weaknesses:** {content}
- **Pricing Model:** {content}

### {Competitor 2}
...

## Comparison Matrix
| Dimension | Self | {Comp 1} | {Comp 2} | ... |
| :-- | :-- | :-- | :-- | :-- |
| {dim 1} | {score} | {score} | {score} | ... |
...

## SWOT Summary
### Strengths
- {derived from comparison}

### Weaknesses
- {derived from comparison}

### Opportunities
- {derived from gaps}

### Threats
- {derived from competitor strengths}
```

```
HTML Output:
Self-contained <!DOCTYPE html> with inline styles.
Sections: BMC Baseline (muted preamble), Competitor Profiles (cards per competitor with Strengths/Weaknesses/Pricing),
Comparison Matrix (HTML table with alternating row colors, header row for Self + competitors),
SWOT Summary (2×2 grid layout).
Empty cells show "—".
Tagged [Scope: CA].
```