# Podyezd

First-person browser horror game in a single file. Night stairwell of a panel house, the player walks up to their flat while someone follows one floor below.

## Run

Open `index.html` in a browser, or `python3 -m http.server 8737` and go to http://localhost:8737.
Needs internet once for three.js (jsDelivr CDN, r160). No build step, no assets.

## Layout of index.html

- constants, renderer, canvas-generated textures
- build: 5 levels (0 lobby, 1-3 puzzle floors, 4 home), dog-leg stairs, lamps
- collision: `free(x,z,y)` for walls, `ground(x,z,y)` for height (floors repeat every 3 m)
- audio: WebAudio synth, `thud()` is the base for all steps and knocks
- follower: `F.c` closeness 0..1, grows while standing, shrinks while walking
- `window.__g` exposes state for testing in the console

## Status

Stage 1 done: movement, stairs, follower. Next: puzzles one at a time, changing rules sheet, finale.
