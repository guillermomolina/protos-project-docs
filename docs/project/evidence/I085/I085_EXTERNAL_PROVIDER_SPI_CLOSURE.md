# I085 closure — external foreign-provider SPI and distributed Protos integration

Date: 2026-10-08

- Product issue: https://github.com/guillermomolina/protos/issues/839
- Audit parent (research, already closed): https://github.com/guillermomolina/protos/issues/818
- Prior work: I082/#830 (runtime/substrate), LIB021/#835 (explicit std:interop), both closed
- Distinct ongoing work: I086 (standard Polyglot embedding, **not** part of I085)
- I085-A published: https://github.com/guillermomolina/protos/commit/b1b86e46d84a07cdbd8aa405277d534fc3731fd8
- I085-B published: https://github.com/guillermomolina/protos/commit/4f459f2de119d956b8671ad1c1da7b9fa6bcfa17

## Closure scope

The completed deliverable is the **general optional external-foreign-provider
infrastructure and an integration test proving it can be consumed in deployed
Protos**. This is not a claim that every possible foreign-language module or
operating-system library already has a published ready-made plugin.

I085-A adds mechanism-independent, public `spi.foreign` interfaces with opaque
provider-owned `Object` handles and provider-defined operations, loaded only
from explicit host-selected external paths/JARs via `ServiceLoader`. The
RuntimeHost's default foreign registry remains empty. D188/D189/PLAT052/PLAT053
runtime, identity, Actor session, authority and lifetime rules remain the
shared substrate. External plugins are selected as trusted in-process by
the current explicit configuration; neither an external Java JAR nor its FFM
calls are sandboxed.

I085-B is published on product `main` at
`4f459f2de119d956b8671ad1c1da7b9fa6bcfa17` and wires
`make test-protos` to the new `test-protos-foreign` phase plus the
existing ordinary Protos test suite. It adds:
- `dist/test_external_foreign_providers.py`, which extracts a portable JVM
  distribution, independently compiles external service-provider JARs against
  its packaged `lib/protos.jar` public SPI, keeps the plugins **outside** the
  installation, and runs the extracted `bin/protos` in fresh OS processes;
- actual `.protos` programs under `protos/tests/foreign-provider/`
  with normal Protos `std:test/Assertions`, including
  `opaque-provider.protos`, `jdk-math.protos`, `jdk-date.protos`,
  `native-ffm-provider.protos`, `multiple-providers.protos`, handled
  and unhandled `missing-provider` programs, and `unused-provider.protos`;
- external test plugin sources under that test tree:
  `inventado:demo` with an opaque Java handle,
  `jdk:math` and `jdk:date` with real JDK
  `java.lang.Math`/`java.time.LocalDate` operations, and
  `oslib:libc` with real JDK 25 FFM downcall to Linux `strlen`;
- positive ordinary import, member invocation and scalar results, repeat
  import identity, negative missing-provider and Java argument/date failures,
  multiple-provider registration, zero ambient discovery, unused-provider
  session laziness, and proof that the extracted installation stays unchanged.

## Maintainer-reported validation

The maintainer reports on 2026-10-08:

~~~text
I085_B_COMMIT=4f459f2de119d956b8671ad1c1da7b9fa6bcfa17
LOCAL_ALL_TESTS=PASS_REPORTED_BY_MAINTAINER
LOCAL_GIT_DIFF_CHECK=CLEAN_REPORTED_BY_MAINTAINER
TEST_RESULTS_INDEPENDENTLY_REEXECUTED_BY_COORDINATOR=NO
I085_PARENT_CLOSURE_SCOPE=EXTERNAL_PROVIDER_INFRASTRUCTURE_AND_INTEGRATION
~~~

The new runner emits named outcomes
(`PROTOS_OPAQUE_PROVIDER`, `PROTOS_JAVA_MATH_PROVIDER`,
`PROTOS_JAVA_DATE_PROVIDER`, `PROTOS_NATIVE_FFM_PROVIDER`, etc.).
Its native gate **may report `UNSUPPORTED`** when the environment is
not Linux x86_64/JDK 25, without failing the entire runner. The user's
aggregate all-tests-PASS statement therefore does **not**, on its own,
independently establish `PROTOS_NATIVE_FFM_PROVIDER=PASS`.
The real FFM implementation and runnable Protos-native gate are published,
but no invented native-specific execution result is recorded.

## Explicit discrepancy with the test-only edit constraint

The owner had required **zero changes to `pom.xml` or `CHANGELOG.md`**
for I085-B. The published commit in fact edits both:
- `pom.xml`: **only** version
  `0.3.284-SNAPSHOT` → `0.3.285-SNAPSHOT`;
- `CHANGELOG.md`: adds the `0.3.285-SNAPSHOT` I085-B entry.

This is a real deviation from the literal no-edit directive and is
**not** described as compliant. GitHub's published commit shows no
foreign-language or FFM Maven dependency added and no change to product
runtime Java code. The deviation is recorded transparently; no product
repository files are changed by this evidence/issue coordinator.

## Boundaries not claimed closed

- No built-in GraalPy, GraalJS, NumPy, Espresso, GTK or Qt provider was
  shipped. Future real providers remain independently installable external
  artifacts; every language/module needs appropriate acquisition logic.
- A retained GUI/native-thread callback ingress facility (for GTK/Qt event
  loops) is **not** implemented by I085, and D189's current baseline does
  not authorize arbitrary asynchronous entry.
- This validates the portable **JVM** distribution, not dynamic plugin
  installation into an arbitrary Native Image build.
- Explicit trusted in-process loading carries host/native authority; it is
  not a security sandbox.
- I086 standard Polyglot Context.eval/bindings embedding remains separate
  and must not be closed as a consequence of I085.

## Resolution

I085-A and I085-B supply the agreed generic substrate, independently
installable provider SPI, deployed-program acceptance tests and opt-in cost
model. Close I085/#839 as completed on the maintainer's published commit
and reported local green validation. Keep its semantic and distribution
limitations visible instead of creating a fictitious "all foreign
libraries supported out of the box" guarantee.

~~~text
I085_A=PUBLISHED
I085_B=PUBLISHED
I085_ISSUE=CLOSED_COMPLETED
I085_AUD019_PARENT=ALREADY_CLOSED
EXTERNAL_PROVIDER_SPI=COMPLETE_FOR_JVM_DISTRIBUTION
PROTOS_LANGUAGE_INTEGRATION_CASES=ADDED
JAVA_JDK_LIBRARY_FIXTURES=ADDED
REAL_LINUX_LIBC_FFM_FIXTURE=ADDED
I085_B_FFM_EXECUTION_INDIVIDUAL_OUTCOME=NOT_PROVIDED
FOREIGN_MAVEN_DEPENDENCIES_ADDED=ZERO
I085_B_POM_AND_CHANGELOG_UNCHANGED=NO_VERSION_AND_CHANGELOG_UPDATED
NATIVE_IMAGE_DYNAMIC_PLUGINS=NOT_PROVEN
ALL_FUTURE_LIBRARY_PLUGINS_BUILT_IN=NO
~~~
