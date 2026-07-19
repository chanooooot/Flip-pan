# SPEC — Pan-Flip Cooking Game (v1)

A one-handed, motion-controlled mini webapp: hold your phone flat like a frying pan, tilt and flip to cook Thai food. Zero cost, zero backend, single HTML file.

---

## 1. Concept

- Player holds phone **flat in palm, screen up, portrait** — like holding a pan.
- **Motion-only during play.** Touch is allowed only to start/restart.
- Food cooks on two sides. Player must **flip** (real pan-toss motion) to cook the other side.
- Burn either side → **instant game over**.
- **Endless survival**: cook speed ramps up over time. **Score = number of successful flips.**
- Thai theme. v1 food default: **ไข่ดาว (fried egg)** — swappable via config.

## 2. Player-Facing Flow

1. **Start screen** — title, best score (from localStorage), "Tap to Start" button.
2. Tap → requests iOS motion permission (if needed) + starts round in one action.
   - Permission denied → message "ต้องอนุญาตการเข้าถึงเซ็นเซอร์เพื่อเล่น" + retry button.
3. **Playing** — pan + food rendered on canvas. Current-side doneness visible (color: raw → golden → burnt). Score visible.
4. **Flip gesture** at the right time → food flips, other side starts cooking, score +1, brief feedback (animation/sound optional).
5. Side reaches burnt → **Game over screen**: score, best score (update localStorage if beaten), "Tap to Restart".
6. **Desktop / no motion sensors** → static message: "เกมนี้เล่นบนมือถือเท่านั้น" (feature-detect `DeviceMotionEvent`).

## 3. Motion Model

| Gesture | Signal | Game effect |
|---|---|---|
| **Tilt** (beta/gamma via DeviceOrientation) | Smooth angle | Food slides toward tilt direction inside pan (visual juice; excessive tilt may slide food to pan edge — v1 keeps it cosmetic, food cannot fall out) |
| **Flip** (pan toss) | Upward acceleration spike (DeviceMotion) **AND** rotation-rate spike (gyro) within a short window | Flips the food; only counts if current side is in the "flippable" doneness zone (see §4) |
| Shake | High-frequency accel oscillation without matching rotation pattern | **Must NOT trigger a flip** — explicitly rejected to avoid cheating by shaking |

- Flip detection = accel spike above `FLIP_ACCEL_THRESHOLD` + rotation rate above `FLIP_GYRO_THRESHOLD` within `FLIP_WINDOW_MS`, followed by a `FLIP_COOLDOWN_MS` lockout.
- **This is the riskiest part of the build.** See CLAUDE.md build order: implement and test flip detection in isolation first.

## 4. Cook Logic

- Each food has two sides: `sideA`, `sideB`, each with doneness `0 → 1.0` (1.0 = burnt).
- Only the **face-down side** cooks; doneness increases at `cookSpeed` per second.
- Doneness zones (per side):
  - `0 – GOLDEN_MIN`: raw/undercooked
  - `GOLDEN_MIN – GOLDEN_MAX`: **golden** (ideal flip zone)
  - `GOLDEN_MAX – 1.0`: overcooking (visual warning — darkening, maybe smoke)
  - `1.0`: burnt → game over
- **Flip rules:**
  - Flip while current side is in golden zone → success, +1 score, other side starts cooking from its current doneness.
  - Flip while raw → allowed, no penalty: sides simply switch, no score awarded (wasted flip).
  - No flip before 1.0 → burn → game over.
- **Difficulty ramp:** `cookSpeed` starts at `COOK_SPEED_START` and increases by `COOK_SPEED_RAMP` **per successful flip**, capped at `COOK_SPEED_MAX`.

## 5. Config Block (top of file, plain JS object)

All tunables in one named object, e.g.:

