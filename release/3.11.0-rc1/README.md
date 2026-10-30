# Akzo Master SDD Agent 3.11.0 release candidate

Akzo SDD Agent â€” Developed and maintained by Arun RG

| File | What it is |
|---|---|
| `Akzo-Agent-3.11.0.zip` | The package: the same 57 files as the source at tag `v3.11.0-rc1` (checked file by file against the tag), plus `release-manifest.json` (SHA-256 of every file). `install.ps1` refuses a package whose files differ or whose manifest names another version. |
| `Akzo-Agent-3.11.0.zip.sha256` | Checksum of the zip |

SHA-256 of the zip: `0FD3A8588912DE38FEC56E6F6FEA2EBBC6E091A87D8A9EF4885D362F43BE49BA`

Check it with `Get-FileHash Akzo-Agent-3.11.0.zip -Algorithm SHA256`. A hash shows the file is unchanged since it was built, not who built or approved it.

What changed: `review` rates each changed file high, medium, or low risk with reasons (rules in `profiles.json`); `trace` checks every acceptance criterion against the tests the plan names and their TRX results (COVERED, FAILING, NOT RUN, MISSING); `prtext` writes the pull request description; `review` also reads your own rules from `profiles-custom\review-rules.json`; `coverage` reports the repository's threshold (`quality.coverage`, default 80%); `-Verify` reports renamed copies of the agent. See `CHANGELOG.md`.

Validation: 386 regression checks and 14 window checks passed (run from Windows PowerShell). The zip installed into an empty profile and verified (package integrity OK, version 3.11.0 everywhere); an install made from the 3.10.0 zip upgraded and verified with existing task files unchanged.

Status: release candidate. Corporate readiness is NOT VERIFIED; see `corporate-security-report.html`. Live Copilot behavior, the DAB and CDMS SonarCloud gates, and TRX files from real DAB and CDMS runs are NOT VERIFIED (not available on this machine).