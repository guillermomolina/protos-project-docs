# AUD009-D1 — I/O core, byte/text protocols, and Process I/O complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#636`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence revision:
`05a8754e55c1ce5cb44effc621f1b454f1fa9697`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#636`, issue comment
`5739573610`.

Owner approval provenance: `guillermomolina/protos#636`, issue comment
`5739586169`, 2026-09-19.

Derived decision routes:

- `D167 / guillermomolina/protos#637` — Core buffered byte wrapper placement
  and contract.
- `D168 / guillermomolina/protos#638` — Process bootstrap snapshot identity
  and argument representation.

Neither derived decision is selected by this audit record.

## Purpose and boundary

AUD009-D1 reviewed the reusable I/O semantic substrate shared by files,
networking, text adapters and Process standard streams:

- asynchronous I/O operation shape;
- producer-side commitment;
- cancellation;
- logical-flow ordering;
- lifecycle and close;
- wrapper ownership;
- byte capability decomposition;
- Encoding;
- TextReader/TextWriter;
- standard byte buffering;
- Process bootstrap data and standard streams;
- Process-local I/O authority and Actor transfer boundaries.

Filesystem- and network-specific public breadth remains for later AUD009-D
slices. Physical poller/NIO/C-prime/backend architecture remains AUD009-G
material except where prior implementation decisions are evidence that a
semantic boundary has real continuing cost.

AUD009-D1 itself authorizes no normative or implementation change.

## Evidence summary

### Production use

Current production Protos code materially uses the retained I/O model:

- `protos/lib/io/Files.protos` uses asynchronous File acquisition, read, write,
  close and cancellation-aware late-acquisition custody;
- Package Tool and Test Tool use `TextReader.owning(...)`, `readText`,
  `readLine`, `TextWriter`, `writeLine`, Process stdout/stderr encodings
  and `process.args()`;
- JSON and CSV streaming adapters use TextReader/TextWriter Future-shaped
  operations;
- Encoding is used throughout Standard Library and Tool code, including UTF-8
  and exact octet-preserving Latin1 paths;
- `std:io/ProcessStreams` explicitly layers text adapters over Process byte
  streams using the Process-selected Encoding.

### No production use found for Core buffered wrappers

Repository search found no `protos/lib` or `protos/tools` consumer of Core
`BufferedReader(...)` or `BufferedWriter(...)`.

They remain required Core-prelude bindings and have substantial continuing
implementation/specification/test surface.

### No production identity dependency found for bootstrap snapshots

Production code uses `process.args()`, but no production Standard Library or
Tool code was found to depend on:

```text
process.args() === process.args()
process.environment() === process.environment()
identityHashOf(snapshot)
IdentityMap(snapshot, ...)
```

D018 nevertheless currently specifies canonical identity for one arguments
snapshot and one Environment snapshot per Process lifetime.

## Final classification ledger

```text
IO Future operation model                              KEEP
I/O commitment separate from Future terminal state    KEEP
first-effect arbitration                              KEEP
pre-commit cancellation                              KEEP
post-commit non-rollback                             KEEP
bounded retention/backpressure                       KEEP
logical-flow ordering                                KEEP
per-write non-interleaving                           KEEP

Closable                                              KEEP
close commitment                                      KEEP
idempotent logical close                             KEEP
fresh Future identity per close invocation            KEEP
shared lifecycle outcome                             KEEP
recorded close Error identity                        KEEP
wrapper borrowing/owning                             KEEP
Actor termination != implicit close                  KEEP
Process termination != successful close/flush/sync   KEEP

ByteReadable                                          KEEP
ByteWritable                                          KEEP
Flushable                                             KEEP
ByteSeekable                                          KEEP
ByteSized                                             KEEP
Truncatable                                           KEEP
Syncable                                              KEEP
ReadShutdown                                          KEEP
WriteShutdown                                         KEEP
universal Stream prototype                            ABSENT / RETAIN ABSENCE
automatic wrapper capability forwarding              ABSENT / RETAIN ABSENCE

Encoding                                              KEEP
explicit String/Bytes encoding boundary              KEEP
UTF8 / UTF16LE / UTF16BE / Latin1                   KEEP
TextReader                                            KEEP
TextReader.readText                                   KEEP
TextReader.readLine                                   KEEP
TextWriter                                            KEEP
TextWriter.writeText                                  KEEP
TextWriter.writeLine                                  KEEP
TextWriter.flush                                      KEEP

Core BufferedReader prelude binding                   REMOVE (D167 ratified)
Core BufferedWriter prelude binding                   REMOVE (D167 ratified)
standard buffered byte capability                     KEEP IN std:io (D167)

Process capability                                    KEEP
bootstrap-local Process authority                     KEEP
Process != operating-system process                  KEEP
process.args() capability                             KEEP
stable Process argument contents/order               KEEP
process.environment() capability                      KEEP
Environment host/native-name semantics               KEEP
process.stdin/stdout/stderr                          KEEP
Process standard-stream Encoding accessors           KEEP
non-transferable live resource is not auto-proxied   KEEP
Process termination revokes Process-local authority  KEEP

canonical Process args snapshot identity             REMOVE_NOW_RECONSIDER_LATER
canonical Process Environment snapshot identity      REMOVE_NOW_RECONSIDER_LATER
cross-Actor canonical re-observation via Process     REMOVE_NOW_RECONSIDER_LATER
special runtime-backed ProcessArguments family       REMOVE_NOW_RECONSIDER_LATER

ambient Process global                               ABSENT / RETAIN ABSENCE
external OS-process control in I/O                   ABSENT / RETAIN ABSENCE
implicit Process->Filesystem/Network authority       ABSENT / RETAIN ABSENCE
implicit P I/O authority                             ABSENT / RETAIN ABSENCE

IOError                                               KEEP
InvalidIOArgument                                     KEEP
IOLifecycleError                                      KEEP
IOCapacityExhausted                                   KEEP
EncodingError                                         KEEP
LineTooLong                                           KEEP
host errno/status prototype tree                     ABSENT / RETAIN ABSENCE
```

