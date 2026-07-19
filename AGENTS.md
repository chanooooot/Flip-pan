# AGENTS.md — Pan-Flip Cooking Game

Instructions for any coding agent (Codex or other) building this project. **SPEC.md** in this folder is the source of truth for what to build; this file defines how to work. Read SPEC.md fully before writing code.

## Ground Rules

1. **Single file.** Deliver one `index.html` containing all HTML/CSS/JS. No frameworks, no build step, no external dependencies (no CDN libraries).
2. **Simplicity first.** Minimum code that satisfies SPEC.md. No features beyond spec. No abstractions for single-use code.
3. **Config discipline.** Every tunable number lives in the `CONFIG` object at the top of the file (SPEC §5). Game logic reads from CONFIG only — no magic numbers in logic.
4. **Surface assumptions.** All game-design decisions are locked in SPEC.md (see Decision Log). If you must make any implementation choice not covered by the spec, record it in a `## Build Notes` comment block at the top of index.html.
5. **Motion code is only testable on a real phone** over HTTPS. Include a hidden debug overlay (triple-tap top-left corner) showing live accel/gyro values and last detected gesture.

## Build Order (risk-first)

Riskiest unknown: **flip detection** (pan-toss vs. shake). Validate before building on top.

1. Flip detector in isolation + debug overlay → verify on real phone: pan-toss triggers FLIP, shaking is rejected.
2. Static canvas render: pan + fried egg, two-sided doneness color model.
3. Cook loop: doneness tick, golden zone, burn → game over.
4. Wire flip → cook: side-switching per SPEC §4, score, cook-speed ramp.
5. Tilt cosmetics: food slides with tilt (visual only), smoothed orientation input.
6. Shell: start screen, iOS permission inside Start tap, game-over screen, localStorage best score, restart.
7. Platform hardening: Wake Lock (acquire/release/re-acquire on visibilitychange), desktop "mobile only" fallback.

## Verification Checklist

- [ ] Runs from a single HTML file over HTTPS
- [ ] iOS motion permission requested inside Start tap; denial shows retry message
- [ ] Hard shaking does NOT register as a flip
- [ ] Burn ends game; score/best score correct; best persists after refresh
- [ ] Desktop shows fallback message, no console errors
- [ ] All tunables in CONFIG; Build Notes records builder-choice decisions

## Prohibited

- Backend, analytics, external assets/libraries
- Phase-2 features listed in SPEC §7
- Touch controls during gameplay (tap only on start/restart screens)
- Silently deviating from SPEC.md — ask the owner if something is ambiguous beyond the flagged builder-choices
