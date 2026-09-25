# Checkpoint 3 — deployment readiness

Date: 2026-09-25

Status: preparation only; no repository, account connection, routine, schedule, connector action, deployment, or email has been created

## Deployment decision

Use two independent GitHub repositories with shared Git history:

- Public `job-search-agent`: reusable Markdown instructions, examples, documentation, and safe synthetic fixtures only.
- Private `my-job-search`: the public implementation plus real candidate files, private configuration, successful history, run state, and generated reports.

The private repository uses `claude/job-search-state` as its default branch. Each cloud run starts from that durable branch and pushes back to it. This avoids assuming cloud filesystem persistence and stays within Claude Code's documented support for `claude/`-prefixed branches.

Anthropic currently documents Routines as a research preview. The instructions below reflect the official documentation available on 2026-09-25 and must be checked against the actual UI during the manual test.

## Files prepared in this checkpoint

- Public `.gitignore`: continues to exclude all real `candidate/`, `data/`, and `reports/` content.
- `templates/private.gitignore.example`: private-root rules that track only approved personal inputs, durable JSON, and Markdown reports while excluding credentials and temporary files.
- `templates/routine-prompt.example.md`: exact single-orchestrator prompt, Git run lock, policy-specific persistence order, and uncertain-send behavior.
- `templates/config.example.md`: now includes Git persistence settings.
- `instructions/reporting.md`: now defines cloud Git durability and push ordering.
- `CLAUDE.md` and `README.md`: now reference the run lock, persistence result, two-repository model, and this guide.

No fourth instruction module, helper program, package, or infrastructure component was added.

## 1. Prepare the public repository after approval

These are future steps; do not run them until remote actions are approved.

1. Initialize local Git with `main` as the branch.
2. Stage only this allowlist:
   - `.gitignore`, `CLAUDE.md`, and `README.md`;
   - `instructions/`;
   - `templates/`, including the private ignore and Routine prompt examples;
   - `fixtures/demo/`;
   - `docs/checkpoints/`.
3. Confirm `candidate/`, `data/`, and `reports/` contribute no staged files.
4. Inspect every staged filename and diff for names, email addresses, credentials, tokens, real resume content, and non-synthetic history.
5. Commit locally only after the inspection.
6. Create public GitHub repository `job-search-agent` and push only after separate approval.

Suggested verification before the first public commit:

```sh
git status --short
git diff --cached --name-only
git diff --cached
git check-ignore -v candidate/profile.md data/reported-jobs.json reports/2099-01-01.md
```

Do not use `git add -f` in the public repository. Empty private directories need not appear in public Git.

## 2. Prepare the independent private repository after approval

Start from the public history so upstream merges remain ordinary Git merges:

```sh
git clone <PUBLIC_REPOSITORY_URL> my-job-search
cd my-job-search
git remote rename origin upstream
git remote set-url --push upstream DISABLED
git switch -c claude/job-search-state
cp templates/private.gitignore.example .gitignore
git remote add origin <PRIVATE_REPOSITORY_URL>
```

Populate only the private paths described below, then stage them normally. The private ignore rules do not require force-adds:

```sh
git add .gitignore candidate data reports
git commit -m "chore: initialize private job-search state"
git push -u origin claude/job-search-state
```

In GitHub, make `claude/job-search-state` the private repository's default branch. Do not enable a rule that blocks this branch from being updated by the connected Claude identity. Do not grant the Routine access to the public repository; it needs only `my-job-search`.

Before any cloud test, verify:

```sh
git remote -v
git branch --show-current
git ls-files candidate data reports
git check-ignore -v .env credentials/example.token data/example.tmp reports/example.tmp
```

Expected tracked private paths are only:

```text
candidate/profile.md
candidate/preferences.md
candidate/config.md
candidate/resume.pdf        optional
data/reported-jobs.json
data/run-state.json
reports/*.md                generated over time
```

The private repository must remain private. Git privacy is an access control, not a substitute for excluding secrets: OAuth tokens, connector credentials, API keys, cookies, and passwords must never be committed.

## 3. Merge public updates into the private repository