```js
const CONFIG = {
  // Flip detection
  FLIP_ACCEL_THRESHOLD: 12,   // m/s^2 — tune on real device
  FLIP_GYRO_THRESHOLD: 200,   // deg/s — tune on real device
  FLIP_WINDOW_MS: 150,
  FLIP_COOLDOWN_MS: 500,
  // Cooking
  GOLDEN_MIN: 0.55,
  GOLDEN_MAX: 0.85,
  COOK_SPEED_START: 0.08,     // doneness/sec
  COOK_SPEED_RAMP: 0.005,
  COOK_SPEED_MAX: 0.30,
  // Food
  FOOD: 'fried_egg',          // v1 default: ไข่ดาว
};
```

Values above are starting guesses — **must be tuned on a real phone**, not assumed correct.

## 6. Tech & Platform

- **Stack:** Vanilla HTML/JS/CSS, single file, `<canvas>` for pan/food rendering. No framework, no build step, no dependencies.

- **iOS:** `DeviceMotionEvent.requestPermission()` called inside the Start tap handler.
- **Wake Lock:** Request `navigator.wakeLock` on game start; release on game over; silently skip if unsupported. Re-acquire on `visibilitychange` if page returns to foreground mid-game.
- **Storage:** `localStorage` key for best score only. No other persistence, no backend, no analytics.
- **UI language:** Thai-primary for all player-facing text (title, buttons, messages). Score numbers plain digits.
- **Hosting:** GitHub Pages (free, HTTPS by default). Repo → Settings → Pages → deploy from main branch.

## 7. Out of Scope (v1) / Phase 2 Backlog

| Deferred item | Revisit when |
|---|---|
| Multiple Thai foods (different cook curves/visuals) | After v1 flip-feel is validated on real device |
| Food can slide out of pan from over-tilt (fail mode) | Phase 2 |
| Lives system instead of instant game-over | Phase 2 |
| Shared leaderboard (needs backend) | Only if game proves fun with friends; breaks zero-cost rule |
| Landscape orientation / wide-pan layout | Phase 2 polish |
| Sound effects / music | Phase 2 polish (v1 may include minimal flip feedback if trivial) |
| Bop-It-style command sequences, stir/shake mechanics | Phase 2 |

## 8. Decision Log

| # | Decision | Rationale |
|---|---|---|
| 1 | Fun game, not product | Learning + party use; validates motion APIs cheaply |
| 2 | Pan-flip cooking concept | Ham's idea; strongest gestures (tilt/flip) map naturally |
| 3 | One-handed, motion-only during play | Real-pan feel; touch only for start/restart |
| 4 | Flat-palm portrait hold | Closest to real pan; portrait simpler than landscape |
| 5 | Single food v1 (ไข่ดาว default) | Prove core loop before variety (Karpathy: no speculative features) |
| 6 | Two-sided cook state | Authentic flip motivation vs. arcade QTE |
| 7 | Burn = instant game over | Simplest fail state; real-cooking tension |
| 8 | Endless survival + ramping cook speed | One difficulty variable; natural high-score chase |
| 9 | Local high score only | Replay value at zero infra cost |
| 10 | Vanilla single file | Scope-appropriate; zero build step; free hosting |
| 11 | Config block for thresholds | Flip tuning is the key risk; fast iteration without touching logic |
| 12 | Start tap = iOS permission + start | Minimum friction; single allowed tap |
| 13 | Wake Lock API | Known failure mode (screen sleep in endless game), cheap fix, graceful fallback |
| 14 | Desktop fallback message | Avoid broken-looking UX on laptops |
| 15 | Score = successful flips | Clearer feedback than survival time; matches skill tested |
| 16 | Raw flip = no penalty, no score | Simplest rule; avoids extra fail state |
| 17 | Cook-speed ramp per successful flip | Difficulty tied to progress, not stalling time |
| 18 | Thai-primary UI text | Matches Thai theme |
| 19 | GitHub Pages hosting | Free, HTTPS by default (required for iOS motion permission) |
