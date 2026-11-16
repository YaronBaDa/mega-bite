# CRUNCH — Handoff Document
*Last updated: 2026-11-16*

## Overview

**CRUNCH** is a single-file HTML/JS/CSS order-of-operations (BODMAS/PEMDAS) learning game for ages 10–18, formerly "Mega-Bite". Players reduce equations step-by-step — first picking which operation crunches first, then solving that chunk — to "clear" 5-equation notebook pages.

- **Live URL:** https://yaronbada.github.io/mega-bite/
- **Repo:** https://github.com/YaronBaDa/mega-bite (branch `main`)
- **Local source file:** `/root/dragons-equation-hoard/mega-bite.html` (~109 KB, self-contained)
- **Deploy clone:** `/root/mega-bite` (its `index.html` is the deployed copy)

## Current Status: COMPLETE

All requested work is done and pushed. The game has gone through 5 rounds of changes:

| Round | Commit | What |
|---|---|---|
| Rebrand | `fee99f4`, `545ac0d` | Mega-Bite → **CRUNCH**; Fraunces+Lexend fonts; BODMAS/PEMDAS toggle; teen copy; difficulty Warm-Up/Standard/Speedrun |
| Part 1 | `f2073d2` | Perforated notebook paper (punch holes, ruled lines), tear-away page flip, red-pen implode/evaporate chunk reduction |
| Part 2 | `ac45846` | Equation #5 Bonus Problem, LOCKED IN Overdrive mode, Daily Pop Quiz + clipboard share |
| Part 3 | `b494ef3` | 4-surface die-cut vinyl sticker ecosystem (8 stickers) |
| Part 4 | `abfa22a` | Mead Black Marble composition-book landing screen |

## Architecture (single-file)

**Core engine (AST-based):**
- `tokenize(expr)` → typed tokens (num/op/paren)
- `parseAST(tokens)` → AST via recursive descent (AS → DM → O → primary)
- `findNextReduction(ast)` → BODMAS-correct next binop to evaluate
- `findCandidateChunks(ast)` → tappable "leaf-level" binop nodes
- `findVisualDistractors(ast)` → fake adjacent num-op-num pairs (teaching traps)
- `replaceNode(root, target, result)` → swaps evaluated chunk for its numeric result
- `simplifyBrackets(ast)` → unwraps single-value brackets
- `getChunkValue/getChunkHTML/getChunkText`, `evalAST`, `astToString`, `countTotalSteps`

**Game flow:** `loadLevel()` → generate/parse → render → Phase 1 (select chunk) → Phase 2 (pick answer) → implode animation → AST update → repeat until single number → `onEquationSolved()` → page-complete (every 5th) → tear + next page.

**Key state (`let state = {...}`):**
```
difficulty, notation, pageNumber, equationsInPage, pageMistakes,
flowStreak, inkEnergy, ast, totalStepsOriginal, phase, selectedChunk,
correctAnswer, isProcessing, inkTimer, cheatSheetOpen, currentCandidates,
pageStartTime, mode ('endless'|'daily'), overdriveUntil, bonusCleared,
dailyDate, dailyDayNumber, dailyEquations, dailyResult,
stickers, bracketBreaks, powerBreaks, maxFlowStreak
```

## Design system

**Fonts:** `--font-brand: 'Fraunces'` (800) for the CRUNCH logo/title ONLY; `--font-ui: 'Lexend'` for everything else. `font-variant-numeric: tabular-nums` on equations/counters/stats.

**Palette (`:root`):**
```
--paper #FAF8F5   --paper-card #FFFFFF   --paper-tint #F1EFEA
--grid-blue rgba(59,130,246,0.14)        --margin-pink rgba(244,63,94,0.28)
--ink #1E293B      --ink-muted #475569   --border-ink #334155
--pen-blue #1D4ED8 --pen-pink #BE123C    --pen-red #E11D48   --pen-green #15803D
--overdrive #0284C7 --overdrive-cyan #38BDF8  --highlighter #FEF08A
```

**localStorage keys:**
- `crunch_v1` — endless progress (`pageNumber`, `equationsInPage`, `pageMistakes`) + `notation` + `difficulty`
- `crunch_muted` — SFX on/off (`'1'` = muted)
- `crunch_bonus` — `bonusCleared` count
- `crunch_daily_YYYY-MM-DD` — per-day Daily Pop Quiz result
- `crunch_stickers` — sticker unlock state + progress (`bracketBreaks`, `powerBreaks`, `maxFlowStreak`)

