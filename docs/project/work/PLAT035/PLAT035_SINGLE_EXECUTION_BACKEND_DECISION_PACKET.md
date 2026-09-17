# PLAT035 — Single execution backend decision packet

Status: **OPEN — NEEDS PROJECT-OWNER DECISION**

Nature: durable non-normative decision research and ratification input

Owner: `PLAT035` / `guillermomolina/protos#551`

Triggered by: `AUD012` / `guillermomolina/protos#541`

Primary prior authority: `PLAT014` — C′ stackful continuation composition over
Truffle Bytecode DSL continuations

Source repository: `guillermomolina/protos`

Research source revision: `5529fa515016f6a44fb15934c4691e162c417d73`

Research date: 2026-09-17

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

Specification changed: **NO**

Implementation changed: **NO**

Candidate selected by this record: **NO**

## 1. Decision in plain terms

Protos currently has one frontend pipeline and two executable backend paths after
the Canonical AST:

```text
source
  -> Surface AST
  -> Canonical AST
       |-> CanonicalToTruffleLowerer -> ProtosExpressionNode tree -> execution
       `-> CanonicalToBytecodeLowerer -> Truffle Bytecode DSL -> execution
```

The decision is not whether AST interpreters are intrinsically bad. They are not.
The decision is whether Protos should continue maintaining two independent
executable realizations of the same canonical semantics after Bytecode DSL has
already become the selected production continuation substrate.

The project-owner invariant recorded when PLAT035 was opened is:

```text
TARGET_EXECUTION_BACKEND = TRUFFLE_BYTECODE_DSL_ONLY
CanonicalToTruffleLowerer = LEGACY_TRANSITIONAL
ProtosExpressionNode execution tree = LEGACY_TRANSITIONAL
NEW_DEPENDENCIES_ON_LEGACY_BACKEND = FORBIDDEN
END_STATE = LEGACY_EXECUTION_BACKEND_REMOVED
```

This packet tests the proposed retirement architecture against current Protos,
prior ratified decisions, external precedent, migration risk and the full
`AGENTS.md` decision criteria. It does not treat that owner invariant as
permission for an unsafe immediate deletion.

## 2. Current Protos evidence

At source revision `5529fa515016f6a44fb15934c4691e162c417d73`,
`ProtosSourceCompiler` still contains both lowerers.

The direct `compile(...)` family parses to Surface AST, canonicalizes to
`CanonicalSequence`, and lowers with `CanonicalToTruffleLowerer` through
`ProtosRootFactory.createCallTarget(...)`.

The `compileBytecode(Source, ProtosLanguage)` path parses the same source,
canonicalizes to the same `CanonicalSequence`, and lowers with
`CanonicalToBytecodeLowerer` into `ProtosBytecodeRootNode`, wrapped by
`ProtosSemanticBytecodeRootNode`.

`ProtosLanguage.parse(...)`, the registered Truffle public language entry,
already calls `sourceCompiler.compileBytecode(...)`. Therefore the production
language boundary already treats Bytecode DSL as the ordinary hosted source
backend.

AUD012's inventory demonstrates why deletion cannot be mechanical. Direct Java
guest entry is used by a mixture of:

- ordinary semantic harnesses;
- Java/runtime/bootstrap tests that intentionally manipulate host-side state;
- compiler/lowering/backend component tests;
- compile-only assertions;
- deliberately unhosted staging paths;
- historical helpers whose ownership still needs TEST002 review.

That heterogeneity is a migration problem. It is not evidence that two permanent
execution backends are required.

## 3. Already-ratified platform constraints

### PLAT014

PLAT014 is already ratified as C′: stackful Protos continuation composition over
Truffle Bytecode DSL continuations. Its durable record explicitly rejected a
permanent AST state-machine continuation framework as avoidable duplicate
infrastructure and rejected a temporary AST continuation hybrid after direct
Bytecode DSL feasibility was proven.

PLAT014 also deliberately deferred the exact migration slices from the old
AST/replay backend to Bytecode DSL. PLAT035 resolves one of those deferred
architecture questions: whether the old executable AST remains a permanent
parallel backend after the migration.

### Tooling decisions

PLAT005, PLAT013, PLAT015, PLAT026 and PLAT034 already define the source,
instrumentation, debugger, root/tag and generic-tool compatibility contracts that
Bytecode execution must preserve. Backend retirement may not redefine those
contracts.

### TEST002

TEST002 owns whether observable semantic tests should move from Java/JUnit into
ordinary Protos/TOOL002 tests. PLAT035 owns backend authority, not test-language
ownership. A test may remain Java-owned while its guest execution moves to the
single Bytecode backend.

## 4. External comparative evidence

### 4.1 GraalPy — closest Truffle migration precedent

GraalPy provides the strongest directly comparable evidence because it operated
with the exact kind of transition PLAT035 is considering.

Its 22.2 changelog introduced an experimental bytecode interpreter and allowed
switching between the previous AST interpreter and the new bytecode interpreter.
In 22.3 the project switched to the new bytecode interpreter as the default
backend, citing improved startup performance and memory footprint while retaining
good JIT-compiled performance.

This is important for PLAT035 because the coexistence phase was a migration and
validation mechanism, not evidence that the project needed two permanent
semantic execution authorities.

Primary sources:

- https://github.com/oracle/graalpython/blob/master/CHANGELOG.md
- https://www.graalvm.org/release-notes/22_3/

### 4.2 Truffle Bytecode DSL — explicit successor substrate, not a second semantic universe

Current Truffle documentation describes Bytecode DSL as a generator for complete
optimizing bytecode interpreters from language-specific operations. The stated
motivation is to preserve AST-quality peak performance while reducing program
representation footprint and improving interpreted performance. It supports
quickening, tiered interpretation, continuations, instrumentation, lazy
source/tooling metadata and serialization.

The documentation also explicitly says Bytecode DSL is still experimental, so
PLAT035 must preserve a stable internal Protos backend boundary and must not let
Bytecode DSL Java types become Protos semantics.

Primary sources:

- https://github.com/oracle/graal/blob/master/truffle/docs/bytecode_dsl/BytecodeDSL.md
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/GenerateBytecode.html

### 4.3 GraalJS / TruffleRuby / Apple Pkl — useful AST counterexamples

Mature Truffle languages demonstrate that an AST interpreter can be a valid,
high-performance canonical backend. GraalJS compilation diagnostics still speak
in AST terms, and TruffleRuby/Pkl are relevant examples of languages that have
successfully invested in AST-based execution.

Their relevance is primarily negative/corrective: PLAT035 must not argue
"AST bad, bytecode good". An AST is not a defect. What requires justification is
maintaining two full post-canonical executable realizations when Protos has
already selected Bytecode DSL for the architecture that ordinary production
execution needs.

Primary sources:

- https://github.com/oracle/graaljs
- https://github.com/truffleruby/truffleruby
- https://github.com/apple/pkl
- https://www.graalvm.org/dev/graalvm-as-a-platform/language-implementation-framework/Optimizing/

### 4.4 Espresso — one bytecode-oriented execution authority can still use specialized nodes

Espresso is useful because "single backend" does not mean "no Truffle nodes".
A bytecode interpreter can use specialized Truffle nodes internally for selected
operations while preserving one authoritative execution pipeline.

This matters for Protos: removal of the legacy `ProtosExpressionNode` execution
tree does not prohibit Bytecode DSL operations or helper nodes from using normal
Truffle specialization mechanisms. The architecture question is authority and
pipeline duplication, not whether a Java `Node` class exists anywhere.

Primary source:

- GraalVM Espresso implementation/documentation and Truffle Bytecode DSL APIs.

### 4.5 SimpleLanguage — reference Bytecode DSL implementation

Truffle's own Bytecode DSL documentation points to the Bytecode DSL version of
SimpleLanguage as a useful reference implementation. This provides a minimal
language-level precedent for building the executable interpreter directly around
the generated Bytecode DSL substrate without needing a parallel executable AST
for ordinary execution.

Primary source:

- https://github.com/oracle/graal/tree/master/truffle/src/com.oracle.truffle.sl

### 4.6 V8 — strong non-Truffle backend-retirement precedent

V8 is not a Truffle implementation, but it is highly relevant to the maintenance
question. During migration, old and new pipelines coexisted. In V8 5.9,
Ignition + TurboFan became the universal and exclusive JavaScript execution
pipeline and Full-codegen/Crankshaft stopped being used for JavaScript execution.
V8 explicitly cited a simpler, more maintainable architecture as a result.

V8 subsequently removed an AST-numbering pass whose main reason for existing had
been coordination between multiple compilers; Ignition bytecode became the common
reference point. This is direct evidence for retiring coordination machinery once
one execution representation becomes authoritative.

Primary sources:

- https://v8.dev/blog/launching-ignition-and-turbofan
- https://v8.dev/blog/v8-release-66
- https://v8.dev/blog/background-compilation

### 4.7 LLVM-style frontend/backend separation — keep useful trees, unify execution lowering

Compiler pipelines routinely preserve rich syntax/semantic ASTs while lowering
them into one backend IR used for code generation. This is the closest conceptual
analogy to the desired Protos end state:

```text
Surface AST -> Canonical AST -> one executable backend representation
```

PLAT035 therefore does not imply removing Surface or Canonical AST. Those remain
frontend/semantic representations unless separately audited. The proposed
retirement target is specifically the second executable backend created after
canonicalization.

## 5. Candidate set

### Candidate A — permanent dual executable backends

Keep `CanonicalToTruffleLowerer`/`ProtosExpressionNode` and Bytecode DSL as
permanent supported execution architectures.

This maximizes short-term compatibility but permanently duplicates semantic
lowering, source/tooling adaptation, test obligations and future language/runtime
changes.

### Candidate B — Bytecode production, permanent legacy Java/testing backend

Make Bytecode DSL authoritative for production but preserve the old AST as a
permanent Java/bootstrap/testing escape hatch.

This reduces production ambiguity but still requires long-term semantic parity.
A Java-owned test can silently become a reason that the entire old backend must
continue compiling and behaving correctly forever.

### Candidate C — one target backend, bounded legacy migration scaffolding

Make Bytecode DSL the sole target executable backend. Existing old-AST uses are
allowed only as quantified migration blockers while equivalent Bytecode-backed
boundaries are introduced. The old executable backend is deleted after the
inventory reaches zero.

Java tests may remain Java. Surface/Canonical ASTs remain. Compiler/frontend
tests remain where they still have a real subject. Only the duplicate executable
backend loses permanent architectural status.

### Candidate D — immediate hard deletion

Delete the legacy executable AST immediately and repair fallout afterward.

This reaches the desired topology fastest but ignores the demonstrated AUD012
heterogeneity and risks losing legitimate Java/runtime/bootstrap evidence before
an equivalent Bytecode-backed harness exists.

## 6. Twelve-axis comparison

Scores use the mandatory `AGENTS.md` 1–5 scale. Confidence refers to the quality
of evidence for the score, not confidence that a candidate is selected.

| Axis | A dual permanent | B test-only permanent AST | C bounded migration to Bytecode only | D immediate deletion |
|---|---:|---:|---:|---:|
| Correctness / invariant preservation | 3 HIGH | 3 HIGH | 5 HIGH | 2 HIGH |
| Protos alignment | 2 HIGH | 2 HIGH | 5 HIGH | 4 MEDIUM |
| Present-need proportionality | 1 HIGH | 2 HIGH | 5 HIGH | 2 HIGH |
| Incremental growth | 2 HIGH | 3 HIGH | 5 HIGH | 2 HIGH |
| Future-option resilience | 2 MEDIUM | 2 MEDIUM | 5 HIGH | 3 MEDIUM |
| Scalability | 2 HIGH | 3 HIGH | 5 HIGH | 4 MEDIUM |
| Conceptual simplicity | 1 HIGH | 2 HIGH | 5 HIGH | 4 HIGH |
| Portability / implementation freedom | 3 MEDIUM | 3 MEDIUM | 4 MEDIUM | 4 MEDIUM |
| Runtime / resource cost | 2 HIGH | 3 HIGH | 5 HIGH | 5 MEDIUM |
| Failure / operability | 2 HIGH | 3 HIGH | 5 HIGH | 2 HIGH |
| Deferral / reversibility / migration | 2 HIGH | 2 HIGH | 4 HIGH | 1 HIGH |
| Evidence maturity / implementation risk | 3 HIGH | 3 HIGH | 5 HIGH | 2 HIGH |

### Score rationale

**A — permanent dual backend.** Correctness is not zero because differential
coverage can keep two implementations conformant, but every new semantic/runtime
feature creates two places to drift. Present-need proportionality and conceptual
simplicity are poor because no current Protos requirement demonstrated by
AUD012 needs two permanent executable semantics. Portability is not as poor as
other axes because both implementations are still Truffle-hosted, but that does
not offset the maintenance institution.

**B — permanent test-only AST.** This is less costly than A operationally, but it
still makes test infrastructure an architectural authority over production
backend maintenance. It creates a durable special case: functionality is
"canonical" in Bytecode DSL while the old backend must remain semantically alive
for selected Java tests. TEST002 and AUD012 already provide better ways to express
test ownership without granting the legacy backend permanent status.

**C — bounded migration.** This preserves correctness because migration is
occurrence-driven, not deletion-driven. It is proportional because Protos pays
for one backend in the end and pays temporary dual-backend cost only where an
identified blocker remains. It fits PLAT014, gives one tooling/optimizer/backend
authority, and matches both GraalPy's actual migration history and V8's
backend-retirement precedent. Portability is scored 4 rather than 5 because
Bytecode DSL is Truffle-specific and experimental; that is acceptable at a PLAT
boundary only while semantic/frontend authority remains independent and
Truffle-specific types stay encapsulated.

**D — immediate deletion.** The end state is simple, but the transition is
underengineered. AUD012 has already proven that direct old-backend users include
bootstrap/runtime/component tests with different needs. Deleting before providing
replacement boundaries would make test loss and accidental semantic change hard
to distinguish from legitimate cleanup.

## 7. Anti-overengineering / underengineering gate

### Candidate A red flag — overengineering

The project would permanently implement the same canonical semantic execution
contract twice without a demonstrated current requirement for two independent
backends. Future-backend optionality does not require keeping a second current
backend alive.

### Candidate B red flag — institutionalized exception

"Tests need it" becomes a permanent architecture institution. The cheaper
alternative is to provide a Bytecode-backed Java execution boundary for tests
that genuinely must remain Java-owned.

### Candidate C — smallest sufficient safe architecture

Candidate C keeps exactly one intended backend and exactly as much transitional
compatibility as the audited migration actually needs. It does not pre-build a
future backend. It preserves the future option to replace Bytecode DSL because
Surface/Canonical semantics remain backend-independent and Truffle-specific
machinery remains behind PLAT boundaries.

### Candidate D red flag — underengineering

Immediate deletion assumes away known migration work. It creates a high risk of
turning missing harness capability into deleted evidence instead of solving the
capability cleanly.

## 8. Candidate C exact retirement contract — proposal

If the project owner ratifies Candidate C, the exact durable contract should be:

1. **Canonical AST is the last backend-independent executable semantic
   representation.** Surface and Canonical ASTs are not retirement targets of
   PLAT035.

2. **All source-backed executable Protos paths SHALL ultimately lower through
   the Bytecode DSL backend.** Production, bootstrap, test and deliberately
   unhosted execution may have different hosting APIs, but not different
   semantic execution backends.

3. **The legacy executable AST has migration status only.** Existing dependencies
   may remain temporarily only when recorded as concrete migration blockers with
   an owner and replacement route. No new dependency may be added.

4. **Java ownership does not imply legacy-backend ownership.** A Java/JUnit test
   that must manipulate host/runtime objects may remain Java-owned while executing
   guest source through an explicit Bytecode-backed Java boundary.

5. **`ProtosSourceCompiler.compile(...)` cannot remain a permanent second-backend
   API.** During migration it may be adapted or split as mechanically necessary;
   at the end it either routes to the canonical Bytecode backend under an
   appropriate language/context boundary or is removed/replaced. The exact API
   spelling is implementation work, not part of this platform decision.

6. **Compiler/frontend component tests remain only where their subject remains.**
   Parser, canonicalizer and Bytecode-lowering tests remain valid. Tests whose
   sole subject is behavior of the obsolete executable AST backend are removed
   after equivalent semantic/backend evidence exists where required.

7. **Fail closed against regression.** CI/repository guidance must make addition
   of new raw `CanonicalToTruffleLowerer` / executable `ProtosExpressionNode`
   dependencies fail unless the dependency is part of the explicitly bounded
   retirement implementation itself. This guard is about backend dependency, not
   a global ban on Java tests.

8. **Tooling contracts remain semantic, not backend-duplicated.** Source sections,
   tags, debugger scopes/values and instrumentation behavior are proved on the
   single Bytecode execution path under the existing PLAT authorities.

9. **Bytecode DSL remains implementation machinery.** Experimental Truffle API
   evolution may require backend maintenance, but it does not create Protos
   syntax, objects, identity or semantic concepts.

10. **Final deletion is evidence-gated.** `CanonicalToTruffleLowerer`, the
    executable `ProtosExpressionNode` backend and helpers/tests owned solely by
    that backend may be physically removed only after repository-wide rescan,
    replacement validation and authoritative full tests prove no required path
    still depends on them.

## 9. Expected migration shape after ratification

This packet does not authorize implementation, but Candidate C naturally yields
an ordered migration:

```text
C1  establish Bytecode-backed Java/unhosted execution boundary
C2  install fail-closed no-new-legacy-dependency guard
C3  migrate Java-owned runtime/bootstrap harnesses
C4  route ordinary semantic Java tests through TEST002/canonical test execution
C5  migrate remaining main-code fallback/bootstrap execution
C6  retire tests/components whose only subject is the obsolete backend
C7  delete CanonicalToTruffleLowerer + executable ProtosExpressionNode backend
C8  repository-wide zero-dependency rescan + full validation
```

These labels are descriptive only. Formal implementation issue/slice structure is
not selected by this document.

## 10. Invariant/delta consistency check

### Owner-approved PLAT035 invariant

`TARGET_EXECUTION_BACKEND = TRUFFLE_BYTECODE_DSL_ONLY`

Candidate C: **PRESERVED**.

### PLAT014 C′ continuation authority

Candidate C: **PRESERVED**. It strengthens the already-selected Bytecode
continuation architecture by removing the possibility that the legacy AST becomes
a permanent parallel continuation/execution authority.

### No Protos-visible semantic change

Candidate C: **PRESERVED BY CONTRACT**. Any discovered migration requirement that
would alter observable Protos semantics must leave PLAT035 and cross the normal
Dxxx gate.

### Existing Java-owned runtime/bootstrap evidence

Candidate C: **PRESERVED**. The contract requires replacement execution boundaries
before deletion and explicitly separates Java ownership from backend ownership.

### TEST002 ownership

Candidate C: **PRESERVED**. PLAT035 does not decide which semantic tests migrate
to Protos; it only forbids test placement from becoming permanent authority for
a duplicate backend.

### Tooling/source/debugger authority

Candidate C: **PRESERVED**. Existing PLAT005/013/015/026/034 contracts remain
required and are validated on the Bytecode backend rather than duplicated.

### New consequences introduced by Candidate C

The material new consequence is intentional and already surfaced in the active
PLAT035 owner invariant: the old executable AST loses permanent architectural
status, and any future proposal to keep or introduce a second executable backend
would require explicitly reopening that invariant through the normal approval
gate.

No hidden public semantic consequence was identified in this packet.

## 11. Intentionally deferred choices

Candidate C does **not** pre-select:

- exact Java helper/API names for unhosted Bytecode-backed execution;
- whether `ProtosSourceCompiler` is renamed, split or ultimately removed;
- exact migration slice identifiers;
- exact ordering among independent test families once the first safe boundary
  exists;
- TEST002's per-test migration decisions;
- future replacement of Truffle Bytecode DSL by a non-Truffle or later backend;
- bytecode serialization as a product/runtime feature;
- new public continuation, async, generator or coroutine concepts.

Those remain ordinary implementation work or separate decision gates as
applicable.

## 12. Recommendation — proposal pending owner approval

**Recommend Candidate C — Bytecode DSL as the sole target executable backend,
with the legacy Truffle AST retained only as bounded migration scaffolding until
its audited dependencies reach zero.**

The decisive reasons are not fashion or a generic preference for bytecode:

- Protos already selected Bytecode DSL for its hardest production execution
  requirement under PLAT014;
- current public language parse already uses the Bytecode path;
- AUD012 shows migration complexity but has not demonstrated a requirement for a
  second permanent backend;
- GraalPy supplies directly comparable Truffle evidence for AST -> bytecode
  transition followed by backend consolidation;
- V8 supplies strong independent evidence that retiring superseded execution
  pipelines materially simplifies language evolution and maintenance;
- Candidate C uniquely reaches the owner-approved single-backend end state while
  preserving the evidence required to get there safely.

## 13. Approval statement required

The project owner can ratify the proposal by explicitly approving:

```text
PLAT035 Candidate C

Bytecode DSL is the sole target executable backend.
The CanonicalToTruffleLowerer / executable ProtosExpressionNode backend is
legacy migration scaffolding only and must be removed after its audited
migration blockers reach zero, under the retirement contract in this packet.
```

Until that exact architecture is approved, PLAT035 remains
`NEEDS_USER_DECISION`, this document remains decision input rather than policy,
and no repository-wide legacy-backend deletion is authorized.
