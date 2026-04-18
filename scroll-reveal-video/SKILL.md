---
name: scroll-reveal-video
description: Create a scroll-driven video scrub section where a video's playback position is controlled by scroll progress. Use when the user wants a scroll-scrub animation, cinematic scroll effect, video timeline scrubbing, parallax video, scroll-synced video, or wants to sync a video to scroll progress. Handles ffmpeg re-encoding for clean frame seeking, sticky scroll layout, progress-mapped currentTime, optional animated text overlays, and GPU-promoted CSS.
user-invocable: true
---

# Scroll-Scrub Video

This skill turns any video into a scroll-driven animation: the video's playback position is mapped directly to scroll progress, so each frame reveals as the user scrolls. Includes optional animated text overlays that fade in and out at defined scroll positions.

**Requires ffmpeg on PATH.**

---

## Step 0 — Gather Inputs

Before starting, confirm these with the user if not already provided:

- **`VIDEO_PATH`** — absolute path to the source video file
- **`OUTPUT_DIR`** — directory where the encoded video and HTML will be saved (default: same directory as the source video)
- **`SECTION_HEIGHT`** — how much vertical scroll travel the section takes up (default: `200vh`; use `300vh` for more time per text card)
- **`DISPLAY_WIDTH`** — max display width in px (default: `1920` for full-bleed `object-cover`; use `1280` for a contained layout)
- **`TEXT_OVERLAYS`** — optional list of 2–4 text cards. For each, ask: label (overline), headline, scroll start fraction (0–1), scroll end fraction (0–1). Skip if the user wants video only.

---

## Step 1 — Probe the Source Video

```bash
ffprobe -v quiet -print_format json -show_streams "VIDEO_PATH" 2>&1
```

From the output, note:
- `width` × `height` — source resolution
- `r_frame_rate` — frame rate (e.g. `"24/1"`)
- `nb_frames` — total frame count
- `duration` — total seconds
- `codec_name` — source codec

If the source width is already ≤ `DISPLAY_WIDTH`, set `ENCODE_WIDTH` to the source width (never upscale). Otherwise set `ENCODE_WIDTH` to `DISPLAY_WIDTH`.

---

## Step 2 — Re-encode for Smooth Scrubbing

All-I-frame encoding means every frame is a full keyframe. This eliminates the blocking artifacts that appear when `video.currentTime` seeks to a B/P inter-frame — the root cause of graininess and stuttering in scroll-scrub effects.

```bash
ffmpeg -y -i "VIDEO_PATH" \
  -an \
  -vf scale=ENCODE_WIDTH:-2:flags=lanczos \
  -g 1 -keyint_min 1 \
  -crf 18 \
  -preset slow \
  -profile:v high \
  -movflags +faststart \
  "OUTPUT_DIR/scrub-video.mp4"
```

**Flag notes:**
- `-an` — strip audio (not needed for scrubbing, reduces file size)
- `-g 1 -keyint_min 1` — every frame is an I-frame; mandatory for clean seeking
- `-vf scale=W:-2:flags=lanczos` — Lanczos downscale to display width; `-2` keeps height divisible by 2
- `-crf 18` — high quality; increase to `23` to trade quality for smaller file
- `-movflags +faststart` — move moov atom to front so browser can seek before full download

Confirm the encode completed: check `frame I:N` in the output matches the source frame count.

---

## Step 3 — HTML Section

Insert this sticky scroll section into the page. Replace `SECTION_HEIGHT`, `BG_COLOR`, and the video `src`:

```html
<!-- ═══════════════════════════════════════════
     SCROLL-SCRUB SECTION
═══════════════════════════════════════════ -->
<section id="scrub-section" class="relative" style="height: SECTION_HEIGHT; background-color: BG_COLOR;">
  <div class="sticky top-0 h-screen w-full overflow-hidden flex items-center justify-center" style="background-color: BG_COLOR;">

    <!-- Scrub video -->
    <video
      id="scrub-video"
      muted
      playsinline
      preload="auto"
      class="w-full h-full object-cover"
      style="will-change: transform;"
    >
      <source src="scrub-video.mp4" type="video/mp4">
    </video>

    <!-- Top/bottom vignette — blend edges into surrounding sections -->
    <div class="absolute inset-0 pointer-events-none" style="background: linear-gradient(to bottom, BG_COLOR 0%, transparent 12%, transparent 88%, BG_COLOR 100%);"></div>

    <!-- Text overlays (add/remove as needed) -->
    <!-- OVERLAY_BLOCKS -->

  </div>
</section>
```

**For full-bleed (`object-cover`):** the video fills the entire viewport. Use `DISPLAY_WIDTH=1920` encode so the browser always downscales — never upscales — into the viewport.

**For contained layout:** use `class="max-w-[1280px] w-full h-auto"` on the video element instead of `w-full h-full object-cover`, and encode at the native source width.

---

## Step 4 — CSS

Add to the page's `<style>` block:

```css
/* ── Scrub text overlays ─────────────────────── */
.scrub-text {
  position: absolute;
  bottom: 5rem;
  left: 50%;
  width: max-content;
  max-width: min(560px, 90vw);
  text-align: center;
  opacity: 0;
  will-change: opacity, transform;
  pointer-events: none;
}

/* ── Animated film-grain overlay (optional) ──── */
@keyframes grain {
  0%   { transform: translate(0,    0);   }
  10%  { transform: translate(-2%,  -3%); }
  20%  { transform: translate(3%,   1%);  }
  30%  { transform: translate(-1%,  3%);  }
  40%  { transform: translate(2%,  -1%);  }
  50%  { transform: translate(-3%,  2%);  }
  60%  { transform: translate(1%,   3%);  }
  70%  { transform: translate(3%,  -2%);  }
  80%  { transform: translate(-2%,  1%);  }
  90%  { transform: translate(2%,  -3%);  }
  100% { transform: translate(0,    0);   }
}
```

