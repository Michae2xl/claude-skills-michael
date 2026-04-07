---
name: crypto-explainer-video
description: Build cinematic crypto/blockchain explainer videos with Remotion — scene architecture, reusable animated components, Apple-keynote design system, and production workflow. Based on the Zcash Privacy video project.
metadata:
  tags: remotion, video, crypto, blockchain, explainer, animation, react
---

## When to use

Use this skill when building **explainer or promotional videos for crypto/blockchain projects** using Remotion. Covers:
- Project scaffolding and scene architecture
- Cinematic animated components (backgrounds, text, charts, quotes, coin animations)
- Apple-keynote-inspired dark design system with brand color theming
- Scene transition choreography with `@remotion/transitions`
- Production render pipeline

Also useful for any **data-driven animated video** that needs premium visual quality (halving charts, comparison tables, timeline visualizations).

## Project Architecture

```
src/
├── index.ts              # registerRoot entry
├── Root.tsx              # Composition registry + TransitionSeries orchestrator
├── lib/
│   ├── constants.ts      # FPS, dimensions, colors, fonts, scene durations, data
│   ├── spring-configs.ts # Reusable SpringConfig presets (FAST, SMOOTH, BOUNCY, SLOW)
│   ├── assets.ts         # staticFile() references for logos, photos
│   └── quotes.ts         # Typed quote data with author/role
├── components/           # Reusable animated building blocks
│   ├── AnimatedBackground.tsx  # Gradient orbs + floating particles + vignette
│   ├── AnimatedText.tsx        # Word-by-word spring entrance with blur
│   ├── QuoteCard.tsx           # Full-screen quote with typewriter + author photo
│   ├── HalvingChart.tsx        # Animated bar chart with rolling coin
│   ├── PersonPhoto.tsx         # Circular photo with animated gradient ring
│   ├── GlassBox.tsx            # Glassmorphism card (transparent vs shielded)
│   ├── ZcashCoin.tsx           # 3D-flipping coin with glow
│   └── ZcashLogo.tsx           # Animated logo with subtle rotation + glow
├── sequences/            # Scene compositions (each is a full-screen sequence)
│   ├── Hook.tsx                # Opening quote scene
│   ├── WhatIsZcash.tsx         # Multi-phase: fork visual → comparison → chart
│   ├── TransparentVsShielded.tsx  # Conveyor belt animation
│   ├── ZkSnarks.tsx            # Acronym + Waldo analogy + ZK machine
│   ├── TheCeremony.tsx         # Orbital diagram + shard destruction
│   ├── PrivacyIsARight.tsx     # Side-by-side comparison + quote montage
│   └── Closing.tsx             # Logo + tagline + CTA
└── public/               # Static assets (logos, photos)
```

### Key Principles

1. **Scenes are self-contained** — each sequence file manages its own animation timeline using `useCurrentFrame()` relative offsets
2. **Multi-phase scenes** — complex scenes use frame ranges to transition between visual phases within a single sequence (e.g., fork → comparison table → chart)
3. **Constants are centralized** — colors, fonts, scene durations, data arrays all live in `lib/constants.ts`
4. **Spring presets are shared** — `SPRING_FAST`, `SPRING_SMOOTH`, `SPRING_BOUNCY`, `SPRING_SLOW` cover 95% of animation needs
5. **Transitions are declarative** — `TransitionSeries` in Root.tsx handles inter-scene transitions (fade, slide, wipe)

## Design System

### Color Palette (Dark Mode + Brand Accents)

```typescript
export const COLORS = {
  // Brand accents — swap these per project
  brandPrimary: "#F4B728",     // e.g., Zcash yellow
  brandSecondary: "#FFD666",
  brandTertiary: "#E09E00",

  // Backgrounds — rich gradients, never flat black
  bgDeep: "#05050F",
  bgNavy: "#0A0E1A",
  bgPurple: "#0D0A1A",
  bgCard: "rgba(255, 255, 255, 0.04)",
  bgGlass: "rgba(255, 255, 255, 0.06)",

  // Text hierarchy
  white: "#F5F5F7",
  textSecondary: "#A1A1A6",
  textTertiary: "#6E6E73",

  // Semantic accents
  blue: "#2997FF",
  green: "#30D158",
  red: "#FF453A",
  purple: "#BF5AF2",
} as const;
```

### Typography Scale

```typescript
export const TYPE = {
  hero: 96,    // Closing taglines
  h1: 76,      // Scene titles
  h2: 56,      // Quote text, large headings
  h3: 42,      // Sub-headings, author names
  body: 28,    // Descriptions, labels
  caption: 22, // Subtitles, uppercase labels
  small: 18,   // Fine print, data labels
} as const;

export const FONTS = {
  display: "SF Pro Display, Inter, -apple-system, sans-serif",
  text: "SF Pro Text, Inter, -apple-system, sans-serif",
  mono: "SF Mono, JetBrains Mono, monospace",
  serif: "New York, Georgia, serif",
} as const;
```

