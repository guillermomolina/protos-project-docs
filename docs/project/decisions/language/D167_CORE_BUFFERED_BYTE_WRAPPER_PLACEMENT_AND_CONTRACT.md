# D167 — Core buffered byte wrapper placement and contract

Status: **RATIFIED — Candidate D (Standard Library surface plus minimal privileged support)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#637`
Trigger: AUD009-D1 / `guillermomolina/protos#636`
Protos evidence revision: `5ce8e039a69489a49fe446d58de7fb39bcbb278f`
Project-record base: `08542d9314341f749c8cebe4f58108905a7d7672`

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative only through the applicable ratified material under
`guillermomolina/protos:spec/**`.

## Decision

D167 selects **Candidate D**.

The standard byte-buffering abstraction remains part of the Protos platform, but
its user-facing placement moves out of Core/prelude and into Standard Library I/O.

Selected boundary:

```text
Core / prelude
    ByteReadable                 KEEP
    ByteWritable                 KEEP
    Flushable                    KEEP
    Closable                     KEEP
    generic I/O commitment       KEEP
    generic I/O lifecycle        KEEP

    BufferedReader               REMOVE AS CORE/PRELUDE BINDING
    BufferedWriter               REMOVE AS CORE/PRELUDE BINDING

Standard Library
    std:io/BufferedReader        ADD / standard buffered byte reader
    std:io/BufferedWriter        ADD / standard buffered byte writer

Private runtime support
    permitted only where required to preserve the selected standard contract
    and the already-ratified generic I/O invariants.
```

The Standard Library wrappers retain the current strong semantics rather than a
weaker source-only approximation.

In particular, the move does **not** weaken or reopen:

- `ByteReadable`, `ByteWritable`, `Flushable` or `Closable`;
- asynchronous Future-shaped I/O operation semantics;
- operation commitment separate from Future terminal state;
- pre-commit zero-effect cancellation;
- post-commit non-rollback;
- wrapper borrowing versus explicit ownership;
- D117 delegated first-effect arbitration;
- no unsafe replay after unknown lower effect;
- close-cutover semantics;
- PLAT031 one-operation/one-lifecycle authority; or
- ActorExecutionDomain / C-prime runtime custody architecture.

The implementation is free to reuse, reduce, reorganize or rewrite the current
buffered runtime classes as long as the Standard Library surface preserves the
selected semantics. Candidate D does not require retaining today's Java class
layout or line count.

## Public placement

The current required prelude bindings:

```text
BufferedReader
BufferedWriter
```

are removed from the Core/prelude surface.

The standard user-facing capability moves to the existing Standard Library I/O
domain, conceptually:

```text
import std:io/BufferedReader
import std:io/BufferedWriter
```

The exact repository module spelling must follow the project's normal module and
import conventions during implementation. D167 owns the semantic layer boundary,
not an implementation-specific source-file loader trick.

The Standard Library facilities retain the same essential factory roles:

```text
BufferedReader(source)          borrowing
BufferedReader.owning(source)   owning

BufferedWriter(target)          borrowing
BufferedWriter.owning(target)   owning
```

Each successful construction still creates a fresh wrapper over the explicitly
supplied lower capability.

## Why this is not Candidate B

D167 does not select removal of standard byte buffering.

Explicit byte buffering is a mature, recurring systems-I/O capability. More
importantly for Protos, recreating a correct asynchronous buffered wrapper later
would have to revisit the same difficult semantic boundaries already closed by
D117 and PLAT031:

- ordered wrapper operations;
- read-ahead retention;
- output buffering/frontiers;
- lower Future observation;
- producer commitment;
- cancellation before versus after irreversible effect;
- unknown lower partial effect;
- close cutover;
- no duplicate replay;
- failure poisoning/reuse policy; and
- lifecycle ownership.

Removing the entire capability therefore has substantial reintroduction risk and
would likely recreate a materially similar semantic problem later.

Historical implementation effort is sunk cost and is not a reason to retain the
feature. The reason to retain it is the high probability that the capability is
useful again together with the concrete cost of re-establishing its strong I/O
contract.

