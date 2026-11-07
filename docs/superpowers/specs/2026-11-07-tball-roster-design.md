# T-Ball Roster Generator: Design

Status: implemented. This document describes the app as it stands, including changes made after the first release (field view, sit-outs, panel layout).

## Purpose
A single self-contained `index.html` (no build step, no dependencies, hosted on GitHub Pages) that a coach uses to generate a game-day lineup for a team of 11 six-year-olds. It replaces a manual spreadsheet. The lineup is usually announced the night before the game.

## Requirements
- Hardcoded roster of 11 players in a `PLAYERS` array: Alex, Leo, Alistair, Gus, Harrison, Hamish, Luca, Liam, Caspar, Prim, Jian. The array sorts itself alphabetically, so the list on screen is always in order.
- 4 innings per game.
- Positions (9): Pitcher, Catcher, 1st Base, 2nd Base, 3rd Base, Shortstop, Left Field, Centrefield, Right Field. Plus Bench.
- Batting order: random per game, fixed for the whole game.
- Rules, per game: no player benched more than once; no player plays the same position twice. These apply when lineups are generated. Manual sit-out edits (below) may break them.
- Output table: columns Batting order, 1st, 2nd, 3rd, 4th; rows in batting order; cells hold positions.
- Absent players are selected first (usually 0 to 2, sometimes more).
- Every step (batting order, each inning) has Shuffle and Accept.
- Reset clears everything for the next game.
- The page remembers an in-progress game across refresh or sleep (`localStorage`).
- Mobile-first, light and dark themes.

## Flow
1. **Players:** tap player chips to mark absent; Continue.
2. **Batting order:** random shuffle of present players; Shuffle / Accept.
3. **Innings 1 to 4:** generated one at a time; Shuffle / Accept.
- Accepted steps are locked. **Undo** reopens the most recently accepted step. **Reset** (with confirm) clears the game. Undo and Reset sit side by side on the title row of the top panel.
- There is no separate step indicator row; the panel title names the current step ("Batting order", "1st inning", "Lineup is set") and the Accept button is labelled to match ("Accept order", "Accept 2nd").

## Inning generation
`genInning(present, history)` is a pure function. `history` holds, per accepted inning, a map of player to position (or Bench or Out).
- `n >= 9`: all 9 positions are filled. The `n - 9` benched players are chosen only from players never benched in an accepted inning.
- `n < 9`: `9 - n` outfield positions (LF, CF, RF) are dropped, chosen randomly each inning. Minimum supported `n` is 6; below that the app shows a message and does not generate.
- Fielders are assigned by shuffled backtracking so that no player repeats a position they already played in an accepted inning.
- If no valid assignment exists, return `null`; the UI shows a message and offers Undo.
- Max `n` is 11, giving at most 8 bench turns over 4 innings, so the bench rule is always satisfiable.

## Table view
- Rows in batting order, with a numbered badge per player. Accepted innings are solid; the inning being previewed is highlighted; future innings are empty placeholders.
- Players marked absent appear in an "Out today" line in the top panel, not as rows.
- Position pills are color-coded by zone: gold for pitcher and catcher, blue for infield, teal for outfield, muted red for Bench, grey for Out. On phones the pills use short labels (P, C, 1B, 2B, 3B, SS, LF, CF, RF); on wider screens they show full names.

## Field view
- A Table | Field toggle appears once at least one inning exists. Switching views must not move the page's scroll position, so the view area keeps its height across a toggle.
- Field view draws a baseball diamond with each player's name at their position, using the same zone colors as the table. Shortstop and 2nd Base sit close to the 2nd-base bag.
- Inning tabs (1st to 4th) pick which inning is drawn. The default is the inning being previewed, or the latest accepted one. Innings with no lineup yet are disabled.
- Benched players show as chips under the diamond, and Out players as grey chips. A dropped outfield position (fewer than 9 players) shows as a dashed empty box.
- View and selected inning are display-only and are not saved.

## Sitting a player out
Used when a player is injured or needs a break during the game. Batting order is not affected.
- In Table view, tapping a position cell in any inning (accepted or previewed) opens a popover with a close button. It also closes on Esc or a tap outside it.
- The popover lists who can cover: benched players that inning ("from bench"), then each outfielder ("from Left Field", and so on), then "Leave empty". Infielders are not offered, since that would leave an infield hole.
- The player who sits out shows as a grey "Out", separate from Bench. The substitute takes their position. An outfielder who covers leaves their own spot empty.
- Edited cells carry a small dot. The popover for an edited inning offers "Undo change", which restores that inning to its original lineup.
- Edits do not re-check or re-generate anything. If an edit leads to a repeated position or a second bench turn, that is accepted by design.
- Shuffling an inning that is still being previewed clears its edits.

## Branding and visual direction
- Team: **Thornleigh Thunderbolts** (Thornleigh Baseball Club, est. 1979).
- Palette from the shirt and club logo: royal blue, lightning gold, navy, white. Dark theme leans navy; light theme leans white with blue and gold accents.
- Style: pro-sports broadcast look. Condensed bold italic display type for the header and panel titles, system sans for the table, jersey-style batting numbers, gold accents.
- Header: team name wordmark plus an inline-SVG lightning bolt. The title scales with screen width so it stays on one line on phones.
- Logos stay inside the single file. The shirt mascot is a photo on a blue background and cannot be cleanly reused; it will be added only if a clean PNG is supplied.

## State and persistence
One state object `{absent, order, innings, pending, error, orig}` saved to a single `localStorage` key (`tball-roster-v1`) on every change. `orig` holds each edited inning's lineup before manual edits, keyed by inning index. Reads and writes are wrapped in try/catch; the app works without storage, and unreadable or old saved data falls back to a fresh game.

## Code layout
Single `index.html`. A `PLAYERS` array at the top for editing the roster. Pure logic (`genOrder`, `genInning`, `subOut`) is separate from state actions and rendering.

Opening the page with `#test` runs a self-check: 1000 random games for each player count 6 to 11, asserting every position is filled (or correctly dropped), no repeated bench, no repeated position; plus state-action cases (minimum players, double-tap on Accept, Undo, Reset, bad saved data) and the sit-out logic. Results print to the console and the page.

## Out of scope
Player pairing preferences, multiple teams, print or export, regenerating later innings after a sit-out edit. These can be added later.
