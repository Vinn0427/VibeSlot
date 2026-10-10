# VibeSlot project instructions

User-facing product name for V0.1 is **EasySlot** (wordmark, page titles,
share text). "VibeSlot" remains only as the repository/codename, and in
technical identifiers such as the `vibeslot-ui` skill and `--vs-` CSS tokens.

## VibeSlot frontend visual rules

When implementing or reviewing any frontend page, UI component, responsive
layout, reel animation, or interaction styling, load and follow
`.agents/skills/vibeslot-ui/SKILL.md` before editing.

Visual target: Neo-Brutalism × hand-drawn stickers × tangible toy-machine feel.
Mobile Web is canonical (375px; support 320px); desktop shows a centered
max-480px app canvas. Reuse shared tokens and components; do not copy reference
mockups as images or invent option text.

Product behavior and content are governed by the documents under `design/`:
`PRODUCT_VISION.md`, `PRD.md`, `CONTENT_DESIGN.md`, and `UX_DESIGN.md`.
Follow the hierarchy stated in the skill; CONTENT owns preset labels and copy.

Before completion, apply the skill's UI review checklist and report what was
actually tested. For backend-only work, this UI skill is not required.

## Implementation workflow

- Implementation approach (stack, data model, storage, engine, animation,
  backend, deployment, coding conventions) is defined in
  `design/TECH_DESIGN.md`; follow its §17 conventions for every task.
- Work is split into tasks T01–T13 in `design/IMPLEMENTATION_PLAN.md`.
  Implement one task at a time, only within that task's scope, and report
  against its acceptance criteria using the template in §7.1.
- Do not add dependencies outside TECH_DESIGN §2, and never implement items
  listed under PRD §12 (明确不做).
- If docs conflict or a rule is missing, stop and list the issue instead of
  deciding product behavior yourself.
