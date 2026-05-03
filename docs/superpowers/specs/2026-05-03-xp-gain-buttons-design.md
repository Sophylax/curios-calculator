# XP Gain Buttons — Design

## Goal

Let the user log a finished match by clicking a button instead of typing
into the Current XP field. Four buttons cover the four common XP outcomes:
Normal Win, Normal Loss, Turbo Win, Turbo Loss.

## Layout

The buttons live inside the existing **The Grind** panel, in a new row
between the `xp-columns` grid and the `xp-note` footer text.

The new row mirrors the two-column structure above it:

- Left half (Normal column): `+Win (650)`  `+Loss (250)`
- Right half (Turbo column): `+Win (425)`  `+Loss (175)`

The visual effect is that each game-type column gains a "log a result" row
directly beneath its stats, so the buttons read as the two outcomes of
that game type.

## Button labels

`+Win (650)` / `+Loss (250)` for Normal, `+Win (425)` / `+Loss (175)` for
Turbo. The XP value is rendered using the existing `XP_WIN`, `XP_LOSS`,
`XP_TURBO_WIN`, `XP_TURBO_LOSS` constants so the labels stay in sync if
those values are ever tuned.

## Behavior

### Add (left-click)

A click adds the button's XP value to the Current XP input, then routes
through the existing `handleXPInput()` function. That function already:

- Promotes the level when XP reaches 1000+ (carrying the remainder)
- Clamps the level at 300
- Calls `updateDisplay()` (which persists to localStorage, redraws the
  XP ring, rebuilds reward badges, and refreshes the Pace Check)

No new math is needed — the buttons just mutate the input value and
delegate.

### Subtract (right-click)

A right-click on a button subtracts the same XP value. Implementation:
attach a `contextmenu` listener that calls `preventDefault()` (so the
browser menu does not appear) and applies the negative delta. The same
`handleXPInput()` path handles the underflow case (XP < 0 borrows from
level; XP < 0 with level 0 clamps to 0).

### Discoverability

Each button gets `title="Right-click to subtract"` so the gesture is
visible on hover. No on-screen affordance beyond the tooltip.

### Touch / mobile

No subtract path on touch devices. Right-click does not exist on touch,
and adding a long-press handler costs nontrivial JS (touch events,
timing, scroll suppression, context-menu suppression) for an edge case.
Mobile users who misclick edit the Current XP / Current Level fields
manually — those fields are unchanged.

## Styling

Reuse the existing `.pace-btn` class (already used by the Mark Progress
and Clear buttons in the Pace Check panel). The base styling — pink
border, dark background, hover state — is already consistent with the
rest of the app. Use the class as-is for the first pass; tune padding
or font size in a follow-up if the buttons feel too small.

The two rows of buttons share the same column dividers as the stats grid
above them so the Normal/Turbo split lines up vertically.

## Edge cases

- **XP overflow past target level:** No special handling. The user can
  exceed the target level; the existing display already supports
  `cur > tgt` (xpNeeded clamps at 0, "Levels to go" shows 0).
- **Level 300 clamp:** Already handled by `handleXPInput`.
- **XP underflow at level 0:** Already handled by `handleXPInput`
  (clamps XP to 0).

## Out of scope

- No undo button.
- No automatic Pace Check snapshot. The Mark Progress button still
  requires an explicit click.
- No new animation beyond what `updateDisplay()` already triggers
  (XP ring transition, reward badge pop/shrink on state change).
- No long-press subtract on touch devices.
- No keyboard shortcut bindings.

## Files touched

Only `index.html`. The change set is:

- Add a `.xp-buttons` row inside `.panel` for The Grind, between
  `#xpColumns` and `.xp-note`.
- Add CSS for the new row (grid columns matching the stats grid; reuse
  `.pace-btn` for buttons).
- Add a small JS handler that, given a delta XP value, mutates the
  Current XP input and calls `handleXPInput()`. Wire it to the four
  buttons via `click` and `contextmenu` listeners.
