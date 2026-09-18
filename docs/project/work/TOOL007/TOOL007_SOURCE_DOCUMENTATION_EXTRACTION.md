# TOOL007 — Source documentation extraction

Status: **IN PROGRESS**

Allocated: **2026-09-18**

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

## Current evidence

TOOL007 was allocated after D138 was durably ratified. The initial TOOL007-A
candidate has been authored against current parser/lexer APIs and syntax-compiled
against API-faithful stubs; repository Maven validation remains the publication
launcher's responsibility before any `guillermomolina/protos` commit/push.

```text
TOOL007_STATUS=IN_PROGRESS
D138_AUTHORITY=RATIFIED_A_PRIME
TOOL007_A=AUTHORED_PENDING_PUBLICATION
TOOL007_B=BLOCKED_BY_A
TOOL007_C=BLOCKED_BY_A_B
SPECIFICATION_CHANGE_REQUIRED=NO
```
