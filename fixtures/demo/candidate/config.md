# Synthetic agent configuration — demonstration only

## Routine

- Routine name: `fixture-daily-job-search`
- Report timezone: `Europe/Istanbul`
- Report language: English

## Discovery limits

- Maximum search queries: 4
- Maximum pages per source: 2
- Maximum jobs evaluated per run: 4
- Stop when maximum jobs evaluated is reached: true
- Repeat identical source/query pairs: false

## Delivery

- Policy: `report_only`
- Recipient: none
- Authorized connector: none
- Allow unattended send: false
- Allow real email during local testing: false

## Isolated state

- Successful history input: `fixtures/demo/data/reported-jobs.before.json`
- Run ledger output: `fixtures/demo/data/run-state.after-run-2.json`
- Report directory: `fixtures/demo/reports/`
- Resume uncertain sends automatically: false
