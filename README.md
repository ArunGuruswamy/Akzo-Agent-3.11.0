# Akzo Master SDD Agent

**Akzo SDD Agent — Developed and maintained by Arun RG** · Version: see [VERSION](VERSION) and [CHANGELOG.md](CHANGELOG.md)

One GitHub Copilot custom agent, **`akzo-master-sdd`**, for spec-driven development in any registered repository. It discovers the repository's architecture, maps every requirement of a refined PBI (plus your supporting documents) to existing code as **EXISTING / REUSABLE / EXTEND / NEW / UNKNOWN**, writes `spec.md` and `plan.md`, implements test first, and validates build, tests, and analyzers. It stops for your approval after the gap map, the spec, and the plan. It runs in Copilot Chat in **VS Code** and **Visual Studio 2026** (18.5+ for user-level agents), and only when you select it. Nothing in the agent is tied to a particular project, framework, or organization: stack-specific rules apply only when the repository's architecture index detects that stack.

> **Release candidate.** Version 3.11.0 is a release candidate. Corporate readiness is **NOT VERIFIED**: see [corporate-security-report.html](corporate-security-report.html) for what was tested, the limitations, and the approvals still required.
>
> **Known limitation:** The developer's authorization is stored in `%USERPROFILE%\.akzo-agent\authorizations.json`. Any process running as that user, including a model with file-edit or terminal tools, can write that file directly. The `authorize` command asks the IDE for confirmation, but the helper cannot tell a command from a direct file write, so the developer's approval is enforced by the IDE's prompts and the agent's instructions, and instructions alone do not enforce approval. Where approval must be enforced, an administrator places a `policy.json` that users cannot write, with `allowDeveloperAttestation` set to `false`: developer statements are then ignored and only listed repositories are processed. G8 reports a developer statement as NOT VERIFIED in every case.

## Four steps
1. **Install:** double-click **`Install-MasterAgent.cmd`** → *Add Repository…* (type, paste, or browse to your local clone; saved at once) → *Install / Upgrade*.
2. **Open** VS Code or Visual Studio 2026 (restart it after installing) and select **akzo-master-sdd** in the Copilot Chat agent picker.
3. **Type `Start Development`.** A Windows file dialog opens (it may appear behind the IDE): choose the refined PBI (`.md`, `.txt`, `.docx`; `.pdf` when `pdftotext` is installed). A second dialog lets you select any number of supporting documents at once (business rules, specs, API contracts, schemas, test cases, sample data, diagrams…); Cancel skips it.
4. **Review at three gates.** The agent stops after the **gap map** (what exists, what is reused, extended, or new), after the **spec**, and after the **plan**. Reply `approve` to continue, `approve all` to let it finish without further stops, or describe changes. After plan approval it implements, builds, tests, and reports.

Alternative to step 3: double-click **`Start-Development.cmd`** (same two dialogs), then type `start` in Copilot Chat (the launcher puts `start` on the clipboard). Add documents later with `attach <files> to <PBI>`.

| Launcher | What it does |
|----------|--------------|
| `Install-MasterAgent.cmd` | Resizable window: **Install / Upgrade**, **Repair**, **Verify Installation** (Agent Installation Status panel), **Uninstall**, and a repositories table with **Add / Edit / Remove / Open in Explorer / Validate / Refresh** |
| `Start-Development.cmd` | Dialogs for the PBI and supporting documents → queued in the inbox → tells you the next step |
| `Uninstall-MasterAgent.cmd` | Asks, then removes only installer-owned files (keeps repos.json, tasks, inbox, specs, knowledge, backups) |

Command-line equivalents (no window): `scripts\Install-MasterAgent.ps1 -Action Install|Repair|Verify|Uninstall -NoGui [-Repo <path>]`, or the engine `scripts\install.ps1` (`-DryRun`, `-Verify`, `-Uninstall`, `-AddRepo <path>`, `-CleanMemory`, `-ConfigureVSCode`, `-Target VSCode|VisualStudio`, `-VisualStudioAgentsDir`).