No D1 mechanism is classified `REMOVE_PERMANENTLY`.

## Common I/O operation model remains

Future state alone cannot answer whether an I/O effect is still cancellable.

A pending Future may already correspond to an externally committed operation.
Conversely, an admitted operation may still be provably zero-effect and therefore
safely cancellable.

The producer-side commitment distinction preserves both cases without adding a
fifth public Future state.

PLAT009 provides concrete implementation evidence for the first-effect race:
non-blocking or completion-oriented output may have an in-flight physical
attempt while the runtime still does not know whether zero or positive effect
occurred.

D117 provides the analogous wrapper evidence: an outer buffered operation cannot
infer zero lower effect from a still-pending lower Future.

This is internal correctness machinery behind the ordinary public shape:

```text
operation() -> Future
Future.cancel()
Future.value()
```

Classification: **KEEP**.

## Closable and lifecycle remain

Generic Closable avoids duplicating lifecycle semantics across File, TCP,
Process streams and wrappers.

The shared model owns:

- permanent close cutover;
- rejection/termination of open-required operations;
- committed-operation aftermath;
- one logical close outcome;
- fresh observation Futures;
- stable lifecycle failure outcome;
- wrapper release ordering.

Production code directly relies on `close().value()` and on cancellation-safe
late acquisition cleanup.

D112 demonstrates that close/Actor-termination interaction is a real semantic
correctness problem rather than decorative complexity.

Classification: **KEEP**.

## Wrapper borrowing and ownership remain

Borrowing and owning wrappers express real custody.

Production Tool code uses owning TextReader wrappers over Files, while Process
stream adapters borrow host/Process-owned standard streams.

Ownership is fixed at construction and does not amplify authority.

Classification: **KEEP**.

## Orthogonal byte capability decomposition remains

The standardized byte capability protocols remain:

```text
ByteReadable
ByteWritable
Flushable
ByteSeekable
ByteSized
Truncatable
Syncable
ReadShutdown
WriteShutdown
```

D1 keeps the **decomposition model**, not every resource-specific exposure.

Concrete D2 and D3 review will still determine whether File and TCP respectively
need each optional capability.

The decomposition avoids a universal Stream institution containing meaningless
or falsely implied operations.

Classification: **KEEP**.

## Encoding and text I/O remain

Encoding, TextReader and TextWriter have substantial current production use.

Explicit Encoding also preserves the already-retained String/Bytes boundary.

TextReader is more than repeated one-shot decode because lower reads can split
encoded scalar sequences and cancellation must reconcile already-read lower
bytes with decoder/read-ahead state.

TextWriter likewise owns one ordered encoder/output/lifecycle domain and must
compose lower write/flush/close effects correctly.

Classification: **KEEP**.

## Core BufferedReader / BufferedWriter are deferred

Approved classification:

