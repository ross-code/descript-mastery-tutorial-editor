---
name: descript-mastery-tutorial-editor
description: >-
  End-to-end editor for Descript Mastery tutorial videos. Descript (via its MCP)
  carries the transcript-based editorial backbone — transcribe, strip fillers,
  tighten, Studio Sound, captions, publish — and the existing Descript Mastery
  HyperFrames brand kit carries the branded graphics layer (intro, "Coming Up"
  roadmap, lower third, outro, subscribe, on-footage "liquid glass" overlays).
  Use whenever the user wants to edit, assemble, caption, add graphics to, or
  finish a Descript Mastery tutorial / lesson / course video, or render and
  composite the brand overlays for one. Brand-accurate: reads the real brand kit
  at ~/code/HyperFrames/Descript Mastery, not generic templates.
---

# Descript Mastery — Tutorial Editor

You edit Descript Mastery tutorial videos end to end. Two halves meet on the Descript
timeline:

- **Descript** owns the **editorial backbone** — transcription, filler removal, tightening,
  Studio Sound, captions, publish — driven through the Descript MCP agent in small,
  verifiable passes.
- **HyperFrames** owns the **graphics layer** — the branded intro, "Coming Up" roadmap,
  lower third, outro, subscribe cards, and on-footage "liquid glass" overlays — rendered
  from the **already-built** Descript Mastery brand kit and composited onto the footage.

> **Brand truth lives on disk, not in this file.** The kit is real and finished. Always read
> the actual sources before producing anything:
> - `~/code/HyperFrames/Descript Mastery/design.md` — the brand source of truth.
> - `~/code/HyperFrames/Descript Mastery/HANDOFF.md` — build status + every HyperFrames gotcha.
> - `~/code/HyperFrames/Descript Mastery/brand/` — the 16:9 (1920×1080) kit.
> - `~/code/HyperFrames/Descript Mastery/brand-vertical/` — the vertical (1080×1920) kit.
> - `~/code/HyperFrames/Descript Mastery/gradients-overlays/` — the canonical on-footage overlay episode to clone.
>
> If those paths aren't readable, stop and ask the user to grant access to the folder — do
> **not** invent brand values or rebuild templates from scratch.

---

## Audience & voice (ICP)

Descript Mastery teaches **first-time editors over 50** ("Novice Neal"). Everything on screen
and in narration is **confident, encouraging, direct — a patient mentor**. Plain words. One idea
per beat. Sentence case. Numerals for numbers. **No hype** ("revolutionary", "game-changing",
"10×", "unlock"), **no FOMO**, **no condescension** ("even your grandma"), **no emoji** anywhere.
Titles 3–7 words; sub-lines one short sentence. (Full voice spec in `design.md`.)

This shapes editing decisions too: hold graphics longer than feels necessary, keep one idea per
card, prefer full-frame cutaways over tiny corner overlays when explaining a step.

---

## Brand quick reference (verify against `design.md` before use)

**"Violet leads, green pays off."** The gradient lives in the logo mark only — every other
surface and all type stays flat. (Exception: the on-footage liquid-glass overlays.)

| Role | Hex | Use |
|---|---|---|
| Primary violet | `#2200D1` | Lead surface / brand color — ~70% of impact frames (intro/outro fields, title backings) |
| Payoff green | `#00E769` | The single high-energy payoff **once per scene** — roadmap number, one underline, the CTA, a success tick |
| Ink (text on light) | `#0A0612` | Body/reading text on paper |
| Paper (default content surface) | `#FAF9FC` | Off-white — never pure `#FFFFFF` for fields |
| On-dark text | `#FFFFFF` | Text/wordmark on violet or ink |

- **Green discipline:** if green appears more than once in a frame it stops being the payoff.
- **Fonts:** display = **Outfit** (built into the HyperFrames renderer, use `font-family:"Outfit"`,
  weights 500/600/700/800). Body = **Hanken Grotesk** (Google Fonts, 400/500/600/700) — the
  render-safe default this skill assumes. *Brand-exact alternative:* Switzer `.woff2` files are
  bundled in `fonts/` and `fonts/switzer.css` is ready — to go pixel-exact, swap the body family to
  Switzer; otherwise leave Hanken.
- **Logo:** `assets/dm-lockup-light.svg` (paper), `dm-lockup-dark.svg` (violet/ink), mono marks
  `dm-mark-{black,white,colour}.svg`. **Never recolor, re-stop, rotate, or skew the gradient mark.**
- **Geometry:** card radius 20px, pills 999px, overlay boxes a flat 10px. Safe area 96px (16:9) /
  64px sides + 220px top + 320px bottom (vertical).
