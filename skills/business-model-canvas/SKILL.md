---
name: business-model-canvas
description: Guides a user through the 9 building blocks of the Business Model Canvas step by step, then compiles the completed canvas into both Markdown and HTML formats. Prompts for each BMC segment in sequence with explanations, allows navigation between blocks, and renders the final artifact.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies: []
argument-hint: "[start]  # begin a new Business Model Canvas session"
user-invocable: true
---
Collect exactly the 9 BMC segments in order. NEVER fabricate a segment's content. Unanswered segments render as empty cells.

1. PHASE 0 (Initialize): Greet the user, explain the 9-block process, set up empty in-memory canvas state. Mode is always `create`.

2. PHASE 1 (Elicit): Walk through the 9 blocks in fixed order. Present each block's prompt one at a time. Wait for user content. Advance after receiving input. Support these interactive commands:
   - `/next` — advance to next block (leave current empty)
   - `/back` — return to previous block
   - `/edit <block>` — jump to a named block by partial match
   - `/done` — finish elicitation, advance to Render
   - `/status` — show filled/remaining blocks

   Block order and prompts:
   1. **Customer Segments** — Who are the customers? Who is the business creating value for?
   2. **Value Propositions** — What specific value is delivered? What problem is solved?
   3. **Channels** — How are customer segments reached? (direct sales, web, partners, etc.)
   4. **Customer Relationships** — What relationship with each segment? (personal assistance, automated, communities, etc.)
   5. **Revenue Streams** — How does the business earn revenue? What are customers willing to pay for?
   6. **Key Resources** — What assets are essential? (physical, financial, intellectual, human)
   7. **Key Activities** — What actions must the business perform to operate successfully?
   8. **Key Partnerships** — Who are the suppliers and partners that make the model work?
   9. **Cost Structure** — What are the most significant costs incurred?

3. PHASE 2 (Render): Compile completed canvas into two formats:
   - **Markdown** — structured document, each block as a heading with its content.
   - **HTML** — self-contained page with inline styles rendering the BMC grid layout. Tag `[Scope: BMC]`.

4. PHASE 3 (Output): Present both outputs. Offer to write to `docs/business-model-canvas/`. Do not write without explicit confirmation. Chat-only fallback: display inline.

Directives:
- One Block Per Interaction: Present exactly one block per turn. Never ask about multiple blocks in a single message.
- Content-First: The user's natural-language response IS the block content. Store verbatim.
- Empty Blocks: `/next` with no content stores empty string. Do not re-prompt unless user invokes `/edit`.
- Anti-Fabrication: Never invent block content. Unanswered = empty cell.
- Output Portability: HTML self-contained (inline styles). Markdown valid CommonMark.

Schema `[Scope: BMC]`:

```
State (in-memory):
  canvas.customer-segments      <string>
  canvas.value-propositions     <string>
  canvas.channels               <string>
  canvas.customer-relationships <string>
  canvas.revenue-streams        <string>
  canvas.key-resources          <string>
  canvas.key-activities         <string>
  canvas.key-partnerships       <string>
  canvas.cost-structure         <string>
  current-block                 <int 0-8>
```

```
Markdown Output:

# Business Model Canvas

## Customer Segments
{content}

## Value Propositions
{content}

## Channels
{content}

## Customer Relationships
{content}

## Revenue Streams
{content}

## Key Resources
{content}

## Key Activities
{content}

## Key Partnerships
{content}

## Cost Structure
{content}
```

```
HTML Output:
Self-contained <!DOCTYPE html> with an inline-styled 9-cell grid reflecting the BMC layout.
Each cell shows the segment name as a heading and its content below.
Cells with empty content show "(not defined)" in muted text.
Tagged [Scope: BMC].
```