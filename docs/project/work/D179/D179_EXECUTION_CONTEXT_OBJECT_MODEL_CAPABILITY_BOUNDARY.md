# D179 — Execution context object-model capability boundary

FORMAL_IDENTIFIER=D179  
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/703  
DECISION_STATE=OPEN / RESEARCH EXPANDED
DECISION_KIND=LANGUAGE / OBJECT-MODEL SEMANTICS  
TRIGGER=PLAT036 / #702  
PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9  
OPENING_PROJECT_RECORD_REVISION=17d0a4629212f51d19bd557e465ae8e2da69e3c0  
CANDIDATE_RECOMMENDED=NONE_PENDING_D179_C
CANDIDATE_RATIFIED=NO  
PREVIOUS_E1_STATUS=PARTIAL_HYPOTHESIS_SUPERSEDED_AS_RECOMMENDATION_NOT_REJECTED
PLAT036_STATE=BLOCKED  
PLAT036_BLOCKED_BY=D179

This document is investigation evidence and a decision packet. It is not a ratification record. No normative specification, product implementation, test, or benchmark change is authorized by this report.

## A. Baseline revisions and authorities

The investigation is revision-bound to:

    PRODUCT:
      guillermomolina/protos
      3e8e6b565c95eb5098c2168d241536ba13ad19e9

    OPENING PROJECT RECORD:
      guillermomolina/protos-project-docs
      17d0a4629212f51d19bd557e465ae8e2da69e3c0

    GOVERNING ISSUE:
      guillermomolina/protos#703

    BLOCKED CONSUMER:
      guillermomolina/protos#702

Repository authority consumed included AGENTS.md, AGENTS.work/DESIGN.md, AGENTS.work/COORDINATION.md, AGENTS.work/REFERENCE.md, the complete relevant normative sections of PROTOS_LANGUAGE_SPEC.md, OBJECT_MODEL.md, EXECUTION_AND_CONTROL.md, CALLABLES.md, VALUES_AND_COLLECTIONS.md, ABSTRACT_RUNTIME.md, and the implementation surfaces named by D179: ProtosActivation, ProtosObjectValue, lexical lookup/create/assign lowering, Closure capture, parameter/default establishment, Bytecode execution and debugger scope projection.

Historical owner-approved evidence that materially constrains this decision:

    AUD009-B1 / #601
      removeSlot(name)                             KEEP
      hasSlot / slotValue / slotNames / parent     KEEP
      freeze / FROZEN                              KEEP
      structural close / CLOSED                    KEEP
      ordinary local structural mutation model     KEEP

    AUD009-B4 / #606
      execution contexts as ordinary objects       KEEP
      Context -> Object                            KEEP
      parameters/locals/module bindings as slots   KEEP
      separate lexical-parent relation             KEEP
      lexical traversal local-only                 KEEP
      context intrinsic                            KEEP
      captured contexts by reference               KEEP
      late/current-context creation                KEEP
      assignment and receiver-fallback rules       KEEP

D179 was explicitly opened to revisit the combined capability consequence of those approved institutions: whether an execution-context slot must inherit the full structural-removal power of an arbitrary open object. Those KEEP outcomes are therefore constraints and reopening provenance, not an automatic answer.

## B. Exact current Protos semantics

### B.1 Context identity and topology

There is no separate language category called a local variable. Local bindings are local slots of execution-context objects. Execution contexts are ordinary Protos objects whose standard prototype is Context:

    executionContext -> Context -> Object

That delegation edge is not the lexical edge. Each activation also has a separate lexical-parent relation. Bare lexical lookup inspects only local slots along that lexical relation. Context / Object delegation is ignored during the lexical phase.

The intrinsic context denotes the current execution-context object. It may escape as an ordinary value and may be used through the ordinary object protocol. Stable object identity is observable.

### B.2 Read, creation and assignment are intentionally different

Bare read is:

    current context local slot
    -> enclosing/captured lexical context local slots
    -> receiver local/delegated member lookup
    -> SlotNotFound

Bare creation x: value performs no lookup and, after evaluating the RHS, creates x only in the current slot-creation context. Inside a function the normative text describes this as conceptually equivalent to context.x: value.

This equivalence matters: current semantics do not distinguish a declaration-derived lexical slot from an explicitly added local slot of the same execution context for subsequent lexical lookup.

Bare assignment x = value selects the nearest existing lexical local slot, otherwise the receiver's own local slot, otherwise fails. The destination is selected before RHS evaluation, is pinned across RHS effects, never delegates and never creates.

### B.3 Capture and late structure

Closures capture genuine lexical execution contexts by reference. They do not snapshot slot values. Consequently later mutation of an existing captured slot and later creation of a local slot in an open captured context are observable.

Parameter binding is ordinary slot establishment on the fresh activation context, left to right. There is no parameter predeclaration, TDZ object, uninitialized guest value or second parameter namespace. A default for a parameter may therefore resolve an outer same-name binding until that parameter's own local slot has actually been established.

### B.4 ABSENT versus present values

The semantic distinction is:

    ABSENT
    PRESENT(null)
    PRESENT(value)

null is an ordinary present Protos value. It never means slot absent.

The implementation matches this contract: local-slot presence is membership in the local-slot table; guest values are non-host-null objects. Removing a local slot removes its table entry.

### B.5 Ordinary structural operations on a context today

Because an execution context is an ordinary open object, its inherited Object surface currently applies to local slots:

    hasSlot(name)       local-only observation
    slotValue(name)     local-only read
    slotNames()         local-only enumeration
    removeSlot(name)    local-only structural removal
    close()             OPEN -> CLOSED; no further add/remove
    freeze()            -> FROZEN; no further mutation
    without(name)       fresh ordinary object; source unchanged
    alias(a,b)          fresh ordinary object; source unchanged

without and alias do not mutate the source context; their result is a fresh ordinary open object and they do not modify the lexical-parent graph.

### B.6 Exact present-day removeSlot consequence

For a current local x, context.removeSlot("x") today removes the local slot when the context is OPEN and returns the exact former value.

Afterward bare lookup runs again. Since the current context no longer has local x, lookup proceeds to lexical parents and then receiver fallback. Therefore:

    outer x exists
    inner local x exists
    inner context removes x
    later bare x

may resolve to outer x.

This is not stated as an independent lexical rule such as "removing a local reveals the outer binding." It is a derived consequence of four rules:

1. contexts are ordinary objects;
2. locals are context slots;
3. open-object removeSlot removes a local slot;
4. lexical lookup continues when a local slot is absent.

That derived consequence is the exact design boundary owned by D179.

### B.7 Required edge cases under current semantics

A. Local x and no outer x: removal makes it absent; lookup proceeds to receiver or SlotNotFound.

