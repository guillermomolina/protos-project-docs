# TEST008 — post-closure validation entry-point reconciliation

Date: 2026-10-05

Owning work item: `TEST008 / guillermomolina/protos#761`

Related platform decision: `PLAT047 / guillermomolina/protos#786`

Triggering compilerability work: `TEST009 / guillermomolina/protos#795`

## Exact publication

~~~text
PROTOS_REVISION=28d057d61f40be32a215b15e2ee463700709028c
PARENT_REVISION=cf00ece1efdfc4e1156918873c4ef7623d57d827
COMMIT_SUBJECT=TEST008: separate check and test validation
IMPLEMENTATION_VERSION=0.3.204-SNAPSHOT
~~~

This is a post-closure TEST008 reconciliation. TEST008 remains closed; the
published change corrects the developer validation contract exposed while
TEST009 compilerability work was using `make check` and `make test`.

## Trigger

Before this publication:

~~~text
make check
  -> compilerability / PE guards
  -> make test

make test
  -> test-java
  -> test-protos

local slow-test regression verdict
  -> advisory

local slow-test configuration/environment ERROR
  -> fail-closed
~~~

That topology caused two independent problems during TEST009-I validation:

1. `make check` unexpectedly entered the full functional test suite even though
   TEST009 needed an independent compilerability/bailout validation surface.
2. semantically green Java tests could still abort `make test` solely because
   the slow-test normalization environment was classified
   `ENVIRONMENT_NOT_COMPARABLE`.

The maintainer explicitly selected a stronger operational contract:

~~~text
make check = compilerability / PE / bailout validation only
make test = functional Java + Protos validation only

slow-test guard = telemetry / diagnostics
slow-test verdict never owns make test exit status

actual Java test failure = nonzero
actual Protos test failure = nonzero
~~~

## Published Makefile contract

At the exact revision above:

~~~text
check:
  toolchain
  check-local-range-index-pe
  check-local-range-operands-pe
  check-local-accessor-pe
  check-bytecode-api-pe
  check-generated-bytecode-bci-pe
  check-truffle-compilation

CHECK_DEPENDS_ON_TEST=NO
CHECK_DEPENDS_ON_TEST_JAVA=NO
CHECK_DEPENDS_ON_TEST_PROTOS=NO
STRICT_TRUFFLE_COMPILATION_IN_CHECK=YES
~~~

The strict Truffle compilation gate is therefore the final bailout authority of
`make check`; the multi-minute textual diagnostic remains manual and is not
part of the aggregate.

Functional validation remains:

~~~text
test: test-java test-protos
~~~

## Slow-test telemetry authority

The slow-test machinery remains present and continues to collect current-run
evidence, run controls, classify suspects, perform bounded confirmation and emit
machine-readable diagnostics.

However, its setup and verdict commands are now non-authoritative with respect
to `make test`:

~~~text
slow-test reset failure = diagnostic / non-blocking
slow-test controls failure = diagnostic / non-blocking
slow-test policy regression = diagnostic / non-blocking
ENVIRONMENT_NOT_COMPARABLE = diagnostic / non-blocking
BASELINE_PENDING = diagnostic / non-blocking
CONFIGURATION_ERROR in slow-test verdict = diagnostic / non-blocking
~~~

The actual Java phases are unchanged in authority:

~~~text
test-java-parallel failure = nonzero / blocking
test-java-serial failure = nonzero / blocking
child Maven/Make status propagation = preserved
~~~

`test-protos` likewise remains fail-closed on the actual Protos test process.

The classification architecture and reviewed baseline are retained. This change
does not make a failing run a passing measurement; it changes only whether
slow-test telemetry owns functional validation's process status.

## Structural regression protection

`tools/java_slow_test_guard_selftest.py` now pins the Makefile contract:

~~~text
JAVA_SLOW_TEST_MODE legacy switch absent
test = test-java + test-protos
slow-test reset/controls/check are non-authoritative
real Java phases remain authoritative
check has no test/test-java/test-protos prerequisite
check includes check-truffle-compilation
~~~

This protects both halves of the owner-selected separation against accidental
reintroduction.

## Validation provenance

Before metadata publication, the maintainer reported the complete requested
validation green:

~~~text
SLOW_TEST_GUARD_SELF_TEST=PASS
MAKE_TEST=PASS
MAKE_CHECK_CONTRACT=PASS
ALL_REQUESTED_VALIDATION=PASS
~~~

The metadata update to `pom.xml` / `CHANGELOG.md` was then performed under
the repository rule that forbids rerunning tests after the version/changelog
bump.

~~~text
TESTS_AFTER_METADATA_BUMP=NONE
PRODUCT_PUBLICATION=PASS
VERSION=0.3.204-SNAPSHOT
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

## Superseded enforcement statement

This owner-selected 2026-10-05 policy supersedes only the enforcement-authority
part of the earlier TEST008/PLAT047 amendments:

~~~text
SUPERSEDED:
  LOCAL_TEST008_GUARD=AUTHORITATIVE_FAIL_CLOSED

CURRENT:
  LOCAL_TEST008_SLOW_TELEMETRY=ADVISORY_DIAGNOSTIC_ONLY
  CI_TEST008_SLOW_TELEMETRY=ADVISORY_DIAGNOSTIC_ONLY
  FUNCTIONAL_TEST_EXECUTION=AUTHORITATIVE_FAIL_CLOSED
~~~

Candidate H's measurement/classification machinery, reviewed constants, no
automatic baseline growth, no timeout/skip semantics and no weakening of actual
test failures remain unchanged.

## Closure state

~~~text
TEST008_STATE=REMAINS_CLOSED
PLAT047_STATE=REMAINS_RATIFIED_WITH_2026_10_05_OWNER_AMENDMENT
TEST009_WORKFLOW_BLOCKER=RESOLVED
~~~

AI assistance: this record was drafted with ChatGPT from the exact published
product revision, the maintained TEST008/PLAT047 records and maintainer-reported
validation.
