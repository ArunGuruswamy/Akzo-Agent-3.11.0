# Plan: <WORK-ITEM-ID> <title>

Status: Draft | Approved (by <who>, <date>) | Waived (Auto Mode, authorized by <who>, <date>) | In progress | Done  
Spec: ./spec.md · Task folder: <path> · Evidence: <task folder>/evidence/

## 0. Start state and resume
- Mode: gated | auto | plan · Authorization: <who, date, "plan approved" or the Auto Mode record below>
- Auto Mode authorization: <authorized by <who> on <date>; scope: repository <id>, branch <branch>, the files and behavior of this plan, existing dependencies only, no contract, data, or permission change beyond the spec, no commit or push> or "not used"
- Repo: <id> · Branch: `<branch>` · HEAD: `<sha>` · Pre-existing changes (do not disturb): <paths or none>
- PBI baseline recorded: <date> · PBI changes reconciled: <date: summary>
- Remaining: <next task or phase>

## 1. Approach
- Selected approach:
- Architecture followed (architecture index rows, sibling features, reference implementation — cite paths):
- Why (architecture fit, existing patterns reused — cite paths):
- Alternatives rejected:

## 1a. Consistency gate (spec <-> plan, before code)
| ID | Category (coverage, duplication, ambiguity, inconsistency, guidance) | Severity | Location | Fix / status |
|----|------------------------------------------------------------------------|----------|----------|--------------|
| C1 | | | | |

## 2. Tasks
Order matters. Every file listed was found in the repo (no invented paths); new files name the existing pattern they copy.
Task: a descriptive title taken from the work itself (e.g. "Implement reference color lookup"); people see this title, the T id stays internal. Objective: what the task delivers and why the PBI needs it.
Task status values: Not Started | In Progress | Implemented - Verification Pending | Blocked: <reason> | Verified | Completed (Completed only when its checks ran and passed).

| T | Task | Objective (what and why) | REQ | Repo | File | Action (reuse / modify / new) | Evidence (`path:line`, or sibling copied) | Depends on | ACs | Status |
|---|------|--------------------------|-----|------|------|-------------------------------|--------------------------------------------|------------|-----|--------|
| T1 | | | REQ-1 | | | | | — | AC1 | Not Started |

Reused without change (components used as they are; no edit planned):
| Component `path` | REQ | Used by task |
|------------------|-----|--------------|

## 3. Code generation
- <e.g. regenerate the generated client with the repo's verified command; never hand-edit generated files> or "None".

## 4. Validation
Quality baseline (before the first edit) and final result:
| Gate | Command | Baseline | Final |
|------|---------|----------|-------|
| Build errors / warnings (changed files: IDs) | | | |
| Tests failing (names) | | | |
| Analyzer issues in changed files (compiler, Sonar S-rules, lint) | | | |

### Sonar (agent section 6a)
Source: <(a) Sonar tools in chat · (b) helper server query · (c) analyzers in build/lint · (d) IDE problems · (e) scanner run (on request) · (f) static review against the code rules, not a Sonar analysis>  
Gate: <project gate from server | defaults: 0 new issues, hotspots reviewed, changed-line coverage >= the repository's threshold (default 80%), new duplication <= 3%>

| Item | Baseline | Final | Evidence |
|------|----------|-------|----------|
| Quality gate (status, failing conditions) | | | |
| New issues in changed files (by severity) | | | |
| Security hotspots to review | | | |
| Coverage of changed lines (`coverage` command) | | | |
| Duplicated lines in new code | | | |
| Issues fixed (rule, `path:line`) / false positives raised | | | |

| V | Command (from registry, verified) | Scope | Result |
|---|-----------------------------------|-------|--------|
| V1 | | narrow: affected test project | not run |

## 5. Risks and rollback
| Risk | Mitigation | Rollback |
|------|------------|----------|

## 6. Progress and evidence
- <date> T1 done — <files changed>, V1 passed (`<command>`, <n> tests)
- Unavailable checks: <check — reason>

Requirement -> test -> code matrix (handover). `trace` reads the Test column: one or more `file::name` per row, separated by commas.
| REQ | Class | AC | Test (file::name) | Code (`path:line`) | Result (passed / failed / not run - reason) |
|-----|-------|----|-------------------|--------------------|---------------------------------------------|

Memory updated: stored <fact ids> · re-verified <ids> · retired <ids> · none
