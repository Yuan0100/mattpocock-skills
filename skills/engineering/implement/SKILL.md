---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

Use /tdd where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, use /code-review to review the work.

Commit your work:
- Check the current branch. If it's `dev` or `main`, create a branch first (`feature/`, `fix/`, `chore/`, `refactor/`, `docs/` + short desc).
- Stage each touched path individually — `git add <path>` per file. Never `git add -A`, `git add .`, or add a whole directory.
- Unrelated changes in the working tree: report them, leave them alone.
