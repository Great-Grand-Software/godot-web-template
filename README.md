# godot-web-template

A Godot 4 web game that builds, tests, deploys and is **playable in a browser**
from the moment you clone it — with no secrets, no API keys, and no paid
services.

Clone it, replace the placeholder game, and every pull request gets its own
playable URL.

---

## For an agent: fresh clone → first green PR

**Read this section first if you are an agent working in a repo created from
this template.** It is the whole of what you need to get from a fresh clone to
a merged pull request. Everything below the next `---` is context; this is the
procedure.

### Do these in order

**1. `scripts/bootstrap.sh`. Always, before anything else.**

```bash
scripts/bootstrap.sh
export PATH="$HOME/.godot-toolchain/bin:$PATH"
```

Nothing works before it — there is no `godot`, no `gdlint`, no GUT on the
container. It is idempotent: run it again if you are unsure. `--check` verifies
without installing.

**2. Prove the untouched checkout is green before you change anything.**

```bash
scripts/check-constraints.sh   # every line PASS
gdlint .                       # "Success: no problems found"
godot --headless --import --path .          # once, on a fresh clone
godot --headless --path . -s addons/gut/gut_cmdln.gd -gconfig=.gutconfig.json
```

The import step is not optional on a clone that has never been opened: GUT's
`class_name`s are not registered until the project is imported once, and the
test run aborts with "Some GUT class_names have not been imported" instead of
failing a test. CI runs the same step for the same reason.

The test run must report **12 tests passing across 3 scripts**. Do this even
when your task looks trivial. A failure you see *after* editing is ambiguous
unless you know the baseline was clean; if the baseline is not clean, stop and
say so rather than debugging your own change.

**3. Edit `project.conf` before you touch code.** It is the only file holding
per-project values — entry scenes, frame size, guardrail budgets. Everything in
`scripts/` and `.github/` is project-neutral and carries forward untouched.
`check-constraints.sh` reads its expectations from `project.conf` and asserts
them against `project.godot`, so those two must agree: change a frame value in
one and you must change it in the other, or `constraints` fails.

**4. Replace the placeholder game.** Delete `scripts/game/`, `scenes/game.tscn`
and their tests, and write yours. The placeholder is a 5×5 tap grid that exists
only to prove the pipeline end to end; it is meant to be deleted.

**5. Re-run the three commands from step 2 before every push.** They are the
first three of the six CI checks, and they run in seconds locally against
minutes in CI.

### The main menu contract

`scenes/main_menu.tscn` is the boot scene (`run/main_scene` in
`project.godot`). Its START button hands off to `scenes/game.tscn` via the
`GAME_SCENE` constant in `scripts/ui/main_menu.gd`.

**Change what START leads to — never whether it exists.** Point `GAME_SCENE` at
your scene. Do not remove the button, rename its unique name (`%Start`), or
delete the menu to boot straight into play.

`tests/unit/test_main_menu.gd` asserts the button is present and that its
target scene resolves, because a START pointing at a missing scene reviews fine
and dead-ends the player. If that test fails, the fix is your scene path, not
the test.

### Files that need a code owner's approval

`.github/CODEOWNERS` gates these. Touching one turns your PR into someone
else's decision, so do not edit them casually, and never as a side effect of
unrelated work:

- `.github/workflows/`, `.github/stamp-build.sh` — CI, previews, deploy
- `scripts/bootstrap.sh`, `scripts/check-constraints.sh`,
  `scripts/apply-repo-settings.sh` — the environment and the gates themselves
- `.claude/`, `CLAUDE.md`, `.github/CODEOWNERS`
- `.gdlintrc`, `project.godot`, `project.conf`

Those last three are gated for the same reason the workflows are: **they define
what the gates mean.** `.gdlintrc` *is* the lint gate — adding a rule to its
`disable:` list makes `lint` pass on code it should reject. `project.godot` and
`project.conf` carry the frame, renderer and threading constraints that no
other job re-derives. Config a gate reads is part of the gate.

> **In a fresh repo, `CODEOWNERS` says `@OWNER`,** a deliberate placeholder.
> GitHub flags it as an unknown user — it does so in this template repository
> itself, where the warning is expected and harmless. Resolving it to a real
> username or team is one of the first things a new project does; until then
> the file renders in the UI but gates nothing.

### What you cannot do — hand these to a human

These need repository-admin rights or the GitHub UI. Do not attempt them, do
not treat them as blockers, and do not work around them. Finish everything else
and say plainly which of these are outstanding:

- **Enabling Pages** (Settings → Pages → branch `gh-pages`, folder `/ (root)`).
  Previews and the deployed game do not exist until this is done.
- **Running `scripts/apply-repo-settings.sh apply`.** It needs an admin token.
  You may run `plan` to show what it would write; it calls nothing.