B. Local x and outer x: removal makes local absent; later bare x may reveal outer x.

C. Captured local x: the same context identity is captured; removal is visible; later Closure lookup may continue beyond the former local binding.

D. Escaped context: another path may remove a slot from the same context identity and later lexical lookup observes it.

E. Local x == null: x is PRESENT(null); removal changes it to ABSENT; assignment to null does not.

F. CLOSED context: removal is rejected and the slot remains.

G. FROZEN context: removal is rejected and the slot remains.

## C. Complete execution-context capability inventory

Current semantics do not store the requested classes in four different namespaces. A captured lexical slot is simply a local slot of another execution-context object reached through the capture chain.

| Capability | Bare/source local | Parameter | Explicit/dynamic context local | Captured local |
|---|---|---|---|---|
| read | yes when present | yes after establishment | yes when present | yes by reference |
| assign existing | yes | yes | yes | yes by reference |
| enumerate | yes | yes | yes | on owning context |
| hasSlot / slotValue | yes | yes | yes | on owning context |
| add/create | : on current context | sequential binder creates | context.name: while OPEN | later creation visible if owner open/reachable |
| remove | yes while OPEN | yes while OPEN | yes while OPEN | yes if owner open/reachable |
| alias / without | fresh-object transforms | same | same | same on owner |
| close / freeze | whole context | whole context | whole context | whole owner |
| delegation | explicit member send uses Context -> Object; bare lexical ignores it | same | same | same |
| identity / escape | stable and first class | context may escape | stable | captured identity stable |
| debugger/reflection | semantic locals | semantic locals | semantic locals | lexical projection |

Practical value and systemic cost:

| Capability | Demonstrated value | Design value | Systemic cost | Independently restrictable? |
|---|---|---|---|---|
| read/assign | extensive | fundamental lexical state | required | no |
| add/create | extensive | explicit growth/shadowing; late capture visibility | presence dynamic before establishment | not without reopening major KEEP invariants |
| reflection read/enumerate | general-object production/tool use; context programmability | ordinary-things-remain-ordinary | projection/materialization surface | technically yes, high philosophy cost |
| remove | no production context caller found | maximal structural uniformity | presence can decrease; later lexical identity can retarget | yes |
| without/alias | general object composition | non-mutating structural transforms | call-local | yes, but no D179 problem shown |
| close | cross-Core fixed-shape mutable state | integrity state | state checks | previously retained |
| freeze | real production/isolation role | immutable publication | state checks | previously retained |
| identity/escape | real semantic/tooling role | first-class execution state | mutation aliases | no without redesign |
| debugger | tooling | semantic transparency | invalidation/projection | separable, no need shown |

removeSlot is the only scoped ordinary-object operation that uniquely changes local membership from PRESENT to ABSENT. close/freeze reduce future authority, without/alias create other objects, assignment preserves membership and creation only increases membership.

## D. Real repository usage/value evidence

Repository-wide search on the baseline found:

    guest source using context.removeSlot(...)        0
    guest source using context.slotNames()             0
    guest source using context.slotValue(...)          0
    guest source using context.hasSlot(...)            0
    guest source using context.without(...)            0
    guest source using context.alias(...)              0

General removeSlot remains deliberately specified and covered for ordinary objects. AUD009-B1 previously recorded NON_TEST_PROTOS_CALLERS=0 and CONFORMANCE_PROTOS_CALLERS=11, then the project owner retained removeSlot as part of the complete same-identity open-object model. D179 does not propose deleting ordinary Object.removeSlot.

Real context programmability does exist. Conformance exercises escaped/current context identity and explicit context creation, including an assignment RHS that obtains the current context and creates a nearer slot. The specification explicitly makes x: value conceptually equivalent to context.x: value.

A concrete systemic consumer of the possibility of removal exists even without a guest context-removal call: ProtosStaticDefinitions clears exact-definition facts after opaque guest invocation because an invoked Closure can mutate or remove slots of a captured execution context. The mere semantic capability therefore imposes current analysis conservatism.

Central value classification:

    Useful current Protos programs requiring deletion of a lexical context binding:
      NONE FOUND

    Current use:
      ordinary-object completeness / conformance: YES
      lexical-context deletion in libraries/tools/apps: NO EVIDENCE
      internal unpublished bootstrap object normalization: YES, but it does not
      require guest-visible lexical-context deletion

    Theoretical value:
      prototype-model uniformity
      ordinary-things-remain-ordinary
      reflection and metaprogramming
      same-identity namespace surgery
      conceptual continuity with highly reflective prototype systems

## E. Prior-art comparison matrix

The survey is evidence, not majority voting.

| System | Local identity static? | Can become absent? | Structural local removal? | Reveals outer same-name? | Scope first-class? | Reflective mutation | Capture / dynamic extension / storage |
|---|---|---|---|---|---|---|---|
| Self | activation-slot based | mirror mutation permits removal in principle | yes through mirrors | exact adversarial case not proven by inspected source | yes, activation objects | yes through mirrors | lexical activations; activation mirrors |
| Squeak/Smalltalk | yes for temps/args | not by ordinary temp deletion | no ordinary structural temp deletion | no ordinary path | thisContext reflective | partial | compiled temp layout; closure/context projection |
| TruffleSqueak | yes / fixed compiled layout | values/state change | no ordinary slot-structure deletion evidence | no evidence | guest ContextObject | partial | ContextObject + MaterializedFrame; fixed descriptor |
| Python | yes per compile-time classification | yes, del unbinds | semantic unbind yes | no; local use can raise UnboundLocalError | frame partial | partial | cells/free vars; frame locals |
| GraalPy | yes | unbound represented | follows Python deletion | no | frame projection partial | partial | Bytecode/frame locals + PCell; frame-locals proxy |
| JavaScript | yes for declarative environments | initialization state exists | ordinary lexical deletion no | no | no ordinary scope object | limited | declarative env distinct from dynamic Object Environment Record |
| GraalJS | yes | semantic init states | follows JS | no | tooling projection | partial | MaterializedFrame closure/block scopes |
| Ruby | yes once parser classifies | value changes, no ordinary deletion | no | no | Binding partial | value read/write; binding-only dynavars | environment pointer; parser-known depth |
| TruffleRuby | frame-slot identity | value mutable | no ordinary lexical structural deletion | no | Binding/tool projection | partial | enclosing/materialized frame machinery |
| SOM / TruffleSOM | yes | values mutable | no ordinary lexical structural deletion | no | no ordinary scope object | tooling | frame slots/materialized outer context |
| Apple Pkl | yes for lexical definitions | no ordinary removal | no | no | no | no ordinary mutation model | FrameSlotVariable / lexical symbol table |
| Lua 5.4 | yes after declaration | value may be nil; identity remains | no | no | no | debug API can read/write values | indexed locals/upvalues |
| Protos current | no stable post-establishment identity if removal occurs | yes through removeSlot | yes | yes, derived | yes ordinary object | yes ordinary Object protocol | dynamic local slots + by-reference context chain |

