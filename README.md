# k8s-trail

What a room full of people did to a Kubernetes cluster, written down as Git
commits.

Nobody pushes to this repository by hand. [GitOps Reverser](https://github.com/ConfigButler/gitops-reverser)
watches objects in the `voter` namespace of `k8s.koudijs.dev` and writes each
change here as a commit. The **author** of each commit is the person the API server
authenticated for that change; the **committer** is `ConfigButler Bot`, and every
commit is signed. `git log --format=fuller` shows both.

Most commits from the room also carry that person's own words as the message:
the app asks for it when you save, and passes it on with the change.

## What is where

| Folder | What it holds | Written from |
|---|---|---|
| [`seats/`](seats/) | The room's seating plan, and one file per row with every claimed seat in it: who, which emoji, which colour | `Seat`, `SeatingPlan` |
| [`quizzes/`](quizzes/) | The quiz rounds: their questions, and whether a round is closed or live. Not the answers | `QuizSession` |
| [`room-pass/`](room-pass/) | The Room Pass rooms the audience joins through, all in `rooms.yaml` | `Room` |
| [`databases/`](databases/) | One file per "I need a database" request, in the requester's words | `Database` |
| [`configs/coffee/`](configs/coffee/) | The coffee bar's menu | `CoffeeConfig` |
| [`configs/gitops-reverser/`](configs/gitops-reverser/) | GitOps Reverser's own configuration: one file per folder above, plus the connections | `GitTarget`, `WatchRule`, `GitBranch`, `GitProvider`, `KubeProvider`, one `Secret` |

Only `spec` and a trimmed `metadata` are written. `status` never is, which is why
a Room's live join code is not here.

## The coffee menu goes through a pull request

Everything above is written to `main`, except the coffee menu. Edits to it land
on the branch **`coffee-change`**. `main`'s `configs/coffee/` is what production
runs: Flux applies it to `demo-production.koudijs.dev`, and can write nothing
else. So the room's edits reach production when someone opens a pull request
from `coffee-change` to `main` and merges it.

Merge it with a **merge commit**. A squash would make whoever merges the author
of everyone's change.

## The one Secret

`configs/gitops-reverser/secrets/voter/k8s-trail-git.sops.yaml` is the deploy
key GitOps Reverser uses to push to this repository, next to the `GitProvider`
that names it. It is encrypted with [SOPS](https://github.com/getsops/sops) and
age before it is written: the values are `ENC[…]`, and the recipient in its
`sops` block is the public half of a key that exists only in the cluster. It is
the only Secret mirrored here: the `WatchRule` selects Secrets carrying the label
`voter.configbutler.ai/mirror: "true"`, and this is the only one that has it.

## History

Until 2026-10-29 this repository used a different layout under `clusters/`,
which included every vote. That folder was removed from `main` in one commit and
remains in the history.
