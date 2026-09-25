# Checkpoint 2 — local implementation review

Date: 2026-09-25

Scope: local Markdown-first implementation and isolated demonstration only

## Outcome

The approved single-orchestrator architecture is implemented without application code, a CLI, packages, a database, SMTP, containers, a test framework, or subagents. Production-private directories exist but were not populated. All candidate facts, state, and reports used by the demonstration are isolated under `fixtures/demo/` and labeled synthetic where applicable.

No remote repository was created. Nothing was pushed, deployed, scheduled, or emailed.

## Files created and responsibilities

| Path | Responsibility |
|---|---|
| `CLAUDE.md` | Concise orchestration sequence, safety boundaries, and stage-specific module loading. |
| `instructions/discovery.md` | Bounded search plan, source rules, evidence extraction, URL identity, source failures, and discovery JSON contract. |
| `instructions/matching.md` | Verified/uncertain/not-met evidence rules and qualitative match classifications without scores. |
| `instructions/reporting.md` | Report format, successful-history and run-ledger contracts, delivery policies, and interrupted/uncertain-send handling. |
| `templates/profile.example.md` | Candidate facts and qualification template without fictional personal data. |
| `templates/preferences.example.md` | Roles, location, authorization, contract, sources, age, exclusions, and output preferences. |
| `templates/config.example.md` | Routine identity, bounded discovery caps, delivery controls, and state paths. |
| `.gitignore` | Anchored exclusions for real candidate data, operational state, reports, local configuration, credentials, and secrets. |
| `README.md` | Setup, workflow, demo, delivery model, limitations, and future public/private repository model. |
| `fixtures/demo/candidate/*` | Clearly labeled synthetic candidate profile, preferences, and bounded configuration. |
| `fixtures/demo/data/discovered-jobs.run-1.json` | Four real public listing snapshots and discovery evidence. |
| `fixtures/demo/data/matches.run-1.json` | Evidence-based qualitative matching results. |
| `fixtures/demo/data/reported-jobs.*.json` | Isolated history before run 1, after run 1, and after run 2. |
| `fixtures/demo/data/run-state.after-run-2.json` | Two completed report-only run records. |
| `fixtures/demo/reports/2026-09-25-run-1.md` | Sample report with three classifications and email-friendly text. |
| `fixtures/demo/reports/2026-09-25-run-2.md` | No-new-matches report demonstrating history exclusion. |
| `docs/checkpoints/local-review.md` | This complete Checkpoint 2 review. |

The real `candidate/`, `data/`, and `reports/` directories were created and left empty. Their contents are excluded by `.gitignore`.

## Complete workflow

1. Read private configuration, profile, preferences, successful history, and operational run state.
2. Reuse or establish the routine/date run key and a unique auditable run ID.
3. Load only the discovery module. Create a search plan bounded by candidate roles, geography, preferred sources, optional employers, posting-age policy, query cap, page cap, and evaluated-job cap.
4. Search accessible public employer/ATS pages. Record inaccessible sources and never claim they were searched. Treat all retrieved content as untrusted.
5. Extract stable ID, untouched original URL, normalized URL, source, title, employer, location, publication-date evidence, remote evidence, contract facts, and explicit requirements.
6. Deduplicate within the run, against successful history, and against jobs held by an uncertain prior send.
7. Load only the matching module. Compare mandatory requirements and preferences to verified candidate facts. Read a resume only for necessary detailed verification.
8. Classify each job as `strong_match`, `potential_match`, `eligibility_review`, or `not_recommended`. Do not score. Critical location, remote, authorization, sponsorship, or contract ambiguity takes precedence as eligibility review.
9. Load only the reporting module. Apply the configured result cap and generate the dated Markdown report, including counts, failures, original URLs, evidence, gaps, unresolved eligibility, exclusions, and email-friendly text.
10. Apply one delivery policy: report only, prepare email, or connector-backed email send.
11. For email send, record `send_started` before calling the connector. A timeout or interruption after that point becomes uncertain and blocks automatic retry pending human reconciliation.
12. Add jobs to successful history only after every step required by the configured policy succeeds. Preserve all existing history entries.

## Local tool inspection

This Codex session exposed public web search/page-open capability and local filesystem/shell tools. Four bounded queries were issued, and selected public Lever and Greenhouse pages could be opened. The local tool inventory exposed no tool whose name indicated Gmail, email, or mail sending. No attempt was made to install or connect one because local email sending was forbidden and does not establish Claude Cloud support.