If the page uses a grain texture overlay (`body::before` with SVG noise), add `animation: grain 0.8s steps(1) infinite;` to it. A static grain over moving video looks gritty; an animated grain looks cinematic.

---

## Step 5 — Text Overlay HTML

For each text card the user provided, generate a `<div class="scrub-text">` inside the sticky container:

```html
<div class="scrub-text z-10" data-start="START" data-end="END">
  <p class="overline-style">LABEL</p>
  <p class="headline-style">HEADLINE</p>
</div>
```

Replace `START` and `END` with the scroll fractions (e.g. `0.05` and `0.30`).

**Suggested fractions for 3 cards:**
| Card | data-start | data-end |
|------|-----------|---------|
| 1    | 0.04      | 0.30    |
| 2    | 0.34      | 0.62    |
| 3    | 0.66      | 0.92    |

Style the text to contrast against the video. Light text on dark video: use `color: rgba(255,255,255,0.9)`.

---

## Step 6 — JavaScript Scrub Handler

Add inside a `<script>` tag (or the page's JS file):

```javascript
(function () {
  const video   = document.getElementById('scrub-video');
  const section = document.getElementById('scrub-section');
  if (!video || !section) return;

  const texts  = Array.from(section.querySelectorAll('.scrub-text'));
  const FADE   = 0.07;   // fraction of scroll range used for fade-in / fade-out
  const seek   = video.fastSeek
    ? t => video.fastSeek(t)
    : t => { video.currentTime = t; };
  let rafId    = null;
  let lastTime = -1;

  function scrub() {
    const rect       = section.getBoundingClientRect();
    const scrollable = section.offsetHeight - window.innerHeight;
    const progress   = Math.min(1, Math.max(0, -rect.top) / scrollable);

    // Seek video — skip if target hasn't changed meaningfully
    if (video.readyState >= 1 && video.duration) {
      const t = progress * video.duration;
      if (Math.abs(t - lastTime) > 0.001) {
        seek(t);
        lastTime = t;
      }
    }

    // Drive text overlay opacity + Y offset from scroll progress
    texts.forEach(el => {
      const start = parseFloat(el.dataset.start);
      const end   = parseFloat(el.dataset.end);
      let opacity = 0;
      if (progress > start && progress < end) {
        const fadeIn  = Math.min(1, (progress - start) / FADE);
        const fadeOut = Math.min(1, (end - progress) / FADE);
        opacity = Math.min(fadeIn, fadeOut);
      }
      el.style.opacity   = opacity;
      el.style.transform = `translateX(-50%) translateY(${(1 - opacity) * 14}px)`;
    });

    rafId = null;
  }

  window.addEventListener('scroll', () => {
    if (!rafId) rafId = requestAnimationFrame(scrub);
  }, { passive: true });

  video.addEventListener('loadedmetadata', scrub);
})();
```

**Why this works:**
- `requestAnimationFrame` throttling prevents scroll handler from firing faster than the display refresh rate
- `fastSeek()` uses the browser's optimised seek path where available (Firefox); falls back to `currentTime` on Chrome
- The `lastTime` delta guard prevents redundant seeks when the user pauses mid-scroll
- `passive: true` on the scroll listener avoids blocking the browser's scroll thread

---

## Step 7 — Screenshot Verification (Two Passes)

Take screenshots at three scroll depths to verify each text card and the video frame quality.

**Using Playwright CLI:**

```bash
# Install once if needed
npx playwright install chromium

# For each scroll depth (15%, 48%, 78%)
node -e "
const fs = require('fs');
const html = fs.readFileSync('index.html', 'utf8').replace('</body>', \`
<script>
window.addEventListener('load', () => {
  const s = document.getElementById('scrub-section');
  window.scrollTo(0, s.offsetTop + (s.offsetHeight - window.innerHeight) * PROGRESS);
});
</script>
</body>\`);
fs.writeFileSync('_preview.html', html);
" && npx playwright screenshot --browser chromium \
     --viewport-size "1440,900" \
     --wait-for-timeout 1500 \
     "file:///ABSOLUTE_PATH/_preview.html" \
     "screenshot-PROGRESS.png" \
  && rm _preview.html
```

Replace `PROGRESS` with `0.15`, `0.48`, `0.78`.

**Pass 1 checklist:**
- [ ] Video fills section edge-to-edge (no background bleed)
- [ ] Text overlays are readable against the video frame
- [ ] No visible blocking/compression artifacts on the video frame
- [ ] Vignette blends top/bottom edges into surrounding sections

**Pass 2 — after any adjustments:**
- [ ] Same viewports, same scroll depths
- [ ] Compare directly: did the fix land without introducing regressions?

---

## Tuning Reference

| Parameter | Effect | Recommended range |
|-----------|--------|-------------------|
| `SECTION_HEIGHT` | Scroll travel (more = slower animation) | `200vh` – `400vh` |
| `FADE` (JS) | Fade window per text card | `0.05` – `0.10` |
| Y lift (14px) | Slide distance on text entrance | `8px` – `20px` |
| `-crf` | Quality vs file size | `15` (high) – `23` (small) |
| `ENCODE_WIDTH` | Display sharpness | Match or below source width |
