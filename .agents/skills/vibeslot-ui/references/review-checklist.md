# UI acceptance checklist for coding agents

Use this at the end of each frontend change. Do not mark checks as passed if not observed or tested.

## A. Product correctness (block release on failure)

- [ ] Page and copy respect the actual project Vision/PRD/CONTENT/UX, not generated reference art.
- [ ] All 5 preset scenes and custom templates use the correct scene structure; no invented options, metadata or unauthorized features.
- [ ] Correct `idle` → `spinning` → `result` → `accepted` logic by `decision`/`game` mode.
- [ ] Precomputed random result; only unlocked/enabled reels move; left-to-right stop sequence.
- [ ] Lock can be selected only in allowed states; all locked disables the main action.
- [ ] Optional reel starts hidden where prescribed; hidden reel is removed from visual/result.
- [ ] Party content type toggles and couple intensity filters map to existing business rules.
- [ ] Result ticket uses actual sentence, exact labels and optional note/notice; no invented descriptions.
- [ ] Buttons show exactly state-appropriate Chinese labels (not whatever appears in a drawing).
- [ ] No casino/monetization/excess alcohol/sexualized illustrations or copy.

## B. Art direction consistency

- [ ] Background off-white, dark near-black outlines, strong flat colors, **hard zero-blur shadows**.
- [ ] Scene color is dominant in the cabinet; color roles remain consistent across screens.
- [ ] Homepage has one leading cook card and four smaller scene cards unless UX changed.
- [ ] Reels sit in ONE visually unified toy cabinet, not three disconnected cards.
- [ ] Result ticket looks like a tangible paper prize/decision slip, with large readable sentence.
- [ ] Decorational SVGs share line weight, proportions and fill style; no platform emoji.
- [ ] Doodles never cover Chinese labels, hit targets, or important state indicators.
- [ ] Press/down, lock/unlock, active segment, disabled/spinning state are identifiable without relying solely on color.
- [ ] No generic SaaS glass, grays-on-white card sea, gradient backgrounds, large purple glow or realistic gambling cabinet.

## C. Responsive and UX

- [ ] Tested/inspected 320, 375, 430px widths and a 375×667 short viewport.
- [ ] Tested/inspected ≥768px: content is still one centered 480px canvas.
- [ ] Body never has horizontal scrolling.
- [ ] 1–4 reel cases survive; 4-reel locks remain tappable; valid long Chinese labels truncate inside reels and read in full in the result card.
- [ ] Primary action reachable in lower thumb zone; safe-area and keyboard handling are correct.
- [ ] Bottom sheet has accessible close action, scrollable body and proper background blocking.
- [ ] Confetti, stickers, floating adornments don't hurt mobile readability or result visibility.

## D. Performance, motion and accessibility

- [ ] ≤2.5s full spin with sequential landings; ≤0.8s reduced-motion result change.
- [ ] Uses transform/opacity for high-frequency animation; no full-app rerender every frame.
- [ ] No unsupported audio/haptics in V0.1.
- [ ] On background/foreground, result settles consistently with source state.
- [ ] All interactive hitboxes at least 44×44px; keyboard focus clear; `aria-live` result semantics implemented.
- [ ] Readable contrast, system Chinese font fallback; no massive webfont download.
- [ ] No significant performance regression, layout shift or new heavy dependencies without reason.

## E. Change discipline and deliverable

- [ ] Existing components/tokens reused or updated intentionally; no duplicate styles for same role.
- [ ] Existing user modifications preserved; no deletion/refactor of unrelated code.
- [ ] Build/typecheck/tests performed if available; failures described accurately.
- [ ] New page screenshot or manual visual QA documented if browser tools available; otherwise explicitly state unverified.
- [ ] Describe what changed and which reference/mockup direction it follows; list conscious differences from concept art.

Suggested short report format:

```text
页面/组件：
视觉约束：通过 / 有偏差（说明）
交互状态：覆盖了哪些
响应式：实测尺寸/未测尺寸
无障碍与性能：实测/未测
文档冲突或待确认：
```