## Why this is not Candidate A

The fact that the capability is worth retaining does not establish that it is a
fundamental Core/prelude institution.

Buffering is an adapter/policy over lower byte capabilities. Protos philosophy
prefers the Core to provide the mechanisms that cannot be implemented safely or
portably at higher levels, while Standard Library owns ordinary reusable
institutions built on those mechanisms.

The public buffered wrapper therefore does not need to be a global prelude
binding merely because some of its correctness requires privileged runtime
support.

Moving it to `std:io` reduces the semantic universe visible to every program
without discarding the capability.

## Why this is not Candidate C

A purely source-level Standard Library implementation with a deliberately weaker
contract is rejected.

D117 is direct evidence that a wrapper cannot in general infer lower producer
commitment from the lower Future. In particular, an ordinary failed
`ByteWritable.write` may have contributed an unobservable prefix, so a wrapper
must not publish a zero-effect cancellation/closure result or replay the payload
merely because source-level code lacks access to producer-side effect evidence.

A source-only implementation could remain correct only by weakening behavior,
for example by committing conservatively before delegation or giving up the
existing cancellation precision.

D167 preserves the stronger established contract instead.

## Candidate D privileged boundary

Candidate D permits private implementation help only for invariants that cannot
be preserved correctly with ordinary source composition.

The retained privileged mechanism should reuse the generic I/O substrate rather
than create a second buffered-only semantic universe:

```text
one logical buffered operation
    -> one generic I/O operation authority
    -> one generic lifecycle authority
    -> private adapter state for buffering/order
```

D167 does not create a public commitment token, fifth Future state, partial-write
count, hidden Task, second scheduler, shadow operation, or generic Stream
hierarchy.

The implementation should minimize privilege over time where mechanical
simplification is possible, but no implementation rewrite is required merely to
make the internal code appear more library-like.

## Approval provenance

The D167 packet initially considered retaining the current wrappers in Core
because the difficult buffering semantics are likely to be needed again.

The project owner then identified the missing distinction:

```text
bueno pero no necesariamente tiene que estar en core, se puede mover a una libreria
```

The decision was re-evaluated by separating:

```text
retain buffering capability
    !=
retain buffering as Core/prelude institution
```

The exact revised recommendation presented for approval was Candidate D:

```text
BufferedReader / BufferedWriter
    STANDARD LIBRARY, no Core

strong buffering contract
    KEEP

D117 semantics
    KEEP

generic I/O commitment/lifecycle
    KEEP in Core

implementation technique
    PRIVATE / free to rewrite
```

The project owner explicitly approved that exact pending candidate on
2026-09-19:

```text
aprobada
```

```text
SELECTED_CANDIDATE=D
DECISION_APPROVAL_PROVENANCE=PASS
```

## GITHUB021 invariant/delta consistency

Applicable D167 fixed invariants:

```text
BYTEREADABLE_RETAINED                            PRESERVED
BYTEWRITABLE_RETAINED                            PRESERVED
FLUSHABLE_RETAINED                               PRESERVED
CLOSABLE_RETAINED                                PRESERVED
COMMON_IO_COMMITMENT_MODEL_RETAINED              PRESERVED
COMMON_IO_CANCELLATION_MODEL_RETAINED            PRESERVED
COMMON_IO_LIFECYCLE_MODEL_RETAINED               PRESERVED
FUTURE_REMAINS_OUTCOME_MECHANISM                 PRESERVED
BORROWING_AND_OWNING_WRAPPERS_REMAIN_VALID       PRESERVED
NO_FALSE_ZERO_EFFECT_AFTER_POSSIBLE_COMMIT        PRESERVED
D117_DELEGATED_EFFECT_CONTRACT                    PRESERVED
PLAT031_ONE_OPERATION_ONE_LIFECYCLE               PRESERVED
```

Observable semantic/API delta:

```text
BufferedReader prelude binding        REMOVED
BufferedWriter prelude binding        REMOVED
standard buffered reader              MOVED TO std:io
standard buffered writer              MOVED TO std:io
buffering semantics                    PRESERVED
strong cancellation/commitment rules   PRESERVED
```

