# Repository readiness review

Date: 2026-09-25

Scope: local security verification and GitHub publication. No Claude Cloud account was connected, and no Routine, schedule, or email connector was created or used.

## Result

Two independent repositories were published under the verified GitHub account `moguzgco`:

- Public implementation: `https://github.com/moguzgco/job-search-agent`
  - Branch: `main`
  - Initial published commit: `70b08087ab851be8009a5f89bbc62c4a28a8535e`
  - Verified GitHub visibility: `PUBLIC`
  - Verified GitHub fork status: `false`
- Private working repository: `https://github.com/moguzgco/my-job-search`
  - Branch: `claude/job-search-state`
  - Initial published commit: `8fe8e868c6e3d165df17b4a4aab857a7f616f423`
  - Verified GitHub visibility: `PRIVATE`
  - Verified GitHub fork status: `false`
  - Fetch-only `upstream`: the public GitHub repository
  - `upstream` push URL: `DISABLED`
  - `remote.pushDefault`: `origin`

The repositories have separate Git directories and no object alternates. The private repository shares reusable commit ancestry so it can merge upstream changes, but it is not a local linked worktree or clone that borrows objects. GitHub confirms that it was created independently rather than as a fork.

The private repository contains placeholder-only candidate inputs, empty history/run state, and no resume or generated job report. Fifty explicit `<TODO: ...>` markers remain for the user. Synthetic demonstration facts were not copied into the real candidate paths.

## Exact public tracked-file list

The public repository tracks exactly 25 files:

```text
.gitignore
CLAUDE.md
README.md
docs/checkpoints/deployment-readiness.md
docs/checkpoints/local-review.md
docs/checkpoints/repository-readiness.md
fixtures/demo/candidate/config.md
fixtures/demo/candidate/preferences.md
fixtures/demo/candidate/profile.md
fixtures/demo/data/discovered-jobs.run-1.json
fixtures/demo/data/matches.run-1.json
fixtures/demo/data/reported-jobs.after-run-1.json
fixtures/demo/data/reported-jobs.after-run-2.json
fixtures/demo/data/reported-jobs.before.json
fixtures/demo/data/run-state.after-run-2.json
fixtures/demo/reports/2026-09-25-run-1.md
fixtures/demo/reports/2026-09-25-run-2.md
instructions/discovery.md
instructions/matching.md
instructions/reporting.md
templates/config.example.md
templates/preferences.example.md
templates/private.gitignore.example
templates/profile.example.md
templates/routine-prompt.example.md
```

Root `candidate/`, `data/`, `reports/`, credentials, secret-like files, and local Claude settings are ignored. The similarly named paths under `fixtures/demo/` are intentionally tracked, clearly labeled synthetic demonstration data.

## Exact private tracked-file list

The private repository tracks exactly 31 files:

```text
.gitignore
CLAUDE.md
README.md
candidate/config.md
candidate/preferences.md
candidate/profile.md
data/reported-jobs.json
data/run-state.json
docs/checkpoints/deployment-readiness.md
docs/checkpoints/local-review.md
docs/checkpoints/repository-readiness.md
fixtures/demo/candidate/config.md
fixtures/demo/candidate/preferences.md
fixtures/demo/candidate/profile.md
fixtures/demo/data/discovered-jobs.run-1.json
fixtures/demo/data/matches.run-1.json
fixtures/demo/data/reported-jobs.after-run-1.json
fixtures/demo/data/reported-jobs.after-run-2.json
fixtures/demo/data/reported-jobs.before.json
fixtures/demo/data/run-state.after-run-2.json
fixtures/demo/reports/2026-09-25-run-1.md
fixtures/demo/reports/2026-09-25-run-2.md
instructions/discovery.md
instructions/matching.md
instructions/reporting.md
reports/README.md
templates/config.example.md
templates/preferences.example.md
templates/private.gitignore.example
templates/profile.example.md
templates/routine-prompt.example.md
```

The private `.gitignore` permits ordinary staging of these operational paths without force-adds:

```text
candidate/profile.md
candidate/preferences.md
candidate/config.md
candidate/resume.pdf       optional and currently absent
data/reported-jobs.json
data/run-state.json
reports/*.md
```

