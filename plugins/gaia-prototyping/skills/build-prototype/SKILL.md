---
name: build-prototype
description: Build HTML prototypes using the Gaia design system. Use this skill whenever the user wants to create a UI prototype, mockup, or proof-of-concept using Gaia components, or when they mention prototyping with Gaia, building a page layout with ga- classes, or recreating a UI screenshot with the Gaia design system.
---

# Build Prototype with Gaia Design System

Build standalone HTML prototypes that use the Gaia design system via its CSS from a public CDN.

## Core constraint — Gaia components only

Every visible UI element in the prototype must be built from Gaia components discovered in Step 2. Do not create custom-styled elements that replicate what a Gaia component already provides — even if it seems simpler or faster. If you need a progress bar, check if `ProgressBar.md` exists before writing custom CSS. If you need a badge, use the Gaia badge. Always cross-reference the discovered component list against what you need.

The only custom CSS allowed is **layout scaffolding**: flexbox/grid containers, spacing between sections, page-level margins and paddings. Everything else — buttons, inputs, cards, tags, progress bars, alerts, etc. — must come from Gaia.

If the UI requires something that has no Gaia equivalent, **stop and ask the user** before proceeding. Present them with options:
1. The closest Gaia alternative(s) you found, with a brief explanation of how they differ from what was requested.
2. Skip the element entirely.
3. Build it with custom CSS (only if the user explicitly chooses this).

Never silently substitute a custom element or skip something — the user may know of a better Gaia component to use, or may have a preference you can't predict. Wait for their answer before continuing.

## Before you start

Clarify the user's intent if not already obvious:

1. **From scratch** — the user describes what they want to build in words.
2. **From existing UI** — the user wants to recreate or adapt an existing design. Suggest they upload a screenshot if they haven't already.

Ask the user which case applies before proceeding. This matters because building from an existing UI means matching structure and layout closely, while from scratch gives more freedom.

## Step 1 — Lock dependency versions

The prototype depends on two CDN-hosted packages. Both must be version-pinned so the prototype remains stable over time.

<!-- unpkg.com does not work in Claude.ai's HTML preview. Use cdn.jsdelivr.net instead. -->

| Package | CDN URL pattern |
|---------|----------------|
| `@vsn-ux/gaia-styles` | `https://cdn.jsdelivr.net/npm/@vsn-ux/gaia-styles@{VERSION}/dist/all.css` |
| `lucide` | `https://cdn.jsdelivr.net/npm/lucide@{VERSION}/dist/umd/lucide.min.js` |

**Creating a new prototype:**
For each package, fetch its latest version from the npm registry (`https://registry.npmjs.org/{package}/latest`) and extract the `version` field from the JSON response. Use these exact versions in the CDN URLs so the prototype is pinned and won't break if packages update later.

**Editing an existing prototype:**
Find the existing `<link>` and `<script>` tags that reference these packages in the HTML. Keep the versions that are already there — do not upgrade or change them.

## Step 2 — Discover available components

<!-- data.jsdelivr.com API is blocked from Claude's sandbox. Use npm pack to download and extract the package instead. -->

Download and extract the package to discover components and read their docs:
```
npm pack @vsn-ux/gaia-styles@{VERSION} --pack-destination /tmp 2>/dev/null && tar -xzf /tmp/vsn-ux-gaia-styles-{VERSION}.tgz -C /tmp
```
Then list the available component docs:
```
ls /tmp/package/dist/docs/
```
Each `{ComponentName}.md` file represents an available component — these are the only components you may use. File names are **PascalCase** (e.g. `FormField.md`, not `form-field.md`).

If the user asks for something that doesn't have a doc file, stop and ask the user — present the closest Gaia alternatives and let them decide how to proceed (see "Core constraint" above).

Before using any component, read its documentation to understand the correct HTML structure from the extracted files at `/tmp/package/dist/docs/{ComponentName}.md`. Do not guess the structure, always consult the docs first.

