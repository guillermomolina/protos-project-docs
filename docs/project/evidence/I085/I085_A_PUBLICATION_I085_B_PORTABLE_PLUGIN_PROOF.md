# I085-A publication — external provider SPI / I085-B separate-process proof

Date: 2026-10-08

Live I085: https://github.com/guillermomolina/protos/issues/839

Audit parent: https://github.com/guillermomolina/protos/issues/818

## Product publication verified

~~~text
PRODUCT_REPOSITORY=guillermomolina/protos
I085_A_COMMIT=b1b86e46d84a07cdbd8aa405277d534fc3731fd8
I085_A_SUBJECT=I085-A: add mechanism-independent external foreign provider SPI
IMPLEMENTATION_VERSION=0.3.282-SNAPSHOT
I085_A_GIT_MAIN_VERIFIED=YES
I085_A_LOCAL_ALL_TESTS=PASS_OWNER_REPORTED_NOT_REEXECUTED
I085_A_LOCAL_GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED_NOT_REEXECUTED
PRODUCT_SOURCE_CHANGES_BY_DOCS_AGENT=NONE
~~~

GitHub commit changes include the public
`com.guillermomolina.protos.spi.foreign` package, opaque `Object` handles,
provider-defined value operations, a host-configured `ServiceLoader` and
dedicated `URLClassLoader`, immutable runtime-host registry integration,
`ProtosForeignProviderConfiguration.trustedInProcess`,
`ProtosPolyglotRuntimeHost.openWithForeignProviders`, and
`ProtosCli --foreign-provider-path`. No GraalPy/GraalJS/NumPy or
foreign-language-specific Maven dependencies were added.

The product change also includes `ProtosExternalForeignProviderTest` and a
focused authority regression in `ProtosForeignProviderEnforcementTest`.

## Already-covered tests — do not duplicate

The current `ProtosExternalForeignProviderTest` builds separately compiled
plugin JAR fixtures with `META-INF/services`. The fixture's compilation
classpath contains only public SPI class files, not internal Protos execution
classes or a foreign runtime. It checks:

- default host: no providers/discovery/foreign class loader, unknown scheme fails;
- invented `inventado:demo` importing successfully and calling `double(21)`
  yields 42, through opaque Java handles (not Polyglot Value/TruffleObject);
- repeated imports and Actor-specific facades/sessions, nontransferable
  foreign values, session lifetime and close invalidation;
- a second independent scheme and non-eager initialization of its session;
- duplicate provider ID/scheme, reserved scheme, unknown options, empty and
  nonexistent provider paths rejected;
- Protos CLI flag through `new ProtosCli().run(...)` **inside the JUnit JVM**.

These are real code tests already included in the published I085-A commit.
The owner's all-tests-PASS statement covers the local suite; no tests were
independently run by the coordinating agent.

## Remaining verification gap — I085-B

Test an *independent packaged consumer*, not another in-process fixture:

1. Build an exact-current-revision portable JVM distribution with the existing
   `make dist` workflow; extract the ZIP to a temporary install directory.
2. Compile an external Java provider with `javac` against the exact installed
   portable `lib/protos.jar` public SPI, never against source classes,
   internal package classes, JUnit classes or the test classpath.
3. Create an external provider JAR containing `META-INF/services`, kept
   **outside** both Protos' extracted install tree and its libraries.
4. Execute the extracted installation's `bin/protos` through a **fresh
   operating-system process**. With no `--foreign-provider-path`, the invented
   scheme must fail; with explicitly supplied JAR, ordinary import/member
   execution must succeed and print the expected value.
5. Add a second external provider fixture using JDK 25 FFM to call the actual
   operating-system C library `strlen` symbol on the supported Linux test
   host. The only native call is inside the *external provider artifact*.
   Test real library invocation (not a hardcoded/Java approximation) and the
   host-selected `TRUSTED_IN_PROCESS` boundary; no sandbox claims.
6. Ensure non-plugin Protos execution, no eager other-provider sessions, no
   special Protos source/import/POM dependency, and correct error/output/exit
   behavior. Artifacts and distribution extraction remain temporary.
7. Package this as a reproducible bounded smoke-test implementation in
   `guillermomolina/protos`, preferably one dedicated Python stdlib runner
   under `dist/`, not broad production refactoring. No new package manager,
   no online downloads, no NumPy, no hardcoded new language.

This test-only next slice must reuse the *published* I085-A classes and tests.
Any genuine discovered product defect should be reported specifically and
patched minimally under the same issue only after failing evidence; do not
reopen the architecture.