Perform upstream merges locally, not inside the daily Routine:

```sh
git fetch upstream
git switch claude/job-search-state
git status --short
git merge --no-ff upstream/main
git diff --check
git push origin claude/job-search-state
```

Rules:

- Begin with a clean worktree and fetch both remotes if necessary.
- Review upstream changes before merging, especially `CLAUDE.md`, `instructions/`, templates, and `.gitignore`.
- Keep the private `.gitignore` when it conflicts, then manually incorporate any new public secret exclusions.
- Never resolve a conflict by deleting private candidate/state/report files.
- Revalidate JSON after conflict resolution.
- Push only to private `origin`; the disabled upstream push URL is a guard against leaking private commits.
- Pause the Routine during an upstream merge. Resume only after `origin/claude/job-search-state` contains the tested merge.

## 4. Populate real private configuration

Do not copy synthetic demonstration facts into `candidate/`.

### `candidate/profile.md`

Copy `templates/profile.example.md` and replace every placeholder with verified facts. Include current location, actual work authorization, sponsorship constraints, verified experience, technologies, and any facts that may affect eligibility. Minimize sensitive identity information; a full address, government identifiers, and compensation records are unnecessary.

### `candidate/preferences.md`

Copy `templates/preferences.example.md`. Define target and adjacent roles, seniority, remote requirement, acceptable hiring geographies, relocation position, authorization constraints, accepted contracts, technologies, employer/source preferences, exclusions, posting-age policy, and maximum daily results.

### `candidate/resume.pdf`

Optional. Add the real PDF only to the private repository. Keep the Markdown profile authoritative for routine matching; the PDF is read only when detailed verification is necessary.

### `candidate/config.md`

Copy `templates/config.example.md` and set:

- a stable routine name;
- `Europe/Istanbul` or the chosen IANA report timezone;
- `manual_test` plus a unique committed label for each manual cloud run, then `production` before scheduling;
- small discovery caps for the first runs;
- `report_only` initially;
- `claude/job-search-state` as the persistent branch;
- the connector name and exact email recipient only when delivery is approved.

The recipient address may be stored in this private file. Connector tokens and mailbox credentials may not. Leave `Allow real email during local testing` false.

### Initial JSON

Create and validate:

```json
{
  "schema_version": 1,
  "reported_jobs": {}
}
```

in `data/reported-jobs.json`, and:

```json
{
  "schema_version": 1,
  "runs": {}
}
```

in `data/run-state.json`.

## 5. Claude Cloud Routine setup after approval

