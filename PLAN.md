# PomoPal — Canonical Implementation Plan (v2.1 — APPROVED)

**Product owner:** Fable 5 Medium (planning, review). **Executor:** Sonnet 5 Ultracode (implementation).
**Status:** Approved by the user 2026-07-17 with amendments A1–A6 (incorporated below). Pass 1 authorized. Each later pass begins only after Fable review of the previous pass's evidence.
Sonnet must not edit this file; Fable records status and deviations here.

---

## §0 Live-product audit summary (evidence basis)

Deployed build (= repo `aa49d58`) measured at 320×568, 375×667, 390×844, 768×1024, 1280×720 with seeded realistic data. Key findings driving decisions below:

- **L1** Real bug: `main`'s transform entrance animation makes it the containing block for `position:fixed` `#setDock` → dock renders mid-page ~0.65s on every load, then snaps to bottom.
- **L2** `#setDock` lives inside `#tab-timer` — settings/ambience/music unreachable from other tabs while music keeps playing.
- **L3** 320×568: dock handle overlaps the Start button by ~26px.
- **L5** Task rows balloon: 72px @768 → 164px @390 → 300px @320 (wrapped note).
- **L6** Touch targets: checkbox 22×22 (all widths); goal target-edit span 16×17; Working-on pill 29px tall; chips 35px.
- **L7** Coarse-pointer tablets >520px get only the `.mv` bump; steppers 30px, checkbox 22px.
- **L8** Goals form defaults to **Count** — the only type that never auto-ticks. Default path silently never progresses.
- **L9** "Planner" vs "Goals" labels are both prospective-sounding; placement of e.g. "read 10 chapters" is ambiguous.
- **L10** Passing: no horizontal overflow anywhere 320–1280; subject modal fits at 320; 16px form fonts (no iOS zoom).
- **L11** Every list rebuild replays the `rise` entrance on all rows (checking one task re-animates the whole list).
- **L13** Native task `<select>` sits inside the ring face; phase icons measure adequate → **ring swipe rejected** (§8).

## §1 Locked product decisions

