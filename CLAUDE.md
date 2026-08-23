# CLAUDE.md

Guidance for Claude Code and other AI assistants working in this repository.

## What this repository is

This is a **GitHub Skills course repository**, generated from the [`skills/github-pages`](https://github.com/skills/github-pages) template. It is not an application and has no product code. Its purpose is to walk a learner through publishing a Jekyll blog with GitHub Pages, using GitHub Actions to detect the learner's progress and rewrite the README with the next lesson.

Two things live here at once, and it matters which one you are touching:

1. **The course machinery** — `.github/workflows/` and `.github/steps/`, plus the step content spliced into `README.md`. This is automation authored by GitHub, driven by Actions.
2. **The learner's Jekyll site** — `_config.yml`, `index.md`, and `_posts/` on the `my-pages` branch. This is the thing that actually gets published.

There is no package manager, no build script, no test suite, and no linter configured. `git` and a text editor are the entire toolchain. Do not go looking for `npm test` or a `Makefile`; nothing of the sort exists.

## Current state

- Repository: `TherapyGramz/meow`
- Course progress: **step 1** (`.github/steps/-step.txt` contains `1`) — the learner has not yet enabled GitHub Pages, or the step-1 workflow has not fired.
- `main` holds the course scaffolding only.
- `my-pages` exists and carries `_config.yml` (currently **empty** — no theme set yet) and `index.md` (title-only front matter stub).
- No `_posts/` directory yet.

Re-check `.github/steps/-step.txt` on both `main` and `my-pages` before acting; the workflows advance it without human involvement, so anything written here about progress goes stale quickly.

## Branch model

| Branch | Role |
| --- | --- |
| `main` | Published source for GitHub Pages. Receives the site only when the learner merges `my-pages`. |
| `my-pages` | Created automatically by the step-0 workflow. All Jekyll edits belong here. |
| `dependabot/*` | Automated GitHub Actions version bumps (monthly, per `.github/dependabot.yml`). |

Site edits go on `my-pages`, never directly on `main`. The step 2, 3, and 4 workflows only trigger on pushes to `my-pages`, so committing site changes to `main` silently breaks course progression — the learner does the work but the README never advances.

## How the step state machine works

Each `.github/workflows/N-*.yml` follows the same shape:

1. A `get_current_step` job checks out the repo and reads `.github/steps/-step.txt` into an output.
2. The main job runs only if the repository is **not** a template (`!github.event.repository.is_template`) **and** `current_step` equals this workflow's number.
3. On success it calls `skills/action-update-step@v2` with `from_step` / `to_step`, which writes the new number into `-step.txt` and replaces the body of `README.md` with the contents of the matching `.github/steps/` file.

The triggers, in order:

| Workflow | Fires on | Advances |
| --- | --- | --- |
| `0-welcome.yml` | push to `main` | 0 → 1. Also creates the `my-pages` branch with an empty `_config.yml` and a stub `index.md`. |
| `1-enable-github-pages.yml` | `page_build` event | 1 → 2 |
| `2-configure-your-site.yml` | push to `my-pages` touching `_config.yml` | 2 → 3 |
| `3-customize-your-homepage.yml` | push to `my-pages` touching `index.md` | 3 → 4 |
| `4-create-a-blog-post.yml` | push to `my-pages` touching `_posts/*.md` | 4 → 5 |
| `5-merge-your-pull-request.yml` | push to `main` | 5 → X (finish) |

Every workflow also accepts `workflow_dispatch`, which is the supported way to nudge a stuck step manually.

Note that the path filters are what matter, not the content. Touching `_config.yml` on `my-pages` at step 2 advances the course whether or not a theme was actually added, because the workflow checks nothing about the file itself.

## README and step content are generated — do not hand-edit

`README.md` is assembled from three parts: a `<header>` block, the current step's markdown from `.github/steps/`, and a `<footer>` block. The middle section is overwritten wholesale by `action-update-step` on every advance.

Consequences to respect:

- Editing the step section of `README.md` by hand is pointless; the next workflow run discards it.
- Editing `.github/steps/*.md` changes the *lesson text* the learner will see on future advances. Only do this if the user explicitly wants to alter the course itself.
- Never edit `.github/steps/-step.txt` by hand to "fix" progress unless the user asks for exactly that. It and the README body must stay consistent — a mismatch leaves the course showing one step while the workflows guard a different one, and no workflow will fire.
- The header and footer comments in `README.md` are author notes from the template. Leave them.

## Jekyll conventions the course teaches

These are the rules the learner is being graded against, and the rules any site edit should follow:

- **Theme** is set in `_config.yml` as `theme: minima`. Other keys (`title:`, `author:`, `description:`) are optional customization.
- **Homepage** is `index.md`. GitHub Pages prefers `index.md` over `README.md` when both exist.
- **Blog posts** must be named `_posts/YYYY-MM-DD-title.md` — hyphens between words in the title, real date in the filename. An invalid date fails the Pages build with "Page build failed: Invalid post date".
- **Front matter** is YAML fenced by `---` at the very top of the file. Posts need at least `title` and `date`.

## Building and previewing

The site is built server-side by GitHub Pages; there is no local build step and no `Gemfile` in the repository. If a local preview is genuinely needed, it requires adding `Gemfile` + `Gemfile.lock` with the `github-pages` gem and running `bundle exec jekyll serve` — treat that as a change to propose, not something to add unprompted, since new tracked files at the repo root can interfere with the course's path-filtered workflows.

Validation before pushing is limited to what you can check by reading: YAML front matter is well-formed, post filenames match `YYYY-MM-DD-title.md`, and workflow YAML parses. Nothing in CI lints these.

## Working conventions for AI assistants

- **Prefer the smallest change.** This is a teaching repository; large refactors of the workflows defeat its purpose.
- **Ask which layer is intended** when a request is ambiguous between "help me finish the course" (edit `my-pages` site files) and "change how the course works" (edit `.github/`).
- **Do not fabricate course progress.** If the user is stuck at a step, diagnose the trigger — wrong branch, wrong path, repository still marked as a template, or `-step.txt` out of sync — rather than bumping the counter.
- **Respect the branch you were assigned.** Session instructions may pin work to a specific `claude/*` branch; that overrides the "site edits go on `my-pages`" convention for documentation-only changes like this file.
- `.gitignore` covers compiled binaries, archives, logs, and OS cruft. It does **not** ignore `_site/`, so a local Jekyll build would need that added before committing.

## License

MIT, © 2023 GitHub. See `LICENSE`.