## Feature inventory

**Equation #5 "BONUS PROBLEM"** — 5th equation of every page (and daily quiz) shows a tilted red badge + uses `generateBonusEquation()` (≥3 precedence tiers or a left-to-right tie-breaker). Clearing it grants +25 Ink and `bonusCleared++` (drives the BONUS HUNTER sticker at 5).

**LOCKED IN Overdrive** — `flowStreak` counts consecutive clean reductions; at 3 → "LOCKED IN • 2×" (blue), at 5 → "LOCKED IN • OVERDRIVE 2×" (cyan `#0284C7`/`#38BDF8` tint on ink bar + operators, ink drain paused 8s via `overdriveUntil`, upward harmonic chime). Any mistake resets to 0.

**Daily Pop Quiz** — date-seeded PRNG (`seededRandom('crunch-daily-' + dateStr)` using mulberry32 + FNV-1a) generates the same 5 equations for everyone each day; eq 5 = bonus. One ranked run/day stored in `crunch_daily_*`; revisit shows completed badge/time. Share button copies a spoiler-free summary and flips to "Copied! ✓".

**Sticker ecosystem (8 stickers)** — first_crunch, flawless_ink, speed_demon, bracket_breaker, power_surge, locked_in, bonus_hunter, page_10_club. Die-cut vinyl look (3px white outline, drop shadow, Lexend SVG badge art, gloss sheen). 4 surfaces: draggable cover (`#stickerCanvas`), mid-game left-margin toast (`#stickerToast`), slap-on-page-clear (`#slapZone`), sticker-sheet drawer (`#stickerSheet`, `Stickers X/8` pills `#stickerPill`/`#stickerPill2`). Stickers persist `{unlocked, unlockedAt, x, y, rotation}`.

**Landing screen (Part 4)** — Mead Black Marble composition book, *landing screen only*: `#cover-bg` (`position: fixed; inset: 0; z-index: -1`) with a 11.5%-wide black spine + 88.5% marbled area via inline SVG `<feTurbulence type="fractalNoise" baseFrequency="0.035" numOctaves="4">` + `<feComponentTransfer>` discrete threshold. Title label box (`#title-label`) centered at 55.75vw, top 11.5vh, 55vw wide, 2.5:1, double black border (18px/22px) with 3 ruled lines; logo+subtitle live inside it. Mascot removed from landing (kept on celebration cards).

## Deployment & verification

1. Edit `/root/dragons-equation-hoard/mega-bite.html`.
2. Copy → `/root/mega-bite/index.html`, commit + push `main`:
   ```bash
   cd /root/mega-bite && cp /root/dragons-equation-hoard/mega-bite.html index.html && git add index.html && git -c user.email="yaron@example.com" -c user.name="YaronBaDa" commit -m "..." && git push origin main
   ```
3. GitHub Pages CDN lags ~10 min. Verify without cache via the **GitHub Contents API** (authoritative blob):
   ```bash
   curl -s "https://api.github.com/repos/YaronBaDa/mega-bite/contents/index.html" | python3 -c "import sys,json; print(json.load(sys.stdin)['size'])"
   ```
4. JS syntax check: extract the `<script>` body and `node --check` it.

## Tooling quirks (learned this session)

- `browser_exec` runs on a **separate filesystem** from the docker sandbox — it cannot read docker `/root` ("permission denied"), and its `document.title` shows a runtime-injected emoji prefix (e.g. `🐴`) not present in the real file. Trust `curl`/GitHub API over the browser's title.
- At session end the browser-automation backend went down ("CDP WS handshake failed" / "sandbox did not answer"). Live browser verification of the marble was **not** completed.
- `node --check` + grep for forbidden terms are the reliable static verification path when the browser is unavailable.

## Known caveats / next steps

1. **Marble density unverified visually.** The filter uses `tableValues="0 0 0 1"` (threshold at 2/3) targeting ~68% black / ~32% white. If the white marbling reads too sparse or dense in the browser, adjust the `tableValues` threshold (e.g. `"0 0 0 1 1"` for more white) — it's the single knob.
2. Re-verify the landing screen at 390×844 and 1024×768 (label/button geometry, marble render) once the browser tool is back.
3. Everything else (Parts 1–3) was browser-verified before the outage.