Material findings:

1. Self is a genuine precedent for the strongest KEEP argument. Self activation state is object-like and mirrors can add/remove slots, including activation mirror kinds. The inspected sources do not establish the exact remove-current-local-then-reveal-outer behavior, so that narrower claim is not made.
2. Python permits deletion but preserves lexical identity classification: an unbound local does not fall through to an outer binding.
3. ECMAScript explicitly separates Declarative Environment Records from Object Environment Records. Object-backed environments can gain/lose names as properties; ordinary lexical declarations do not inherit ordinary property deletion.
4. Lua, Ruby, Pkl, SOM and Smalltalk preserve compiled local identity much more strongly than current Protos.
5. Ruby shows that first-class reflection does not require ordinary source lexical structure to equal a mutable property map: Binding can reflect/mutate locals and host binding-only dynamic variables.
6. TruffleSqueak shows that a guest context object and optimized frame/materialized-frame state can be coordinated rather than becoming independent semantic authorities.
7. Protos is unusual in combining first-class ordinary context object, dynamic context growth that immediately affects lexical lookup, and structural deletion that can retarget a later lexical read.

Implementation-source anchors used for the runtime half of the comparison:

    GraalPy        cebcc10a20c502f1956a0c436c94f53380944d61
    GraalJS        b700215562db9a8b33fb391d971faeb652131184
    TruffleSqueak  818519b2b6a6556bc524e9e0d08f7b51969cb61a
    TruffleRuby    0e6fa6a950dce7154d54f3c9c63056c4eb925ffd
    TruffleSOM     73f6d2e654022565ec7c7e8ba95ae18340a862ce
    Apple Pkl      d5a7dc021e1ea0c71afd52509238926c1d614851

The TruffleRuby and TruffleSOM anchors were already inspected in the immediately preceding PLAT036 comparative investigation and were reused as implementation evidence, not as authority for D179's semantic recommendation.

Primary/current external evidence included Python 3.14 documentation, the current ECMAScript specification, Lua 5.4, Ruby 3.4 documentation, current Pkl documentation, the Self handbook and implementation source for GraalPy, GraalJS, TruffleSqueak and Pkl.

## F. Candidate set

### Candidate A — KEEP current semantics

Contexts stay fully ordinary open objects. A present local can be removed. A later bare lookup observes absence and resumes outer lexical / receiver lookup.

### Candidate B — lexical UNBOUND tombstone

Removal changes BOUND(value) to UNBOUND. Lexical identity remains and a later read errors rather than revealing an outer name. This introduces a state distinct from ABSENT, PRESENT(null) and PRESENT(value), plus new reflection, debugger and reassignment rules.

### Candidate C — fixed source/parameter lexical membership plus removable dynamic extension

Source/parameter-derived slots are structurally fixed while separately classified dynamic context slots can be added/removed.

This has a Protos-specific defect: x: value is currently conceptually equivalent to context.x: value, and every local context slot participates in lexical lookup. A provenance split creates a new binding category. If the distinction is removed, C collapses toward E1.

### Candidate D — strong fixed lexical namespace

Activation lexical structure is fixed; dynamic context extension is absent or moved behind a separate model.

This offers the cleanest conventional fixed-layout implementation but conflicts with current late establishment: parameter defaults may see outer same-name bindings before their own parameter exists, and later context-slot creation is intentionally visible through capture. Predeclaration would change lookup.

### Candidate E1 — MONOTONIC EXECUTION-CONTEXT MEMBERSHIP

Keep contexts as ordinary Protos objects and keep dynamic growth, but impose one invariant while an object serves as an execution-context identity:

    ABSENT -> PRESENT(value)     allowed by creation while OPEN
    PRESENT(value) -> PRESENT(v) allowed by assignment when writable
    PRESENT(value) -> ABSENT     rejected for execution-context local slots

The rule applies to all local slots of an execution context, regardless of establishment path: bare :, parameter binding, explicit context.name:, module binding or later reflective creation. There is no static-vs-dynamic slot caste.

Consequences:

- context.removeSlot(name) on a present execution-context local fails before mutation;
- a missing local retains the existing missing-local failure;
- ordinary non-context Object.removeSlot stays unchanged;
- x: value remains equivalent to context.x: value;
- late creation remains visible through captured contexts;
- hasSlot, slotValue, slotNames, identity, escape, assignment, capture, close, freeze, without and alias retain current semantics;
- CLOSED additionally prevents future growth while permitting existing writable values;
- FROZEN prevents all mutation;
- no hidden UNBOUND guest state is introduced.

This is not a new guest-visible object type. It is an invariant on an object while it is the semantic execution-context identity, analogous in kind to the already separate lexical-parent relationship. Ordinary delegation and protocols remain; an operation can fail because of receiver state/invariant just as mutation already fails on CLOSED/FROZEN receivers.

### Candidate F — DEFER / KEEP current for now

No semantic decision now. PLAT036 must preserve full current presence-loss and fallback semantics and remains blocked until D179 is later resolved.

## G. Candidate-by-candidate semantic behavior

| Behavior | A | B | C | D | E1 | F |
|---|---|---|---|---|---|---|
| late x: creation | current | current | dynamic class | constrained/separate | current | current |
| sequential parameters | current | current | provenance required | predecl pressure | current | current |
| captured later creation | current | current | dynamic class | constrained | current | current |
| assign existing | current | current | current | admitted slots | current | current |
| remove current local | removes | tombstones | reject lexical / allow dynamic | reject | reject all context locals | removes |
| read after remove | may reveal outer | unbound error | dynamic may still reveal | unchanged/error | removal failed; local unchanged | may reveal outer |
| reflection after remove | slot absent | new UNBOUND rules needed | class-dependent | fixed namespace policy | unchanged because removal failed | absent |
| null | present | present | present | present | present | present |
| escaped context | full structural mutation | tombstone model | provenance-aware | restricted | add/write yes; remove no | full structural mutation |
| ordinary context object | yes | tombstone pressure | privileged slot classes | weaker | yes with invariant | yes |

## H. Stress/counterexample analysis

Legend: PASS means coherent without reopening the tested invariant; REQUIRES_SEMANTIC_CHANGE is a deliberate delta; FAIL means a required current invariant cannot be preserved without collapsing into another candidate or adding a larger mechanism; AMBIGUOUS means another rule must first be invented.

