# I163 — D253-C1 explicit Core Clock implementation and publication evidence

**Date:** 2026-11-09  
**Formal work item:** [I163 #1102](https://github.com/guillermomolina/protos/issues/1102)  
**Native design parent:** [D253 #1101](https://github.com/guillermomolina/protos/issues/1101), verified via GitHub `/parent` and `/sub_issues` on 2026-11-09.  
**Owner-approved design:** [D253-C1 ratification](../../decisions/language/D253_EXPLICIT_CORE_WALL_CLOCK_NANOSECONDS_C1_RATIFIED_2026_11_09.md) and [approval/research audit](../D253/D253_C1_OWNER_APPROVAL_GITHUB010_GITHUB021_2026_11_09.md).  
**PROTOS_REVISION:** `d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18`  
**Implementation version:** `0.3.459-SNAPSHOT`  
**Global specification revision:** `0.1.501`  
**Published product commit:** https://github.com/guillermomolina/protos/commit/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18  
**Downstream:** [I153 / RealWorld #1068](https://github.com/guillermomolina/protos/issues/1068); native GitHub `blocked_by` relationship to I163 verified on 2026-11-09.

This is revision-bound **implementation evidence**, not a new design approval, independently executed test report, physical clock accuracy certification, deployed RealWorld authentication result or verification of Native executable parity. The prior ratification is a historical design snapshot from before the product implementation; its then-pending implementation and native-parent statements describe that earlier checkpoint, not the present GitHub/product state.

## Published source and specification

The exact GitHub product commit changes **54 files**, with **3,093 additions and 116 deletions**:

- [`protos/lib/core/clock/Clock.protos`](https://github.com/guillermomolina/protos/blob/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18/protos/lib/core/clock/Clock.protos): authority-free, frozen, source-backed Core protocol exposing only synchronous `epochNanoseconds()`.
- [`spec/io/CLOCK.md`](https://github.com/guillermomolina/protos/blob/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18/spec/io/CLOCK.md): normative authority, exact negative and large Integer epoch-nanoseconds, wall-clock limitations, synchronous errors, lifecycle revocation, Actor/P non-transferability and explicit Instant composition.
- [`ProtosClockHostFacility.java`](https://github.com/guillermomolina/protos/blob/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18/src/main/java/com/guillermomolina/protos/execution/facilities/ProtosClockHostFacility.java), [`ProtosClockCapabilityValue.java`](https://github.com/guillermomolina/protos/blob/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18/src/main/java/com/guillermomolina/protos/runtime/io/clock/ProtosClockCapabilityValue.java), [`ProtosClockBackend.java`](https://github.com/guillermomolina/protos/blob/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18/src/main/java/com/guillermomolina/protos/runtime/io/clock/ProtosClockBackend.java) and [`ProtosHostClockBackend.java`](https://github.com/guillermomolina/protos/blob/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18/src/main/java/com/guillermomolina/protos/execution/host/io/ProtosHostClockBackend.java): genuine private host grant, authenticated access, one `InstantSource` sample and exact `BigInteger` conversion through the existing semantic Integer normalization. Per-grant synchronization serializes provider-read admission with retirement; no Process-wide monitor is held across the provider call.
- The Core bootstrap, Prelude and Process runtime install the optional `clock` only in the initial RootActor's private module context. A failed bootstrap does not retire a Clock belonging to another Process.
- [`ProtosCli.java`](https://github.com/guillermomolina/protos/blob/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18/src/main/java/com/guillermomolina/protos/cli/ProtosCli.java), the standalone/service host paths, workspace/package drivers and standard Polyglot embedding preserve independently authorized `--allow-clock` and `protos.AllowClock`, defaulting to absent while retaining existing Java overload compatibility.
- Actor transfer and isolated P explicitly reject Clock grants; completed Integer samples use the ordinary transfer rules. The product changelog records a synchronous `NonParallelValue` outcome instead of leaking an internal exception when rejected P input is used.
- Normative companion updates: `spec/io/PROCESS_IO.md`, `spec/io/IO_CORE.md`, `spec/concurrency/ACTORS.md`, `spec/concurrency/PARALLEL_EXECUTION.md`, `spec/semantics/MODULES.md`, and the [spec changelog](https://github.com/guillermomolina/protos/blob/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18/spec/PROTOS_SPEC_CHANGELOG.md) at `0.1.501`.
- Added deterministic Clock authority/backend/transfer/embedding/CLI/service tests, retained architecture and packaged-Core guards, documentation for Process authority, CLI, Java embedding, datetime, JWT and logging, and [the opt-in example](https://github.com/guillermomolina/protos/blob/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18/protos/examples/clock/current-time.protos).
- The [I163 JVM/Native smoke source](https://github.com/guillermomolina/protos/blob/d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18/scripts/i163-clock-smoke.py) is published. **Publication of a script does not demonstrate that it was executed.**

The approved Protos-first public composition is:

```protos
Instant: import("std:datetime/instant")
now: Instant(clock.epochNanoseconds())
```

The operation returns exact mathematical epoch nanoseconds in a semantic Integer. It is **not** a monotonic timer, deadline facility, attested timestamp, background task or `Future`. `JWT.issue` and `JWT.verify` continue to receive a caller-provided genuine Instant; no RealWorld or JWT API modification is part of this commit.

## Validation provenance and acceptance boundaries

| Gate or evidence | Outcome and provenance |
| --- | --- |
| Six Clock-specific focal JUnit suites | **PASS**, maintainer reported before broader validation |
| `git diff --check` | **PASS**, maintainer reported after product push |
| `make test` | **PASS**, maintainer reported on local checkout |
| `make check` | **PASS**, maintainer reported on local checkout |
| Product push to `guillermomolina/protos` | **VERIFIED**, exact public GitHub commit `d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18` |
| Implementation version and spec revision | **VERIFIED IN PRODUCT**, `0.3.459-SNAPSHOT` and `0.1.501` |
| Native D253 → I163 parent relationship | **PASS**, verified via live GitHub REST in both directions |
| I153 `blocked_by` I163 | **PASS**, verified via live native dependency endpoint |
| Rebuilt JVM `scripts/i163-clock-smoke.py` execution | **NOT REPORTED**, no executable output supplied |
| Rebuilt/installed Native CLI Clock smoke and parity | **NOT REPORTED / NOT VERIFIED**; the implementation Issue explicitly includes Native acceptance |
| Test ordering relative to the metadata bump | **NOT INDEPENDENTLY VERIFIED**; the product revision contains the version/changelog change |
| Direct docs-repository `git diff --check` | **NOT EXECUTED** by this connector-driven publication |

The local green full-suite reports are recorded without inventing test counts, Maven output, Native passes, installed binary identity or an independent CI run.

## Current coordination interpretation

- **D253 design:** owner-approved C1, durably ratified with exact GITHUB010/GITHUB021 evidence; native child relationship now exists. Its own design outcome is eligible for formal closure independently of the downstream implementation's final Native acceptance.
- **I163 implementation:** code, specs, examples and human-reported full-suite validation are published. **The full acceptance clause for rebuilt Native Clock parity remains unverified**; do not label that gate PASS or silently treat the Issue as fully accepted.
- **I153 RealWorld:** the reusable language Clock is now available in published product source, so a Protos-first integration may use `Instant(clock.epochNanoseconds())`. This commit does **not** prove RealWorld JWT deployment, live expiry behavior or a Native production end-to-end result.

```text
FORMAL_IDENTIFIER=I163
DESIGN=D253-C1
PROTOS_REVISION=d0251c3d8ca0d9393b57ddf860e3ab9b7965ca18
IMPLEMENTATION_VERSION=0.3.459-SNAPSHOT
SPEC_REVISION=0.1.501
FILES_CHANGED=54
FOCAL_TESTS=PASS_OWNER_REPORTED
GIT_DIFF_CHECK=PASS_OWNER_REPORTED
MAKE_TEST=PASS_OWNER_REPORTED
MAKE_CHECK=PASS_OWNER_REPORTED
PRODUCT_PUSH=PASS_GITHUB_VERIFIED
NATIVE_PARENT=PASS_GITHUB_VERIFIED
I153_NATIVE_BLOCKED_BY=PASS_GITHUB_VERIFIED
JVM_CLOCK_SMOKE=NOT_REPORTED
NATIVE_CLOCK_SMOKE=NOT_REPORTED
I163_FULL_ACCEPTANCE=PENDING_NATIVE
```
