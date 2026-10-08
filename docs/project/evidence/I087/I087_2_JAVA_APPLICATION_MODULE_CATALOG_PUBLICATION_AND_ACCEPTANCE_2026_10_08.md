# I087-2 — Java application module catalog: verified publication and acceptance evidence

**Date:** 2026-10-08.  
**Implementation Issue:** [I087/#841](https://github.com/guillermomolina/protos/issues/841).  
**Approved architecture:** [PLAT055/#844](https://github.com/guillermomolina/protos/issues/844), closed/completed after exact owner approval “apruebo A1”.  
**Design record:** [PLAT055 approved catalog contract](../../decisions/platform/PLAT055_JAVA_APPLICATION_MODULE_CATALOG.md).  
**Implementation commit:** [`guillermomolina/protos@011e14353eda636d33c7bbba3910980e9c05bb35`](https://github.com/guillermomolina/protos/commit/011e14353eda636d33c7bbba3910980e9c05bb35).  
**Implementation version:** `0.3.307-SNAPSHOT`; **normative specification:** `0.1.451` unchanged.  
**Last accepted baseline before the decision:** `6dab6ecc08c9a2102a388e00908c710a15cf2a4c` / `0.3.298-SNAPSHOT`. Multiple unrelated, concurrent product commits occurred between the baseline and the exact I087 publication; they are not attributed to this slice.

## Outcome and provenance

**I087-2 product publication: VERIFIED.** The named implementation commit was read directly from GitHub main with its complete nine-file changed-path inventory. The public API, Context/Process integration, resolver, JUnit test class and portable probe were inspected from the exact published revision. GitHub main at the acceptance check equalled the implementation commit `011e14353eda636d33c7bbba3910980e9c05bb35`.

**Maintainer-reported validation:** The project owner explicitly reported `git diff --check` CLEAN and **all local tests PASS** after stating the I087-2 implementation was pushed. The published implementation changelog additionally records that the focal Java embedding tests, integrated `make test` and portable embedding smoke **passed**. The previous agent/human handoff stated portable build/validation should use `python3 dist/build_portable.py --allow-dirty` followed by `sh dist/validate_portable.sh` when uncommitted changes prevented standard validation. The coordinator did **not** execute these commands and did **not** inspect raw validation logs. Do not promote these reported PASS claims into independently reproduced CI evidence, or assert that the coordinator proved command exactness.

**Native scope:** The native CLI and a native-compiled Java Polyglot embedding host are different surfaces. This slice provides the public API for Java hosts on the JVM; **native-hosted Java Polyglot embedding was not implemented or tested**. No parity or speedup claim.

## Exact nine-file product delta

1. `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedModules.java` — new public Java `install(Context, Map<String,String>)` host entry, input validation before Context mutation and entered Context access via `initialize/enter/leave`.
2. `src/main/java/com/guillermomolina/protos/execution/ProtosApplicationModuleResolver.java` — new exact `app:` catalog resolver with immutable defensive copy; `fromCharacters` source loading and standard resolver fallback; absent/empty catalog returns the existing standard resolver instance.
3. `src/main/java/com/guillermomolina/protos/execution/ProtosLanguageContext.java` — Context-local catalog state, synchronization with `embeddedProcessLock`, pre-bootstrap admission, driver-owned Context rejection in both directions. The additional guard in `bindHostExecutionContextForRuntime` was **expressly approved** by the owner: prevent silently ignoring installed modules if a host driver tries to claim that Context.
4. `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedProcess.java` — compose catalog-over-standard resolution at existing lazy Process bootstrap. Existing `ProtosModuleRuntime`, `ProtosStandardActorProtocol` and `ProtosActorBootstrap` remain unchanged.
5. `src/test/java/com/guillermomolina/protos/execution/ProtosEmbeddedModulesTest.java` — **16 `@Test` methods**: exact/cached identity, cycles, failure/retry, authorization, Actor isolation and own bootstrap, standalone `Context.eval`, invalid/duplicate/late/closed installation, initialization/bindings/parsing, concurrent bootstrap, Context/Engine isolation, no ambient authority, close, immutable copy and lazy loading, absent/empty catalog pay-as-you-grow.
6. `dist/Plat054EmbeddingProbe.java` — external Java application's `application-modules` probe imports `app:worker`, calls `Actor.spawn("app:worker", "start", 41)`, requests `increment` and verifies `42` plus absence of `filesystem` and `network` host bindings.
7. `dist/smoke_polyglot_embedding.sh` — invokes the new `application-modules` mode using only extracted portable `lib/protos.jar` and runtime JARs outside the checkout and emits `PLAT055_APPLICATION_MODULES_PORTABLE: PASS` on successful execution.
8. `pom.xml` — product version `0.3.307-SNAPSHOT`.
9. `CHANGELOG.md` — I087-2 features, acceptance claim and Native-hosted embedding non-goal under `0.3.307-SNAPSHOT`.

The changes preserve the exact `app:<nonempty>` canonical source key, fail-closed missing specifier, `std:` and foreign domain separation, one Context-local immutable catalog, same module runtime, Actor-local cache/semantics, and the Process's existing standard hosting topology. No normative specification files changed.

## I087 acceptance matrix

| Criterion | Evidence | Finding |
|---|---|---|
| Exact approved design | PLAT055/#844, approved A1 and 2026-10-08 durable ratification | PASS, approved/ratified |
| Public Java install boundary | New `ProtosEmbeddedModules.java`, portable Java consumer | IMPLEMENTED, inspected |
| No global/foreign module registry or implicit path lookup | `ProtosApplicationModuleResolver`, standard fallback and exact key checking | IMPLEMENTED, inspected |
| Import, canonical identity and Actor-local cache | Same existing `ProtosModuleRuntime`; test methods for cache/cycles/retry | COVERED, owner reports PASS |
| Actor destination own bootstrap and isolation | Unchanged Actor bootstrap plus new destination tests and portable request/response | COVERED, owner reports PASS |
| Standalone eval/Source identity | Dedicated JUnit regression | COVERED, owner reports PASS |
| Bad/late/duplicate configuration, races and driver binding | Context locks and registration tests; owner-approved new driver guard | COVERED, owner reports PASS |
| Other Contexts, same Engine, closing | Cross-Context/Engine and close tests | COVERED, owner reports PASS |
| No guest Filesystem/Network authority | Focused tests; external Java probe checks absence in no-grant Context | COVERED, owner reports PASS |
| PAY AS YOU GROW | Catalog absent/empty returns original resolver; no eager Process in install | COVERED, owner reports PASS |
| Local JVM/full regression validation | Owner reports local tests PASS; changelog explicitly states focal and `make test` PASS | HUMAN-REPORTED PASS |
| Portable distributed JAR | Probe compiles against extracted runtime JARs in a different directory, now includes `application-modules`; changelog states smoke PASS | HUMAN-REPORTED PASS; STATIC PATH VERIFIED |
| Native Image implications | Native-hosted Java Polyglot embedding explicitly out of scope | CLASSIFIED, NOT CLAIMED |
| Product SHA/version/durable record | Product commit and this immutable evidence publication | VERIFIED |
| Native issue hierarchy | I087/#841 native parent I086/#840; PLAT055/#844 native child, closed | VERIFIED; issue closure is a separate GitHub mutation |

The new Java configuration API follows host authority already permitted in `spec/semantics/MODULES.md`; `spec/concurrency/ACTORS.md` §8 and `spec/io/PROCESS_IO.md` remain normative and unchanged. The additional driver binding guard is fail-closed and within A1's Context ownership invariant. **GITHUB021 consistency: PASS** for A1's owner-approved invariants; no new ratification choice needed.

## Closure and follow-on

Product behavior, tests and distribution smoke are accepted on the **human-reported** validation standard, with read-only inspection of publication. No missing feature is identified within I087's bounded scope. **Do not allocate a speculative I087-3 or reopen completed PLAT055/#844 or I086/#840.** After this durable evidence is re-read, the Issue coordinator may close I087/#841 as `completed` if live native hierarchy and Issue status still satisfy closure rules; record closure on the Issue itself. Derived GitHub Project synchronization is automation-owned, not independently verified here.

Follow-on work belongs to independently tracked Issues such as LM010/#493, LM012/#671, PERF034/#845 or PERF035/#846, respecting their live state; PERF032/#831 remains `status:paused` and must not be silently resumed.

```text
ISSUE=I087/#841
SLICE=I087-2
PRODUCT_REVISION=011e14353eda636d33c7bbba3910980e9c05bb35
PRODUCT_VERSION=0.3.307-SNAPSHOT
SPEC_REVISION=0.1.451
DECISION=PLAT055/#844_RATIFIED_CLOSED
OWNER_DIFF_CHECK=CLEAN_REPORTED
OWNER_ALL_LOCAL_TESTS=PASS_REPORTED
FOCAL_AND_MAKE_TEST=PASS_REPORTED_IN_PRODUCT_CHANGELOG
PORTABLE_SMOKE=PASS_REPORTED_IN_PRODUCT_CHANGELOG
PORTABLE_PROBE_STATIC_WIRING=VERIFIED
NATIVE_HOSTED_JAVA_EMBEDDING=NOT_TESTED_NOT_CLAIMED
AGENT_EXECUTED_TESTS=NO
GITHUB021=PASS
ISSUE_CLOSE_ALLOWED_AFTER_RECORD_AND_NATIVE_CHECK=YES
```
