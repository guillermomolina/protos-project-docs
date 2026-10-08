# I086 / PLAT054-3E3-SPEC — B011 Filesystem bootstrap-abort normative publication

Date: 2026-10-08  
Issue: [I086 / #840](https://github.com/guillermomolina/protos/issues/840)  
Architecture: [PLAT054 / #838](https://github.com/guillermomolina/protos/issues/838)  
Blocker: B011  
Slice: PLAT054-3E3-SPEC (normative publication only)

## Verified product publication

The independently read GitHub `guillermomolina/protos/main` HEAD is [`1d6d537d79d69fae3ad1e613cefd76b7a97abe38`](https://github.com/guillermomolina/protos/commit/1d6d537d79d69fae3ad1e613cefd76b7a97abe38), commit message **`PLAT054-3E3-SPEC: ratify Filesystem bootstrap failure policy`**.

Exactly two changed product paths, confirmed from that commit:

1. `spec/io/PROCESS_IO.md`
2. `spec/PROTOS_SPEC_CHANGELOG.md`

The newly published global specification revision is **`0.1.451`**, dated 2026-10-08. The commit diff and final normative paragraphs were read, including the removal of the `0.1.450` explicit A/B open clause.

## Owner decision and published normative consequence

The owner explicitly selected **Alternative A (bootstrap abort)** in the PLAT054-3E3-FS0 investigation: “evidentemente apruebo A”. [Exact approval and GITHUB010/GITHUB021 comparison](../PLAT054/PLAT054_3E3_FS0_FILESYSTEM_BOOTSTRAP_ABORT_OWNER_APPROVAL.md).

Specification `0.1.451`, `PROCESS_IO.md` → `Embedding filesystem base`, now mandates:

- Effective host File I/O authorization with an unrepresentable or unconfineable configured provider/Context working-directory base **aborts initial Process bootstrap before any initial module source expression**.
- The embedding host observes an explicit bootstrap provisioning failure. This cannot be silently converted into a successful bootstrap with an absent `filesystem` slot.
- No guest-observable partially established Filesystem capability, broader fallback, ambient `java.nio` authority, new public Protos Error family or physical-host-directory assumption for virtual providers.
- Without effective File I/O grant, the bootstrap-local `filesystem` slot remains absent, not `null`.
- Ordinary authorized Filesystem operation failures follow the existing FS/I/O contracts, rather than poisoning a safely provisioned Process.
- Failed bootstrap preserves cleanup and the non-reusable/never-established Process distinction; successful provisioning does not imply eager creation of unused Filesystem operation resources.

HOST-FS-1/2, the eight PLAT054 invariants, Provider confinement in `FILESYSTEM.md` §§20/20.1, IO lifecycle, Actors, module semantics, Network independence, and PAY AS YOU GROW remain unchanged.

## Validation provenance and non-claims

The maintainer reports that `git diff --check` was clean, **all local tests passed**, and commit/push completed. These are human reports, not test execution, CI log analysis or independent validation by the coordinating assistant.

The coordinator independently confirmed the GitHub product commit, the two changed paths, the exact normative text, and specification revision `0.1.451`.

**No Filesystem runtime/backend implementation was published by this spec-only commit**. At the checked product HEAD, the standard embedding's `ProtosEmbeddedProcess.bootstrap()` still passes `null` for the default Filesystem capability. No positive filesystem grant, custom virtual provider conformance, Native Image or portable distribution result is established by this publication.

## B011 transition and next slice

**B011: BLOCKED → READY**, now that the exact owner choice has been published normatively and verified. READY means the runtime implementation is authorized, **not** that Filesystem works in the standard embedding. B011 moves to CLOSED only when the corresponding runtime/backend implementation is published and verified.

**B012 remains CLOSED**, independently completed with the earlier Network slice. **I086/#840 remains OPEN** for B011 implementation and final Native/portable conformance. PLAT054/#838 maintains its independent native hierarchy/closure postconditions; I087/#841 application-module resolution and PERF032/#831 performance/graph parity are separate.

Next coherent product slice: **`PLAT054-3E3-FILESYSTEM` — TYPE=IMPLEMENTATION — REPOSITORY=`guillermomolina/protos` — AGENT=EDITOR — HUMAN_EXECUTOR=YES**. Implement and test the provider-confined default Filesystem capability, including physical/restricted/virtual providers, open/entries/replace/remove, safe name resolution, cancellation/cleanup, positive and negative bootstrap, Context close, Context independence and PAY AS YOU GROW. No new public semantics or unrelated repo edits.

No new Issue is required: this is a bounded implementation slice of I086, not independently scheduled work.

## Evidence and assistance

This document was prepared using ChatGPT AI assistance based on the exact published GitHub commit and the maintainer's reported validation. It is an immutable-style checkpoint; later runtime publications must have separate evidence. This publication does not change the product specification, and no agent-side build, Maven invocation or product Git operation is claimed.

```text
ISSUE=I086/#840
DESIGN=PLAT054/#838
SLICE=PLAT054-3E3-SPEC
TYPE=IMPLEMENTATION_NORMATIVE_PUBLICATION
PRODUCT_REVISION=1d6d537d79d69fae3ad1e613cefd76b7a97abe38
NORMATIVE_SPEC=0.1.451
OWNER_APPROVAL=A_BOOTSTRAP_ABORT
PRODUCT_CHANGED_PATHS=spec/io/PROCESS_IO.md;spec/PROTOS_SPEC_CHANGELOG.md
SPEC_PUBLICATION=PASS
TESTS=HUMAN_REPORTED_ALL_LOCAL_PASS
DIFF_CHECK=HUMAN_REPORTED_CLEAN
B011=READY_FOR_RUNTIME
B012=CLOSED
FILESYSTEM_RUNTIME=NOT_IMPLEMENTED_IN_STANDARD_EMBEDDING
NATIVE_IMAGE=UNVERIFIED
PORTABLE_DISTRIBUTION=UNVERIFIED
NEXT_SLICE=PLAT054-3E3-FILESYSTEM
NEXT_REPOSITORY=guillermomolina/protos
HUMAN_EXECUTOR=YES
```
