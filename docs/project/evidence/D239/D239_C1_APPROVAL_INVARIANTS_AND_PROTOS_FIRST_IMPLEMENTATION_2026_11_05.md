# D239 — C1 owner approval, comparative evidence, invariant audit and I145 handoff

**Date:** 2026-11-05.
**Decision owner:** [D239 #1055](https://github.com/guillermomolina/protos/issues/1055).
**Implementation owner:** [I145 #1056](https://github.com/guillermomolina/protos/issues/1056), single maximal Protos-first slice I145-A.
**Durable decision:** [D239 — Ordinary construct hook C1](../../decisions/language/D239_ORDINARY_CONSTRUCT_HOOK_C1_RATIFIED.md).
**Product inspection revision:** [protos@5fc18249ea425d99fcfcfe387be950678b7ac5c5](https://github.com/guillermomolina/protos/commit/5fc18249ea425d99fcfcfe387be950678b7ac5c5). The exact revision is an inspection baseline only; no D239 product source or normative patch exists in it.
**Publication category:** Non-normative research/owner approval evidence. Human-executed product validation and publication **NOT RUN/NOT CLAIMED**.

## Owner decision and conflict check (GITHUB021)

The active conversation supplied a D239 decision packet with five alternatives, comparative precedents, adversarial cases, a 12-dimension scoring analysis, and explicit material breaks. It recommended **“D239-C1 — Ordinary construct hook”**, explaining: ordinary `receiver(args)` still selects `call`; the actual standard `Object.call` forwards an ordinary eligible non-Closure receiver to `receiver.construct(P,K)`; standard `Object.construct` allocates a fresh child, invokes its selected `init` exactly once, ignores the result, and returns that child; a custom `construct` returns its chosen value with no automatic second initialization.

The packet disclosed (a) a changed meaning for pre-existing own/inherited `construct` slots, including non-Closure shadow errors; (b) the six-slot canonical Range surface replacing own `call` with own `construct` and the new inherited meaning of `Range.call`; (c) the need to preserve both D237 transparent forwarding layers and D013 source-order lookup; (d) exact Map literal/canonical factory and native-value-family reconciliation; and (e) no new parser syntax, callable type or `[[Construct]]` dispatch. The project owner replied **“Apruebo C1”** on 2026-11-05 in the same conversation and instructed issue/evidence updates. This approves the **presented** C1 delta, not unspecified later semantic variations.

**GITHUB021 check:** D013 ordinary late `call` selection, receiver/home, Closure overrides, object slot lookup, and structural callable testing remain intact. D013's existing direct allocation inside `Object.call` is deliberately replaced by a normal `construct` message. D236's canonical Range own-call membership is deliberately replaced by own-construct membership, while direction, instance parent, frozen state, equality and derived-receiver behavior remain intact. D237 complete positional/named vectors and target-signature association must survive two transparent standard bridges. No other owner invariant is silently reopened. Final specification wording belongs to I145; failures to preserve these invariants are implementation defects, not permission to invent a fallback.

## Exact source-grounded audit

| Product files at pinned baseline | Observed constraint and implementation consequence |
|---|---|
| `AGENTS.md`; `AGENTS.work/DESIGN.md`; `AGENTS.work/COORDINATION.md`; `AGENTS.work/REFERENCE.md`; `AGENTS.work/IMPLEMENTATION.md` | GITHUB010/GITHUB021 explicit approval, human executor, source vs publication metadata discipline, Protos-first philosophy |
| `spec/semantics/CALLABLES.md` ` Polymorphic Invocation and Object Construction | D013 ordinary slot `call`, post-argument lookup, selected Closure, standard `Object.call` child+`init`, transparent D237 forwarding |
| `spec/semantics/OBJECT_MODEL.md`; `EXECUTION_AND_CONTROL.md`; `MODULES.md`; `spec/PROTOS_GRAMMAR.md` | Immutable delegation parent, ordinary reflection/super and method-home, left-to-right evaluation, Actor-local module contexts and no new parser entrypoint |
| `spec/semantics/VALUES_AND_COLLECTIONS.md` | Array standard indexed state, Map/IdentityMap custom state, `%{...}` canonical eligibility, D236 exact six Range slots and opposite-parent instances |
| `spec/runtime/ABSTRACT_RUNTIME.md` | Informative model must follow normative changes, not redefine them |
| `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardObjectProtocol.java` | Actual standard root `call` closure, `call`/`init` installation, native existing child allocation then `init`; add canonical `construct` slot and move default construction to it |
| `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosPreparedCalls.java`; `.../structured/ProtosStructuredControlCalls.java` | **Structured standard Object.call currently preallocates a child before preparing `init`** and forces child identity as result; C1 must instead select `construct` and return a custom result unchanged. Default standard construct then owns its own init-child completion |
| `src/main/java/com/guillermomolina/protos/execution/bytecode/named/ProtosNamedArgumentAssociation.java`; `ProtosNamedCallPreparation.java`; `ProtosSendPreparation.java` | Standard native name forwarding is currently specific to Object.call; add equivalent selected-implementation forwarding for actual Object.construct, never name-based privilege |
| `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardMapProtocol.java` | `selectFactoryCallForMapConstruction` currently requires canonical Map `call` and invokes it directly; canonical acceptance after C1 must cover normal selected Object.call plus selected Map construct while rejecting overridden call/construct |
| `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardArrayProtocol.java`; `ProtosStandardIdentityMapProtocol.java`; `src/main/java/com/guillermomolina/protos/execution/protocols/numeric/ProtosStandardNumericConversionProtocol.java`; `src/main/java/com/guillermomolina/protos/execution/protocols/io/ProtosStandardTextReaderProtocol.java`; `ProtosStandardTextWriterProtocol.java` | Native factory `call` installers needing migration to ordinary factory `construct` without losing typed state, arity, receiver domain or source-level API |
| `protos/lib/core/Range.protos`; `protos/lib/core/network/{IpAddress,IpEndpoint}.protos`; `protos/lib/collections/{Set,IdentitySet}.protos` | Source-backed factory `call` methods, canonical Range super dispatch, module factory returns different representation |
| `protos/lib/datetime/` (eight factory modules), `protos/lib/logging/` (five), `protos/lib/text/{Style,StyledText}.protos`, `protos/lib/test/Test.protos` | Source-factory inventory: rename only factory definitions; nested executable `Test.call` must remain intact |
| `protos/tests/conformance/call/object-call-and-init.protos`; `surface-sugar/named-arguments.protos`; `library/collections/range-prelude-and-module.protos`; `collections/array-construction-and-identity.protos`; `numeric-conversion/integer-factory-receiver.protos` | Existing conformance and reflective expectations to extend/adapt rather than changing just the implementation |

The inspection is a targeted source audit; no commands, test suites, benchmarks, Maven or Git changes were executed in `guillermomolina/protos`.

## Verified comparative precedents and Truffle source

- **Self:** [2024.1 Handbook glossary](https://handbook.selflanguage.org/2024.1/glossary.html) specifies shallow prototype cloning and separate inherited slots. Different from Protos fresh child with immutable delegation parent.
- **Io:** [language guide](https://iolanguage.com/guide/guide.html) says `clone` creates a child and activates `init`; an analogous two-stage source-oriented pattern, not proof of Io-style multiple parents for Protos.
- **Smalltalk/Pharo:** [Pharo by Example](https://files.pharo.org/books-pdfs/updated-pharo-by-example/2017-01-14-UpdatedPharoByExample.pdf) defines `new` as `basicNew initialize`.
- **Ruby/TruffleRuby:** [Ruby Class.new](https://docs.ruby-lang.org/en/master/Class.html) allocates then initializes. [TruffleRuby ClassNodes.java](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/core/klass/ClassNodes.java) lines 289–303 has separately dispatched `__allocate__` and `initialize` under a specialized Java/Truffle node; physical specialization need not dictate guest semantics.
- **Python/GraalPy:** [Python data model](https://docs.python.org/3/reference/datamodel.html) separates `__new__`/`__init__` and checks whether the new result belongs to the requested class before initialization. [GraalPy TypeBuiltins.java](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/type/TypeBuiltins.java) lines 494–555 implements `tp_call` → `tp_new` → conditional `tp_init`. C1 explicitly **does not** import this type-dependent second-init rule.
- **Dart:** [factory constructor specification](https://dart.dev/language/constructors) permits an existing cached instance or subtype with unchanged call-site syntax but constructor-specific declaration. C1 captures separation without adding Dart's special declaration syntax or class system.
- **JavaScript/GraalJS:** [JSFunction.java](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/builtins/JSFunction.java) lines 291–306 uses distinct call and construct targets; [ECMAScript Reflect.construct](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/construct) reflects the separate internal constructor protocol. C1 **rejects** adding that second universe to Protos.
- **Lua:** [Lua 5.4 manual, `__call`](https://www.lua.org/manual/5.4/manual.html) invokes a metamethod on non-function values, which need not construct an object.
- **Truffle SimpleLanguage:** [SLInvokeNode.java](https://github.com/oracle/graal/blob/master/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/nodes/expression/SLInvokeNode.java) invokes through executable interop, not a comparable default prototype factory.
- **Apple Pkl:** [language reference](https://pkl-lang.org/main/latest/language-reference/index.html) has context-sensitive typed/dynamic `new` objects and a prototype chain for configuration; not directly substitutable for Protos ordinary messages. Espresso/Sulong execute Java/LLVM's existing allocation instructions, not a public prototype-message construction contract.

These are comparative observations, not an approval to copy foreign language semantics, memory models, special built-in hierarchies or optimized code.

## Comparative decision scoring (GITHUB010)

Candidates: **A** existing `call`/`init`; **B** named factory with ordinary-call adapter; **C1** inherited ordinary `construct` hook (selected); **D** `factory:` source sugar for ordinary `call`; **E** distinct call/construct dispatcher. 1–5 means least to most favorable; H/M/L = evidence confidence. Each entry supplies the reason for its assessment, not a performance measurement.

| Criterion | A | B | C1 | D | E |
|---|---|---|---|---|---|
| Invariant correctness | 5H no change | 5H ordinary slots | 3M explicit breaks | 4M alias semantics | 2M parallel dispatch |
| Protos philosophy | 4H one universe | 4H normal messages | 5M normal messages | 2M parser magic | 1M privileged kinds |
| Present proportionality | 5H no cost | 4M adapter boilerplate | 3M extra hook | 2M syntax tooling | 1M two universes |
| Incremental growth | 2M opaque factories | 4M add locally | 5M shared extension | 3M sugar limited | 4L broad hooks |
| Future resilience | 3M coupled intent | 4M escape exists | 5M roles distinct | 3M hidden alias | 4L flexible complex |
| Scalability | 5H proven model | 4M extra sends | 4M one more send | 5M same runtime | 3L more paths |
| Conceptual simplicity | 3H overloaded call | 3M duplicated wrapper | 4M two hooks clear | 2M dual names | 1M classification |
| Portability | 5H VM-free | 5H VM-free | 5M VM-free | 5M syntax-only | 3L runtime category |
| CPU/resources | 5H current | 3L adapter cost | 3L lookup not measured | 5M same path | 2L dispatch complexity |
| Failure/operability | 4H specified | 3M wrapper errors | 4M explicit stages | 2M reflection alias | 2L ambiguity |
| Deferral/migration | 3M debt grows | 4M local migration | 2M public breaks | 3M tooling change | 1L sweeping redesign |
| Evidence/risk | 5H implemented | 4M common technique | 4M Ruby/Io precedent | 3M untested alias | 2L risky split |

**Non-compensating gate:** E fails Protos's small-universe/no-special-kinds requirement and imposes complexity now; D invents a mismatch between syntax and reflectable slots; neither is rescued by future-option scores. A avoids today's implementation risk but retains the confusion. B is compatible but spreads adapters and two names per factory. C1 is the smallest **shared** ordinary-message hook that resolves the demonstrated Range/Date/Set/Test cases while preserving D013. The strongest argument **against C1** is the deliberately introduced general semantic collision with pre-existing `construct` slots and the reflection break for built-in factory `call`.

**Future stress and escape:** no global constructor registry, singleton storage, per-instance dispatch metadata, second reflective hierarchy, inferred type checks or automatic Future awaiting. Source-defined factory caches remain Actor-local; immutable shared Prelude retains no mutable guest-state. Custom factory results and standard init failure propagate under existing Error/return-home/cancellation/ensure rules. Alternative runtimes may specialize the ordinary sends without changing guest semantics. If a later workload exposes unacceptably large lookup cost, use existing Truffle guards/invalidation and measure correct tier/stable workload before proposing a different architecture; preserve the C1 contract.

## Adversarial acceptance matrix

1. `Point(0,0)` creates two distinct children on repeated calls, correct immediate parent, chosen child `init` executed once; discard its return.
2. `object.construct` custom can return a foreign-parent object or preexisting cache identity, even scalar; no hidden default allocation or duplicate `init`.
3. Pre-existing local `construct: 42` shadows inherited standard construct; inherited `Object.call` signals Error; never fall back.
4. Own user `call` remains authoritative, including if it is a non-Closure; no implicit `construct` lookup on a custom-call path.
5. Rebinding `call` or `construct` during left-to-right argument evaluation takes effect at the appropriate **post-argument** lookup point.
6. User-defined named `construct` binds its own signature; actual standard `Object.call` and `Object.construct` both forward exact D237 packets, even when standard method Closure is copied/aliased/extracted.
7. `super.construct` has correct receiver/home and reaches default construction on derived Range; `super.call` must not be used as a surrogate for default construct.
8. Canonical Range opposite-direction parent and same-direction frozen-state behavior; six own slots with `construct` not own `call`, while inherited `Range.call` still works.
9. Array/Map/IdentityMap/numeric conversions/TextReader/TextWriter keep exact receiver-domain checks, native state, copied/inherited factory behavior and error class.
10. `%{...}` rejects noncanonical `call` and noncanonical `construct` before evaluating entries, but accepts the exact standard bridge and Map factory; plain `[]` remains ordinary call lowering.
11. Error/SlotNotFound precedence, extracted method metadata, equality, return-home, `ensure`, cancellation, suspension, Task/Actor/Context and frozen graph invariants remain intact.
12. Bound and native callback callability inspection stays structural on `call`, not on `construct`. Call-only behavior is left to D207.

**Limits:** no product source/spec/test patch executed under D239 ratification, no test success or benchmark speedup asserted. I145 is separately responsible for human-validated implementation and normative publication.
