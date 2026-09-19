# AUD009-D2 — Filesystem, File, Path, and captured-tree complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#639`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence and closure revalidation revision:
`927f530f925f241bf6b1ec7eef7832bb4b18f674`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#639`, issue comment
`5739671499`.

Owner approval provenance: `guillermomolina/protos#639`, issue comment
`5739718132`, 2026-09-19.

Derived decision routes:

- `D169 / guillermomolina/protos#640` — Path rooted/parent traversal and
  file-URL bridge necessity.
- `D170 / guillermomolina/protos#641` — Advanced File append, seek, size,
  truncate, and sync surface.
- `D171 / guillermomolina/protos#642` — Public Filesystem captureTree versus
  host/runtime immutable custody.

None of the derived decisions is selected by this audit record.

## Purpose and boundary

AUD009-D2 reviewed the filesystem-specific layer built on the retained D1 I/O
operation/lifecycle model:

- Filesystem authority;
- Path representation and traversal breadth;
- File acquisition/open semantics;
- optional advanced File capabilities;
- namespace replacement/removal;
- directory observation;
- captured-tree semantics;
- URL/filesystem conversion.

Networking remains AUD009-D3. Physical filesystem backend architecture remains
AUD009-G except where current backend shape is evidence for present necessity.

AUD009-D2 itself authorizes no normative or implementation change.

## Evidence summary

### Production use

Current production Protos code materially uses:

```text
Filesystem.open
Path.relative
Path.child
Filesystem.entries
Filesystem.replace
Filesystem.remove
File.read
File.write
File.close
```

Package Tool and Test Tool use Filesystem capabilities for explicit authority.
Package metadata publication uses:

```text
createNew staging File
write
close
replace staging -> target
```

Package ContentIdentity and Test Tool resource discovery use
`Filesystem.entries`.

### No production use found for rooted/parent Path breadth

No production `protos/lib` or `protos/tools` use was found for:

```text
Path.rooted()
Path.parentComponent()
```

Current production NIO Filesystem backends reject rooted Paths and parent
components; production source uses relative downward normal components.

### No implemented file-URL bridge

The normative specification describes a conceptual
`filesystem.pathFromURL(url)`, but:

- no implementation exists;
- no public selector exists;
- no public Core URL binding exists;
- the exact spelling remains explicitly open;
- no production consumer exists.

### No production use/backend exposure for advanced File capabilities

No production `protos/lib` or `protos/tools` use was found for:

```text
File.position()
File.seek(...)
File.seekBy(...)
File.seekToEnd()
File.size()
File.truncate(...)
File.sync()
append: true
```

Current production NIO Filesystem backends expose seekable/sized/truncatable/
syncable as false.

The File runtime nevertheless contains dedicated request/backend paths for these
operations. Append additionally requires cross-File coordination across
distinct handles that alias one underlying resource.

### captureTree distinction

No production Protos source invokes `Filesystem.captureTree`.

Immutable captured-tree machinery is nevertheless used by production Package
Tool architecture through host/runtime custody:

```text
already-selected package root
    -> ProtosCapturedFilesystemCustody.captureSelectedRoot(...)
    -> immutable captured backing
    -> read-only Filesystem view
    -> bundled-Protos ContentIdentity verification
    -> same verified custody retained for package source resolution
```

PLAT012 requires this verified immutable custody and prohibits reopening the
original mutable store/source Path after verification.

AUD009-D2 therefore distinguishes the public general
`Filesystem.captureTree` selector from retained host/tool immutable custody.

## Final classification ledger

