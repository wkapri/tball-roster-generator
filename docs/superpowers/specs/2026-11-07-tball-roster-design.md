# T-Ball Roster Generator: Design

## Purpose
A single self-contained `index.html` (no build step, no dependencies, hostable on GitHub Pages) that a coach uses at the field to generate a game-day lineup for a team of 11 six-year-olds. It replaces a manual spreadsheet.

## Requirements
- Hardcoded roster of 11 players: Alex, Leo, Alistair, Gus, Harrison, Hamish, Luca, Liam, Caspar, Prim, Jian.
- 4 innings per game.
- Positions (9): Pitcher, Catcher, 1st Base, 2nd Base, 3rd Base, Shortstop, Left Field, Centrefield, Right Field. Plus Bench.
- Batting order: random per game, fixed for the whole game.
- Rules, per game: no player benched more than once; no player plays the same position twice.
- Output table: columns Batting Order, 1st, 2nd, 3rd, 4th; rows in batting order; cells hold positions.
- Absent players are selected first (usually 0 to 2, sometimes more).
- Every step (batting order, each inning) has Regenerate and Accept.
- Reset clears everything for the next game.
- The page remembers an in-progress game across refresh or sleep (`localStorage`).
- Visual design is free (the old spreadsheet colors need not be copied). Mobile-first, light and dark themes.

## Flow
Six steps: Absent, Batting Order, Inning 1, 2, 3, 4.
1. **Absent:** tap player chips to mark absent; Continue.
2. **Batting order:** random shuffle of present players; Regenerate / Accept.
3. **Innings 1 to 4:** generated one at a time; Regenerate / Accept.
- Accepted steps are locked. **Undo** reopens the most recently accepted step. **Reset** (with confirm) returns to step 1 with no absentees.

## Inning generation
`genInning(present, history)` is a pure function. `history` holds, per accepted inning, a map of player to position (or Bench).
- `n >= 9`: all 9 positions are filled. The `n - 9` benched players are chosen only from players never benched in an accepted inning.
- `n < 9`: `9 - n` outfield positions (LF, CF, RF) are dropped, chosen randomly each inning. Minimum supported `n` is 6; below that the app shows a message and does not generate.
- Fielders are assigned by shuffled backtracking so that no player repeats a position they already played in an accepted inning.
- If no valid assignment exists, return `null`; the UI shows a message and offers Undo.
- Max `n` is 11, giving at most 8 bench turns over 4 innings, so the bench rule is always satisfiable.

## UI
- Table rows in batting order. Accepted innings are solid; the inning being previewed is highlighted; future innings are empty placeholders.
- Absent players appear as chips above the table, not as rows.
- Position chips are color-coded; Bench is muted red.
- Large tap targets for phone use. Built with the frontend-design skill.

## Branding and visual direction
- Team: **Thornleigh Thunderbolts** (Thornleigh Baseball Club, est. 1979).
- Palette from the shirt and club logo: royal blue, lightning gold, navy, white. Dark theme leans navy; light theme leans white with blue and gold accents.
- Style: pro-sports broadcast look (MLB scoreboard and lineup-card inspired). Condensed bold italic display type for the header and column heads, clean sans for the table, jersey-style batting numbers, gold accents.
- Header: team name wordmark plus an inline-SVG lightning bolt mark drawn for the page.
- Logos stay inside the single file (inline SVG or base64 data URI). The shirt mascot is a photo of a shirt on a blue background and cannot be cleanly reused; the mascot is used only if a clean PNG is supplied, otherwise the SVG bolt stands in.
- Printing the lineup card is still out of scope, but table styling should look good on a screenshot.

## State and persistence
One state object `{absent, order, innings, step}` saved to a single `localStorage` key on every change. Reads and writes are wrapped in try/catch; the app works without storage.

## Code layout
Single `index.html`. A `PLAYERS` array at the top for editing the roster. `genInning` holds all logic. Opening the page with `#test` runs a self-check: 1000 random games for each player count 6 to 11, asserting every position is filled (or correctly dropped), no repeated bench, no repeated position. Results print to the console and the page.

## Out of scope
Player pairing preferences, multiple teams, print or export. These can be added later.
