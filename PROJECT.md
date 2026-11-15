# Podyezd

First-person browser horror game: `index.html` plus `sounds.js` (recorded sounds in base64, sources in `SOUNDS.md`). Night stairwell of a panel house, the player walks up to their flat while someone follows one floor below.

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

## Deploy

GitHub Pages from `main`, root. Repo: https://github.com/Pisakj/podyezd, live at https://pisakj.github.io/podyezd/. A push to `main` republishes in about a minute.

## Tuning

URL parameters, all optional:

| Parameter | Default | Meaning |
|---|---|---|
| `fog` | 0.2 | Fog density. Higher: things vanish sooner |
| `torch` | 6 | Torch brightness |
| `rate` | 35 | Seconds of standing still for the follower to come all the way |
| `speed` | 1.6 | Walking speed in m/s. Stairs are slower in proportion (x0.81) |
| `order` | random | Puzzle on floors 1-3, e.g. `switch,back,loop` |

Other values worth knowing, constants in `index.html`: `NEAR` (0.88, closeness at which everything goes silent), `HINT_AFTER` (90 s stuck on a floor before the door gap shows the answer), `DOOR_SND_COUNT` (3 flats with sound), `minC` in `updateFollower` (0.12 per floor), silence before the catch (6-10 s), torch dropouts (0.3-1.5 s every 40-90 s), distant sounds (`farT`, every 40-90 s).

## Design rules

Fear comes from silence, waiting and unpredictability, not from more events. Less happens, each event weighs more. No jump scares, the follower is never shown, the player cannot lose. The follower catches the player at most once per run.

## Testing

`?order=switch,back,loop` fixes which puzzle is on floors 1-3 (random otherwise).
A full run can be scripted through `window.__g` (player state, puzzle state) with synthetic key events.

## Status

Whole game is in and tuned for quiet dread (2026-11-15). Now: playtests and manual tuning of the values above.
