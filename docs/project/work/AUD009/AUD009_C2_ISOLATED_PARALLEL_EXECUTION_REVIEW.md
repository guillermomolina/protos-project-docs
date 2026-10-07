# AUD009-C2 — Isolated parallel execution, parallel collections, and byte-region complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#624`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence and closure revalidation revision:
`a5f4444f25d1f2722669f4cfdf5d7f62b1af6d88`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#624`, issue comment
`5739365892`.

Owner approval provenance: `guillermomolina/protos#624`, issue comment
`5739377593`, 2026-09-19.

Derived decision routes:

- `D160 / guillermomolina/protos#625` — Parallel collection algorithm
  placement.
- `D161 / guillermomolina/protos#626` — Exclusive writable byte-region
  capability.

Neither derived decision is selected by this audit record.

## Purpose and boundary

AUD009-C2 reviewed the isolated CPU-parallel layer between Actor-local
cooperative Future/task execution and persistent Actor isolation.

The review separated five concerns:

1. the minimal P execution institution;
2. Closure projection and P value transfer;
3. scheduling, progress, authority, failure and cancellation boundaries;
4. the five Core Array parallel algorithms;
5. Bytes/ByteRegion exclusive writable partitioning.

C2 did not reopen the Future/task model already classified by AUD009-C1, the
pending D159 Future-detachment decision, or Actor mailbox/lifecycle semantics
reserved for later AUD009-C work.

C2 is an evidence/classification slice. It does not itself alter normative
semantics or implementation.

## Evidence summary

Repository-wide Protos-source search at the approved baseline found no
production `protos/lib` or `protos/tools` consumer of:

```text
Closure.parallel
Array.parallelMap
Array.parallelFilter
Array.parallelFindIndex
Array.parallelReduce
Array.parallelSort
Bytes.parallelRange
ByteRegion.parallelRange
```

Current public P use is concentrated in examples, conformance and benchmarks.

That absence did not eliminate the foundational P institution. Retained
PERF001-F reference evidence showed real multicore scaling through the P
mechanism: the canonical `parallel-array-map` workload reported approximate
steady-median speedups of:

```text
width 1   1.000x
width 2   1.397x
width 4   2.582x
width 8   4.168x
```

The current implementation/specification footprint is also material:

```text
ProtosParallelRuntime.java      ~468 lines / ~29 KiB
ProtosByteRegionValue.java       ~84 lines / ~3.6 KiB
PARALLEL_EXECUTION.md          ~1687 lines / ~80 KiB
parallel programming guide      ~686 lines / ~22 KiB
```

The audit therefore distinguished the irreducible runtime guarantee from
higher-level policies and speculative optimization machinery.

## Final classification ledger

```text
P isolated execution domain                         KEEP
Closure.parallel(arguments...)                      KEEP
fresh caller-domain Future result                   KEEP
explicit parallel request                           KEEP
nested explicit P                                   KEEP
task-backed structured ownership                    KEEP

P Closure projection                                KEEP
explicit P input transfer                           KEEP
input snapshot before successful return             KEEP
cycles/aliasing inside one transfer graph           KEEP
no mutable alias to caller domain                   KEEP
NonParallelValue                                    KEEP
P result/Error transfer                             KEEP
safe physical sharing unobservable                  KEEP

bounded P scheduling                                KEEP
nested-P bounded-carrier progress                   KEEP
weak fairness                                       KEEP
scheduler-result independence                       KEEP
cooperative cancellation                            KEEP
no rollback of committed effects                    KEEP
P process-local                                     KEEP
no ambient Actor/Process/I/O/scheduler authority    KEEP

public P object/prototype/namespace                 ABSENT / RETAIN ABSENCE
public worker/executor/task handle                  ABSENT / RETAIN ABSENCE
remote P placement                                  ABSENT / RETAIN ABSENCE
P scheduler controls                                ABSENT / RETAIN ABSENCE
NUMA/CPU-affinity semantic surface                  ABSENT / RETAIN ABSENCE
SIMD/vectorization semantic surface                 ABSENT / RETAIN ABSENCE

Array.parallelMap                                   REMOVE_NOW_RECONSIDER_LATER
Array.parallelFilter                                REMOVE_NOW_RECONSIDER_LATER
Array.parallelFindIndex                             REMOVE_NOW_RECONSIDER_LATER
Array.parallelReduce                                REMOVE_NOW_RECONSIDER_LATER
Array.parallelSort                                  REMOVE_NOW_RECONSIDER_LATER
Array.parallelEach                                  ABSENT / RETAIN ABSENCE

Bytes.parallelRange                                 REMOVE_NOW_RECONSIDER_LATER
ByteRegion family                                   REMOVE_NOW_RECONSIDER_LATER
ByteRegion.parallelRange                            REMOVE_NOW_RECONSIDER_LATER
exclusive byte-range reservations                   REMOVE_NOW_RECONSIDER_LATER
parallel-region commit/publication protocol         REMOVE_NOW_RECONSIDER_LATER

ParallelRegionOverlap                               REMOVE_WITH_MECHANISM
ParallelRegionInUse                                 REMOVE_WITH_MECHANISM
ParallelRegionOutsideP                              REMOVE_WITH_MECHANISM

generic writable Array/object partitioning          ABSENT / RETAIN ABSENCE
general borrow/linear/affine ownership for P        ABSENT / RETAIN ABSENCE
```

