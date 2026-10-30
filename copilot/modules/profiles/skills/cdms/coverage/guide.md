# CDMS coverage

> Profile skill `cdms-coverage`: CDMS changed-code coverage target and how it is measured and reported. Load only when the helper's profile is cdms.

Shipped baseline from the 3.6.0 brief (see `profiles/README.md`). Applies only when `profile` reports `cdms`.

- Target: at least the repository's threshold (default **80%**) line coverage of changed code where it can be measured:
  the server gate's coverage condition when SonarCloud answers, else `quality.coverage` in the registry, else 80%.
- Measure: run the tests with `--collect:"XPlat Code Coverage"` writing to the task's evidence folder, then
  `coverage -Repo <id> -Report "<evidence>\...\coverage.cobertura.xml"`. It returns coverable, covered, and uncovered
  changed lines and partly covered branches per file.
- Report the measured line coverage and the partly covered branches as the helper reports them. Never estimate or round up
  a number, and never report coverage for a run that did not finish.
- Below the target: add tests for the uncovered new behavior. Never exclude code from coverage or lower the target.
- SonarCloud's new-code coverage, when a server is configured, is reported separately and named as such.
