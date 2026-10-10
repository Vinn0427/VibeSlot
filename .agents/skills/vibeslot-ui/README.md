# VibeSlot UI Skill

A local Agent/Codex skill for VibeSlot's **Neo-Brutalism × sticker illustration × tangible toy** visual system.

## What it does

Constrain UI agents to a coherent mobile-first implementation: flat bold colors, hard black outlines/shadows, a unified toy slot cabinet, accurate states/copy, readable result tickets, fast staggered reel motion, and a narrow centered desktop canvas.

## Install

Copy this entire folder to:

```text
<your-project>/.agents/skills/vibeslot-ui/
```

The repo's `AGENTS.md` already contains the trigger snippet from `references/integration.md`. The VibeSlot design documents live in `design/`:

- `design/PRODUCT_VISION.md`
- `design/PRD.md`
- `design/CONTENT_DESIGN.md`
- `design/UX_DESIGN.md`

Read `SKILL.md` first. Only read the reference file relevant to the coding task to save context.

## Visual references

`assets/ui-references/01-home.png` through `04-party-settings.png` are the design direction the user approved; `01-home.png` is additionally confirmed as the homepage target look (see `design/UX_DESIGN.md` §4). They are **generated conceptual images**, not screenshots of functional software and not precise content/state specifications.

## Boundaries

The Skill guides **how to implement the UI**, not which product features are authorized. Respect the product documentation over illustration details or Skill proposals. All design token hex values in `references/visual-system.md` are practical starting points, pending real-device visual review.

## Example use

> Use `$vibeslot-ui` to build the cook play page according to the project PRD and UX docs. Implement the unified reel cabinet and result ticket as real React components, verify 320px and 375px layouts, and explain any deviations from the concept illustration.
