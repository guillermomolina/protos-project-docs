# I163 — JVM and rebuilt Native Clock executable acceptance

**Date:** 2026-11-09  
**Formal implementation:** [I163 #1102](https://github.com/guillermomolina/protos/issues/1102), under [D253 #1101](https://github.com/guillermomolina/protos/issues/1101)  
**Owner-approved contract:** D253-C1  
**PROTOS_REVISION:** `d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18` ([published product commit](https://github.com/guillermomolina/protos/commit/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18))  
**Implementation version:** `0.3.459-SNAPSHOT`; **normative specification revision:** `0.1.501`  
**Prior revision-bound implementation record:** [I163 publication evidence](I163_D253_C1_EXPLICIT_CORE_CLOCK_IMPLEMENTATION_PUBLICATION_2026_11_09.md)

This **later acceptance addendum** supersedes the prior record's *historically accurate* "Clock executable smoke not reported" checkpoint. It does not rewrite that earlier snapshot. The user's uploaded local build/smoke transcript, reproduced as decisive result markers below, is direct maintainer-provided evidence rather than a log executed by a GitHub connector. The exact public product revision is independently verified on GitHub. The transcript identifies version `0.3.459-SNAPSHOT` and `/workspaces/protos`, but the visible portion does not include the output of `git rev-parse HEAD`; exact provenance of the local built binary by a directly printed SHA is not asserted.

## Maintainer-executed acceptance

The owner supplied a terminal transcript from the Protos development checkout with these outcomes:

| Gate | Evidence in transcript | Result |
| --- | --- | --- |
| JVM `make build` | Maven shaded `target/protos-0.3.459-SNAPSHOT.jar`; `BUILD SUCCESS` | **PASS** |
| JVM executable identity | `bin/protos --version` prints `Protos 0.3.459-SNAPSHOT` | **PASS** |
| JVM `scripts/i163-clock-smoke.py ./bin/protos --mode jvm` | `I163_JVM_EXECUTABLE_SMOKE=PASS` | **PASS** |
| Native `make build-native` | GraalVM CE 25.5.5.0.3 / Java 25; `target/native/protos` executable generated (55.75 MiB); `BUILD SUCCESS` | **PASS** |
| Native executable version diagnostic | `PROTOS_HOME="$PWD" target/native/protos --version` prints `Protos development` | **Observed** — not a SNAPSHOT version-identity assertion |
| Native `scripts/i163-clock-smoke.py ./target/native/protos --mode native` | `I163_NATIVE_EXECUTABLE_SMOKE=PASS` | **PASS** |
| Clock focal tests | Owner previously confirmed green six-class focal batch | **PASS (owner-reported)** |
| `git diff --check` | Owner previously confirmed clean check after publication | **PASS (owner-reported)** |
| `make test` | Owner previously confirmed full local tests green | **PASS (owner-reported)** |
| `make check` | Owner previously confirmed full local checks green | **PASS (owner-reported)** |
| Standalone `make test-native` (Native CLI broader regressions plus Native TLS integration) | No execution result supplied for this command | **NOT REPORTED** |

The [published smoke source](https://github.com/guillermomolina/protos/blob/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18/scripts/i163-clock-smoke.py) probes authority absent by default, explicit Clock grant, independence from Network/Entropy, genuine semantic Integer, construction of `Instant`, synchronous invalid-arity signaling, duplicate CLI flag rejection, unauthorised service mode, and runtime wall-time samples over fresh executable process launches. A PASS marker means the script's checks passed for the respective JVM or **rebuilt Native** executable. It does **not** certify physical time accuracy, clock monotonicity, anti-rollback security, standalone `make test-native`, or RealWorld production deployment.

## Formal completion scope

The source, normative contract, tests, docs, independent host/Process authority and Actor/P isolation were published in the verified [product revision](https://github.com/guillermomolina/protos/commit/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18). The owner reports full `make test` / `make check` PASS and has now supplied both requested **JVM** and **newly rebuilt Native Clock-specific** smoke results. This satisfies the outstanding I163 executable-parity acceptance gate; I163 may be marked **completed** under the formal Issue closure-evidence procedure.

The Native binary's own `--version` text is `Protos development`, and this record makes no stronger version-identification assertion for that binary; the build transcript does show it is the newly generated `target/native/protos` in the `0.3.459-SNAPSHOT` Maven build.

The downstream [I153 RealWorld #1068](https://github.com/guillermomolina/protos/issues/1068) is a separate application undertaking and remains open until its own completion evidence is available. It may now proceed using the published Protos-first `Instant(clock.epochNanoseconds())` and existing JWT APIs. The native `blocked_by` edge is a project coordination relation, not evidence that I153 itself is complete.

```text
FORMAL_IDENTIFIER=I163
DESIGN=D253-C1
PROTOS_REVISION=d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18
IMPLEMENTATION_VERSION=0.3.459-SNAPSHOT
SPEC_REVISION=0.1.501
JVM_BUILD=PASS_OWNER_LOG
JVM_EXECUTABLE_SMOKE=PASS_OWNER_LOG
NATIVE_BUILD=PASS_OWNER_LOG
NATIVE_EXECUTABLE_SMOKE=PASS_OWNER_LOG
GIT_DIFF_CHECK=PASS_OWNER_REPORTED
MAKE_TEST=PASS_OWNER_REPORTED
MAKE_CHECK=PASS_OWNER_REPORTED
MAKE_TEST_NATIVE=NOT_REPORTED
LOCAL_BINARY_COMMIT_SHA=NOT_PRINTED_IN_SUPPLIED_EXCERPT
I163_CLOCK_EXECUTABLE_ACCEPTANCE=PASS
I163_FULL_ACCEPTANCE=PASS
```
