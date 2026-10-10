# Pages and component contracts

This document covers **visual structures**. Exact function, button semantics, states, content and wording live in PRD/UX/CONTENT.

## 1. Information hierarchy by page

| Page | Visual lead | Secondary | Should fade back |
|---|---|---|---|
| Home | brand + scene card wall | my machines & create | feedback/footer |
| Play idle | unified cabinet + giant "转一下" | scene name and optional controls | settings, helper text |
| Play spinning | moving reel strips, sequential stopping | disabled stable action | everything else |
| Play result | **result ticket sentence** + cabinet | accept/complete and respin | metadata, decoration |
| Play accepted | stamped result & share action | restart/next round | prior helper tips |
| Editor | editable columns and short labels | sentence preview | decorative artwork |
| Import preview | template summary + "直接转一下" | save and reveal options | illustrations |
| Bottom sheet | clear title and rows/toggles | close/done action | background machine |

## 2. Homepage

- The composition in the baseline UX is **one large `cook` card followed by a 2×2 grid** (`date`, `party`, `mix`, `couple`), then "我的老虎机", then create/feedback.
- Cards: saturated flat fill, strong black outline, solid offset shadow, playful but consistent shape and icon placement.
- Pressed state: card translates ~2–3px down/right and shadow becomes shorter. Motion must not shift neighboring cards.
- A card itself is one large focusable/tappable link. Do not place inert arrow-buttons inside a clickable card as separate focus targets.
- Small card copy may be truncated gracefully; titles must remain fully readable if possible. Show all five presets and creation entry on a scrollable phone home page (no need to force into one viewport).
- "我的老虎机" list items use **one generic icon** (e.g. a small toy slot machine) for every custom template. Do not infer illustrations from user-written template content (the toast/drink art in `01-home.png` is only an example).
- On 320px, hide the brand-area mini machine illustration and sticker first.
- No login, navigation drawer, leaderboard, or complex carousel.
- Detailed homepage spec: `design/UX_DESIGN.md` §4.

## 3. SlotCabinet: the physical toy centerpiece

Recommended component roles (use existing architecture if available):

```
PlayScreen
├── PlayHeader
├── SceneControls?          // intensity, optional reel, content types
├── SessionStatus?          // remaining spins or round number
├── SlotCabinet
│   ├── ReelColumn × 1–4
│   │   ├── ReelLabel          // e.g. 主料 / 配菜 / 风味
│   │   ├── ReelWindow         // 3 rows, central result marker
│   │   └── LockButton        // disabled / locked / unlocked
│   └── Decoration?           // small, non-interactive sticker
├── ResultTicket?            // result and accepted states only
└── BottomPrimaryAction      // sticky in thumb area
```

- The reels share **one housing** with a scene-color background, one continuous outer border, and one visual baseline. This is the "physical toy" cue.
- Each reel window has three visual rows; top and bottom appear faded and less important, the middle is selected and visually "snaps" into the selection line.
- Stick to text-only option content per CONTENT. Never invent icon metadata from labels or use illustrations in a way that changes how options are read.
- Width equals the available cabinet width distributed among N reels with fixed narrow gaps. 1–2 reels stay centered, not stretched to comical width.
- Each lock has an independent ≥44px-tall hit area, but the caption can be shortened for a 4-reel case while preserving an accessible full label.
- Lock can only become interactive after a valid result. Locked reel does not move, shake, or pulse during subsequent spins.
- **No decorative handle/lever** (decided in UX §5.1): it causes horizontal overflow, tap confusion and gambling resemblance. The real action is always the primary button.
- Don't use a huge fixed-height illustration: the cabinet must fit on a 375×667 viewport alongside a readable result card and persistent CTA with vertical scroll if necessary.

### Reel structure and text

- Label `label` is a plain, short text string: default target ≤6 Chinese characters; actual maximum in content/PRD is 8.
- Reel center has visual emphasis (background, arrows/markers and bold text). Neighbors use opacity and a vertical fade.
- Avoid the browser's native 3D wheel picker appearance. Build a two-dimensional reel with the impression of a physical drum.
- Results are chosen **before** animation per PRD. The reel animation reveals the precomputed result, never decides the value.
- If truncated inside reel (`text-overflow: ellipsis`), show full option in result ticket and option list sheet.

## 4. ResultTicket: paper decision slip

- Appears only after the final participating reel lands; animate in lightly.
- Information order: small result labels (joined with ×), prominent template `sentence`, optional `note`/scene hint, then context actions. For custom template without sentence, show joined labels once at large size.
- Use thick dark outline and off-white paper fill. A small tilt/stamp accent may appear in the `accepted` state, without deforming the readable body.
- The **sentence** is the largest usable text after the spin. Scene-selected words may have colorful marker-like underline/highlight but stay readable.
- Do not inject generated explanatory text, recipes, derived images, or content details that PRD did not authorize.
- Party/couple results are used by a small group watching one phone: target >=24px result sentence when width allows and ensure high contrast.
- Add a share affordance only in states allowed by PRD; `accepted` and `result` differ by mode.

## 5. PrimaryAction: the thumb-zone toy button

- Persistent bottom location: width nearly full (page padding), ~56px height, bold typography, dark outline, hard solid lower shadow.
- Press: 2–3px translate + shadow contraction; stable layout.
- Text and enabled state are driven by the PRD state machine (e.g. `转一下` / `再转一次` / `换一个` / `下一轮` / `重新开始` / `转动中…` / `命运已定`).
- When all columns are locked or spin limit exhausted, disabled state must explain why without faking an animation.
- During `spinning`, stable dimensions; suppress double-tap and make disabled state visually clear.
- Use bottom safe-area padding and avoid blocking the result ticket without accessible scroll.

## 6. Top controls, editor and sheets

- Intensity selector only for `couple` (including applicable copied templates): `温和 / 心动 / 升温`. Its selected state is clear by fill, border and text; do not make three large competing primary buttons.
- Optional reel toggles (`mix`, `couple`) use compact chips; turned off means no visible optional reel. The editor cannot invent capabilities outside PRD.
- Party `内容 3/4 类` links to section in settings sheet; content group toggles reflect real exclusion behavior. Alcohol group defaults excluded by CONTENT/PRD.
- Settings and option list use a bottom sheet with rounded top corners, strong outline, scrollable rows, 44px controls, darkened/backdrop layer (not frosted glass). Max height follows UX (~85vh) and accommodates keyboard/safe areas.
- Editor is **form-first**: calm off-white cards and inputs, restrained stickers. Keyboard/paste workflow has priority over illustration. Actual form rules from PRD F12.
- Import screen prioritizes "直接转一下" and summary; do not leak option content when UX says default collapsed.

## 7. Content/image reference mismatch warnings

These examples are NOT authority:
- Generated couple mocks may mention items such as "卧室里" that are not in the official preset. Use the official list.
- Generated illustrations may attach food or gesture icons directly to every reel item. Official candidates only supply `label`; treat iconography as decoration, not promised content.
- Some generated examples use confusing `锁定/解锁/已锁` wording. Follow the UX/PRD documented button semantics.
- `01-home.png` shows the wordmark "VibeSlot"; the product name is **EasySlot** — keep the sticker wordmark style, change the text.
- Some mockups show a lever. Don't implement it (UX §5.1).
- `02-cook.png` shows the locked button in grey; follow UX §5.5 (ink-black fill) instead.
- Mocked remaining spins/round index are just example states, not visible on every template at all times.