No C2 mechanism is classified `REMOVE_PERMANENTLY`.

## Foundational P remains justified

C2 explicitly tested removing P entirely.

Future/task execution is not an equivalent replacement because ordinary
Actor-local `future()` intentionally shares one mutable execution domain and
serializes guest execution except at suspension boundaries.

Actors are not an equivalent replacement because finite CPU computation would
then pay for persistent incarnation identity, bootstrap module/binding identity,
mailbox/delivery, ActorRef communication, lifecycle, termination and
supervision/failure-authority semantics.

The retained distinction is:

```text
C / Future
    shared mutable domain
    cooperative execution

P
    finite isolated computation
    value/snapshot crossing
    simultaneous CPU eligibility
    Future result

Actor
    persistent isolated identity
    mailbox + lifecycle + communication
```

Moving only the public name to a library would not remove the required privileged
runtime authority for simultaneous guest execution, isolation, transfer,
projection, bounded scheduling or Future/cancellation integration.

Classification: **KEEP**.

## Projection and value transfer remain necessary

Ordinary Closures capture lexical contexts by reference.

Letting a normal captured caller environment cross into P would either create
shared mutable cross-domain state or silently reinterpret ordinary Closure
capture as copy-by-value for one operation.

The retained P model therefore projects executable Closure behavior into a fresh
P-local root and requires data/authority to cross explicitly.

The successful-return snapshot point prevents delayed worker admission from
changing logical P input after later caller mutations.

`NonParallelValue` remains the explicit boundary failure for values and
capabilities with no P transfer contract.

Physical copy-on-write, immutable sharing, storage reuse and similar
optimizations remain permitted only when semantically invisible.

Classification: **KEEP**.

## Scheduler and authority boundaries remain intentionally narrow

P keeps bounded carriers, nested-progress guarantees and weak fairness while
leaving worker count, queue policy, chunk size, work stealing, priority, NUMA,
CPU affinity and SIMD/vectorization outside portable semantics.

Process-local P also remains intentional. Remote placement would introduce code
availability/versioning, transport, authentication, retry, partition,
uncertainty and distributed-failure semantics into fine-grained CPU work.

Classification: **KEEP / RETAIN ABSENCE**.

## Core Array parallel algorithms are removal candidates

Approved classification:

```text
Array.parallelMap          REMOVE_NOW_RECONSIDER_LATER
Array.parallelFilter       REMOVE_NOW_RECONSIDER_LATER
Array.parallelFindIndex    REMOVE_NOW_RECONSIDER_LATER
Array.parallelReduce       REMOVE_NOW_RECONSIDER_LATER
Array.parallelSort         REMOVE_NOW_RECONSIDER_LATER
```

There are no current production Library/Tool consumers of these selectors.

The project design philosophy already places runtime guarantees such as
isolation and simultaneous execution below policies such as parallel map, sort,
batching and partitioning.

Serial `std:collections/Array` algorithms are source-backed library policy;
parallel variants are therefore plausible library peers rather than inherently
new Array value semantics.

The audit does not claim that the current five contracts can be reproduced
byte-for-byte using only naïve `Closure.parallel` plus `Future.all`.
Current Core algorithms include privileged semantics such as all-child
prevalidation, aggregate cancellation and deterministic indexed failure
selection. The absence of production demand means those additional guarantees
must be justified explicitly rather than retained automatically.

### Required route

D160 / #625 must compare:

- retaining the current Core family;
- Standard Library parallel collection facilities over the minimal P kernel;
- a Standard Library surface plus a genuinely narrower privileged helper;
- removal with no immediate replacement;
- any evidence-backed subset.

Standard Library placement is a mandatory first-class candidate.

### Reconsideration trigger

Concrete application/Standard Library demand for parallel collection algorithms,
with workload evidence showing which ordering, cancellation, failure and batching
guarantees are actually required.

## Bytes/ByteRegion writable partitioning is a removal candidate

Approved classification:

```text
Bytes.parallelRange                  REMOVE_NOW_RECONSIDER_LATER
ByteRegion                           REMOVE_NOW_RECONSIDER_LATER
ByteRegion.parallelRange             REMOVE_NOW_RECONSIDER_LATER
exclusive reservations               REMOVE_NOW_RECONSIDER_LATER
commit/publication protocol          REMOVE_NOW_RECONSIDER_LATER
```