| Stress case | A | B | C | D | E1 | F |
|---|---|---|---|---|---|---|
| 1 simple read/write | PASS | PASS | PASS | PASS | PASS | PASS |
| 2 nested shadowing | PASS | PASS | PASS | PASS | PASS | PASS |
| 3 same-name outer/local | PASS | PASS | PASS | PASS | PASS | PASS |
| 4 remove current binding | PASS | REQUIRES_SEMANTIC_CHANGE | REQUIRES_SEMANTIC_CHANGE | REQUIRES_SEMANTIC_CHANGE | REQUIRES_SEMANTIC_CHANGE | PASS |
| 5 remove then read same name | PASS | REQUIRES_SEMANTIC_CHANGE | REQUIRES_SEMANTIC_CHANGE | REQUIRES_SEMANTIC_CHANGE | REQUIRES_SEMANTIC_CHANGE | PASS |
| 6 remove after Closure capture | PASS | REQUIRES_SEMANTIC_CHANGE | REQUIRES_SEMANTIC_CHANGE | REQUIRES_SEMANTIC_CHANGE | REQUIRES_SEMANTIC_CHANGE | PASS |
| 7 mutate after capture | PASS | PASS | PASS | PASS | PASS | PASS |
| 8 create after capture | PASS | PASS | PASS | FAIL unless separate extension | PASS | PASS |
| 9 escaped context modified elsewhere | PASS | PASS | PASS with provenance | restricted | PASS for add/write; remove rejected | PASS |
| 10 null versus absence | PASS | PASS but adds UNBOUND | PASS | PASS | PASS | PASS |
| 11 parameter binding | PASS | PASS | AMBIGUOUS provenance | FAIL if visibly predeclared | PASS | PASS |
| 12 default reads outer same-name | PASS | PASS | PASS if no predecl | FAIL if local visible before establishment | PASS | PASS |
| 13 default reflectively mutates context | PASS | PASS | class-dependent | restricted | PASS for add/write; remove rejected | PASS |
| 14 close | PASS | PASS | PASS | PASS | PASS | PASS |
| 15 freeze | PASS | PASS | PASS | PASS | PASS | PASS |
| 16 debugger enumeration | PASS | AMBIGUOUS UNBOUND visibility | provenance leak/hidden rule | AMBIGUOUS uninitialized names | PASS | PASS |
| 17 debugger write | PASS | AMBIGUOUS rebind | PASS with metadata | admitted slots | PASS existing slots | PASS |
| 18 suspension/resumption | PASS | PASS with extra state | PASS | PASS | PASS | PASS |
| 19 non-local return/unwind | PASS | PASS | PASS | PASS | PASS | PASS |
| 20 many nested scopes | PASS with dynamic burden | PASS | PASS | PASS | PASS | PASS with same burden |
| 21 many Closures | PASS with alias/removal burden | PASS | PASS | PASS | PASS | PASS with same burden |
| 22 multiple Truffle Contexts | PASS | PASS | PASS | PASS | PASS | PASS |
| 23 hypothetical non-Truffle backend | PASS | PASS | PASS | PASS | PASS | PASS |

The decisive adversarial cases are late creation and sequential/default parameters. They disqualify naïve predeclaration. E1 does not say that membership is fixed from activation creation; it says membership becomes stable only after establishment.

## I. Systemic implementation/runtime consequences

These are semantic/architecture constraints, not benchmark claims.

A/F current semantics:

    SEMANTIC CONSTRAINT:
      a present lexical binding is not stable; it may become absent

    IMPLEMENTATION COMPLEXITY:
      presence/removal and fallback must remain observable

    OPTIMIZATION / ANALYSIS BARRIER:
      exact binding facts cannot generally survive an opaque guest call that may
      reach the owning context

    MEASURED PERFORMANCE COST:
      NOT ESTABLISHED BY D179

    POSSIBLE PERFORMANCE COST:
      representation/guard/fallback complexity only; no numeric claim

The existing ProtosStaticDefinitions invalidation is direct evidence of this barrier.

B regains static identity but adds UNBOUND and all of its capture, reflection, debugger and reassignment semantics.

C can optimize a subset but needs persistent provenance metadata and conflicts with the current equivalence of bare and explicit context creation.

D makes layouts easiest but only by removing or relocating current language capabilities.

E1 makes membership monotonic after establishment:

    before establishment: ABSENT
    after establishment:  PRESENT(value), thereafter membership stable

It does not imply all names are known when the activation is created and does not make every read a fixed load. Opaque code may still add a nearer binding to an open captured context; parameters still appear sequentially.

It removes one uncertainty only: after a particular local binding exists, an arbitrary call cannot make that same membership disappear and retarget future lookup outward.

This can enable later indexed/frame architectures, but D179 makes no claim that the change explains PERF010/PERF011 or produces any measured speedup.

## J. GITHUB010 scoring matrix

Scores are 1–5. No arithmetic total is used.

### 1. Correctness / invariant preservation

A 5 — exact current semantics and prior KEEP outcomes — HIGH.  
B 3 — stable identity but new unbound/reflection semantics — HIGH.  
C 2 — conflicts with x: / context.x: equivalence unless expanded — HIGH.  
D 1 — conflicts with late establishment/default visibility in naïve form — HIGH.  
E1 4 — changes one derived structural capability while preserving every other identified execution invariant — HIGH.  
F 5 — no semantic change now — HIGH.

### 2. Protos alignment

A 5 — maximum everything-is-object structural uniformity — HIGH.  
B 3 — hidden UNBOUND state is alien to the present slot model — HIGH.  
C 2 — creates privileged slot provenance inside one context — HIGH.  
D 2 — materially weakens dynamic object/context model — HIGH.  
E1 4 — keeps ordinary identity/growth/reflection; adds one role invariant — HIGH.  
F 5 — current philosophy unchanged — HIGH.

### 3. Present-need proportionality / pay for what you need

A 2 — no real lexical-deletion consumer found; systemic possibility remains — HIGH.  
B 2 — adds a new state to preserve an unused deletion operation — HIGH.  
C 3 — targets some cost but adds provenance category — MEDIUM.  
D 1 — removes more capability than current need justifies — HIGH.  
E1 5 — removes only the unneeded decreasing-membership capability from contexts — HIGH.  
F 2 — continues paying the current constraint without demonstrated need — HIGH.

### 4. Incremental growth / grow as you need

A 3 — already maximally dynamic; later restriction is breaking — MEDIUM.  
B 2 — tombstone semantics become permanent surface — MEDIUM.  
C 4 — growth retained but provenance complicates evolution — MEDIUM.  
D 2 — restoring dynamic growth later would be substantial — HIGH.  
E1 5 — keeps growth; explicit unbinding can be designed later if real need appears — HIGH.  
F 2 — postpones the boundary and keeps PLAT036 constrained — HIGH.

### 5. Future-option resilience

