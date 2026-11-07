# BUG035 — Canonical Bytes prototype in Actor and Group value transfer (0.3.452)

**Evidence date:** 2026-11-07. **Work item:** [BUG035 #1081](https://github.com/guillermomolina/protos/issues/1081). **Affected application:** [I153 RealWorld #1068](https://github.com/guillermomolina/protos/issues/1068).

This is a **non-normative publication and validation record**, not a change to Actor or Bytes semantics. The normative owners remain `spec/concurrency/ACTORS.md` §§16–18 and `spec/io/BYTE_IO.md` §3 in `guillermomolina/protos`.

## Verified Protos publication

- **PROTOS_REVISION:** [`bfb276f9b76c663d3ef1fb68823e13a22ede136c`](https://github.com/guillermomolina/protos/commit/bfb276f9b76c663d3ef1fb68823e13a22ede136c).
- **COMMIT_TITLE:** `BUG035: share canonical Bytes prototype in Actor value transfer`.
- **IMPLEMENTATION_VERSION:** `0.3.452-SNAPSHOT`, verified against [`pom.xml`](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/pom.xml) and [`CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/CHANGELOG.md) at that exact revision.
- **PUBLISHED_DELTA:** 12 product paths: two production Java files, two Java test/fixture files, two new guest conformance sources, two conformance manifests, two worker modules, and `pom.xml` plus `CHANGELOG.md`. GitHub independently exposes the revision and path list.
- **SPEC_CHANGE:** None. The existing immutable standard-prelude sharing exception is applied to the previously unrecognized hidden Bytes prototype.

## Fault and bounded repair

A standard `Bytes` instance created through `Encoding.Latin1.encode` delegated to the hidden, frozen, canonical Bytes prototype. Actor-value graph validation already copied mutable `ProtosBytesValue` instances but traversed that hidden parent as an ordinary object, reaching native Closure slots and signaling `NonTransferableValue` before delivery. The user observed `ARGS_FAILED` and `WORKER_RPC_FAILED` with an HTTP method String plus Bytes target in the separate RealWorld two-Worker application; its original probes were uncommitted local application work.

Published repair:

1. `ProtosPrelude.isBytesPrototypeForRuntime(Object)` is a nullable-safe, exact-identity check for the Prelude's canonical hidden Bytes prototype. Reduced Preludes return false; foreign Prelude prototypes and instances do not match.
2. `ProtosActorValueTransfer.Copier.isSharedStandardObject` recognizes that exact frozen standard anchor before traversing parent slots.
3. Each ordinary Bytes instance still receives its **own destination copy**, preserving its parent, indexed octets, local slots, open/closed/frozen state, cycles, and aliases. Closure-bearing and otherwise non-transferable reachable graphs still fail atomically. No arbitrary application prototype is newly shared.
4. The HTTP host fixture's outdated claim that standard Bytes cannot transfer was corrected without changing fixture behavior.

## Published regression surfaces

- [Java copier tests](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/src/test/java/com/guillermomolina/protos/runtime/ProtosActorValueTransferTest.java): exact prototype identity, reduced Prelude, canonical-parent and root-parent Bytes, String+Bytes argument vector, nested HTTP-shaped response, requester Future resolution, cycles/aliases, independent mutation, closed/frozen states, five fail-closed cases and unchanged source graphs.
- [Actor guest corpus](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/protos/tests/conformance/actor/bytes-transfer.protos): 12 cases including request, reply, send, spawn, snapshot timing and rejection. Registered in the Actor manifest.
- [Group guest corpus](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/protos/tests/conformance/group/bytes-transfer.protos): 9 cases including request, reply, aliases, isolation, synchronous send validation and rejection. Registered in the Group manifest.
- **Ordering qualification:** Group send-to-request ordering is not guaranteed and `SendOperation` supplies no acceptance observation. The Group test uses completed `store` and `load` requests to observe a separate snapshot; it does not claim that the earlier `send` was accepted or processed.

## Maintainer-reported validation

| Check | Result | Provenance |
| --- | --- | --- |
| Focused Java tests covering Actor value transfer, Prelude, Actor and Group harnesses, and public Actor API | PASS reported | Maintainer's earlier confirmation that all requested tests passed |
| `make test-protos` | PASS reported | Maintainer's earlier all-tests-green confirmation |
| `make test` | PASS reported | Explicit 2026-11-07 maintainer report after push |
| `make check` | PASS reported | Explicit 2026-11-07 maintainer report after push |
| `git diff --check` | CLEAN reported | Explicit 2026-11-07 maintainer report |
| Protos commit and publication | VERIFIED | Exact GitHub commit and published file contents checked independently |

The evidence recorder **did not execute** Maven, Protos conformance, `make test`, `make check`, the RealWorld application, Native Image, or Hurl. The version bump occurred after green substantive tests per the maintainer's workflow; no new independent test run is inferred from metadata verification.

## Outstanding application acceptance

The original BUG035 acceptance criterion 4 asks for a new real two-Worker RealWorld loopback reproduction that no longer reports `ARGS_FAILED` or spurious HTTP 500 responses. **That execution has not been reported.** Passing Protos unit/conformance/integrated gates proves the isolated BUG035 repair candidate, not the behavior of an installed launcher or uncommitted RealWorld application state.

Thus the **product implementation is published and locally green**, but **BUG035 issue-level closure remains pending the RealWorld smoke evidence**. I153 remains independently open; do not claim full RealWorld API/Hurl conformance. The small next check is to launch the affected two-Worker application against a CLI that reports the fixed Protos revision, then record the exact launcher version and observed request/HTTP results. The original two-Worker code/probes are not treated as committed repository history.

## Evidence markers

```text
WORK_ITEM=BUG035
ISSUE=guillermomolina/protos#1081
PROTOS_REVISION=bfb276f9b76c663d3ef1fb68823e13a22ede136c
IMPLEMENTATION_VERSION=0.3.452-SNAPSHOT
PUBLISHED_PATHS=12
PRODUCT_PUBLICATION=GITHUB_VERIFIED
PROTOS_FOCAL_TESTS=PASS_REPORTED
PROTOS_CONFORMANCE_TESTS=PASS_REPORTED
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_MAKE_CHECK=PASS_REPORTED
HUMAN_GIT_DIFF_CHECK=CLEAN_REPORTED
SPEC_CHANGE=NONE
REALWORLD_TWO_WORKER_SMOKE=NOT_REPORTED
OFFICIAL_REALWORLD_HURL=NOT_CLAIMED
BUG035_ISSUE_CLOSURE=PENDING_REALWORLD_ACCEPTANCE
```
