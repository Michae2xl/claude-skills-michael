---
name: awesome-design-md
description: "Curated collection of 55 DESIGN.md files from real developer-focused websites. Use when building UI that needs to match the visual identity of known products (Stripe, Vercel, Linear, Coinbase, Claude, Supabase, etc.). Each DESIGN.md contains: color palette, typography, components, spacing, elevation, do/don'ts, responsive behavior, and AI prompt guide."
---

# Awesome Design MD

**Source:** https://github.com/Michae2xl/awesome-design-md  
**Original repo:** https://github.com/VoltAgent/awesome-design-md  
**Count:** 55 DESIGN.md files · Format: Google Stitch DESIGN.md spec

## What is DESIGN.md?

A plain-text design system file that AI agents read to generate consistent, pixel-perfect UI. No Figma, no JSON schemas — just Markdown. Drop it in the project root and any AI coding agent (or Google Stitch) instantly understands how the UI should look.

```
AGENTS.md  → how to BUILD the project
DESIGN.md  → how the project should LOOK
```

## How to Use This Skill

1. User says "build a page that looks like Stripe" or "use Vercel's design system"
2. Fetch the raw DESIGN.md from GitHub:
   ```
   https://raw.githubusercontent.com/VoltAgent/awesome-design-md/main/design-md/<site>/DESIGN.md
   ```
3. Read the file and extract: colors, fonts, component styles, spacing
4. Apply to the UI being built

## Each DESIGN.md Contains

| Section                   | What it captures                        |
| ------------------------- | --------------------------------------- |
| Visual Theme & Atmosphere | Mood, density, design philosophy        |
| Color Palette & Roles     | Semantic name + hex + functional role   |
| Typography Rules          | Font families, full hierarchy table     |
| Component Stylings        | Buttons, cards, inputs, nav with states |
| Layout Principles         | Spacing scale, grid, whitespace         |
| Depth & Elevation         | Shadow system, surface hierarchy        |
| Do's and Don'ts           | Design guardrails, anti-patterns        |
| Responsive Behavior       | Breakpoints, touch targets              |
| Agent Prompt Guide        | Quick color reference, ready prompts    |

## Full Collection (55 sites)

### AI & Machine Learning

| Site            | Style                                       | Key colors            |
| --------------- | ------------------------------------------- | --------------------- |
| **Claude**      | Warm terracotta, editorial layout           | Terracotta accent     |
| **Cohere**      | Vibrant gradients, data-rich dashboard      | Multi-color gradients |
| **ElevenLabs**  | Dark cinematic, audio-waveform aesthetics   | Dark + neon           |
| **Minimax**     | Bold dark interface, neon accents           | Neon on dark          |
| **Mistral AI**  | French minimalism, purple-toned             | Purple                |
| **Ollama**      | Terminal-first, monochrome simplicity       | Monochrome            |
| **OpenCode AI** | Developer-centric dark theme                | Dark                  |
| **Replicate**   | Clean white canvas, code-forward            | White                 |
| **RunwayML**    | Cinematic dark, media-rich                  | Dark cinematic        |
| **Together AI** | Technical, blueprint-style                  | Blueprint blue        |
| **VoltAgent**   | Void-black, emerald accent, terminal-native | #000 + emerald        |
| **xAI**         | Stark monochrome, futuristic minimalism     | B&W                   |

### Developer Tools

| Site           | Style                                 | Key colors      |
| -------------- | ------------------------------------- | --------------- |
| **Cursor**     | Sleek dark, gradient accents          | Dark + gradient |
| **Expo**       | Dark theme, tight letter-spacing      | Dark            |
| **Linear**     | Ultra-minimal, precise                | Purple accent   |
| **Lovable**    | Playful gradients, friendly dev       | Gradients       |
| **Mintlify**   | Clean, reading-optimized              | Green accent    |
| **PostHog**    | Playful, developer-friendly dark      | Dark + playful  |
| **Raycast**    | Sleek dark chrome, vibrant gradients  | Dark + vibrant  |
| **Resend**     | Minimal dark, monospace accents       | Dark minimal    |
| **Sentry**     | Dark dashboard, data-dense            | Pink-purple     |
| **Supabase**   | Dark emerald, code-first              | Emerald         |
| **Superhuman** | Premium dark, keyboard-first          | Purple glow     |
| **Vercel**     | Black and white precision, Geist font | B&W             |
| **Warp**       | Dark IDE-like, block-based            | Dark IDE        |
| **Zapier**     | Warm orange, illustration-driven      | Orange          |

