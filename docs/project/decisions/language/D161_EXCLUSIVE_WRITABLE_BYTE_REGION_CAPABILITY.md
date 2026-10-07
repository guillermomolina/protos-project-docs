# D161 — Exclusive writable byte-region capability

Status: **RATIFIED — Candidate B (remove now; reconsider from workload evidence later)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#626`
Trigger: AUD009-C2 / `guillermomolina/protos#624`
Protos evidence revision: `ecf563ed01275929d5b85330e8e6259cc85d73d8`
Project-record base: `1710f45282fa8f1c42ad236ebc5c09cde6d7350a`

This is a durable non-normative decision record. Observable Protos semantics remain authoritative under `guillermomolina/protos:spec/**`.

## Decision

D161 selects **Candidate B**.

```text
Bytes.parallelRange                    REMOVE_NOW_RECONSIDER_LATER
ByteRegion                             REMOVE_NOW_RECONSIDER_LATER
ByteRegion.parallelRange               REMOVE_NOW_RECONSIDER_LATER
exclusive byte-range reservations      REMOVE_NOW_RECONSIDER_LATER
parallel-region publication protocol   REMOVE_NOW_RECONSIDER_LATER
ParallelRegionOverlap                  REMOVE
ParallelRegionInUse                    REMOVE
ParallelRegionOutsideP                 REMOVE
```

Retained authority:

```text
Closure.parallel / isolated P          KEEP
P snapshot/value isolation             KEEP
safe invisible physical sharing        KEEP
ordinary Bytes                         KEEP
arbitrary shared mutable memory        REMAINS ABSENT
generic writable object regions        REMAINS ABSENT
general borrow/linear ownership        REMAINS ABSENT
```

D161 does not reject future high-performance buffer partitioning. It removes the current byte-only institution until concrete workload evidence justifies explicit writable partition authority.

## Approval provenance and invariant check

The complete D161 packet presented to the project owner covered current repository evidence, prior art, Candidates A–E, twelve-dimension scoring, failure modes, future-workload stress, deferral/reintroduction cost, compatibility, the recommendation, and its strongest objection.

Current non-use was explicitly not treated as sufficient evidence. The decisive additional fact is that the mechanism imposes continuing representation and operation cost on ordinary `Bytes` even when parallel regions are never used.

The project owner explicitly approved Candidate B in the active interaction on 2026-09-19:

```text
ok aprobada
```

Candidate B preserves all fixed D161 authority. The approved observable delta is only removal of the writable-region institution, its reservation machinery, and its mechanism-specific errors.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Repository evidence

At the evidence revision, repository-wide search found no Protos-source invocation of `parallelRange`, including Library, Tool, examples and canonical benchmarks.

The mechanism nevertheless affects ordinary `Bytes` continuously:

- every `ProtosBytesValue` owns reservation state;
- `Bytes.at` and `Bytes.atPut` test indexed reservations;
- `Bytes.add` and `Bytes.removeAt` test whether any reservation exists;
- the runtime owns `ProtosByteRegionValue`, recursive subdivision, overlap detection, P-only validity, cancellation release and success commit;
- transfer, diagnostics and interop explicitly recognize `ByteRegion`;
- Core owns three region-specific error prototypes.

The current implementation snapshots the selected range into a fresh `ByteRegion` and copies successful contents back on commit. It is therefore not presently a zero-copy implementation whose measured benefit independently justifies retaining the institution.

## Comparative evidence

The investigation compared materially different approaches. Rust uses general mutable-slice ownership and disjoint slicing. Legion makes regions and privileges a general runtime institution. Java Foreign Memory/NIO-style buffers and .NET `Span<T>`/`Memory<T>` provide bounded views without making every slice an exclusive authority. C++ spans/parallel execution largely leave writable alias discipline to the surrounding type/program model. Chapel supplies array/domain slicing and parallel iteration without installing Protos-style byte-only reservation semantics on every ordinary byte collection.

The relevant pattern is that strong writable-region authority is normally part of a broader ownership/region theory. The current Protos mechanism is unusually specialized because it installs that theory only for `Bytes`.

## Candidate result

- **A — retain current Core ByteRegion / parallelRange:** rejected because ordinary `Bytes` continues to pay for an unproven specialized capability.
- **B — remove now; reconsider from workload evidence later:** **selected**. It restores a P-independent ordinary `Bytes` model while retaining isolated P and snapshot/result isolation.
- **C — smaller privileged byte-partition kernel:** rejected for now; privileged exclusivity, cancellation and publication machinery would still be required and no smaller necessary kernel is demonstrated.
- **D — library facade:** rejected; a library cannot honestly enforce the authority contract without retaining privileged runtime machinery.
- **E — broader future Buffer/Region/ownership abstraction:** deferred as speculative without current workload evidence.

The twelve-dimension comparison scored A=42, B=57, C=48, D=30 and E=38 out of 60. The arithmetic total was supporting evidence, not decision authority.

## Strongest argument against Candidate B

Future compression, cryptography, image processing, serialization, protocol or other large-buffer workloads may benefit materially from processing disjoint writable portions concurrently without whole-buffer copies. The current mechanism has already solved overlap rejection, recursive subdivision, cancellation without partial publication, successful commit, and exclusion of ordinary access during reservation.

That objection is accepted. Candidate B is still selected because no present workload demonstrates the need, the implementation is not zero-copy, ordinary `Bytes` continually carries the specialized institution, and future reintroduction can be additive on top of retained P rather than requiring a foundational P redesign.

## Deferral, compatibility and migration

If copying later dominates useful P computation, the project may reconsider a byte-region facility, broader Buffer/Region model, invisible storage sharing/copy-on-write, or another justified ownership/partition abstraction. D161 reserves none of those designs and does not promise the same `ByteRegion` or `parallelRange` API.

The cost of deferral is bounded because `Closure.parallel`, isolated P, snapshot/value transfer, Future ownership/cancellation and parallel scheduling remain.

Current repository evidence shows no production Protos-source consumer of the removed family. Implementation reconciliation must remove the public selector/value family, reservation state/checks, mechanism-specific errors and region-only transfer/diagnostic/interop handling; reconcile normative P/runtime/Bytes/error text, changelog, guide and tests; and preserve ordinary `Bytes`, isolated P, Future/task and Actor boundaries.

No compatibility alias, hidden reservation hook or reserved selector is retained merely for possible future reintroduction.

## Intentionally deferred

D161 does not decide whether Protos will eventually have general Buffer/Region or typed/native buffers, a zero-copy/copy-on-write/copy-commit region model, general ownership/borrow/linear semantics, future API spelling, or use of regions by future parallel collection algorithms.

## Routing

Candidate B changes observable Core semantics and requires a separate implementation owner. That work must follow current `AGENTS.md`, including focused/full validation, normative/spec changelog reconciliation, source-license/version rules, and required CI closure gates.

```text
D161_STATUS=RATIFIED
SELECTED_CANDIDATE=B
BYTE_REGION_CAPABILITY=REMOVE_NOW_RECONSIDER_LATER
FOUNDATIONAL_ISOLATED_P=KEEP
ORDINARY_BYTES=KEEP
NORMATIVE_RECONCILIATION_REQUIRED=YES
IMPLEMENTATION_RECONCILIATION_REQUIRED=YES
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
