# I086 / PLAT054-3E3 — Default Filesystem Polyglot embedding implementation publication

Date: 2026-10-08  
Product owner: [I086 / #840](https://github.com/guillermomolina/protos/issues/840)  
Platform contract: [PLAT054 / #838](https://github.com/guillermomolina/protos/issues/838)  
Blocker closure: B011

## Verified product identity

Published HEAD: [`guillermomolina/protos@6dab6ecc08c9a2102a388e00908c710a15cf2a4c`](https://github.com/guillermomolina/protos/commit/6dab6ecc08c9a2102a388e00908c710a15cf2a4c).

Exact commit message: `I086 PLAT054-3E3: default Filesystem in the standard Polyglot embedding`.

Implementation version: **`0.3.298-SNAPSHOT`**. Normative specification remains **`0.1.451`**; no normative spec files changed.

The independently read commit changes exactly eight paths:

1. `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedFilesystemCustody.java` (new);
2. `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedProcess.java`;
3. `src/main/java/com/guillermomolina/protos/execution/ProtosStandardFilesystemProtocol.java`;
4. `src/test/java/com/guillermomolina/protos/execution/ProtosEmbeddedFilesystemTest.java` (new);
5. `src/test/java/com/guillermomolina/protos/execution/ProtosEmbeddedNetworkTest.java`;
6. `docs/guide/11-process-io-filesystems-and-authority.md`;
7. `pom.xml`;
8. `CHANGELOG.md`.

No change to `guillermomolina/protos-benchmarks`, PERF032/PERF033 harness, I087 module resolver, or existing normative specification is implied by this publication.

## Implemented behavior and source verification

The published `ProtosEmbeddedProcess.bootstrap()` now provisions `ProtosEmbeddedFilesystemCustody.provisionOrNull(Env)` ahead of initial guest module execution; the optional `ProtosFilesystemValue` marker is passed through the existing `ProtosStandaloneProcessBootstrap`, then `ProtosStandardFilesystemProtocol.installOperations(...)` installs the already-standardized operations on the initial RootActor-bound capability before guest code can see the marker. The existing independent Network grant remains governed by effective socket access.

The new custody implementation:

- checks effective `Env.isFileIOAllowed()`; **no permission yields no Filesystem capability** or FS custody;
- derives its base via `Env.getCurrentWorkingDirectory()` in the configured `TruffleFile` provider namespace and rejects an unsafe/unrepresentable initial base with host-facing `ProtosEmbeddingException`, satisfying **spec 0.1.451 Alternative A / bootstrap abort**;
- uses provider-backed `TruffleFile` operations for `open`, `entries`, `replace`, and `remove`, including provider-relative virtual and custom providers rather than ambient Java NIO filesystem access;
- owns channel/resource lifecycle and revocation, including Process termination and Context close;
- does not create an open-resource set until first successful File acquisition (PAY AS YOU GROW);
- has explicit race-sensitive open-or-create, atomic-move and selected-resource logic in the implementation and corresponding regression tests.

The source changes reuse the canonical `ProtosStandardFilesystemProtocol` and existing I/O/Future flows, rather than a second guest-facing Filesystem API.

The newly published `ProtosEmbeddedFilesystemTest` includes coverage for permission/thread/socket combinations; valid and invalid working-directory bootstrap; no implicit Filesystem on subsequent host entries; opened resource identity; create/truncate/positioned I/O; `entries`, `replace`, `remove`; virtual/restricted/read-only and rejecting providers; selected namespace-race cases; Context independence; cancellation, close and resource revocation; and no-grant/unused-grant resource laziness. `ProtosEmbeddedNetworkTest` reconciles its old Filesystem-absent assumption with effective file permission. These are verified published tests; this record does not claim independent coordinator execution of them.

## Validation provenance

The maintainer explicitly reports:

```text
GIT_DIFF_CHECK=CLEAN
ALL_LOCAL_TESTS=PASS
COMMIT_AND_PUSH=PUBLISHED
```

GitHub HEAD, changed-file set, version, changelog and source/test presence were inspected independently through read-only access. No raw test logs, independent test execution or Native Image results were provided in this handoff.

## B011 and I086 state

**B011 = CLOSED — owner-approved normative contract and positive implementation both published.** The design choice was approved in FS0, normative `0.1.451` published in 3E3-SPEC (product `1d6d537d79d69fae3ad1e613cefd76b7a97abe38`), and the actual default provider-aware Filesystem runtime is now published at `6dab6ecc08c9a2102a388e00908c710a15cf2a4c`. This is the recorded objective closure condition for B011.

**B012 remains CLOSED** following the earlier default Network implementation. Host Future.value and Core embedding implementation remain published.

**I086/#840 remains OPEN** pending final standard-embedding acceptance: assess the already existing tests and focused distribution smoke gates, ensure Native Image and portable distribution validation on exact product revision, reconcile residual documented acceptance criteria, and publish final provenance before closure. Do not manufacture a new functionality phase merely from historical descriptions.

**PLAT054/#838** remains a ratified architectural decision, with its Issue closure requiring separately governed native hierarchy/project postcondition verification. Do not close it merely by closing B011. **PERF033/#832 is completed**; **PERF032/#831** has its own profiling-first optimization scope, independent of Filesystem and I086 final acceptance.

## Next handoff

```text
NEXT_SLICE=I086-FINAL
TYPE=IMPLEMENTATION_ACCEPTANCE
REPOSITORY=guillermomolina/protos
AGENT=EDITOR_AND_CONFORMANCE_REVIEW
HUMAN_EXECUTOR=YES
NO_UNJUSTIFIED_SOURCE_EDIT=YES
NO_REDO_GREEN_TESTS_WITHOUT_RELEVANT_CHANGE=YES
NATIVE_IMAGE=UNVERIFIED
PORTABLE_DISTRIBUTION=UNVERIFIED
```

A final-acceptance slice may be evidence/validation-only with **zero source modifications** if existing test and packaging coverage satisfy the current published I086 contract. If a genuine gap is proven, implement only the focused correction and have the human rerun the minimum necessary affected gates. Preserve the normal human-executor version/changelog ordering for any actual implementation changes.

## Evidence and assistance

Prepared with ChatGPT assistance from public GitHub repository HEAD, commit patch, source/test coverage, and the maintainer's explicit validation report. This does not prove every provider's behavior dynamically, assert that Native Image or portable distribution gates already passed, claim benchmark parity, or change the normative language specification.

```text
ISSUE=I086/#840
SLICE=PLAT054-3E3-FILESYSTEM
PRODUCT_COMMIT=6dab6ecc08c9a2102a388e00908c710a15cf2a4c
PRODUCT_VERSION=0.3.298-SNAPSHOT
SPEC_REVISION=0.1.451
HUMAN_TESTS=ALL_LOCAL_PASS_REPORTED
HUMAN_GIT_DIFF_CHECK=CLEAN_REPORTED
B011=CLOSED_NORMATIVE_AND_RUNTIME_PUBLISHED
B012=CLOSED
NEXT=I086-FINAL_ACCEPTANCE
NATIVE_IMAGE=UNVERIFIED
PORTABLE_DISTRIBUTION=UNVERIFIED
PERF033=COMPLETED_SEPARATE
PERF032=OPEN_SEPARATE
```