- **Motion:** in `cubic-bezier(0.16,1,0.3,1)` 420ms · out `cubic-bezier(0.4,0,1,1)` 220ms · default
  hold 2000ms · per-word stagger 60ms · scene crossfade 300ms. Bounce only on a payoff.
- **Never** blend Descript Mastery with any other channel's or project's assets/colors/fonts.

---

## The workflow — three approval gates

Mirror the human-in-the-loop pattern from the Book-to-Film pipeline. **Stop and get the user's OK
at each gate** before spending the next block of effort.

```
   raw footage
        │   Descript editorial passes
        ▼
 ┌──────────────┐
 │  GATE 1      │  Rough cut approved (editorial done, before graphics)
 └──────────────┘
        │   plan the graphics beats against the transcript
        ▼
 ┌──────────────┐
 │  GATE 2      │  Graphics plan approved (which templates, what copy, what timecodes)
 └──────────────┘
        │   render + color-exact composite
        ▼
 ┌──────────────┐
 │  GATE 3      │  Composited cut approved (before publish/delivery)
 └──────────────┘
```

---

## Phase A — Descript editorial backbone (MCP)

Use the Descript MCP. Core tools: `import_media`, `prompt_project_agent` (the natural-language
editor), `wait_for_job`, `list_jobs`, `cancel_job`, `list_projects`, `get_project`.

**Known MCP behaviors — bake these in:**

1. **Create a project** by calling `import_media` with the media plus a `project_name`, and include
   `add_compositions` so the footage lands on the timeline. To create an empty project, omit media.
2. **Always `wait_for_job`** after any async call (`import_media`, `prompt_project_agent`) before the
   next step — don't fire-and-forget.
3. **Inspect before editing:** `list_projects` → `get_project` to read the real composition UUIDs.
   Pass those exact `composition_id`s to `prompt_project_agent`; never guess them.
4. **Re-authenticate** if the Drive is switched.
5. **Edit in small, verifiable passes** — one instruction per `prompt_project_agent` call, checking
   the result, rather than one giant prompt. This matches how Novice Neal's videos are reviewed and
   makes each change easy to undo.

**Editorial pass order (one `prompt_project_agent` call each, `wait_for_job` between):**

1. **Transcribe** the footage.
2. **Strip filler words** (um, uh, you know, like) — review, don't blindly accept.
3. **Tighten** — remove long pauses, false starts, and repeated takes. Keep the teaching cadence
   slow; do not over-cut. Novice Neal needs breathing room.
4. **Studio Sound** for clean audio.
5. **Captions** — generate from the transcript; style them to brand (sentence case, legible at small
   sizes, min 24px on-screen). Captions matter a lot for the 50+ ICP.
6. Leave **room for graphics** — note where the intro, roadmap topics, lower third, and outro land.

→ **GATE 1: rough cut approved.**

---

## Phase B — Plan the graphics beats

Get the transcript timecodes so graphics land *as the host says them*:

```bash
cd "~/code/HyperFrames/Descript Mastery/<episode>"
npx hyperframes transcribe source.mp4 --model small.en --language en
```

Map each branded beat to a template and a timecode, then write the copy in brand voice. The kit's
fillable variables (read each composition's `data-composition-variables` / `hfVars(...)` defaults to
confirm):

| Template | File (in `brand/compositions/`) | Variables |
|---|---|---|
| **Intro** | `intro.html` | `kicker` ("LESSON 01"), `title`, `subtitle` |
| **Coming Up** roadmap | `coming-up.html` | `title`, `topic1`…`topic5` (topics land transcript-synced; prior topics dim to 0.55) |
| **Lower third** | `lower-third.html` | `name`, `role`, `holdSeconds` |
| **Outro** | `outro.html` | `tagline`, `ctaText`, `siteUrl` |
| **Outro (webinar)** | `outro-webinar.html` | `eyebrow`, `headline`, `subscribeText`, `webinarText`, `descCta` |
| **Subscribe** (chroma/bar variants) | `subscribe-chroma*.html`, `subscribe-bar-chroma.html` | see each file; these are pre-built for Descript keying |
| **On-footage overlays** | `overlays/glass-overlays.css` + clone `gradients-overlays/` | liquid-glass rail / bottom band / lower-third |

Vertical episodes use the parallel files in `brand-vertical/compositions/` (same variable names,
re-flowed layout — not just scaled). Type goes up ~1.25–1.4×; the roadmap restacks from a row to a
vertical stack; the lower third rises above platform UI.

→ **GATE 2: graphics plan approved.**

---

## Phase C — Render the graphics (HyperFrames)

