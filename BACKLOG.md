# Backlog

Work deferred on purpose, with enough context to pick each one up cold.
None of it is a bug. Started 16 Sep 2026.

---

## 1. Make it usable by other people

**Status:** scoped 16 Sep 2026, not started. Roughly a day.

Several people can already use the app in parallel today with no work at all:
it is a static page, every device keeps its own `localStorage`, and there is
no server or account to collide over. Two people sharing one phone is the only
case that breaks, since there is a single storage key and no profiles.

What stops it being pleasant for someone who is not me:

1. **Session rename, add and remove.** The big piece. `SESSIONS` is a `const`
   of five and every exercise points at one by `s: "lowerA"`. Needs the same
   overlay pattern already used for exercises: `state.plan.sessions` holding
   renames, an off map and an order, plus a `rebuildSessions()` mirroring
   `rebuildProgramme()`. The ripple is the cost, not the overlay: the "what's
   next" logic works by position in a rotation that could now change length,
   and removing a session with logged history needs the bench-don't-delete
   treatment or the history orphans. Decision still open, and the simpler
   answer is probably yes: make every removal a bench, never a delete.
2. **First-run setup.** Detect a genuinely fresh device (no log, no bodyweight,
   no loads) and ask three questions: sex, bodyweight, dumbbell convention.
   Needs a `settings.setupDone` flag, and a "restore a backup instead" escape
   on the same screen so returning to a new phone does not mean being marched
   through setup before your own file can be loaded.
3. **Start from an empty programme.** Nearly free, since `plan.off` already
   exists: a switch that flips all the shipped ids off at once. Worth knowing
   before building it that the Strength standard runs on the `STD` conversion
   ratios attached to the shipped lifts, so a blank start gives a working
   Progress number and a mostly empty Strength panel unless the user types a
   ratio per lift, which most will not. Correct behaviour, but it makes "start
   empty" a poorer first run than "start from mine and switch things off".
4. **Home-screen nudge and an icon.** A dismissible note on iOS Safari when
   `navigator.standalone` is false, explaining that installing it is what stops
   Safari clearing the data after seven days of not visiting. Plus an
   `apple-touch-icon`, since the home screen currently gets a screenshot.

**This does not give cross-device sync**, for me or anyone. Without a server
there is no channel between two devices, so hand export and import remains the
only route. Obsidian syncs across devices but the app only writes to it and
there is no importer, which was judged a bigger job than it is worth.

---

## 2. Offline

**Status:** known gap since 8 Sep 2026, prerequisite already done.

The app will not load offline at all. Dropping Google Fonts and embedding
Archivo as base64 removed the only external request, which was the
prerequisite, but no service worker exists yet.

Half an hour to write, and one permanent piece of discipline. It has to be a
separate same-origin file, so the app stops being a single file, which is the
real cost. The trap is the cache version string: forget to bump it on a deploy
and every installed phone freezes on an old version with no way to tell.
Cache-then-revalidate with a small "new version available, reload" nudge is the
safest shape, because a stale device then corrects itself.

---

## 3. Whoop

**Status:** designed 8 Sep 2026, blocked. Do not build until I own a Whoop.

Recovery, Strain, HRV, resting heart rate and sleep stay hand-typed until then.
Route already chosen: a Cloudflare Worker proxy with `/auth` and `/callback`,
holding the rotated refresh token in Workers KV. A static page cannot call
Whoop directly, and this is settled rather than assumed: the token exchange
needs a `client_secret` that Whoop's own guidance says must never reach a
client. A scheduled GitHub Action was rejected because this repo is public and
that would publish health data.

API specifics are recorded in the project memory and should be re-checked
before building, since they move.

---

## 4. InBody import

**Status: DONE.** Shipped in commit `2d358f5`, written against the real export
rather than from guesswork. Listed here only so it is not mistaken for
outstanding work.

The parser reads the unit out of each header and converts only when needed,
matches headers on a normalised form so the misspelled "Left leg Lean Mass"
column still lands, and treats "-" as absent rather than zero, which is why
lean mass is derived as weight minus fat instead of read from the column that
arrived empty. Paste the CSV on Body > Composition, it shows what it found,
nothing is written until it is confirmed.

Nothing is left to build. The one thing still ahead is simply time: the BW
versus lean chart needs a second scan before it has anything to compare.

---

## 5. Small open questions

- **Trap bar deadlift is filed under legs, not back.** A deliberate call, never
  confirmed. Worth a decision rather than leaving it implicit.
- **The add-on plate has never been weighed.** Some pin loads are written as
  "13+", meaning the stack at pin 13 plus the small loose plate that rests on
  top of it. That plate is not half a step down the stack, it is its own
  object, so the app holds its weight separately in `settings.addOnKg` and adds
  it whenever a load carries the "+". The figure currently sitting there,
  1.125kg, is a guess nobody has checked, and every "+" load inherits it. Put
  the plate on a scale and correct it under Setup > Cable stacks, and every
  affected load becomes right at once.
- **The `STD` conversion ratios are still unsourced.** The five group benchmark
  tables now trace to real published data, but the ratios that translate a
  non-anchor lift into what it implies for the group's benchmark lift were
  written from general familiarity. No source appears to exist for most of
  them. Do not describe the strength score as fully sourced.