- **Creating repositories, changing visibility, changing permissions**, adding
  collaborators, or adding secrets.
- **Approving or merging a PR.** Every PR needs a human review, and an agent
  cannot approve one. Auto-merge only switches on when a person adds the
  `tested` label, meaning they played the preview build.

### Hard rules

Violating one of these makes a PR wrong no matter how good the code is:

- **No threads.** Never enable `variant/thread_support` in
  `export_presets.cfg`, and never raise `worker_pool/max_threads`. GitHub Pages
  cannot serve the COOP/COEP headers Godot's threaded Web export requires — the
  build succeeds and then fails in the browser.
- **No C#/.NET.** C# cannot export to Web in Godot 4 at all.
- **No 3D.** 2D only: `Control`, `Node2D`, sprites, tilemaps.
- **No secrets, no paid services.** Every job runs on the token GitHub mints
  per run. Adding a credential is what makes a pipeline stop being reusable.
- **Never weaken a gate to make CI pass.** Do not relax a `.gdlintrc` rule,
  raise a `project.conf` budget, delete or skip a test, or drop a required
  check because it is red. A red gate is information. If a limit is genuinely
  wrong, raise it deliberately in its own PR that says why — and expect a code
  owner to read it.

### When CI is red

Check for **"cancelled", not just "failed"**. Pushing twice quickly cancels the
first run, and a cancelled run reports no failures *and* no successes — the PR
sits unmergeable with nothing red on it. Push an ordinary commit to re-run.

---

## What you get

| | |
|---|---|
| **Six CI checks** | constraints, lint, unit tests, headless boot, Web export, real-browser boot |
| **A playable preview per PR** | posted as a comment, deleted when the PR closes |
| **Automatic deploy** | every merge to `main` republishes the public page |
| **Build provenance** | a stamp in the corner tells you which commit you are playing |
| **Branch protection as code** | one script, checked in and reviewable |
| **Zero secrets** | every job runs on the token GitHub mints per run |

That last row is the one that makes this cheap to reuse. There is nothing to
provision, expire, or pay for.

---

## Start a new game from this template

**1. Make the repo.** Click **Use this template → Create a new repository** on
GitHub. Make it **public** — Pages on a private repo needs a paid plan, and the
previews depend on Pages.

**2. Get it running.**

```bash
git clone https://github.com/<owner>/<your-repo>.git
cd <your-repo>
scripts/bootstrap.sh
export PATH="$HOME/.godot-toolchain/bin:$PATH"
```

`bootstrap.sh` installs Godot, the export templates, gdtoolkit and GUT at
pinned versions. It is idempotent and takes about ninety seconds the first
time, under a second after that. It is the single source of truth for the
environment — CI runs this exact script, so there is one definition rather than
two that drift.

**3. Push to `main` and let CI run once.** Required status checks have to be
seen by GitHub by name before they can be required, so the first run has to
happen before step 5.

**4. Turn on Pages.** Settings → Pages → Source: **Deploy from a branch** →
branch **`gh-pages`**, folder **`/ (root)`**.

> **This is the step everyone gets wrong.** The dropdown defaults to `main`.
> `main` holds your *source*, which has no `index.html`, so Jekyll renders your
> README as a text page and the game never appears. It must be `gh-pages`.
> That branch does not exist until the first deploy runs, which is why this
> step comes after step 3, not before.

**5. Lock the branch.**

```bash
export GH_TOKEN=<a token with admin rights on the repo>
scripts/apply-repo-settings.sh plan     # prints what it will write, calls nothing
scripts/apply-repo-settings.sh apply
scripts/apply-repo-settings.sh verify   # exit 0 and all PASS
```

Owner and repo are read from your git remote, so there is nothing to edit.

Your game is now at `https://<owner>.github.io/<your-repo>/`.

---

## Make it your game

**Edit `project.conf`.** It is the only file with project-specific values in
it — frame size, entry scenes, and the guardrail budgets. Everything in
`scripts/` and `.github/` is project-neutral and carries forward untouched.

```bash
ENTRY_SCENES="res://scenes/main_menu.tscn res://scenes/game.tscn"
VIEWPORT_WIDTH=720                      # asserted against project.godot
VIEWPORT_HEIGHT=960
MAX_AUTOLOADS=1
MAX_RASTER_PX=512
```

**Then replace the placeholder.** Delete `scripts/game/`, `scenes/game.tscn`
and the tests for them, and write yours. Keep `scripts/autoload/` down to one
file, or raise `MAX_AUTOLOADS` deliberately.

