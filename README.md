# POLY-SIEGE: OPERATION OMEGA

A browser-based, low-poly tactical FPS inspired by *Rainbow Six: Siege* — built entirely in **Three.js** and **vanilla JavaScript**, with no build step, no backend, and no external art assets. Everything you see (the gun, the house, the defenders, the bullets) is procedurally generated from primitive geometry at runtime.

Run it by opening a single HTML file in a browser.

---

## Play it

1. Download `poly-siege.html` from this repo.
2. Open it in a modern desktop browser (Chrome/Edge/Firefox). No server, no install, no build — it loads Three.js from a CDN via an import map.
3. Choose **VS BOTS** from the main menu, pick an operator, and you're in.

> **Note on the "ONLINE" menu option:** it's there, but it's intentionally non-functional. This is a single static file with no backend to matchmake players — see [Multiplayer](#multiplayer) below for what it would actually take to add real online play.

---

## Features

### Core shooter
- First-person movement: WASD, mouse-look, sprint, toggleable crouch and lean (Q/E)
- Hitscan weapons with a visible, straight-flying bullet projectile (separate from the invisible hit-detection ray, so it never looks like it "bends")
- Recoil, reload animation (with a visible magazine swap), ADS with FOV zoom and a hollow, see-through holographic scope
- Dynamic crosshair, hit markers, damage vignette, weapon sway/bob

### The Killhouse
- A two-story procedurally built building: multiple entry points (door, two windows, a roof hatch), an interior staircase, a drop hatch (floor 2 → floor 1, no stairs), and an exterior ladder
- Door and window trim, a roof parapet, interior/exterior cover crates
- **Two distinct ways to break through walls:**
  - **Reinforced interior walls** — immune to gunfire and melee; only Assault's Breaching Charge destroys them (instantly)
  - **The main door** — a closed, 3-panel green door. Melee (**V**) knocks out one panel per hit (3 hits to clear it); gunfire takes exactly **27 bullet hits** regardless of weapon damage; a shotgun blast clears it in one hit

### Enemies (bots)
- Finite-state-machine AI: Patrol → Combat → Investigate → Flank → Dead, with headshot/torso/limb hitboxes and per-part damage multipliers
- Enemies patrol and go about their business from the moment the round starts, but can't see or engage the player until Action Phase begins — they're not aware you're still scouting

### Round system
- **Scouting phase**: every round starts with the attacker piloting a small drone (not their body) to scout the map — WASD to fly, Space to hop, Shift to boost, with real collision against walls and gravity that respects whichever floor it's hovering over. You're locked into it until the Prep timer runs out; there's no early deploy.
- **Prep → Action → Round Over** flow with a plant/defuse bomb objective
- **Three operators**, each with a different weapon and a one-per-round gadget:

  | Operator | Weapon | Gadget |
  |---|---|---|
  | Assault | Rifle (30 rd, 700 RPM) | Breaching Charge — destroys the nearest reinforced wall within 3m |
  | Recon | SMG (25 rd, 900 RPM) | Deployable scouting drone |
  | Support | Shotgun (8 shells, pellet spread) | Deployable bullet-blocking shield |

### HUD
- Health bar, ammo counter, kill feed, dynamic crosshair, hold-Tab scoreboard, round timer/phase readout, and a Settings panel (mouse sensitivity, FOV, invert Y)

---

## Controls

| Action | Key |
|---|---|
| Move | `W A S D` |
| Look | Mouse |
| Sprint | `Shift` |
| Crouch (toggle) | `C` |
| Lean left / right (toggle) | `Q` / `E` |
| Fire | Left Mouse |
| Aim down sights | Right Mouse |
| Reload | `R` |
| Melee | `V` |
| Gadget | `F` |
| Plant / defuse bomb (hold) | `G` |
| Scoreboard (hold) | `Tab` |
| Pause / Settings | `Esc` |

While piloting a drone (scouting phase or Recon's gadget): `W A S D` fly, `Space` hop, `Shift` boost.

---

## Tech notes

- **Single file, no build step.** Three.js and its addons (the `Sky` shader) load via an ES module import map pointing at a CDN — open the HTML file directly and it works.
- **Everything is procedural geometry.** No textures, no `.glb`/`.gltf` models, no audio files (audio/"game juice" is a planned next step — see [Roadmap](#roadmap)).
- **Collision** is a lightweight custom system: static axis-aligned bounding boxes for walls, a raycast-based ground check (with enough tolerance to walk up stairs without a jump), and the same wall-collision logic reused for the player, enemies, and drones.
- **Rendering**: a real atmospheric-scattering sky (Three.js's `Sky` addon) instead of a flat background color, with lighting tuned for visibility rather than moody realism.

---

## Multiplayer

The "ONLINE" menu entry is a placeholder by design, not a bug — a static HTML file can't matchmake players on its own. Making this genuinely multiplayer would mean:

1. **A backend server** (Node.js + a WebSocket library like `ws`, or a framework built for this like Colyseus) that owns the authoritative game state — player positions/health, round phase, bomb state, which walls are destroyed — so clients can't just lie about having won.
2. **Rooms/matchmaking** to group connecting players into a match.
3. A **client ↔ server loop**: each client sends input to the server, the server validates and broadcasts resulting state to everyone in the room, and each client renders other players from that state instead of from local AI.
4. **Client-side prediction + server reconciliation** to hide latency — move locally the instant input happens, then quietly correct if the server disagrees.
5. Swapping the bot FSM out for remote-player rendering driven by network state.
6. **Hosting** the server somewhere with a public address and persistent uptime (Railway, Fly.io, Render, a VPS, etc.).

Note: pages like this one, when hosted via a sandboxed preview environment, typically run under a content-security policy that blocks arbitrary outbound connections — so real multiplayer would need to be served as its own standalone web app (this HTML file plus your own server), not from inside a locked-down preview.

---

## Roadmap

This was built step by step against a full original spec. Completed so far:

- [x] Scene, player controller, weapon system, view-model + scope
- [x] Enemy AI, hitboxes, damage
- [x] Killhouse level + destructible walls
- [x] Round system, operators, gadgets, scouting drone phase
- [x] HUD, menus, settings

Still open:

- [ ] Audio (Web Audio API — gunshots, footsteps, reloads, explosions)
- [ ] Additional game juice (screen shake, blood particles, slow-mo on the last kill of a round)

---

## Known limitations

- Single-player vs. bots only (see [Multiplayer](#multiplayer))
- No persistent progression/stats between sessions
- Enemy pathfinding is a simple waypoint graph on the ground floor only — bots don't currently navigate stairs/ladders to reach upper floors
- No audio yet

PS. This is all VibeCoded