```text
Core BufferedReader                         REMOVE_NOW_RECONSIDER_LATER
Core BufferedWriter                         REMOVE_NOW_RECONSIDER_LATER
standard Core byte-buffer wrapper family    REMOVE_NOW_RECONSIDER_LATER
```

The feature is implemented and semantically coherent, but no current production
consumer was found.

The continuing implementation burden is material. Approximate primary Java
surface at the D1 baseline:

```text
ProtosBufferedByteIo.java                     1320 lines
BufferedReader C-prime                         320 lines
BufferedWriter C-prime                         536 lines
standard buffered protocol                     197 lines
```

D117 and PLAT031 further show continuing cross-cutting maintenance around
commitment, cancellation, lower Future effects, close, flush, read-ahead,
output failure and C-prime ownership.

D1 does not reject buffering as a capability.

D167 must compare:

1. current Core wrappers;
2. removal with no immediate replacement;
3. a simpler Standard Library wrapper contract;
4. a Standard Library surface plus only the smallest privileged helper if a
   real required invariant cannot be expressed compositionally.

A literal relocation of the current implementation behind an `std:` name is
not by itself simplification.

### Route

D167 / #637 owns the exact decision.

### D167 ratification reconciliation

D167 / #637 subsequently selected **Candidate D**.

The approved result preserves the standard buffered byte capability but changes
its public placement:

```text
Core/prelude BufferedReader     REMOVE
Core/prelude BufferedWriter     REMOVE

std:io/BufferedReader           ADD / KEEP capability
std:io/BufferedWriter           ADD / KEEP capability

D117 strong delegated-effect semantics     KEEP
PLAT031 one-operation/lifecycle model       KEEP
private runtime support                     ALLOWED WHERE REQUIRED
```

Therefore the D1 `REMOVE_NOW_RECONSIDER_LATER` classification is only partially
realized: the **Core institution is removed**, but the capability itself is not
discarded or deferred. It becomes a Standard Library institution over retained
generic Core I/O mechanisms.

The D1 historical classification remains useful evidence for why Core placement
was challenged, but it is superseded as current authority by D167.

Implementation migration is owned by I060 / `guillermomolina/protos#663`.

Durable decision record:

`docs/project/decisions/language/D167_CORE_BUFFERED_BYTE_WRAPPER_PLACEMENT_AND_CONTRACT.md`

## Process and Process I/O remain

Process is already retained as an execution/failure/bootstrap domain and current
Tool code uses its standard I/O and argument capabilities.

D1 retains:

- explicit Process capability authority;
- no ambient global service locator;
- byte-oriented stdin/stdout/stderr;
- stable Process-local logical stream bindings;
- explicit Process-selected standard-stream Encodings;
- explicit Actor delegation;
- no implicit resource auto-proxy;
- Process termination as authority/custody revocation rather than successful
  close/flush/sync.

Classification: **KEEP**.

## Stable bootstrap content remains; canonical snapshot identity is deferred

D018 currently requires exactly one semantic arguments-snapshot identity and one
Environment-snapshot identity per Process.

D1 retains the stable content contract but classifies the identity promise:

```text
canonical arguments-snapshot identity          REMOVE_NOW_RECONSIDER_LATER
canonical Environment-snapshot identity        REMOVE_NOW_RECONSIDER_LATER
cross-Actor canonical re-observation           REMOVE_NOW_RECONSIDER_LATER
```

The useful stable-snapshot invariant does not itself require repeated accessor
results to compare `===`.

The current identity rule also creates a special distinction:

```text
transfer already-acquired snapshot
    -> ordinary isolation-copy identity

delegate Process, then call accessor
    -> re-observe one Process-global semantic identity
```

No current production consumer was found that needs this distinction.

D018 remains normative authority until D168 is ratified.

## Special ProcessArguments family is deferred

`process.args()` itself remains retained.

The special ProcessArguments family currently owns dedicated runtime
representation, protocol code, transfer branches, interop and structured
Bytecode/C-prime `each` handling.

A frozen standard Array already provides:

- ordered String values;
- zero-based indexing;
- size;
- each;
- shallow immutability;
- ordinary transfer behavior.

The non-equivalence is real: a frozen Array still exposes Array mutation
selectors which fail due to frozen state, whereas ProcessArguments exposes a
narrower positive protocol.

That trade-off requires D168 rather than silent substitution.

Environment is different and remains specialized because native
environment-variable name identity/representability semantics are not ordinary
Map semantics.

### Route

D168 / #638 owns the exact decision.

## Error model remains deliberately small

