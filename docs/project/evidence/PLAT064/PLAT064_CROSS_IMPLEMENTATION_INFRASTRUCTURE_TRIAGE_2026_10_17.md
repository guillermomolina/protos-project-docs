# PLAT064 — Cross-implementation Truffle Bytecode DSL infrastructure divergence triage (2026-10-17)

**Owner:** [PLAT064 / #911](https://github.com/guillermomolina/protos/issues/911)  
**Type:** investigation-only platform evidence; **not** an implementation approval, design selection, test result, or performance claim.  
**Product snapshot:** [`guillermomolina/protos@c00b36c0edaf6678b7d8101de3a405124f179644`](https://github.com/guillermomolina/protos/tree/c00b36c0edaf6678b7d8101de3a405124f179644).  
**Declared Truffle/GraalVM dependency:** 25.4.4.1.1 (product `pom.xml`).  
**Upstream code inspection:** [`oracle/graal@95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b`](https://github.com/oracle/graal/tree/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b), release/graal-vm/25.4 branch snapshot, `GenerateBytecode.java`. The branch snapshot has not been proven byte-identical to the exact 25.4.4.1.1 Maven artifact; any change relying on processor internals must verify that exact artifact first.  
**Method:** read-only current-source and prior-decision inspection; no build, test, runtime execution, benchmark, patch, or product publication performed.

## Objective and authority

This is a **multi-implementation, bounded host-framework integration audit**. Its purpose is to find genuine candidates for *further investigation*, not to take decisions now. Divergence is grounds to inspect, never automatic authority to imitate a peer or modify the product. A framework-alignment correction does **not** require measured performance gain; correctness and compatibility gates would still apply at implementation time.

Binding constraints: product `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `src/AGENTS.md`, `spec/AGENTS.md`, directly governing normative specification, and ratified platform decisions. In particular:

- PLAT036 [#702](https://github.com/guillermomolina/protos/issues/702): frame-backed **single authority**, stable first-class execution contexts, captured lexical references and dynamic overflow.
- D179 [#703](https://github.com/guillermomolina/protos/issues/703), I071 [#717](https://github.com/guillermomolina/protos/issues/717): **C0 was restored**; legal `PRESENT -> ABSENT` and subsequent creation are supported on open contexts. The earlier monotonic-C3 model is superseded.
- PLAT042 [#760](https://github.com/guillermomolina/protos/issues/760) and PLAT043 [#763](https://github.com/guillermomolina/protos/issues/763): tagged semantic root versus untagged auxiliary root, with approved standard Boolean ownership exception. **Do not merge roots by inference**.
- PLAT062 [#907](https://github.com/guillermomolina/protos/issues/907) owns Closure/call lowering; PLAT063 [#908](https://github.com/guillermomolina/protos/issues/908) owns emitted semantic-operation granularity.
- AUD016 [#701](https://github.com/guillermomolina/protos/issues/701) is a completed earlier capability audit: consume its resolved findings, do not re-run or overwrite historical decisions.
- I110 [#912](https://github.com/guillermomolina/protos/issues/912) is already independently **owner-authorized but not yet implemented**: include `boolean.class` in boxing elimination in both roots, without a speedup gate. Not a PLAT064 dependency or newly requested design decision.
- UPSTREAM004/BUG013 and Native guest JIT are **not** PLAT064's goal or acceptance gate.

Normative counterexamples include captured-context removal/recreation, `PRESENT(null) != ABSENT`, lexical-before-receiver lookup, assignment destination fixed before RHS, parameter activation/default ordering, return-home/Task validity, Error handler consumption *before crossed ensure cleanup*, and **suspension is not scope exit**.

## Comparator corpus

A peer vote is useful only when the project uses **generated Truffle Bytecode DSL** and the facility exists in that version. Check differences in maturity and semantic requirements before deciding whether the precedent transfers.

| Peer | Reference root (pinned commit) | Declared Truffle/GraalVM |
| --- | --- | --- |
| GraalPy | [`PBytecodeDSLRootNode`](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java) | Peer revision pinned; Maven release not established |
| Oracle SimpleLanguage | [`SLBytecodeRootNode`](https://github.com/graalvm/simplelanguage/blob/5a2b35790539ab1fe5c61f02b71e3485fd0e0fb1/language/src/main/java/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java) | 24.2.0 |
| TrufflePascal | [`PascalBytecodeRoot`](https://github.com/d45h71/TrufflePascal/blob/fcae9007500369dd194c7e09c932fd4807d64fa8/language/src/main/java/org/trufflepascal/bytecode/PascalBytecodeRoot.java) | **25.4.4.1.1** |
| Bridje | [`BridjeRootNode`](https://github.com/bridje/bridje/blob/189de1af103b5b7eb8cabd1bcbd64a137f493c1e/language/src/main/kotlin/brj/nodes/BridjeRootNode.kt) | 25.3.4.1 |
| SWALox | [`LoxBytecodeRootNode`](https://github.com/hpi-swa-lab/swalox/blob/38aa0428bc2471254da6d8ae594265c041a593bd/src/main/java/de/hpi/swa/lox/bytecode/LoxBytecodeRootNode.java) | 24.2.0 |
| Cadenza | [`BytecodeRoot`](https://github.com/ekmett/cadenza/blob/e2b66e241527cde5d29014af1e4f83d9f2402f88/src/cadenza/bytecode/BytecodeRoot.java) | 25.3.4.1 |
| Hara HBC | [`HbcBytecodeRootNode`](https://github.com/hara-lang/hara-native/blob/9e52e5377740fd67a01d148cdb6c756336f7e745/core/java/src/main/java/hara/truffle/bytecode/HbcBytecodeRootNode.java) | 25.0.3 |

[Apple Pkl](https://github.com/apple/pkl/blob/ef5e34c21009fe3d200b647551dbd672d288e710/pkl-core/src/main/java/org/pkl/core/ast/PklRootNode.java), GraalJS and Sulong are architectural controls but **not votes on `@GenerateBytecode` settings**: their examined roots are AST/other interpreter mechanisms. Their frame, call, source and diagnostic patterns may still be informative when applicable.

The previously collected majority observations are descriptive, **not a scoring system**: 6/7 external peers enable at least one boxing-elimination carrier, 4/6 peers with boxing elimination include primitive Boolean, 5/7 enable block scoping, 1/7 enables materialized local accesses, and 1/7 stores BCI in frame. These figures concern the roots enumerated above, not whole-runtime coverage or a representative statistical sample of all Truffle languages.

## Verified product source and generator facts

- [Semantic generated root](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosSemanticBytecodeRootNode.java): yield, tags, true RootTag, false RootBodyTag, materialized locals, tail handlers, uncached interpreter, int/long/double boxing; 9 numeric/Boolean `@OperationProxy` annotations. Compact source-call entry and lazy `CurrentActivation`; source-root Error interceptor records first diagnostic origin and delegates handler selection.
- [Auxiliary generated root](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java): same capabilities except automatic RootTag false, RootBodyTag false; structured/Task/I/O C-prime, shared Error selection before crossed cleanup; comments acknowledge legacy source-only operation declarations may remain.
- Neither root explicitly sets `storeBytecodeIndexInFrame` (upstream default false), `enableBlockScoping` (default true), `enableSerialization` (default false), `enableQuickening` (default true), or uncached threshold (default 16).
- [`GenerateBytecode.java`](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/GenerateBytecode.java) documents that a materialized local **must be in scope**. `storeBytecodeIndexInFrame=true` enables dynamic checks for access outside scope, but incurs frame writes/size; `@Bind("$bytecodeIndex")` already supplies fast-path BCI inside an operation.
- [`CanonicalToBytecodeLowerer.emitRootBody`](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java) allocates every statically declared guest binding immediately under `beginRoot()`, ensuring root scope. Persistent frame authority is installed only when required. Captured reads can emit the builtin `LoadLocalMaterialized` after selecting a present owner.
- [`ProtosFrameLexicalBindingAuthority`](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosFrameLexicalBindingAuthority.java) keeps declaring root and retained materialized frame; `declaringLocation.update()` and the current BytecodeNode avoid stale tier-local metadata.
- [`ProtosDiagnosticTraceCapture`](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosDiagnosticTraceCapture.java) records the innermost Error position; the uncached path records it early because finally/ensure unwind can overwrite the frame's position. Continuation and inline callback frames are projected as semantic, not physical helper, frames.
- Tests inspected: `ProtosI075DLexicalAuthorityCurrentBytecodeNodeTest`, `ProtosBug013FrameLexicalAuthorityMaterializationTest`, `ProtosPerf006B5ABytecodeDebuggerScopeTest`, `ProtosPerf006B5BSourceInstrumentationLocationTest`. These supply **existing source evidence**, not freshly executed results.

## Finite 18-facility triage matrix

`YES` means a *falsifiable follow-up research question*, **not** demonstrated bug, performance regression, owner approval, or permission to implement.

| # | Facility | Observed divergence/current mechanism | Triage | Preliminary classification / next action |
| --- | --- | --- | --- | --- |
| 1 | BCI in frame | Protos default false, GraalPy true; captured materialized access exists | **YES → A** | `INCONCLUSIVE / DESIGN_GATE only if proof reveals new architectural decision`. Prove all materialized accesses' valid owner/scope/lifetime before considering global BCI storage |
| 2 | Block scoping | Protos true; GraalPy/Bridje false; semantic bindings are root-scoped while implementation temporaries use blocks | **YES → A** | `REQUIRED_SEMANTIC_CHOICE / KEEP pending exact scope audit`; distinguish physical scope, dynamic semantic presence and lifetime |
| 3 | Materialized locals | Protos true, uncommon in peers; required for capture, but helper generator has related operations | **YES → A,C** | `REQUIRED_SEMANTIC_CHOICE` for semantic root; helper-side need `INCONCLUSIVE`; do not weaken PLAT036 |
| 4 | Root lifecycle hooks | GraalPy and TrufflePascal use @Prolog/@Epilog*; Protos uses lazy entry plus interceptors and operations | **YES → B** | `INCONCLUSIVE / DESIGN_GATE if structural change`; compare exact obligations and order, preserve error/unwind/suspension contracts |
| 5 | Two generated roots | Protos has two classes under ratified PLAT042; peer example set uses single principal generated class | **YES → C** | `REQUIRED_SEMANTIC_CHOICE / KEEP topology`; audit duplication/cost, not root merge |
| 6 | Duplicate declarations | Textual count: 235 helper, 175 semantic, 76 same operation names; does not imply 76 duplicated algorithms | **YES → C** | `INCONCLUSIVE / PROPOSE_BOUNDED_Ixxx if eligible`; classify duplication of DSL metadata vs implementation and cache |
| 7 | Possibly un-emitted helper operations | 178 of 235 helper operation names referenced in inspected production emitters; **57 not found there**, but some do have test consumers | **YES → C** | `INCONCLUSIVE / PROPOSE_BOUNDED_Ixxx only if no consumer proven`; expand complete source and test consumer graph |
| 8 | Helper tag instrumentation | Helper has tags enabled, automatic root/body disabled, no obvious tags in examined production emitters | **YES → D** | `INCONCLUSIVE / PROPOSE_BOUNDED_Ixxx if no consumer`; verify runtime tag tree and tooling, not annotation text only |
| 9 | Root/RootBody tags | Semantic root has automatic RootTag and no RootBodyTag; approved debugger/source contracts | **YES → D** | `REQUIRED_SEMANTIC_CHOICE / KEEP unless actual tooling contract gap`; assess together with 8 |
| 10 | Boxing elimination | Protos int/long/double; internal Java primitive Boolean missing vs numerous peers | **EXISTING OWNER** | `ADOPT_EXISTING_WORK_OWNER: I110/#912` — owner authorized, **not implemented**, no benchmark gate |
| 11 | ForceQuickening | Automatic quickening already enabled; no standalone failing site demonstrated | **NO** | `CONFIG_ONLY / KEEP` (AUD016 no global @ForceQuickening policy); concrete future counterexample belongs to existing specialized owner |
| 12 | OperationProxy | Nine in semantic root, none in helper; proxies may offer shared Node declarations | **YES → C** | `INCONCLUSIVE / PROPOSE_BOUNDED_Ixxx only for exact eligible operations`; preserve cache/specialization and uncached contracts |
| 13 | TypeSystemReference | Used in selected peers, not Protos; explicit numeric domains and conversions already exist | **NO** | `CONFIG_ONLY / DEFER`; no concrete redundant conversion/admission gap demonstrated |
| 14 | ShortCircuitOperation | Some peers use it, Protos uses ordinary Boolean messages and PLAT043 post-selection dispatch | **NO** | `REQUIRED_SEMANTIC_CHOICE / KEEP` to avoid changing overridable guest control semantics |
| 15 | Serialization | Some peers enable it; no current serialized-bytecode consumer demonstrated | **NO** | `CONFIG_ONLY / DEFER` per AUD016; add only for actual persistence/cache requirement |
| 16 | Uncached transitions | Protos uses uncached interpreter, default threshold and specialized handling on tier transition | **YES → A** | `INCONCLUSIVE / KEEP flag pending lifecycle proof`; do not invent threshold adjustment |
| 17 | Yield/tail handlers | Yield supports suspensions; tail handler outlining is host compilation machinery | **NO (standalone)** | `REQUIRED_SEMANTIC_CHOICE / KEEP` with correctness cases in A/B; no proposal to disable |
| 18 | Source/trace integration | Custom first-position capture, early uncached BCI save, logical inline/continuation frame projection | **YES → E** | `INCONCLUSIVE / DESIGN_GATE only if needed`; verify direct DSL/API equivalents on exceptional paths |

The 12 YES entries are #1–#9, #12, #16, #18. They collapse into **five** research scopes, not 12 automatic issues, and no issue authorizes implementation.

## Independently closable research children

The `ISSUE-SLICE-BOUNDARY` rule permits dedicated child Issues because the scopes have independent evidence and closure criteria; they can progress in parallel or be blocked independently. Each must have GitHub **native** parent #911, not only a textual Parent line. If GitHub intake reconciliation has not yet produced that relation, it is **PENDING**, not a successful hierarchy publication.

| Child | Parent scope | Independent completion question |
| --- | --- | --- |
| [PLAT064-A / #913](https://github.com/guillermomolina/protos/issues/913) | 1, 2, 3, 16 | Are all materialized/frame-local accesses valid through scope, cached/uncached, yield, capture and escape? |
| [PLAT064-B / #914](https://github.com/guillermomolina/protos/issues/914) | 4 | Which, if any, root lifecycle responsibilities exactly match standard DSL hooks? |
| [PLAT064-C / #915](https://github.com/guillermomolina/protos/issues/915) | 5, 6, 7, 12 | Which declarations/operations are required or shared, and which have truly no current consumers? |
| [PLAT064-D / #916](https://github.com/guillermomolina/protos/issues/916) | 8, 9 | Does helper tag infrastructure have actual debug/instrumentation consumers? |
| [PLAT064-E / #917](https://github.com/guillermomolina/protos/issues/917) | 18 | Could supported Truffle APIs replace any custom source/trace responsibilities exactly? |

Investigation completion is **not** design ratification or implementation. If a finding is a routine internal framework-alignment change, propose an independently bounded `Ixxx` after owner direction. If it changes a ratified architecture or observable semantics, prepare the full GITHUB010 12-axis candidate comparison, explicit counterexamples and GITHUB021 invariant audit, then stop for the owner's decision. An investigation can conclude **KEEP**.

## Potential adversarial cases and decision boundaries

1. **Captures:** source root and retained frame survive normal root exit; nearer execution-context binding is removed/recreated; `PRESENT(null)` must not be treated as absent.
2. **Tier transitions:** authority installation in UNCACHED, `Yield`, continuation resume in CACHED; no stale BytecodeNode/local-kind metadata; a physical temporary's scope is not confused with root-scoped bindings.
3. **Root lifecycle:** supplied vector and default evaluation order; no mandatory eager rich activation; Error handler selected/deactivated before `ensure` cleanup; cleanup may suspend without replay; non-local returns obey home and Task boundaries.
4. **Instrumentation:** RootTag belongs to semantic source, not physical C-prime helper; inline callbacks report correct logical source frames and tooling scopes. A test that only asserts a flag is not evidence that a real tag is emitted.
5. **Code reuse:** reducing two generator annotation surfaces must not introduce a third runtime, bypass the native Builder, affect operation specialization, or duplicate PLAT063's semantic lowering study.
6. **Diagnostics:** preserve the *original* failing position, including exceptional uncached cleanup, continuation and inlined callbacks. Enabling `captureFramesForTrace` without a consumer would charge ordinary execution for an optional path.

## Full audit evidence inventory / limits

Materially inspected: `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, `src/AGENTS.md`, `spec/AGENTS.md`; `spec/PROTOS_LANGUAGE_SPEC.md`, `spec/semantics/CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `ERRORS.md`, and selected provisions of `OBJECT_MODEL.md`, `VALUES_AND_COLLECTIONS.md`, `spec/concurrency/FUTURES_AND_TASKS.md` and `ACTORS.md`; generated root annotations and operation declarations, canonical and structured lowerers, lexical authority, diagnostics, tag exports, numeric proxy, selected C-prime emitters, focused tests, and earlier formal issues/AUD016. Huge normative files were inspected by relevant headings/ranges; **not every line of all 4,000+ line normative owners was read**. A child that proposes any substantive change must finish whole-document normative coverage per `DESIGN.md` before its decision packet.

Counts above are **textual current-source observations** and selected-producer searches. They do not prove dead code, generated code size, runtime impact, a failing test, or a Truffle bug. Other peers are evidence of possible idioms, not conformance authority. Reinspect actual product HEAD before each child study, preserve concurrent changes, and publish per-child results under this canonical evidence owner.

**PLAT064 remains OPEN** pending documented classification of all bounded families and GitHub structural reconciliation. I110 stays independently actionable and its progress is not a parent completion requirement.
