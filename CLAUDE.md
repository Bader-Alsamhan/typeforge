# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file browser typing speed game (`index.html`) — no build step, no dependencies, no package manager. Open directly in a browser.

## Running the Game

Open `index.html` in any modern browser. To serve it (e.g. for preview tools):

```bash
python3 -m http.server 3799
```

Then visit `http://localhost:3799`.

## Architecture

Everything lives in one file: `index.html`. The `<script>` block is structured as a series of IIFEs and module-like objects:

| Object | Role |
|---|---|
| `SoundFX` | Web Audio API synth — no external audio files |
| `WordBank` | Sentence lists by difficulty; `getWords(diff, count)` stitches sentences into a word array |
| `XP` | Level/XP system; persists to `localStorage` under key `typeforge_xp` |
| `Store` | Leaderboard, streak, game count — all `localStorage` under `typeforge_*` keys |
| `Achievements` | Badge definitions + unlock tracking (`typeforge_ach` in localStorage) |
| `Survival` | Canvas-based falling-words mode — self-contained IIFE with its own `requestAnimationFrame` loop |
| `state` | Single global mutable object shared by all game logic |
| `startGame / endGame` | Top-level flow control; branch by `state.mode` |

### Game flow

Menu → `startGame()` → countdown overlay → mode-specific start → typing loop → `endGame()` → results screen → back to menu or replay.

Screens (`#screen-menu`, `#screen-game`, `#screen-results`) are all `position: fixed`; visibility is controlled by toggling the `.hidden` CSS class (opacity + pointer-events, not display).

### localStorage keys

All keys are prefixed `typeforge_`: `xp`, `streak`, `scores`, `games`, `ach`.

## Key Constraints

- **No build tooling** — edits go directly into `index.html`.
- **`const Audio` is taken** — the sound module is named `SoundFX` to avoid shadowing the browser's built-in `Audio` constructor.
- **All localStorage access must be wrapped in try/catch** — sandboxed iframes and some private-browsing contexts throw `SecurityError` on storage access, which would kill the entire script before event listeners are registered.
- Survival mode runs its own `requestAnimationFrame` loop stored in a closure; call `Survival.stop()` before navigating away or it keeps running.
