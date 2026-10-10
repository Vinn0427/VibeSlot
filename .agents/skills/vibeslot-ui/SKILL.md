---
name: vibeslot-ui
description: Use when implementing, styling, reviewing, or refactoring VibeSlot's frontend pages, slot reels, toy-like controls, result cards, bottom sheets, stickers, animations, or responsive layouts. Applies to React/TypeScript/CSS UI work and visual QA, not backend-only changes.
---

# VibeSlot UI — Neo-Brutalist Toy Arcade

## Mission

Build **a tangible, playful pocket toy**, not a dashboard, casino, ad landing page, or generic component-demo site.

The approved direction is **Neo-Brutalism × hand-drawn sticker art × physical toy controls**:
- Bold blocks of mostly flat color; near-black thick outlines; hard, unblurred offset shadows.
- An off-white foundation, punchy scene identity colors, human hand-drawn accent shapes.
- A coherent toy-machine housing around the reels, unmistakable mechanical feedback, and a large readable result.
- Mobile Web is the **canonical product**. Desktop/tablet show the same interaction in a centered narrow canvas.

Do not implement screens as screenshots or a large flattened bitmap. All labels, reels, locks, buttons, options, and sheets must be real semantic DOM controls. Decorative artwork may be SVG or small optimized image assets.

## When this Skill activates

Use for all VibeSlot frontend tasks including: homepage, scene card, `/play/*`, slot machine, reel, lock buttons, result ticket, filter/optional reel, editing page, import page, sheet/dialog, share entry, theme tokens, styling, animation, icons, accessibility and responsive QA. Skip for isolated backend/API work with no visible UI effect.

## Source of truth / conflict resolution

Read the project documentation **before editing related UI**. The project files, when present, outrank this Skill for product semantics:

1. `PRODUCT_VISION.md`: product principles, especially *像游戏，不像赌博* and mobile-first.
2. `PRD.md`: required behavior, allowed content, state machine, button labels, performance, share workflow and accessibility.
3. `CONTENT_DESIGN.md`: canonical scene labels, content groups, sentence templates, notices, notes. **Do not invent, paraphrase, or replace option labels.**
4. `UX_DESIGN.md`: layout, component placement, reel structure, timings, and interaction specs.
5. **This Skill**: visual implementation system, production constraints and visual QA.

If a design reference conflicts with any written rule, the written rule wins. If docs conflict with each other, use their stated hierarchy (Vision > PRD > UX; Content owns preset copy), flag the conflict and avoid silently changing product decisions.

The reference images are **concept art, not verified UI specifications**. Some show sample copy, icons in reels, unusually tall toy cabinets, decorative levers or lock labels that differ from documented behavior; do not copy these mistakes.

## Load supporting references only when relevant

| Task | Read |
|---|---|
| Any new screen/theme/illustration | `references/visual-system.md` |
| Homepage/reels/result card/sheets/editor | `references/pages-and-components.md` |
| Spinning, responsiveness, accessibility, performance | `references/motion-responsive.md` |
| UI implementation or review completion | `references/review-checklist.md` |
| Installing and keeping the Skill triggered | `references/integration.md` |

Art direction references (local to this Skill):
- `assets/ui-references/01-home.png` — brand, home cards, color blocks.
- `assets/ui-references/02-cook.png` — unified cabinet, three reels, result hierarchy.
- `assets/ui-references/03-couple.png` — pink variation, intensity selector, game actions.
- `assets/ui-references/04-party-settings.png` — yellow variation, bottom-sheet switches.

Use these images to understand **visual density, outlines, stickers, palette and toy feeling**. Do not use them as CSS backgrounds; reconstruct with components.

## Mandatory implementation workflow

