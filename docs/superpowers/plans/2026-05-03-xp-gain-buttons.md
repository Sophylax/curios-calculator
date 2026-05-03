# XP Gain Buttons Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add four buttons (Normal Win/Loss, Turbo Win/Loss) inside the Grind panel that increment the Current XP input by the matching XP value on left-click and decrement on right-click.

**Architecture:** Buttons live in a new `.xp-buttons` row between `#xpColumns` and `.xp-note`, in a two-column grid that mirrors the existing Normal/Turbo column split. Click and `contextmenu` listeners mutate the existing `#currentXP` input and delegate to the existing `handleXPInput()` function, which already handles XP overflow, level promotion/borrow, clamping, persistence, and display refresh.

**Tech Stack:** Vanilla HTML/CSS/JS in the single `index.html` file. No build, no test framework. Verification is manual in a browser.

**Spec:** `docs/superpowers/specs/2026-05-03-xp-gain-buttons-design.md`

---

## File Structure

Only one file is touched:

- **Modify:** `index.html`
  - Add CSS rules for `.xp-buttons` and `.xp-btn-col` in the existing `<style>` block.
  - Add a `.xp-buttons` row to the Grind panel markup.
  - Add a small JS block to populate button labels from the existing XP constants and wire `click` / `contextmenu` listeners.

There is no test file. Verification is manual — start a local HTTP server and exercise the UI in a browser.

---

## Task 1: Add HTML markup and CSS for the button row

**Files:**
- Modify: `index.html` (CSS in `<style>` block, HTML in the Grind panel)

This task adds the visible structure with no interactivity. After this commit, the buttons exist on the page and look correct, but clicking them does nothing.

- [ ] **Step 1: Add CSS for the new row**

In `index.html`, locate the `/* ── The Grind ── */` section (currently ends at the `.xp-note` rule). Add the following CSS block immediately after the `.xp-note` rule:

```css
/* Buttons row inside The Grind */
.xp-buttons {
  display: grid;
  grid-template-columns: 1fr 1fr;
  border-top: 1px solid var(--border-subtle);
}
.xp-btn-col {
  display: flex;
  gap: 6px;
  justify-content: center;
  padding: 10px 18px;
}
.xp-btn-col:first-child {
  border-right: 1px solid var(--border-subtle);
}
.xp-btn { min-width: 92px; }
```

- [ ] **Step 2: Add the responsive override at the 700px breakpoint**

In `index.html`, locate the existing `@media (max-width: 700px)` block (the one that sets `.xp-columns` to `1fr`). Inside that block, add:

```css
.xp-buttons { grid-template-columns: 1fr; }
.xp-btn-col:first-child { border-right: none; border-bottom: 1px solid var(--border-subtle); }
```

- [ ] **Step 3: Add the HTML markup**

In `index.html`, locate the Grind panel:

```html
  <!-- The Grind -->
  <div class="panel">
    <div class="panel-header">
      <h3>The Grind</h3>
      <div class="xp-summary" id="xpSummary"></div>
    </div>
    <div class="xp-columns" id="xpColumns"></div>
    <div class="xp-note">Average rows assume a 50% win rate. Hover rows for details.</div>
  </div>
```

Insert the new row between `<div class="xp-columns" id="xpColumns"></div>` and `<div class="xp-note">...`:

```html
    <div class="xp-buttons">
      <div class="xp-btn-col">
        <button class="pace-btn xp-btn" id="btnNormalWin" title="Right-click to subtract"></button>
        <button class="pace-btn xp-btn" id="btnNormalLoss" title="Right-click to subtract"></button>
      </div>
      <div class="xp-btn-col">
        <button class="pace-btn xp-btn" id="btnTurboWin" title="Right-click to subtract"></button>
        <button class="pace-btn xp-btn" id="btnTurboLoss" title="Right-click to subtract"></button>
      </div>
    </div>
```

The button labels are intentionally empty — the JS in Task 2 will populate them from the XP constants. Until Task 2 is wired up, the buttons will render as empty rectangles.

- [ ] **Step 4: Manual visual check**

Start a local server from the repo root:

```bash
python3 -m http.server 8000
```

Open `http://localhost:8000` in a browser. Expected:

- The Grind panel now has a horizontal divider below the Normal/Turbo stats.
- Below the divider, two columns each contain two empty button-shaped rectangles, separated by a vertical divider that lines up with the one above.
- The `.xp-note` text appears below the new row, unchanged.
- Resize the window narrower than 700px: the buttons stack into a single column and the vertical divider becomes a horizontal one, matching the stats grid behavior above.

Stop the server (Ctrl-C) when done.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add XP gain button row to Grind panel (markup + styling)"
```

---

## Task 2: Wire up button labels and click handlers

**Files:**
- Modify: `index.html` (JS block at the bottom)

This task makes the buttons functional: populates their labels from the XP constants, applies a delta on left-click, and subtracts on right-click.

- [ ] **Step 1: Add the helper and init block to the JS**

In `index.html`, locate the `/* ── Init ── */` section. Add the following block immediately before the existing `$('currentLevel').addEventListener('input', updateDisplay);` line (so it runs after the input values are hydrated from localStorage but before the first `updateDisplay()` call):

```js
/* ── XP Gain Buttons ── */
function applyXpDelta(delta) {
  const xpInput = $('currentXP');
  xpInput.value = (parseInt(xpInput.value) || 0) + delta;
  handleXPInput();
}

