# Hands-On Test — Pan Flip

Live URL: https://chanooooot.github.io/Flip-pan/

Test on a real phone over HTTPS. Desktop only needed for the fallback check (#12).

1. **Flip vs shake.** Real pan-toss motion → registers FLIP. Hard shaking → must NOT trigger a flip. Riskiest part — if off, tune `FLIP_ACCEL_THRESHOLD` / `FLIP_GYRO_THRESHOLD` / `FLIP_WINDOW_MS` in CONFIG (top of index.html).
2. **iOS permission flow.** Tap Start → permission prompt fires inside that tap. Deny it → retry message shows, can tap Start again.
3. **Doneness/color.** Watch food color go raw → golden → darkening → burnt over time.
4. **Golden-zone flip = score.** Flip during golden window → score +1, other side starts cooking, cook speed ramps slightly.
5. **Raw flip = no penalty.** Flip while still raw → sides switch, no score, no game-over.
6. **Burn = game over.** Leave one side alone → hits 1.0 → game-over screen, correct score shown.
7. **Best score persists.** Beat best → game-over screen shows new best → refresh page → best still there.
8. **Restart flow twice in a row**, no refresh needed between.
9. **Tilt cosmetics.** Tilt phone → egg slides toward tilt direction, smooth, no jitter, stays inside pan.
10. **Wake Lock.** Screen shouldn't sleep mid-game. Background the tab then return → still works.
11. **Debug overlay.** Triple-tap top-left corner → toggles overlay showing live accel/gyro + last event (FLIP / SHAKE rejected).
12. **Desktop check.** Open URL on laptop browser → fallback message, no console errors.

Priority: #1 first. Everything else is secondary until flip detection feels right on the actual phone.
