---
name: html-merge
description: Renders markdown content into an HTML document template. Converts markdown to HTML (via pandoc or agent-inline), slots the result at the `{{CONTENT}}` marker in a caller-provided HTML template, and writes a self-contained HTML file. Used by report-generating skills (audits, letterheads) as a rendering leaf.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies: []
user-invocable: false
---

Invoked by caller with three parameters:
- `<markdown_path>` — path to the markdown file to render
- `<template_path>` — path to the HTML template containing `{{CONTENT}}`
- `<output_path>` — path to write the final HTML file

```mermaid
flowchart TD
    START(["Invoke html-merge"]) --> A["Read template from\n<template_path>"]
    A --> B{Marker<br>{{CONTENT}}<br>present?}
    B -->|No| HALT(["HALT: template must\ncontain {{CONTENT}}"])
    B -->|Yes| C["Read markdown\nfrom <markdown_path>"]
    C --> D{Pandoc<br>available?}
    D -->|Yes| E["Convert:\npandoc <markdown_path>\n-f markdown -t html"]
    D -->|No| F["Convert:\nagent-inline\nmarkdown→HTML"]
    E --> G["Replace {{CONTENT}}\nwith rendered HTML"]
    F --> G
    G --> H["Validate output\nwell-formed HTML"]
    H --> I["Write to\n<output_path>"]
    I --> DONE(["Done"])
```

1. PHASE 1 (Validate): Read `<template_path>`. Verify `{{CONTENT}}` marker exists exactly once. If absent or duplicated: HALT with message `Template must contain exactly one {{CONTENT}} marker.`. Read `<markdown_path>`. Verify file exists and is non-empty. Report missing or empty inputs to caller.

2. PHASE 2 (Render): Convert markdown to HTML. Try `pandoc <markdown_path> -f markdown -t html` first. If pandoc unavailable, use agent-inline markdown→HTML conversion (CommonMark: headings, lists, tables, code blocks, links, images, bold/italic, blockquotes). Wrap inline-rendered output in `<div class="markdown-content">` for scoped styling.

3. PHASE 3 (Merge): Replace `{{CONTENT}}` with rendered HTML. Validate well-formedness (matching tags, doctype). Retry on failure; never write broken output.

4. PHASE 4 (Output): Write merged HTML to `<output_path>`. Create parent directories if needed. Confirm completion to caller with output path and file size.

Directives:
- Template-Owner: The caller supplies the template. html-merge handles marker replacement only. Never modify template structure beyond the `{{CONTENT}}` slot.
- Single Marker: `{{CONTENT}}` is the sole insertion point. Other markers are the caller's responsibility to preprocess before invoking html-merge.
- Fallback Doctype: If the template lacks `<!DOCTYPE>` or `<html>`, wrap in `<!DOCTYPE html><html><head><meta charset="UTF-8"><title>Document</title></head><body>{content}</body></html>`.
- Style Scoping: Inline-rendered content wraps in `<div class="markdown-content">` to avoid template-style conflicts. Pandoc output left as-is.
- Anti-Hallucination: Never invent content or interpolate missing markers. Fail loudly on absent `{{CONTENT}}`.
- Output Determinism: Same inputs produce byte-identical output for the same rendering path. Agent-inline rendering must produce stable, reproducible output across invocations.