### Spring Presets

```typescript
export const SPRING_FAST: SpringConfig = { damping: 12, stiffness: 200, mass: 0.5 };
export const SPRING_SMOOTH: SpringConfig = { damping: 15, stiffness: 120, mass: 0.8 };
export const SPRING_BOUNCY: SpringConfig = { damping: 8, stiffness: 180, mass: 0.6 };
export const SPRING_SLOW: SpringConfig = { damping: 20, stiffness: 80, mass: 1.2 };
```

## Core Components

### AnimatedBackground

Multi-variant background with slow-moving gradient orbs, floating particles, noise texture, and vignette:

```typescript
<AnimatedBackground variant="warm" />   // Gold/amber tones
<AnimatedBackground variant="cool" />   // Blue/purple tones
<AnimatedBackground variant="ceremony" /> // Purple/mystical
<AnimatedBackground variant="default" /> // Balanced mix
```

Pattern: 3 gradient orbs with sinusoidal movement + 30 floating particles + noise overlay + radial vignette.

### AnimatedText

Word-by-word spring entrance with blur dissolve:

```typescript
<AnimatedText
  text="How ZK-SNARKs Work"
  fontSize={TYPE.h1}
  color={COLORS.brandPrimary}
  delay={5}
/>
```

Each word enters with `SPRING_FAST`, staggered by 4 frames. Includes translateY + opacity + blur animation.

### QuoteCard

Full-screen quote with typewriter text reveal, oversized decorative quote mark, gradient divider, and optional author photo:

```typescript
<QuoteCard
  text="Privacy is not about something to hide."
  author="Edward Snowden"
  role="Whistleblower"
  photoSrc={PHOTO_SNOWDEN}
/>
```

### PersonPhoto

Circular photo with animated conic-gradient ring and subtle pulse:

```typescript
<PersonPhoto src={PHOTO_SNOWDEN} size={190} delay={8} />
```

### HalvingChart

Animated bar chart with decay curve, rolling coin, grid lines, and halving indicators:

```typescript
<HalvingChart delay={335} />
```

Data-driven from a `HALVING_DATA` array in constants.

## Scene Patterns

### Multi-Phase Scene

Use frame ranges to create visual phases within one sequence:

```typescript
export const MyScene: React.FC = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Phase 1: 0–120 frames
  const showPhase1 = frame < 120;

  // Phase 2: 120–320 frames
  const showPhase2 = frame >= 120 && frame < 320;
  const phase2Fade = interpolate(frame, [120, 150], [0, 1], {
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });

  // Phase 3: 320+ frames
  const showPhase3 = frame >= 320;

  return (
    <AbsoluteFill>
      <AnimatedBackground variant={showPhase3 ? "warm" : "default"} />
      {showPhase1 && <Phase1Content />}
      {showPhase2 && <div style={{ opacity: phase2Fade }}><Phase2Content /></div>}
      {showPhase3 && <Phase3Content />}
    </AbsoluteFill>
  );
};
```

### Staggered List Entrance

Animate list items with staggered spring delays:

```typescript
{items.map((item, i) => {
  const rowSpring = spring({ frame: frame - 140 - i * 15, fps, config: SPRING_FAST });
  return (
    <div
      key={i}
      style={{
        opacity: rowSpring,
        transform: `translateY(${interpolate(rowSpring, [0, 1], [20, 0])}px)`,
      }}
    >
      {item.content}
    </div>
  );
})}
```

### Conveyor Belt / Process Flow

Animate an object moving through a transformation machine:

1. Calculate `coinProgress` from frame interpolation (0 to 1)
2. Map progress to X position along belt
3. Toggle visibility when inside machine (`insideMachine` boolean)
4. Change appearance after machine exit (`pastMachine` boolean)
5. Add processing effects (sparks, glow pulse, indicator dots)

### Orbital Diagram

Place items in a circle using angle math:

```typescript
const PARTICIPANTS = [
  { angle: 0, label: "P1", city: "Denver" },
  { angle: 60, label: "P2", city: "London" },
  // ...
];

{PARTICIPANTS.map((p, i) => {
  const rad = (p.angle * Math.PI) / 180;
  const px = Math.cos(rad) * radius;
  const py = Math.sin(rad) * radius;
  return (
    <div style={{
      position: "absolute",
      top: "50%", left: "50%",
      transform: `translate(calc(-50% + ${px}px), calc(-50% + ${py}px))`,
    }}>
      {/* participant content */}
    </div>
  );
})}
```

### Quote Montage

Cycle through quotes using `<Sequence>` inside a scene:

