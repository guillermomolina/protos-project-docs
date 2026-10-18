# I113 — unified Bytecode DSL operation consolidation: 0.3.381 publication

**Date:** 2026-10-18  
**Implementation owner:** [I113/#927](https://github.com/guillermomolina/protos/issues/927)  
**Related historical research:** [PLAT064-C/#915](https://github.com/guillermomolina/protos/issues/915) and its [107-operation source-signature feasibility screen](https://github.com/guillermomolina/protos/issues/915#issuecomment-6247083595)  
**Exact published product revision:** [`136ab787a7e76c295f3cae299cefa016749ed68c`](https://github.com/guillermomolina/protos/commit/136ab787a7e76c295f3cae299cefa016749ed68c)  
**Product commit subject:** `I113: consolidate the Bytecode DSL operation surface to 144 operations (0.3.381)`  
**Source version:** `0.3.381-SNAPSHOT`. This is a source implementation version; no release, tag, Native Image run, or distributed binary is claimed.

## Revision-pinned result

The published `ProtosBytecodeRootNode.java` was independently read at the exact product revision above. Its annotations have been recounted directly from source, separately for registered operations and proxies:

| Metric | Before I113 | Published 0.3.381 | Net change |
| --- | ---: | ---: | ---: |
| Direct `@Operation` | 347 | **144** | **-203 (-58.5%)** |
| Existing `@OperationProxy` | 9 | **9** | 0 |
| Combined registrations | 356 | **153** | **-203** |

The published `CHANGELOG.md` records the Bytecode root as **10,200 lines**, versus a historical baseline of 13,050 lines in that changelog. Earlier implementation discussion used a different original-line snapshot (13,016); this record does not treat the two historical line baselines as identical. The definitive post-publication operation count is the directly verified source-annotation census, not a line-count estimate.

## Published implementation inventory

The change consolidates operations with shared dynamic operand shapes through typed `@Specialization` methods and, where needed, builder-fixed `@ConstantOperand` discriminators. The following descriptions are grounded in the published changelog and files in the referenced product revision:

- **Map and structured dispatch:** shared Map comparison phase operations and later `ProtosStructuredPhase`-driven `StructuredHasNext`, `StructuredPrepareChildCall`, `StructuredAcceptChildOutcome`, and `StructuredAdvance`; the phase values preserve HASH, EQUALITY, CHILD, comparison entry/exit, and iteration semantics.
- **Protocol admission/preparation:** `IsStructuredCall` and `PrepareStructuredCall` use `ProtosStructuredCallKind`; specialized INDEXED_EACH and TWO_PARAMETER_EACH preparations retain their concrete prepared-state objects and Environment snapshot behavior.
- **Lexical/binding:** name-keyed access (`ProtosNamedLexicalAccess`), slot mutation (`ProtosSlotMutation`), captured/materialized access, inline argument/fallback reads, frame binding and multiple local-slot creation share compatible typed operations. `ProtosCapturedLexicalAccess` and `ProtosFrameLocalBindings` own extracted Java bodies. Destination resolution remains ordered before RHS evaluation, assignment after RHS evaluation.
- **Prepared Closure, super-send and continuations:** compatible vector/variadic preparations and completion operations are consolidated without rewriting Send0/Send1/Arguments/Tail shapes. `ProtosResultKind` includes continuation classification while retaining the class-profiled PIC behavior; Java-only `IsContinuation` helpers remain callable.
- **C-prime I/O:** transport preparation, wrap/failure, await/resume and completion phases are expressed through typed shared operations, retaining release-specific suspension authority and error contracts.
- **Code factoring:** `ProtosGuardedSendSelection` and `ProtosGuardedSendExecution` hold extracted guarded send and execution logic. This is code organization, not an independently demonstrated runtime speedup.
- **Generated instruction oracles:** `ProtosInstructionNames` decodes builder-fixed discriminators back to historical protocol-level names for structural, presence/absence and ordering assertions. Source/builder oracles and C-prime emitters were updated in the same publication.
- **Unemitted registrations:** historical ArrayEach, BytesEach, EnvironmentEach, IdentityMapEach, and MapEach registries were retired when no production emission was found; Java helpers with direct consumers remained available rather than being deleted only because their DSL registration was unused.

The publication preserves the previously approved behavior and fast-path contracts for PICs, guards, assumptions, caches, lazy/compact activations, ReturnHome, D179, suspension/continuation, cancellation, C-prime I/O, NORMAL/DEBUG and guest error handling. These are implementation-scope invariants and maintainer acceptance statements, not a new semantic design approval.

## Provenance and validation

| Gate | Evidence and qualification |
| --- | --- |
| Product publication | Exact [commit `136ab787a7e76c295f3cae299cefa016749ed68c`](https://github.com/guillermomolina/protos/commit/136ab787a7e76c295f3cae299cefa016749ed68c) verified in GitHub; source, lowerers, emitters, tests, `pom.xml`, and `CHANGELOG.md` are published together. |
| Source census | Exact revision readback: 144 `@Operation`, 9 `@OperationProxy`; this source count was independently checked. |
| `git diff --check` | **CLEAN**, maintainer-reported after publication. |
| Full local suite | **PASS**, maintainer-reported (`make test`); the coordinating agent did not execute it independently and no separate test manifest was supplied. |
| Generated Bytecode BCI guard | Published changelog reports regenerated baseline and **557 proven transitions**, with **0 risk and 0 unknown**; no independent rerun by this coordinating agent. |
| Version metadata | `pom.xml` and `CHANGELOG.md` present in the exact product commit; `0.3.381-SNAPSHOT` read from the published pom. |
| Additional runtime/PE performance validation | **NOT CLAIMED**: no measured graph-size reduction, speedup, matched before/after benchmark, or independent Native Image outcome is established by this publication. |

## Remaining boundary: root versus inline

The 144-operation result is a validated and published consolidation milestone; it does **not** prove the theoretical minimum of the DSL operation surface. Root/inline twin families remain distinct because inline sites carry real `LocalRangeAccessor`/`ProtosFrameLexicalLayout` constants while root sites do not; the generated DSL builder rejects null constant operands. Fabricating empty layouts/ranges would add meaningless constants and violates the approved pay-as-you-grow constraints. Eager materialization of inline activations would likewise violate the existing implementation contract.

The prior [PLAT064-C 107-operation ledger](https://github.com/guillermomolina/protos/issues/915#issuecomment-6247083595) was an **uncompiled source-signature feasibility budget**, not evidence that 107 operations could be safely implemented; it is superseded as a *current census* by this published 144-operation implementation, but remains useful as a historical candidate coverage ledger.

Further investigation may examine true per-site metadata ownership, root/inline specialization compatibility, cached/uncached code generation, bytecode regeneration, PE, PIC/fast-path retention and the effect of any proposed abstraction on latency/allocation. Such investigation is **not** authorized to silently add sentinels, generic dispatch, extra `@OperationProxy` substitutions, or semantic changes merely to hit 100-120.

**Issue-state boundary:** I113's 0.3.381 implementation and local acceptance are published. Follow-up architecture research mentioned by the owner remains pending; keep [I113/#927](https://github.com/guillermomolina/protos/issues/927) open unless its formal scope is explicitly reconciled. [PLAT064-C/#915](https://github.com/guillermomolina/protos/issues/915) retains its historical deferred-decision status; this record does not ratify an unapproved architecture.

## Material inspected for this publication record

- [Published `ProtosBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/136ab787a7e76c295f3cae299cefa016749ed68c/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java).
- [Published implementation changelog](https://github.com/guillermomolina/protos/blob/136ab787a7e76c295f3cae299cefa016749ed68c/CHANGELOG.md) and [pom.xml](https://github.com/guillermomolina/protos/blob/136ab787a7e76c295f3cae299cefa016749ed68c/pom.xml).
- [Published product commit with changed-file inventory](https://github.com/guillermomolina/protos/commit/136ab787a7e76c295f3cae299cefa016749ed68c).
- [I113/#927](https://github.com/guillermomolina/protos/issues/927) current scope and [PLAT064-C/#915](https://github.com/guillermomolina/protos/issues/915) historical scope.
- `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/COORDINATION.md`, `AGENTS.work/REFERENCE.md` in the product repository; `AGENTS.md`, `docs/project/README.md`, and `docs/project/evidence/README.md` in the documentation repository.

No normative specification changes or independently executed performance measurements are asserted by this evidence publication.