```text
Filesystem capability                                 KEEP
Filesystem authority confinement                     KEEP
no ambient/global Filesystem                         KEEP

Path value                                            KEEP
Path.relative                                         KEEP
Path.child                                            KEEP
Path structural equality                             KEEP
no implicit String -> Path coercion                  KEEP
Path.rooted                                           REMOVE (D169 ratified)
Path.parentComponent                                  REMOVE (D169 ratified)

file-URL -> Path Core bridge                         REMOVE (D169 ratified)

Filesystem.open                                      KEEP
ordinary-object open options                         KEEP
read/write access dimensions                         KEEP
existing/create/createNew                            KEEP
truncate-on-open                                     KEEP
positioned ordinary File I/O                         KEEP
append open mode                                     REMOVE_NOW_RECONSIDER_LATER

stable File selected-resource binding                KEEP
per-File ordering/lifecycle                          KEEP
internal sequential logical position                 KEEP
File readable/writable/closable                      KEEP

File ByteSeekable exposure                           REMOVE_NOW_RECONSIDER_LATER
File ByteSized exposure                              REMOVE_NOW_RECONSIDER_LATER
File Truncatable exposure                            REMOVE_NOW_RECONSIDER_LATER
File Syncable exposure                               REMOVE_NOW_RECONSIDER_LATER

Filesystem.replace                                   KEEP
Filesystem.remove                                    KEEP
namespace final-entry no-follow                      KEEP
namespace atomic/failure-atomic visibility           KEEP
namespace crash-durability guarantee                 ABSENT / RETAIN ABSENCE

Filesystem.entries                                   KEEP
ordinary frozen entry descriptors                    KEEP
regular/directory/link/other kinds                   KEEP
entries ordering unspecified                        KEEP
incremental directory cursor/stream                  ABSENT / RETAIN ABSENCE

public Filesystem.captureTree                        REMOVE_NOW_RECONSIDER_LATER
PLAT012 verified immutable package custody           KEEP

Directory prototype                                  ABSENT / RETAIN ABSENCE
DirectoryEntry prototype                             ABSENT / RETAIN ABSENCE
general stat/exists                                  ABSENT / RETAIN ABSENCE
mkdir                                                ABSENT / RETAIN ABSENCE
general rename/move                                  ABSENT / RETAIN ABSENCE
recursive remove                                     ABSENT / RETAIN ABSENCE
symlink management                                   ABSENT / RETAIN ABSENCE
```

No D2 mechanism is classified `REMOVE_PERMANENTLY`.

## Filesystem authority remains Core

Filesystem is present authority, not naming convenience.

Current Package Tool and Test Tool execution receives explicit Filesystem
capabilities. Current host backends use pinned/secure roots and reject uncertain
confinement.

Removing Filesystem authority would push filesystem access toward ambient host
paths, Process-global state, or tool-specific intrinsics.

Classification: **KEEP**.

## Minimal Path remains

Path remains distinct from String because one Path normal component must remain
one logical namespace child regardless of host separators, drive syntax, Unicode
normalization or native path rules.

The retained present surface is:

```text
Path.relative()
path.child(name)
structural equality/hash
```

It is sufficient for current production use and composes with explicitly
provisioned Filesystem bases.

Classification: **KEEP**.

## Rooted and parent traversal are deferred

Approved classification:

```text
Path.rooted()                 REMOVE_NOW_RECONSIDER_LATER
Path.parentComponent()        REMOVE_NOW_RECONSIDER_LATER
rooted/relative Path breadth  REMOVE_NOW_RECONSIDER_LATER where not otherwise
                              required by a retained public facility
parent-component kind         REMOVE_NOW_RECONSIDER_LATER
```

No production consumer uses these operations and current production NIO
backends reject them.

Their presence forces portable rules for authority-root versus base, upward
traversal, non-lexical collapse, symlink/reparse interaction and backend
confinement that current applications do not need.

A future facility can add rooted/parent traversal without invalidating existing
relative downward Paths.

### Route

D169 / #640 owns the exact decision.

### D169 ratification reconciliation

D169 / #640 subsequently selected **Candidate B — minimal relative/downward
Core Path**.

The D2 removal classification is therefore ratified for the public Core surface:

```text
Path.rooted()                         REMOVE
rooted/relative Path flag             REMOVE
Path.parentComponent()                REMOVE
Parent component kind                 REMOVE
Core file-URL -> Path semantics       REMOVE
conceptual filesystem.pathFromURL     REMOVE

Path.relative()                       KEEP
Path.child(name)                      KEEP
Path structural equality/hash         KEEP
Filesystem explicit authority         KEEP
Filesystem confinement                KEEP
```