## Status

~~~text
I085_A_STATUS=PUBLISHED
I085_B_STATUS=NEXT_TEST_PROOF_NOT_YET_EXECUTED
I085_PARENT_STATUS=OPEN
FOLLOWUP_KIND=TEST_ONLY_IMPLEMENTATION
FOLLOWUP_REPOSITORY=guillermomolina/protos
FOREIGN_LIB_DEPENDENCIES_IN_PROTOS=NONE
NATIVE_PLATFORM_PROOF=LINUX_JDK25_LIBC_FFM
NUMPY_REQUIRED=NO
PROTOS_SOURCE_MODIFICATIONS_FOR_NEW_EXTERNAL_PROVIDER=NO
~~~

No I085-B test result or I085 closure is asserted here.

## AI assistance disclosure

Prepared with AI assistance from the live GitHub product commit,
`ProtosExternalForeignProviderTest`, CLI/loader/SPI code, distribution
launcher/build description, I085 authority, and the maintainer's test report.
Published evidence is descriptive, not independently re-executed proof.

## Owner clarification — real Protos tests and strictly zero POM changes

The owner tightened I085-B acceptance after the initial test-only allocation:
**`pom.xml` must not be modified at all**, including no extra
test-scope Maven dependencies and no version bump in this test-only slice.
`CHANGELOG.md` is likewise left untouched in this bounded test slice.
A provider may use only APIs already shipped with GraalVM/JDK 25 or external
test-owned fixture code. The selected native proof is the Java 25 Foreign
Function & Memory API (part of JDK 25), calling the actual Linux libc
`strlen` symbol from an independently packaged *external provider JAR*.

Crucially, the gate must include **programs and assertions authored in the
Protos language**, not just Java/JUnit or Python harness checks. Add actual
`.protos` cases under `protos/tests/foreign-provider/`, executed against
the extracted portable distribution with the external plugin opt-in and
checked for successful assertions/output. Include a separate negative
no-provider `.protos` case or a deterministic missing-provider failure
case. No new special test-only language syntax.

Reproducible smoke workflow:

1. Use the existing portable distribution builder and extract its ZIP to a
   temporary directory. Use the exact packaged `lib/protos.jar` for SPI
   client compilation, never the repository implementation/test classpath.
2. Independently compile an external **plain Java opaque-object** plugin
   and a second external **JDK 25 FFM/native libc** plugin with the
   platform `javac`, package `META-INF/services`, keep them outside the
   extracted Protos distribution.
3. Spawn the extracted `bin/protos` in separate OS processes. Check no
   implicit plugin discovery, explicit `--foreign-provider-path`,
   successful Protos-language assertions on imported plugin values,
   scalar/member-call behavior, repeat imports, real `strlen` value,
   error cases, printed PASS markers and exit status.
4. Native proof is a Linux/JDK25 gate, not an assumed universal JVM/Native
   Image behavior; on unsupported OS/JDK report `UNSUPPORTED` and never
   claim the FFM test passed. Explicitly trusted in-process native code is
   not sandboxed by this SPI.
5. No regular `make test-protos` manifest entry that would require a plugin
   to be magically installed. The black-box test harness owns running these
   opt-in Protos programs. Preserve the existing I085-A tests without
   duplicating their fixture design.

~~~text
I085_B=TEST_ONLY_IMPLEMENTATION
I085_B_REPOSITORY=guillermomolina/protos
I085_B_SOURCE_PROGRAMS=PROTOS_LANGUAGE_TESTS_REQUIRED
I085_B_SOURCE_TESTS_ACTUALLY_EXECUTED=REQUIRED
I085_B_BLACK_BOX_PORTABLE=REQUIRED
I085_B_FRESH_JVM_OS_PROCESS=REQUIRED
I085_B_EXTERNAL_JAR=REQUIRED
I085_B_FFM_JDK25_LIBC=LINUX_GATE
I085_B_POM_EDIT=FORBIDDEN
I085_B_CHANGELOG_EDIT=FORBIDDEN
I085_B_ADDITIONAL_DEPENDENCIES=NONE
I085_B_NUMPY_OR_GRAALPY_INSTALL=NO
I085_B_GRAALJS_OR_GTK_QT_INSTALL=NO
I085_B_CURRENT_STATUS=AWAITING_IMPLEMENTATION_AND_MAINTAINER_EXECUTION
~~~

The earlier historical I085-B note's lack of Protos-language programs is
superseded by this explicit owner requirement. There is **no claim** that
the Protos black-box tests or libc/native smoke have been run yet.