The retained portable categories are:

```text
IOError
InvalidIOArgument
IOLifecycleError
IOCapacityExhausted
EncodingError
LineTooLong
```

Cancellation remains an ordinary Future cancellation outcome rather than an
I/O Error category.

A host errno/status taxonomy remains absent.

Classification: **KEEP / RETAIN ABSENCE**.

## Strongest attempted removals

```text
remove generic I/O commitment
    rejected -> KEEP

use Future state as effect state
    rejected -> KEEP producer commitment separately

remove generic Closable
    rejected -> KEEP

use one canonical close Future
    rejected -> KEEP fresh Future observations

remove borrow/own wrapper distinction
    rejected -> KEEP

collapse capability protocols into universal Stream
    rejected -> RETAIN ABSENCE

make String/Bytes conversion implicitly UTF-8
    rejected -> KEEP explicit Encoding

remove TextReader/TextWriter
    rejected -> KEEP

replace Process authority with ambient global
    rejected -> KEEP explicit capability

auto-proxy non-transferable live resources across Actors
    rejected -> RETAIN ABSENCE

Core BufferedReader/BufferedWriter
    survives -> REMOVE_NOW_RECONSIDER_LATER

D018 canonical bootstrap snapshot identity
    survives -> REMOVE_NOW_RECONSIDER_LATER

special ProcessArguments value family
    survives -> REMOVE_NOW_RECONSIDER_LATER

special Environment host-backed semantics
    removal rejected -> KEEP
```

## Required AUD009 routing

```text
FEATURE=Core BufferedReader / BufferedWriter placement
CURRENT_OUTCOME=MOVE_TO_STDLIB
SEMANTIC_DECISION_OWNER=D167 / guillermomolina/protos#637 RATIFIED
IMPLEMENTATION_OWNER=I060 / guillermomolina/protos#663

FEATURE=Process bootstrap canonical snapshot identity
PROPOSED_OUTCOME=REMOVE_NOW_RECONSIDER_LATER
SEMANTIC_DECISION_OWNER=D168 / guillermomolina/protos#638
IMPLEMENTATION_OWNER=TBD_AFTER_D168_RATIFICATION

FEATURE=special ProcessArguments runtime family
PROPOSED_OUTCOME=REMOVE_NOW_RECONSIDER_LATER
SEMANTIC_DECISION_OWNER=D168 / guillermomolina/protos#638
IMPLEMENTATION_OWNER=TBD_AFTER_D168_RATIFICATION
```

## Boundary handoffs

- **D167 / #637** owns buffered-wrapper necessity and placement.
- **D168 / #638** owns Process bootstrap snapshot identity and argument
  representation.
- **AUD009-D2** owns Filesystem, Path, File, open, directory and captured-tree
  complexity, including concrete File seek/size/truncate/sync necessity.
- **AUD009-D3** owns Network/TCP complexity, including concrete directional
  half-close necessity.
- **AUD009-G** owns physical NIO/poller/C-prime/backend implementation
  complexity where it is not part of retained portable semantics.

## Owner approval and routing

```text
ISSUE=guillermomolina/protos#636
CHECKPOINT_COMMENT=5739573610
APPROVAL_COMMENT=5739586169
DATE=2026-09-19

CORE_BUFFERED_WRAPPERS=MOVE_TO_STDLIB
STANDARD_BUFFERED_BYTE_CAPABILITY=KEEP
BUFFERED_WRAPPER_DECISION=D167 / guillermomolina/protos#637 RATIFIED

PROCESS_BOOTSTRAP_CANONICAL_IDENTITY=REMOVE_NOW_RECONSIDER_LATER
PROCESS_ARGUMENTS_SPECIAL_FAMILY=REMOVE_NOW_RECONSIDER_LATER
PROCESS_SNAPSHOT_DECISION=D168 / guillermomolina/protos#638

NORMATIVE_CHANGE_AUTHORIZED_BY_D1=NO
IMPLEMENTATION_CHANGE_AUTHORIZED_BY_D1=NO
```

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_REVISION=05a8754e55c1ce5cb44effc621f1b454f1fa9697

D167_NATIVE_PARENT=#636 PASS
D168_NATIVE_PARENT=#636 PASS

D167_PROJECT_ROUTING=PASS
D168_PROJECT_ROUTING=PASS

REMOVAL_ROUTES=D167,D168
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_D1_CLASSIFICATION=COMPLETE
```

AUD009-D1 is complete once this durable record and required live GitHub closure
postconditions are verified.
