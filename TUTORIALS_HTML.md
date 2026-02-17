# Tutorials + Highlights + Settings Gate

## Goal
Provide step-by-step, contextual tutorials for the main screen **and** for key modals. Tutorials should:
- Highlight UI targets with a **distinct orange glow**.
- Be skippable.
- Auto-run **only the first time** a user enters that area, unless manually re-opened.
- Be resettable via a hidden settings panel accessed with a passcode.

This app already implements the pattern; use it as the reference.

## UX Overview
- A floating lightbulb button opens the **main tutorial** anytime.
- Each major modal has its own lightbulb button to open its tutorial (Case Setup, Issue, Add Issues, Notes).
- Tutorials appear as an overlay card with `Back`, `Next`, and `Skip` actions.
- The highlighted target element glows orange to focus attention.
- Tutorials auto-run the first time a user opens a given area (tracked in `localStorage`).

## UI Pieces (index.html)
- Main tutorial button: `#tutorialBtn` (lightbulb).
- Modal tutorial buttons: `#caseSetupTutorialBtn`, `#issueTutorialBtn`, `#addIssuesTutorialBtn`, `#premedNotesTutorialBtn`.
- Tutorial overlay + card:
  - `#tutorialOverlay`, `#tutorialCard`, `#tutorialTitle`, `#tutorialBody`, `#tutorialBackBtn`, `#tutorialSkipBtn`, `#tutorialNextBtn`.
- Settings access: app icon button `#settingsBtn` triggers passcode prompt and opens `#settingsModal`.
- Settings modal includes “Reset Tutorials” (`#resetTutorialsBtn`).

## Tutorial Data Model (main.js)
Tutorials are defined as arrays of steps with title/body/target selector:
- `tutorialStepsMain`
- `tutorialStepsCaseSetup`
- `tutorialStepsIssue`
- `tutorialStepsAddIssues`
- `tutorialStepsNotes`

Each step:
```js
{ title: 'Case Setup', body: 'Start here...', target: '#caseSetupBtn' }
```

## Behavior and Flow (main.js)
### Start/End
- `startTutorial(steps, returnFocusEl, key, force)`
  - If `force` is false and `tutorialSeen:${key}` is already in `localStorage`, it **returns early**.
  - Sets current steps, renders first step, shows overlay.
- `endTutorial()`
  - Marks the tutorial as seen (per key) and hides overlay.
  - Clears any highlight and restores focus.

### Step Rendering
- `showTutorialStep(index)`:
  - Adds `.tutorial-highlight` to the target selector.
  - Scrolls target into view.
  - Positions the tutorial card near the target via `positionTutorialCard()`.
  - Updates title/body text and button states.

### First-Time Auto-Run
- Each tutorial is invoked at the natural entry point of its UI:
  - Main (after privacy modal closes; and manual lightbulb click).
  - Case Setup modal open.
  - Add Issues modal open.
  - Issue modal open (or when a user first tries to save with no action content).
  - Notes modal open.
- A `localStorage` key controls “seen” state: `tutorialSeen:<key>`.
  - `isTutorialSeen(key)` checks if it’s already completed.
  - `markTutorialSeen(key)` writes to `localStorage`.

### Skip / Back / Next
- `Skip` calls `endTutorial()` immediately.
- `Back`/`Next` walk the step list and update button state.
- Final `Next` button changes to `Done` (calls `endTutorial()`).

## Orange Glow Highlight (styles.css)
Targets get `.tutorial-highlight` applied, which creates the glow:
- Dual orange box shadows
- Orange outline

This is the glow the user likes:
```css
.tutorial-highlight {
  box-shadow: 0 0 0 3px rgba(242, 140, 56, 0.45),
              0 0 0 12px rgba(242, 140, 56, 0.22) !important;
  outline: 3px solid rgba(242, 140, 56, 0.75) !important;
  outline-offset: 3px;
}
```

## Settings Passcode + Reset Tutorials
- Clicking the app icon (`#settingsBtn`) prompts for passcode.
- Passcode is hard-coded to `3384`.
- On success, settings modal opens and exposes “Reset Tutorials”.
- Reset clears all `tutorialSeen:*` keys so tutorials show again.

Code references:
- `main.js`: `ui.settingsBtn` click handler, `resetAllTutorials()`.

## Manual Re-Run (Always Available)
Each tutorial button calls `startTutorial(..., force = true)` so the user can re-run tutorials on demand even if already “seen”.

## Implementation Notes for Another App
To recreate:
1. Add a tutorial overlay + card with back/next/skip controls.
2. Define step arrays `{ title, body, target }` per screen/modal.
3. Highlight target elements with `.tutorial-highlight` and orange glow styling.
4. Track completion per tutorial in `localStorage` and skip auto-run if seen.
5. Provide a hidden settings panel accessible via passcode for resetting tutorials.
6. Use `force = true` for manual “Show tutorial” buttons.

## Files to Copy or Reference
- `index.html` (tutorial buttons, overlay, settings modal)
- `main.js` (step definitions, start/end/skip logic, localStorage)
- `styles.css` (orange glow highlight styling)
