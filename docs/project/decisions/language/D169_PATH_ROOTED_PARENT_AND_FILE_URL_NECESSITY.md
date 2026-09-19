# D169 — Path rooted/parent traversal and file-URL bridge necessity

Status: **RATIFIED — Candidate B (minimal relative/downward Core Path)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#640`
Trigger: AUD009-D2 / `guillermomolina/protos#639`
Protos evidence revision: `5ce8e039a69489a49fe446d58de7fb39bcbb278f`
Project-record base: `f5061307d966b8427cc5a32c490daded37fae54c`

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative only through the applicable ratified material under
`guillermomolina/protos:spec/**`.

## Decision

D169 selects **Candidate B — minimal relative/downward Core Path**.

Core Path v0.1 retains exactly the portable authority-free structure needed by
current capability-confined Filesystems:

```text
Path.relative()
path.child(name)
immutable Path value
structural equality/hash
one normal component = one exact semantic String
no implicit String -> Path coercion
no host separator/drive/UNC interpretation
```

The following current/future-facing breadth is removed from Core v0.1:

```text
Path.rooted()
rooted/relative Path flag
Path.parentComponent()
Parent path-component kind
Core file-URL -> Path conversion semantics
conceptual filesystem.pathFromURL(...)
```

These removals are **REMOVE_NOW_RECONSIDER_LATER**, not permanent prohibitions.
A later evidence-backed decision may reintroduce substantially similar capability
if a concrete Filesystem/backend/user requirement demonstrates it.

## Retained authority boundary

D169 does not weaken the existing Filesystem/Path capability model:

```text
Path
    -> inert structural value
    -> carries no Filesystem authority

Filesystem
    -> explicit namespace authority

File
    -> already-selected resource authority
```

The following remain fixed:

- Filesystem remains explicit authority;
- Path remains distinct from String;
- Path construction performs no filesystem access;
- Path structural equality remains filesystem-independent;
- one String supplied to `child(name)` remains one logical normal component;
- host slash, backslash, drive, UNC, device-prefix and similar syntax inside a
  component do not gain structural meaning;
- no implicit String-to-Path conversion exists;
- no Path, URI or URL-like value grants ambient filesystem/network authority;
- Filesystem resolution must remain confined to its explicitly provisioned
  namespace authority.

## D037 consistency and explicit narrowing

D037 established **structural filesystem-independent Path equality**.

That invariant remains.

D169 explicitly narrows the structure that participates in equality.

Before D169, D037's concrete structure included:

```text
rooted/relative flag
+
ordered sequence of component kinds/values
```

After D169 the surviving Core v0.1 structure is:

```text
ordered sequence of normal component String values
```

Therefore:

```text
STRUCTURAL_FILESYSTEM_INDEPENDENT_EQUALITY = PRESERVED
ROOTEDNESS_AS_EQUALITY_DIMENSION            = REMOVED
PARENT_COMPONENT_KIND_AS_EQUALITY_DIMENSION = REMOVED
```

This is not a hidden contradiction. The candidate presented for approval
explicitly stated that after D169 structural equality remains D037 authority
while the structure reduces to ordered normal components. The project owner
approved that exact candidate.

## Why rooted Paths are removed now

Rooted Paths are not intrinsically invalid design.

Many conventional path APIs distinguish relative and absolute/rooted paths.

The Protos question is narrower: whether Core must standardize the following
two-anchor model today:

```text
Path.relative() -> Filesystem configured base
Path.rooted()   -> Filesystem namespace root
```

Current production evidence does not require that distinction:

- production Protos source uses relative downward Paths;
- current production Filesystem backends are provisioned around one explicit
  authority root;
- those backends reject rooted Paths;
- no production Standard Library or Tool consumer uses `Path.rooted()`;
- no retained operation requires a separate public base-versus-root distinction.

A future requirement may justify rootedness, but it may also choose a different
capability-native model, for example deriving/provisioning a different
Filesystem whose own base is the desired namespace root.

Keeping `Path.rooted()` today would therefore preselect a public model before
the project has a consumer proving that a rootedness bit belongs in Path rather
than in Filesystem capability construction/derivation.

## Why parent traversal is removed now

`Path.parentComponent()` carries more semantic cost than its implementation
size suggests.

Correct backend interpretation cannot generally collapse:

```text
a / b / parent / c
```

into:

```text
a / c
```

before resolution, because intermediate namespace indirection may make them
observably different.

Supporting parent components therefore forces every conforming backend to define
or reject parent traversal under:

- symlink/reparse indirection;
- authority confinement;
- concurrent namespace mutation;
- virtual/mounted namespaces;
- race-safe traversal;
- host-specific namespace behavior.