1. **Inspect**: identify target screen, `PRD` interaction mode (`decision`/`game`), state (`idle`/`spinning`/`result`/`accepted`), allowed reels (1–4), optional controls, real content labels, and responsive width.
2. **Check consistency**: inspect existing tokens/components first; reuse them. No one-off new colors, shadow styles, fonts or corner radii unless a genuine new semantic state requires it.
3. **Sketch structure**: preserve a clear hierarchy: slot cabinet → result ticket → bottom primary action. Place rarely-used controls in the header or bottom sheets; avoid competing hero elements.
4. **Implement as real UI**: semantic buttons and accessible labels, responsive CSS, SVG stickers only where purposeful. Do not embed entire screens as images. Do not let drawings become part of user content labels.
5. **Implement states**: all documented states and lock/filter permutations must be visually understandable. A polished static happy-path alone is insufficient.
6. **Verify motion/performance**: result decided before reel animation; sequential stops, reduced motion, background-tab recovery, and no unwanted layout shifts.
7. **Visually QA** at widths 320, 375, 430, 768 and desktop; check at least one short viewport (375×667), one four-reel case, long Chinese labels and the largest bottom sheet.
8. **Report**: summarize pages/components changed, any deviations from tokens/UX docs, verification actually run, and unresolved issues. Never claim browser/visual QA if not performed.

## Non-negotiable design rules

- **Mobile-first**: target 375px design width, support 320px; desktop stays centered at max 480px. Don't build a second dashboard-like desktop layout.
- **One toy, one focus**: hero is the **unified reel cabinet**. Result sentence becomes the primary focus **after** spinning. The persistent bottom action is immediately identifiable.
- **Visual grammar**: solid colors + 2–4px black outlines + hard offset shadows, no frosted glass, gradients, glossy 3D chrome, blurry glows or soft neumorphic cards.
- **Decorative restraint**: stickers, hearts, stars, food doodles, hand-drawn bursts are accents, not UI labels. Avoid clutter at 320px.
- **Scene consistency**: one dominant scene color and at most two secondary accent colors on an individual play page; homepage may show all five.
- **No casino cues**: avoid `777`, cherries, `BAR`, coins, chips, betting, jackpot, winning language, red-gold casino decor, metallic gambling-machine realism. Decorative toy handles from concept art must not displace the primary button or imply gambling.
- **No generic SaaS**: avoid purple/blue gradients, glass cards, endless rounded white cards, sidebars, abstract AI glow and dashboard widgets.
- **Text is product content**: short, readable Chinese reel labels; show option `label` as authored. Ellipsize inside narrow reels, show full text in result ticket.
- **Correct state over attractive mockup**: in `idle` the preview is not a result; only selected reels spin; locked reel stays frozen; `spinning` disables specified controls; skip/“换一个” is always accessible in game mode.
- **No extras without authorization**: V0.1 has no sound/vibration, virtual currency, special prizes, AI-generated dishes, login or social feed.
- **Performance**: animate `transform`/`opacity`; target 60fps and ≤2.5s per full spin; respect `prefers-reduced-motion`.
- **Accessibility**: ≥44×44px touch targets; clear `:focus-visible`; readable contrast, logical tab order, accessible result announcements, safe-area-aware bottom actions.

## Defaults when no decision was specified

- Background off-white; outlines near black; box-shadow is a **hard 4–5px offset**, no blur.
- Use native CJK system fonts for **interactive text**, heavy weight for headings/results. Give brand wordmarks a more playful SVG treatment if available; avoid large downloaded Chinese font packages.
- 2D illustrated SVG iconography with rounded ends and bold strokes; avoid platform-dependent emoji as production icons.
- Reels display three vertical text rows (previous / current / next), with the centered active row emphasized. Build one scene-colored cabinet enclosing all reels.
- Result ticket is off-white, bold and typographic; emphasize selected words using a restrained scene accent/highlight.
- Use icon + text for lock controls; distinguish lock states through text, outline and color, never color alone.

## Handoff quality bar

Reject a UI implementation as visually incomplete if any of these apply:
- It can be mistaken for a casino or generic SaaS interface.
- Reels are just unrelated picker cards without a shared toy housing.
- The hero slot animation is polished but locked/filter/reduced-motion states are broken.
- At 320px, a four-reel layout clips controls or produces horizontal page scroll.
- Large decorative images replace real controls or obscure important copy.
- The result sentence is hard to read from an arm's length or the primary action is unreachable with one thumb.
- A screenshot looks good but the page overflows or actions fail on a short phone viewport.

Proceed with the simplest implementation that meets these criteria; avoid adding new dependencies solely for decorative effects.