A 3 — maximal metaprogramming but narrows representation options — MEDIUM.  
B 4 — stable identity helps runtimes, but unbound contract persists — MEDIUM.  
C 4 — supports hybrid runtimes at the cost of slot classes — MEDIUM.  
D 3 — runtime-friendly but semantically overcommitted — MEDIUM.  
E1 5 — preserves growth, identity, capture and backend freedom — HIGH.  
F 2 — blocks the current architecture decision and defers migration risk — HIGH.

### 6. Scalability

A 2 — deep/many-scope reasoning remains fully presence-dynamic — MEDIUM.  
B 4 — fixed identity helps scalable indexing/materialization — MEDIUM.  
C 4 — fixed subset scales; dynamic subset remains generic — MEDIUM.  
D 5 — strongest static layout — HIGH.  
E1 4 — stable established membership plus dynamic growth is a scalable hybrid input — MEDIUM.  
F 2 — same constraint as A with no decision — MEDIUM.

### 7. Conceptual simplicity

A 4 — one ordinary object rule; surprising lexical retarget is the cost — HIGH.  
B 2 — ABSENT/PRESENT/null plus UNBOUND — HIGH.  
C 2 — two kinds of otherwise local context slot — HIGH.  
D 3 — simple static namespace only after removing current dynamic behavior — MEDIUM.  
E1 5 — one rule: execution contexts may grow but not shrink — HIGH.  
F 4 — current rule simple, design unresolved — HIGH.

### 8. Portability / implementation freedom

A 3 — implementable but forces highly dynamic lexical presence — MEDIUM.  
B 4 — common unbound/cell patterns exist — HIGH.  
C 4 — hybrid static/dynamic patterns are well precedented — HIGH.  
D 5 — conventional compiled-local model — HIGH.  
E1 5 — backend-neutral invariant; no frame technology mandated — HIGH.  
F 2 — retains the strongest dynamic constraint on future backends — MEDIUM.

### 9. Runtime / resource cost

A 2 — removal/presence-aware genericity remains required — MEDIUM.  
B 3 — fixed identity plus tombstone state/checks — MEDIUM.  
C 4 — compact static subset; provenance/dynamic overflow remain — MEDIUM.  
D 5 — easiest compact fixed layout — HIGH.  
E1 4 — removes decreasing-presence case; dynamic growth still needs presence handling — MEDIUM.  
F 2 — same as A — MEDIUM.

This is architecture/resource pressure, not a measured performance score.

### 10. Failure / operability

A 3 — successful removal may silently retarget a later bare name outward — HIGH.  
B 4 — later use errors instead of retargeting, but new unbound failures exist — HIGH.  
C 3 — operation success depends on hidden slot provenance — HIGH.  
D 4 — fixed failures predictable, capability loss broad — MEDIUM.  
E1 5 — attempted shrink fails at mutation site; no later silent retarget — HIGH.  
F 3 — current delayed retarget remains — HIGH.

### 11. Cost of deferral / reversibility / migration

A 2 — keeping deletion longer can bake it further into PLAT036 architecture — MEDIUM.  
B 3 — migration requires unbound state across spec/runtime/debugger — MEDIUM.  
C 3 — migration requires provenance and compatibility rules — MEDIUM.  
D 1 — largest source-semantic migration — HIGH.  
E1 4 — narrow compatibility break; explicit unbind can be added deliberately later — HIGH.  
F 1 — PLAT036 remains blocked/constrained and later change grows costlier — HIGH.

### 12. Evidence maturity / implementation risk

A 5 — current behavior directly reconstructed and implemented — HIGH.  
B 4 — strong Python-like precedent, Protos reflection integration unresolved — HIGH.  
C 3 — common runtime idea, weak fit to current Protos equivalence — HIGH.  
D 4 — abundant precedent, direct Protos incompatibility clear — HIGH.  
E1 4 — small evidence-backed delta; implementation architecture remains for PLAT036 — MEDIUM.  
F 5 — no immediate implementation risk because nothing changes — HIGH.

## K. Cost-of-deferral analysis

Deferring is not neutral because PLAT036 depends on whether an established lexical binding may disappear.

Under deferral PLAT036 must assume:

    PRESENT -> ABSENT is possible
    later lookup may resume outward
    captured/escaped aliases can trigger the transition
    static definition facts may be invalidated by opaque calls
    frame/index identity must preserve dynamic removal/fallback semantics

PLAT036 remains possible but needs the more general mechanism. A later restriction then migrates both language semantics and whatever representation was built to preserve them.

No real lexical-deletion consumer was found and the blocked platform decision is current work, so the cost of deferral is material.

## L. Compatibility/migration consequences

For E1 the source incompatibility is narrow but real:

    A program that successfully calls removeSlot on an execution-context local
    today would instead receive a mutation failure and the local would remain.

No such production repository program was found; unknown external code may exist.

Unaffected: ordinaryObject.removeSlot, local assignment, local creation, explicit context creation, parameter/default establishment, late captured-context creation, by-reference capture/mutation, identity/escape, Context -> Object, lexical ordering, receiver fallback, reflection, without/alias, close/freeze, null semantics and debugger writes to existing slots.

A future explicit lexical-unbind feature remains possible. If demand appears, it should define lookup/reflection/Closure semantics directly rather than inheriting object-shape deletion accidentally.

## M. Explicitly preserved invariants

E1 preserves:

    execution contexts are ordinary Protos objects
    fresh context per invocation
    Context -> Object and public Context binding
    context intrinsic and stable identity
    separate lexical-parent relation
    lexical traversal local-only
    receiver/delegation fallback
    locals/parameters/module bindings are context slots
    bare x: creates on current context
    x: value conceptually equivalent to context.x: value
    late context growth while OPEN
    Closure capture by reference
    later mutation visible through capture
    later creation visible through capture
    sequential/default parameter visibility
    ABSENT distinct from PRESENT(null)
    assignment destination and pinning rules
    writes never delegate
    object-construction/method capture rules
    close / CLOSED
    freeze / FROZEN
    ordinary Object.removeSlot for non-context objects
    local reflection
    without / alias fresh transforms
    suspension/resumption and control semantics
    debugger/source/tag semantics
    multiple Truffle Context isolation
    backend independence

## N. Explicitly changed/reopened invariants, if any

### N.1 GITHUB021 reconciliation

Previously owner-approved facts:

    B1: removeSlot is KEEP in the complete ordinary open-object structural model
    B4: execution contexts are ordinary objects
    B4: locals are execution-context slots

Their combination currently implies that an OPEN execution context can remove a local slot. D179 explicitly reopened this combined consequence.

E1 replacement:

    OLD:
      an OPEN execution-context object may remove any present local slot

    NEW:
      execution contexts remain ordinary objects, but while an object serves as
      an execution-context identity its local membership is monotonic:
        add while OPEN = permitted
        write existing when writable = permitted
        remove present local = rejected

