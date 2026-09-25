# Agent configuration

> Private operational configuration. Keep values small until the routine is verified.

## Routine

- Routine name: `<stable-name>`
- Report timezone: `<IANA timezone, for example Europe/Istanbul>`
- Report language: `<language>`
- Run mode: `production` # use `manual_test` only during approved cloud tests
- Manual test label: `none` # set a unique label such as `cloud-1` in manual_test mode
- Allow more than one production completion per report date: `false`

## Discovery limits

- Maximum search queries: `<positive integer>`
- Maximum pages per source: `<positive integer>`
- Maximum jobs evaluated per run: `<positive integer>`
- Stop when maximum jobs evaluated is reached: `true`
- Repeat identical source/query pairs: `false`

## Delivery

- Policy: `report_only` # `report_only`, `prepare_email`, or `send_email`
- Recipient: `<required only for send_email; keep private>`
- Subject template: `Job search report — YYYY-MM-DD`
- Authorized connector: `<connector name or none>`
- Allow unattended send: `false`
- Allow real email during local testing: `false`

## State

- Successful history: `data/reported-jobs.json`
- Run ledger: `data/run-state.json`
- Report directory: `reports/`
- Resume uncertain sends automatically: `false`

## Cloud Git persistence

- Enabled: `true`
- Private origin remote: `origin`
- Persistent branch: `claude/job-search-state`
- Require clean worktree: `true`
- Require fast-forward-only synchronization: `true`
- Push a run lock before discovery: `true`
- Allow force push: `false`
- Public upstream remote: `upstream`
- Allow routine to push upstream: `false`
