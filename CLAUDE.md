# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A tic-tac-toe game ("Noughts & Crosses") built as one self-contained `index.html`: inline CSS and inline vanilla JS, with Google Fonts as the only external dependency. There is no build step, package manager, linter or test suite.

- **Run locally:** open `index.html` in a browser.
- **Deployed:** GitHub Pages at https://tanaysaveshere.github.io/tic-tac-toe/, served straight from `main`. Pushing to `main` publishes the change.

## Architecture (inside `index.html`)

**State.** A single mutable object `S`, created by `fresh()`, holds `mode` (`"cpu"`/`"pvp"`), `diff` (`"easy"`/`"medium"`/`"hard"`), `score {X, O, D}`, `starter`, `board` (9 cells: `null`/`"X"`/`"O"`), `turn` and `result` (`{who, line}` from `winner()`, or `null`).

**Render loop.** Every state change ends in a call to `render(justPlayed?)`, which rebuilds all nine cell buttons, the strike-through line, the status text, the scoreboard and the control states from `S`. Nothing is patched in place, so to change something you update `S` and call `render`. Only the cell at index `justPlayed` gets the `draw` animation class.

**Turn flow.** `play(i)` is the human input path (clicks and keys 1–9). It ignores input while it's the CPU's turn. `place(i)` writes the mark, checks for a win or draw, updates the score and flips `turn`. In CPU mode the human is always X and the computer is always O. CPU moves run on a `setTimeout` delay, and that callback re-checks `S.turn`/`S.result` because the state might have changed while it waited.

**Round starts.** `newRound()` swaps `starter` before resetting the board. That's why changing mode or difficulty sets `S.starter = "O"` first: it makes X move first in the new round. Changing mode or difficulty also resets the score.

**AI (`cpuMove`).**
- Easy: picks a random square 60% of the time, otherwise plays as Unbeatable.
- Medium: takes a winning move, then blocks the opponent's winning move, then prefers the center (70%), then plays randomly. Forks beat it.
- Hard ("Unbeatable"): full `minimax` with depth-adjusted scores (`10 - depth` / `depth - 10`), with a random corner or center as the opening move.

**Hot-reload hooks.** The script ends by calling the optional `window.claude?.hot` API (`snapshot`/`ready`/`data`), which keeps `S` across live reloads in a Claude artifact preview. Those calls do nothing in a normal browser. Keep them, and keep the `start(data)` entry point that restores `S` from `data.S`.

**Theming.** Colors are CSS custom properties on `:root`, overridden for dark mode both through `prefers-color-scheme` (guarded by `:not([data-theme="light"])`) and through `[data-theme="dark"]`. A new color needs a value in all three places. Animations use `stroke-dasharray` with `pathLength="1"` and turn off under `prefers-reduced-motion`.
