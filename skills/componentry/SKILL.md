# Componentry — Premium UI Components for React

> Copy-paste interactive components built with React 19, Tailwind CSS 4, and Framer Motion. Inspired by shadcn/ui.

## When to Use

- Building landing pages, demo pages, pitch pages, or any UI that needs visual impact
- Adding animated, interactive components to Next.js / React projects
- When the user asks for "cool UI", "animated components", or "premium look"
- Building hackathon demos that need to impress judges
- Any project using Tailwind CSS 4 + React 19

## Installation

Install individual components via shadcn CLI:

```bash
npx shadcn@latest add @componentry/<component-name>
```

## Available Components (46)

### Visual Effects & Backgrounds

| Component           | Description                           | Use For                 |
| ------------------- | ------------------------------------- | ----------------------- |
| `animated-gradient` | Smooth animated gradient backgrounds  | Hero sections, cards    |
| `border-beam`       | Animated beam traveling along borders | Feature cards, CTAs     |
| `circuit-board`     | Animated circuit board pattern        | Tech/crypto themes      |
| `dither-gradient`   | Retro dithered gradient effect        | Unique backgrounds      |
| `liquid-blob`       | Organic liquid blob animation         | Background accents      |
| `matrix-rain`       | Matrix-style falling characters       | Hacker/crypto aesthetic |
| `particle-galaxy`   | Interactive particle system           | Hero backgrounds        |
| `shimmer-button`    | Button with shimmer effect            | Primary CTAs            |
| `spotlight-card`    | Card with mouse-tracking spotlight    | Feature showcases       |
| `webgl-liquid`      | WebGL liquid distortion               | Premium hero sections   |

### Interactive & Motion

| Component                           | Description                 | Use For              |
| ----------------------------------- | --------------------------- | -------------------- |
| `cursor-driven-particle-typography` | Text that reacts to cursor  | Headlines            |
| `eye-tracking`                      | Element that follows cursor | Playful interactions |
| `image-ripple-effect`               | Ripple effect on images     | Image galleries      |
| `magnetic-dock`                     | macOS-style magnetic dock   | Navigation bars      |
| `scroll-choreography`               | Scroll-driven animations    | Long-form pages      |

### Data Display

| Component            | Description                  | Use For            |
| -------------------- | ---------------------------- | ------------------ |
| `flight-status-card` | Styled flight info card      | Status/info cards  |
| `github-calendar`    | GitHub contribution calendar | Activity displays  |
| `mac-keyboard`       | Interactive keyboard visual  | Keyboard shortcuts |
| `music-player`       | Styled music player UI       | Media interfaces   |
| `split-flap-display` | Airport-style flip display   | Counters, stats    |

## Component Philosophy

- **Copy-paste friendly** — you own the code, not a dependency
- **Simple APIs** — fewer props is better
- **Composition > configuration** — combine simple components
- **Explicit code** — readable without external context
- **Styles live close** — no global magic

## Usage Pattern

When building UI with componentry:

1. **Identify the visual need** — what impression should this section make?
2. **Pick the right component** — match the component to the use case (see table above)
3. **Install via shadcn CLI** — `npx shadcn@latest add @componentry/<name>`
4. **Customize inline** — modify the copied code directly for your needs
5. **Combine components** — layer effects (e.g., `particle-galaxy` bg + `spotlight-card` foreground)

## Example Combinations

### Hackathon Demo Page

```
- Hero: particle-galaxy + cursor-driven-particle-typography
- Features: spotlight-card grid with border-beam
- Stats: split-flap-display for live counters
- CTA: shimmer-button
```

### Crypto/DeFi Dashboard

```
- Background: circuit-board or matrix-rain
- Cards: spotlight-card with animated-gradient borders
- Navigation: magnetic-dock
- Data: split-flap-display for prices
```

### Pitch Page

```
- Hero: webgl-liquid or liquid-blob background
- Scroll sections: scroll-choreography
- Feature cards: spotlight-card with border-beam
- Final CTA: shimmer-button with animated-gradient
```

## Tech Requirements

- React 19+
- Tailwind CSS 4+
- Framer Motion (for motion components)
- Next.js 15+ (recommended but not required)

## Source

- **Repository:** https://github.com/harshjdhv/componentry
- **Website:** https://componentry.fun
- **License:** MIT