Official documentation says a Routine packages a prompt, repositories, environment, and connectors; every run is a new cloud session. Repositories are cloned at run start, cloud environments control network access, all connected connectors are initially included in the form unless removed, and run sessions expose logs and changes. See [Automate work with routines](https://code.claude.com/docs/en/routines), [Use Claude Code in the cloud](https://code.claude.com/docs/en/claude-code-on-the-web), and [Configure cloud environments](https://code.claude.com/docs/en/cloud-environments).

### Connect the private GitHub repository

1. Confirm the account plan and organization policy permit Claude Code cloud sessions and Routines.
2. Connect GitHub using either the Claude GitHub App or `/web-setup`. Prefer the GitHub App with repository access limited to private `my-job-search`.
3. Do not grant access to other private repositories or the public upstream unless a demonstrated need appears.
4. Confirm the connected identity can clone and push the private default branch `claude/job-search-state`.
5. Confirm the repository default branch contains the real private files and empty initial state.

Official docs describe both GitHub connection methods and state that each run clones the repository afresh. Whether this account and repository can push the persistent default branch must be validated manually.

### Create a narrow cloud environment

Create a dedicated environment such as `job-search-production` with Custom network access. Begin with only the domains actually needed, for example:

```text
jobs.lever.co
job-boards.greenhouse.io
boards.greenhouse.io
<approved employer career domains>
```

Add domains only after a run log shows a legitimate blocked source. Do not choose Full network access merely for convenience. GitHub uses Claude's separate GitHub proxy, and enabled connector traffic is routed separately, so neither requires adding its service hosts to this list according to the current documentation.

The availability and behavior of a general web-search tool inside this Routine is not proven by the local Codex demo. The first cloud test must determine whether search works directly, needs more allowed domains, or requires discovery to rely on configured employer/ATS pages.

Do not add environment variables or setup scripts for V1. This project has no dependencies to install. Never place mailbox credentials in environment variables.

### Create the Routine without enabling daily execution

1. Open `https://claude.ai/code/routines` and choose **New routine**.
2. Name it clearly, for example `Daily private job search`.
3. Select only private `my-job-search`.
4. Select `job-search-production`.
5. Use the content of `templates/routine-prompt.example.md` as the saved prompt.
6. Select an appropriate model within the available usage budget.
7. Remove every connector from the Routine for the initial `report_only` tests.
8. Do not enable a recurring daily trigger yet.

Current documentation centers Routine creation around a trigger but also provides **Run now** after creation. If the live form requires a trigger, use a one-off time safely in the future, create the Routine, immediately pause it, and use **Run now** for testing. Confirm this UI behavior at test time; do not assume it from local work.

### Minimum permissions

- GitHub: read and write only private `my-job-search`.
- Branch: current/default `claude/job-search-state`; no force pushes.
- Network: Custom allowlist containing only approved discovery domains.
- Connectors: none for initial tests; later include only the selected email connector.
- Environment secrets: none.
- Public repository: no Routine access.

Routines are unattended and do not stop for ordinary approval prompts. Any included connector may expose write tools, so removing unused connectors is mandatory.

### Timezone and daily schedule after tests pass

The report timezone stays explicit in `candidate/config.md`. Official docs say schedule times are entered in the user's local timezone and converted automatically; runs may begin a few minutes late due to a consistent stagger.

Before enabling a daily trigger:

1. Confirm the Claude account/browser displays the intended local zone.
2. Select daily cadence and the approved wall-clock time.
3. Verify the displayed next-run timestamp and compare it with `Europe/Istanbul`.
4. Use only one scheduled trigger. Do not add API or GitHub triggers for V1.
5. Choose a time comfortably separated from manual maintenance windows.
6. Keep the schedule paused until both manual runs and any approved email test pass.

## 6. Exact Git persistence protocol

The cloud checkout is disposable. Only a confirmed push to private `origin/claude/job-search-state` is durable.

### Start-of-run lock

1. Confirm/switch to `claude/job-search-state`.
2. Fetch and fast-forward only from the same remote branch.
3. Stop if the worktree is dirty, the branch diverged, or any prior run remains unresolved (`started`, `report_written`, `send_started`, `sent`, or `delivery_uncertain`). A person must reconcile it first.
4. Write a unique run ID and `started` status to `data/run-state.json`.
5. Commit that one state transition and push it.
6. Treat a non-fast-forward rejection as an overlap lock failure. Stop before web discovery or email.

Two runs that started from the same commit cannot both acquire the lock: the first push advances the branch and the second push is rejected. The second run must not pull/merge and continue automatically.

### Report-only or prepare-email completion

Write the report, merge successful job entries without deleting history, set the run to `completed`, commit the files together, and push. A local commit without a successful push is a failed run and must not be reported as persisted.

### Email completion

1. Generate the report and push `report_written`.
2. Push `send_started` before invoking the email connector.
3. Send once with the run ID in the subject.
4. Only an unambiguous connector success permits successful-history updates.
5. Commit/push history plus `completed` after success.

If the connector response is ambiguous, record and push `delivery_uncertain` when possible. If the session crashes, the already-pushed `send_started` has the same meaning. The next run stops; a person checks the provider's Sent folder using the run ID and marks the state sent or safe to retry. Exactly-once delivery is not claimed.

### Failed commits, pushes, and conflicts

- Commit failure: do not send; inspect the run log and Git identity/configuration.
- Initial lock push rejected: assume overlap or user update; stop without side effects. The remote branch remains authoritative.
- Completion push rejected: do not auto-merge; preserve the session link and reconcile locally.
- Push fails after confirmed email send: remote state remains `send_started`; treat as uncertain and do not resend.
- Stale `started` or `report_written`: do not reclaim automatically. After confirming no earlier session is active and no send began, a person may commit `interrupted` or `failed`, then start a new run.
- Merge conflict: pause the Routine, resolve locally in the private clone, validate JSON, and push the repaired state branch.
- Network/GitHub outage: no push means no durable success. Do not rely on the cloud worktree for the next run.

## 7. Email delivery preparation

Keep `report_only` until discovery, matching, and persistence pass twice. Then test `prepare_email`.

Current official connector information shows that Gmail exposes a `send_message` tool, while Google Workspace documentation says email sends require explicit approval by default and that Team/Enterprise owners control whether members can allow sends without repeated approval. See [Gmail connector](https://claude.com/connectors/gmail) and [Google Workspace connectors](https://support.claude.com/en/articles/10166901-use-google-workspace-connectors).

Unattended Routine sending is therefore a dependency to test, not an assumed capability:

1. Connect a chosen mailbox only after separate approval.
2. Review OAuth/admin scopes and organizational policy.
3. Include only that email connector in the Routine; remove all others.
4. Confirm its send tool is available to the Routine without an interactive prompt.
5. Use an approved recipient—preferably the same mailbox—for the first controlled send.
6. Verify subject run ID, content, Sent-folder appearance, connector response, and final Git state.
7. If unattended sending is blocked, revert to `prepare_email` and document the missing permission. Do not implement SMTP.

Reading LinkedIn job-alert emails is optional and requires separately authorized mailbox read access. It does not permit scraping authenticated LinkedIn pages.

## 8. Manual cloud test plan

Do not enable a recurring schedule until every required gate passes.

### Test 0 — private repository and security

- [ ] Private repository visibility is private.
- [ ] Default branch is `claude/job-search-state`.
- [ ] Public remote push is disabled in the private clone.
- [ ] Only approved candidate/state/report paths are tracked.
- [ ] No placeholders remain in private inputs.
- [ ] No credentials, secrets, cookies, or synthetic candidate data are in real private paths.
- [ ] Delivery policy is `report_only`.

### Test 1 — first manual cloud execution

Use **Run now** with no recurring trigger enabled.

Set `Run mode: manual_test` and `Manual test label: cloud-1` in the private configuration and push it before this run. The expected report suffix is `-cloud-1`; production naming remains untouched.

- [ ] Routine reads `CLAUDE.md` and only the three stage modules.
- [ ] Real `candidate/profile.md`, preferences, and configuration are accessible.
- [ ] Optional resume is not read unless a concrete verification need occurs.
- [ ] Start-of-run lock commit reaches the private state branch.
- [ ] Bounded discovery works within configured query/page/job caps.
- [ ] At least one approved public source is accessible, or failures are reported accurately.
- [ ] Original URLs open to the intended public job pages.
- [ ] Remote, location, authorization, contract, and qualification decisions follow explicit preferences.
- [ ] A report is generated with counts, failures, reasons, gaps, and unresolved eligibility.
- [ ] Report, history, and completed run state are committed and pushed.
- [ ] A fresh local clone sees those files from the private default branch.
- [ ] No email is sent.

### Test 2 — separate cloud execution

Start a new cloud execution so it receives a fresh clone.

Change the committed manual test label to `cloud-2` first. This permits a second audited run on the same calendar date without weakening the production one-completion-per-day guard.

- [ ] It reads run-1 history from Git rather than relying on prior filesystem state.
- [ ] Previously reported stable IDs/normalized URLs are excluded.
- [ ] Existing history entries are preserved byte-for-byte except intentional additions.
- [ ] A no-new-matches report is produced if nothing new qualifies.
- [ ] Second-run state is pushed successfully.
- [ ] There is no duplicate delivery.

### Test 3 — overlap and failure behavior

- [ ] A deliberate second concurrent/manual run cannot acquire the optimistic lock and stops before discovery/delivery.
- [ ] A simulated blocked job domain is shown as a source failure in the transcript/report.
- [ ] A rejected test push is treated as failure, not durable completion.
- [ ] Run logs clearly expose the reason and final persistence status.

### Test 4 — optional email, separately approved

- [ ] `prepare_email` output is correct before connector use.
- [ ] Selected connector and exact send tool are visible in the Routine.
- [ ] Unattended permission policy is confirmed.
- [ ] One approved test email is sent to the approved recipient.
- [ ] Run ID appears in subject and Sent folder.
- [ ] Git shows `send_started` before the send and `completed` afterward.
- [ ] An ambiguous-outcome exercise or documented review confirms no automatic resend.

Only after Tests 0–2 pass—and Test 4 if `send_email` is desired—request approval to add and enable the daily schedule.
Before scheduling, restore `Run mode: production` and `Manual test label: none`, commit, and push that configuration.

## 9. Review logs and failed runs

Open each run from the Routine detail page. Official docs warn that a green run status only means the session started and exited without infrastructure error; it does not prove task success. Review:

- transcript for blocked network requests, inaccessible sources, missing connector tools, or permission failures;
- final discovery and recommendation counts;
- report path and run ID;
- Git commit and push confirmation;
- remote branch contents in GitHub;
- delivery status and connector response;
- absence of unexpected file or connector access.

Keep the schedule paused after any failure. Correct configuration locally or in the Cloud environment, run manually again, and resume only after reviewing the new transcript and remote state.

## 10. Verified capabilities and unresolved dependencies

| Capability | Evidence now | Still requires real cloud test |
|---|---|---|
| Markdown workflow and two-run deduplication | Verified locally in Checkpoint 2 | Real private candidate behavior |
| Public Lever/Greenhouse access | Verified in local Codex session | Claude Routine network/search access |
| Cloud Routines and Run now | Documented by Anthropic | Account/plan/org availability |
| Fresh clone per cloud run | Documented by Anthropic | Private branch checkout behavior |
| `claude/` branch pushes | Documented by Anthropic | Push to this private default state branch |
| Custom domain allowlist | Documented by Anthropic | Exact domains needed for discovery |
| Connectors in Routines | Documented by Anthropic | Connector availability and permissions on this account |
| Gmail send tool | Documented by connector listing | Unattended Routine send approval |
| Schedule local-time conversion/stagger | Documented by Anthropic | Account timezone and displayed next run |
| Git persistence protocol | Designed and locally reviewed | Two separate cloud executions and overlap rejection |
| PDF access | Repository files are cloned | Resume parsing quality and need |

## 11. Security checks

- Public staged-file allowlist reviewed before every public push.
- Private repository visibility confirmed before personal files are added.
- Public upstream has no usable push URL in the private clone.
- Cloud GitHub access limited to one private repository.
- Custom network allowlist instead of Full access.
- No setup script, extra package, `.mcp.json`, or environment secret for V1.
- Only necessary connector enabled; all default-added connectors removed.
- Job descriptions treated as data, never commands.
- No authenticated LinkedIn scraping or access-control bypass.
- No SMTP fallback.
- No force push or automatic conflict resolution.
- Session sharing remains private; transcripts may contain candidate data.
- Reports avoid unnecessary personal details.
- Routine stays paused during maintenance or unresolved delivery state.

## 12. Outstanding decisions requiring approval

- Public and private GitHub owners and final repository URLs.
- Permission to create each remote repository and make the first push.
- Real profile, preferences, resume choice, source/employer list, and posting-age policy.
- Private report retention policy and whether reports should remain indefinitely in Git history.
- Account plan/organization and GitHub connection method.
- Approved Custom-network domains after the first access test.
- Desired daily time in `Europe/Istanbul` and acceptable stagger.
- Whether delivery remains `report_only`, advances to `prepare_email`, or tests `send_email`.
- Email provider/connector, approved recipient, and authorization for one real test message.
- Whether LinkedIn alert mail should be read through an authorized mailbox connector.
- Manual reconciliation owner and procedure for `delivery_uncertain` runs.

Checkpoint 3 prepares these actions but authorizes none of them. Final approval is required before any remote or external action.
