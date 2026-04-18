---
name: hero-loop-background
description: Create a full-bleed hero section with a looping boomerang background video that plays forward then reverses seamlessly. Use when the user wants a video hero background, boomerang loop, hero section video, cinematic hero, animated landing page background, seamless video loop, or video background that doesn't hard-cut on loop. Handles ffmpeg boomerang encoding (concat forward + reversed), autoplay video element with object-cover, gradient overlays for text legibility, and poster image fallback.
user-invocable: true
---

# Hero Boomerang Video

This skill creates a full-bleed hero section backed by a looping boomerang video — the clip plays forward, then reverses, creating a seamless loop with no hard cut. Gradient overlays are calibrated so hero text remains readable regardless of which frame is playing.

**Requires ffmpeg on PATH.**

---

## Step 0 — Gather Inputs

Confirm these with the user before starting:

- **`VIDEO_PATH`** — absolute path to the source video file
- **`OUTPUT_DIR`** — where to write the encoded video (default: `assets/`)
- **`HEADLINE`** — the hero headline text (needed to calibrate overlay contrast)
- **`TEXT_COLOR`** — `light` (white text on dark video) or `dark` (dark text on light video). Default: `light`
- **`POSTER_PATH`** — optional fallback image shown while video loads. If not provided, skip the `poster` attribute.
- **`HERO_HEIGHT`** — default `100vh` (full viewport). Change for partial-height heroes.

---

## Step 1 — Probe the Source Video

```bash
ffprobe -v quiet -print_format json -show_streams "VIDEO_PATH" 2>&1
```

Note: `width`, `height`, `r_frame_rate`, `duration`. The boomerang will be `duration × 2` seconds long.

---

## Step 2 — Create the Boomerang

The boomerang filter reverses the clip in memory and concatenates it after the original: `forward → reverse`. At the loop point, the last frame of the reversed segment is identical to the first frame of the forward segment — a truly seamless loop.

```bash
ffmpeg -y -i "VIDEO_PATH" \
  -an \
  -filter_complex "[0:v]reverse[rv];[0:v][rv]concat=n=2:v=1[outv]" \
  -map "[outv]" \
  -crf 20 \
  -preset slow \
  -movflags +faststart \
  "OUTPUT_DIR/hero-bg.mp4"
```

**Flag notes:**
- `-an` — strip audio (background video is muted; no need to encode audio)
- `[0:v]reverse[rv]` — creates a reversed copy of the video stream in the filter graph
- `concat=n=2:v=1` — concatenates the original + reversed streams
- `-crf 20` — slightly lower quality than scrub video is fine here (always playing, never paused on a frame)
- `-movflags +faststart` — moov atom at front so autoplay starts before the full file loads

**Memory note:** The `reverse` filter must load the entire video into RAM. For clips longer than ~30 seconds, consider trimming first with `-t 8` to keep only the most interesting portion.

---

## Step 3 — HTML Hero Section

Replace placeholder values with the user's actual headline, subtext, and button copy:

```html
<!-- ═══════════════════════════════════════════
     HERO SECTION
═══════════════════════════════════════════ -->
<section class="relative overflow-hidden flex items-center justify-center" style="height: HERO_HEIGHT;">

  <!-- Background video -->
  <div class="absolute inset-0 z-0">
    <video
      autoplay
      muted
      loop
      playsinline
      poster="POSTER_PATH"
      class="w-full h-full object-cover"
    >
      <source src="OUTPUT_DIR/hero-bg.mp4" type="video/mp4">
    </video>

    <!-- Gradient overlays for text legibility -->
    <!-- OVERLAY_BLOCK -->
  </div>

  <!-- Hero content -->
  <div class="relative z-10 text-center px-6 max-w-4xl mx-auto">
    <h1>HEADLINE</h1>
    <p>SUBTEXT</p>
    <!-- CTA button if needed -->
  </div>

</section>
```

Omit `poster="POSTER_PATH"` if no poster image was provided.

---

## Step 4 — Gradient Overlays

Choose the overlay block based on `TEXT_COLOR`:

### Light text on dark video (`TEXT_COLOR=light`)

