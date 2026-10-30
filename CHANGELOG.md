# Changelog

Akzo SDD Agent — Developed and maintained by Arun RG. The version is in `VERSION`; `install.ps1 -Verify` shows the installed and available versions.

## 3.11.0 — 2026-10-29
Change risk, acceptance-criteria trace, PR text, custom review rules, and a per-repository coverage threshold. Gates G1-G5 and G8, `decide`, `guard`, repository memory, and profile isolation are unchanged. No new dependency, tool, or network call.

**Added**
- `review` returns `risk`: each changed file with a level (high, medium, low) and its reasons, from the `riskRules` in `profiles.json` (shared build-file rules plus the detected profile's rules; another profile's rules never apply), plus "no test named after it" (the `affected` mapping) and more than 200 changed lines. `highRisk` and `riskNext`: a high file means `affected` stage D runs before G5. Without a profile only the shared rules apply, and the output says so.
- `trace -Id <PBI> [-Path <.trx file or folder>]`: each AC of spec.md, the tests the plan section 6 Test column lists for it (`file::name`), and their outcome in the TRX results: COVERED, FAILING, NOT RUN, or MISSING. A bare name found in several classes is reported as ambiguous (NOT RUN); `file::name` picks the class; data rows count as one test; the newest result per test wins. Without results nothing is assumed to pass.
- `prtext -Id <PBI>`: a PR description (summary, tasks, trace, change risk, the Sonar table or "SonarCloud not verified", and the time elapsed between approvals, labelled as including review time) written to the task's `evidence\pr-description.md` and printed. Secrets are redacted; it never creates a pull request, commits, or pushes.
- Custom review rules: `review` also reads `<AKZO_HOME>\profiles-custom\review-rules.json` (`[{ id, profile, pathGlob, regex, severity, message }]`) on changed lines. An unreadable file or an invalid rule is one `custom-rule-invalid` warning; the built-in scenarios are unchanged.
- `coverage` returns `threshold` and `meetsThreshold`: the repository's `quality.coverage` from the registry, else 80%. A coverage condition of the server quality gate replaces both.

**Changed**
- Agent: change risk listed in the handover; `trace` before G5 (G5 blocked on MISSING or FAILING, the handover matrix built from `trace` output); `prtext` for the PR text; Fix PR comments proposes a custom review rule as text for a recurring verified comment (the developer approves the file change); Fix Issue retries a fix up to 3 times instead of ending with a findings list; the Sonar table lists the rules checked (only rules confirmed active count as Sonar rules), compiler and .NET analyzer warnings go with the build results, and "SonarCloud passed" needs the server or the CI gate.
- Profiles: `riskRules`; the DAB and CDMS Sonar rules list S3900, S4055, S1541, and S1200 as "confirm active first" and CS1572, CS1573, and CA1860 as build warnings. Every fixed "80%" reads "the repository's threshold (default 80%)".
- `install.ps1 -Verify`: a renamed copy of the agent inside the install folders (`.copilot\agents`, the Visual Studio agents folder) is an Old-version conflict. The version comparison with the package already existed.
- Plan template: `trace` reads the Test column.

**Tests:** 16 new checks (change risk without a profile, DAB, CDMS; trace statuses, ambiguity, missing results; prtext; custom rules and an unreadable rules file; renamed agent copy; coverage threshold; agent wording). Result 2026-10-29: 386 passed, 0 failed; window tests (`test-gui.ps1`) 14 passed, 0 failed.

**NOT VERIFIED:** how Copilot follows the new instructions in a live session; the SonarCloud gates of DAB and CDMS (their `quality.coverage` is not set until read from SonarCloud); `trace` on TRX files from the real DAB and CDMS test runs; Visual Studio 2026.

**Deferred:** golden-case evaluation (`eval`; agent behavior in a live session stays NOT VERIFIED until it exists), a doctor score, a metrics command, a `learn` command, moving the built-in review rules to JSON, and new guide files. The UI (Angular) profile is blocked until its repository files are available.

## 3.10.0 — 2026-10-29
Readable gates and tasks, recorded gate decisions, and Git and generated-client checks. No application source and no setup window changed.

**Audit before the change**
- The agent has six gates, not eight: G1 Clarification, G2 Gap map, G3 Spec, G4 Plan, G5 Verification, G8 Corporate security. Build, test, and quality checks are part of G5 (section 6); there is no G6 or G7. The proposed G1-G8 titles were therefore mapped onto the real gates, and no gate was added or changed in meaning.
- Already present and kept: checkpoints and resume (plan section 0 Remaining, `task` statuses, Resume and Modify modes), Git safety (`guard`, never stash/reset/clean, `review` with `git diff --check`), build, coverage, and Sonar evidence, CDMS/DAB profiles and reference implementations.

