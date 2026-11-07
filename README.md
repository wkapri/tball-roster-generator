# Thornleigh Thunderbolts Lineup Generator

A single-file web app that builds a T-ball game-day lineup: a random batting order plus fielding positions for 4 innings.

**Live:** https://wkapri.github.io/tball-roster-generator/

## How to use

1. Tap any players who are missing, then Continue.
2. Accept or Shuffle the batting order.
3. Accept or Shuffle each of the 4 innings.
4. Use Undo to reopen the last accepted step. Use Reset to clear the game for next time. Both are at the top right of the panel. A page refresh keeps the game in progress.

### Table and field views

Once an inning exists, switch between Table and Field. Field draws a baseball diamond with each player at their position, and a tab for each inning. Benched and sitting-out players are listed under the diamond.

### Sitting a player out

If a player is injured or needs a break, tap their cell in the table for that inning. Pick who covers: a benched player, an outfielder, or leave the position empty. The player shows as Out and the substitute takes the position. Edited cells have a dot; tap one and choose "Undo change" to restore the inning. Edits don't re-check the rules below, and regenerating an inning you haven't accepted yet clears its edits.

## Rules

- No player is benched more than once.
- No player plays the same position twice.
- Fewer than 9 players: random outfield positions are dropped. Minimum 6 players.

These apply when lineups are generated. Manual sit-out edits can break them.

## Edit the roster

Change the `PLAYERS` array at the top of the script in `index.html`. The list sorts itself alphabetically.

## Run locally and test

Open `index.html` in a browser. No build step, no dependencies. Open `index.html#test` to run the self-check on the lineup logic.

## Design docs

- Design: `docs/superpowers/specs/2026-11-07-tball-roster-design.md`
- Implementation plan: `docs/superpowers/plans/2026-11-07-tball-roster.md`
