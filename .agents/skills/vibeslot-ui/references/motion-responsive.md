# Motion, responsive layouts, performance & accessibility

## 1. Motion philosophy: a toy with weight

Movement is reserved for user-driven mechanical feedback: pressing a button, spinning reels, individual reels landing, the result ticket arriving, and confirming a decision. More subtle controls remain quiet.

The user should feel a sequence: **press → acceleration → fast roll → staggered braking → tiny overshoot → snap → result ticket**.

Do not fake randomness with changing results during/after the animation. Per PRD the result is fixed when the spin begins.

## 2. Reel timing baseline (UX proposal)

| Moment | Preferred timing | Implementation hint |
|---|---:|---|
| Press feedback | ~80–120ms | translate and hard-shadow reduction |
| Acceleration | first ~150ms | `ease-in` |
| Fast rolling | ~20–25 visual items/sec (perception) | pre-rendered sequence / transform strip |
| Participating reel i stop | `T_i = 1100 + 300 × i` ms | i = 0-based among *unlocked* reels |
| Final braking window | ~600ms before each reel stops | cubic-bezier-like ease-out |
| Overshoot | ~6–8px | only active reel, avoid layout work |
| Snap back | ~120ms | transform only |
| Result ticket appears | ~100ms after last stop | fade + translateY(16px→0) for ~220ms |
| Total full spin | **≤2500ms** | preserve with 1–4 reels |

Tune after device testing. Implement sequence/callbacks using existing stack; an animation library is optional, not required. Prefer CSS transforms + requestAnimationFrame orchestration for virtual strips; clean up RAFs and callbacks on unmount/navigation.

### Locked and hidden reels

- Only participating (unlocked, enabled) reels animate. Locked reel is pixel-stable.
- Stagger participating reels left to right, independently of locked columns' absolute indices.
- Optional reel disabled: no animation, no column, no result text. Enabled mid-result: placeholder until next roll per PRD.
- Disable interactions during spin according to PRD. P1 emergency stop (if implemented) must use the **same precomputed result**.
- Screen hidden/backgrounded: on return settle immediately on precomputed result, don't replay 2s offscreen animation.

## 3. Reduced motion

Respect `prefers-reduced-motion: reduce`:
- Do not simulate scrolling reels, overshoot, bounce or confetti.
- Switch reel text with a brief staggered fade; all outcomes visible within **≤0.8s**.
- Result announcement and semantics unchanged.
- Never make accessibility mode change random selection behavior, content or lock semantics.

```css
@media (prefers-reduced-motion: reduce) {
  .vs-confetti,
  .vs-decorative-motion { display: none !important; }
  .vs-button,
  .vs-result-ticket,
  .vs-reel-strip { animation: none; }
}
```

The snippet is illustrative; ensure the actual reduced-motion implementation still presents new results clearly.

## 4. Mobile-first viewport contract

Use 375px as design baseline, support **320px through large phones** and a short 375×667 viewport.

| Width | Layout | Priority |
|---|---|---|
| 320–374px | same layout, tighter padding/gaps, simplified stickers, no decorative lever | all interactions fit; reel text may truncate |
| 375–430px | canonical full-width mobile experience | hero cabinet + readable result + thumb button |
| 431–767px | centered content, max about 480px | modest surrounding space |
| ≥768px | centered 480px app canvas, decorative sides if desired | never turn it into multi-column dashboard |

**No horizontal body scroll** at any breakpoint. Support one through four reels; ensure the fourth column does not make a 320px phone unusable.

Guidelines:
- Page max width: `min(100%, 480px)` excluding purely decorative desktop background.
- Horizontal page padding ~16px; at 320px a smaller gap may be warranted if all buttons remain tappable.
- Reel available width: `(cabinetInnerWidth - gaps) / reelCount`. Labels ellipsize; every option fully readable in result ticket or expansion sheet.
- Main bottom button stays ~56px tall, safe-area-aware. Avoid `100vh` traps in mobile browser chrome; use viewport units with fallbacks (`100dvh` where supported) and allow natural document scroll.
- Sheets scroll internally and never hide their close/done action under iOS home indicator or virtual keyboard.
- When viewport height is tight, prioritize **cabinet, result and CTA**; remove or shrink ornamental doodles, oversized titles, excessive margins and extra hero icons before shrinking user text.

### Example shell CSS (adapt to existing code)

```css
.vs-app-shell {
  width: min(100%, var(--vs-content-max, 480px));
  min-height: 100dvh;
  margin-inline: auto;
  padding-inline: 16px;
  padding-top: env(safe-area-inset-top, 0px);
  background: var(--vs-bg);
}
.vs-bottom-action {
  position: sticky;
  bottom: 0;
  padding-block: 8px calc(12px + env(safe-area-inset-bottom, 0px));
  background: var(--vs-bg);
}
.vs-reel-row { display: flex; gap: 8px; min-width: 0; }
.vs-reel-column { min-width: 0; flex: 1 1 0; }
.vs-reel-option {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

If `position: sticky` cannot satisfy the app shell and scroll behavior, use a carefully measured fixed action area and add matching scroll padding. Do not assume one implementation will fit every browser.

## 5. Frame budget / bundle discipline

- Target 60fps on a representative mid-range mobile device; do not animate layout properties (`top`, `height`, `margin`, etc.) on many nodes every frame.
- Animate strips with `transform: translateY`, buttons with `transform`, entrance with `opacity/transform`. Avoid continuous `filter: blur`, large box-shadow blur and SVG filter effects.
- Prefer small SVG or optimized illustrations; preload only truly critical assets. Lazy load offscreen scene art if needed.
- Avoid huge runtime dependencies for confetti. CSS/light DOM limited confetti is fine at acceptance; reduced-motion disables it.
- Comply with PRD network budget: 4G first interactive <2s and first-screen gzip JS <200KB as an **aspiration to measure** (do not assert without a build measurement).
- Sound and vibration are **not V0.1**, though motion events may expose hooks for future audio/haptics.
- Ensure slow network or absent decorative assets never block spin functionality.

## 6. Interaction accessibility

- All clickable controls have minimum effective hitbox 44×44px; if visual controls are narrower, use an expanded invisible hit area without overlap.
- Labels are actual text (not text baked into image). Native button/link/input semantics, logical focus order, visible `:focus-visible`.
- `aria-live="polite"` for final result announcement; don't announce rapid changing reel filler texts.
- Lock actions have unambiguous aria labels (`锁定主料`, `已锁定主料`) and `aria-pressed` where appropriate.
- Disabled states must be programmatically disabled and visually distinct. Prevent accidental second spin.
- Contrast: WCAG AA for body/result; verify scene colored text/foreground combos. Avoid high saturation behind dense copy.
- Decorative SVG and doodles use `aria-hidden="true"`; user state is never communicated only through color.
- Preserve readability when OS text is enlarged. At 200% zoom, allow scroll and wrapping, never horizontal clipping of critical actions.

## 7. Visual testing cases to capture

Capture or inspect, whenever tooling exists:
1. home empty `我的老虎机` on 320 and 375 width;
2. home with a saved custom template;
3. `cook` idle and result (one reel locked) at 375×667;
4. `couple` default **温和** and switched 心动 with fourth optional reel enabled;
5. `party` result with settings sheet open and default alcohol excluded;
6. custom four-reel longest valid labels at 320px;
7. `accepted` stamp and correct share/next action;
8. reduced-motion, keyboard focus, and long bottom sheet with safe area.

When tests cannot be run, state the gap and what device/browser remains unverified.
