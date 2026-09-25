# Reporting and delivery instructions

Load this module only when producing the report, preparing delivery, or updating persistent state.

## Report

Write `reports/YYYY-MM-DD.md` using the configured timezone. If that path already belongs to a different completed run, preserve it and use a documented suffix rather than overwriting it. Include:

- run ID, generated time, delivery policy, and candidate input paths;
- discovery/evaluation/recommendation counts and configured bounds;
- source failures and skipped inaccessible sources;
- sections for strong matches, potential matches, and eligibility-review jobs;
- for each recommendation: title, company, location, original URL, source, publication date or “not available,” explicit remote evidence, matching reasons, significant gaps, and unresolved eligibility requirements;
- exclusions summary;
- an explicit no-new-matches result when applicable;
- an email-friendly plain-text/Markdown section.

Do not expose secrets or unnecessary candidate details in a report.

## Persistent files

`data/reported-jobs.json` is successful recommendation history. Preserve every existing entry. Add a job only after the configured report and delivery policy succeeds.

```json
{
  "schema_version": 1,
  "reported_jobs": {
    "stable-job-id": {
      "original_url": "https://...",
      "normalized_url": "https://...",
      "first_reported_at": "ISO-8601",
      "run_id": "string",
      "report_path": "reports/YYYY-MM-DD.md"
    }
  }
}
```

`data/run-state.json` is a separate operational ledger. It may be updated before success so interrupted or uncertain sends can be detected without falsely marking jobs as reported.

```json
{
  "schema_version": 1,
  "runs": {
    "run-key": {
      "run_id": "string",
      "status": "started|report_written|delivery_prepared|send_started|sent|completed|failed|interrupted|delivery_uncertain",
      "delivery_policy": "report_only|prepare_email|send_email",
      "job_ids": ["string"],
      "report_path": "string|null",
      "last_updated_at": "ISO-8601",
      "detail": "string|null"
    }
  }
}
```

Use a stable run key derived from configured routine name and report date. A retry resumes the existing nonterminal run instead of silently creating another send attempt. Use a unique run ID for auditability.

## Delivery policies

- `report_only`: success after the report is durably written.
- `prepare_email`: success after the report and email-friendly body are durably written; no message is sent.
- `send_email`: inspect the actual runtime for an authorized unattended-send connector. If unavailable, record failure/missing integration, leave successful history unchanged, and do not implement SMTP.

For `send_email`, write `send_started` before invoking the connector. Change it to `sent` only when the connector gives an unambiguous success response, then update successful history and mark the run `completed`.

If the call times out, the result is ambiguous, or execution stops after `send_started`, set or interpret the status as `delivery_uncertain`. Do not retry automatically and do not add the jobs to successful history. A person must reconcile the provider's sent items and explicitly mark the run sent or safe to retry. This reduces duplicate risk; it does not provide exactly-once delivery.

If interrupted before a send begins, retain `interrupted` state and resume from the last verified artifact. Never send during local testing without explicit permission.

When a policy succeeds, merge new entries into `reported-jobs.json` without replacing existing keys. When a policy fails or remains uncertain, leave it unchanged.

## Cloud Git persistence

Cloud files are not durable merely because they were written in a run. When `candidate/config.md` selects Git persistence, the configured private branch is the source of truth. It must use the `claude/` prefix and be the private repository's default branch; the recommended name is `claude/job-search-state`.

At run start, fast-forward that branch from `origin`, verify a clean worktree, and push a commit containing the new `started` ledger entry before discovery. Treat that first push as an optimistic lock. If it is rejected or the branch has diverged, stop without discovery or delivery. Never force-push and never resolve a concurrent update automatically.

Persistence order depends on delivery policy:

- `report_only`: commit and push the report, completed run state, and successful-history additions together.
- `prepare_email`: commit and push the report/email body, completed run state, and successful-history additions together.
- `send_email`: commit and push `report_written`; then commit and push `send_started`; only then call the connector. After an unambiguous send result, commit and push successful history and `completed`. If the final push fails, the remote `send_started` state deliberately blocks automatic resend.

A commit that exists only in the cloud checkout is not persisted. A run is successful only after the required push is confirmed. Leave failed/rejected commits for log inspection; the next run must begin again from the remote branch.
