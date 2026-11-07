# T-Ball Roster Generator Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** A single-file `index.html` that generates a 4-inning T-ball lineup (batting order plus fielding positions) with per-step Regenerate/Accept, remembered across refresh.

**Architecture:** One HTML file, vanilla JS, no dependencies. A pure logic layer (`genOrder`, `genInning`) is separate from a state/render layer. State is one object saved to `localStorage`. Opening `index.html#test` runs the logic self-check instead of the app.

**Tech Stack:** HTML, CSS, vanilla JS (ES2020). Google Fonts are not used (file must work offline); use system font stacks.

**Spec:** `docs/superpowers/specs/2026-11-07-tball-roster-design.md`

## Global Constraints

- Single file `index.html`; no build step; no external scripts, fonts or images (inline SVG only). Must work offline.
- Roster: `Alex, Leo, Alistair, Gus, Harrison, Hamish, Luca, Liam, Caspar, Prim, Jian` in a `PLAYERS` array at the top of the script.
- 4 innings. Positions (9): `Pitcher, Catcher, 1st Base, 2nd Base, 3rd Base, Shortstop, Left Field, Centrefield, Right Field`, plus `Bench`.
- No player benched twice; no player plays the same position twice in a game.
- Fewer than 9 present: drop random outfield positions (LF/CF/RF), fresh each inning. Minimum 6 present.
- Table columns: Batting Order, 1st, 2nd, 3rd, 4th. Rows in batting order.
- State saved to one `localStorage` key `tball-roster-v1`, all access in try/catch.
- Team: Thornleigh Thunderbolts. Palette: royal blue, lightning gold, navy, white. Mobile-first, light and dark theme, large tap targets.
- Frontend-design skill is used for Task 3.

## Review Focus

- All 11 absent, or fewer than 6 present: show a message, never crash or generate.
- Exactly 9 present: nobody benched, all 9 positions filled each inning.
- `genInning` returns `null` (dead end): UI shows message with Undo, no blank or broken table.
- Corrupt or old `localStorage` JSON: app starts fresh instead of throwing.
- Rapid double-tap on Accept or Regenerate must not skip or double-accept an inning.

---

### Task 1: Logic layer and self-check

**Files:**
- Create: `index.html` (script block with logic and `#test` runner only; minimal HTML shell)

**Interfaces:**
- Produces:
  - `const PLAYERS: string[]`, `const POSITIONS: string[]` (9, order as in Global Constraints), `const OUTFIELD = ['Left Field','Centrefield','Right Field']`
  - `shuffle<T>(arr: T[]): T[]` returns a new shuffled array
  - `genOrder(present: string[]): string[]`
  - `genInning(present: string[], history: Array<Record<string,string>>): Record<string,string> | null` maps each present player to a position or `'Bench'`; `history` is the accepted innings so far
  - `runSelfTest(): {ok: boolean, failures: string[]}`

- [ ] **Step 1: Write the failing self-test** in `runSelfTest`: for each present count `n` in 6..11 (first `n` of `PLAYERS`), play 1000 games of 4 innings via `genInning`, asserting: result never `null`; every present player has a value; fielders occupy distinct positions; `n>=9` gives `n-9` benched and all 9 positions used; `n<9` gives exactly `9-n` dropped, all dropped being outfield; no player has `'Bench'` twice; no player has the same non-bench position twice. Wire `location.hash === '#test'` to run it and print failures (or `PASS`) into the page and console.
- [ ] **Step 2: Run it** by opening `index.html#test`. Expected: fails because `genInning` is not defined.
- [ ] **Step 3: Implement `shuffle`, `genOrder`, `genInning`.** `genInning`: compute slots (9 positions, or minus randomly dropped outfield when `n<9`); choose bench from never-benched players when `n>9`; assign fielders to slots by shuffled backtracking, skipping any position a player already has in `history`; return `null` if none works.
- [ ] **Step 4: Run `#test` again.** Expected: `PASS`.
- [ ] **Step 5: Commit** (`git init` first if no repo): `git add index.html && git commit -m "feat: lineup generation logic and self-test"`

### Task 2: State, flow and persistence

**Files:**
- Modify: `index.html` (add state layer and plain unstyled render)

**Interfaces:**
- Consumes: `genOrder`, `genInning`, `PLAYERS`.
- Produces:
  - `state: {absent: string[], order: string[]|null, innings: Array<Record<string,string>>, pending: object|null, error: string|null}` (step is derived: no order means step 0 or 1; `innings.length` gives the inning step)
  - `present(): string[]`, `load(): void`, `save(): void`, `render(): void`
  - actions: `toggleAbsent(name)`, `startGame()`, `regenerate()`, `accept()`, `undo()`, `reset()`

- [ ] **Step 1: Add self-test cases** in `runSelfTest` for state actions on a fresh state: `startGame` with <6 present sets `error` and leaves `order` null; `accept` called twice quickly adds one inning only; `undo` after inning 1 re-opens it as `pending`; `reset` returns the initial state; `load` with garbage JSON in storage yields initial state.
- [ ] **Step 2: Run `#test`.** Expected: new cases fail.
- [ ] **Step 3: Implement the state layer.** `startGame` generates `pending` order; `accept` moves `pending` into `order` or `innings` and immediately generates the next pending (none after inning 4); `regenerate` replaces `pending`; `genInning` returning `null` sets `error` and leaves `pending` null. Every action calls `save()` then `render()`. `reset` confirms via `confirm()` in the UI wrapper, not in the logic.
- [ ] **Step 4: Run `#test`.** Expected: `PASS`. Manually open `index.html` and refresh mid-game; the state survives.
- [ ] **Step 5: Commit:** `git commit -am "feat: state, flow, persistence"`

### Task 3: UI and branding

**Files:**
- Modify: `index.html` (CSS, markup, inline SVG bolt, final `render()`)

**Interfaces:**
- Consumes: all Task 2 actions and `state`.
- Produces: finished UI per spec's UI and Branding sections.

- [ ] **Step 1: Invoke the frontend-design skill** and apply its guidance to this task.
- [ ] **Step 2: Build the UI:** header with "Thornleigh Thunderbolts" wordmark and inline SVG bolt; step 0 shows player chips (tap toggles absent) and a Continue button, disabled with a message when fewer than 6 present; later steps show the lineup table (accepted innings solid, pending highlighted, future innings empty placeholders), absent chips above the table, Regenerate and Accept buttons for the pending step, Undo, and Reset behind a `confirm()`. Position chips color-coded, Bench muted red. Ignore Accept and Regenerate clicks when `pending` is null (covers double-tap).
- [ ] **Step 2b: Show `state.error`** in a banner with an Undo button.
- [ ] **Step 3: Verify in the browser** at phone width (375px) and desktop, in light and dark: play a full game with 10 present, then one with 11, then one with 7 present. Expected: complete 4-inning table each time, no repeated bench or position, refresh keeps the game, Reset clears it.
- [ ] **Step 4: Run `#test`.** Expected: `PASS`.
- [ ] **Step 5: Commit:** `git commit -am "feat: Thunderbolts UI"`