Observable delta:

    OLD:
      inner local x -> context.removeSlot("x") succeeds
      -> later x may reveal outer x

    E1:
      inner local x -> context.removeSlot("x") fails without mutation
      -> local x remains the binding

This is substantive and cannot be ratified from prior audit approval. Exact owner approval of E1 is required.

No proposal is made to reverse B1's general Object.removeSlot KEEP decision.

## O. Questions intentionally deferred

D179 does not decide:

- PLAT036's BytecodeLocal / indexed-store / frame-first / cell / adapter architecture;
- exact Truffle invalidation or quickening;
- lazy physical context-object creation;
- PERF010/PERF011 causal attribution;
- a future lexical unbind feature or UNBOUND tombstone;
- selector naming for structural close;
- general transferability of user-created objects delegating to Context;
- debugger UI beyond preserving semantic observation;
- serialization/distributed transport of contexts.

The exact existing Error family to use for prohibited execution-context removal is also deferred to normative implementation if E1 is ratified; D179 does not invent a new Error family.

## P. Strongest argument for KEEPING the current model

The strongest case is semantic uniformity, not sunk implementation.

Protos deliberately makes execution state an ordinary first-class object. context can be reflected on, passed and mutated with the same protocol as other mutable state. OPEN then has one uniform meaning: shape may grow and shrink; CLOSED fixes shape; FROZEN fixes values.

Self demonstrates serious prior art for activation state that is object-like and reflectively structurally manipulable. Restricting one inherited capability only for execution contexts creates an exception programmers and implementations must know.

If maximal reflective uniformity outranks representation freedom, A is coherent and defensible.

## Q. Strongest argument against the historical E1 partial hypothesis

E1 weakens the strongest possible meaning of "context is an Object."

Two otherwise ordinary OPEN objects may reject the same removeSlot solely because one is serving as an execution-context identity. That is a privileged invariant not derivable from ordinary OPEN/CLOSED/FROZEN state alone.

Absence of current lexical-deletion consumers is not proof that live programming, debugging or metaprogramming will never need it. Self-style activation reflection is unusually close to Protos' philosophy, so abandoning full structural uniformity could be premature.

## R. Historical partial recommendation — superseded pending expanded audit

The original investigation recommended Candidate E1 — MONOTONIC EXECUTION-CONTEXT MEMBERSHIP. After owner review exposed that the packet had over-focused on `removeSlot`, E1 is retained only as a partial hypothesis and evidence artifact. It is **not** the current D179 recommendation and is not ready for owner approval. Parent D179 will reconstruct the candidate set after D179-A, D179-B and D179-C complete.

Historical E1 candidate:

Exact candidate:

    1. Execution contexts remain ordinary Protos objects with stable identity,
       Context -> Object delegation, ordinary reflection and ordinary sends.

    2. Local context membership is dynamic but monotonic.

    3. While an execution context is OPEN:
         ABSENT -> PRESENT(value)        allowed
         PRESENT(value) -> PRESENT(v2)   allowed
         PRESENT(value) -> ABSENT        rejected

    4. The no-removal rule applies to every local slot, regardless of
       establishment path. No lexical-vs-dynamic slot class is introduced.

    5. context.removeSlot(name):
         present local on execution context -> fail before mutation
         missing local -> existing missing-local failure

    6. Object.removeSlot is unchanged for ordinary non-context objects.

    7. close/freeze retain current behavior:
         CLOSED additionally forbids growth but permits existing writes;
         FROZEN forbids growth and existing writes.

    8. x: value and context.x: value remain conceptually equivalent.

    9. Late creation and captured-context growth remain legal and visible.

    10. No UNBOUND/tombstone guest state is introduced.

Why E1:

- B adds a fourth binding state and reflection rules solely to preserve an unused deletion capability.
- C creates slot provenance classes that current Protos explicitly does not have.
- D changes too much: late creation and sequential/default parameter semantics are useful, approved and coherent.
- E1 removes only decreasing membership while preserving dynamic growth and ordinary context identity.

This is evidence only until the project owner approves this exact candidate.

## S. Exact observable semantic delta of the recommendation

Current:

    outer x exists
    inner:
      x: localValue
      context.removeSlot("x")   succeeds
      x                         may resolve outer x

E1:

    outer x exists
    inner:
      x: localValue
      context.removeSlot("x")   signals; no mutation
      x                         still denotes unchanged local x if execution resumes

The same applies after capture or escape. A Closure or another holder of the context may still read, assign existing writable slots, create new local slots while OPEN, enumerate/reflect, close and freeze; it may not make an existing local absent.

Before establishment behavior is unchanged:

    a default parameter may still read outer same-name binding
    a later parameter remains absent until established
    later x: may still shadow outer x
    a Closure may still observe a slot created later in captured open context

## T. Consequence for PLAT036

If and only if E1 is explicitly owner-approved and later durably ratified, PLAT036 may assume:

    LEXICAL_IDENTITY_FIXED_AFTER_ESTABLISHMENT=YES
    LEXICAL_PRESENCE_FIXED_BEFORE_ESTABLISHMENT=NO

    LEXICAL_BINDING_STRUCTURAL_REMOVAL=REJECT_ON_EXECUTION_CONTEXT
    REMOVAL_REOPENS_OUTER_LOOKUP=NO

    DYNAMIC_CONTEXT_SLOTS=ADD_ALLOWED_WHILE_OPEN_REMOVE_REJECTED
    LATE_CAPTURED_CONTEXT_GROWTH=YES

    CONTEXT_REMAINS_PROTOS_OBJECT=YES
    CONTEXT_IDENTITY_STABLE=YES
    CONTEXT_REFLECTION_REQUIRED=YES

    ABSENT_TO_PRESENT_TRANSITION=YES
    PRESENT_TO_ABSENT_TRANSITION=NO
    PRESENT_NULL_REMAINS_PRESENT=YES

    FRAME_FIRST_REPRESENTATION_SEMANTICALLY_ADMISSIBLE=CONDITIONAL

CONDITIONAL means PLAT036 must still preserve absence before establishment, sequential parameter/default visibility, dynamic growth on open contexts, by-reference capture, stable semantic context identity, exact reflection/debugger projection, one coherent value authority without divergent frame/object copies, and dynamic fallback where binding identity is not statically proved.

D179 does not ratify a frame-first implementation, Bytecode DSL local model, cell model, context adapter or other PLAT036 candidate.

