# Task: update the pi0.5 phrasing survey to the 22-task v2 set

## Context

There's a live static survey at `mikeywatts.xyz/robot-phrasing-survey/` (repo:
`~/dev/mikeywatts.xyz`, this file's own repo) that shows friends short robot-arm
clips and asks them to type an instruction, in one of three registers
(robot-assistant / child / adult). It's built as 6 plain static HTML pages —
no build step, no framework — plus a random-entry redirector and a link hub.

The pi0.5 (LIBERO) task set it draws from just changed from 20 tasks to a new,
finalized 22-task set. This task is: swap in the new 22, leaving the other
half of the survey (8 WidowX clips from `labeling_pack`) untouched, and update
every place a task id is used so CSV merge-back still works.

**This has already been confirmed with the user** — proceed directly, but
still show the diff and ask before the final `git push` (that's the standing
norm for this repo: build → test locally → show diff → confirm → push).

## Hard constraints (do not skip these)

1. **Never display the canonical instruction anywhere in the built pages.**
   The v2 manifest (see below) includes each task's `instruction` field —
   that's the ground-truth phrase policies were evaluated on. It must never
   appear in the HTML, JS, or any visible text. Respondents only ever see the
   raw clip and a blank textarea. This mirrors what the project's own
   `PREREG.md` log for this bank calls "purer elicitation."
