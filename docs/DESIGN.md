# Star Charter — Pro Expansion · Design Doc

*Living document. Built on the existing `star-charter/index.html` prototype, which already
implements the core game. This doc covers only the expansion to "v1 Pro."*

---

## 1. Vision

A constellation-charting roguelite. Pilot a survey probe through escalating sectors of the
dark, charting constellations while dodging predictable-but-deadly space phenomena, and
refit with one of three upgrades every third plate to grow a run build. Cosmic, mysterious,
slightly ominous. Desktop-first, single HTML file, no dependencies.

Reference for feel: Death Must Die / survivor-likes — short runs, high replay, builds that
compound. But the verb is **chart**, not **kill**: combat (auto-weapons) is the safety net
that lets you keep charting under pressure.

---

## 2. What already exists (do NOT rebuild)

The prototype is far past "early dodge-and-collect." Already working:

- Constellation charting loop (waypoint stars → fill plate → warp onward).
- Drift control (currently *hold-to-thrust toward pointer, release to coast*).
- Refit: 1-of-3 every 3rd plate, **with per-upgrade stack caps already coded** (`max:`).
- Hazards: asteroids, comets, black holes (gravity-wave shove), space-fauna, shooting aliens.
- One auto-weapon (Survey Beam), shields, a phase/dodge chance.
- Generated audio, screen shake, sparks, popups, toasts.
- Records (best score + best plate, saved to localStorage) and a stats screen.
- Cosmic title screen + story framing ("The Glass Archive").

The expansion is additive. The engine, render loop, spawn director, refit system, and save
layer are reused as-is.

---

## 3. Locked decisions (this round)

| Topic | Decision |
|---|---|
| **Control** | Switch to **direct-follow + focus**. Ship continuously accelerates toward the cursor (no hold), capped speed + drift/overshoot. Hold **Focus** (Shift / right-mouse) to slow down and tighten for precision weaving. Fixes the "weird/disconnected mouse" feel for a high-APM hand. |
| **Structure** | **Soft sectors, no hard ending.** Stays endless and ramping, but named sectors + milestone plates give it a journey shape. |
| **First effort** | (1) Grow the **upgrade pool + add weapons**, (2) **Presentation**: attract-mode title + rich records. |
| **Later in v1** | Two probes; new hazards (pulsars + asteroid belts). Still shipping in v1, just sequenced after. |

---

## 4. Control spec — direct-follow + focus

Replaces hold-to-thrust. Each frame while the run is playing and the pointer is over the canvas:

- **Target** = current cursor position (no button needed).
- **Accelerate** toward target: `a = normalize(target − pos) * thrustA`, but **scaled down inside
  a small dead-zone** around the cursor so the probe can rest without jitter.
- **Integrate** velocity with drag → natural overshoot/drift. `Star Anchor` upgrade increases
  drag (tighter); `Ion Vanes` increases max speed/accel.
- **Pointer leaves canvas** → no target, coast (keep current velocity, apply drag).
- **Focus** (hold Shift or right-mouse): multiply max speed by ~0.45, raise drag, optionally
  shrink the effective hitbox slightly, and show a faint reticle. This is precision-on-demand —
  the "a bit of both at different times" you wanted, on a button.

Touch is out of scope for Pro (desktop-only), but the model degrades to tap-and-hold-to-aim if
ever needed.

---

## 5. Upgrade pool (first pass — target ~16, all with stack caps)