Final investigation state:

    D179_STATE=OPEN / RESEARCH EXPANDED
    CANDIDATE_RECOMMENDED=NO
    CANDIDATE_RECOMMENDED_ID=NONE_PENDING_D179_A_B_C
    PREVIOUS_E1_STATUS=PARTIAL_HYPOTHESIS_SUPERSEDED_AS_RECOMMENDATION_NOT_REJECTED
    CANDIDATE_RATIFIED=NO

    PLAT036_STATE=BLOCKED
    PLAT036_BLOCKED_BY=D179

    SPECIFICATION_CHANGED=NO
    RUNTIME_CHANGED=NO
    TESTS_CHANGED=NO
    BENCHMARKS_CHANGED=NO


## U. Expanded capability audit after owner review

The project owner clarified that D179 must answer the broader question originally
intended by AUD016/PLAT036: whether **any** current execution-context semantic
capability, not only `removeSlot`, materially constrains or penalizes faithful
adoption of Truffle Bytecode DSL mechanisms.

The first packet is therefore classified as **partial**. It remains useful
research, especially for structural removal and lexical membership, but its E1
recommendation is withdrawn as premature rather than rejected on the merits.
No exact candidate is currently pending owner approval.

Three independently meaningful investigation children now own the missing
surfaces:

- D179-A / #704 — execution-context structural mutation capability audit;
- D179-B / #705 — first-class execution-context reflection and escape boundary
  audit;
- D179-C / #706 — lexical dynamism and Bytecode DSL static-identity audit.

Their division is intentional:

    D179-A
      asks which structural mutations/authority changes invalidate static facts,
      including remove, late add, escaped/captured mutation, close and freeze.

    D179-B
      asks what first-class Object identity/reflection/escape actually requires
      semantically versus what is merely current physical storage.

    D179-C
      asks which lexical identities/presence/depth can genuinely be static and
      where dynamic fallback remains semantically necessary.

D179-C consumes material findings from A/B before final classification. Parent
D179 then rebuilds the complete candidate set, GITHUB010 scoring and owner
recommendation from the combined evidence.

The intended final question is now explicitly:

> For every observable execution-context capability, what useful Protos behavior
> does it provide, what Bytecode DSL/native-runtime fact does its mere possibility
> invalidate or complicate, can it be preserved on a slow/materialized path, and
> is any restriction justified as a language decision?

The children do not independently ratify semantics. PLAT036 / #702 remains
blocked by D179 as a whole.

### Coordination caveat

Each child Issue was created with an explicit `Parent: #703` declaration. The
GitHub connector available during allocation did not expose native Parent/Sub-
issue mutation. Under GITHUB006/GITHUB015, textual parent prose does not satisfy
the native-parent postcondition; native hierarchy reconciliation therefore
remains visibly pending until repository intake automation or another supported
interface establishes and verifies the relation.

Current expanded state:

    D179_STATE=OPEN / RESEARCH EXPANDED
    D179_A=#704 OPEN / READY
    D179_B=#705 OPEN / READY
    D179_C=#706 OPEN / READY

    CANDIDATE_RECOMMENDED=NO
    PREVIOUS_E1_STATUS=PARTIAL_HYPOTHESIS_SUPERSEDED_AS_RECOMMENDATION_NOT_REJECTED
    CANDIDATE_RATIFIED=NO

    PLAT036_STATE=BLOCKED
    PLAT036_BLOCKED_BY=D179

    SPECIFICATION_CHANGED=NO
    RUNTIME_CHANGED=NO
    TESTS_CHANGED=NO
    BENCHMARKS_CHANGED=NO


## V. D179-A completion checkpoint

D179-A / #704 completed the structural-mutation capability audit against the
same product baseline:

    PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9

The central result materially refines the historical E1 hypothesis:

    REMOVE_SLOT_UNIQUELY_PROBLEMATIC=NO

`removeSlot` is uniquely the audited operation that permits
`PRESENT -> ABSENT`, but arbitrary late creation in a nearer escaped or
captured context can perform `ABSENT -> PRESENT` and also retarget later bare
lookup from an outer/receiver binding to the newly-created nearer binding.

Consequently:

    E1_REMOVES_REMOVAL_DRIVEN_RETARGETING=YES
    E1_REMOVES_ALL_STATIC_IDENTITY_CONSTRAINTS=NO

D179-A also separates:
- existing-value mutation, which preserves binding identity/topology;
- structural add/remove, which can invalidate presence, identity, depth and
  fallback decisions; and
- close/freeze authority transitions, which can be preserved with mutation-path
  state guards/invalidation and do not force generic ordinary reads.

D179-A found a plausible no-change preservation model using fixed/indexed locals
for statically admitted bindings, explicit presence state, materialized captured
locals, dynamic overflow for runtime-introduced names, structural invalidation,
and a coherent semantic context/tooling adapter. This is evidence only and does
not select PLAT036 architecture.

Parent D179 is not ready for a new recommendation yet:

    D179_A=COMPLETE
    D179_B=REQUIRED
    D179_C=REQUIRED
    CANDIDATE_RECOMMENDED=NO
    PREVIOUS_E1_STATUS=PARTIAL_HYPOTHESIS_SUPERSEDED_AS_RECOMMENDATION_NOT_REJECTED
    PLAT036_STATE=BLOCKED

The native GitHub hierarchy was re-read after the D179-A investigation and the
D179-A/B/C children are now attached natively to #703, satisfying the previously
pending hierarchy postcondition.

## VI. D179-B completion checkpoint

D179-B / #705 completed the first-class execution-context reflection and escape
boundary audit against the same product baseline.

Exact durable evidence:

    D179_B_PROJECT_RECORD_REVISION=88f87f5d368ef0ccba1202b02def6f98ee60cc03
    D179_B_RESULT=COMPLETE

The central result is that the current language semantics require a stable
identity-bearing first-class context object/value and exact Object/reflection
behavior, but do not require every ordinary lexical value to remain physically
stored in `ProtosObjectValue.localSlots`.

D179-B establishes:

    CONTEXT_SEMANTIC_IDENTITY_REQUIRED=YES
    EAGER_PHYSICAL_CONTEXT_OBJECT_REQUIRED=NO
    ESCAPE_REQUIRES_OBJECT_BACKED_VALUES=NO
    REFLECTION_CAN_PROJECT_FRAME_LOCALS=YES
    ONE_SEMANTIC_VALUE_AUTHORITY_REQUIRED=YES
    EXTERNAL_STRUCTURAL_MUTATION_REQUIRES_INVALIDATION=YES
    CAPTURE_REQUIRES_SHARED_OR_MATERIALIZED_BINDING_AUTHORITY=YES
    DEBUGGER_REQUIRES_MATERIALIZATION=NO
    EXPLICIT_SEMANTIC_PRESENCE_REQUIRED=YES
    PRESENT_NULL_DISTINCT_FROM_ABSENT=YES

