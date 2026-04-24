# CLAUDE.md — Web Design

A general operating manual for any web design project. Project-specific details (stack, brand, file structure) belong in a separate CLAUDE.md layered on top of this one.

---

## Non-negotiables

1. **Two-pass screenshot verification** for every visual change. Reading code is not seeing the result.
2. **Parallel execution** for independent file work — components, styles, tests, and stories ship in one batch.
3. **Use the design tools you actually have.** See the Tools section below before defaulting to "I'll just write it blind."

---

## Two-pass screenshot workflow

Whenever you make a visual change, add a component, or are asked to evaluate how something looks, screenshot the rendered output **twice**.

**Pass 1 — Baseline.** Render the current state. Capture at desktop (1440×900) and mobile (390×844). Write 3–5 specific observations: what works, what breaks, what feels off. This pass is diagnostic — do not skip to fixing.

**Pass 2 — Verification.** Apply the change. Re-screenshot at the same viewports as Pass 1. Compare directly. Did the fix land? Any regressions? If Pass 2 doesn't resolve what Pass 1 revealed, iterate — don't ship.

**When this is required:** after creating or modifying any component, after the user says "this doesn't look right," before declaring any task complete, and any time you're evaluating your own visual work.

### Screenshot storage (mandatory)

All screenshots go in `.screenshots/` at the project root. This folder is **disposable and user-owned**.

- Add `.screenshots/` to `.gitignore` on first use.
- One subfolder per task: `.screenshots/<task-slug>/`
- Filename format: `pass-<1|2>-<viewport>-<timestamp>.png`
  - Example: `.screenshots/hero-redesign/pass-2-mobile-2026-04-24T14-31.png`
- Never overwrite — timestamps preserve iteration history.
- **Never auto-delete this folder.** The user clears it with `rm -rf .screenshots/` whenever they want.
- Create the folder if it doesn't exist before taking the first shot of a session.
- Never save screenshots anywhere else (not `src/`, not `public/`, not the project root).

---

## Tools you should be reaching for

Before writing visual code blind, check what's available in the environment:

- **Chrome DevTools MCP** — the primary tool for screenshotting, inspecting, navigating, and debugging rendered pages. Use it for both passes of the screenshot workflow. If it's connected, always prefer it over CLI alternatives.
- **`frontend-design` skill** at `/mnt/skills/public/frontend-design/SKILL.md` — read this before building any web UI. It encodes design tokens, component patterns, and styling constraints specific to this environment. Loading it is not optional for design work.
- **Image search / web fetch** — pull real design references before composing a layout. Don't invent from priors.
- **Lighthouse** — run after meaningful changes; capture the score into Pass 2.

If Chrome DevTools MCP isn't connected, say so explicitly rather than skipping the verification step.

---

## Design quality principles

Beautiful sites are composed, not decorated.

**Typography.** Two typefaces max (or one superfamily). Modular scale (1.2–1.333 ratio). Body line-height 1.5–1.7, display 1.1–1.25. Body measure 45–75 characters.

**Color.** Tokens, not scattered hex values. Minimum: bg, surface, text, text-muted, border, accent, accent-hover. Dark mode from day one. Body text contrast ≥ 4.5:1.

**Spacing.** Scale on a 4px or 8px base (4, 8, 12, 16, 24, 32, 48, 64, 96, 128). No arbitrary values like `margin: 23px`. Whitespace is a design element — if it feels cramped, it is.

**Layout.** Grid first, flexbox for components. One dominant element per section. Mobile-first.

**Motion.** Default subtle. 150–250ms for micro-interactions, 300–500ms for transitions. `ease-out` in, `ease-in` out. Always respect `prefers-reduced-motion`. Motion reinforces hierarchy or feedback — never decoration.

**Detail.** Consistent radius scale. Elevation system, not one-off shadows. Visible focus state on every interactive element. Loading, empty, and error states designed — not afterthoughts.

**Avoid the generic AI aesthetic:** center-aligned purple gradients, rounded-full everything, emoji-heavy headers, uniform card grids, lorem-ipsum spacing. Be specific and considered.

---

## Accessibility floor (non-optional)

Semantic HTML first — `<button>` is not a `<div onClick>`. All images have `alt` (empty for decorative). Forms have labels, not just placeholders. Keyboard navigation works end-to-end. Focus indicators visible and high-contrast. ARIA only when HTML semantics can't express intent. Target WCAG 2.1 AA. Run `axe` or Lighthouse a11y before declaring done.

---

## Parallel execution pattern

When creating a component, batch the entire ecosystem in one tool call — never sequentially:

```
[BatchTool]:
  - Write("components/Hero/Hero.tsx", ...)
  - Write("components/Hero/Hero.module.css", ...)
  - Write("components/Hero/Hero.test.tsx", ...)
  - Write("components/Hero/index.ts", ...)
```

Same applies to design tokens, related components, and test suites that don't depend on each other.

---

## The working loop

For every web design task:

1. **Understand** — what's changing, what tokens/components are affected.
2. **Read the `frontend-design` skill** if not already loaded this session.
3. **Plan in parallel** — list every file to create/edit in one batch.
4. **Build** — execute the batch.
5. **Prepare** — ensure `.screenshots/<task-slug>/` exists.
6. **Pass 1 screenshots** — desktop + mobile. Write observations.
7. **Refine** based on Pass 1.
8. **Pass 2 screenshots** — same viewports. Compare directly.
9. **Iterate** if Pass 2 isn't there. Don't ship early.
10. **Verify** — keyboard nav, a11y check, Lighthouse if relevant.
11. **Summarize** — what shipped, what the screenshots show (reference the path), what's deferred.

---

## What this file is NOT for

Backend, database, auth, API, DevOps. Project-specific stack choices, brand tokens, file layout. Decision logs. Those belong in a project-level CLAUDE.md layered on top of this one — keep that file short too.