Four lanes so builds diverge. Caps prevent any single pick from breaking the run. `minPlate`
staggers when each can first appear. **Weapon types capped at 3 distinct** (per your "2–3 auto
weapons max").

### Movement
| id | Name | Cap | Effect |
|---|---|---|---|
| vanes | Ion Vanes *(exists)* | 4 | Higher top speed / acceleration toward cursor. |
| anchor | Star Anchor *(exists)* | 4 | More drag — tighter stops, less overshoot. |
| gyros | Focusing Gyros | 3 | Focus mode slows more and tightens hitbox further. |
| slip | Slipstream | 3 | Brief speed burst right after charting a star. |

### Survivability
| id | Name | Cap | Effect |
|---|---|---|---|
| aegis | Stardust Aegis *(exists)* | 3 | +1 shield layer, restore one now. |
| veil | Phase Veil *(exists)* | 3 | +12% chance to phase through a strike. *(Cap lowered from 4 — 48% dodge stacked with Focus + Aftershell trivialized deep sectors.)* |
| aftershell | Aftershell | 3 | Longer invulnerability window after a hit. |
| plating | Event Horizon Plating | 3 | Reduced black-hole shove + reduced near-hazard damage. |

### Charting / Collection
| id | Name | Cap | Effect |
|---|---|---|---|
| lens | Chart Lens *(exists)* | 3 | Wider charting radius — stars catch from farther. |
| magnet | Magnet Coil | 3 | Pickups/motes drift toward the probe. |
| eye | Surveyor's Eye | 3 | +score per star and bigger plate-complete bonus. |
| quickplot | Quick Plot | 3 | Chain bonus: stars charted within 4s of the last build escalating bonus score. *(Redefined — charting is instant proximity capture, so "faster charting" had nothing to act on.)* |

### Motes — *(added; required by Magnet Coil, the Collector archetype, and the records screen)*

Small drifting score pickups (+3), spawned ambiently (cap 6) and dropped by weapon kills
(~40% chance per destroyed rock). Magnet Coil pulls motes **and** supply pods; the attract-mode
AI seeks them; "motes collected" is a tracked record. Without them the Coil was a dead pick.

### Weapons (pool ships with exactly 3 distinct types — the "max 3 distinct" cap is future-proofing)
| id | Name | Cap | minPlate | Effect |
|---|---|---|---|---|
| beam | Survey Beam *(exists)* | 3 | 9 | Auto-fires at nearest rock/creature; faster recharge per level. |
| halo | Halo Drones | 3 | 6 | Orbiting bodies that damage threats on contact; +1 drone per level. |
| flare | Scatter Flare | 3 | 12 | Periodic radial burst clearing nearby small threats; bigger/faster per level. |

### Filler
| id | Name | Cap | Effect |
|---|---|---|---|
| cache | Supply Cache *(already a fallback)* | ∞ | Instant score + small shield top-up. Pool filler when nothing else is offerable. |

**Intended build archetypes (light synergy, nothing rigid):**
- **Turret/Survivor** — Beam + Halo + Flare + Aftershell → "feel safe after stacking."
- **Glass racer** — Vanes + Slipstream + Veil → outrun everything.
- **Collector** — Magnet + Lens + Eye + Quick Plot → chart fast, score high.
- **Tank** — Aegis + Plating + Aftershell → brute through the deep sectors.

---

## 6. Soft sectors + milestones

Keep the endless ramp; overlay a journey:

- **Sector I — The Near Dark** (plates 1–10)
- **Sector II — The Long Drift** (plates 11–20)
- **Sector III — The Deep** (plates 21–30)
- **Sector IV — Uncharted** (31+, the endless tail; difficulty keeps climbing)

At each milestone plate (11, 21, 31): a **name card** ("SECTOR II ✦ THE LONG DRIFT") + a
small reward (shield restore, or +150 if shields are full). Records track **deepest sector
reached** and **deepest plate**. No wall, but the run now *goes somewhere*.
*(No card at plate 1 — it collided with the existing "PLATE 01 ✦ BEGIN" toast; Sector I is
implied at run start.)*

---

## 7. Presentation

### Attract-mode title
Behind the logo, a real run plays itself: a lightweight AI probe seeks the nearest mote and
steers away from the nearest hazard, with reduced spawn rates, dimmed/blurred behind the glass
panel. Reuses the live update/draw loop with an `attract` state and an AI controller standing in
for the pointer. Cheap, because the engine already exists.

### Rich end-of-run records
Track per-run and persist personal bests:

- Deepest plate · deepest sector reached
- Time survived
- **Longest no-hit streak** (clean-flying time between hits)
- Stars charted · motes collected · score
- **Build taken** (the list of upgrades, as a compact loadout)
- Dodges / near-misses · aliens downed

Death/records screen shows the run's numbers and **flags any new personal record** with a marker.
Persist alongside the existing `best` / `bestPlate` in the save object.

---

## 8. Two probes (later in v1)

Implemented as tuning presets over existing knobs (`thrustA`, drag, shields, hitbox) + a
probe-select screen at run start.

| | **Surveyor** | **Skimmer** |
|---|---|---|
| Shields | High (start 2, cap higher) | Low (start 1) |
| Speed / accel | Slower, heavier drift | Fast, snappier |
| Hitbox | Slightly larger | Small |
| Trait | Bigger charting/magnet radius | Stronger Focus (extra slow + brief i-frames) |

---

## 9. New hazards (later in v1) — predictable / telegraphed

- **Pulsar** — fixed emitter with a rotating beam that sweeps the field on a readable rhythm;
  telegraphed (charge glow) before it fires. Contact damage. Learn the pattern, feel smart.
- **Asteroid belt / river** — a moving stream of clustered rocks with a **safe lane gap**;
  read it and slot into the gap. Avoid-only in v1 (destructible rocks = later). Variants:
  slow+dense vs fast+sparse.

(Black hole already covers the "gravity" niche, so it's not re-added.)

---

## 10. Build order

1. **Phase 1 — Feel + builds** *(first)*
   - Control change: direct-follow + focus.
   - Expand upgrade pool to ~16; add Halo Drones + Scatter Flare weapons.
   - Tune caps / minPlate gating; verify nothing breaks at max stacks.
2. **Phase 2 — Presentation** *(second)*
   - Attract-mode title.
   - Rich records + PR tracking; sector names + milestone cards.
3. **Phase 3 — Probes** — Surveyor vs Skimmer + select screen.
4. **Phase 4 — Hazards** — pulsar + asteroid belt.

Each phase is independently shippable and testable. After every phase: playtest, confirm 60fps,
confirm no crash at max upgrade stacks.

---

## 11. Parked (post-v1, not now)

Click-drift alt control mode · push-your-luck "lock-in" plates · destructible asteroids · boss
plates · more probes/weapons · between-run meta-progression · audio expansion · touch/mobile
adaptation layer (offset follow-target above the finger, second-finger Focus, larger hit targets
— touch drag already half-works, so this is an additive layer, not a separate build).

---

## 12. Open / to-confirm

- Exact tuning numbers (thrust, drag, focus multiplier, spawn ramps) — dial in during Phase 1
  playtests, not on paper.
- Naming of the two new weapons (Halo Drones / Scatter Flare are placeholders).
- ~~Whether Focus binds to Shift, right-mouse, or both.~~ **Resolved: both** (it was free).

---

## 13. Implementation log — v1 Pro shipped

All four phases implemented in `Star-Charter-Pro/index.html` (single file, no dependencies).
Deltas from the doc above, beyond the inline corrections already noted:

- **Controls**: direct-follow with an 80px arrive dead-zone, hard speed cap, Focus = Shift
  *or* right-mouse (cap ×0.45-ish via probe stat, +drag, hitbox ×0.85, crosshair reticle).
  Pointer leaving the window coasts. Keyboard steering retained as fallback.
- **Motes** added per §5 note. Beam/Halo/Flare kills drop them; Coil farms them.
- **Pulsar**: spawns plate 13+ (cap 1; 2 from plate 24), 1.5s dashed-line telegraph, then a
  sweeping bipolar beam for 5.5s. **Belt**: plate 8+, telegraphed band + teal safe-lane wash,
  ~8s stream, one gap lane.
- **Probes**: Surveyor (2 shields, heavy, wide lens + innate 60px coil) vs Skimmer (1 shield,
  fast, small hull, Focus grants 0.3s phase, 2.5s cooldown). Select screen between title and
  run; last choice persisted.
- **Records**: score, deepest plate/sector, time, longest clean streak, stars, motes, aliens
  downed, loadout line; per-stat ✦PR markers. Save key renamed `starCharterProSave` (schema
  grew; the classic prototype's save is untouched).
- **Attract mode**: ghost pilot seeks motes / flees rocks behind the title glass; resets on
  return to title.
- Stat tweaks while wiring: star score 10+4/Eye tier; plate bonus ×(1+0.25·Eye); milestone
  reward = shield-else-150; Stardust Cache now also restores one shield layer (per §5 filler row).