[
  ['btnNormalWin',  'Win',  XP_WIN],
  ['btnNormalLoss', 'Loss', XP_LOSS],
  ['btnTurboWin',   'Win',  XP_TURBO_WIN],
  ['btnTurboLoss',  'Loss', XP_TURBO_LOSS],
].forEach(([id, label, xp]) => {
  const btn = $(id);
  btn.textContent = `+${label} (${xp})`;
  btn.addEventListener('click', () => applyXpDelta(xp));
  btn.addEventListener('contextmenu', e => {
    e.preventDefault();
    applyXpDelta(-xp);
  });
});
```

This relies on `XP_WIN`, `XP_LOSS`, `XP_TURBO_WIN`, `XP_TURBO_LOSS`, `$`, and `handleXPInput` — all already defined earlier in the same `<script>` block.

- [ ] **Step 2: Manual verification — labels render**

Start the server again:

```bash
python3 -m http.server 8000
```

Open `http://localhost:8000`. Expected button labels:

- Normal column: `+Win (650)` and `+Loss (250)`
- Turbo column: `+Win (425)` and `+Loss (175)`

- [ ] **Step 3: Manual verification — left-click adds XP**

Set Current Level to `0`, Current XP to `0`. Click `+Win (650)`. Expected:

- Current XP becomes `650`.
- The XP ring around the level badge fills to ~65%.
- The Grind summary updates (XP needed decreases by 650).

Click `+Loss (250)`. Expected:

- Current XP becomes `900`.
- Ring fills to ~90%.

Click `+Win (650)` again. Expected:

- `handleXPInput` overflow logic fires: Current Level becomes `1`, Current XP becomes `550` (900 + 650 = 1550, carry one level).
- Level badge displays `1`.

Click `+Win (425)` (Turbo). Expected:

- Current XP becomes `975`.

- [ ] **Step 4: Manual verification — right-click subtracts XP**

Set Current Level to `1`, Current XP to `500`. Right-click `+Win (650)`. Expected:

- The browser context menu does NOT appear.
- Current Level becomes `0`, Current XP becomes `850` (handleXPInput borrow logic: 500 - 650 = -150, borrow one level → level 0, xp = 1000 - 150 = 850).

Right-click `+Loss (250)`. Expected:

- Current XP becomes `600`.

- [ ] **Step 5: Manual verification — clamps and edge cases**

Set Current Level to `0`, Current XP to `0`. Right-click `+Loss (250)`. Expected:

- Current Level stays `0`, Current XP stays `0` (clamp at zero).

Set Current Level to `300`, Current XP to `0`. Click `+Win (650)`. Expected:

- Current Level stays `300` (clamp at max), Current XP becomes `650`.

- [ ] **Step 6: Manual verification — persistence and reload**

Set values to something distinct (e.g. Level 5, XP 200). Click `+Win (650)` to make XP 850. Reload the page. Expected:

- Current Level reads `5`, Current XP reads `850` (saved to localStorage by `updateDisplay` via `handleXPInput`).

- [ ] **Step 7: Manual verification — tooltip**

Hover any of the four buttons. Expected:

- Browser tooltip shows `Right-click to subtract`.

Stop the server (Ctrl-C) when done.

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "Wire XP gain buttons to add/subtract XP via click and right-click"
```

---

## Self-Review Notes

- **Spec coverage:** Layout (Task 1, steps 1–3), labels generated from constants (Task 2, step 1), left-click adds (Task 2, step 3), right-click subtracts with `preventDefault` (Task 2, step 4), `title` tooltip (Task 1, step 3 markup; Task 2, step 7 verification), reuse `.pace-btn` (Task 1, step 3 — `class="pace-btn xp-btn"`), reuse `handleXPInput` overflow/clamp (Task 2, steps 3–5), no undo / no Pace Check coupling / no animation / no long-press (intentionally omitted).
- **Touch / mobile:** Spec says no subtract on touch. The implementation uses `contextmenu`, which on mobile typically does not fire from a tap, so touch users get add-only behavior automatically. No extra code needed.
- **Placement of init code:** Placed before the existing input listeners are wired so that ordering is consistent with how the rest of init reads — input values hydrated, then all listeners attached, then `updateDisplay()` runs.
- **No new constants or types introduced** — the button list inlines the four `[id, label, xp]` triples; trivially small, no need for a top-level constant.

---

Plan complete and saved to `docs/superpowers/plans/2026-05-03-xp-gain-buttons.md`. Two execution options:

**1. Subagent-Driven (recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration.

**2. Inline Execution** — Execute tasks in this session using executing-plans, batch execution with checkpoints.

Which approach?
