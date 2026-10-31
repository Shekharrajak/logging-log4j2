<!-- SPDX-License-Identifier: Apache-2.0 -->
# IOStreams metadata validation

This branch adds verification to the two-file change on `experiment/4034-annotation-metadata`. The producer POM and changelog are identical on both branches.

With JDK 17 and Gradle 8.14.6, run from the repository root:

```bash
./mvnw -pl :log4j-iostreams -am '-Dtest=org.apache.logging.log4j.io.*Test' -Dsurefire.failIfNoSpecifiedTests=false verify
gradle -p src/it/gradle-metadata --refresh-dependencies --rerun-tasks check
```

The Maven command runs the focused IOStreams tests and reactor build checks. The separate Gradle fixture checks the generated metadata, compiles strict classpath and modular consumers, and writes the resolved graphs to `src/it/gradle-metadata/target/dependency-graph.json`.

Expect two direct OSGi annotation dependencies in IOStreams `apiElements`, and no direct annotation dependencies in `runtimeElements`. The inherited annotations can still arrive through `log4j-api`; a smaller resolved classpath or compilation speedup is not assumed.

Observed for this prototype on Corretto 17.0.18: 142 IOStreams tests passed, with one intentionally skipped H2 test; both consumer compilation modes and metadata checks passed. The upstream and candidate graphs contained the same nine compile artifacts and two runtime artifacts.

To compare with upstream, build API and IOStreams in a separate upstream checkout, then run this same fixture with `-Pbaseline -Plog4jProducer=/path/to/upstream -PconsumerBuildDirectory=/tmp/log4j-4034-baseline-consumer`. That run expects the original five direct annotation dependencies. Compare the two `dependency-graph.json` files.

`-Plog4jRepository=/path/to/repository` can instead select an already captured publication. `-Plog4jVersion=...` selects another producer version.
