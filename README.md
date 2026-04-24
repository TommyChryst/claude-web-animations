# claude-web-design

A collection of Claude Code skills for building high-quality web design effects — built from real production work on a luxury watch brand site.

---

## Skills

### [`scroll-reveal-video`](./scroll-reveal-video/)

Turn any video into a scroll-driven animation. The video's playback position maps directly to scroll progress — each frame reveals as the user scrolls. Includes optional animated text overlays that fade in and out at scroll positions you define.

**Good for:** product explodes, process reveals, timeline animations, cinematic storytelling sections

### [`hero-loop-background`](./hero-loop-background/)

Full-bleed hero section backed by a seamless boomerang video loop (forward → reverse, no hard cut). Gradient overlays are calibrated so headline text stays readable across all video frames.

**Good for:** landing page heroes, product launches, brand intros, any hero that needs motion without distraction

---

## Prerequisites

Both skills require:

- **[Claude Code](https://claude.ai/code)** CLI installed and running
- **[ffmpeg](https://ffmpeg.org/download.html)** available on `PATH`
- **[Node.js](https://nodejs.org/)** for Playwright screenshot verification

---

## Installation

Copy the skill folder(s) you want into your Claude skills directory:

```bash
# macOS / Linux — install both
cp -r scroll-reveal-video hero-loop-background ~/.claude/skills/

# Windows (PowerShell) — install both
Copy-Item -Recurse scroll-reveal-video "$env:USERPROFILE\.claude\skills\"
Copy-Item -Recurse hero-loop-background "$env:USERPROFILE\.claude\skills\"
```

Claude Code picks up skills automatically — no restart needed.

---

## Usage

Just describe what you want. Claude will recognise the intent and load the right skill:

```
Add a scroll-reveal video section using assets/explode.mp4
```
```
Create a hero with a boomerang loop from this video: product.mp4
```
```
I want the video to animate as I scroll — use footage/watch.mp4
```

---

## How these were built

Both skills were developed while building a luxury watch brand website. The techniques were validated end-to-end:

- **scroll-reveal-video** came from the challenge of making a 121-frame watch explosion video scrub smoothly without blocking artifacts or grain. The key insight: H.264 inter-frame compression causes artifacts at non-keyframe seek positions — all-I-frame encoding eliminates this entirely.

- **hero-loop-background** came from wanting a dynamic hero background that loops cleanly. The ffmpeg `reverse` filter + `concat` creates a mathematically seamless loop that no easing or crossfade can match.

---

## File structure

```
claude-web-design/
├── README.md
├── CLAUDE.md                        ← web design operating manual
├── LICENSE
├── scroll-reveal-video/
│   ├── SKILL.md                     ← Claude skill instructions
│   └── README.md                    ← documentation
└── hero-loop-background/
    ├── SKILL.md
    └── README.md
```

---

## License

MIT — see [LICENSE](./LICENSE)