D037's structural/filesystem-independent equality principle remains authority,
but D169 explicitly narrows the surviving Path structure to the ordered sequence
of normal component Strings. Rootedness and Parent component kind no longer
participate because those semantic dimensions are removed.

The historical D2 classification remains useful evidence but is superseded as
current authority by D169.

Implementation migration is owned by I062 / `guillermomolina/protos#665`.

Durable decision record:

`docs/project/decisions/language/D169_PATH_ROOTED_PARENT_AND_FILE_URL_NECESSITY.md`

## File-URL conversion is deferred

Approved classification:

```text
Core file-URL -> Path conversion semantics
    REMOVE_NOW_RECONSIDER_LATER
```

The current normative text predefines URL parsing/decoding/authority/confinement
rules before a public API or implementation exists.

The retained safety invariant is simpler:

> Path, String and URI/URL-like values do not themselves grant Filesystem
> authority.

A future bridge can be designed against the then-current URI/URL library and
Filesystem capability model.

### Route

D169 / #640 owns the exact decision together with Path traversal breadth.

## Basic File/open remains

Current production code uses read-existing, createNew, create + truncate-on-open,
File read/write and close.

AUD009-D2 retains:

- stable captured open options;
- preflight validation;
- race-safe existing/create/createNew acquisition;
- empty new-file semantics;
- failure-atomic truncate-on-open;
- open commitment/cancellation/resource custody;
- stable selected-resource File binding;
- independence of separately opened File capabilities.

The Path used for acquisition remains a name/selector, not ongoing File identity.

Classification: **KEEP**.

## Advanced File surface is deferred

Approved classification:

```text
append mode                       REMOVE_NOW_RECONSIDER_LATER
File ByteSeekable                 REMOVE_NOW_RECONSIDER_LATER
File ByteSized                    REMOVE_NOW_RECONSIDER_LATER
File Truncatable                  REMOVE_NOW_RECONSIDER_LATER
File Syncable                     REMOVE_NOW_RECONSIDER_LATER
```

This does not reopen D1's generic protocol decomposition.

The decision concerns whether standard File v0.1 should expose these optional
capabilities now.

No current production consumer or production backend exposure was found.

The continuing semantic/runtime cost includes seek/end-position/size/truncate/
sync request paths, durability commitment, append-specific backend interfaces,
append logical-position rules and cross-alias append coordination.

`truncate: true` at open remains retained because current Files helpers use it.

### Route

D170 / #641 owns the exact decision.

## Atomic namespace mutation remains

`Filesystem.replace` and `remove` are current production operations.

Atomic replace is required for prepared-file publication without exposing a
partial target. Final-entry no-follow selection is also an authority/safety
property.

The contract correctly distinguishes live atomic namespace visibility from
crash durability.

Classification: **KEEP**.

## Directory observation remains

`Filesystem.entries` is used by current Package Tool and Test Tool code.

Its result uses ordinary Array + frozen ordinary descriptors rather than new
Directory/DirectoryEntry institutions.

Unspecified order is retained; callers that need canonical order sort
explicitly.

Classification: **KEEP**.

## Public captureTree is deferred; verified immutable package custody remains

Approved classification:

```text
public Filesystem.captureTree(path)
    REMOVE_NOW_RECONSIDER_LATER

PLAT012 verified immutable package custody
    KEEP
```

The public D046 operation generalizes immutable recursive capture into a
language-wide Filesystem capability despite no current guest-code consumer.

Current Package Tool requirements are narrower and already use host/runtime
custody over an exact selected package root.

Any implementation following D171 must preserve:

```text
same immutable verified package backing
no original source/store reopen after verification
run-scoped custody
fresh Actor-domain read-only Filesystem views may borrow the backing
ContentIdentity verifies the exact custody later used for source reads
```

### Route

D171 / #642 owns the exact decision.

## Strongest attempted removals

