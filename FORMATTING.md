# Java formatting

Projects inheriting `sarto-org-pom` automatically format maintained Java source at
Maven's `validate` phase, before compilation and tests. `mvn verify` therefore fixes
formatting locally and in CI instead of failing because of whitespace.

Spotless 2.43.0 runs Google Java Format 1.35.0 and removes unused imports. Use JDK 21
or newer. Each module covers its own `src/main/java` and `src/test/java`; generated,
vendored and upstream directories are excluded. Additional source roots require an
explicit module configuration. SpotBugs remains a separate bug analysis tool.

Run `mvn spotless:apply` for formatting alone. Review and commit the resulting source
changes with your work. CI formats its checkout and tests that result; it does not
push commits automatically. `mvn spotless:check` remains available for a deliberate
read-only audit, but is not a formatting-only build gate.

Override `spotless.version` or `google-java-format.version` centrally when upgrading.
Use `-Dspotless.skip=true` when a build must leave the checkout untouched. For a
module containing immutable upstream Java, set that property in the module POM or
override the Java includes/excludes instead of reformatting the upstream snapshot.

Consumers receive this behavior when they resolve the updated parent. Unrelated
standalone builds and explicit `spotless:check` steps are not changed by a parent
update and should be migrated when those projects adopt this policy.