Other candidate files, temporary JSON/report writes, `.env` files, credentials, keys, tokens, and local Claude settings remain ignored.

## Git maintenance and history audit

The superseded initial commit containing former local author metadata was checked before deletion:

- In the public repository it was unreachable and was contained by no branch or tag.
- In the private repository an obsolete local `main` branch still pointed to it. Its tree exactly matched the neutral replacement commit, and it was not an ancestor of `claude/job-search-state`.
- The obsolete private `main` branch was deleted.
- Reflogs were expired and `git gc --prune=now` was run in both repositories.
- The old commit object is now absent from both object databases, and full `git fsck` reports no unreachable objects.

The readiness document itself formerly contained an account-specific absolute local path. Before publication, the unpublished local readiness commit and corresponding private merge were rewritten to contain only `<LOCAL_WORKSPACE>` paths, then their superseded objects were pruned. Final scans cover every blob reachable from every branch and tag, not only checked-out files.

## Final remote configuration

Public repository:

```text
origin  https://github.com/moguzgco/job-search-agent.git (fetch)
origin  https://github.com/moguzgco/job-search-agent.git (push)
```

Private repository:

```text
origin    https://github.com/moguzgco/my-job-search.git (fetch)
origin    https://github.com/moguzgco/my-job-search.git (push)
upstream  https://github.com/moguzgco/job-search-agent.git (fetch)
upstream  DISABLED (push)
```

The private repository sets `remote.pushDefault=origin`. An unqualified push therefore targets the private repository, while an explicit `git push upstream` fails because its push URL is `DISABLED`.

## Placeholder private configuration

`candidate/profile.md` requests verified location, work authorization, sponsorship, experience, technologies, domains, education, and evidence notes. It contains no inferred or demonstration qualifications.

`candidate/preferences.md` requests target roles, seniority, technologies, geography, remote evidence, relocation, authorization, contracts, sources, employer constraints, posting age, exclusions, and maximum results.

`candidate/config.md` remains safe by default:

- `manual_test` mode with a unique test label;
- conservative discovery caps;
- `report_only` delivery;
- no recipient and no connector;
- `claude/job-search-state` persistence;
- no automatic recovery of unresolved runs or uncertain sends;
- no force pushes or public-upstream pushes.

The two JSON files contain empty version-1 collections. `reports/README.md` retains the directory without pretending that a real report exists.

## Verified cloud-state protocol

The remote private branch is the sole durable source of truth. The workflow does not depend on a cloud filesystem surviving between executions.

### 1. Retrieve authoritative state

Each run must use a fresh checkout of the private repository and then:

1. Confirm that the current branch is `claude/job-search-state`.
2. Fetch `origin/claude/job-search-state` and fast-forward only to that exact remote head.
3. Stop if the worktree is dirty, the branch diverged, or synchronization fails.
4. Read `data/run-state.json` and `data/reported-jobs.json` from that committed state.

No earlier cloud-local file or unpushed commit is treated as durable.

### 2. Detect unresolved work

Before creating a run, stop if any ledger entry is `started`, `report_written`, `send_started`, `sent`, or `delivery_uncertain`.

- A person may mark `started` or `report_written` as `interrupted` or `failed` only after proving the earlier execution has stopped and no send began.
- `send_started`, `sent`, and `delivery_uncertain` require provider-side reconciliation by run ID before repair or retry.
- A run is never reclaimed automatically merely because it is old.

### 3. Acquire the optimistic Git run lock

Create a unique run ID, add a `started` ledger entry, commit only that change, and push it before discovery. Two runs starting from one remote head race on this push: the winner advances the branch; the loser receives a non-fast-forward rejection and stops before discovery or delivery. The loser must not merge, rebase, retry the lock, or force-push.

### 4. Persist reports and history

- `report_only`: commit the completed report, merged successful-history additions, and `completed` state together; push once.
- `prepare_email`: do the same and include the email-friendly body; do not send.
- `send_email`: commit and push `report_written`, then commit and push `send_started`. Only then invoke the authorized connector once. Following an unambiguous send result, commit the history additions and `completed` state and push.

History is merged by stable job ID without deleting existing entries. A completed run is durable only when its final push is confirmed.

