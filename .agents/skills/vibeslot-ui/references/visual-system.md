# Visual system: Sticker Arcade / 玩具机台

## 1. Brand vocabulary

**Descriptors:** bold / friendly / tangible / playful / hand-drawn / poster-like / high-contrast / accessible.

**Physical metaphors:** sticker, plastic toy casing, paper result ticket, button that presses down, reel that lands in place, a stamped decision, a hinged sheet.

Use controlled imperfection in *illustrations* (tiny rotations, uneven-looking hand-drawn strokes, doodle bursts), but exact grids for *layout* (equal reel widths, aligned columns, reliable hitboxes).

## 2. Tokens — recommended starting points

These are proposed implementation values, not approved product facts. Adjust after testing screenshots on a device; keep all values in shared CSS variables/theme config.

```css
:root {
  /* Base */
  --vs-bg: #fffaf1;
  --vs-surface: #fffefa;
  --vs-ink: #171717;
  --vs-muted-ink: #595959;
  --vs-outline: #171717;
  --vs-disabled: #dedbd4;

  /* Scene identities */
  --vs-cook: #ff704f;    /* tomato orange */
  --vs-date: #ff96b2;    /* peach pink */
  --vs-party: #ffdc4d;   /* lemon yellow */
  --vs-mix: #a8e75f;     /* lime green */
  --vs-couple: #ff597f;  /* rose pink */
  --vs-custom: #478fff;  /* toy blue */

  /* Accents, used sparingly within a play page */
  --vs-accent-blue: #478fff;
  --vs-accent-lime: #b5f179;
  --vs-accent-yellow: #ffdc4d;
  --vs-accent-pink: #ffc0d3;

  /* Form and motion */
  --vs-border-strong: 3px solid var(--vs-outline);
  --vs-border-normal: 2px solid var(--vs-outline);
  --vs-shadow-card: 4px 5px 0 var(--vs-outline);
  --vs-shadow-press: 0 0 0 var(--vs-outline);
  --vs-radius-sm: 12px;
  --vs-radius-md: 18px;
  --vs-radius-lg: 24px;
  --vs-space-1: 4px;
  --vs-space-2: 8px;
  --vs-space-3: 12px;
  --vs-space-4: 16px;
  --vs-space-6: 24px;
  --vs-space-8: 32px;
  --vs-page-padding: 16px;
  --vs-content-max: 480px;
}

[data-scene='cook'] { --vs-scene: var(--vs-cook); }
[data-scene='date'] { --vs-scene: var(--vs-date); }
[data-scene='party'] { --vs-scene: var(--vs-party); }
[data-scene='mix'] { --vs-scene: var(--vs-mix); }
[data-scene='couple'] { --vs-scene: var(--vs-couple); }
[data-scene='custom'] { --vs-scene: var(--vs-custom); }
```

Before shipping, measure contrast for EVERY text/background combination; the colors above are illustrations, not guarantees of WCAG compliance. Prefer dark text over yellow/pink/lime and avoid light body text on saturated color.

## 3. Allowed color distribution

- **Home:** off-white canvas + five independent large scene-color cards. Each card may use 1–2 small sticker colors, but the full home should not look randomly rainbow.
- **Play:** off-white canvas, single dominant scene-colored toy cabinet; accent colors only for small header tabs, doodle marks or selected-word highlights.
- **Result:** off-white paper ticket with black outline, selectively highlighted terms. Keep black body text.
- **Settings/editor:** return to mostly off-white, with color used to indicate status/selection. Never make every form row a different saturated color.

A color should signify scene, interaction, or a specific moment — never arbitrary decoration.

## 4. Lines and shadows

- Outer cabinet: ~3–4px dark outline. Inner reel window, primary actions, cards: ~2–3px. Chips/tags: ~2px.
- Shadows are solid **offset blocks**, `box-shadow: 4px 5px 0 #171717`, with **zero blur**.
- Pressed toy buttons visually move toward their shadow: shift `translate(3px, 4px)` while reducing shadow offset to 1px. Avoid more than ~100–150ms press feedback.
- Do not use multiple glows, feathered drop shadows, glassmorphism, noise filters, skeuomorphic metal shine, chrome textures or heavy bevels.
- Border radius is moderately rounded for approachable toy plastics, not universally full-pill. Example: outer machine 20–24px, cards 16–20px, mini chips 10–14px, primary button pill if UX specifies.

## 5. Type

| Usage | Mobile starting size | Weight | Notes |
|---|---:|---:|---|
| Brand wordmark | custom SVG / ~32px | 900 | Optional doodle styling only here |
| Screen title | 18–22px | 750–850 | Never crowd top bar |
| Active reel label: 3 columns | ~20px | 700–800 | Single line, ellipsis if necessary |
| Active reel label: 4 columns | ~16px | 700–800 | Single line, ellipsis if necessary |
| Result sentence | 22–28px | 750–850 | Visual payoff; wrap freely |
| Primary CTA | ~17–20px | 750–850 | Prominent, stable |
| Standard body | 15–16px | 400–600 | Clear hierarchy |
| Metadata | 12–13px | 400–600 | Do not go below 12px |

Implementation font stack (no large web font dependency):

```css
font-family: -apple-system, BlinkMacSystemFont, 'PingFang SC',
  'Microsoft YaHei', 'Noto Sans CJK SC', sans-serif;
```

Use text outlines/white sticker backing only for decorative headings; not for long paragraphs, result details or button labels. Avoid cartoon-shadow glyph effects that reduce reading clarity.

## 6. Illustration & sticker art

- **Preferred:** handmade-style 2D SVG artwork. Use flat fills, 2.5–3px black strokes, rounded linecaps, simplified silhouettes, slightly irregular accents.
- **Do:** food bowl, heart duo, clinking cups, map pin, cocktail glass, small rays, outlined stars. Keep subject appropriate to scene.
- **Don't:** platform emoji as production illustrations; random icon packs mixed with hand-drawn art; brand logos of drinks; suggestive sexualized imagery; realistic casino hardware.
- **No detailed drawings inside reels by default**: the options are defined as short `label` strings. Icon decorations should stay outside the text reel or be deliberately implemented as non-semantic art without altering the content.
- **Budget:** use one recognisable hero doodle per home card, up to 1–2 small decorative elements around a play cabinet, and at most one small result illustration when space allows. Hide nonessential doodles first on 320px screens.
- Every decorative SVG must have `aria-hidden="true"`; meaningful icons/buttons need accessible names.

## 7. Consistency examples

- **Good:** orange cabinet / off-white reels / dark thick outlines / yellow micro-accents / white result ticket / yellow bottom button with a toy pressed state.
- **Bad:** red translucent glass cabinet / neon border glow / gradients / huge confetti while spinning / tiny low-contrast result.
- **Good:** lock state has clear `已锁` text + enclosed lock icon + saturated status background.
- **Bad:** two lock states differ by a 10% tint shift only.

## 8. References and interpretation

View `../assets/ui-references/*.png` for art direction only. They are generated illustrations and may include inconsistent label text, oversized reel icons, ornamental pull handles, impossible reel/item pairings or extra decorations. Translate **their materials and vibe** into functioning CSS/SVG components. Do not try to pixel-match illustration artifacts that hurt real touch interaction or readability.