### Infrastructure & Cloud

| Site           | Style                                  | Key colors      |
| -------------- | -------------------------------------- | --------------- |
| **ClickHouse** | Technical docs style                   | Yellow          |
| **Composio**   | Modern dark, colorful icons            | Dark + color    |
| **HashiCorp**  | Enterprise-clean                       | B&W             |
| **MongoDB**    | Developer docs focus                   | Green           |
| **Sanity**     | Content-first editorial                | Red accent      |
| **Stripe**     | Signature purple gradients, weight-300 | Purple gradient |

### Design & Productivity

| Site          | Style                               | Key colors     |
| ------------- | ----------------------------------- | -------------- |
| **Airtable**  | Colorful, friendly                  | Multi-color    |
| **Cal.com**   | Clean neutral, developer simplicity | Neutral        |
| **Clay**      | Organic shapes, soft gradients      | Soft gradients |
| **Figma**     | Vibrant multi-color, playful        | Multi-color    |
| **Framer**    | Bold black/blue, motion-first       | Black + blue   |
| **Intercom**  | Friendly blue, conversational UI    | Blue           |
| **Miro**      | Bright yellow, infinite canvas      | Yellow         |
| **Notion**    | Warm minimalism, serif headings     | Warm minimal   |
| **Pinterest** | Red accent, masonry, image-first    | Red            |
| **Webflow**   | Blue-accented, polished marketing   | Blue           |

### Fintech & Crypto

| Site         | Style                           | Key colors      |
| ------------ | ------------------------------- | --------------- |
| **Coinbase** | Clean blue, institutional trust | Blue            |
| **Kraken**   | Purple dark, data-dense         | Purple dark     |
| **Revolut**  | Sleek dark, gradient cards      | Dark + gradient |
| **Wise**     | Bright green, friendly          | Green           |

### Enterprise & Consumer

| Site        | Style                            | Key colors      |
| ----------- | -------------------------------- | --------------- |
| **Airbnb**  | Warm coral, photography-driven   | Coral           |
| **Apple**   | Premium white space, cinematic   | White           |
| **BMW**     | Dark premium surfaces            | Dark premium    |
| **IBM**     | Carbon design, structured blue   | Blue structured |
| **NVIDIA**  | Green-black energy, technical    | Green + black   |
| **SpaceX**  | Stark B&W, full-bleed imagery    | B&W             |
| **Spotify** | Vibrant green on dark, bold type | Green + dark    |
| **Uber**    | Bold B&W, urban energy           | B&W             |

## Raw File URLs

Pattern:

```
https://raw.githubusercontent.com/VoltAgent/awesome-design-md/main/design-md/<slug>/DESIGN.md
```

Examples:

- Stripe → `design-md/stripe/DESIGN.md`
- Vercel → `design-md/vercel/DESIGN.md`
- Linear → `design-md/linear.app/DESIGN.md`
- Supabase → `design-md/supabase/DESIGN.md`
- Coinbase → `design-md/coinbase/DESIGN.md`
- Claude → `design-md/claude/DESIGN.md`

## When to Apply

- User asks to match an existing product's visual identity
- User says "make it look like [known site]"
- User wants a professional dark dashboard (Supabase, Linear, Vercel, Raycast)
- User wants fintech/crypto UI (Stripe, Coinbase, Revolut, Kraken)
- User wants editorial/minimal (Notion, Vercel, Apple)
- User wants vibrant/playful (Figma, Lovable, PostHog)

## Quick Reference — Best for Dark Dashboards

For crypto/fintech dark UIs (like SharkTank):

- **Kraken** — purple dark, data-dense, crypto native
- **Coinbase** — clean blue, institutional, crypto
- **Supabase** — dark emerald, developer dashboard
- **Linear** — ultra-minimal dark, precise
- **Raycast** — sleek dark chrome, vibrant accents
- **VoltAgent** — void-black, emerald, terminal-native (closest to SharkTank)
