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