| # | Decision |
|---|----------|
| D1 | **IA: Timer / Plan / History.** Inside Plan: **Today** and **Targets** subsections behind a segmented control (shared pattern with the settings dock's sub-tabs). History keeps its name. Mental model: Targets = direction, Today = current commitment, Timer = the work, History = the record. |
| D2 | **Primary target types are auto-derived:** **Pomodoros** (count of auto-logged focus completions `h.auto === true` with `h.at ≥ createdAt`, optional subject scope) and **Minutes** (existing `unit:"min"` derivation). Both support optional subject + deadline. **Form default = Pomodoros.** |
| D3 | **Legacy manual count goals preserved as "Milestone (manual)"** — data, ticks, and progress untouched; rows get a `· milestone` meta tag; creation stays available as the last, non-default type option. No silent deletion or reinterpretation. |
| D4 | **Labels-only rename; storage unchanged.** Keys `study.plan`/`study.goals`/`study.tab`, `data-tab="plan"/"hist"`, section ids, function names all stay. New key: `study.planSub`. |
| D5 | **Runtime tab fallback (permanent):** stored `"goals"` → show `plan` + set `planSub="targets"` (persisted); any unmatched tab → `"timer"`. `study.planSub` validated everywhere it is read: `today`\|`targets`, unknown → `today`. **A5:** `study.planSub` is added to `DATA_KEYS` (exported/imported; imported values validated on restore). |
| D6 | **Swipe right on a Today row = complete.** Incomplete rows only; done rows give no right-swipe response. Un-completing stays on the checkbox. |
| D7 | **Swipe left = reveal actions, never destroy.** Tray shows **Edit note** and **Delete** (≥64px each). **A4:** the label is exactly "Edit note" (it opens the existing inline note editor); no full task-edit modal in this effort. Delete routes through the existing delete+undo. One tray open at a time; outside tap / scroll / right-swipe closes it. |
| D8 | Gesture engineering: pointer events, `pointerType !== "mouse"`, 10px axis lock, `setPointerCapture`, `touch-action: pan-y`, capture-phase click suppression after movement, delegated handlers on `#planList`, per-task-id state, mid-gesture re-render cancels, excluded starts on `input`/`.mv`/`.x`/active-contentEditable. Every gesture keeps a visible button equivalent. |
| D9 | **Haptics: semantic `haptic(kind)` only** (§4). Toggle rendered only when `'vibrate' in navigator`; independent of Sound; default ON. |
| D10 | **Ring phase-swipe: rejected** (§8). Pass 4 is timer motion only. |
| D11 | **Dock:** `#setDock` moves out of `<main>` to document level (fixes L1) and becomes global (fixes L2, deliberate). **A3:** Timer, Plan, and History all get bottom clearance ≥ handle height + `env(safe-area-inset-bottom)` so the handle never covers final content; **the dock closes automatically on top-level tab navigation** (open state does not persist across tabs). |
| D12 | **List motion (A2, corrected):** existing rows render immediately, always — including on initial page load and after backup import. Only the row created by the **current user action** gets the entrance animation, driven by explicit one-shot creation state (`freshId.plan/goal/hist`), set **only** in the genuine creation handlers (`planForm`/`goalForm`/`histForm` submit). Freshness is never inferred from whether an ID has rendered before. The next render applies `.fresh` to the matching row only, clears the one-shot ID on consumption, and removes the class on `animationend` with a 400ms timeout backstop (reduced-motion / hidden-container safe). Auto-logged history rows and undo-restored rows are NOT marked fresh (Fable decision: they are not the current user's creation gesture on that list). Task-completion visual state uses the existing `done` class and never `.fresh`. Returning to Plan, switching Today/Targets, and post-completion rebuilds replay nothing. Sub-tab switching is instant in Pass 1; a restrained container fade (never per-item replay) may be added in Pass 5. |
| D13 | Unchanged: motion tokens (§3), no new dependencies, no visual redesign, old-backup compatibility, one pass per commit, Fable review after every pass. |

## §2 Interaction specifications

**2.1 Today rows.** Base per D8. Right-swipe: mint check beneath, follow-finger; commit at 80px or 48px with release velocity >0.5 px/ms; **haptic: one `success` fired at the moment the threshold is first crossed, latched — nothing on release or commit (A1)**; sub-threshold release snaps back over `var(--dur-reveal)`. Left-swipe: tray with **Edit note** | **Delete**; snaps open at tray width; **haptic: one `selection` when the tray locks open (A1)**; tray buttons are ordinary focusable buttons. No interaction may stack `selection` + `success` (A1); the utility's coalescing window is the backstop.

**2.2 Row layout (A6).** Normal rows compact; long notes clamp to **two lines at rest** (full text reachable via Edit note); no controls hidden; no horizontal overflow at 320px; visible touch controls comfortably tappable. (Implemented in Pass 3.)

**2.3 Targets form (A6).** No fixed pixel-height target. Criteria: no horizontal overflow; logical field grouping; readable labels; primary controls ≥44px tall where practical; no unnecessary single-field rows; the form must not visually dominate the initial mobile viewport; no information or functionality removed merely to reduce height. (Implemented in Pass 5.)

**2.4 Not in scope:** ring swipe, long-press reorder, swipes on Targets/History rows, tab-swipe, pull-to-refresh, task-edit modal.

## §3 Motion tokens & motion-quality fixes

Tokens: `--dur-fast .15s` (absorbs .15/.18/.2), `--dur-reveal .25s` (absorbs .25/.28/.3), `--dur-slide .35s`, `--dur-entrance .45s`, `--dur-celebrate .8s` (absorbs .8/.9), `--ease-spring cubic-bezier(.3,1.4,.4,1)` (absorbs the .3,1.3 variant). Deliberate literals: `breathe 4s ease-in-out`, ring `linear`. `rise` unifies on `ease-out`. Cheer's `setTimeout(900)` → `animationend` + 1s reduced-motion safety fallback. Plus: D11 dock relocation (Pass 2) and D12 fresh-row mechanism (Pass 1).

## §4 Haptic specification

```js
const HAPTIC = { selection: 10, success: 20, sessionComplete: [30, 50, 30] };
const PRIORITY = { selection: 0, success: 1, sessionComplete: 2 };
let lastHaptic = { t: 0, p: -1 };
function haptic(kind) {
  try {
    if (!settings.haptics || !("vibrate" in navigator)) return;
    if (document.visibilityState !== "visible") return;
    const now = performance.now();
    if (now - lastHaptic.t < 250 && PRIORITY[kind] <= lastHaptic.p) return;
    lastHaptic = { t: now, p: PRIORITY[kind] };
    navigator.vibrate(HAPTIC[kind]);
  } catch {}
}
```

**Complete site list (A1-final):** right-swipe completion threshold first crossed → `success` (once per gesture, latched); left-swipe tray locks open → `selection`; checkbox completion (checking only) → `success`; completed **live** focus session → `sessionComplete`. **Non-sites:** release/commit after a threshold haptic, start, pause, undo, phase switching, automatic task/target completion, target ticks, reorder buttons, navigation, seek. Settings row absent when `'vibrate' in navigator` is false.

## §5 Accessibility & reduced-motion

Targets (mostly Pass 7): checkbox, goal target-edit, Working-on pill hit areas ≥40px (visuals may stay compact — pseudo-element halos / padded invisible controls); tour dots ≥24px; tap-target bumps duplicated into `(pointer:coarse)` (L7). Plan sub-tabs: `role="tablist"`/`aria-selected` mirroring the dock pattern; subsections keyboard-reachable. Reduced-motion blanket kill stays; swipe transforms are direct style writes; tray reveal instant under reduced motion; haptics independent of reduced-motion.

## §6 Data migration

No key renames or data rewrites. `unit:"poms"` added (new rows only); `goalProgress` extended; legacy `count` rows untouched. `study.planSub` added + validated + in `DATA_KEYS` (A5). `study.tab` runtime map (D5). `settings.haptics` joins defaults (Pass 2). Old 4-tab backups: import → reload → `"goals"` lands on Plan/Targets.

## §7 Passes

**Evidence rules:** §9 statuses move to "in progress" only when Sonnet 5 Ultracode actually begins the pass. Where an acceptance item requires a physical touch or vibration-capable device, Sonnet must not claim success without one — it provides emulated/instrumented evidence, clearly labeled, and records physical-device verification as **pending**.

### Pass 1 — Plan IA, Targets model, compatibility migration ← **AUTHORIZED**

**Files:** `index.html`, `README.md` only. PLAN.md is Fable-owned.

**Scope (complete; nothing else):**
1. Nav: 3 buttons — Timer / **Plan** / History (`data-tab` values `timer`/`plan`/`hist` unchanged); Goals button removed; `#navPill` width → the 1/3 pattern already used by `.setPill`.
2. Plan section: segmented control (Today | Targets) above the content, reusing the dock sub-tab pattern. If the existing styles are id-bound to `#setTabs`, generalize to a shared class used by both containers (each pill positioned by a custom property set on its own container); the settings dock's behavior must remain pixel-identical. Subsection containers: Today = existing planner form+list block (h2 "Today's plan" and hint unchanged); Targets = the goals hint/form/list block moved in from `#tab-goals` (which is removed). `showPlanSub(name, persist=true)` mirrors `showSetTab`: aria-selected, pill var, hidden toggling, `save("study.planSub", ...)` when `persist`.
3. Targets vocabulary: h2 "Your **targets**"; hint: "Bigger than today. Pomodoro and minute targets fill up on their own as you focus; milestones tick by hand."; label "Goal"→"Target"; text placeholder → "A steady study week"; empty state `emptyState("No targets yet — aim at something.")` (mascot OK — only one list visible per view now); delete toast "Target deleted".
4. Targets model: `#goalUnit` options in order — `poms` "Pomodoros" (first = default), `min` "Minutes", `count` "Milestone (manual)". `goalProgress` for `poms`: `hist.filter(h => h.auto && h.at >= (g.createdAt||0) && (!g.subjectId || h.subjectId === g.subjectId)).length`. Subject select (`#goalSubjF`) visible when unit ≠ `count`; submit stores `subjectId` for `poms` and `min`. Meta for poms: `cur + " / " + target + " poms"`; poms/min never show "+N over" (cap bar at 100%, "· done ✦" as today); count keeps existing over-display; count rows additionally get `· milestone` in meta. Target default value 10 stays.
5. Tab fallback (D5/A5): in `showTab`, raw `"goals"` → set `planSub="targets"` (persisted) and show `plan`; unmatched → `"timer"`. `study.planSub` read via validated load (`today`|`targets` else `today`), written by `showPlanSub`, added to `DATA_KEYS`.
6. Fresh-row mechanism (D12, corrected — applies to `#planList`, `#goalList`, `#histList`): explicit one-shot creation state `freshId = { plan, goal, hist }`, set only in the three creation submit handlers immediately before their render call. During render, the row whose id matches consumes the one-shot (gets `.fresh`, id cleared); `.fresh` is removed on `animationend` (delegated per list) with a 400ms timeout backstop. Persisted rows do not animate on initial load; imported rows do not animate; auto-logged and undo-restored rows do not animate; tab return / sub-switch / rebuild replay nothing.
7. Tour: step 4 `{tab:"plan", sub:"today", sel:"#planForm", h:"Today", ...}`; step 5 `{tab:"plan", sub:"targets", sel:"#goalForm", h:"Targets", p:"Bigger than today. Pomodoro and minute targets fill up on their own; milestones tick by hand."}`; `tourShow` honors an optional `sub` via `showPlanSub(sub, false)` (**no persistence from the tour**); `tourEnd` restores `showPlanSub(load-validated planSub, false)` alongside the saved tab. Step copy for the old planner/goals steps rewritten to Today/Targets vocabulary; History step unchanged.
8. README: Planner section → "Plan" describing Today + Targets subsections; Goals section content folds in (auto pomodoro/minute targets, milestones tick by hand); "subjects shared across planner, log, and goals" → updated wording; nav description updated. History wording untouched.
9. Strings sweep: every "Planner"/"plan"/"Goals"/"goal" user-visible string from the rename-surface audit addressed with Plan/Today/Targets vocabulary. Internal identifiers untouched (D4).

**Explicitly out of scope for Pass 1:** dock move/clearance/auto-close (Pass 2), motion tokens (Pass 2), `haptic()` (Pass 2), swipes (Pass 3), note clamping (Pass 3), form layout work (Pass 5), any haptic call site, ring anything.

**Edge cases that must work:** `timerTaskId`/`renderTaskSel` coupling unchanged; goals created pre-change (min + count) render and compute correctly; `study.tab === "plan"` users land on Today (default sub); import of a pre-restructure backup (incl. `study.tab:"goals"`, no `study.planSub`) lands on Plan/Targets after reload; re-running the tour from `?` doesn't corrupt the persisted sub-state; `console.assert` self-checks still pass.

**Acceptance criteria:**
- A. Three tabs; pill 1/3 width and correct position on each; no dead nav states.
- B. Persisted rows never animate on initial load; imported rows never animate as newly created; Today|Targets switch (instant, persisted) and returning to Plan replay zero entrances; rebuilding a list after a completion replays zero.
- C. Only the row created by the current user action (task add / target add / manual log) receives the entrance, exactly once; auto-logged and undo-restored rows do not; task-completion state uses the existing `done` class and never `.fresh`.
- D. Stored `"goals"` (boot, import-reload, tour-end) lands Plan/Targets; unknown tab → Timer; `plan`/`hist`/`timer` unaffected.
- E. `study.planSub` exports/imports, validates (garbage → `today`).
- F. New Pomodoros target auto-progresses after a live focus completion; Minutes target unchanged; legacy count row keeps value, ticks, `· milestone` tag; subject select hidden only for Milestone.
- G. Full tour runs, Targets step lands correctly, sub-state not polluted.
- H. Old-format backup imports cleanly.
- I. README accurate.
- J. Zero console errors; zero horizontal overflow at 320/375/390/768/1280; timer/music/history smoke-pass.

### Pass 2 — Shared motion, interaction, haptic foundations
Tokens + call-site migration; dock relocation out of `<main>`, global availability, **auto-close on top-level tab switch, bottom clearance on all three tabs (A3)**; `haptic()` + conditional Vibration toggle (no call sites); cheer `animationend`; gesture CSS groundwork. *Accepted when:* no dock jump on load; dock reachable from all tabs and closes on tab switch; Start clear of handle at 320×568; final content on every tab clears the handle; tokens verified via computed styles; toggle absent without vibrate support; visuals otherwise unchanged.

### Pass 3 — Today task touch & gesture polish
§2.1 swipes with A1 haptic timing hooks left as no-ops until Pass 6 (structure the commit/lock points so Pass 6 only adds calls); §2.2 row layout incl. two-line note clamp; row-resident touch-target halos. *Accepted when:* all §2.1 behaviors verified with touch emulation + one real coarse-pointer device; done rows ignore right-swipe; tray never deletes on release; "Edit note" opens the inline editor; vertical scroll never hijacked; desktop drag-reorder untouched; 320px row measurably more compact with nothing removed.

### Pass 4 — Timer motion
Cheer/celebrate token migration verification, load-jump regression check, no gesture code on the ring. *Accepted when:* cheer clears via `animationend` (and fallback under reduced motion); no dock jump; ring untouched by pointer handlers.

### Pass 5 — Targets inside Plan, polish
§2.3 form criteria; hint/empty-state final copy; optional restrained subsection fade (container-level only, D12); fresh-completion pulse intact. *Accepted when:* §2.3 criteria all verifiably met at the five audit widths; pulse fires once; both sub-tabs keyboard-reachable.

### Pass 6 — Restrained haptic call sites
Exactly the §4 site list. *Accepted when:* each fires on a vibrate-capable device; no stacked selection+success from any single interaction (A1); coalescing verified; OFF silences all; muted-audio users keep haptics; nothing fires from restore/auto paths.

### Pass 7 — Accessibility & reduced-motion hardening
§5 items; keyboard walkthrough; reduced-motion sweep over Passes 1–6. *Accepted when:* all targets measure (≥40px, dots ≥24px); coarse-pointer 768 emulation gets bumps; keyboard parity with every gesture; zero decorative motion under reduced motion.

### Pass 8 — Cross-browser & migration QA
Five viewports light+dark; old-backup import; complete tour; timer regression set (reload-mid-session attribution, undo queue, duration clamp); music smoke; iOS sanity; README final. *Accepted when:* all pass with recorded evidence and Fable signs off.

## §8 Deviations & audit-driven decisions log (append-only)

- Ring phase-swipe **rejected** for this effort (L6 icons adequate, L13 select conflict, zero discoverability). Revisit only with real-device evidence.
- Dock globalization is a deliberate product change (L1 bug fix + L2), with A3 auto-close-on-navigation.
- Milestone creation retained but demoted to last, non-default option (L8, plausible non-activity job).
- A1–A6 amendments incorporated 2026-07-17 (haptic de-duplication; new-row-only entrances; dock clearance + auto-close; "Edit note" label; `study.planSub` in DATA_KEYS with validation; qualitative layout criteria replacing pixel-height targets).
- **Pass 1 corrections (Fable-decided during review):** (1) `trackFresh` gained a 400ms `setTimeout` backstop removing `.fresh` — `animationend` cannot fire on rows hidden mid-animation or under reduced motion, and a stale `.fresh` would replay the entrance on re-show (verifier-caught race against acceptance B). (2) The Targets form's numeric field relabeled "Target" → **"Amount"** — the plan's literal string mapping had produced two adjacent "Target" labels (implementer-flagged).
- **Pass 1 verification notes:** the boot-time re-save of migrated arrays (pre-existing, idempotent) is accepted as-is; ARIA tablist roles on the Plan segmented control are deferred to Pass 7 as planned; empty-state rows no longer animate (consistent with D12 — only new rows animate).
- **D12 contradiction correction (user-directed, blocking):** the initially shipped seen-ID freshness inference animated ALL rows on first paint (persisted and imported rows included), violating D12's "existing rows always render immediately." Replaced with explicit one-shot creation state (`freshId.plan/goal/hist`) set only in the three creation submit handlers; auto-logged and undo-restored rows deliberately not marked (Fable decision). Acceptance B/C rewritten to match. Metadata corrections applied in the same revision: planning model named "Fable 5 Medium"; §9 status semantics ("in progress" only when Sonnet actually begins); physical-device evidence rule added to §7.

## §9 Completion status

| Pass | Status |
|---|---|
| 1 — Plan IA, Targets model, migration | **complete** — implemented by Sonnet (`de0fd8d`), then the user-flagged D12 contradiction corrected by Sonnet (explicit one-shot `freshId`, this commit); verified by independent reviewers + Fable live review both times; acceptance A–J pass under the corrected B/C. Physical-device verification pending (per §7 evidence rules): visual confirmation of the entrance animation on a real painting foreground tab, and reduced-motion behavior on a real device. |
| 2 — Foundations | not started |
| 3 — Today swipes | not started |
| 4 — Timer motion | not started |
| 5 — Targets polish | not started |
| 6 — Haptics | not started |
| 7 — A11y hardening | not started |
| 8 — QA & migration | not started |