It also identifies persistent/special cases that cannot be collapsed into one
ordinary ephemeral-activation representation:

    MODULE_CONTEXT_PERSISTENT_AUTHORITY_REQUIRED=YES
    OBJECT_CONSTRUCTION_CONTEXT_REQUIRES_SEPARATE_CLASSIFICATION=YES
    FROZEN_PRELUDE_REQUIRES_SEPARATE_CLASSIFICATION=YES

No current context capability was proven to require restriction merely to make
a frame/local representation semantically possible. Reflection, explicit
context member access/mutation, structural add/remove, close/freeze,
`without`/`alias`, debugger projection and escaped structural mutation all
retain plausible semantics-preserving slow/projection paths.

This does not ratify a D179 candidate or select PLAT036 architecture. D179-C
must now classify the exact static/indexed admission boundary and dynamic
fallback set using both D179-A and D179-B evidence.

Current expanded state:

    D179_STATE=OPEN / RESEARCH EXPANDED
    D179_A=#704 COMPLETE
    D179_B=#705 COMPLETE
    D179_C=#706 OPEN / READY

    CANDIDATE_RECOMMENDED=NO
    CANDIDATE_RATIFIED=NO
    NEXT_REQUIRED_RESEARCH=D179_C

    PLAT036_STATE=BLOCKED
    PLAT036_BLOCKED_BY=D179

    SPECIFICATION_CHANGED=NO
    RUNTIME_CHANGED=NO
    TESTS_CHANGED=NO
    BENCHMARKS_CHANGED=NO


## VII. D179-C completion checkpoint

D179-C / #706 completed the lexical-dynamism and Bytecode DSL static-identity
audit against the same product baseline:

    PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9

Exact durable child evidence:

    D179_C_PROJECT_RECORD_REVISION=5bf11ca33d152cb944067a7e4d39b30d58900ee4
    D179_C_RESULT=COMPLETE

D179-C confirms and sharpens the combined D179-A/B result.

The current implementation loses useful source-known lexical facts before the
Bytecode DSL:

    SurfaceName
      -> CanonicalLookup(String)
      -> Lookup(ProtosActivation, String)
      -> dynamic context traversal

while existing repository static-analysis machinery can already prove exact
binding origins for a bounded set and deliberately invalidates those facts at
opaque effect barriers.

Central result:

    STATIC_INFORMATION_LOST_BEFORE_DSL=YES
    BYTECODE_DSL_PREVENTS_STATIC_LEXICAL_IDENTITY=NO

D179-C establishes a useful static/indexed admission set without requiring a
language restriction:

    CURRENT_PROTOS_SEMANTICS_PERMIT_A_USEFUL_STATIC_INDEXED_LEXICAL_ADMISSION_SET=YES
    D179_LANGUAGE_RESTRICTION_REQUIRED_FOR_USEFUL_STATIC_ADMISSION_SET=NO

Definitely-established current parameters/locals and fixed-membership existing
bindings can use stable current-frame/indexed identity. Definitely-established
captured bindings can use fixed lexical depth/binding identity when the target is
PRESENT and all nearer same-name candidates are proven ABSENT.

The audit also preserves the sibling correction that membership is the material
dynamic fact:

    PRESENT -> ABSENT
      can retarget lookup

    ABSENT -> PRESENT in a nearer context
      can also retarget lookup

Those cases require invalidation/guards/fallback for affected accesses, not
universal String-key lexical resolution:

    ARBITRARY_STRUCTURAL_MUTATION_FORCES_ALL_LEXICAL_ACCESSES_DYNAMIC=NO
    STRUCTURAL_MUTATION_REQUIRES_INVALIDATION=YES
    GENERIC_DYNAMIC_FALLBACK_REQUIRED_FOR_ALL_LEXICAL_ACCESSES=NO

Source-known future bindings may have preallocated physical slots provided
semantic presence remains explicit:

    SOURCE_KNOWN_LATE_CREATION_CAN_USE_PREALLOCATED_SLOT_PLUS_PRESENCE=YES
    EXPLICIT_PRESENCE_STATE_REQUIRED=YES
    PRESENT_NULL_DISTINCT_FROM_ABSENT=YES

This preserves sequential/default parameter semantics and module partial
initialization. A physical slot is not itself evidence that a semantic binding is
PRESENT.

D179-C additionally confirms:

    CONTEXT_IDENTITY_REQUIRES_GENERIC_LEXICAL_LOOKUP=NO
    REFLECTION_REQUIRES_GENERIC_LEXICAL_LOOKUP=NO
    ESCAPE_REQUIRES_GENERIC_LEXICAL_LOOKUP=NO
    CAPTURE_BY_REFERENCE_REQUIRES_GENERIC_LEXICAL_LOOKUP=NO
    EXISTING_VALUE_MUTATION_INVALIDATES_BINDING_IDENTITY=NO

The assignment rule remains especially important: the writable destination is
selected before RHS evaluation and must remain pinned even if the RHS changes
same-name context structure. Direct binding identity is therefore compatible
with, and naturally represents, the current semantic rule.

The genuinely dynamic fallback set still includes unresolved names, semantic
absence during sequential establishment, invalidated nearest-binding topology,
runtime/nonstatic structural names, receiver/delegation fallback, runtime-name
reflection and construction-specific dynamic structure.

D179-C separately classifies:

    MODULE_CONTEXT
    FROZEN_PRELUDE
    OBJECT_CONSTRUCTION_CONTEXT
    DEBUGGER_SEMANTIC_SCOPE_PROJECTION

so PLAT036 must not collapse them blindly into one ephemeral invocation-local
frame model.

The child does not select a BytecodeLocal/cell/materialized-local/context-adapter
architecture. It only establishes the semantic admission and fallback boundary.

### Combined expanded-research state

All three expanded children are now complete:

    D179_A=#704 COMPLETE
    D179_B=#705 COMPLETE
    D179_C=#706 COMPLETE

Therefore the parent can now perform the step that was intentionally deferred
when the original E1 recommendation was withdrawn:

    rebuild the complete meaningful D179 candidate set
    re-score surviving candidates under GITHUB010
    re-run adversarial/invariant-delta checks
    publish one RECOMMENDATION — PENDING OWNER APPROVAL

This does not itself approve any candidate.

Current state:

    D179_STATE=OPEN / DECISION PACKET REBUILD READY
    D179_PARENT_DECISION_READY=YES
    CANDIDATE_RECOMMENDED=NO
    CANDIDATE_RATIFIED=NO
    NEXT_REQUIRED_RESEARCH=D179_PARENT_SYNTHESIS

    PLAT036_STATE=BLOCKED
    PLAT036_BLOCKED_BY=D179

    SPECIFICATION_CHANGED=NO
    RUNTIME_CHANGED=NO
    TESTS_CHANGED=NO
    BENCHMARKS_CHANGED=NO