The kit pins **HyperFrames 0.6.97** via npm scripts (`dev`, `check`, `render`, `publish`). Keep the
global binary current too (`npm install -g hyperframes@latest`; needs ≥0.6.51 or rendered frames get
a ~100px bottom bar).

**Non-negotiable framework rules (full list in `HANDOFF.md` / the `/hyperframes` + `/gsap` skills):**

1. Every timed element needs `data-start`, `data-duration`, `data-track-index`, **and**
   `class="clip"` (visibility keys off it).
2. **`--variables` is silently ignored unless the root `data-composition-id` div carries an explicit
   `data-duration`.** Every composition in the kit already has it; any new one must too.
3. Customize per-episode with top-level `--variables` (verified working), e.g.:
   ```bash
   cd "~/code/HyperFrames/Descript Mastery/brand"
   npx hyperframes render -c compositions/intro.html \
     --variables '{"kicker":"LESSON 03","title":"Add captions in minutes","subtitle":"So every viewer can follow along."}'
   ```
   Per-instance `data-variable-values` inside a multi-template reel did **not** apply reliably in
   0.6.97 — prefer top-level `--variables`, or render each template on its own.
4. **Deterministic only** — no `Date.now()`, `Math.random()`, or network fetches; finite, seekable
   animations (looping "breathe" effects use a fixed `repeat:` count).
5. Base media must be named **`source.mp4`** (no spaces — `render` URL-encodes them and freezes on
   frame 1). The base `<video muted>` **and** a separate `<audio>` element both need `class="clip"`.
6. In sub-comps: never call `window.__hyperframes.getVariables()` (returns `{}`); use the scoped
   `hfVars()` helper the compositions already include. Use `#<composition-id>` selectors, not
   `[data-composition-id="…"]`.
7. After every edit: `npm run check` (lint + validate + inspect); fix all errors. `npm run dev` is
   long-running — launch in the background. The **validate contrast audit throws false positives**
   over footage / on hidden sub-comps — verify real legibility from rendered frames, not the warning.

**On-footage overlay episodes:** copy `gradients-overlays/` to a new sibling, drop in the new
`source.mp4`, re-time to the new transcript, swap the rail/lower-third/CTA copy, and keep the
`.glass-*` styles intact. Liquid-glass rules: violet→pink gradient + blur on `.glass-violet`, white
text on the violet majority, pink end ≤0.44 alpha away from type; green payoffs use green→mint glass;
10px radius everywhere; reserve left-centre for the presenter PiP; right rail left edge ≥~1460px;
bottom band screen-centred at `bottom:340px`; keep one thing always animating; only one green payoff
on screen at once.

---

## Phase D — Color-exact composite & deliver

`npm run render` color-shifts base footage ~±2/255 via Chrome compositing, so for over-footage comps
deliver via the **overlay-twin** path (the kit's color-exact render recipe):

1. Keep an `overlay.html` twin of the over-footage comp: same overlays/timeline, **no**
   `<video>`/`<audio>`, `body { background: transparent }`.
2. `npx hyperframes render -c overlay.html --format mov -o renders/overlay.mov` (ProRes 4444 + alpha).
3. **Blip gate-check** the overlay before compositing
   (`zsh blipscan.sh renders/overlay.mov alpha`, plus a crop-box scan of the panel
   rect — a full-width scan can read clean on a blipped narrow panel). If dirty, re-render; if it
   persists, add `--workers 1`.
4. Composite onto the original with ffmpeg (`-c:a copy` to keep audio bit-identical), then blip-scan
   the final.
5. **Pure intro/outro cards** (no base footage) skip the composite path — render straight to MP4.

**Descript import caveat:** Descript strips alpha on import. To key a graphic inside Descript,
deliver a flat **`#FF00FF` magenta** body with all box/drop shadows stripped and key via the Green
Screen effect (this is exactly what `subscribe-chroma*.html` are pre-built for), **or** an
Apple-native HEVC-alpha `.mov`.

→ **GATE 3: composited cut approved**, then publish (Descript) / hand off the final render.

---

## Guardrails recap

- Read `design.md` + `HANDOFF.md` before producing anything; never invent brand values.
- Violet leads; green is one payoff per scene; gradient = logo mark only (plus liquid-glass overlays).
- No hype / FOMO / condescension / emoji; sentence case; numerals; one idea per beat.
- `data-duration` on every root; `class="clip"` on video + audio; `source.mp4` exact filename;
  deterministic animations; `npm run check` after every edit.
- Small verifiable Descript passes; `wait_for_job` after every async call; real composition UUIDs.
- Never blend Descript Mastery with any other channel's or project's assets.
- Stop at all three gates.