### 5. Handle failed pushes and uncertain sends

- Lock push failure: stop before discovery or any external action.
- Report-only or prepared-email completion push failure: the remote remains `started`; the overall run is failed, not successful.
- `report_written` or `send_started` push failure: do not call the connector.
- Final push failure after a connector success: the remote remains `send_started`; the workflow is not marked successful and no automatic resend occurs. Reconcile the provider by run ID.
- Timeout or ambiguous connector response: persist `delivery_uncertain` if possible, leave successful history unchanged, and never retry automatically.
- Any non-fast-forward update or merge conflict: stop the Routine, reconcile locally, validate both JSON files, and push normally. Force-push is prohibited.

## Pre-publication security audit

### Checks passed

- Public tracked files contain no root candidate documents, private configuration, operational job history, or private reports.
- Public fixtures are confined to `fixtures/demo/` and identify candidate data as synthetic.
- No resume exists in either tracked-file list.
- Private candidate files contain placeholders, not demonstration facts; 50 TODO markers remain.
- Private history and run ledger are empty valid JSON.
- Private delivery is `report_only`; recipient and connector are unset.
- Public has no remote. Private has no `origin`; `upstream` is fetch-only with push disabled.
- Both repositories have independent `.git` directories and no object alternates.
- The old metadata-bearing object has been pruned from both object stores.
- Reachable-history scans found no account-specific local path, obvious token, private-key block, or non-neutral commit identity.
- The public repository still contains exactly the three approved instruction modules and no application code or additional infrastructure.
- Both worktrees are clean after the local audit commits.

### Remaining exposure risks and controls

| Risk | Control before publication or operation |
|---|---|
| Personal content added to a public template or fixture | Review all staged content and reachable history; never use `git add -f` in public |
| Private repository created with public visibility | Query `visibility` and `isFork` immediately after creation and before its first push |
| Private commits pushed to public | Keep `upstream` push URL `DISABLED`; set `remote.pushDefault=origin`; use explicit remote names |
| Secrets committed even in private | Use account-managed authorization; keep tokens and credentials out of files and Git |
| Candidate data exposed in cloud transcripts | Review Claude account/workspace sharing and retention before connection |
| Duplicate mail after an uncertain result | Persist `send_started` before sending and reconcile the provider by run ID; never auto-retry |
| Concurrent runs overwrite state | Require the first `started` push as the lock and stop on every non-fast-forward rejection |
| Cloud-local work mistaken for persistence | Treat only confirmed commits on the private remote branch as durable |

## GitHub publication verification

GitHub CLI 2.101.0 was installed through Homebrew. Interactive authentication completed for the user-confirmed account `moguzgco`; no credential value was written to this repository.

Both repositories were created without `--push`. Visibility and fork status were queried before either first push:

| Repository | Visibility | Fork | Default branch | Initial published commit |
|---|---|---|---|---|
| `moguzgco/job-search-agent` | `PUBLIC` | `false` | `main` | `70b08087ab851be8009a5f89bbc62c4a28a8535e` |
| `moguzgco/my-job-search` | `PRIVATE` | `false` | `claude/job-search-state` | `8fe8e868c6e3d165df17b4a4aab857a7f616f423` |

Immediately after each first push, `git ls-remote` and the GitHub branches API returned the same commit as the local branch. Each repository had only its intended branch. The public security gate was repeated immediately before creation and found:

- exactly 25 expected tracked files;
- no root candidate, data, report, credential, environment, or resume paths;
- no non-neutral reachable commit metadata;
- no account-specific path, obvious credential, token, or private-key pattern in reachable history;
- a clean worktree and valid object graph.

This publication record is necessarily committed after the two initial branch tips listed above. Its enclosing commit cannot self-record its own hash; final post-record remote tips are verified and reported at handoff.

## Remaining approval gates

1. User reviews the publication results and separately approves Claude Cloud configuration.
2. Private profile, preferences, and configuration TODOs are completed with verified information before cloud testing.
3. Claude Cloud connection, manual cloud runs, email authorization, and scheduling remain separately gated.

No Claude Cloud, Routine, schedule, or email action is authorized by this publication record.
