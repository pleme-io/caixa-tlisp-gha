# caixa-tlisp-gha

GitHub Actions operations for tatara-lisp: run state, dispatch, polling to a
terminal conclusion, and reusable-workflow ref repointing.

Eighth member of the `caixa-tlisp-*` family (`stdlib`, `core`, `git`, `cargo`,
`helm`, `handoff`, `rollout`).

## Why

A reusable workflow is referenced by its callers as
`uses: owner/repo/.github/workflows/name.yaml@master`. A branch containing a
change to that workflow is therefore **never the thing that runs** — dispatching
from the branch still resolves `@master`.

The only way to exercise the change before merging is to repoint a caller at the
branch, dispatch, and put the ref back. That sequence is easy to leave
half-finished, and a repo left pointing at a feature branch is worse than never
having qualified anything. Hence `gha:uses-restore` as a first-class primitive,
callable on its own — including by a later session that did not do the
repointing.

## Surface

| Function | Does |
|---|---|
| `gha:latest-run-id repo workflow` | newest run id for a workflow |
| `gha:run-status repo id` | `queued` / `in_progress` / `completed` |
| `gha:run-conclusion repo id` | `success` / `failure` / … |
| `gha:run-url repo id` | browser URL |
| `gha:wait-run repo id polls interval` | poll to a terminal conclusion |
| `gha:job-lines repo id` | per-job conclusions, one per line |
| `gha:dispatch repo workflow ref inputs` | `workflow_dispatch` with `key=value` inputs |
| `gha:uses-ref-of file target` | the `@ref` a caller currently pins |
| `gha:uses-repoint file target from to` | move that ref, leaving a `.caixa-bak` |
| `gha:uses-restore file` | put it back |
| `gha:uses-repointed? file target expected` | is this caller off its expected ref? |

## Notes

**Polling, never sleeping.** `gha:wait-run` polls to a terminal state with a
bounded retry count. A guessed fixed delay is either a wasted wait or a false
read — self-hosted spot runners regularly spend minutes in cold start before a
job begins.

**Inputs are discrete arguments.** `gha:dispatch` expands each `key=value` into
its own `-f` argument rather than interpolating one string, so a value
containing a space or a quote cannot change the shape of the command.

## Use

Inline it — `require` does not import at `tatara-script` top level. Concatenate
this file (minus any `provide` form) ahead of your script, alongside
`caixa-tlisp-handoff` when the work needs an operator's hands.