with the three `ParallelRegion*` errors removed if the mechanism is removed.

Repository search found no Protos-source invocation of `parallelRange`,
including examples and canonical benchmarks.

The continuing cost leaks into ordinary Bytes. The runtime keeps reservation
state and ordinary `Bytes.at`, `atPut`, `add` and `removeAt` must
participate in the reservation protocol.

The mechanism also creates the special P-local ByteRegion family, recursive
subdivision, overlap/in-use/outside-P failure categories, transfer exclusions,
atomic commit-versus-cancellation semantics, and diagnostic/interop treatment.

The strongest retention argument remains valid in principle: a writable byte
partition can let several CPU cores modify disjoint byte ranges without
whole-buffer copying, locks, atomics or general shared mutable memory.

C2 found no current workload that requires that capability.

Removing the mechanism does not remove P. Ordinary isolated value/snapshot
parallelism remains available, and implementations retain invisible
copy-on-write/storage-sharing freedoms.

### Required route

D161 / #626 must compare:

- retaining the current capability;
- removing it until real workload evidence appears;
- a genuinely smaller privileged partition kernel;
- a library facade only if it actually reduces total mechanism;
- another buffer/ownership abstraction only if present evidence justifies it.

A namespace-only relocation of the same reservation machinery is not a
simplification.

### Reconsideration trigger

Real large-buffer workloads showing that isolated P copy/result semantics cannot
meet required performance without portable exclusive writable partition
authority.

## Generic writable object partitioning remains absent

C2 does not replace ByteRegion with a general ownership/partition system.

Disjoint Array indexes do not prove disjoint reachable mutable authority:

```text
array[0] -> sharedMutableObject
array[1] -> sharedMutableObject
```

Adding a borrow/linear/affine language mode merely to preserve an unused
specialized optimization would cross a far larger semantic threshold.

Classification: **ABSENT / RETAIN ABSENCE**.

## Strongest attempted removals

```text
remove P entirely
    rejected -> KEEP

replace P with Actor.spawn/request
    rejected -> KEEP P

move Closure.parallel to library while retaining hidden equivalent primitive
    rejected -> KEEP Core primitive

allow caller lexical capture to cross P
    rejected -> KEEP projection/explicit arguments

make P remote/distributed-capable
    rejected -> RETAIN process-local boundary

expose workers/executor/carrier/scheduler controls
    rejected -> RETAIN ABSENCE

Core Array parallel algorithm family
    survives removal -> REMOVE_NOW_RECONSIDER_LATER

Bytes/ByteRegion writable partition family
    survives removal -> REMOVE_NOW_RECONSIDER_LATER

generic writable Array/object partitioning
    rejected -> RETAIN ABSENCE
```

## Boundary handoffs

- **D160 / #625** owns parallel collection algorithm placement.
- **D161 / #626** owns the exclusive writable byte-region capability.
- **D159 / #623** independently owns Future detachment from C1.
- **Later AUD009-C work** owns Actor-specific concurrency/lifecycle complexity.
- **AUD009-E** owns general Standard Library breadth.
- **AUD009-G** owns concrete worker/runtime/backend implementation complexity.

## Owner approval and routing

```text
ISSUE=guillermomolina/protos#624
CHECKPOINT_COMMENT=5739365892
APPROVAL_COMMENT=5739377593
DATE=2026-09-19

P_MINIMAL_KERNEL=KEEP
P_PROJECTION_TRANSFER=KEEP
P_SCHEDULING_AUTHORITY_BOUNDARY=KEEP

ARRAY_PARALLEL_FAMILY=REMOVE_NOW_RECONSIDER_LATER
ARRAY_PARALLEL_DECISION=D160 / guillermomolina/protos#625

BYTE_REGION_FAMILY=REMOVE_NOW_RECONSIDER_LATER
BYTE_REGION_DECISION=D161 / guillermomolina/protos#626

GENERIC_WRITABLE_GRAPH_PARTITION=ABSENT_RETAIN_ABSENCE

NORMATIVE_CHANGE_AUTHORIZED_BY_C2=NO
IMPLEMENTATION_CHANGE_AUTHORIZED_BY_C2=NO
```

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_REVISION=a5f4444f25d1f2722669f4cfdf5d7f62b1af6d88
CLOSURE_REVALIDATION=a5f4444f25d1f2722669f4cfdf5d7f62b1af6d88
D160_NATIVE_PARENT=#624 PASS
D161_NATIVE_PARENT=#624 PASS
D160_PROJECT_ROUTING=PASS
D161_PROJECT_ROUTING=PASS
REMOVAL_ROUTES=D160,D161
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_C2_CLASSIFICATION=COMPLETE
```

AUD009-C2 is complete once this durable record and the required live GitHub
closure postconditions are verified.
