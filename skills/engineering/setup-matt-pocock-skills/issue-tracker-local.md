# Issue tracker: Local Markdown

Issues and specs for this repo live as markdown files in `.scratch/`.

## Conventions

- One feature per directory: `.scratch/<feature-slug>/`
- The spec is `.scratch/<feature-slug>/spec.md`
- Implementation issues are one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01`, never a single combined tickets file
- Triage state is recorded as a `Status:` line near the top of each issue file (see `triage-labels.md` for the role strings)
- Comments and conversation history append to the bottom of the file under a `## Comments` heading

### The resolved state

The five triage roles in `triage-labels.md` answer "who should act on this ticket next". None of them means "done" — on GitHub that is `closed`, a separate axis from labels. A single `Status:` line has no second axis, so the resolved state needs a string of its own:

**`resolved`** — the ticket's acceptance criteria are met. Optionally followed by a parenthetical citing the commits and any deliberate deviations: `Status: resolved (implemented in abc1234; one deviation: …)`.

Don't reach for a triage role to express completion. `ready-for-human` on a finished ticket reads as "waiting for a human to build it", which is the opposite of what happened. Conversely, a ticket whose remaining work genuinely needs a human — a step the agent is sandboxed out of, say — stays `ready-for-human` and is not resolved, however much of it is already built.

`resolved` is the same string the wayfinding section below uses.

`.scratch/` **is tracked by git.** Don't add it to `.gitignore`. It holds specs and wayfinder maps, not just tickets — losing it loses the reasoning behind the work.

### Archiving a finished feature

When every ticket in a feature directory is `resolved`, **delete the directory** — one commit per feature, `chore(tracker): archive <feature-slug>`. Git history is the archive: the deletion is itself tracked, so "losing the reasoning" doesn't apply — `git log --oneline -- .scratch/<feature-slug>/` brings the whole feature back.

Three ordering constraints:

- **Close-out before archive.** A directory holding any open or `ready-for-human` ticket is not finished, however much of it is built — never archive around a live ticket.
- **Archive before inbox triage.** Clear finished features first, so `/triage` promotes cards into a tracker that only shows live work.
- **Sweep dead pointers before delete.** Grep for long-lived files pointing into the directory. Git history preserves the content, not the pointer — a `See .scratch/<slug>/issues/05-….md` line in a module docstring survives the deletion as a dangling reference, and the reader of that module has no way to know it once resolved. Move what each pointer was carrying to where it will actually be read, then drop the pointer.

## Inbox: ideas that don't belong to a feature yet

- Path: `.scratch/inbox/<slug>.md` — repo-global, not under any `<feature-slug>/`
- Not numbered — numbering happens on promotion
- Carries `Status: needs-triage`
- Shape matches the rest of the tracker: a `Status:` line near the top, then a plain markdown body. Nothing else — no frontmatter, no extra structure.

A card that needs to carry material with it — a salvaged script, a screenshot, a log — becomes a directory instead: `.scratch/inbox/<slug>/<slug>.md` alongside its files. Same card, same `Status:` line; the directory is just where the attachments live. Reach for it only when the material genuinely can't be a link — the single file is the default.

Example inbox card, `.scratch/inbox/dark-mode-toggle.md`:

```markdown
# Dark mode toggle

Status: needs-triage

Users keep asking for a dark mode toggle in settings. Not sure yet
which feature this belongs to, or whether it's its own feature.
```

Promoting an inbox card onto the board — filling it out, assigning it an id, setting its `Status`, and moving it into `<feature-slug>/issues/` — is the `triage` skill's job, not defined here. This section only defines what the card looks like and where it lives before that happens.

## When a skill says "publish to the issue tracker"

Create a new file under `.scratch/<feature-slug>/` (creating the directory if needed).

## When a skill says "fetch the relevant ticket"

Read the file at the referenced path. The user will normally pass the path or the issue number directly.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a file with one **child** file per ticket.

- **Map**: `.scratch/<effort>/map.md` (the Notes / Decisions-so-far / Fog body).
- **Child ticket**: `.scratch/<effort>/issues/NN-<slug>.md`, numbered from `01`, with the question in the body. A `Type:` line records the ticket type (`research`/`prototype`/`grilling`/`task`); a `Status:` line records `claimed`/`resolved`.
- **Blocking**: a `Blocked by: NN, NN` line near the top. A ticket is unblocked when every file it lists is `resolved`.
- **Frontier**: scan `.scratch/<effort>/issues/` for files that are open, unblocked, and unclaimed; first by number wins.
- **Claim**: set `Status: claimed` and save before any work.
- **Resolve**: append the answer under an `## Answer` heading, set `Status: resolved`, then append a context pointer (gist + link) to the map's Decisions-so-far in `map.md`.