**Added / changed**
- `modules\plan\assets\workflow.json`: each gate's title, purpose, activities, deliverable, what to review, and what follows approval, plus the task status vocabulary. People see titles; ids stay in files and logs. Titles can be edited there.
- `task` output: `gates` (id and title), `nextGate` (the gate waiting for a decision, with its explanation and the choices Approve / Request changes / Custom response), `planTasks` (each plan task's title and status; plans with `todo` / `done` map to Not Started / Completed), `approvals` (latest decisions).
- `decide -Id <PBI> -Gate G2|G3|G4 -Decision approve|changes|custom [-Text ...]`: records the decision in the task's `approvals.jsonl` (gate id and title, choice, feedback with secrets redacted, user, time, SHA-256 of the document) and writes the status line. Refuses a second approval, an approval before the earlier gate, an approval after changes were requested and before the revision is presented (`ALREADY_APPROVED`, `OUT_OF_ORDER`, `NEEDS_REVISION`), and a decision on a gate with no deliverable.
- `branchinfo`: staged, unstaged, untracked, conflicted files, and a merge, rebase, cherry-pick, or revert in progress.
- `review`: an empty generated `.cs` file is an error (`generated-empty`).
- Agent: gates presented by title with the three choices; ambiguous replies are never approval; tasks named by title; statuses Not Started / In Progress / Implemented - Verification Pending / Blocked / Verified / Completed with implementation and verification kept apart; a later message never restarts a Verified or Completed task; stop on a merge in progress or conflicts; generated GraphQL clients checked at the baseline and regenerated, never resolved by taking one side; `restore` added to the Git commands never run without a request.
- Plan template: descriptive task title, an Objective column, and the new statuses.

**Fixed during the work:** the new `-Decision` parameter shared its name with `branchuse`'s `$decision` variable (PowerShell names ignore case), which made `branchuse` fail; the parameter variable is now `$GateDecision` with the alias `-Decision`.

**Tests:** 13 new checks (workflow file, gate titles on an old task, pending gate, plan task titles and old statuses, `decide` refusals, request-changes cycle, approval record and redaction, resume, merge and conflict state, empty generated client and conflict markers, agent wording). Result 2026-10-29: 370 passed, 0 failed; window tests (`test-gui.ps1`) 14 passed, 0 failed. Run from Windows PowerShell: started from another shell (Git Bash here), the unreadable-folder check cannot lock its folder and fails.

**NOT VERIFIED:** how Copilot presents the choices in a live session (`benchmark/agent-evaluations.md` 28-35); PBI 305912 on the DAB and CDMS repositories, the installed agent at `C:\Users\RGA\.akzo-agent`, and StrawberryShake generation, none of which are on this machine.

## 3.9.0 — 2026-10-28
Setup: repository management, installation status, and agent discovery. No application source and no SDD workflow, skill, or profile changed.

**Root cause found (agent not in the picker)**
- 3.8.0 added `Small work stays small: Fix Issue...` to the agent's unquoted `description`. A `: ` inside a plain YAML value is a syntax error for a strict YAML parser. VS Code's own front-matter parser (taken from the installed 1.143.1) accepts it, so VS Code is not affected; Visual Studio's parser is not documented, so this is a likely but NOT VERIFIED cause there. Fixed (` - `), and a test keeps the value free of `: ` and ` #`.
- Checked on the installed products, not assumed: VS Code 1.143.1 reads user agents from `~/.copilot/agents` (default `chat.agentFilesLocations`: `.github/agents`, `.claude/agents`, `~/.copilot/agents`); `.github/agents` is workspace-relative, so the `%USERPROFILE%\.github\agents` copy serves Visual Studio only. Visual Studio 2022 17.14 has no custom agents (no `.agent.md` support in its Copilot binaries). Visual Studio 2026 18.5+ was not available here.
- Other causes the status panel now reports with their fix: an old VS Code build without `~/.copilot/agents`, a user setting that turns it off, `chat.disableAIFeatures`, Visual Studio older than 18.5, a duplicate agent in another discovery folder, installing for another Windows user.

**Setup window** (`Install-MasterAgent.ps1`, same engine)
- Resizable. Repositories table (name, path, validation, agent status; tooltips, keyboard, double-click) with Add, Edit, Remove, Open in Explorer, Validate, Refresh. Add/Edit dialog: name suggested from the folder; path typed, pasted (quotes removed, spaces, UNC) or browsed; Browse, Validate, Save, Cancel. Saved at once; nothing changes until Save succeeds.
- Agent Installation Status panel: Verified / Warning / Failed / Not Applicable / Not Checked per check. IDE picker visibility is always Not Checked, with manual steps. Install and Repair run Verify afterwards and list what is still unresolved.

**Engine** (`install.ps1`)
- `-ValidateRepo`, `-ListRepos` (JSON), `-EditRepo -RepoPath -RepoName`, `-RemoveRepo`, `-AddRepo ... -RepoName -RegistryOnly`. Paths are kept as given (a subfolder is no longer swapped for the repository root); duplicates are found after normalization; missing, relative, malformed, and unreadable folders are refused; a folder that is not a Git clone is registered with a warning. Remove and path edits withdraw the developer authorization recorded for that id (backed up), so it cannot carry over to another folder; the folder, task files, and architecture map are kept.
- `-Verify` prints `CHECK [status] name: detail` lines; `-VSCodeInstallDir` for tests.
- Package integrity: a working copy is reported as `NOT VERIFIED - development working copy`; a release manifest for another version is refused; the install manifest records which (`package`).

**Tests:** `scripts\test.ps1` gains checks for pasted and quoted paths, invalid and unreadable folders, duplicates, several repositories, edit, remove without deleting source, persistence, old entries without a name, the status lines, old VS Code builds and settings, duplicate agents, invalid metadata and version mismatch, repeated repair, and release-manifest version. 357 checks pass. `test-gui.ps1` (14 checks pass) clicks *Verify Installation* and saves a pasted, quoted path through the real Add Repository dialog.

## 3.8.0 — 2026-10-28
Fast task routing, minimal context, and evidence-based fixes. Akzo SDD PBI Builder was not changed; no application source was changed.

**Root causes found**
- The agent itself told Copilot to always read `copilot-instructions.md` and `AGENTS.md` (section 2), although the IDE already adds them to every request.
- Every task, a one-line Sonar fix included, went through the PBI pipeline, and `profile -Query` loaded the shared skills plus the profile's core skills: 6-8 guides. Trigger patterns also matched words inside class names (`Create...Handler` matched `create` and `handler`).
- Repository side (this agent cannot change it): in VS Code 1.143 / Copilot Chat 0.71 (defaults read from the installed product), `copilot-instructions.md`, root `AGENTS.md` and `CLAUDE.md`, and `applyTo: '**'` instructions join every request of every agent. A line there that says "read X before any task" makes every agent read X. Linked files are not attached automatically.

**Added / changed**
- Agent: "Start and task routing" (the five-choice `start` menu, direct-task routing, a per-mode reading ceiling, IDE-loaded instructions not claimed to be overridden, missing-tool fallbacks); **Fix Issue** and **Refactor** modes (Debug folded into Fix Issue); New Development / Review / Custom map onto the existing modes; evidence rules; the five-line Fix report.
- `profile -Task fix|refactor|new|review`: fix loads source grounding and the Sonar rules only; refactor adds the guides the smell triggers; new and review are unchanged. Output field `task`.
- `guidance -Repo <id>` (read-only, auto-approved): instruction files with their load scope, the documents and skills they reference (file:line, mandatory wording, unconditional, affects the regular agent, missing), and a scoping recommendation. `repoGuidance` rows gain `scope`; `CLAUDE.md` is now listed.

**NOT VERIFIED:** live Copilot runs of the routing cases and the S1200 regression case (`benchmark/agent-evaluations.md` 17-27). No DAB or CDMS repository is on this machine, so their instruction files were not audited.

## 3.7.0 — 2026-10-28
Faster development loop (lazy profile loading, cached detection, exact-symbol lookup, staged tests) and reusable development and PR-review intelligence for any DAB or CDMS work item. Combines two briefs: the high-performance engine and universal DAB development and PR review. Akzo SDD PBI Builder was not changed, and no application source was changed. No feature was removed; the security controls and every gate are unchanged.

**Existing capabilities (findings before the change)**
- Already present and reused rather than rebuilt: the per-repository index (`architecture.json`, `component-index.tsv`, `symbols.tsv`), fingerprinted by Git blob ids and refreshed incrementally for committed, staged, untracked, renamed, and deleted files; verified facts revalidated against the blob of every evidence file (stale facts are never served as verified); `context` retrieval with budgets; corrupted memory set aside and rebuilt. Brief items 1, 4, and 5 (discovery once, analysis cache keyed to content, incremental reanalysis) were therefore already met by the index; this release adds the pieces that were missing.
- Measured hot spots (613-file generated repository): `context` spent about 0.9 s tokenizing every path even when no term could match and about 0.4 s on up to 25 separate `git grep` reference counts; `profile` and `guard -Repo` re-ran full detection (file list, 300 project files, code grep) on every call; nothing chose tests by impact; every profile skill loaded on every request.
- No Business Rule Branch Coverage skill existed in this package or in the PBI Builder folders, so it was created (no duplicate). The DAB architecture skill told the agent to treat plugin-derived rules as out of scope; brief 2 replaces that policy.

**Added / changed**
- `profile -Query "<request>"`: loads the shared skills, the profile's `core` skills, its Sonar rules, and the skills whose `triggers` match the request; the others are returned as `deferred` (read when the work reaches the topic). `neverLoad` and isolation are unchanged. Without `-Query` everything loads, as before. Detection is cached in `maps\<repo>\profile-detection.json`, keyed by HEAD, `git status`, size and time of every changed file, and the hash of `profiles.json`; a corrupted file is set aside (`cache: recovered`) and a deleted one is rebuilt.
- `context`: a code-shaped single name (`CreateOrderValidator`, `order.service.ts`) returns `exact: true` with the file, its tests, and its direct `callers`; a substring pre-filter skips files that cannot match before tokenizing (ranking output byte-identical on 21 query/mode combinations); one `git grep` for all reference counts.
- `affected -Repo <id> [-Base <branch>]` (read-only): changed files -> owning projects -> reverse project references -> stage A build commands, stage B `--filter` for tests named after the changed classes, stage C dependent test projects, stage D the full suite once at the final gate. Solution, props, package, or root files -> `BUILD_WIDE`. Changed files without a matching test are listed. Auto-approved in VS Code like `profile` and `review`.
- `review`: new warning `single-use-abstraction` for a newly added `...Helper/Collaborator/Wrapper/Manager/Provider/Factory/Container/Dependencies/Utility` type that at most one other production file uses.
- New shared skill `business-rule-branch-coverage` (decision table per approved rule: Yes/No paths, required fields, duplicates, state transitions, side effects, failures, transaction boundaries; COVERED / PARTIAL / MISSING / UNKNOWN with evidence; legacy-derived rules ported without a runtime dependency on the legacy platform).
- Source-grounded skill: search order with a stop rule, **validation ownership**, and **abstractions: add or remove** checks. Pre-PR review skill: twelve reusable review categories, recurring problems searched across the feature, a disposition per comment, review-only stops. DAB architecture: no separate DAB plugin; approved plugin/workflow/JavaScript/web-resource rules are implemented in DAB layers with no CRM runtime dependency. DAB FluentValidation: extend the existing validator; not every rule belongs in FluentValidation. CDMS architecture: plugin-owned behavior stays in the plugin; DAB conventions never apply.
- Agent: `profile -Query` in section 1a, exact-name search order in section 2, branch tables in the gap map (one implemented branch = EXTEND), staged tests through `affected` in sections 5 and 6, remote Sonar queries at baseline and final gate only (6a step 2), a **Review-only / plan-first** mode, a completion gate that covers branches, abstractions, validation layer, registrations, and error contracts, and a "Fast development" section replacing "Context efficiency". The agent text still names no project.
- `scripts\perf-bench.ps1` (interleaved before/after helper benchmark on a generated Functions/CQRS/GraphQL-shaped repository) and `benchmark\agent-evaluations.md` (16 reusable evaluation cases; which are automated, which need live Copilot runs).

**Upgrade:** nothing to re-authorize or re-index; `repos.json`, authorizations, tasks, memory, and `profiles-custom` are kept. A customized copy of a changed profile file keeps winning: compare it with the shipped 3.7.0 version.

**NOT VERIFIED:** no DAB or CDMS repository, real SonarCloud project, or live Copilot session was available. Agent behavior (tool calls, tokens, files opened, questions asked, review-only stops) and test-run durations are unmeasured; only helper calls are benchmarked. Brief 2's evaluation cases that depend on model judgement are listed as live cases, not as passed.

## 3.6.0 — 2026-10-28
Repository development profiles for DAB and CDMS, source-grounded development, and a pre-PR review that checks the six recorded PR review findings. Akzo SDD PBI Builder is a separate product and was not changed. No feature was removed; the security controls are unchanged.

**Existing architecture (findings before the change)**
- One Copilot agent file (project-agnostic since 3.2.0, enforced by a test) plus a helper and templates under `%USERPROFILE%\.akzo-agent\modules`, deliberately outside Copilot's skills and instructions folders: Copilot loads those for every agent (the leak fixed in 3.0.0).
- Generic .NET, Azure Functions, and GraphQL stack rules (S5-S7), Sonar sources and changed-line coverage (section 6a, `sonar`, `coverage`), gates, the G8 security gate, and a command guard already existed. Nothing selected rules per repository, nothing reviewed a diff mechanically, and `guard` did not know which solution belongs to which repository.

**Added**
- `copilot\modules\profiles\`: `profiles.json` (detection data), 2 shared skills (`source-grounded-development`, `pre-pr-review`), 7 DAB skills, 9 CDMS skills, and separate Sonar catalogs (`instructions\dab\`, `instructions\cdms\`). Installed with the modules, not into a Copilot discovery folder. The brief named the skill files `SKILL.md`; they are `guide.md` in the requested folders, with their name and purpose in a header line instead of YAML front matter, because an existing isolation test (kept unchanged) forbids anything under `modules` that a skill scanner could pick up: a discovered skill would be offered to every agent, the leak fixed in 3.0.0.
- `profile -Repo <id> [-Key <profile>]`: `DETECTED` only when the profile's solution file is in the repository (`AN.ColorAppsAzure.BL.sln` / `AN.ColorAppsAzure.BL.ColorDMS.sln`); packages, files, and code are reported as evidence. Two profile solutions -> `AMBIGUOUS` (the agent asks, then `-Key`); markers without a solution -> `UNCERTAIN`; neither -> `NO_PROFILE`. Returns the files to `load` (shared + one profile), `neverLoad` (the other profile), and the build and test commands with the solution's folder and a check for `coverlet.runsettings`.
- `review -Repo <id> [-Base <branch>]` (read-only): against the merge base of the branch, reports the six PR regression scenarios (`redundant-validation-conversion` for PUT and POST functions when exception middleware handles the validation exception, `duplicate-change`, `validator-pattern`, `dependency-organization`, `xml-doc-missing` / `xml-doc-outdated` / `xml-doc-incomplete`), the profile's 200-character line limit (CDMS) and broad catches, placeholders, and `git diff --check`.
- `guard -Repo <id>`: a `dotnet` command naming a solution that is not in that repository, or that belongs to the other profile, is `BLOCK` / `wrong-solution`.
- Agent: section 1a (profile first, load only its files, guarded commands, anti-hallucination rules), pre-PR review and a **Ready / Ready with Warnings / Not Ready** completion gate in section 7, and the requests "Implement this requirement", "Fix PR comments", "Review changes", and "Run tests". The agent text still names no project.
- `%USERPROFILE%\.akzo-agent\profiles-custom\`: the developer's own version of any profile file (same relative path) is used instead of the shipped one; the installer never writes there. An installed file edited by hand is now reported with a warning before it is backed up and replaced. `-Verify` checks that every profile file is installed.

**Not available when this was written (NOT VERIFIED):** the team's AN.ColorAppsAzure.Formula and ColorDMS agent instructions, the CDMS SonarCloud rule catalog, the `add-feature` and `run-tests` skills, and the full text of the six PR comments were not supplied. The shipped files were written from the brief; place the originals in `profiles-custom`. The brief lists S2345 for interface parameter names; in SonarC# that rule is S927 (S2345 concerns `[Flags]` enums), so the CDMS catalog names both and asks for a check against the quality profile. The CDMS test command's `./TestResults` is replaced by the task's evidence folder so nothing is written into the repository. No DAB or CDMS repository, real SonarCloud project, or live Copilot session was available, so detection, review, and the profiles are verified on fixtures only.

**Tests**: 309 checks (22 new): DAB, CDMS, ambiguous, uncertain, and unrelated repositories; isolation of the two profiles; profiles-custom override; guard against another repository's solution; each of the six PR scenarios reproduced on a feature branch and the corrected change passing; CDMS documentation, line-length, and broad-catch rules; `-Base` validation; agent and profile-file static checks; separate Sonar catalogs; upgrade preserving custom files and reporting an edited installed file; Verify counting the installed profile files.

## 3.5.0 — 2026-10-26
Restores the complete development workflow. No feature was removed; the security controls are unchanged.

**Why these behaviours were lost (root causes, checked against the shipped 3.0.0-3.4.1 packages)**
- *File dialog and fresh tasks.* The dialog code (`pick`) is byte-identical in 3.2.0, 3.3.0, and 3.4.1; it was not broken. The agent's instructions made it unreachable for new work: plain "start" took the **newest inbox item** (after a finished PBI, that same PBI), step 2 said to "continue without asking", the `IN_INBOX` rule added in 3.4.0 took inbox items in "without asking", and "New Development" / "Start Implementation" were never entry phrases. A cancelled or failed dialog ended with a one-line note and no next step.
- *Separate spec and plan, task isolation.* The spec folder was named from the title only (`CreatePaintSystem`, no PBI number), and spec/plan creation never overwrites. A second PBI with the same title therefore landed in the first one's folder and **inherited its approved spec, approved gap map, and "Done" plan** (reproduced: PBI 300999 reported the statuses of PBI 300186). Nothing checked whether a folder belonged to another work item.
- *Branch selection and creation.* Never implemented in any version: the helper only read the branch name; the agent's only branch instructions were "inspect only" and "never switch branches".
- *Gap analysis and approval gates.* Present since 3.2.0, but reachable only after intake; with the entry flow above, an old task (or its inherited approved statuses) let the agent skip straight to later steps.
- *Memory versus PBI progress.* Repository memory accepted any statement, so PBI-specific progress could be stored as an architecture "fact" and shown to every later task.

**Fixes**
- Entry: "Start Development", "New Development", "Start Implementation" (no id) always start a fresh task with the file dialog; never the previous PBI or the newest inbox item. Ambiguous requests get one choice: a) new development, b) modify an existing implementation, c) resume an existing PBI. `resume <PBI>` loads that task directly.
- `pick`: a dialog that cannot open or a missing file returns `PICK_FAILED` with the file-path and launcher alternatives; `CANCELLED` says nothing was created. The agent never skips intake.
- `intake`: an existing task id is reported (`TASK_EXISTS` with resume / modify / separate-task choices and a suggested new id) instead of being reused; `-Resume` reuses it deliberately.
- Unique spec folders: `<specsDir>/<id>-<title-slug>/` (e.g. `300186-create-order-export`); `-Feature` still sets another name. A folder recorded for, or holding the work of, another PBI is refused (`SPEC_LOCATION_TAKEN`). Resuming a PBI on a branch that lacks its spec returns `WRONG_BRANCH` instead of recreating it from templates.
- Branches: new section 0a. `branchinfo` (read-only) shows repository, branch, HEAD, uncommitted changes, branch-to-PBI records, a proposed `feature/<id>-<title>` name with an existence check and an alternative, and base candidates to confirm. `branchcreate` / `branchuse` act only with `-Confirmed` after the developer chose; they refuse with uncommitted changes unless the developer agrees (`-AllowDirty`); nothing is stashed, reset, or discarded. In repository spec mode, no file is created in the repository until the decision is recorded (`BRANCH_DECISION_NEEDED`). Neither command is auto-approved.
- Workflow: the 11 steps are explicit in the agent (intake, branch, investigation, gap map, spec, review, plan, consistency and approval, staged implementation, stage validation after every stage, final validation and handover). Auto Mode still produces gap map, spec, plan, and evidence, and still asks the branch decision unless its authorization names it.
- Modify mode: `snapshot` copies spec, plan, PBI, inputs, and state into the task's `history\` before changes; gates are reopened for the changed parts.
- Task isolation: `tasks` lists every task of a repository with its own statuses; `task` reports `taskKind`, `completed`, `branchState`, and `overlap` (earlier, possibly completed, tasks that look like the same feature, so the gap map checks them instead of reimplementing). `remember` refuses facts that name a work item: memory is architecture knowledge only.

**Tests**: 287 checks (21 new) including fresh development after a completed PBI, memory reuse without task progress, dialog failure, branch choices with existing branches and uncommitted changes, unique and protected spec folders, wrong-branch resume, modify and resume, and a pre-3.5.0 task resuming unchanged; the window test now also cancels the agent's own dialog.
## 3.4.1 — 2026-10-26 (release candidate)
Release candidate. Supersedes 3.4.0, which was packaged but never distributed and contained the three defects below; the 3.4.0 zip and its checksum were deleted on 2026-10-26 and this entry, with corporate-security-report.html, keeps the defect history. No new features. Corporate readiness: NOT VERIFIED (approvals listed in the report).

**Known limitation, not technically prevented (documented, not fixed):** The developer's authorization is stored in `%USERPROFILE%\.akzo-agent\authorizations.json`. Any process running as that user, including a model with file-edit or terminal tools, can write that file directly. The `authorize` command asks the IDE for confirmation, but the helper cannot tell a command from a direct file write, so the developer's approval is enforced by the IDE's prompts and the agent's instructions, and instructions alone do not enforce approval. Where approval must be enforced, an administrator places a `policy.json` that users cannot write, with `allowDeveloperAttestation` set to `false`: developer statements are then ignored and only listed repositories are processed. G8 reports a developer statement as NOT VERIFIED in every case.

**Security defects found by review and testing, fixed**
- Path traversal through the work-item id: `guard`, `attach`, `role`, and `security` accepted ids such as `..\..` and resolved them to folders outside the tasks folder (a log file could be written there). Ids are now validated wherever a task folder is looked up.
- Sonar token over plain http, and across redirects: the token could be sent unencrypted to an allowed host, or follow a redirect to another place. It is now sent only over https (a loopback server is exempt, for local testing), redirects are not followed, and an insecure host stops before any request (`INSECURE_HOST`; the G8 SonarQube control reports NOT VERIFIED with the reason).
- Memory poisoning: `remember` stored any text, so instruction-like text from a hostile document could be saved as a "verified fact" and loaded in later sessions. Facts that read like instructions to the agent are refused.
- Checked and not reproduced: slow-regex (ReDoS) behaviour of the symbol rules on five hostile line shapes (at most 9 ms each; lines over 400 characters are skipped). Not exhaustive: marked NOT VERIFIED beyond those shapes.

**Performance (measured, `benchmark\results-3.4.1.md`; no check was removed)**
- Sonar setup in `task` reuses the file list the architecture sync already has and skips the machine-tool detection (use the `sonar` command for that): 249 ms -> 44 ms.
- Fingerprint built from an ordered string builder instead of a pipeline; folder-permission checks through .NET instead of `Get-Acl`; one git call fewer when fingerprinting (deleted files come from the same listing); memory-log hit rate read without a pipeline; symbol rules compiled on first use; comment and closing-brace lines skipped before any rule runs.
- Measured with interleaved runs (`scripts\ab-bench.ps1`, 6 rounds, same repository, medians; `benchmark\interleaved-3.4.1.md`): cold `task` 3424 ms (3.4.0) -> 3078 ms (3.4.1), about 10% faster; warm `task` 2249 ms -> 1820 ms, about 19% faster. Against 3.3.0 in the same rounds, 3.4.1 is still +83% cold and +23% warm (3.4.0 was +104% and +52%). An earlier estimate of "about a third faster" came from runs taken one after another while machine load changed and is withdrawn. The G8 preflight, command guard, and memory protection are unchanged.
- Deferred (gain too small for the risk, or adds attack surface): one combined git status call for fingerprinting; extracting symbols lazily on the first `context`; a background indexer process; a real parser such as Tree-sitter (only if measurements show regex extraction missing symbols).

**Tests and process**: 264 checks (5 new: task-id traversal, instruction-like facts, https and redirect handling of the Sonar token with a local stand-in server; the new `ab-bench.ps1` is parse-checked). Each stage ran a strict parse of every script plus a focused smoke test; the full suite and the release validation ran before packaging.

## 3.4.0 — 2026-10-26
Upgrade note: after upgrading, each registered repository needs a one-time authorization (the agent asks; see README, "Upgrading"), and the old `architecture.json` is rebuilt once in the new format. Tasks, `repos.json`, inputs, specs, and notes are untouched.

**Persistent repository memory and retrieval**
- Content fingerprint of every file (git blob id): committed and uncommitted edits, staged and untracked files, deletions, and renames are detected; only changed files are re-read. `architecture.json` (schema 2) adds languages, frameworks from manifests, and inferred patterns; new `symbols.tsv` (types, methods, functions, routes, decorators, imports for C#, TypeScript/JavaScript, Python, Java/Kotlin, Go).
- Verified facts (`facts.json`) with `path:line` evidence and content fingerprints: types architecture, pattern, decision, component, validation, testing, integration, command, convention, pitfall. Stale when evidence changes (dependants too), expired when evidence is gone or retention passes. New commands `remember` and `forget`; secret-looking values are refused. Memory log with hit rate. Corrupt memory files are set aside and rebuilt; writes are atomic.
- New command `context -Query -Mode compact|balanced|deep`: facts first, then ranked files with symbols, linked tests, reference counts, and siblings, within a configurable budget (`retrieval` in `repos.json`); paths beyond the budget are still listed. Agent: four retrieval levels, memory lifecycle, memory update before handover.
- "Implement PBI <id>": a PBI waiting in the inbox is found (`IN_INBOX`) and taken in with the open workspace as the repository.
- **Auto Mode** (Run is its earlier name): authorization record in plan section 0, gates recorded as `Waived (Auto Mode, ...)`, explicit stop list, `resume` understands waived gates. Spec/plan templates: security and performance lines, test level and scenario, requirement -> test -> code matrix.
- Sonar gate also reports the commit of the last server analysis and whether it is behind HEAD.

**Corporate security gate G8 (mandatory, never waived)**
- Preflight: no indexing, extraction, or content output until a policy or the developer's recorded statement authorizes the repository (`policy`, `authorize`; `NEEDS_AUTHORIZATION`, `BLOCKED_BY_POLICY`; fail closed on an unreadable policy; restricted data needs an administrator policy; approved models, allowed hosts, memory disabled, and retention come from the policy). Installer discovery waits for the authorization.
- `guard`: ALLOW / NEEDS_APPROVAL / BLOCK for terminal commands by category (external transfer, remote execution, security-tool tampering, execution policy, registry, elevation, credential access; publishing, dependencies, database, destructive, deployment, analysis upload), logged with redaction. The deny list works by category and behaviour, not by naming attack tools.
- `scan` (secrets, added suppressions and skipped tests, dependency changes on changed lines; specific secret patterns rank above the generic one), `security` (13 controls, report `corporate-security-report.html` in the task's evidence), `purge`. Redaction of secret-looking values in logs, notes, and output; owner-only access on memory and task folders; helper network calls only to allowed hosts and logged.
- `scripts\make-release.ps1` builds the release zip with `release-manifest.json` (SHA-256 of every file) and `.sha256`; `install.ps1` refuses a package with a changed, missing, or unlisted file before writing anything.
- Package report: [corporate-security-report.html](corporate-security-report.html): controls, test evidence, endpoint-protection findings during development, limitations, and the approvals still required.

**Fixes found while testing**
- `inbox` failed on a file without a heading (existing since 1.3.0).
- Dependency detection named only the first package on a changed line; it now compares the manifest's packages with HEAD.
- A connection string with a password was classified as a low-certainty finding because a generic pattern was checked first.
- `guard` failed on commands with inner double quotes under Windows PowerShell 5.1; the pieces are rejoined (the agent is told to use single quotes and to treat a failing guard as blocked).
- Tests: 259 checks (C#, Angular, and Python fixtures; memory, incremental indexing, facts, retrieval, policy, guard, scan, G8, purge, release integrity, syntax-tree audit of the shipped scripts); `make-release.ps1` and `benchmark.ps1` are now parse-checked.

## 3.3.0 — 2026-10-25
- **SonarQube and code rules (agent section 6a):** Sonar sources used strongest first and named in every result: SonarQube MCP tools in chat, read-only helper server queries, Sonar analyzers in the build or lint run, SonarQube for IDE problems, and only on request a scanner upload; otherwise a static review against the code rules, labelled as not a Sonar analysis. Baseline, gate for new code (project gate, or stated defaults: 0 new issues, hotspots listed for review, changed-line coverage >= 80%, new duplication <= 3%), issue-fixing procedure, false positives reported instead of suppressed.
- **Code rules** applied while writing code: complexity and size, unused and dead code, error handling, literals, security, resources and async, state; the repository's own rule configuration wins.
- New helper command `sonar`: Sonar setup of the repository and machine (project key, organization, server, token set or not, pipeline Sonar steps, analyzer packages, coverage tools, rule configuration, exclusions, installed tools), also in the `task` output as `sonar`; with `-Check gate|issues|hotspots|coverage|duplication|rule` a read-only Sonar Web API query. The token is read from `SONAR_TOKEN` and sent only in the request header; branch, pull request, and rule key are validated before use.
- New helper command `coverage`: coverage of the lines changed against HEAD (edits and new files) from Cobertura or LCOV reports: per file coverable, covered, uncovered line ranges, partly covered branches.
- Plan template: Sonar table (source, gate, new issues, hotspots, changed-line coverage, duplication, fixes). Installation guide: SonarQube section and troubleshooting. VS Code auto-approve covers `sonar` and `coverage`.

## 3.2.0 — 2026-10-22
- **Project-agnostic agent:** no project, application, framework, or vendor names in the workflow or templates; stack rules (.NET, serverless functions, GraphQL/generated clients) are profiles that apply only when the repository's architecture index detects that stack.
- **Gated by default:** stops for approval after the gap map (G2), the spec (G3), and the plan (G4); `approve`, `approve all`, or changes. `develop <PBI> end to end` keeps the no-stop Run mode. Approvals are written only after the user's message; the helper reports `gapMapStatus`, `specStatus`, `planStatus`. Limit: the host cannot block file edits, so gates are instruction-enforced.
- **Requirement-to-source and requirement-to-code map (spec §3a):** REQ ids, supporting/contradicting documents, a recorded multi-angle search, EXISTING / REUSABLE / EXTEND / NEW / UNKNOWN with `file:line` and evidence level; NEW only after the full search. Plan tasks carry REQ ids, a per-file action (reuse / modify / new), evidence, and a "reused without change" list; the consistency gate blocks re-implementing existing code.
- **Multi-file input:** `pick` opens a second, multi-select dialog for supporting documents (`-Supporting` without the dialog); new `attach` adds documents later; new `role` corrects a guessed role. Managed copies under `supporting\` (originals untouched, duplicates detected, same names kept as `name-2`). `intake` reports duplicates and renames instead of silently skipping.
- **Input manifest and cache:** `inputs.json` per task (hash, type, role, status, source path, first seen, last changed); `.docx`, `.pdf`, and new `.xlsx` support converted once into `.extract\` keyed by content hash; `task` returns a compact `inputIndex`, `inputIssues` (missing, unsupported, unreadable, duplicate, second primary PBI, rebuilt manifest), and `inputsChanged`. The agent reads supporting documents on demand instead of all at once.
- **Architecture knowledge:** `maps\<repo>\architecture.json` and `component-index.tsv` from `git ls-files`, rebuilt only when HEAD moves (new `discover` to force); built during *Add repository*; per repository, never mixed. Verified facts keep going to `maps\<repo>.md`, updated incrementally.
- `task` output is compact JSON. Auto-approve rule covers `attach`, `role`, `discover`, and allows `;` only inside a quoted path list. `Verify` checks the Visual Studio version (18.5+ for user-level agents).

## 3.1.0 — 2026-10-21
- **Ambiguity scan** before the spec is final: eight categories rated Clear / Partial / Missing; at most 5 highest-impact questions; anything nobody can answer is marked `[NEEDS CLARIFICATION]` instead of guessed; answered questions are never asked again.
- **Requirement quality and readiness:** every AC measurable and testable, no placeholders; spec records Ready / Ready with open questions / Not ready.
- **Entities and state transitions** table in the spec (identity, validation, from -> to, trigger, rule), e.g. initial status and version on creation.
- **Consistency gate (section 4a)** between plan and code: coverage (AC without task, task without AC), duplication, ambiguity, inconsistency, guidance conflicts, with severities; CRITICAL/HIGH fixed before implementation; Plan mode includes it. Plan template section 1a records the findings.
- **Optional read-only subagents** (VS Code `runSubagent`) for parallel investigation; never for edits.

## 3.0.0 — 2026-10-21
- **One agent:** `akzo-sdd` merged into `akzo-master-sdd` and no longer shipped; upgrades remove the old agent files (manifest, or an Akzo agent file found without one), after a backup.
- **One-click setup:** `Install-MasterAgent.cmd` (Install/Upgrade, Repair, Verify, Uninstall, Add repository), `Uninstall-MasterAgent.cmd`, `Start-Development.cmd`. `install.ps1` gains `-AddRepo`, `-CleanMemory`, rollback on a failed write, and short backup names for paths beyond MAX_PATH.
- **Start Development:** the agent runs the new helper command `pick`, which opens a Windows file dialog, converts .md/.txt/.docx (.pdf with pdftotext) into `<inbox>\<name>\refined-pbi.md` with the original under `source\`, and detects PBI number and title; when number, title, and repository are clear it starts without questions.
- **Autonomous quality loop:** quality baseline before the first edit; Build → Test → Analyze → Fix → Re-validate until 0 build errors, 0 newly failing tests, 0 new warnings/Sonar issues in changed files; gate integrity (no suppressions, skipped tests, placeholders, or reworded ACs); evidence kinds; 3-attempt limit; status DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT. Run is the default mode.
- **Decisions:** mechanical (silent), taste (decided and listed for review), user decision (asked once per phase). Value completeness for changed statuses/enums, scope lock and scope check, root-cause debugging with regression test, command safety list.
- **Agent isolation fix:** VS Code loads Copilot user memory into every agent; the agent never writes its rules there, `-Verify` reports Akzo text in user/repository memory and the VS Code `prompts` folder, Repair cleans it.
- Plan template: baseline vs final quality table. GUI test `scripts\test-gui.ps1`.

## 2.0.0 — 2026-10-20
- New agent `akzo-master-sdd`: the complete akzo-sdd workflow plus an engineering-skills section (S1-S19) applied per phase — requirement sharpening, source-driven decisions, Clean Architecture/CQRS design review, contract design, C#/.NET coding, Azure Functions, DAB GraphQL/StrawberryShake, test-driven slices, incremental delivery, systematic debugging, security, performance, observability, simplification, quality-bar guard, doubt pass, five-axis review, git hygiene, decision records — and a Debug mode.
- Installer ships every agent in `copilot/agents` to both IDEs; `akzo-sdd` is unchanged and still installed.
- Debug mode never bisects or checks out other commits; without a PBI it reports in chat instead of a plan. Docs and installer messages name both agents.

## 1.3.0 — 2026-10-20
- Guided start: say "start" to pick a refined PBI from the inbox (`%USERPROFILE%\.akzo-agent\inbox`). The agent asks once for item, PBI number, title, repository, and mode, copies the PBI and notes into the task folder, names the spec folder from the title, and starts.
- Accuracy: naming and structure are taken from 2-4 sibling features (conventions table in the spec); existing GraphQL operations are reused before new ones are added; an integration check (registered, referenced, used, no duplicates, no new warnings) runs before handover.
- Continuous execution in Run mode; new Align mode for reviewing and fixing an existing implementation.
- `-ConfigureVSCode` also raises `chat.agent.maxRequests` to 400 (VS Code pauses with "Continue?" after 50 by default).
- Agent version shown in the agent, the installer output, the manifest, and `-Verify`.

## 1.2.0 — 2026-10-20
- Whole workflow inside the agent file (no workflow-file reads); one helper call starts or resumes a task.
- `-ConfigureVSCode`: auto-approve rule for the helper only, read access to the Akzo folders.
- Per-repository `specsDir` (e.g. `src/.github/specs/<Feature>/`); repository skills and instructions discovered and followed; reference implementation (e.g. CDMS -> DAB) mapped and mirrored; questions in one batch with options.

## 1.1.0 — 2026-10-19
- Refined-PBI intake: `<tasks>\<repo-id>\<PBI-ID>\refined-pbi.md`, change detection against a baseline, resume, configurable `tasksDir`.

## 1.0.0 — 2026-10-19
- First release: akzo-sdd custom agent for VS Code and Visual Studio 2026, installer with dry run, verify, uninstall, and backups.