## Chat commands
| You type | Mode | What happens |
|----------|------|--------------|
| `Start Development`, `New Development`, `Start Implementation` | Gated (default) | Always a **fresh task**: dialogs for PBI + supporting documents, branch decision, then the run with stops at G2 (gap map), G3 (spec), G4 (plan). Never resumes the previous PBI |
| `start` | Gated | Takes the item `Start-Development.cmd` just queued; otherwise asks: a) new development, b) modify an existing implementation, c) resume an existing PBI |
| `develop 305912` | Gated | Same, for a PBI already in the tasks folder |
| `implement PBI 305912` | Gated | Finds the PBI (task folder or inbox), uses the open workspace as the repository when it is registered, loads repository memory, then as `develop` |
| `implement PBI 305912 in Auto Mode` | Auto | The whole PBI in one go inside the scope you authorize: analysis, gap map, spec, plan, test-first implementation, tests, analyzers/Sonar, memory update, G8 security gate, handover. Stops only for business ambiguity, destructive or production actions, permission changes, new dependencies, and anything outside the scope |
| `develop 305912 end to end` | Auto | The earlier name for Auto Mode |
| `plan 305912` | Plan | Gap map, spec, plan, and consistency gate; stops before any code |
| `attach C:\docs\notes.docx to 305912` | — | Adds supporting documents to an existing task |
| `review and align 305912` | Align | Code exists: compares it with siblings and the gap map, fixes deviations, validates |
| `debug <symptom>` | Debug | Root-cause fix with a regression test |
| `modify 305912` | Modify | Snapshots spec, plan, and statuses into the task history, takes in the change, and reopens the gates for the changed parts |
| `resume 305912` | Resume | Continues from the first gate neither approved nor waived, or from "Remaining" in plan.md |
| `Implement this requirement` | Gated / Auto | The requirement becomes the task's PBI; the repository profile is selected first, then the full pipeline in the mode in force |
| `Fix PR comments` | — | Verifies each comment against the code before changing anything, fixes what the code proves right, reruns build, tests, Sonar sources, and the pre-PR review, then drafts a short reply per comment |
| `review ... before changing anything`, `plan first` | — | Analysis only: issues, proposed file-by-file changes, business-rule impact, regression risk, test and Sonar plan; stops before any code change |
| `Review changes` | — | Read-only: pre-PR review of the branch, the diff, architecture, validation, API compatibility, tests, Sonar; reports Ready / Ready with Warnings / Not Ready |
| `Run tests` | — | The detected profile's test command (never another repository's solution) |