2. **Some of these 22 tasks are the actual sealed FINAL_EVAL set** (see
   `stratum`/`origin: sealed` in the manifest — not just informal dev tasks).
   The commit that added them (`6ef837c5`, "pi05 FINAL_EVAL: human-labeling
   media pack for the 22 eval tasks") and the manifest's own `"prereg":
   "2026-09-04 FINAL_EVAL design"` field indicate this was already logged
   under this project's sealed-set discipline in another session — you don't
   need to re-clear it, but do not do anything beyond what this doc asks
   (e.g. don't print/log the instructions anywhere outside this file, don't
   commit a copy of the manifest's `instruction` text into the site repo).
3. **On-page task labels stay anonymous.** Existing pages show only "Clip N
   of M" — never a task id, slug, or name. Keep that. The real id only lives
   in the (invisible) form field `name` attribute, which is how CSV
   merge-back works.
4. **Videos never get tab focus.** Every `<video>` has `tabindex="-1"` so Tab
   skips its native controls straight to the next textarea. Keep this on the
   new clips too.
5. **Don't touch the WidowX clips or their field names at all.** Only the
   pi0.5/LIBERO half changes.

## Source data (new, replaces the old 20)

Repo: `phrase-rl` (a sibling checkout should exist at
`~/dev/robotics/phrase-rl` or one of its worktrees — `cd` there, or clone
`git@github.com:sttawm/phrase-rl.git` fresh, then `git fetch origin main`).

Path: `results/analysis/pi05_bank/human_eval_v2/assets/<slug>/clip.mp4`
(also has `start.png`/`done.png` per task — **not used** by the survey, same
as before: only `clip.mp4` goes into the form).

Manifest: `results/analysis/pi05_bank/human_eval_v2/manifest.json` — has
`n_tasks: 22` and a `tasks` array with `slug`, `instruction` (DO NOT
display), `stratum`, `origin` per task.

Pull the videos like this (from inside the phrase-rl checkout):

```bash
mkdir -p /tmp/pi05_v2
for slug in libero_spatial_00 libero_spatial_05 libero_spatial_07 libero_spatial_08 \
            libero_object_01 libero_object_02 libero_object_06 libero_object_09 \
            libero_90_51 libero_90_30 libero_90_47 libero_90_67 libero_90_20 \
            libero_90_07 libero_90_43 libero_90_53 libero_90_00 libero_90_13 \
            libero_90_45 libero_90_49 libero_90_73 libero_90_27; do
  git show "origin/main:results/analysis/pi05_bank/human_eval_v2/assets/$slug/clip.mp4" \
    > "/tmp/pi05_v2/$slug.mp4"
done
```

(That's the full list of 22 slugs — also re-derivable from the manifest's
`tasks[].slug` field if you want to double-check it against the live repo
rather than trust this copy.)

## What changes in the site repo

Current structure (`~/dev/mikeywatts.xyz/robot-phrasing-survey/`):

```
index.html          -- link hub (6 links)
start.html           -- random-entry redirector, list of 6 page ids, no changes needed here except nothing (page ids don't change)
thanks.html          -- interstitial ("thanks, do another set?" / "all done"), no changes needed
robot-1.html / robot-2.html
kid-1.html   / kid-2.html
adult-1.html / adult-2.html
videos/              -- flat directory, files named <task-or-slug>.mp4
```

Each of the 6 survey pages is self-contained (own `<style>`, own `<script>`),
generated from a Python script pattern (not currently checked into the repo —
recreate one, see skeleton below). Each page has:
- a "Part N of 2" tag
- a "continued-banner" (shown when arriving via `?continued=1`)
- an "About you" demographics block (age/gender/AI-familiarity radios) — id
  `demographics-block`, hidden via a `skip-on-continue` class when arriving
  via the loop
- N task cards, each: `<p class="task-index" data-dir="...">Clip</p>` (text
  gets overwritten to "Clip X of N" by JS after shuffling), a
  `<video tabindex="-1" autoplay controls muted loop playsinline
  src="videos/....mp4">`, a label with the audience's question, and a
  `<textarea name="{merge_id}_{audience}">`
- a submit row
- a `<script>` block: shuffles task cards each load, restores/persists
  demographics via `localStorage`, and on submit walks a fixed 6-page CYCLE
  (`["robot-1","robot-2","kid-1","kid-2","adult-1","adult-2"]`) to compute
  the next unfinished part (tracked via `localStorage.pi05_completed`),
  setting the hidden `_next` field to
  `https://mikeywatts.xyz/robot-phrasing-survey/thanks.html?next=<nextpage>`
  (or no `?next=` if the loop is complete).

Formspree endpoint (shared across all 6 pages): `https://formspree.io/f/mdeogdlo`

### The actual change

- **Merge-id scheme, old → new:** `pi05_task_07` (etc.) → `pi05_libero_90_07`,
  `pi05_libero_spatial_00`, `pi05_libero_object_01`, etc. — i.e.
  `pi05_<slug>` where `<slug>` is exactly the manifest's `slug` field. Field
  name becomes `pi05_<slug>_<audience>` (e.g. `pi05_libero_spatial_00_robot`).
  This is the literal string that becomes the Formspree/CSV column header —
  it's how old and new responses stay mergeable by task, so get it exact.
- **Video files:** copy each pulled clip into
  `robot-phrasing-survey/videos/<slug>.mp4` (e.g. `videos/libero_spatial_00.mp4`).
  Delete the old 20 `task_NN.mp4` files (task_07, task_08, task_11, task_18,
  task_26, task_30, task_43, task_45, task_51, task_53, task_58, task_62,
  task_65, task_69, task_73, task_80, task_83, task_85, task_86, task_88) —
  they're being fully replaced, not added to.
- **Per-page split, 15 total each (was 14):** 11 LIBERO + 4 WidowX per page,
  no repeats across the two pages. WidowX split is unchanged from what's
  already in the pages (first 4 vs last 4 of the existing 8 — check the
  current `robot-1.html`/`robot-2.html` for exactly which 4 go where, since
  that part doesn't change). For the 22 LIBERO slugs, split 11/11 — order
  doesn't matter since pages shuffle client-side anyway; simplest is first
  11 of the manifest's `tasks` array on part 1, remaining 11 on part 2.
- **Copy text:** anywhere the page text says a task count (e.g. "You'll
  watch 14 short clips...", "14 clips" in a progress line if still present,
  any lingering "20 sealed tasks" language) — update to 15 / drop stale
  counts as needed. Check all 3 audiences' lede text.
- **`index.html`** mentions total clip counts too ("28 clips total" or
  similar) — update to the new total (30).

### Recommended approach: rebuild via a generator script

Rather than hand-editing 6 files with sed (error-prone for a change this
size), write a small Python generator — same pattern this project has used
each time the page set changed — that emits all 6 pages from one source of
truth (audience copy + task list + CSS + JS templates), then diff the output
against git before committing. Read the current `robot-1.html` in full first
to copy its exact CSS/JS verbatim (don't redesign — same look, same behavior,
just new task list + counts). Pay special attention to reproducing the
shuffle/loop/demographics-persistence script exactly; it's easy to
subtly break the CYCLE/nextPage logic when regenerating by hand.

## Testing before push

This repo has no dev server config checked in (by design — it's created
per-session and cleaned up after). To preview locally:

```bash
mkdir -p .claude
cat > .claude/launch.json <<'EOF'
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "site-preview",
      "runtimeExecutable": "python3",
      "runtimeArgs": ["-m", "http.server", "8899", "--directory", "/Users/sttawm/dev/mikeywatts.xyz/robot-phrasing-survey"],
      "port": 8899
    }
  ]
}
EOF
```

Then use the browser preview tool to load each of the 6 pages and confirm:
- 15 clips per page, videos actually play
- no task id/slug visible anywhere on the page (view page text, not just
  screenshot)
- textarea `name` attributes contain the new `pi05_<slug>_<audience>` ids
  (check via `document.querySelectorAll('textarea')` in devtools/JS console)
- the loop still computes a sane `_next` value on submit (can test by
  dispatching a `submit` event on the form and reading the hidden `_next`
  field's value afterward, without actually letting it navigate to Formspree)

Delete `.claude/launch.json` when done (don't commit it).

## Push

Stage everything under `robot-phrasing-survey/`, write a commit message that
says what changed (20→22 task pi0.5 set, new source path, new merge-id
scheme, 14→15 per page), **show the user the diff/stat and get an explicit
go-ahead before running `git push origin main`** — this is a live public
site and every prior change in this project went through that same
confirm-before-push step.
