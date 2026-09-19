# D170 — Advanced File append, seek, size, truncate, and sync surface

Status: **RATIFIED — Candidate C**

Approval date: **2026-09-19**  
Decision issue: `guillermomolina/protos#641`  
Trigger: AUD009-D2 / `guillermomolina/protos#639`  
Protos evidence revision: `3968473f9117fc9f4b4a5f2a347b7b2e15ce7969`  
Project-record base: `56ef506c9aa5268658b3484a9d789f3ca2f02187`

This is a durable non-normative decision record. Observable Protos semantics remain authoritative under `guillermomolina/protos:spec/**`.

## Decision

D170 selects **Candidate C — retain the evidence-backed primitive File capabilities and remove append for now**.

```text
ordinary File read/write/close                     KEEP
internal logical sequence position                 KEEP
truncate-on-open                                   KEEP
generic I/O commitment/cancellation/lifecycle      KEEP

File ByteSeekable exposure                         KEEP
File.position()                                    KEEP
File.seek(position)                                KEEP
File.seekBy(offset)                                KEEP
File.seekToEnd()                                   KEEP

File ByteSized exposure                            KEEP
File.size()                                        KEEP

File Truncatable exposure                          KEEP
File.truncate(size)                                KEEP

File Syncable exposure                             KEEP
File.sync()                                        KEEP
File.sync != namespace crash durability            KEEP

append open option                                 REMOVE_NOW_RECONSIDER_LATER
append-specific File mode                          REMOVE_NOW_RECONSIDER_LATER
AppendWritableResource                             REMOVE_NOW_RECONSIDER_LATER
mandatory cross-File same-resource append
placement coordination                             REMOVE_NOW_RECONSIDER_LATER

ByteSeekable protocol family                       KEEP
ByteSized protocol family                          KEEP
Truncatable protocol family                        KEEP
Syncable protocol family                           KEEP
```

## Core placement

D170 explicitly confirms that `ByteSeekable`, `ByteSized`, `Truncatable`, and `Syncable` belong in Core as primitive capability contracts.

These are not Standard Library conveniences:

- a library cannot synthesize true random-access positioning if the underlying resource cannot provide it;
- a library cannot fabricate a current-size query with correct resource semantics;
- a library cannot truncate an underlying resource without a primitive resource operation;
- a library cannot manufacture a durability barrier when the backend exposes no meaningful one.

The standard File therefore remains allowed to expose these protocols conditionally when its backend can honestly satisfy them.

A File/backend that cannot provide one of these capabilities does not expose that protocol.

## Why AUD009-D2 is only partially superseded

AUD009-D2 provisionally classified all of:

```text
append
File ByteSeekable
File ByteSized
File Truncatable
File Syncable
```

as `REMOVE_NOW_RECONSIDER_LATER`.

D170 supersedes that classification for the four orthogonal File capabilities:

```text
File ByteSeekable  -> KEEP
File ByteSized     -> KEEP
File Truncatable   -> KEEP
File Syncable      -> KEEP
```

The D2 removal classification remains accepted for append.

The reason is not historical implementation cost. It is present architectural fit and reintroduction cost.

Seek, size, truncate and sync are primitive, conditional, backend-honesty capabilities. They impose little/no public/runtime cost on Files that do not provide them and map directly onto resource abilities that a source library cannot honestly emulate.

Append is materially different because the current Protos append contract imposes special cross-File coordination and append-placement semantics across distinct File capabilities selecting the same underlying resource.

## Primitive File capability rationale

### ByteSeekable

File already maintains a logical sequence position for ordinary sequential reads/writes.

When the backend additionally supports the standard positioning contract, exposing:

```text
position
seek
seekBy
seekToEnd
```

is a direct capability extension over state File already conceptually owns.

The capability remains optional and backend-dependent.

### ByteSized

`size()` is a direct resource observation.

It introduces no new authority, identity, lifecycle or cross-File ordering domain.

A File exposes it only when the backend can provide the standard current-size semantics.

### Truncatable

`truncate(size)` is a primitive mutation of the selected resource.

It cannot be reconstructed honestly in Standard Library code from ordinary read/write alone while preserving the same resource semantics.

Its exposure remains conditional on writable authority and backend support.

This is distinct from retained `truncate: true` at open time; support for one does not imply the other.

### Syncable

`sync()` is the primitive durability boundary when a backend has one.

It remains deliberately scoped:

```text
File.sync()
    != parent-directory sync
    != namespace durability
    != replace/remove crash-durability guarantee
    != global Filesystem durability
```

Keeping Syncable in Core preserves the ability to build databases, logs, journals, WAL-like facilities and durable publication protocols without pretending that ordinary write completion already means durable storage.

## Why append is removed for now

The current append contract is stronger than an initial `seekToEnd()` followed by ordinary positioned write.

It requires a special append-placement invariant across separately opened File capabilities that select the same underlying filesystem resource:

```text
concurrent Protos append writes
    -> relative order may be nondeterministic
    -> contributed byte sequences do not overlap/interleave
    -> partial failed append does not reserve an uncommitted suffix
```

This can require shared backend coordination across distinct handles and aliases.

The current runtime reflects that semantic institution with append-specific capability state and a dedicated `AppendWritableResource` path.

No production `protos/lib` or `protos/tools` consumer currently requires append, and current production filesystem backends do not expose the advanced File capabilities under review.

