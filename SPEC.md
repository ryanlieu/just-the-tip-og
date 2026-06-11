# JUST THE TIP OG — Game Spec

A single-file browser game recreating OG Anunoby's game-winning tip-in for the Knicks in Game 4 of the 2026 NBA Finals. Plays like an old-school Flash game, drawn like MS Paint.

## 1. Concept

The shot goes up, it rims out — and OG (#8, New York) comes flying in from half court at the last second to tip the rebound back in over a comically tall Victor Wembanyama. You get **unlimited attempts** — keep crashing the glass until it drops. **One make wins the Finals.**

## 2. Tech

- **One file: `index.html`** — vanilla JS + `<canvas>`, zero dependencies, no build step. Open it in a browser and play.
- Fixed logical resolution ~**960×600** (landscape side view), scaled to fit the window with `image-rendering: pixelated` so the chunky art stays chunky.
- 60fps `requestAnimationFrame` loop with fixed-timestep physics.
- All sound synthesized with **WebAudio** (no audio files).
- All art drawn in code with canvas primitives (rects, arcs, crude lines) in the MS Paint style of the reference screenshot.

## 3. Scene & Art Direction (per the reference screenshot)

Side view of the right half of the court. Basket on the **right** edge: gray backboard with crooked panel lines, orange rim, scribbly white net, a gray support arm coming in from offscreen.

- **Background:** an actual MSG crowd photo, crunched — "New-York Knicks in the Madison Square Garden" by vincent desjardins (Wikimedia Commons, CC BY 2.0), cropped into arena tiers (jumbotron + dark upper deck, packed lit lower bowl), downscaled to 320×180 with a 64-color palette, embedded as base64, and drawn pixelated at 3× so it stays recognizably MSG but crunchy. A light haze keeps the action popping, with a courtside ad board ("THE GARDEN ✶ NEW YORK") at the base. Tan/yellow court floor with wobbly hand-drawn paint lines.
- **OG (player):** the #8 character from the screenshot — brown skin, flat-top hair, angry determined eyebrows, white "New York" jersey with orange **8**, noodle arms that stretch up when he jumps, comically big hands (the airborne tipping mitt is the gameplay hitbox). ~90px tall.
- **Wembanyama (defender):** absurdly tall — roughly **1.8× OG's height** — skinny noodle limbs, black Spurs-style jersey with a **1**, blank menacing stare. His standing reach already grazes the rim.
- **Other defenders (2):** full cartoon characters in the same style as Wemby — **Stephon Castle (#5)** (short crop) and **Dylan Harper (#2)** (curly mop), black Spurs jerseys, arms spread wide in a shuffling defensive stance. No name labels. All players share the same brown skin-tone family. On the ground they body-check OG ("OOF", knockback) — but landing on their **heads** springs OG back up Mario-style ("BOING!", a modest hop, weaker than a full jump).
- **The shooter:** **Jalen Brunson (#11, New York)**, bearded, frozen in shooting follow-through at the left side of the court — the missed shot launches from his hand each try. The miss is HIS.
- **Ball:** orange circle, black seam lines, slight squash on bounce.

## 4. Controls

| Key | Action |
|---|---|
| ← / → or A / D | Run left / right |
| ↑ or W | Jump (alias) |
| Space | Jump |
| Space (menus) | Start / play again |

- OG runs fast (he's sprinting in from half court) with slight acceleration/skid.
- One jump per airborne arc, fixed jump impulse, normal gravity. While airborne his arm is fully extended overhead — his **hand hitbox** is a small circle at the fingertip, and it's the only thing that can tip the ball.

## 5. Core Loop — One Try

1. **Reset:** OG spawns at the far **left** edge (half court). Defenders take up positions in/around the paint. The shot is already in the air toward the rim.
2. **The miss:** Brunson's shot hits the rim and **rebounds randomly** — random bounce angle/strength off the iron, sometimes a second rim bounce, sometimes a high lazy arc, sometimes a fast carom. Every try looks different. The ball uses floatier gravity than the players (flash-game hang time), and an **untipped** ball that drifts back over the cylinder rattles out rather than scoring — only OG's tip can win it.
3. **The run:** you sprint right, weaving past ground defenders, and time your jump.
4. **The tip:** if OG's hand circle overlaps the ball while the ball is above rim height ±~60px, the ball is tipped **toward the hoop** — the exact trajectory depends on where the hand contacts the ball (under it = pop straight up and in; side = angled tip). A clean contact near the rim is nearly automatic; tipping from far away sends it on a riskier arc that can rim out.
5. **Resolution — a try ends when:**
   - **MAKE** — ball passes down through the rim cylinder → **YOU WIN.**
   - **MISS** — tipped ball rims out and hits the floor.
   - **BLOCK** — Wemby's hand touches the ball.
   - **DEAD BALL** — untouched rebound hits the floor.
6. After any non-make, an announcer text pop (see §9) and a brief beat (~1.5s), then reset to step 1. Attempts are unlimited; the HUD counts them up.

## 6. Defenders

- **Wembanyama — the shot blocker.** Patrols the paint, sliding side to side, tracking the ball's predicted landing spot. He **jumps to swat** when you tip the ball near him or when OG leaves his feet nearby. His swat only counts **mid-jump and only against a tipped ball** (playtesting showed an always-on hand hitbox let him passively kill every rebound — unwinnable). If his hand touches your tip, it's a **BLOCK**: ball spiked to the floor, try over. He has a recovery window after landing (~0.8s) — baiting his jump and tipping behind him is the core skill move.
- **Stick defenders ×2 ("A PLAYER", "ANOTHER PLAYER").** Slow ground obstacles shuffling horizontally in the mid-court area. Touching one **body-checks** OG: knocks him back and kills his momentum (costing precious time), but never ends the try directly. They can't jump and never touch the ball.

## 7. Difficulty

One fixed, hand-tuned difficulty (no levels, no progression, nothing persisted): Wemby patrols at a beatable speed with a generous post-swat recovery window, rebounds are mildly random, and the two Spurs defenders shuffle at a hoppable pace. There is no game over — you attempt until you make it, and winning just offers to run it back.

## 8. Game States

1. **Title screen** — MS Paint title card: wobbly hand-drawn "JUST THE TIP OG" logo, OG sprite, "PRESS SPACE TO START", and a 2-line how-to-play (←→/AD run · SPACE/W jumps · tip the ball in).
2. **Playing** — HUD top-left: a little MS Paint basketball with the current attempt number. Scoreboard strip: NYK 105 – SAS 106, "Q4 0:00.5".
3. **Try resolution** — announcer text pop (see §9), short pause, reset.
4. **Winning make → slow-mo replay** — the last ~1.2s of ball + players replays at 0.3× speed (replay buffer of recent frame states), white "INSTANT REPLAY" text flickering in the corner, then →
5. **Victory screen** — confetti rectangles, scoreboard flips to NYK 107 – SAS 106, giant **"KNICKS WIN GAME 4"**, attempts used, "PRESS SPACE TO RUN IT BACK".

## 9. Announcer Text

Big wobbly comic-style text pops (slight rotation, pop-in scale, 1s hold):

- Make: **"OG WITH THE TIP!! KNICKS WIN GAME 4!"**
- Block: **"BLOCKED BY WEMBY!"** / "NOT IN MY HOUSE"
- Any miss or dead ball (random, shared pool): **"EVEN SPIKE LEE CAN'T WATCH"** / "SAME OLD KNICKS..." / "THE GARDEN GOES QUIET..." / "TAKING THE 7 TRAIN HOME..." / "TIMOTHÉE CHALAMET SIGHS COURTSIDE"
- Body-check: small "OOF" puff at contact · Head stomp: "BOING!"

## 10. Sound (WebAudio, all synthesized)

- Ball bounce (pitch drops with bounce energy) · rim **clank** (metallic square-wave hit) · **swish** (filtered noise burst) · jump *boing* · block **swat** + crowd "ooooh" · crowd roar on the make (big noise swell) · sad buzzer on game over · low crowd-murmur loop during play · slow-mo pitch-down during replay.

## 11. File Structure

```
og_tip_in/
├── SPEC.md          ← this file
└── index.html       ← entire game (markup + CSS + JS inline)
```

## 12. Out of Scope (v1)

Mobile/touch controls, online leaderboards, multiple playable characters, real NBA assets/likeness art (everything stays crude on purpose).