```text
remove Filesystem authority
    rejected -> KEEP

use String as Path
    rejected -> KEEP Path

remove structural Path equality
    rejected -> KEEP

Path.rooted
    survives -> REMOVE_NOW_RECONSIDER_LATER

Path.parentComponent
    survives -> REMOVE_NOW_RECONSIDER_LATER

file-URL -> Path bridge
    survives -> REMOVE_NOW_RECONSIDER_LATER

remove Filesystem.open
    rejected -> KEEP

remove create/createNew/truncate-on-open
    rejected -> KEEP; current production use and atomic acquisition semantics

append-mode File
    survives -> REMOVE_NOW_RECONSIDER_LATER

File seek/size/truncate/sync exposure
    survives -> REMOVE_NOW_RECONSIDER_LATER

remove replace/remove
    rejected -> KEEP; production atomic publication use

replace atomic replace with source-level copy/delete
    rejected -> cannot preserve atomic publication

remove Filesystem.entries
    rejected -> KEEP; production consumers

canonically sort entries in Core
    rejected -> retain unspecified order

public Filesystem.captureTree
    survives -> REMOVE_NOW_RECONSIDER_LATER

remove immutable verified package custody itself
    rejected -> KEEP; PLAT012 and package verification require it
```

## Required AUD009 routing

```text
FEATURE=Path rooted/parent traversal + file-URL bridge
CURRENT_OUTCOME=REMOVE
SEMANTIC_DECISION_OWNER=D169 / guillermomolina/protos#640 RATIFIED
IMPLEMENTATION_OWNER=I062 / guillermomolina/protos#665

FEATURE=advanced File append/seek/size/truncate/sync surface
PROPOSED_OUTCOME=REMOVE_NOW_RECONSIDER_LATER
SEMANTIC_DECISION_OWNER=D170 / guillermomolina/protos#641
IMPLEMENTATION_OWNER=TBD_AFTER_D170_RATIFICATION

FEATURE=public Filesystem.captureTree
PROPOSED_OUTCOME=REMOVE_NOW_RECONSIDER_LATER
SEMANTIC_DECISION_OWNER=D171 / guillermomolina/protos#642
IMPLEMENTATION_OWNER=TBD_AFTER_D171_RATIFICATION

FEATURE=PLAT012 verified immutable package custody
PROPOSED_OUTCOME=KEEP
```

## Boundary handoffs

- **D169 / #640** owns rooted/parent Path breadth and file-URL bridge.
- **D170 / #641** owns advanced standard File exposure.
- **D171 / #642** owns the public captureTree boundary while preserving PLAT012.
- **AUD009-D3** owns Network/TCP complexity.
- **AUD009-G** owns physical filesystem backend/capture-store/NIO factoring not
  retained as portable semantics.

## Owner approval and routing

```text
ISSUE=guillermomolina/protos#639
CHECKPOINT_COMMENT=5739671499
APPROVAL_COMMENT=5739718132
DATE=2026-09-19

PATH_ROOTED_PARENT_AND_FILE_URL=REMOVE
PATH_DECISION=D169 / guillermomolina/protos#640 RATIFIED

ADVANCED_FILE_SURFACE=REMOVE_NOW_RECONSIDER_LATER
FILE_DECISION=D170 / guillermomolina/protos#641

PUBLIC_CAPTURE_TREE=REMOVE_NOW_RECONSIDER_LATER
CAPTURE_TREE_DECISION=D171 / guillermomolina/protos#642

PLAT012_VERIFIED_IMMUTABLE_PACKAGE_CUSTODY=KEEP

NORMATIVE_CHANGE_AUTHORIZED_BY_D2=NO
IMPLEMENTATION_CHANGE_AUTHORIZED_BY_D2=NO
```

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_REVISION=927f530f925f241bf6b1ec7eef7832bb4b18f674

D169_NATIVE_PARENT=#639 PASS
D170_NATIVE_PARENT=#639 PASS
D171_NATIVE_PARENT=#639 PASS

D169_PROJECT_ROUTING=PASS
D170_PROJECT_ROUTING=PASS
D171_PROJECT_ROUTING=PASS

REMOVAL_ROUTES=D169,D170,D171
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_D2_CLASSIFICATION=COMPLETE
```

AUD009-D2 is complete once this durable record and required live GitHub closure
postconditions are verified.