Reintroducing append later is additive over retained Filesystem/File/open/ByteWritable semantics. A future append decision can then choose, from concrete consumer/backend evidence, whether Protos should guarantee:

- the current strong cross-handle non-interleaving contract;
- host/backend append semantics;
- a bounded atomic append unit;
- a serialized append service/capability; or
- another narrower model.

D170 therefore removes the current append institution now without declaring append intrinsically contrary to Protos.

## Comparative evidence

The decision packet compared multiple established approaches, including POSIX file descriptors, Java NIO channels, Rust File/Seek, Go os.File, .NET FileStream, Python/Node file APIs and related host/filesystem semantics.

The recurring pattern is that random access, size/truncation and durability are direct resource capabilities, while append semantics differ materially across hosts and APIs in their atomicity/interleaving guarantees.

That distinction supports retaining the four orthogonal primitive capabilities while deferring Protos-specific append coordination until justified by an actual consumer/backend contract.

## Candidate result

### Candidate A — retain the complete advanced File surface

Rejected.

It keeps useful primitive capabilities but also freezes the current strong append institution and its cross-File coordination without current evidence that this exact append guarantee is needed.

### Candidate B — remove the complete advanced File surface

Rejected.

It treats optional primitive resource capabilities as expendable merely because current production code does not consume them. Their later reintroduction would recreate basic Core resource primitives without materially simplifying current Files that do not expose them.

### Candidate C — retain seek/size/truncate/sync; remove append

**Selected.**

This preserves Core primitive resource capability breadth while removing the one current advanced File feature that creates a special cross-capability coordination contract.

### Candidate D — move source-expressible conveniences outward

Rejected as the primary answer.

Standard Library may build conveniences on retained primitives, but it cannot honestly synthesize seekability, current-size semantics, resource truncation or a durability barrier when the underlying File/backend does not provide them.

## Strongest argument against Candidate C

Append is also a common file-system operation and likely future use cases include logs, journals and append-only records.

Removing it now may require future redesign/reimplementation.

D170 accepts that cost because the question is not whether append is useful. The current semantic commitment is specifically a strong cross-File/cross-alias placement guarantee. There is insufficient present evidence that every future standard File backend should provide or emulate that exact guarantee.

The retained File/open/ByteWritable architecture leaves future append additive rather than foundational.

## Pay-for-what-you-need

```text
ordinary File
    -> readable/writable/closable only as applicable

seekable backend
    -> may expose ByteSeekable

sized backend
    -> may expose ByteSized

truncatable writable backend
    -> may expose Truncatable

backend with meaningful durability boundary
    -> may expose Syncable

backend without those capabilities
    -> pays no public selector obligation for them
```

D170 does not require production backends to begin exposing capabilities they do not currently implement.

## Grow-as-you-need

Future work may:

- add concrete backend support for any retained optional File capability;
- add Standard Library convenience operations composed from retained primitives;
- reintroduce append through a new Dxxx when a real consumer/backend contract justifies its exact atomicity and coordination semantics.

No append syntax/option/hook is reserved by D170 after implementation reconciliation.

## Fixed-authority consistency

D170 preserves all fixed authority:

```text
D1 generic ByteSeekable family                     KEEP
D1 generic ByteSized family                        KEEP
D1 generic Truncatable family                      KEEP
D1 generic Syncable family                         KEEP
ordinary File read/write/close                     KEEP
File internal logical position                     KEEP
truncate-on-open                                   KEEP
Filesystem create/createNew                       KEEP
stable selected-resource binding                   KEEP
generic I/O commitment/cancellation/lifecycle      KEEP
```

Only the File append institution is removed.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Implementation consequence

D170 requires bounded specification/runtime/test/documentation reconciliation for append removal only.

The implementation owner must:

1. remove `append: true` from standard Filesystem open semantics/options;
2. remove append-specific File semantics and capability state;
3. remove `AppendWritableResource` and append-only runtime/backend paths that exist solely for the removed standard contract;
4. remove cross-File/cross-alias append coordination tests/requirements;
5. preserve ordinary positioned writes and internal logical sequence position;
6. preserve File ByteSeekable/ByteSized/Truncatable/Syncable protocols and conditional exposure exactly;
7. preserve truncate-on-open;
8. preserve generic I/O commitment/cancellation/lifecycle contracts;
9. introduce no replacement append helper or Standard Library facade in the same slice.

```text
D170_STATUS=RATIFIED
SELECTED_CANDIDATE=C

FILE_BYTE_SEEKABLE=KEEP
FILE_BYTE_SIZED=KEEP
FILE_TRUNCATABLE=KEEP
FILE_SYNCABLE=KEEP

APPEND_OPEN_MODE=REMOVE_NOW_RECONSIDER_LATER
APPEND_FILE_SEMANTICS=REMOVE_NOW_RECONSIDER_LATER
APPEND_WRITABLE_RESOURCE=REMOVE_NOW_RECONSIDER_LATER
APPEND_CROSS_FILE_COORDINATION=REMOVE_NOW_RECONSIDER_LATER

NORMATIVE_RECONCILIATION_REQUIRED=YES
IMPLEMENTATION_RECONCILIATION_REQUIRED=YES
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Approval provenance

The exact Candidate C boundary was presented to the project owner in the active interaction on 2026-09-19, including the explicit clarification that the four retained protocols/exposures belong in Core rather than Standard Library.

The project owner explicitly approved it:

```text
ok aprobado
```