```typescript
const quoteMontageStart = 190;
const framesPerQuote = Math.floor(remainingFrames / quotes.length);

{quotes.map((quote, i) => (
  <Sequence key={i} from={quoteMontageStart + i * framesPerQuote} durationInFrames={framesPerQuote}>
    <QuoteCard text={quote.text} author={quote.author} role={quote.role} photoSrc={quote.photo} />
  </Sequence>
))}
```

## Scene Transition Orchestration

Use `TransitionSeries` in Root.tsx to chain scenes:

```typescript
import { TransitionSeries, linearTiming, springTiming } from "@remotion/transitions";
import { fade } from "@remotion/transitions/fade";
import { slide } from "@remotion/transitions/slide";
import { wipe } from "@remotion/transitions/wipe";

const TRANSITION_FRAMES = 20;

<TransitionSeries>
  <TransitionSeries.Sequence durationInFrames={SCENES.hook.duration}>
    <Hook />
  </TransitionSeries.Sequence>

  <TransitionSeries.Transition
    presentation={fade()}
    timing={linearTiming({ durationInFrames: TRANSITION_FRAMES })}
  />

  <TransitionSeries.Sequence durationInFrames={SCENES.next.duration}>
    <NextScene />
  </TransitionSeries.Sequence>
</TransitionSeries>
```

**Transition guidelines:**
- `fade()` — emotional/cinematic moments (opening, closing, ceremony)
- `slide({ direction: "from-right" })` — content progression
- `wipe({ direction: "from-left" })` — topic shift
- `springTiming` for organic feel, `linearTiming` for clean cuts

## Composition Registry

Register both the full video and individual scenes for development:

```typescript
export const RemotionRoot: React.FC = () => (
  <>
    <Composition id="FullVideo" component={VideoWithTransitions} durationInFrames={TOTAL} fps={FPS} width={WIDTH} height={HEIGHT} />
    {/* Individual scenes for previewing */}
    <Composition id="Hook" component={Hook} durationInFrames={SCENES.hook.duration} fps={FPS} width={WIDTH} height={HEIGHT} />
    <Composition id="WhatIsZcash" component={WhatIsZcash} durationInFrames={SCENES.whatIsZcash.duration} fps={FPS} width={WIDTH} height={HEIGHT} />
    {/* ... */}
  </>
);
```

## Production Workflow

### 1. Scaffold

```bash
npm init -y
npm i remotion @remotion/cli @remotion/media-utils @remotion/transitions react react-dom
npm i -D typescript @types/react @types/react-dom
```

### 2. Develop

```bash
npx remotion studio   # Live preview at localhost:3000
```

### 3. Render

```bash
npx remotion render FullVideo out/video.mp4
```

### 4. Config

```typescript
// remotion.config.ts
import { Config } from "@remotion/cli/config";
Config.setVideoImageFormat("jpeg");
Config.setOverwriteOutput(true);
```

## Adapting for a New Crypto Project

1. **Swap brand colors** in `constants.ts` — change `brandPrimary/Secondary/Tertiary`
2. **Replace logo assets** in `public/` and update `assets.ts`
3. **Update data arrays** (halving data, feature comparisons, quotes, participants)
4. **Adjust scene durations** in `SCENES` object
5. **Reuse components as-is** — AnimatedBackground, AnimatedText, QuoteCard, PersonPhoto, HalvingChart are all parametric
6. **Add/remove scenes** — each sequence is independent, just wire into TransitionSeries

## Visual Effects Reference

| Effect | Technique | Used In |
|--------|-----------|---------|
| Gradient orbs | Radial gradient + sinusoidal position + blur | AnimatedBackground |
| Floating particles | Array of positioned dots with sinusoidal opacity | AnimatedBackground |
| Word-by-word text | Staggered spring per word + blur dissolve | AnimatedText |
| Typewriter reveal | `text.slice(0, charsToShow)` + frame interpolation | QuoteCard |
| Rotating gradient ring | `conic-gradient(from ${rotation}deg, ...)` | PersonPhoto |
| Cinematic letterbox | Animated `height` bars at top/bottom | Hook |
| Gradient text | `WebkitBackgroundClip: "text"` + `WebkitTextFillColor: "transparent"` | Multiple |
| Glow pulse | `filter: drop-shadow()` with sinusoidal intensity | ZcashCoin, Closing |
| Data-driven chart | Array.map over data + spring-animated bar heights | HalvingChart |
| Rolling coin on curve | Coin position interpolated along SVG path points | HalvingChart |
| Conveyor belt | Frame-driven X position + machine enter/exit states | TransparentVsShielded |
| Shard destruction | Sequential opacity fade per participant by index | TheCeremony |
| Progress bar fill | `width: ${spring * 100}%` inside a track | Multiple |
