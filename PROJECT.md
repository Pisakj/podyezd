# Podyezd

First-person browser horror game: `index.html` plus `sounds.js` (recorded sounds in base64, sources in `SOUNDS.md`). Night stairwell of a panel house, the player walks up to their flat while someone follows one floor below.

## Run

Open `index.html` in a browser, or `python3 -m http.server 8737` and go to http://localhost:8737.
Needs internet once for three.js (jsDelivr CDN, r160). No build step, no assets.

## Layout of index.html

- constants, renderer, canvas-generated textures
- build: 7 levels, dog-leg stairs, lamps. 0 lobby; 1, 3, 5 puzzle floors; 2, 4 empty floors; 6 home
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
| `order` | random | Puzzles on floors 1, 3, 5, e.g. `switch,back,loop` |
| `dark` | random | Which empty floor (2 or 4) has the announced darkness. The other one gets the flickering lamp |
| `shepard` | 0.012 | Volume of the falling tone on the loop floor |
| `res` | 300 | Picture height in pixels (120-1080), stretched to the window. Lower is coarser |
| `trail` | 1.5 | How many flights behind the player the footprints stay |
| `show` | none | `steps`, `window`, `door` (comma separated): make that quiet visual event happen at once, for checking |

Other values worth knowing, constants in `index.html`: `NEAR` (0.88, closeness at which everything goes silent), `HINT_AFTER` (90 s stuck on a floor before the door gap shows the answer), `DOOR_SND_COUNT` (3 flats with sound), `minC` in `updateFollower` (0.08 per floor, 0.48 on the top floor), silence before the catch (6-10 s), torch dropouts (0.3-1.5 s every 40-90 s), distant sounds (`farT`, every 40-90 s).

## Design rules

Fear comes from silence, waiting and unpredictability, not from more events. Less happens, each event weighs more. No jump scares, the follower is never shown, the player cannot lose. The follower catches the player at most once per run.

One effect per floor at most:

| Floor | Its one effect |
|---|---|
| 0 lobby | nothing; the follower is not awake yet |
| 1, 3, 5 | the puzzle (with its rules sheet notes and, after 90 s stuck, a door that opens with the answer). The loop floor also plays the Shepard tone until solved |
| 2 or 4 | announced darkness: the sheet says the light goes out for 20 s; 15-25 s after stepping onto the floor all lamps and the torch die for exactly 20 s, steps come up to 0.8 and stop, then nothing. Once per run, no catch during it. The clock only runs while the player is on that floor |
| the other of 2, 4 | nothing but a flickering lamp. No sounds behind doors, dark peepholes |
| 6 home | the finale |

Peepholes: E on any closed flat door looks in. A blurred glow if the light is on inside, black if not. Once per run at most (30% per look) the glow is slowly covered from the other side.

Quiet visual events. They make no sound, nothing is sudden, and nothing appears or changes on screen: every change waits until the spot is off screen (`unseen()`, by camera direction). None of them is on a puzzle floor.

| Event | Where | What |
|---|---|---|
| Footprints (`updateTrail`) | stair treads only | Wet prints follow about 1.5 flights behind, left and right in turn. After the player has stood 3 s the chain ends in a pair side by side. A print dries after 60 s |
| Window opposite (`mYard`, `updateYard`) | landing windows | Each pane is a small ray tracer (a shader): it finds what the line from the eye through the glass hits outside: the yard ground, a street lamp 9 m out, the facade of the house opposite 22 m away, or the sky. So the view has real parallax against the frame. One window opposite is dimly lit with a still dark shape in it, always on the floor level with the player and on the player's line of sight through the window; it is moved only while the windows are off screen. From level 4 the shape is larger |
| Door behind (`updateBehind`) | the empty floor without the darkness | Once per run the door by the stairs stands ajar after the player has climbed a flight |
| Scribble (`updateGraffiti`) | one landing, never the first | "НЕ СТОЙ" reads "НЕ СТОЙ ЗДЕСЬ" after it has been looked at once |

Random events (a lamp going out, the torch cutting out, distant sounds, a step behind, fingers on a rail) fire only on the stairs between floors. The follower itself works everywhere above the lobby.

## Testing

`?order=switch,back,loop&dark=2` fixes the puzzles on floors 1, 3, 5 and the dark floor (random otherwise).
To test the darkness without waiting, set `__g.D.timer = 0.5` once the player is on that floor.
A full run can be scripted through `window.__g` (player state, puzzle state) with synthetic key events.

## Status

Whole game is in and tuned for quiet dread (2026-11-16): seven levels with empty floors, announced darkness, Shepard tone, peepholes, quiet visual events, coarse 300-line picture. Now: playtests and manual tuning of the values above.
