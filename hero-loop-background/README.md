# hero-loop-background

A Claude Code skill that creates a full-bleed hero section backed by a looping boomerang video. The clip plays forward then reverses — no hard cut on loop, no awkward flash. Gradient overlays are calibrated automatically so your headline stays readable across all video frames.

---

## What it does

1. **Probes** your source video for resolution, fps, and duration
2. **Creates the boomerang** with ffmpeg — concatenates the clip with its own reverse so it loops seamlessly forever
3. **Generates** the complete hero HTML with a `<video autoplay muted loop playsinline>` background
4. **Applies** gradient overlays (top-to-bottom + radial vignette) tuned for your text colour scheme
5. **Verifies** with Playwright screenshots at desktop (1440px) and mobile (390px)

---

## Prerequisites

- [Claude Code](https://claude.ai/code) CLI installed
- [ffmpeg](https://ffmpeg.org/download.html) available on `PATH`
- [Node.js](https://nodejs.org/) (for Playwright screenshot verification)

---

## Installation

Copy the `hero-loop-background` folder into your Claude skills directory:

```bash
# macOS / Linux
cp -r hero-loop-background ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse hero-loop-background "$env:USERPROFILE\.claude\skills\"
```

---

## Usage

Any of these prompts will activate the skill:

```
Make this video boomerang and use it as the hero background: assets/product.mp4
```
```
Create a cinematic hero section with a looping video background from hero.mp4
```
```
I want an animated hero — take this video and make it loop as a boomerang behind the headline
```

Claude will ask for the headline text, text colour (light on dark / dark on light), and an optional poster fallback image before proceeding.

---

## What gets created

| File | Description |
|------|-------------|
| `hero-bg.mp4` | Boomerang-encoded video (forward + reversed), placed in your output directory |
| HTML snippet | Full hero `<section>` with video background, gradient overlays, and content layer |

---

## Customisation

| Parameter | Default | Description |
|-----------|---------|-------------|
| `HERO_HEIGHT` | `100vh` | Section height |
| `TEXT_COLOR` | `light` | `light` = white text / `dark` = dark text |
| `-crf` | `20` | Quality. `18` = higher quality, `24` = smaller file |
| Gradient alpha | Auto-calibrated | Adjust overlay stop values to taste |
| Source clip length | Any | Clips over ~15s should be trimmed before encoding to avoid memory issues with the `reverse` filter |

---

## How it works

### The boomerang filter

```
INPUT ──┬──► forward frames ──────────────────┐
        │                                      ▼
        └──► reverse ──► reversed frames ──► concat ──► OUTPUT
```

The ffmpeg filter graph splits the input into two streams: the original and a reversed copy. These are concatenated sequentially. Because the first frame of the reversed segment is the last frame of the forward segment (and vice versa), the loop point is perfectly seamless.

### Why gradient overlays instead of a dark overlay?

A uniform dark overlay (`background: rgba(0,0,0,0.5)`) kills the video — it looks like a poster with a filter slapped on it. Two targeted gradients (directional + radial) darken only the areas where text lives (top nav, headline, bottom CTA) while leaving the visual centre of the video unobscured, preserving the cinematic quality.

### Poster fallback

The `poster` attribute on the `<video>` element specifies an image to show before the video has loaded. Without it, the hero area is blank during initial load. Use a representative still frame from the video (first or most visually striking frame) exported as a JPEG.

---

## Tips

- **Best source clips:** 4–10 seconds, clear subject, smooth motion with no hard cuts. Abrupt motion reversal looks awkward.
- **Avoid:** clips that end on a very different composition from where they start — the boomerang join will be noticeable even though it's technically seamless.
- **Performance:** strip audio (`-an`) and use `-crf 20` or higher. Background videos are rarely watched frame-by-frame — quality can be slightly lower than foreground content.
- **Mobile:** always include `playsinline` — without it, iOS Safari opens the video fullscreen instead of playing inline.

---

## License

MIT — see [LICENSE](../LICENSE)
