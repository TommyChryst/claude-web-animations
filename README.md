# Claude-Web-Animations

A collection of Claude Code skills for building high-quality web animations and visual effects — built from real production work on a luxury watch brand site. Each skill pairs ffmpeg video processing with production-ready HTML, CSS, and JavaScript.

All skills in this repo follow the [CLAUDE.md](./CLAUDE.md) operating manual: two-pass screenshot verification, parallel execution, and design quality principles that avoid generic AI aesthetics.

---

## Skills

### [`scroll-reveal-video`](./scroll-reveal-video/)

Turn any video into a scroll-driven animation. The video's playback position maps directly to scroll progress — each frame reveals as the user scrolls. Includes optional animated text overlays that fade in and out at defined scroll positions.

**Good for:** product explodes, process reveals, timeline animations, cinematic storytelling sections

### [`hero-loop-background`](./hero-loop-background/)

Full-bleed hero section backed by a seamless boomerang video loop (forward → reverse, no hard cut). Gradient overlays are calibrated so headline text stays readable across all video frames.

**Good for:** landing page heroes, product launches, brand intros, any hero that needs motion without distraction

---

## How skills work

Each skill is a Claude Code slash command — a markdown file with step-by-step instructions that Claude follows when you describe what you want. Claude reads the skill, asks for any missing inputs, then executes the full workflow: ffmpeg encoding, HTML/CSS/JS generation, and two-pass screenshot verification.

Every skill follows the workflow defined in [CLAUDE.md](./CLAUDE.md):

1. Understand what's being built
2. Plan all file changes in parallel
3. Build and encode
4. **Pass 1 screenshots** — desktop (1440×900) + mobile (390×844), write observations
5. Refine based on what Pass 1 reveals
6. **Pass 2 screenshots** — same viewports, compare directly
7. Verify accessibility and performance

Screenshots are saved to `.screenshots/<task-slug>/` and never committed.

---

## Prerequisites

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

- **scroll-reveal-video** — the challenge was making a 121-frame watch explosion video scrub smoothly without blocking artifacts or grain. Root cause: H.264 inter-frame compression produces artifacts when `video.currentTime` seeks to a non-keyframe position. All-I-frame encoding (`-g 1`) eliminates this entirely.

- **hero-loop-background** — the goal was a dynamic hero background that loops with no hard cut. The ffmpeg `reverse` filter + `concat` creates a mathematically seamless forward→reverse loop that no crossfade can replicate.

---

## File structure

```
claude-web-animations/
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