No owner-approved invariant is silently contradicted.

D117 remains fully applicable to the standard output wrapper after relocation:
its contract is a wrapper-composition rule, not a requirement that the wrapper
be a Core-prelude binding.

PLAT031 remains the current internal architecture authority unless later platform
work explicitly replaces its implementation mechanism while preserving the same
observable semantics.

```text
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Prior-art comparison

The investigation considered at least five mature systems across different
language/runtime approaches.

### Java

Java provides buffered byte and character wrappers in its standard libraries
rather than as language primitives. Buffering is an ordinary I/O abstraction
over lower streams.

Contribution: strong evidence that buffering is useful but need not be a
fundamental language/Core institution.

### .NET

`BufferedStream` is a standard library adapter over another `Stream`.

Contribution: again separates general underlying I/O capability from optional
buffering policy.

### Rust

`std::io::BufReader` and `BufWriter` are standard-library adapters over
`Read`/`Write`.

Contribution: supports Standard Library placement and makes explicit that
buffering exists to amortize lower I/O calls rather than define primitive I/O
identity.

### Tokio

Tokio provides async buffered wrappers whose cancellation/progress behavior
requires careful treatment of partial progress.

Contribution: strong evidence that asynchronous buffering remains a real
semantic problem and that moving the public abstraction out of Core does not
make the runtime/cancellation problem disappear.

### Go

`bufio.Reader` and `bufio.Writer` are ordinary standard-library wrappers over
`io.Reader`/`io.Writer`; exact lower progress lets Go preserve unwritten
suffixes after partial writes.

Contribution: supports library placement while also demonstrating why Protos,
whose standard lower write failure does not expose an exact prefix count, needs
its own conservative delegated-effect rule.

### Python

`io.BufferedReader` and `io.BufferedWriter` are standard I/O-library
abstractions over raw binary streams.

Contribution: reinforces that byte buffering is conventional reusable I/O
functionality rather than a universal language-prelude primitive.

No surveyed system establishes that a Protos-style strong asynchronous
zero-effect cancellation contract is source-implementable without additional
lower progress/commitment information. That issue remains Protos-specific and is
already owned by D117.

## Comparative scoring

Scores use 1–5; confidence is H/M/L.

| Criterion | A Core | B Remove | C stdlib simple | D stdlib + private support |
| --- | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 5H | 5H | 3H | 5H |
| Protos alignment | 4H | 4H | 3H | 5H |
| Present-need proportionality | 3H | 5H | 4M | 5H |
| Incremental growth | 4H | 3M | 3M | 5H |
| Future-option resilience | 4M | 4M | 3M | 5M |
| Scalability | 5H | 4M | 4M | 5H |
| Conceptual simplicity | 3H | 5H | 4M | 5M |
| Portability / implementation freedom | 4H | 5H | 5H | 5H |
| Runtime / resource cost | 5H | 5H | 5H | 5H |
| Failure / operability | 5H | 5H | 2H | 5H |
| Deferral / reversibility / migration | 4M | 2H | 3M | 5M |
| Evidence maturity / implementation risk | 5H | 4H | 3M | 4M |

The arithmetic is only a comparison aid.

Candidate A carries an over-placement concern: standard buffering is useful, but
its presence as a required Core/prelude institution is not justified by a
fundamental semantic need.

Candidate B carries an underengineering concern: removing a likely recurring
systems-I/O capability would probably require reconstructing much of the same
semantic work later.

Candidate C carries a correctness/semantic-weakening red flag under the already
retained I/O contract.

Candidate D best separates public placement from required low-level correctness.

## Incremental-design analysis

### Smallest sufficient Core

The smallest retained Core mechanism is:

```text
ByteReadable
ByteWritable
Flushable
Closable
Future
generic I/O operation commitment/cancellation/lifecycle
```

Core does not need globally visible buffered-wrapper names.

### Pay for what you need

Programs that do not import/use buffered I/O need not receive its public
conceptual surface.

Runtime state for buffering remains allocated only when a buffered wrapper is
constructed.

The project still pays implementation maintenance for any private privileged
support, but this cost now corresponds to a reusable optional Standard Library
facility rather than a mandatory Core institution.

### Grow as you need

Future Standard Library facilities such as compression, binary codecs, HTTP or
other streaming components may compose with the standard buffered wrappers where
useful.

Nothing in D167 requires those facilities to use buffering, exposes buffer sizes
as permanent semantics, or freezes one future streaming architecture.

### Cost of removal

Removing buffering entirely would delete substantial code and tests but would
also discard already-resolved composition semantics whose hard parts follow from
retained I/O guarantees rather than from accidental implementation structure.

### Cost of relocation

Relocation requires a bounded but real migration:

- remove two required prelude bindings;
- move/create Standard Library modules;
- preserve public factory/ownership semantics under imported modules;
- update normative specification ownership/placement;
- adapt bootstrap/native bridge wiring;
- update conformance and architecture tests;
- update examples/guides if any rely on global names; and
- preserve D117/PLAT031 behavior through the cutover.

This cost buys a concrete architectural simplification: Core exposes mechanisms,
while optional standard buffering becomes a library institution.

### Reversibility

The retained Standard Library implementation remains redesignable.

If a real consumer later shows that the current buffering contract, API shape or
runtime substrate is wrong, a future approved decision may rewrite it.

D167 deliberately does not convert today's private Java implementation into
permanent semantic authority.

## Strongest argument against Candidate D

The current implementation is already functioning and integrated in Core.
Relocation creates migration work, module/import changes and another
library-to-runtime bridge while eliminating only two prelude names and some Core
surface.

If the distinction between Core and Standard Library were purely cosmetic,
Candidate A would be cheaper.

D167 accepts that migration cost because Protos explicitly values a small Core
semantic universe, the abstraction is naturally an optional I/O adapter rather
than a primitive byte capability, and the difficult correctness machinery can
remain private without forcing every program to treat buffering as a Core
institution.

## Intentionally deferred

D167 does not decide:

- public buffer-size configuration;
- peek/unread APIs;
- generic Stream hierarchy;
- automatic capability forwarding;
- public lower-operation commitment/progress introspection;
- partial-write prefix results;
- compression/TLS/HTTP buffering policy;
- adaptive buffering algorithms;
- zero-copy APIs;
- vectored/scatter-gather I/O;
- whether future implementation work can mechanically reduce the current
  buffered runtime classes; or
- any change to D117/PLAT031 semantics beyond the relocation described here.

## Implementation consequences

D167 requires implementation work.

```text
NORMATIVE_SPEC_CHANGE_REQUIRED=YES
CORE_PRELUDE_CHANGE_REQUIRED=YES
STANDARD_LIBRARY_CHANGE_REQUIRED=YES
RUNTIME_BRIDGE_RECONCILIATION_REQUIRED=YES
TEST_MIGRATION_REQUIRED=YES
IMPLEMENTATION_OWNER_REQUIRED=YES
```

I060 / `guillermomolina/protos#663` owns the bounded migration.

## Ratified result

```text
D167_STATUS=RATIFIED
SELECTED_CANDIDATE=D

CORE_BUFFEREDREADER_BINDING=REMOVE
CORE_BUFFEREDWRITER_BINDING=REMOVE

STANDARD_BUFFERED_BYTE_CAPABILITY=KEEP
STDLIB_BUFFEREDREADER=ADD
STDLIB_BUFFEREDWRITER=ADD

STRONG_BUFFERING_CONTRACT=KEEP
D117_DELEGATED_EFFECT_CONTRACT=KEEP
PLAT031_OPERATION_LIFECYCLE_MODEL=KEEP
PRIVATE_RUNTIME_SUPPORT=ALLOWED_WHERE_REQUIRED

PUBLIC_COMMITMENT_PROGRESS_API=NOT_ADDED
GENERIC_STREAM_HIERARCHY=NOT_ADDED
BUFFER_SIZE_API=NOT_ADDED

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
