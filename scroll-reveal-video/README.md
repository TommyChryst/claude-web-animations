# scroll-reveal-video

A Claude Code skill that turns any video into a scroll-driven animation. The video's playback position is mapped directly to scroll progress — each frame reveals as the user scrolls. Optionally adds animated text overlays that fade in and out at defined scroll positions.

---

## What it does

1. **Probes** your source video with `ffprobe` to get resolution, fps, and frame count
2. **Re-encodes** with ffmpeg using all-I-frame encoding (`-g 1`) so every seek position is a clean keyframe — no blocking artifacts
3. **Generates** the HTML sticky-scroll section, CSS, and JavaScript scrub handler
4. **Optionally adds** 2–4 text overlay cards that fade in/out at scroll-fraction positions you define
5. **Verifies** with Playwright screenshots at multiple scroll depths

---

## Prerequisites

- [Claude Code](https://claude.ai/code) CLI installed
- [ffmpeg](https://ffmpeg.org/download.html) available on `PATH`
- [Node.js](https://nodejs.org/) (for Playwright screenshot verification)

---

## Installation

Copy the `scroll-reveal-video` folder into your Claude skills directory:

```bash
# macOS / Linux
cp -r scroll-reveal-video ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse scroll-reveal-video "$env:USERPROFILE\.claude\skills\"
```

Claude Code will pick it up automatically — no restart needed.

---

## Usage

Trigger the skill by describing what you want. Any of these prompts will activate it:

```
Add a scroll-scrub video section using assets/explode.mp4
```
```
Create a cinematic scroll animation from this video: /path/to/video.mp4
```
```
I want the video to scrub as the user scrolls — use hero.mp4
```

Claude will ask for any missing details (section height, text overlay copy, display width) before proceeding.

---

## What gets created

| File | Description |
|------|-------------|
| `scrub-video.mp4` | Re-encoded all-I-frame video, placed in your output directory |
| HTML snippet | `<section id="scrub-section">` with sticky layout and video element |
| CSS | `.scrub-text` overlay styles + optional animated grain keyframes |
| JS | `requestAnimationFrame`-throttled scroll handler with `fastSeek` + delta guard |

---

## Customisation

| Parameter | Default | Description |
|-----------|---------|-------------|
| `SECTION_HEIGHT` | `200vh` | Vertical scroll travel. `300vh` = slower, more time per frame |
| `DISPLAY_WIDTH` | `1920` | Encode width. Set to source width for contained layout |
| `-crf` | `18` | Quality. Range `15` (high quality) → `23` (smaller file) |
| `FADE` | `0.07` | Fraction of scroll range used for text card fade in/out |
| Text overlay Y lift | `14px` | Upward slide distance on text entrance |

---

## How it works

### Why all-I-frame encoding?

H.264 stores most frames as "difference frames" (P/B frames) that encode only what changed from the previous keyframe. When `video.currentTime` seeks to a difference frame, the browser must reconstruct it from the nearest keyframe — which produces visible blocking artifacts on the static frame.

Setting `-g 1 -keyint_min 1` makes every frame a full keyframe (I-frame). Any seek position decodes cleanly. File size increases ~2–4× but the scrubbing is artifact-free.

### Why `fastSeek()` + delta guard?

`video.fastSeek(t)` uses the browser's optimised seek path (available in Firefox; falls back gracefully on Chrome). The `lastTime` delta guard skips seeks when the target time hasn't changed by more than 1ms — preventing redundant decode calls when the user scrolls slowly or pauses.

### Why animated grain?

A static noise overlay (`body::before` with SVG feTurbulence) creates a "screen door" effect when content moves beneath it. Adding `animation: grain 0.8s steps(1) infinite` makes the grain snap to a new random position every ~80ms, matching how real film grain behaves and removing the visual tension between the static overlay and the moving video.

---

## Example output

The skill generates a section that looks like this mid-scroll:

```
┌─────────────────────────────────┐
│                                 │
│   [  video frame N at 48%  ]   │
│                                 │
│        Overline Label           │
│    Headline Text Goes Here      │
│                                 │
└─────────────────────────────────┘
```

Text cards slide up 14px and fade in over a 7% scroll window, hold, then fade out before the next card appears.

---

## License

MIT — see [LICENSE](../LICENSE)
