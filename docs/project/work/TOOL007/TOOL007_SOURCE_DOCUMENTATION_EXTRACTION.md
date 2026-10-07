# TOOL007 — Source documentation extraction

Status: **CLOSED**

Allocated: **2026-09-18**

Closed: **2026-09-18**

Live Issue: `guillermomolina/protos#556`

Consumes: D138 Candidate A′ — Source-local documentation

Decision record: `docs/project/decisions/tooling/D138_SOURCE_DOCUMENTATION_MODEL.md`

## Purpose

Implement the smallest Protos-owned source-documentation layer ratified by D138
without reviving the abandoned pre-D066 generalized TOOL003-B platform.

TOOL007 owns implementation, tests, integration and closure evidence for the
source-local documentation model. It does not own new documentation semantics.

## Fixed implementation boundary

The implementation must:

- reuse opt-in line-comment observation from `ProtosLexer`;
- reuse the authoritative `ProtosParser` Surface AST and `SourceSpan`;
- recognize module source units and explicit named `SurfaceSlotCreation`
  occurrences as the initial independent documentation owners;
- support bare/member targets and top-level/nested slot creations;
- associate `//!` and `///` deterministically and fail closed on invalid
  documentation placement;
- preserve ordinary tokenization/execution semantics;
- keep nested/local ownership source-local rather than assigning D064 durable
  identity to every occurrence;
- treat authored CommonMark as human semantic prose rather than a static
  type/specification authority; and
- keep D064/D067 as downstream Standard Library API/publication concerns.

## Explicit exclusions

TOOL007 does not initially add:

- runtime documentation objects/reflection;
- semantic Protos documentation-reference syntax;
- parameter documentation owners;
- documentation inheritance/copy through aliasing, delegation or composition;
- a global documentation graph, registry or background index;
- publication/visibility policy changes;
- website/VS Code presentation features; or
- a generalized multi-package documentation platform.

## Implementation slices

### TOOL007-A — source-local projection and association

Add the reusable Java-side source-documentation projection over the lexer and
Surface AST, with focused coverage for:

- module docs with ordinary preamble comments;
- top-level, nested and member-target slot docs;
- transparent grouping;
- same-name occurrences remaining distinct;
- parameter non-ownership;
- blank-line/ordinary-comment association breaks;
- no scope jumping;
- assignment non-ownership;
- inline marker rejection;
- late/multiple module documentation rejection.

TOOL007-A is an intermediate shared-production slice. Full integrated closure
remains owned by TOOL007.

### TOOL007-B — Standard Library extractor reconciliation

Reuse the source-local association layer where appropriate while retaining
D064 durable API identity, D067 top-level publication/coverage behavior and
exact-SHA artifact constraints.

### TOOL007-C — integrated closure

Run complete unrestricted repository validation on the exact closure candidate,
reconcile obsolete D062-era implementation assumptions/tests, record final
evidence and close TOOL007 only when no unresolved design choice remains.

## Stop condition

If implementation exposes a materially new semantic or durable architecture
choice not fixed by D138, stop the affected slice and route the exact question
through the normal Dxxx/PLATxxx gate.

## Final closure evidence

TOOL007 completed the D138 Candidate A′ implementation in bounded publication
slices and then reconciled the existing Standard Library documentation path.

Published implementation:

- TOOL007-A1 — source-local owner inventory:
  `guillermomolina/protos@9ce0ff09b05daedcb880a9d5f33ad1e760bf5ee2`
  (`0.3.30-SNAPSHOT`);
- TOOL007-A2 — module `//!` association:
  `guillermomolina/protos@bb715ba9d8cdf6caed7d3f76591ba34951c3e1b6`
  (`0.3.31-SNAPSHOT`);
- TOOL007-A3 — named slot `///` association:
  `guillermomolina/protos@3a05a807ae67c2ed6da60f648bb4713fa7016eb9`
  (`0.3.33-SNAPSHOT`);
- TOOL007-B — Standard Library extractor reconciliation:
  `guillermomolina/protos@6c2ffc8412889bb9d26d08714f2ce56307239d95`
  (`0.3.34-SNAPSHOT`).

The final state establishes `ProtosSourceDocumentation` as the source-local
D138 authority for module and named slot documentation association. The
Standard Library extractor delegates authored documentation ownership and
validation to that layer while retaining D064/D067 top-level publication and
coverage behavior.

D062-era duplicate association machinery was removed from
`ProtosStandardLibraryDocumentationExtractor`. The obsolete test assumption
that nested `///` documentation must be rejected was reconciled: nested D138
documentation is valid source documentation, while D064 publication remains
limited to the top-level Standard Library surface.

Validation evidence:

- focused source-documentation tests: PASS;
- Standard Library extractor regression tests: PASS;
- complete unrestricted repository validation via `make test`: PASS on the
  exact final TOOL007-B publication candidate;
- final published closure candidate:
  `6c2ffc8412889bb9d26d08714f2ce56307239d95`;
- earlier A1/A2 bounded-validation debt is discharged by later unrestricted
  validation on A3 and the final B/closure candidate;
- specification changed: NO;
- observable Protos runtime semantics changed: NO;
- unresolved D138 implementation/design choice: NO.

```text
TOOL007_STATUS=CLOSED
D138_AUTHORITY=RATIFIED_A_PRIME
TOOL007_A1=PUBLISHED
TOOL007_A2=PUBLISHED
TOOL007_A3=PUBLISHED_FULL_VALIDATION
TOOL007_B=PUBLISHED_FULL_VALIDATION
TOOL007_C=COMPLETE
FULL_VALIDATION=PASS
CLOSURE_REVISION=6c2ffc8412889bb9d26d08714f2ce56307239d95
SPECIFICATION_CHANGE_REQUIRED=NO
```
