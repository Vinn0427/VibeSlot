# Install, trigger, and maintain the Skill

## 1. Recommended location

Place the entire `vibeslot-ui/` folder in the **VibeSlot repository**:

```text
<repo-root>/
├── AGENTS.md                         # project agent instructions (present)
├── design/                           # product & design documents
│   ├── PRODUCT_VISION.md
│   ├── PRD.md
│   ├── CONTENT_DESIGN.md
│   └── UX_DESIGN.md
└── .agents/
    └── skills/
        └── vibeslot-ui/
            ├── SKILL.md
            ├── agents/openai.yaml
            ├── references/*.md
            └── assets/ui-references/*.png
```

Codex commonly discovers project skills in `.agents/skills/`. If your specific Codex setup uses another skills directory, follow that installation convention. Repository version-control is recommended so UI rules travel with the code.

## 2. Repo-level `AGENTS.md` snippet

Add this snippet to the repository's existing AGENTS.md (preserve existing constraints):

```markdown
## VibeSlot frontend visual rules

When implementing or reviewing any React/TypeScript/CSS page, UI component,
responsive layout, reel animation, or interaction styling, load and follow
`.agents/skills/vibeslot-ui/SKILL.md` before editing.

Visual target: Neo-Brutalism × hand-drawn stickers × tangible toy-machine feel.
Mobile Web is canonical (375px; support 320px); desktop shows a centered
max-480px app canvas. Reuse shared tokens and components; do not copy reference
mockups as images or invent option text. Product behavior is governed by
design/PRODUCT_VISION.md, design/PRD.md, design/CONTENT_DESIGN.md and
design/UX_DESIGN.md.

Before completion, apply the Skill's UI review checklist and report what
was actually tested. For backend-only work, this UI Skill is not required.
```

The snippet is useful because a Skill's discovery description can still be bypassed in some agent sessions; a local project agent instruction makes the expected workflow explicit.

## 3. Example Codex invocation

Use direct invocation if needed:

```text
Use $vibeslot-ui to implement the mobile home page according to
design/PRD.md and design/UX_DESIGN.md §4, using the included `01-home.png`
as the approved target look (structure and copy still follow the docs).
Maintain real semantic DOM controls, focus states, and no casino styling.
Check 320px, 375px and desktop 480px canvas before reporting completion.
```

For another task:

```text
Use $vibeslot-ui to implement the `/play/cook` toy cabinet and result ticket.
Keep the documented random/lock/reel state machine intact. Read
references/motion-responsive.md, and do not treat 02-cook.png as an exact spec.
```

## 4. Modification workflow

- When an aesthetic decision is accepted, first update **this Skill's tokens/components**, then implement the component(s).
- When a new product interaction is accepted, update the appropriate PRD/UX docs **before** changing the Skill; don't overwrite business decisions with visual preferences.
- Keep reference images as moodboards; replace them only with accepted screenshots from actual implementation after a proper visual review.
- Avoid installing several overlapping global design Skills that contradict these scene-specific constraints. If a general frontend-design Skill is also used, **the VibeSlot UI Skill wins for VibeSlot visual decisions**.

## 5. Maintainer decisions worth validating with real users

These are proposals; adjust after reviewing actual mobile pages:
- Exact 6-color palette and shadow offsets.
- How much sticker decoration a 320px screen can support.
- How much result text can be highlighted before it affects legibility.
- How the first visual example translates to the shorter 375×667 viewport.

Already decided in `design/UX_DESIGN.md` (no longer open):
- Decorative cabinet handles/levers are **not implemented** (UX §5.1).
- `01-home.png` is the approved homepage target (UX §4).
- Locked reel buttons use an ink-black fill with off-white text (UX §5.5).
- Desktop keyboard shortcuts are out of scope (UX §13).