## Repository profiles: DAB and CDMS (3.6.0)
The agent file stays project-agnostic; repository rules live in `copilot\modules\profiles\` and are loaded only after the helper has identified the repository from its solution file (`profile -Repo <id>`):

| Profile | Repository | Detected when | Loads |
|---|---|---|---|
| `dab` | AN.ColorAppsAzure.BL | `AN.ColorAppsAzure.BL.sln` is in the repository | 7 DAB skills (architecture, CQRS and API contracts, FluentValidation, StrawberryShake GraphQL, DI, SonarCloud, testing) + DAB Sonar rules |
| `cdms` | ColorDMS | `AN.ColorAppsAzure.BL.ColorDMS.sln` is in the repository | 9 CDMS skills (architecture, add feature, XRM service, CQRS, DI, SonarCloud, XML documentation, run tests, coverage) + CDMS Sonar rules |

- Both profiles also load the shared **source-grounded development** and **pre-PR review** skills. Package, file, and code markers (StrawberryShake, CRM SDK, Scrutor, `CrmOperationBase`, ...) are evidence only: without the solution file the result is `UNCERTAIN` and no profile is applied; with both solutions it is `AMBIGUOUS` and the agent asks. Feature names (Formula, PaintSystem) are never used to decide.
- Isolation: the other profile's files are returned as `neverLoad`, and `guard -Repo <id>` refuses a `dotnet` command that names a solution not in that repository.
- `review -Repo <id> -Base main` checks the branch against the six recorded PR review findings (validation-exception conversion in PUT and POST functions that the middleware already handles, duplicate changes, a validator off the FluentValidation pattern, misplaced command/query dependencies, missing or outdated XML documentation), plus CDMS line length (200), broad catches, and `git diff --check`. Errors block **Ready**.
- Your own versions of any profile file go in `%USERPROFILE%\.akzo-agent\profiles-custom\<same relative path>`; the helper uses them instead of the shipped ones and upgrades never touch them. The shipped files were written from the 3.6.0 brief; the team's original agent instructions and SonarCloud catalogs belong there.
- The files are not placed in a Copilot skills or instructions folder on purpose: Copilot loads those for every agent and repository, which would apply DAB and CDMS rules everywhere.

## Fast development and review intelligence (3.7.0)
- **Only the files a request needs:** `profile -Repo <id> -Query "<request>"` loads the shared skills, the profile's core skills (architecture, testing, SonarCloud), its Sonar rules, and the skills whose topic matches the request; the others come back as `deferred` and are read when the work reaches them. Example: a DAB validator change loads FluentValidation but defers GraphQL and DI. `neverLoad` is unchanged. Detection is cached until HEAD, the working tree, or the profiles change; a corrupted or deleted cache is rebuilt.
- **Exact names first:** `context -Query "<ExactTypeOrFile>"` answers with `exact: true`, the file, its tests, and its direct callers. Free-text ranking returns the same results, 22-34% faster in the benchmark.
- **Staged tests:** `affected -Repo <id> [-Base main]` maps the change to its projects and returns A (build those projects), B (tests named after the changed classes), C (test projects that depend on them), and D (the full suite with coverage, once, at the final gate, never skipped). Solution, props, or package changes return `BUILD_WIDE` (full build and suite). Changed files with no matching test are listed.
- **Business rule branch coverage** (new shared skill): every approved rule is checked path by path (Yes/No, required fields, duplicates, state transitions, side effects, failures, transaction boundaries) against the code and tests; one implemented branch is not "existing".
- **Validation ownership and abstraction review** in the source-grounded skill, twelve reusable **review categories** and a disposition per PR comment in the pre-PR review skill, and a new `single-use-abstraction` review warning for a new helper/collaborator-style type with one consumer or none.
- **DAB:** there is no separate DAB plugin. Approved rules derived from CRM plugins, workflows, JavaScript, or web resources are implemented in DAB layers, with no CRM runtime dependency. **CDMS:** plugin-owned behavior stays in the plugin, and DAB conventions never apply.
- **Review-only or plan-first** requests stop after the analysis, with the proposed file-by-file changes, impact, risks, and test plan. Authorized implementation runs without extra prompts.
- Measured helper timings against 3.6.0: [benchmark/perf-3.7.0.md](benchmark/perf-3.7.0.md) (`scripts\perf-bench.ps1`).

## Task routing: small work stays small (3.8.0)
- **`start`** shows five choices: 1 Fix Issue, 2 Refactor Existing Code, 3 New Development / PBI, 4 Review / Analyze Only, 5 Custom Task (number or name). A task given directly (a Sonar finding, an error, a failing test, a PR comment, a PBI) is routed without the menu.
- **Fix Issue** runs locate -> read -> root cause -> smallest fix -> build -> focused tests, and reports Issue / Cause / Fix / Verification / Status. No PBI, spec, plan, or SDD documents. **Refactor** measures the smell, reuses existing abstractions, and never adds a single-use collaborator only to move a metric.
- **Only what the mode needs:** `profile -Task fix` loads the source-grounding skill and the Sonar rules (2 files instead of 7-8); `-Task refactor` adds the guides the smell triggers; New Development and Review keep the full lazy-loading set.
- **Why Copilot reads documents you did not ask for:** `guidance -Repo <id>` lists each repository instruction file, when the IDE adds it (every request, matching files, by description, when invoked), and every document it tells Copilot to read, marking unconditional reads with file and line. The agent no longer reads `copilot-instructions.md`/`AGENTS.md` itself (the IDE already adds them).

**Scoping repository instructions** (VS Code 1.143 / Copilot Chat 0.71 defaults, read from the installed product): `.github/copilot-instructions.md`, a root `AGENTS.md` or `CLAUDE.md`, and any `.instructions.md` with `applyTo: '**'` go into every request of every agent, the regular one included. Linked documents are not attached automatically (`chat.includeReferencedInstructions` is off), but a line such as "always read X before any task" makes the model read them. Keep those files to short rules for all tasks; move each document rule into `.github/instructions/<topic>.instructions.md` with an `applyTo` glob for the files it concerns, or into a skill with a description, worded "read X when <task or files>". This agent cannot unload those files; scoping them is the repository team's change. Visual Studio 2022 17.14 reads `copilot-instructions.md` and `.instructions.md` files but has no custom agents or skills.

## Change risk, test trace, and PR text (3.11.0)
- **Change risk:** `review -Repo <id> -Base main` also returns `risk`, each changed file with high, medium, or low and the reasons (profile rules in `profiles.json`, no matching test, more than 200 changed lines). Any high file means the full suite runs before G5.
- **Trace:** `trace -Id <PBI>` checks every acceptance criterion against the tests the plan names for it and their TRX results: COVERED, FAILING, NOT RUN, MISSING. G5 waits while one is MISSING or FAILING.
- **PR text:** `prtext -Id <PBI>` writes the pull request description to the task's evidence folder; you paste it.
- **Your own review rules:** `%USERPROFILE%\.akzo-agent\profiles-custom\review-rules.json`, a list of `{ id, profile, pathGlob, regex, severity, message }`, is checked by `review` on changed lines. The agent proposes a rule for a recurring PR comment; you decide whether to add it.
- **Coverage threshold:** `"quality": { "coverage": <percent> }` per repository in `repos.json`, copied from the SonarCloud gate; 80% when it is not set.

## How a PBI is analyzed
1. **Inputs:** the refined PBI is authoritative. Supporting documents are copied to the task (`supporting\`, originals untouched; duplicates detected; same names never overwritten) and listed in `inputs.json` with hash, type, guessed role (correct it with `role`), status, and source path. `.docx`, `.pdf` (with `pdftotext`), and `.xlsx` are converted once into cached text keyed by content hash; images and unknown formats are listed, never guessed at. The agent reads only the sections it needs.
2. **Architecture knowledge:** a per-repository index from `git ls-files` (stack, folders, files by component role: entry points, commands, queries, handlers, validators, services, repositories, models, contracts, UI, tests…), rebuilt only when HEAD moves; plus verified facts the agent records per repository and reuses in later PBIs.
3. **Gap map (G2):** every requirement gets an id (REQ-n), the documents that support or contradict it, a recorded multi-angle code search, and a class: EXISTING, REUSABLE, EXTEND, NEW (only after the full search), or UNKNOWN (not enough evidence), with `file:line` evidence and an evidence level (source, test, run). Conflicts between documents become questions, never silent choices.
4. **Spec (G3):** ambiguity scan, at most 5 questions, measurable acceptance criteria linked to requirements, existing behavior kept as "no change", entities and state transitions.
5. **Plan (G4):** tasks with requirement ids and a per-file action (reuse / modify / new) following the discovered folders and naming; a "reused without change" list; a consistency gate that blocks re-implementing anything classified EXISTING, REUSABLE, or EXTEND.
6. **Implement and validate:** test first; build → test → analyzers/Sonar → fix until 0 build errors, 0 newly failing tests, 0 new issues in changed files; never suppressions or skipped tests.
6a. **SonarQube and code rules:** code is written to a built-in rule set (complexity, unused code, error handling, literals, security, resources, async) and checked against the project's Sonar quality gate using the strongest source available: SonarQube MCP tools in chat, read-only server queries through the helper (gate, issues, hotspots, coverage, duplication, rule details; token from `SONAR_TOKEN`, never shown), Sonar analyzers in the build, or SonarQube for IDE. Coverage of changed lines is measured from the repository's own Cobertura or LCOV reports. Without a server it uses stated defaults (0 new issues, hotspots listed for review, changed-line coverage ≥ 80%, new duplication ≤ 3%). It never uploads an analysis unless asked. Setup: [installation-guide.html#sonar](installation-guide.html#sonar).
7. **Handover (G5):** requirement → class → acceptance criterion → evidence table, baseline vs final counts, open questions, proposed commit message. It never commits or pushes.

## Each PBI is its own task (3.5.0)
- Task folder `<tasks>\<repo-id>\<PBI-ID>\` and a spec folder unique per work item: `<specsDir>\<id>-<title>\` (e.g. `300186-create-order-export\spec.md`). Approvals, gate statuses, plan progress, and completion belong to that task only; a folder that belongs to another PBI is never reused.
- Before any file is created in the repository the agent shows repository, branch, HEAD, and uncommitted changes and asks: a) create a new feature branch (`feature/<id>-<title>`, recommended on a shared branch or one that belongs to another PBI; the base branch is confirmed, an existing name is detected), b) continue on the current branch, c) select an existing branch. It never stashes, resets, or discards changes; switching with uncommitted changes needs your agreement.
- A new PBI reuses architecture memory (after checking its evidence is current), not another PBI's progress; earlier tasks that look like the same feature are listed so the gap map checks whether the requirement is already satisfied instead of reimplementing it.

## Repository memory (3.4.0)
Per repository, under `%USERPROFILE%\.akzo-agent\maps\<repo-id>\`, never mixed between repositories and never uploaded:
- `architecture.json`, `component-index.tsv`, `symbols.tsv`: languages, frameworks (from manifests), inferred patterns, files by role, and the types, methods, functions, routes, and imports of C#, TypeScript/JavaScript, Python, Java/Kotlin, and Go (regex extraction, not a parser: good for finding and ranking, never proof).
- Every file is fingerprinted by content (git blob id), so committed and uncommitted edits, staged and untracked files, deletions, and renames are noticed on the next call and only the changed files are re-read.
- `facts.json`: verified facts (end-to-end patterns, reusable components, validation and testing conventions, integrations, decisions, commands, pitfalls), each with `path:line` evidence and the evidence's content fingerprint. A fact turns **stale** when its evidence changes (and so do facts that depend on it), and **expires** when the evidence file is gone or the policy's retention period passes. The agent must revalidate a stale fact before using it. Values that look like secrets are refused.
- `context -Query "<terms>" [-Mode compact|balanced|deep]`: ranked files with key symbols, linked tests, and matching facts within a size budget (balanced by default; set `retrieval.mode` and `retrieval.budgets` in `repos.json`). Ranked by deterministic matching, not proof: the agent reads the cited lines.
- Unreadable memory is set aside as `*.corrupt-<time>` and rebuilt. `purge -Repo <id> -Confirmed` deletes a repository's memory.

## Corporate security gate (G8, 3.4.0)
Mandatory in every mode, including Auto Mode; never waived. See [corporate-security-report.html](corporate-security-report.html) for the controls, test evidence, limitations, and the approvals still required.
- **Preflight (technical):** before a repository is indexed, documents are extracted, or content is output, the helper requires an authorization: an administrator policy (`%ProgramData%\AkzoAgent\policy.json`, or `policy.json` in the Akzo folder) that lists the repository, or the developer's own recorded statement (`authorize`, with the reference they hold and the data classification). Without one, `task` returns `NEEDS_AUTHORIZATION`; an unreadable or forbidding policy returns `BLOCKED_BY_POLICY`. Restricted data needs an administrator policy. A developer statement is reported as NOT VERIFIED, never as corporate approval.
- **Command guard:** the agent checks every non-helper terminal command with `guard`: ALLOW, NEEDS_APPROVAL (git commit/push, dependency installs, deployments, database changes, destructive deletes, analysis uploads), or BLOCK (external transfer, remote code execution, security-tool tampering, execution-policy and registry changes, elevation, credential access). A blocked command is never run and no approval releases it. Checks are logged to the task's evidence with secrets redacted. The guard is run by the agent: Copilot offers no hook that forces it, so the log shows what the agent reported (see the report's limitations).
- **Network:** the helper's only network call is the read-only Sonar query, and only to a host the policy or authorization lists.
- **Data protection:** secret-looking values are redacted from logs, notes, and output; memory and task folders are restricted to the owner, SYSTEM, and Administrators; retention and `memory: disabled` come from the policy.
- **Gate (`security -Id <PBI>`):** 13 controls with PASS / FAIL / NOT VERIFIED / NOT APPLICABLE and the evidence for each, written to `<task>\evidence\corporate-security-report.html`. FAIL blocks the handover; NOT VERIFIED allows DONE_WITH_CONCERNS at best; only PASS allows DONE. Automated verification is never corporate approval.
- **Release integrity:** `scripts\make-release.ps1` builds the zip with `release-manifest.json` (SHA-256 of every file) and a `.sha256` file; `install.ps1` refuses a package whose files differ. Hashes show the package is unchanged, not who made or approved it.

**Approval limits:** the gates are enforced by the agent's instructions and by the statuses recorded in spec.md and plan.md (the helper reports them on every call, so a resumed session respects them). Copilot offers no hook that blocks file edits, so the IDE cannot technically stop a model error or an explicit instruction from skipping a gate.

## Where files go
| What | Where |
|------|-------|
| Chosen PBI and supporting documents (before intake) | `%USERPROFILE%\.akzo-agent\inbox\<name>\` (`refined-pbi.md`, `source\`, `supporting\`) |
| Task workspace | `%USERPROFILE%\.akzo-agent\tasks\<repo-id>\<PBI-ID>\` (`refined-pbi.md`, `supporting\`, `inputs.json`, `.extract\`, `evidence\`) |
| spec.md, plan.md | `<repo>\<specsDir>\<FeatureName>\` when the repo has a spec folder, otherwise in the task workspace |
| Project knowledge | `%USERPROFILE%\.akzo-agent\maps\<repo-id>.md` (verified facts) and `maps\<repo-id>\architecture.json`, `component-index.tsv` |
| Code changes | your repository, uncommitted, visible in Git Changes |
| Registry, backups | `%USERPROFILE%\.akzo-agent\` |

## Upgrading from 3.3.0 or earlier
Run **Install / Upgrade** (or `install.ps1`) from the new package; nothing else is needed for tasks, `repos.json`, inputs, or specs. Two things change, both by design:
1. **Authorize once per repository.** The first time the agent works on a registered repository after the upgrade, `task` returns `NEEDS_AUTHORIZATION` and the agent asks whether you hold an authorization (reference and data classification); it records your answer. Administrators can instead place `policy.json` (see the report). *Add repository* indexes a new repository only after it is authorized.
2. **Memory is rebuilt once.** The old `architecture.json` is replaced by the new format on first use; your notes in `maps\<repo-id>.md` are kept and shown as unverified hints. Roll back by installing the previous package's zip over this one (backups of replaced files are in `%USERPROFILE%\.akzo-agent\backups\`).

## What install, upgrade, and repair do
- Copies the agent to `%USERPROFILE%\.copilot\agents` (VS Code) and `%USERPROFILE%\.github\agents` (Visual Studio 2026), and the helper and templates to `%USERPROFILE%\.akzo-agent\modules`.
- **Idempotent:** unchanged files are skipped; a rerun never duplicates anything. **Repair** restores missing or modified files and cleans leaked Akzo text from Copilot memory.
- **Upgrade/migration:** files from earlier versions that are no longer shipped are removed via the install manifest; an older `akzo-sdd` agent found without a manifest entry is removed too (only if it is an Akzo agent file). Existing tasks keep working: their manifest is built on the next `task` call.
- **Backups and rollback:** every file replaced or removed is first copied to `%USERPROFILE%\.akzo-agent\backups\<timestamp>\` (deep paths listed in `index.txt`). If a write fails, every file changed by that run is restored.
- **Repositories:** *Add Repository…* (or `-AddRepo`; `-RegistryOnly` skips the install; `-EditRepo`, `-RemoveRepo`, `-ValidateRepo`, `-ListRepos`) registers a folder in `repos.json` as given (backed up first; a folder that is not a Git clone only with a warning; never the same folder twice). Remove deletes only the registration; editing a path or removing withdraws the authorization recorded for that id. It uses an existing spec folder (`openspec/changes`, `src/.github/specs`, `.github/specs`, `docs/specs`, `specs`), and builds its first architecture index. Nothing is written into your repositories by the installer.
- **VS Code (optional, on by default in the window):** a marked settings block auto-approves only the helper's own commands (a `;` is allowed only inside a quoted path list), grants read access to the Akzo folders, and raises the agent step limit to 400.
- `Verify` also warns when Visual Studio 2026 18.5+ (needed for user-level agents) is not installed.

## Other agents keep their normal behavior
The agent's rules live only in its agent file. Nothing is written to `AGENTS.md`, `.github/copilot-instructions.md`, `.github/instructions`, prompt files, or skills: Copilot applies those to **every** agent. If another agent still behaves like the Master agent: (1) **Copilot memory** — VS Code loads the first 200 lines of user memory into every agent; **Repair** (`-CleanMemory`) removes memory files that mention Akzo after a backup, `Verify` reports them as `LEAK`; (2) **same chat** — switching agents keeps the history, start a new chat; (3) **leftover agent files** — `Verify` reports them, Install migrates them.

## Develop and test this package
- `scripts\make-release.ps1` — builds `Akzo-Agent-<version>.zip` with `release-manifest.json` and a `.sha256` file. `scripts\ab-bench.ps1` — interleaved timing of up to three helper versions on one repository (results in `benchmark\interleaved-3.4.1.md`). `scripts\benchmark.ps1` — timing of the previous package against this one on a generated repository (results in `benchmark\results-3.4.1.md`; single-run cold and warm figures are superseded by the interleaved ones; model tokens and Copilot interactions are not measurable here).
- `scripts\test.ps1` — 386 checks in a temporary profile (repository memory on C#, Angular, and Python fixtures, incremental indexing, facts, retrieval, G8 preflight, command guard, scanning, policy, purge, release integrity, Sonar against a local stand-in server, and changed-line coverage included): static agent checks (incl. project-agnostic wording, gates, gap map), install, upgrade/migration, rerun, repair, rollback, AddRepo with first discovery, repository add/edit/remove/validate and installation-status checks, memory-leak cleanup, VS Code settings and auto-approve rule, intake, `pick` with supporting documents, `attach`, `role`, manifest, duplicate/missing/unsupported/second-primary detection, `.docx`/`.xlsx` extraction and cache reuse, change detection, interrupted-processing recovery, architecture index (roles, refresh on new commit, per-repository isolation), launchers, uninstall. Never touches your real profile.
- `scripts\test-gui.ps1` — drives the real windows through the `.cmd` launchers (PBI dialog, multi-select supporting dialog, setup window, uninstall confirmation) in a temporary profile. Needs a desktop session.

## Ownership and licensing
- Original implementation by Arun RG. No third-party code, prompts, or documentation are bundled; see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
- License: **not yet selected.** It will be chosen once ownership and organizational requirements are confirmed. Until then, no license is granted.