**Keep the main menu.** `scenes/main_menu.tscn` is the boot scene and ships
with a working START button that hands off to `scenes/game.tscn`, plus a QUIT
that hides itself on the web build where quitting means nothing. Every game
from this template gets a deliberate entry point rather than dropping the
player straight into play. Change *what START leads to*, not whether it exists
— `tests/unit/test_main_menu.gd` asserts the button is there and that its
target scene actually resolves, because a START button pointing at a missing
scene looks fine in review and dead-ends the player.

**Resolve `@OWNER` in `.github/CODEOWNERS`.** It ships as a placeholder, so
GitHub renders an "unknown user" warning against it — including in the template
repository itself, where that warning is expected. Replace it with the username
or team that should approve changes to the gated files; until you do, the file
displays in the UI but gates nothing.

**Then rewrite `CLAUDE.md`.** It ships with the constraints that are true of
*any* Godot Web build — no C#, no threads, no 3D — plus placeholders for the
ones specific to your game. Agents working in the repo read it on entry, and
the `constraints` check enforces the mechanical half of it.

The placeholder game is a 5×5 grid you tap. It exists to prove the pipeline
end to end, and it is deliberately small enough to delete without regret.

---

## Daily loop

1. Branch **in this repo** — not a fork. Fork PRs get a read-only token, so
   the preview cannot deploy.
2. Push. CI runs six checks and builds a preview.
3. A bot comments the preview URL. **Open it and play.** CI proves the build
   is not broken; only a person can say whether it is *right*.
4. Get a review from another person. You cannot approve your own PR.
5. Add the `tested` label. Auto-merge takes it from there once checks and the
   review are green.

Run the fast checks yourself before pushing:

```bash
scripts/check-constraints.sh
gdlint .
godot --headless --path . -s addons/gut/gut_cmdln.gd -gconfig=.gutconfig.json
```

(On a clone that has never been imported, run `godot --headless --import --path .`
once first — GUT's `class_name`s are not registered before that.)

---

## Why it is built this way

**No secrets.** An earlier version of this pipeline used an LLM reviewer, which
needed an API key, which needed billing and a decision about whose account paid
for it. Replacing the mechanical half of that review with
`scripts/check-constraints.sh` — a deterministic script with no network
access — removed the last credential from the repo. What remained was
judgement, which is what human review is for.

**Checks that fail for the right reason.** `check-constraints.sh` only asserts
what can be asserted without false positives. A check that cries wolf gets
ignored, and an ignored check is worse than no check.

**Config the gates read is gated too.** `.gdlintrc` *is* the lint gate —
relaxing a rule there makes `lint` pass on code it should reject. So it sits in
`CODEOWNERS` alongside the workflows, and so does `project.godot`, which
carries constraints no other job re-derives.

**One source of truth for the environment.** Tool versions live in
`scripts/bootstrap.sh` and nowhere else. CI reads the Godot version out of that
file rather than repeating it, and the toolchain cache key is a hash of it, so
a version bump invalidates the cache by construction.

---

## Layout

```
.
├── project.conf              ← the only per-project file
├── CLAUDE.md                 ← rules for agents and reviewers
├── project.godot             ← frame, renderer, threads off
├── export_presets.cfg        ← Web preset; thread_support MUST stay false
├── .claude/                  ← SessionStart hook: git identity + bootstrap
├── scripts/
│   ├── bootstrap.sh          ← run first; single source of truth
│   ├── check-constraints.sh  ← the `constraints` gate; run it locally
│   ├── apply-repo-settings.sh← branch protection, derived from your remote
│   ├── autoload/             ← GameState, the only autoload
│   ├── game/                 ← pure logic, testable without a scene
│   └── ui/                   ← screens
├── scenes/                   ← main_menu.tscn (boot) + game.tscn
├── tests/unit/               ← GUT suite
└── .github/workflows/        ← the six checks, previews, deploy
```

---

## Gotchas worth knowing before you hit them

- **Pages must point at `gh-pages`.** See step 4. It is the single most common
  failure, and it looks like a broken build when it is a wrong dropdown.
- **The preview URL is case-sensitive in its repo path.** The owner is
  lowercased because a hostname must be; the repo is spelled as GitHub spells
  it. The Pages settings page shows the authoritative URL.
- **Pushing twice quickly cancels the first CI run.** `cancel-in-progress` is
  deliberate, but a cancelled run reports no failures *and* no successes — so a
  PR can sit unmergeable with nothing red on it. Look for "cancelled", not just
  "failed".
- **A merged PR's preview is deleted.** That is by design: its build becomes
  the root URL. The link on a merged PR will 404.
- **`git` identity in a fresh container.** The SessionStart hook sets it from
  whoever the session is authenticated as, so commits link to real accounts
  instead of all landing under one anonymous name. Working outside Claude Code,
  set `user.email` to your `users.noreply.github.com` address yourself.
