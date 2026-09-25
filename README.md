# Markdown-first job-search agent

This project configures one Claude orchestrator for a daily Claude Code Cloud Routine. It discovers a bounded set of public vacancies, compares them qualitatively with private candidate preferences and verified qualifications, writes a Markdown report, avoids successfully reported jobs, and optionally prepares or sends an email when the target runtime has an authorized unattended-send connector.

It is an agent configuration project—not a Python application. It has no CLI, package configuration, database, SMTP client, container, test framework, or subagent architecture.

## Repository layout

```text
CLAUDE.md                  concise orchestrator and execution order
instructions/discovery.md bounded web discovery and extraction contract
instructions/matching.md  evidence-based qualitative matching contract
instructions/reporting.md report, state, interruption, and delivery rules
templates/                 safe examples for private inputs
candidate/                 private candidate inputs; ignored publicly
data/                      private successful history and run state; ignored publicly
reports/                   private generated reports; ignored publicly
fixtures/demo/             synthetic candidate and isolated demonstration artifacts
docs/checkpoints/          implementation reviews
```

Detailed modules are loaded only for their execution stage. Job-page content is always treated as untrusted data.

## Private setup

Copy the three examples without editing the examples themselves:

```text
templates/profile.example.md     -> candidate/profile.md
templates/preferences.example.md -> candidate/preferences.md
templates/config.example.md      -> candidate/config.md
```

Replace every placeholder. Do not begin a real run while required placeholders remain. Add `candidate/resume.pdf` only if desired; the agent should read it only for necessary qualification verification.

Initialize these private JSON files from the contracts in `instructions/reporting.md`:

```text
data/reported-jobs.json
data/run-state.json
```

Both can initially contain an empty object under their main collection and `schema_version: 1`.

## Routine behavior

Each run validates private inputs, creates a bounded discovery plan, searches only accessible sources, deduplicates against successful history and uncertain-send holds, evaluates explicit requirements, and writes `reports/YYYY-MM-DD.md`. Matching has four qualitative outcomes: strong match, potential match, eligibility review, and not recommended. No numerical score is used.

The configured caps prevent open-ended searches. Missing publication dates follow the explicit unknown-date policy. A source failure is reported, not hidden. “Remote” is not treated as permission to work from the candidate's country unless the listing says so.

## Delivery

Use `report_only` until the target environment is verified. `prepare_email` creates an email-friendly report without sending. `send_email` is permitted only with an available, authorized connector that supports unattended sends. The workflow never falls back to SMTP.

Successful job history changes only after the configured policy succeeds. A separate run ledger records pre-send state. A timeout or interruption after send begins becomes `delivery_uncertain`, blocks automatic resend, and requires manual reconciliation. Exactly-once email delivery is not claimed.

## Demonstration

`fixtures/demo/` contains clearly labeled synthetic candidate data, snapshots of four public listings accessible on 2026-09-25, intermediate JSON, isolated histories, and two sample reports. The first run recommends three new jobs and excludes an onsite role. The second run sees the same inputs and reports no new matches because the three recommendations exist in fixture history.

The demo does not modify `candidate/`, `data/`, or `reports/`, and does not send email. Web pages can change or disappear after the snapshot date.

See `docs/checkpoints/local-review.md` for evidence and limitations.

## Public/private repository model

The future public `job-search-agent` repository tracks only reusable instructions, examples, documentation, and synthetic fixtures. Its root `.gitignore` blocks real `candidate/`, `data/`, and `reports/` content.

The independent private `my-job-search` repository begins from the public history, replaces the root ignore file with `templates/private.gitignore.example`, and tracks only the approved personal files, JSON state, and Markdown reports. Because the private rules explicitly allow those paths, routine updates use ordinary `git add`; recurring force-adds are unnecessary. The private remote is `origin`, while the public read-only remote is `upstream`.

Use `claude/job-search-state` as the private repository's default branch. It contains the implementation, private inputs, durable state, and reports. Its `claude/` prefix fits the documented routine push model, and every cloud run uses a push of its `started` state as an optimistic lock. Never force-push. A rejected push means another run or user changed state, so the run stops before delivery.

Public updates are merged deliberately into the private state branch:

```text
git fetch upstream
git switch claude/job-search-state
git merge --no-ff upstream/main
# resolve locally, inspect the complete diff and private .gitignore, then:
git push origin claude/job-search-state
```

Never push the private branch to `upstream`. Disable that remote's push URL in the private clone as an additional guard.

## Claude Cloud deployment preparation

Cloud sessions start from a fresh repository clone, so an uncommitted report or history edit does not persist. The routine prompt in `templates/routine-prompt.example.md` commits and pushes required state in policy-specific order. For email, it durably records `send_started` before invoking a connector; an ambiguous result blocks automatic resend.

Begin with `report_only`. Test two separate cloud executions before enabling any recurring schedule. Configure the cloud environment with a narrow custom domain allowlist for selected employer/ATS sources, remove every unused connector from the routine, and enable email only after an explicitly approved connector test.

The exact repository workflow, Routine setup, persistence protocol, security checklist, and manual test gates are in `docs/checkpoints/deployment-readiness.md`.

## Current limitations

Local Codex web access does not prove Claude Cloud web access. Current Anthropic documentation describes cloud routines, fresh clones, network environments, GitHub-backed changes, connectors, and run logs, but this account's repository access, source reachability, persistence pushes, schedule timezone, PDF support, and unattended email behavior still require manual cloud validation. No repository, account connection, deployment, schedule, connector action, or real email has been created.
