# itsglo-careers — CLAUDE.md

_Layers on top of ~/CoworkOS/itsGLO/Build/CLAUDE.md._

## Identity

The public job-application funnel at **careers.itsglocleaning.ca** — where new cleaner candidates apply. A single self-contained `index.html` (HTML + CSS + vanilla JS, no build step, no dependencies except the Bootstrap-Icons CDN). Deployed via **GitHub Pages** from `main` (repo `salimhourieh/itsglo-careers`, `CNAME` = careers.itsglocleaning.ca).

## ⚠️ Critical — do not break these

- **Push to `main` = live.** GitHub Pages auto-publishes (~1–2 min). There is no staging. Commit/push only when the user says so; confirm before pushing.
- **Portal wiring is load-bearing — never touch it without reason.** The page feeds the people-portal:
  - `?role=` → submit URL `/intake/website/{ROLE}/` (default `cleaner-toronto`); `?invite=` skips knockout steps; `?src=` channel attribution. See the `PORTAL INTEGRATION` block (`ROLE`/`INVITE`/`SRC`/`PORTAL_API_BASE`).
  - `submitForm()` POSTs the payload to the portal intake endpoint (`X-Intake-Key`) **and** a legacy Google Apps Script backup. Keep both.
  - `loadPortalCopy()` pulls admin-edited question copy from `…/{ROLE}/schema/` and overlays it onto the baked-in `T` dict. So **portal-stored copy overrides the baked strings** for any key the admin has edited (hero_title/apply_badge currently are NOT overridden — verify before assuming your copy will show).
- **The base64 team-photo line is enormous** — it breaks the Read tool. Use `awk 'length($0)<400'` / grep to inspect; never try to Read its line range.
- **Bilingual via the `T` dict (`en`/`es`).** Every visible string should have a `data-t` key with BOTH `T.en` and `T.es` entries. `applyTranslations()` runs on load and on language switch — if a key's dict value is stale, it will overwrite the baked HTML. Keep dict ↔ baked HTML in sync.

## How the flow works

- A full-page `#lang-gate` overlay shows first (before `step-0`) — pick a language to dismiss it (`.hide`). It sits ON TOP of everything, so for headless screenshots you must `document.getElementById('lang-gate').classList.add('hide')` or the shot just captures the gate.
- Landing = `step-0`. Browser language auto-detected on load (`navigator.language`); corner **EN|ES toggle** (`setLang`) switches in place; single **Apply Now** CTA (`startApplication`) → `goToStep(1)`.
- Once past the landing, `goToStep()` hides the marketing hero (`.hero.hide`) and shows a slim sticky logo bar (`#form-topbar`) — focus mode. Back is clamped to step 1, so the hero stays collapsed through the funnel.
- Steps (as of 2026-07-20): 0 landing · 1 job overview · **2–5 knockouts (auth+visaType / physical / transit / smartphone+data)** · 6 experience (follow-ups required if not "none") · 7 availability (5+ days **+ full-time 9–5:30 / part-time 9–2 schedule confirmation**, both 5 days/wk; "can't commit" → `sorry`) · 8 personal (name+phone / email / area / languages+gender) · 9 about-you · 10 FAQ+submit · `sorry`/`thanks`. `KNOCKOUT_STEPS=[2,3,4,5]` (invite-skip); `updateProgress()` divides by 10.
- **Palette** (2026-07-20 de-scam pass): solid sage buttons (`--teal-dark`), no gradient/glow; muted `--teal` `#2E9670`; amber `--gold` warning borders. Payload adds `scheduleCommit` (full-time|part-time), dropped `flexibility`.
- Logos live in-repo: `itsglo-logo.png` (dark text, for light/favicon), `itsglo-logo-white.png` (white text, for the dark hero).

## Workflow

1. Inspect with grep/awk (length-filtered), not full Reads.
2. Edit `index.html` locally; preview with `open index.html` or headless-Chrome screenshots at 1280px + 390px.
3. After any logic edit, sanity-check JS: extract the `<script>` block and `new Function(it)`.
4. Verify portal-wiring lines are untouched before committing.
5. Push only on the user's go.
