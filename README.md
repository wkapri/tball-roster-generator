# Thornleigh Thunderbolts Lineup Generator

A single-file web app that builds a T-ball game-day lineup: a random batting order plus fielding positions for 4 innings.

**Live:** https://wkapri.github.io/tball-roster-generator/

## How to use

1. Tap any players who are missing, then Continue.
2. Accept or Regenerate the batting order.
3. Accept or Regenerate each of the 4 innings.
4. Reset (bottom right) clears the game for next time. A page refresh keeps the game in progress.

## Rules

- No player is benched more than once.
- No player plays the same position twice.
- Fewer than 9 players: random outfield positions are dropped. Minimum 6 players.

## Edit the roster

Change the `PLAYERS` array at the top of the script in `index.html`.

## Run locally and test

Open `index.html` in a browser. No build step, no dependencies. Open `index.html#test` to run the self-check on the lineup logic.
