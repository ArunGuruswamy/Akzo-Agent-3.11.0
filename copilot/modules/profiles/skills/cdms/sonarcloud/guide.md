# CDMS SonarCloud

> Profile skill `cdms-sonarcloud`: CDMS SonarCloud rules, repository policies, and how their status is reported. Load only when the helper's profile is cdms.

Shipped baseline from the 3.6.0 brief (see `profiles/README.md`). Applies only when `profile` reports `cdms`.

- Rule catalog: `instructions/cdms/sonar-rules.instructions.md`, or its profiles-custom replacement (the team's CDMS
  SonarCloud rules). Apply it while writing code.
- CDMS requirements beyond the common categories: optional-parameter guidance (S2360), parameter names consistent between
  interface and implementation, lines of at most 200 characters, the CDMS XML documentation policy, and at least the
  repository's threshold (default 80%) coverage of changed code where measurable. `review -Repo <id>` checks line length, XML documentation, and broad catches
  on changed lines.
- Some of these are repository policy rather than SonarCloud rules. Check the analyzer configuration and the SonarCloud
  quality profile (`sonar -Repo <id>`, `-Check rule -Key <key>` when a server is configured) and report each as a Sonar
  finding or as a policy finding.
- A local build does not show that SonarCloud passes. Without a server result write "SonarCloud: not verified - <reason>".
- Never suppress, exclude, or downgrade a rule to pass.
