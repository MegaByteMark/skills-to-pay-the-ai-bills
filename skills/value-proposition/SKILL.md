---
name: value-proposition
description: Guides a user through the Value Proposition Canvas, ingesting the business-model-canvas output to establish Customer Profile and Value Map context, then compiles the completed canvas into Markdown and HTML. Only asks for detail beyond what the BMC already captured — never re-asks for redundant information.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - business-model-canvas
argument-hint: "[path/to/bmc-markdown.md]  # optional; defaults to docs/business-model-canvas/"
user-invocable: true
---
Cross-skill integration: load BMC context before prompting. NEVER re-ask what the BMC already provided. NEVER fabricate BMC content — if BMC output is absent, report gracefully and halt.

1. PHASE 0 (Load BMC Context): Resolve BMC output. Check `docs/business-model-canvas/` for markdown output file. If argument provided, use that path. Parse the document to extract **Customer Segments** and **Value Propositions** content. If no BMC output found: report `BMC output not found — run business-model-canvas first` and halt. Display the extracted BMC segments to the user as established baseline.

2. PHASE 1 (Elicit — Customer Profile): Walk through the 3 Customer Profile blocks in fixed order. Present each with a prompt tying it back to the BMC Customer Segments baseline. Wait for user content. Advance after receiving input. Support interactive commands: `/next`, `/back`, `/edit <block>`, `/done`, `/status`.
   - **Customer Jobs** — What functional, social, and emotional tasks are these customers trying to perform? (Context: BMC Customer Segments)
   - **Pains** — What negative emotions, undesired costs, and risks do they experience before, during, or after getting the job done?
   - **Gains** — What benefits do they expect, desire, or would be surprised by?

3. PHASE 2 (Elicit — Value Map): Walk through the 3 Value Map blocks in fixed order. Present each with a prompt tying it back to the BMC Value Propositions baseline. Support same interactive commands.
   - **Products & Services** — What does the business offer to help customers get the job done?
   - **Pain Relievers** — How do these offerings alleviate specific customer pains?
   - **Gain Creators** — How do these offerings create customer gains?

4. PHASE 3 (Render): Compile the Value Proposition Canvas into two formats:
   - **Markdown** — structured document with Customer Profile and Value Map sections, each block as a sub-heading. Include the imported BMC segments as a preamble.
   - **HTML** — self-contained page with inline styles rendering a two-column layout (Customer Profile | Value Map) with sub-block cells. Tag `[Scope: VP]`.

5. PHASE 4 (Output): Present both outputs. Offer to write to `docs/value-proposition-canvas/`. Do not write without explicit confirmation. Chat-only fallback: display inline.

Directives:
- BMC-First: Always establish BMC context before the first VP prompt. Display what the BMC already says about Customer Segments and Value Propositions.
- No Redundancy: Never ask the user to re-state content already captured in the BMC. Reference it. Each VP prompt must assume the BMC baseline is known.
- One Block Per Interaction: Present exactly one block per turn.
- Content-First: User's natural-language response IS the block content. Store verbatim.
- Empty Blocks: `/next` with no content stores empty string. Do not re-prompt unless `/edit`.
- Anti-Fabrication: Never invent BMC or VP content. Unanswered = empty cell.
- Graceful Degradation: No BMC output found = halt with clear instruction. No silent fallback.

Schema `[Scope: VP]`:

```
Input Context (from BMC):
  bmc.customer-segments  <string>
  bmc.value-propositions <string>

State (in-memory):
  vp.customer-jobs       <string>
  vp.pains               <string>
  vp.gains               <string>
  vp.products-services   <string>
  vp.pain-relievers      <string>
  vp.gain-creators       <string>
  current-block          <int 0-5>
```

```
Markdown Output:

# Value Proposition Canvas

## BMC Baseline
### Customer Segments
{bmc content}

### Value Propositions
{bmc content}

## Customer Profile
### Customer Jobs
{content}

### Pains
{content}

### Gains
{content}

## Value Map
### Products & Services
{content}

### Pain Relievers
{content}

### Gain Creators
{content}
```

```
HTML Output:
Self-contained <!DOCTYPE html> with inline-styled two-column layout.
Left column: Customer Profile (Customer Jobs, Pains, Gains as sub-cells).
Right column: Value Map (Products & Services, Pain Relievers, Gain Creators as sub-cells).
BMC baseline shown in a muted preamble row above the two columns.
Empty cells show "(not defined)" in muted text.
Tagged [Scope: VP].
```