## Step 3 — Build the prototype

Output a single standalone HTML file. Structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{Prototype title}</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@vsn-ux/gaia-styles@{VERSION}/dist/all.css">
  <style>body { font-family: 'Inter', sans-serif; }</style>
</head>
<body>
  <!-- Prototype content using only ga- classes -->
  <!-- Use <i data-lucide="icon-name"></i> for icons -->

  <script src="https://cdn.jsdelivr.net/npm/lucide@{VERSION}/dist/umd/lucide.min.js"></script>
  <script>
    lucide.createIcons();
  </script>
</body>
</html>
```

Rules:
- The prototype must be a single self-contained `.html` file. All JavaScript, styles, and content go in this one file. Do not create any additional files (no separate `.js`, `.css`, `.json`, images, etc.).
- Every UI element must use a Gaia component with the HTML structure from its docs (see "Core constraint" above). Before writing any custom-styled element, check the full component list from Step 2 — if a Gaia component covers the need, use it.
- Keep the HTML semantic and clean.
- Use inline `<style>` only for layout scaffolding (grid, flexbox positioning, page-level spacing) — never to style UI elements that Gaia already provides.
- Use inline `<script>` for any interactive behavior (tabs, modals, form validation, mock data, etc.).
- If building from a screenshot, see the "Building from a screenshot" section below.

## Building from a screenshot or sketch

First, determine the source type — this changes how you approach the build:

### Gaia screenshot (from a production app already using Gaia)

The user may be using this as a base to build upon later, so accuracy is critical.

**Component identification:** Look carefully at each UI element and match it to the exact Gaia component. Do not substitute similar-looking components — e.g. a segmented control is not two side-by-side buttons, and vice versa. When in doubt, check the component docs to compare.

**Iterative refinement (Claude Code with `chrome-devtools-mcp` only):**
If the `chrome-devtools-mcp` MCP server is available, do 2-3 rounds of visual comparison:
1. Open the HTML file in Chrome via the MCP.
2. Take a screenshot of the rendered prototype.
3. Compare it side-by-side with the original screenshot, focusing on:
   - Correct component choices (not just visually similar ones)
   - Layout and spacing
   - Typography and sizing
   - Icon placement
   - Colors and states (active, disabled, selected)
4. Fix any discrepancies and repeat.

Only present the prototype to the user once you're confident it closely matches the screenshot.

If `chrome-devtools-mcp` is not available (e.g. in Claude.ai chat or Cowork), skip the visual comparison rounds and present the prototype after best-effort construction.

### Non-Gaia screenshot or sketch (different design system, wireframe, hand-drawn)

The goal here is to translate the *intent and layout* into Gaia — not to replicate the original look pixel-for-pixel. The source is a reference for structure and functionality, not a visual target.

- Map each UI element to the closest Gaia equivalent. A Material "chip" becomes a Gaia Tag, a Fluent "dialog" becomes a Gaia Modal, etc.
- Preserve the layout, flow, and information hierarchy.
- Do not try to replicate the visual style of the source design system — let Gaia's own styling take over.
- If the source contains elements that have no Gaia equivalent, stop and ask the user before proceeding — list what's missing, suggest the closest Gaia alternatives, and let them decide (see "Core constraint" above).

If it's not obvious which case applies (e.g. a clean screenshot that could be Gaia or something else), ask the user.

## Icons

Use Lucide icons via `<i data-lucide="icon-name"></i>`. The Lucide UMD script and `lucide.createIcons()` call must be placed at the end of `<body>` — they replace all `data-lucide` elements with inline SVGs at load time. If JavaScript adds icons dynamically after page load, call `lucide.createIcons()` again after inserting the new elements.

## General design guidelines

<!-- TODO: Add link to Gaia design guidelines when available (e.g. external markdown/txt hosted on a documentation service) -->

General Gaia design guidelines will be linked here in the future. For now, follow standard UX best practices: consistent spacing, clear visual hierarchy, and accessible markup.
