# CLAUDE.md — Pan-Flip Cooking Game

You are building a motion-controlled mini webapp from **SPEC.md** in this folder. Read SPEC.md fully before writing any code. SPEC.md is the source of truth; this file tells you how to work.

## Ground Rules

1. **Single file.** Deliver one `index.html` containing all HTML/CSS/JS. No frameworks, no build step, no external dependencies (no CDN libraries).
2. **Simplicity first.** Minimum code that satisfies SPEC.md. No features beyond spec. No abstractions for single-use code. If a function could be 10 lines, don't write 40.
3. **Config discipline.** Every tunable number lives in the `CONFIG` object at the top of the file (see SPEC §5). Game logic reads from CONFIG only — no magic numbers in logic.
4. **Surface assumptions.** All game-design decisions are locked in SPEC.md (see Decision Log). If you must make any implementation choice not covered by the spec, record it in a short `## Build Notes` comment block at the top of index.html.
5. **The owner cannot debug motion code on desktop.** Everything motion-related must be testable on a real phone via a hosted HTTPS URL. Include a hidden debug overlay (see Build Order step 1) toggled by triple-tapping the top-left corner — shows live accel/gyro values and last detected gesture.

## Build Order (risk-first)

The riskiest unknown is **flip detection** (distinguishing a pan-toss from a shake). Validate it before building anything on top.

1. **Flip detector in isolation** — page that shows live sensor values + logs "FLIP" / "SHAKE (rejected)" events with the debug overlay. → verify: on a real phone, a pan-toss motion triggers FLIP; vigorous shaking does NOT.
2. **Static render** — canvas pan + fried egg (ไข่ดาว), two-sided doneness color model (raw → golden → darkening → burnt). → verify: doneness value visibly maps to color.
3. **Cook loop** — doneness ticking on face-down side, golden zone, burn → game over state. → verify: leaving it alone burns and ends the game at the expected time.
4. **Wire flip → cook** — flip gesture switches sides per SPEC §4 rules, score increments, cook-speed ramp. → verify: full round playable end-to-end.
5. **Tilt cosmetics** — food slides with device tilt (visual only, cannot fall out). → verify: smooth, no jitter (apply light smoothing to orientation values).
6. **Shell** — start screen, iOS permission-in-start-tap, game-over screen, localStorage best score, restart. → verify: full flow twice in a row without refresh.
7. **Platform hardening** — Wake Lock (acquire/release/re-acquire on visibilitychange), desktop "mobile only" fallback. → verify: feature-detection paths don't throw on desktop.

Do not skip step 1's real-device verification gate. If the owner reports flip detection feels wrong, tuning happens in CONFIG only.

## Verification Checklist (before handover back)

- [ ] Opens and runs from a single HTML file over HTTPS
- [ ] iOS: motion permission requested inside Start tap; denial shows retry message
- [ ] Shaking hard does not register as a flip
- [ ] Burn ends game; score and best score correct; best persists after refresh
- [ ] Desktop shows fallback message, no console errors
- [ ] All tunables in CONFIG; Build Notes comment records builder-choice decisions

## What NOT to Do

- No backend, no analytics, no external assets/fonts/libraries
- No phase-2 features from SPEC §7, even if easy
- No touch controls during gameplay (tap only on start/restart screens)
- Don't "improve" the spec — if something seems wrong or ambiguous beyond the flagged builder-choices, stop and ask the owner
