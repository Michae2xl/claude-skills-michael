---
name: ai-pfp-generator
description: Build AI-powered profile picture (PFP/avatar) generator web apps with customizable filters, logo compositing, and consistent art style. Use when creating avatar generators, PFP makers, community identity tools, or any app that generates illustrated character portraits with brand logos overlaid on clothing.
---

# AI PFP Generator

Build web apps that generate illustrated character avatars via AI image generation, with server-side logo compositing onto the character's clothing.

## Architecture Overview

The system has two decoupled stages:

1. **AI Generation** — Generate a plain avatar (no logo) using `generateImage()` with a fixed-composition prompt and style reference images.
2. **Logo Composite** — Overlay the user's chosen logo onto the avatar's clothing using `sharp` on the server. This ensures pixel-perfect placement regardless of AI variance.

Never ask the AI to draw logos on clothing — it produces inconsistent, blurry results.

## Workflow

Building a PFP generator involves these steps:

1. Create reference images (generate 5-10 manually to anchor the style)
2. Design the fixed-composition prompt (see `references/prompt-engineering.md`)
3. Build the logo catalog (collect PNG logos with transparent backgrounds)
4. Implement server-side compositing (see `references/logo-compositing.md`)
5. Build the database schema (see `references/database-schema.md`)
6. Build the frontend Generator UI (see `references/frontend-patterns.md`)
7. Calibrate logo position against reference images

### Step 1: Create Reference Images

Generate 5-10 reference avatars manually using the AI image generation tool. These serve as **style anchors** — pass one randomly as `originalImages` in every generation call to maintain visual consistency.

Requirements for reference images:
- Fixed composition: solid dark background, colored circle behind head, plain clothing
- Bold line art / comic book style with limited color palette
- Character in tight close-up (fills 70-80% of frame)
- Plain clothing with NO logos or symbols

### Step 2: Design the Fixed-Composition Prompt

The prompt must enforce a **rigid layout** so every generated avatar has identical framing. Read `references/prompt-engineering.md` for the proven prompt template.

Key elements:
- **Solid black background** filling entire image
- **Brand-color circle** (halo) behind the character's head
- **Tight close-up** framing (head near top, shoulders wide, mid-chest crop)
- **Plain solid clothing** (no logos — composited later)
- **Critical rules block** appended to every prompt to prevent AI deviation

Customizable elements (via user filters):
- Archetype/personality (affects expression only, not composition)
- Hair style, facial hair, skin tone, glasses
- Gender (male/female with gender-specific options like makeup, accessories)
- Custom text prompt (sanitized to block composition-breaking terms)

### Step 3: Build the Logo Catalog

Collect brand logos as PNG files with transparent backgrounds. Structure:

```ts
interface LogoEntry {
  id: string;        // "brand-variant" slug
  label: string;     // "Brand Yellow" display name
  group: string;     // "Brand" organization
  url: string;       // CDN URL of full-res transparent PNG
  thumbnail: string; // Same URL (for picker preview)
}
```

Clean logos with white backgrounds using Pillow before adding to catalog. See `references/logo-compositing.md` for the cleanup script.

### Step 4: Implement Server-Side Compositing

Use `sharp` to overlay logos at a fixed position on the avatar's clothing:

```ts
const BADGE_CONFIG = {
  centerX: 0.66,     // Horizontal position (0-1)
  centerY: 0.88,     // Vertical position (0-1)
  sizeRatio: 0.11,   // Logo size as fraction of image width
};
```

Pipeline: download avatar → download logo → trim transparent padding → resize → composite → upload to S3. See `references/logo-compositing.md` for full implementation.

### Step 5: Build the Database Schema

Store avatars with all filter metadata for gallery and history features. Use session-based access (client UUID in localStorage) for anonymous generation. See `references/database-schema.md` for the Drizzle ORM schema.

### Step 6: Build the Frontend

Two-column layout: filters on left, avatar preview on right. See `references/frontend-patterns.md` for component patterns including:
- Filter sections (archetype cards, physiognomy dropdowns, logo picker grid)
- Avatar preview with loading states
- Download button using fetch+Blob (not `<a download>`)
- Session history sidebar
- Dark theme with brand accent color

### Step 7: Calibrate Logo Position

After generating test avatars:
1. Open generated images in an editor, note desired logo center coordinates
2. Divide by image dimensions to get ratios for `BADGE_CONFIG`
3. Test with multiple avatars (male + female) to verify consistency
4. Acceptable tolerance: ±0.005 on each axis

## Key Decisions

| Decision | Rationale |
|---|---|
| Logo composited server-side, not by AI | AI produces blurry/inconsistent logos; sharp gives pixel-perfect results |
| Fixed composition enforced in prompt | Ensures logo placement area is always in the same position |
| Reference images as style anchors | Prevents style drift across generations |
| Custom prompts sanitized | Blocks terms that could break the fixed layout (logo, background, circle) |
| Session-based access | Allows anonymous generation without requiring auth |
| fetch+Blob for download | `<a download>` fails for cross-origin CDN URLs |

## Common Pitfalls

- **AI ignoring "no logo" instruction**: Reinforce with 3+ repetitions in the prompt. Use CRITICAL RULES block.
- **Inconsistent framing**: Always pass a reference image. Specify exact framing percentages (70-80% fill).
- **Logo on white background**: Always trim and clean logos before adding to catalog.
- **Download not working**: Never rely on `<a download>` for CDN URLs. Use fetch+Blob.
- **Generation errors (500)**: These are transient provider errors. Show a retry button, not an error page.