No production Filesystem backend currently needs that capability.

There is also direct project evidence that navigation syntax need not become
persistent Path structure: Package Tool path-dependency resolution accepts
`.`/`..` in its own domain, proves confinement against the resolution root,
and then produces a clean canonical relative/downward Path.

This demonstrates an incremental alternative:

```text
domain-specific navigation syntax
        -> resolve/validate in owning domain
        -> canonical relative/downward Path
```

D169 therefore does not reserve parent traversal in Core merely because future
domains may use `..` syntax.

## Why Core file-URL conversion is removed now

The current file-URL bridge is specification-only:

- no public selector exists;
- no runtime implementation exists;
- no production consumer exists;
- the exact selector spelling is explicitly open.

Despite that, the specification already commits to detailed behavior for:

- `file:` recognition;
- URL authority;
- URL hierarchy;
- percent decoding;
- dot-segment treatment;
- component conversion;
- UNC-like authority handling;
- lossless filename mapping; and
- Filesystem confinement.

Protos now also has ordinary URI library work, making it more appropriate to
design any future file-URI bridge against the then-current URI and Filesystem
models instead of freezing a large Core contract before an API exists.

The only safety rule retained now is the fundamental authority invariant:

> A URI/URL-like value and a Path are data. They do not grant Filesystem or
> network authority.

A future explicit bridge may live in Filesystem, `std:io`, `std:uri`, host
interop or another suitable layer depending on the concrete requirement.

## Approval provenance

AUD009-D2 first classified:

```text
Path.rooted()                    REMOVE_NOW_RECONSIDER_LATER
Path.parentComponent()           REMOVE_NOW_RECONSIDER_LATER
file-URL -> Path Core bridge     REMOVE_NOW_RECONSIDER_LATER
```

D169 then performed the required dedicated comparison.

The exact recommendation presented to the project owner was:

```text
D169 = Candidate B — minimal relative/downward Core Path

KEEP:
    Path
    Path.relative()
    Path.child(name)
    immutable Path value
    structural equality/hash
    one String = one normal component
    no implicit String -> Path
    Filesystem explicit authority
    capability confinement

REMOVE_NOW_RECONSIDER_LATER:
    Path.rooted()
    rooted/relative flag
    Parent component kind
    Path.parentComponent()
    Core file-URL -> Path conversion semantics
    conceptual filesystem.pathFromURL(...)

RETAIN ABSENCE:
    host-path parsing in Core
    implicit / \ drive UNC semantics
    ambient filesystem authority
```

The recommendation also explicitly stated that D037 structural equality remains,
but the surviving structure reduces to ordered normal components.

The project owner then explicitly approved the candidate:

```text
ok aprobado
```

```text
SELECTED_CANDIDATE=B
DECISION_APPROVAL_PROVENANCE=PASS
```

## GITHUB021 invariant/delta consistency

Applicable fixed authority:

```text
FILESYSTEM_EXPLICIT_AUTHORITY                     PRESERVED
PATH_REMAINS_AUTHORITY_FREE_VALUE                 PRESERVED
PATH_DISTINCT_FROM_STRING                         PRESERVED
PATH_RELATIVE                                     PRESERVED
PATH_CHILD                                        PRESERVED
PATH_STRUCTURAL_EQUALITY                          PRESERVED
FILESYSTEM_INDEPENDENT_PATH_EQUALITY              PRESERVED
ONE_STRING_ONE_COMPONENT                          PRESERVED
NO_IMPLICIT_STRING_TO_PATH                        PRESERVED
NO_HOST_SYNTAX_IMPLICIT_STRUCTURE                 PRESERVED
NO_AMBIENT_AUTHORITY_FROM_PATH_OR_URL             PRESERVED
```

Explicitly reopened/narrowed D037 consequence:

```text
ROOTEDNESS_IN_PATH_STRUCTURE                      REMOVED
PARENT_COMPONENT_KIND_IN_PATH_STRUCTURE           REMOVED
```

Observable API/semantic delta:

```text
Path.rooted()                                     REMOVED
Path.parentComponent()                            REMOVED
rooted/relative flag                              REMOVED
Parent component kind                             REMOVED
Core file-URL conversion contract                 REMOVED
conceptual filesystem.pathFromURL(...)            REMOVED

Path.relative()                                   UNCHANGED
Path.child(name)                                  UNCHANGED
structural equality/hash of surviving Paths       UNCHANGED IN PRINCIPLE
Filesystem authority/confinement                  UNCHANGED
```

No materially new consequence was introduced after the exact candidate was
presented for approval.

