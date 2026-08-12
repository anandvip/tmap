# Thought Mapper — Drift Instrument

**Created:** 2026-08-12 (v3) · **Tool:** Claude Opus 5 (Anthropic) · **Type:** single-file HTML/CSS/JS, offline-capable, zero dependencies
**Lineage:** `anandvip/thought-map` (v1, 2024-06-30) → v2 → v3

---

## Purpose

A thought-observation instrument, not a notes app.

Every thought is stamped at capture with **who was speaking** — the persona holding the floor. The tool draws the switching pattern back at you, so you can see how erratically attention changes hands and which voice most often cuts another one off.

The 2024 original had the right idea and the wrong data model: it asked for a persona name and never wrote it onto the thought. Drift was structurally invisible.

---

## Undefined is a reading, not a blank

If you cannot tell who is speaking, log it anyway. It lands as **Undefined** — a first-class persona with its own treatment: hatched bands in the ribbon, dashed card border, its own count in the readout.

An unrecognised voice is signal interference, not missing data. Naming it later is the practice, not a cleanup chore.

---

## What it shows

**The drift ribbon.** One segment per thought. Width is dwell. Colour is the persona's ink. Hatching is Undefined. A bright left hairline marks a handover. A `⟋` break marks a session boundary. Segments that fade at the right edge have **open** dwell — the next thought is in another session, so the true hold time is unknown rather than enormous. Tap any segment to jump to its card.

**The readout.** Thoughts, switch rate, longest uninterrupted hold, typical within-session gap, who interrupts most, and how much is still unnamed.

> No score, no streak, no consistency rating. The moment observation becomes scorekeeping, another persona has taken the wheel.

---

## Voices

Personas are **objects** — name, description, ink — not bare strings.

- **Voices** (masthead) lists every voice with its description and count. Editing here propagates to every thought that voice stamps.
- **Editing from a card** defaults to **this thought only**, with an explicit *apply to all N* checkbox. A corrected misreading shouldn't silently erase the record of having been unsure.
- Descriptions appear in italic under the persona label on every card it stamps.

---

## Sessions

A silence longer than the **session gap** (default 2h, in Settings) starts a new session. The ribbon breaks visibly instead of drawing one enormous band across a night's sleep, and the exported review page groups entries by session with a note field for each.

---

## The review export

`Export review page` produces a **standalone HTML file** with the frozen flow, its ribbon, per-session notes, and a note field under every thought.

Observer and observed are separated by *time*, not by a text field. New in v3:

- **Reviewing as** — the observer has a persona too, and it stamps anything returned.
- **Return this to the stream** — a per-note checkbox that appears once you start writing. A promoted note re-enters the thought record on import as a real thought with a `parentId` back to what it was written about, notched in the ribbon and labelled *returned*. Unpromoted notes stay commentary and never touch the trace.

Notes persist in that page's own `localStorage`; `Export notes JSON` round-trips back through **Restore**.

---

## Using it

| Action | How |
|---|---|
| Set the speaking voice | Type in **Speaking** (draft until you log), or tap a chip |
| Log a thought | `Ctrl`/`⌘` + `Enter`, or **Log** |
| Edit a thought | Type in the card — saves on blur |
| Re-stamp one thought | Tap the persona label |
| Rename a voice everywhere | **Voices** → tap it |
| Name a voice upfront | `+ name a voice` chip |
| Filter | Tap a chip (tap again to clear) |
| Undo a delete | Toast holds **Undo** ~6s |

Storage: IndexedDB `ThoughtMapperDB` v3, stores `thoughts` / `personas` / `meta`. v1 and v2 databases migrate automatically on first open.

---

## Fixed in v3

**Ink leak.** v2 bound `input` on the persona field to `setPersona()` → `inkFor()`, which minted and persisted a persona on every keystroke. A real export contained **41 personas** — `b`, `bi`, `bik`, `bike`, `biker` … — one per character, and the 8-colour palette had cycled five times before the name was finished, so actual voices got arbitrary inks. Personas are now committed only at **log** time; the field is a draft until then. Orphans (zero thoughts) are pruned on boot, on delete, and on import — which cleans legacy exports automatically.

**Browser dialogs.** v2 used `prompt()` to rename a persona and had no confirm path. Native dialogs carry no swatch, no description, no scope choice, and can't express intent — `prompt()` returns a string. Every input now runs through one in-app **sheet** primitive (bottom sheet on mobile, centred panel on desktop) handling persona editing, the voices manager, settings, and confirmations. **Zero calls to `alert` / `confirm` / `prompt` anywhere in the file.**

**Export format.** v3 adds sessions, persona objects, reviewer persona, session notes, and the promote flag.

## Fixed earlier, in v2

| | |
|---|---|
| `db` undefined | `loadPersonaFromDB()` ran on the last line before `onsuccess` assigned the handle — `TypeError` on every load |
| `thought.persona` undefined | Never persisted per thought |
| HTML injection | User text interpolated into `innerHTML` inside a `<textarea>`; typing `</textarea>` shattered the card |
| Masonry leak | Re-instantiated per render, never destroyed. Removed — CSS multi-column replaces it |
| Write per keystroke | Full DB write per character; now committed on blur |
| Single-blob storage | `{ id: 1, ...persona }` rewrote the whole array every save, which is exactly what made per-thought attributes impossible |
| CDN dependencies | `normalize.css` and `masonry.pkgd` broke offline use. Zero external requests now |
| No export | JSON backup, JSON restore, HTML review export added |

---

## Design notes

Ground is deep ink-indigo rather than black — a plotter's paper at night. Channel colours are desaturated pen inks. Thought text is set in a serif and every measurement in monospace: the thought is human, the instrument is not.

## Deploy

Drop `index.html` at the repo root. Works from `file://`, GitHub Pages, or any static host. No build step.
