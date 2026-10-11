# I094 — Published source-owned Path semantics and inert Filesystem boundary

**Record date:** 2026-10-11
**Formal implementation owner:** [I094 / #875](https://github.com/guillermomolina/protos/issues/875)
**Originating audit:** [AUD021 / #874](https://github.com/guillermomolina/protos/issues/874)
**Approved architecture:** Candidate C; see [AUD021-B/C owner approval](../AUD021/AUD021_BC_CANDIDATE_C_OWNER_APPROVAL_AND_IMPLEMENTATION_BOUNDARY_2026_10_11.md)
**PROTOS_REVISION:** `0980d59b9f654b1668f836f1c5286dedab90770b`
**Prior product revision:** `b47be4b698ca27b537898abee89dd6f4c8c49fc3`
**Published change:** [I094 source/test commit](https://github.com/guillermomolina/protos/commit/0980d59b9f654b1668f836f1c5286dedab90770b)
**Version observed at published product revision:** `0.3.334-SNAPSHOT`
**Evidence classification:** revision-pinned implementation inspection and maintainer-reported validation; no independent execution by the recording agent.

## Publication and implemented slices

The product commit titled **"I094: transfer Paths as inert descriptors and materialize Filesystem results in the owning Actor"** is published on the repository's default branch. Comparison with the prior product revision is one commit ahead and includes 41 changed paths (15 production Java files, 22 Java test files, and four Protos source/conformance files).

The three internally sequenced slices are represented by this single published revision, not by independently published commits:

- **I094-A1 — source semantics:** [`protos/lib/core/Path.protos`](https://github.com/guillermomolina/protos/blob/0980d59b9f654b1668f836f1c5286dedab90770b/protos/lib/core/Path.protos) now defines source-backed `relative`, `child`, `_corePathEquals` (installed as `==`) and `hash`. The private `_corePathKernel` is limited to `mint`, `componentCount` and `componentAt`. The [installer](https://github.com/guillermomolina/protos/blob/0980d59b9f654b1668f836f1c5286dedab90770b/src/main/java/com/guillermomolina/protos/execution/ProtosStandardPathProtocol.java) authenticates source closures and installs the physical bridge. The immutable Java carrier remains; its former Java child/equality/hash methods were removed. Hash is calculated with explicit signed 32-bit wrap semantics in Protos.
- **I094-A2 — inert Filesystem seam:** [`ProtosPathComponents`](https://github.com/guillermomolina/protos/blob/0980d59b9f654b1668f836f1c5286dedab90770b/src/main/java/com/guillermomolina/protos/runtime/ProtosPathComponents.java) is a defensive, immutable `List<String>` descriptor. The open, replace/remove and entries flows capture guest Paths before calling the backend. The standard backend interface and all five production implementations use the inert descriptor instead of `ProtosPathValue`. Host-native NIO/TruffleFile resolution, captured-resource access and confinement remain backend responsibilities.
- **I094-A3 — destination rematerialization and owner-domain results:** [Actor transfer](https://github.com/guillermomolina/protos/blob/0980d59b9f654b1668f836f1c5286dedab90770b/src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java) uses inert descriptors and the existing detached-graph/destination Prelude machinery, memoizing aliases; [P transfer](https://github.com/guillermomolina/protos/blob/0980d59b9f654b1668f836f1c5286dedab90770b/src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java) likewise uses its existing transfer memo and destination materialization. [Open results](https://github.com/guillermomolina/protos/blob/0980d59b9f654b1668f836f1c5286dedab90770b/src/main/java/com/guillermomolina/protos/runtime/ProtosFilesystemOpenFlow.java) and [tree entries](https://github.com/guillermomolina/protos/blob/0980d59b9f654b1668f836f1c5286dedab90770b/src/main/java/com/guillermomolina/protos/runtime/ProtosFilesystemTreeObservationFlow.java) use the existing targeted Actor-domain completion mechanism for external callbacks, with direct handling for owner-executed synchronous completion and late/cancelled-result checks.

The change did not amend normative `spec/` documents. The architectural selection remains Candidate C, not a new D/PLAT decision.

## Validation evidence and its limits

**Reported by the human executor on 2026-10-11:**

- `git diff --check` — clean.
- All local tests — PASS.
- The I094 product commit has been pushed.

**Independently inspected using GitHub read-only access:** the exact published SHA and its diff summary, current `Path.protos`, the private kernel and carrier, the inert descriptor, Actor/P transfer branches and Filesystem completion boundaries. This inspection verifies publication and implementation shape; it is not a compiler run, a security proof, a reproduced test report or a benchmark.

**Not supplied:** complete test transcript, exact command sequence/test counts, CI status for this SHA, profiler evidence, or measured overhead. Do not infer any of those. No tests were rerun by the coordinating agent.

## Quantitative Java-surface checkpoint

From the exact GitHub comparison `b47be4b...0980d59b`:

| Scope | Files | Added lines | Deleted lines | Net |
| --- | ---: | ---: | ---: | ---: |
| Production Java | 15 | 495 | 120 | +375 |
| Java tests | 22 | 1,201 | 97 | +1,104 |
| Protos source and conformance | 4 | 122 | 1 | +121 |

These are line-change counts, **not** a runtime performance claim or evidence of a net reduction in Java lines. The implementation moves the Path semantics into source and removes guest Path coupling from host backends, but the overall production Java textual surface increased. Preserve this distinction in any future audit or optimization decision; do not retroactively describe the Java line count as reduced.

## Administrative closure reconciliation

At this observed product revision, `pom.xml` still declares `0.3.334-SNAPSHOT` and `CHANGELOG.md` has no I094 entry. The project's human-executor version/changelog publication rule has not been independently evidenced for this commit; reconcile with the maintainer without allowing this documentation agent to write product repository contents.

The native GitHub parent/sub-issue relationship **AUD021 → I094** was recorded as pending during allocation. The available GitHub connector does not expose native parent/sub-issue mutation; textual links are not a substitute under GITHUB006/GITHUB015. Do not assert that the formal hierarchy is reconciled or close I094 while this required structural postcondition is unknown/pending.

I094's implementation is published and the maintainer reports local tests PASS. Formal issue closure remains pending the two coordination/publication checks above. AUD021's separate audit-closure check likewise remains outstanding.

## Provenance

This record is non-normative evidence of a published implementation. It follows the earlier [AUD021-A research](../AUD021/AUD021_A_PROTOS_FIRST_PATH_BOUNDARY_RESEARCH_CHECKPOINT_2026_10_11.md) and [Candidate C approval](../AUD021/AUD021_BC_CANDIDATE_C_OWNER_APPROVAL_AND_IMPLEMENTATION_BOUNDARY_2026_10_11.md) without overwriting either. Product source authority is the exact `PROTOS_REVISION` above; GitHub Issues remain the live status authority.