```html
<!-- Top-to-bottom dark gradient -->
<div class="absolute inset-0" style="background: linear-gradient(to bottom, rgba(0,0,0,0.55) 0%, rgba(0,0,0,0.15) 40%, rgba(0,0,0,0.15) 60%, rgba(0,0,0,0.75) 100%);"></div>
<!-- Radial vignette — darkens edges, keeps centre clear -->
<div class="absolute inset-0" style="background: radial-gradient(ellipse at 50% 45%, transparent 30%, rgba(0,0,0,0.45) 100%);"></div>
```

### Dark text on light video (`TEXT_COLOR=dark`)

```html
<!-- Soft white wash -->
<div class="absolute inset-0" style="background: linear-gradient(to bottom, rgba(255,255,255,0.45) 0%, rgba(255,255,255,0.10) 40%, rgba(255,255,255,0.10) 60%, rgba(255,255,255,0.65) 100%);"></div>
<div class="absolute inset-0" style="background: radial-gradient(ellipse at 50% 45%, transparent 30%, rgba(255,255,255,0.35) 100%);"></div>
```

### Custom brand colour overlay

If the site has a primary brand colour (e.g. `#061b0e`), use it instead of black/white for a more cohesive blend:

```html
<div class="absolute inset-0" style="background: linear-gradient(to bottom, BRAND_COLOR_60 0%, BRAND_COLOR_20 40%, BRAND_COLOR_20 60%, BRAND_COLOR_85 100%);"></div>
```

where `BRAND_COLOR_60` = `rgba(R,G,B,0.60)` etc.

**Contrast check:** The headline must pass WCAG AA (4.5:1 for normal text, 3:1 for large text). Take a screenshot and visually verify the text is clearly readable across light and dark frames of the video. If not, increase the alpha values on the gradient stops by 0.10–0.15 and re-check.

---

## Step 5 — Screenshot Verification (Two Passes)

**Pass 1 — baseline** (before any overlay adjustments):

```bash
npx playwright screenshot --browser chromium \
  --viewport-size "1440,900" \
  --wait-for-timeout 1500 \
  "file:///ABSOLUTE_PATH/index.html" \
  "hero-pass1-desktop.png"

npx playwright screenshot --browser chromium \
  --viewport-size "390,844" \
  --wait-for-timeout 1500 \
  "file:///ABSOLUTE_PATH/index.html" \
  "hero-pass1-mobile.png"
```

**Pass 1 checklist:**
- [ ] Video fills the hero edge-to-edge (no letterboxing, no colour bars)
- [ ] Headline text is readable — clear contrast against the video frame
- [ ] No hard-cut visible (the boomerang should loop smoothly at the ~5s mark)
- [ ] Poster image showed before video loaded (check Network throttling in DevTools)

**Pass 2 — after any overlay/contrast adjustments:**
- [ ] Same viewports
- [ ] Headline still readable on both desktop and mobile
- [ ] Gradient doesn't overpower the video (video content should still be clearly visible)

---

## Tuning Reference

| Parameter | Effect | Recommended range |
|-----------|--------|-------------------|
| Gradient alpha top | Readability of nav / top text | `0.40` – `0.65` |
| Gradient alpha bottom | Readability of CTA / bottom text | `0.65` – `0.90` |
| Radial vignette alpha | Edge darkening | `0.35` – `0.55` |
| `-crf` | Quality vs file size | `18` (high) – `24` (smaller) |
| Clip duration | Boomerang length = `duration × 2` | 4–10s source → 8–20s loop |

---

## Troubleshooting

**"reverse filter ran out of memory"**
The source clip is too long. Trim it first:
```bash
ffmpeg -i "VIDEO_PATH" -t 8 -c copy trimmed.mp4
```
Then run the boomerang encode on `trimmed.mp4`.

**Video doesn't autoplay**
Browsers require `muted` for autoplay. Confirm the `<video>` element has both `autoplay` and `muted` attributes. On iOS, `playsinline` is also required.

**Hard cut visible on loop**
The boomerang is inherently seamless (last reversed frame = first forward frame). If a cut is visible, the source clip likely has a hard transition near its end. Trim a few frames from the end of the source with `-t DURATION-0.5` before re-encoding.

**Poster image not showing**
The `poster` attribute path must be resolvable from the HTML file's location. Use a relative path or absolute URL.