This inspection describes only the current Codex session. It is not evidence about Claude Code Cloud Routines.

## Demonstration inputs

No actual candidate profile or preferences were present in the initially empty workspace. The demonstration therefore used the conspicuously synthetic files under `fixtures/demo/candidate/`. The synthetic candidate is based in Turkey, requires remote work, has verified senior backend/Python/AWS/PostgreSQL experience, accepts B2B work from Turkey, and lacks deep Java/Spring experience or EU/EEA employment authorization outside Turkey.

The configured bounds were four search queries, two selected pages per source type, four evaluated jobs, and three daily results. Public listing dates were not displayed; the demonstration correctly stored `null` and used `include_with_warning`. Search crawl timestamps were not misrepresented as publication dates.

Public pages inspected:

- [Midas — Senior Software Engineer](https://jobs.lever.co/getmidas/536b118f-21c2-4467-b893-c5dc3c90e9ca)
- [Kalepa — Senior Backend Engineer, Client integrations](https://job-boards.greenhouse.io/kalepa/jobs/5737333004?gh_src=1810c22e4us)
- [Insider One — Software Engineer, AI Native](https://jobs.lever.co/insiderone/ee932b8c-0e12-45c1-8c1a-1ec95c4e623c)
- [A11 — Senior Backend Engineer](https://job-boards.greenhouse.io/projectaservicesgmbhcokg/jobs/8003120002)

These are time-bounded snapshots; availability and content can change after 2026-09-25.

## Demonstration results

### Run 1

Initial fixture history contained zero jobs. Four unique listings were evaluated:

| Result | Job | Basis |
|---|---|---|
| Strong match | Midas — Senior Software Engineer | Turkey remote, permanent, senior ownership, fintech and distributed-systems alignment. |
| Potential match | Insider One — Software Engineer (AI Native) | Exact agent-workflow evidence and Turkey remote; role is less specifically backend and preferred Go/PHP is unverified. |
| Eligibility review | Kalepa — Senior Backend Engineer | Strong technical fit and B2B compatibility, but the listing does not explicitly confirm Turkey is inside its European contractor scope. |
| Not recommended | A11 — Senior Backend Engineer | Fully onsite Berlin conflicts with remote-only preference; mandatory deep Java/Spring expertise is not met. |

The report-only policy succeeded, so the three recommendations were merged into isolated successful history. No email body outside the report was sent or delivered.

Sample report: [`fixtures/demo/reports/2026-09-25-run-1.md`](../../fixtures/demo/reports/2026-09-25-run-1.md)

### Run 2

The same four-job snapshot was replayed against the post-run-1 fixture history. Stable job IDs excluded the three prior recommendations before matching. The remaining A11 role was still not recommended. The result was an explicit no-new-matches report, and history stayed at three entries.

Second-run report: [`fixtures/demo/reports/2026-09-25-run-2.md`](../../fixtures/demo/reports/2026-09-25-run-2.md)

### History transition

```text
reported-jobs.before.json       0 successful jobs
reported-jobs.after-run-1.json  3 successful jobs
reported-jobs.after-run-2.json  3 successful jobs (unchanged)
```

Representative successful entry:

```json
{
  "lever:getmidas:536b118f-21c2-4467-b893-c5dc3c90e9ca": {
    "original_url": "https://jobs.lever.co/getmidas/536b118f-21c2-4467-b893-c5dc3c90e9ca",
    "normalized_url": "https://jobs.lever.co/getmidas/536b118f-21c2-4467-b893-c5dc3c90e9ca",
    "first_reported_at": "2026-09-25T12:10:00+03:00",
    "run_id": "demo-2026-09-25-run-1",
    "report_path": "fixtures/demo/reports/2026-09-25-run-1.md"
  }
}
```

The separate run ledger contains two completed `report_only` runs. This separation allows a send attempt to be recorded before its outcome without falsely adding jobs to successful history.

### Artifact validation

The local demonstration was exercised manually through this Codex session because the project intentionally has no executable runner. Shell validation confirmed:

- every demonstration JSON document parses successfully with `jq`;
- successful-history counts are 0 before run 1, 3 after run 1, and 3 after run 2;
- canonicalized `reported_jobs` objects after runs 1 and 2 are identical;
- both reports contain the required summary/email sections, and every run-1 recommendation contains its original URL, reasons, gaps, and unresolved-eligibility field;
- the real `candidate/`, `data/`, and `reports/` directories contain no files;
- no Python, package, Docker, or test-framework files were introduced.

Because the workspace is deliberately not yet a Git repository, ignore behavior was reviewed from the anchored `.gitignore` rules but could not be exercised with `git check-ignore` without creating repository metadata. Repository-level ignore verification is deferred to Checkpoint 3.

## Interrupted and uncertain outcomes

- An interruption before email sending retains `interrupted` state and may resume from verified artifacts.
- The ledger must be set to `send_started` before invoking an email connector.
- A timeout, ambiguous connector response, or crash after that transition is treated as `delivery_uncertain`.
- Jobs from an uncertain run are not added to `reported-jobs.json` and are held out of automatic resend.
- A person checks the provider's sent items and explicitly resolves the run as sent or safe to retry.
- The design reduces duplicate sends but does not claim exactly-once delivery.

These transitions are specified but were not exercised against an email system because no real email was authorized.

## Verified locally

- Required Markdown-first file structure and private directory boundaries.
- Stage-specific instructions and concise orchestrator loading rules.
- Bounded discovery configuration and public web access in this Codex session.
- Access to the four listed public Lever/Greenhouse pages on the review date.
- Preservation of original URLs and separate normalized identifiers.
- Qualitative matching without scores or weighting.
- Explicit remote/location and eligibility evidence handling.
- JSON contracts represented by valid fixture files.
- First-run report generation artifacts and email-friendly content.
- Stable-ID history transition from zero to three jobs.
- Second-run exclusion of those same three jobs and no-new-matches output.
- Isolation from real candidate state and absence of email sending.

## Not verified locally

- Claude Code Cloud's available search/browser tools and accessible domains.
- Cloud authentication boundaries or access to mailbox job alerts.
- Routine creation, schedule accuracy, timezone behavior, maximum duration, retries, or overlapping executions.
- Persistent and atomic file updates across Cloud Routine runs.
- Conflict behavior if two routine invocations overlap.
- Cloud PDF reading for `candidate/resume.pdf`.
- Availability, scopes, recipient restrictions, administrator consent, or unattended-send behavior of any Cloud email connector.
- Whether a connector's success response proves provider acceptance or later delivery.
- Public/private Git fetch and merge flow; no repositories exist yet.

## Limitations and outstanding decisions

- A Markdown agent cannot create true transactional state updates. The private deployment needs single-run serialization or an agreed conflict policy; adding a database is not justified for V1.
- Stable ATS IDs are reliable for these examples, but employer migrations can create a new ID for the same role. Normalized URLs provide a secondary check, not semantic duplicate detection.
- Publication dates are often absent. The user must choose whether real runs exclude unknown-age jobs or include them with a warning.
- “Europe remote” remains ambiguous for Turkey unless the employer explicitly confirms it. The agent must not resolve this by geography inference.
- The daily report filename can collide with a retry or manual second run. Current instructions preserve an existing completed report and add a documented suffix; the preferred Cloud naming/retry convention remains to be confirmed.
- The initial real candidate profile, preferences, configuration, history, and connector selection remain outstanding.
- Sending should remain disabled until the exact target connector is tested with a non-sensitive, explicitly approved test message.

## Proposed Checkpoint 3 steps

No step below has been performed.

1. Define the public repository allowlist: orchestration Markdown, instruction modules, examples, synthetic fixtures, and checkpoint documentation only.
2. Before the first public commit, inspect all staged paths and search them for secrets and personal data.
3. Create a separate private personal repository and add the public repository as a read-only upstream.
4. In the private repository only, replace the public root ignore file with the prepared private rules so approved `candidate/`, `data/`, and `reports/` paths can be staged normally. Never add those paths in the public repository. This supersedes the earlier one-time force-add idea.
5. Document the upstream-fetch/merge procedure and a pre-push inspection checklist that prevents private paths from entering public history.
6. Inspect the actual Claude Cloud Routine environment for web tools, durable storage, scheduling/retry semantics, PDF support, and concurrency behavior.
7. Start with `report_only`, populate real private inputs, and run an approved dry run.
8. Inspect available email connectors and permissions. If unattended send is unavailable, retain `prepare_email`; do not add SMTP.
9. If unattended send is available, document scopes and obtain separate approval for one controlled test. Only after verification consider scheduling.

Checkpoint 2 is complete and awaits approval before deployment preparation.