```text
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Comparative evidence

The investigation compared conventional host-path models, capability-oriented
filesystem models, portable virtual-filesystem naming and URL/path bridges.

### Java NIO

Java Path supports rooted/absolute forms and parent traversal in a
provider/filesystem-specific path model.

Contribution: rooted/parent forms are mature and useful when one Path abstraction
must describe a broad host/provider namespace, but this does not establish that
a capability-relative Core Path must standardize them before use.

### .NET System.IO

.NET path handling includes rooted/absolute host-path forms and platform-specific
root behavior.

Contribution: conventional host-path APIs gain practical breadth at the price of
platform/path-root rules that Protos deliberately keeps behind Filesystem
authority.

### Rust std::path

Rust Path/PathBuf preserve platform path structure including parent components
and roots. Lexical parent cleanup is not generally equivalent to filesystem
resolution because symbolic indirection may matter.

Contribution: confirms that parent components carry real resolution semantics;
their implementation is not merely one enum case.

### Rust cap-std

Capability-oriented directory APIs operate relative to explicit directory
authority and make escaping/ambient authority a separate concern.

Contribution: strong precedent that a capability model can remain useful with
relative paths as the normal public form and express broader namespace authority
through capability provisioning rather than absolute path data.

### Python pathlib/os

Python provides rich rooted/absolute and file-URI behavior coupled to concrete
host-platform path rules.

Contribution: demonstrates the usefulness of those features but also that
file-URI/path conversion inevitably acquires platform-specific policy that Protos
need not freeze before a consumer exists.

### Go io/fs and path/filepath

Go distinguishes a portable virtual-filesystem path model from native
`filepath` behavior. Portable `io/fs` names are unrooted, slash-separated and
exclude `.`/`..` elements.

Contribution: direct evidence that a useful portable filesystem abstraction can
standardize a deliberately narrower relative/downward naming model while native
host-path breadth lives elsewhere.

### Swift/Foundation

Swift/Foundation path/URL facilities provide rooted/native file-path and file URL
interoperation in a host-oriented framework.

Contribution: reinforces that file URL conversion is a useful library/platform
facility, not evidence that it must be part of the smallest language Core Path.

## Candidate comparison

### Candidate A — retain current breadth

Rejected.

It preserves options but forces Core and every future backend to continue
accounting for root/base distinction and parent traversal despite no current
consumer/backend support. It also retains a large unimplemented file-URL
specification.

### Candidate B — minimal relative/downward Path

**Selected.**

It matches current production use and capability-confined backends, removes
unused semantic breadth, and leaves future rooted/parent/file-URL functionality
additive rather than foundationally blocked.

### Candidate C — retain rooted Paths only

Strongest rejected alternative.

Rootedness is cheap in representation and common in conventional systems.

It is rejected because current evidence does not establish that future
base-versus-root navigation belongs in Path rather than in explicit Filesystem
capability derivation/provisioning. Retaining the bit would preselect that model
without a consumer.

### Candidate D — another narrow subset

No distinct hybrid demonstrated enough present value to justify a separate
contract.

In particular, retaining the file-URL bridge while removing rooted/parent
breadth would be internally awkward because file URL mapping is itself one of
the strongest potential consumers of richer path/root semantics and has no
implemented public API today.

## Comparative scoring

Scores use 1–5; confidence H/M/L.

| Criterion | A Keep all | B Minimal | C Rooted only | D Hybrid |
| --- | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 5H | 5H | 5H | 5M |
| Protos alignment | 3H | 5H | 4H | 4M |
| Present-need proportionality | 2H | 5H | 3H | 3M |
| Incremental growth | 4M | 5H | 5H | 4M |
| Future-option resilience | 3M | 5H | 4M | 4M |
| Scalability | 4M | 5H | 5H | 4M |
| Conceptual simplicity | 2H | 5H | 4H | 4M |
| Portability / implementation freedom | 3H | 5H | 4H | 4M |
| Runtime / resource cost | 4H | 5H | 5H | 5H |
| Failure / operability | 3M | 5H | 5H | 4M |
| Deferral / reversibility / migration | 3M | 5H | 5H | 4M |
| Evidence maturity / implementation risk | 4H | 5H | 4M | 3M |

Arithmetic is not decision authority.

Candidate A carries an overengineering red flag because it standardizes
resolution complexity unsupported by current consumers.

Candidate B does **not** carry the same underengineering warning as D164 because
reintroducing rooted/parent capability is additive to the retained relative
model and no persisted/public external representation currently makes the
removal difficult to reverse.

## Incremental-design analysis

### Smallest sufficient solution

Current production requirements need:

```text
Path.relative()
Path.child(name)
structural equality/hash
Filesystem confinement
```

No current production requirement justifies a second Path anchor, retained parent
components or Core file-URL conversion.

### Pay for what you need

Removing the unused forms reduces public Core concepts, equality dimensions,
transfer cases and backend obligations.

The main savings are conceptual/backend-contract savings rather than raw runtime
CPU or memory savings.

### Grow as you need

A future backend may add rooted paths or parent traversal through a new approved
decision without changing what existing relative/downward Paths mean.

A future domain may also resolve navigation syntax into a clean relative Path,
as Package Tool already does.

### Cost of deferral

If rooted Paths are later needed, likely work includes:

- reintroducing a rootedness dimension;
- defining Filesystem base versus namespace-root behavior;
- extending equality/hash/transfer;
- implementing backend rooted resolution; and
- adding conformance coverage.

This is bounded and additive.

If parent traversal is later needed, likely work includes:

- reintroducing a Parent component kind;
- defining backend confinement/resolution in the presence of namespace
  indirection; and
- adding backend/security/conformance coverage.

This is more semantically serious than rootedness but still does not invalidate
existing normal relative Paths.

If file-URI conversion is later needed, it should be designed against the
then-current URI and Filesystem layers; no current implementation must be
migrated.

### Reversibility

No current persisted Path interchange format, network Path protocol or production
consumer was found that makes this narrowing expensive to undo.

## Future-scenario stress test

Multiple workspace/project/cache/SDK namespaces can naturally be represented as
different Filesystem capabilities plus relative/downward Paths.

If a real application later needs two anchors *within the same Filesystem
capability*:

```text
configured base
namespace root
```

that requirement provides direct evidence for reintroducing rootedness.

If real navigation later requires safe upward traversal of a mutable filesystem
namespace, Protos can design that capability with concrete backend/security
requirements rather than preserving an unused Parent component today.

## Strongest argument against Candidate B

Rooted Paths are common, cheap to represent and may plausibly return almost
unchanged. Removing them creates implementation/spec/test churn now and possible
reintroduction churn later.

D169 accepts that risk because the current Protos capability model does not yet
demonstrate that rootedness belongs in Path rather than in Filesystem authority
provisioning, and because reintroduction would be additive without invalidating
the surviving relative/downward model.

## Intentionally deferred

D169 does not decide:

- future Filesystem derivation/subtree APIs;
- host-native path parsing;
- host absolute-path interoperability;
- cwd semantics;
- future rooted Path spelling;
- future parent-navigation semantics;
- symlink-aware upward traversal APIs;
- file URI API placement;
- URI/URL normalization policy;
- Windows drive/UNC mapping;
- file URI host/authority mapping; or
- native-path interop/FFI.

## Implementation consequences

D169 requires implementation work.

```text
NORMATIVE_SPEC_CHANGE_REQUIRED=YES
CORE_PATH_API_CHANGE_REQUIRED=YES
PATH_REPRESENTATION_SIMPLIFICATION_REQUIRED=YES
BACKEND_REJECTION_BRANCH_RECONCILIATION_REQUIRED=YES
GUIDE_EXAMPLE_RECONCILIATION_REQUIRED=YES
TEST_MIGRATION_REQUIRED=YES
IMPLEMENTATION_OWNER_REQUIRED=YES
```

I062 / `guillermomolina/protos#665` owns the bounded migration.

## Ratified result

```text
D169_STATUS=RATIFIED
SELECTED_CANDIDATE=B

PATH=KEEP
PATH_RELATIVE=KEEP
PATH_CHILD=KEEP
PATH_STRUCTURAL_EQUALITY=KEEP
FILESYSTEM_EXPLICIT_AUTHORITY=KEEP
FILESYSTEM_CONFINEMENT=KEEP

PATH_ROOTED=REMOVE_NOW_RECONSIDER_LATER
PATH_ROOTED_FLAG=REMOVE_NOW_RECONSIDER_LATER
PATH_PARENT_COMPONENT=REMOVE_NOW_RECONSIDER_LATER
PATH_PARENT_COMPONENT_KIND=REMOVE_NOW_RECONSIDER_LATER
CORE_FILE_URL_PATH_BRIDGE=REMOVE_NOW_RECONSIDER_LATER

D037_STRUCTURAL_EQUALITY_PRINCIPLE=KEEP
D037_ROOTEDNESS_EQUALITY_DIMENSION=SUPERSEDED_BY_D169
D037_PARENT_COMPONENT_EQUALITY_DIMENSION=SUPERSEDED_BY_D169

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
