# PLAT060 — Universal five-slot ordinary Closure-call ABI (B5)

**Status:** RATIFIED — I102 implementation/validation in progress; see the 2026-10-15 approved caller-slot clarification below. **Owner approval:** 2026-10-15, explicit statement “ok apruebo B5” following the B5/B6/C reevaluation and explicit earlier rejection of different NORMAL/DEBUG ABIs. **Issue:** [PLAT060/#895](https://github.com/guillermomolina/protos/issues/895). **Product source audited:** [guillermomolina/protos@df4142919d0222635ab8f9f8c54e7a1966e6edaa](https://github.com/guillermomolina/protos/commit/df4142919d0222635ab8f9f8c54e7a1966e6edaa). **Project-record baseline:** `guillermomolina/protos-project-docs@932e7b2042e58b3fd2121acfc3af2d310e5e23b0`. **Evidence:** [PLAT060 B5 research, falsifiers, scoring and GITHUB021 approval reconciliation](../../evidence/PLAT060/PLAT060_B5_OWNER_APPROVAL_INVARIANT_RECONCILIATION_2026_10_15.md).

## Ratified decision (exact scope)

**One single, stable, ordinary physical `Object[]` CallTarget argument ABI for Protos Closure execution, identical in NORMAL and DEBUG.** Use the existing **B5 full-header five-slot model** as the minimal migration target:

```text
0: invoked Closure, or the same invocation's published ProtosActivation
1: receiver for an immediate method call, or DIRECT_CLOSURE_CALL marker
2: methodHome for immediate method; owning Task or null for direct Closure call
3: exact caller in unpublished-Closure state; retained caller or null in published-activation state (see approved addendum below)
4: exact per-invocation returnHome (including existing proven-unobservable markers)
5...: supplied positional arguments (arity = array.length - 5)
```

This is one physical positional contract. Slots 1 and 2 are a tagged union **within the same fixed ABI**, not distinct layouts. The direct-call marker is private and cannot be a guest receiver. A published rich activation may replace slot 0 of that *same* array; this is a **state transition, not a second entry ABI**. An already-materialized ordinary Closure entry must also obey the B5 contract; reconstructing the remaining fields must preserve actual provenance, receiver/home/Task/return identity without inventing a new allocation merely to satisfy the header.

**Eliminate as ordinary entry ABIs:** `MINIMAL_A/B/C/D`, short immediate-method forms, NORMAL-only empty `Object[0]` host entry (`headerless`), and the rich ordinary `[activation]` single-element argument layout. All ordinary host, guest, method, direct Closure, Task-owned and source-backed interoperability entries must converge. No arity-, task-, return-home-, or NORMAL/DEBUG-dependent header lengths. Special internal continuation/resumption and truly non-Closure roots are separate semantic contracts and must be assessed, not blindly retyped; no second ordinary Closure ABI is permitted.

**Retain:** lazy `ProtosActivation`/guest `context`; exact one activation identity per invocation when observed; frame-native lexical locals and by-reference capture; parameter count/read directly from offset 5 before activation materialization; Call0/Call1/N call-site specializations and no intermediate supplied-array on their fast paths; guarded dispatch/PIC, preselected arity-rejection root, return-home/unobservable/primitive-result guards; physically inline callback execution that creates no separate CallTarget frame; conditional RootTask/Actor/Process; and distinct immutable Context-owned NORMAL/DEBUG **lowering/instrumentation** profiles.

**Profile invariant:** NORMAL and DEBUG have *exactly the same* ABI. NORMAL may emit a smaller root and omit DEBUG-only instructions, scopes, tags, publication and materialization when the established proofs permit it. DEBUG emits the provenance and tooling behavior required for late debugger attachment within DEBUG contexts. `BytecodeConfig` updates cannot switch the semantic compilation profile. Replacing the `headerless` scope check with a profile/tooling-policy check does not authorize late debugging of NORMAL contexts.

**Explicit existing-decision reconciliation:** PLAT040 F′ (guarded calls and lazy guest state) remains; its ratification did **not** require the later four short layout encodings. PLAT058-D physical NORMAL/DEBUG emission specialization remains intact. The **physical** PERF032 NORMAL `headerless` exception and tests assuming a zero-length frame are explicitly superseded by this owner-approved B5 decision; historical evidence stays intact. PLAT036 frame-owned lexical state and D179 mutable Context slot presence/removal remain normative; PLAT044/057 inline semantic/callback rules remain; PLAT046 pay-as-you-grow remains. There is no change to Core language semantics or `spec/` in this ratification.

## Non-goals and stop conditions

- No eager universal activation, `context`, `RootTask`, `RootActor`, `RootProcess`, synthetic `FrameDescriptor`, universal context-copy or rich `Arguments` aggregate.
- No B6 six-slot expansion, C three/four-slot packing, NORMAL/DEBUG split ABI, retained host-only ABI exception, or alternative compact Call0. Those are **not approved**.
- Do not infer that a five-slot array is performance-free after Graal inlining or that B5 measured faster. The issue is an architectural simplification with a bounded compiler regression gate.
- Do not modify semantics or invent a new rule to accommodate migration. If one ordinary source-backed path cannot honor B5 with exact return/caller/receiver/task/activation identity, stop and raise the concrete conflict instead of silently adding another ABI.
- Non-Closure internal frames (continuations, module infrastructure) and physical inline callbacks do not become ordinary Closure calls by fiat. Classify each case against its true execution contract.
- `pom.xml` and `CHANGELOG.md` product implementation-version changes are human-executed **after** reported green tests and immediately before publication; no further tests after those metadata edits.

## Implementation and empirical acceptance

Create independent **I-family** implementation ownership. It is authorized to realize *this exact approved architecture*, not to reopen C/B6. Group the representation, all producers/consumers, host and tooling migration, and regression coverage in cohesive publishable slices using the current product HEAD. Changes to product source/test/benchmarks, test execution and product Git publication remain human-executor gated by `guillermomolina/protos/AGENTS.md`.

Required acceptance: all ordinary Closure entry types exhibit B5 in both immutable profiles; arity and argument ordering/precedence remain exact; direct and receiver-bound extracted methods retain identity; Task inheritance, captured state, exact return home and NLR, cancellation/ensure/yield/resume work; interop and multi-Context isolation hold; DEBUG late attachment exposes scope; NORMAL omits unnecessary emitted debugging and rejects unsupported scope requests by policy instead of array length; no mandatory activation or Task; independent handwritten Bytecode DSL Builder oracles retained per PLAT058-D. Compare identical-workload pre/post Tier-1/Tier-2 graph stages, partial escape, materialized allocations, inlining and stable timing where available without changing benchmark stabilization policy. A performance regression does not grant license to recreate a forbidden ABI variant.

**Status boundary:** owner-approved architecture RATIFIED; **not** proof of implementation, green tests, Graal speedup, Native Image portability or completed migration. GitHub PLAT060 owns decision closure only; the I-family item owns migration and product acceptance.

## Approved 2026-10-15 clarification — published caller slot

Following explicit owner approval in I102-B, physical B5 slot 3 must contain the exact non-null caller while slot 0 holds an unpublished Closure. After activation publication, slot 3 retains the exact caller if it was carried by the original compact call; an already-published activation entered through the rich path may use null in slot 3 when no exact dynamic caller was retained. Published-state consumers must use the activation for provenance, not slot 3. The header remains five positions, identical in NORMAL and DEBUG, and the argument offset remains five. See [approved invariant and evidence](../../evidence/PLAT060/PLAT060_B5_PUBLISHED_CALLER_SLOT_APPROVAL_2026_10_15.md).


## 2026-10-18 — Explicit owner-authorized B5+ physical caller-reference amendment (PLAT066-C)

The project owner approved converging `InvokeClosureOneTail` toward GraalPy's lazy caller-state preparation and subsequently clarified: **"D013, lo dejamos para mas adelante por si hace falta, y el resto ajustar lo que haga falta."** This expressly authorizes adjusting platform choices needed for that mechanism, while leaving the D013 semantic decision untouched. The exact owner/source-evidence record is [PLAT066-C](PLAT066_C_GRAALPY_SHAPED_TERMINAL_CLOSURE_CALL_CONVERGENCE.md), with [source and dependency audit](../../evidence/PLAT066/PLAT066_C_SOURCE_DIFF_OWNER_APPROVAL_2026_10_18.md), implemented separately by [I115/#931](https://github.com/guillermomolina/protos/issues/931).

**Amended physical ABI:** retain one and the same five-header-slot `Object[]` layout `[closure|published Activation, receiver|DIRECT tag, methodHome|Task/null, caller, returnHome, supplied...]`, supplied offset 5, no profile-specific alternative. For an *eligible admitted direct terminal Closure invocation*, the unpublished slot-3 caller carrier may be either its existing exact `ProtosActivation` **or a deferred reference to that same caller's B5 argument array**, lazily resolved by the shared frame-argument authority to the exact same published `ProtosActivation` when needed. This is a backend representation extension, not a second guest caller identity, a different calling convention, or a change in D013, D179, ReturnHome, C′, Task or DEBUG behavior.

**Non-negotiable consumer cutover:** update all readers currently assuming `(ProtosActivation)arguments[3]` (including compact activation materialization, header validation, provenance, Task, debugging, lexical capture/deferred owner) before enabling the new producer. Preserve exact reference equality of the eventual published activation, capture-by-reference, proper fresh context, escalation on actual observation, precise errors/cancellation and identical NORMAL/DEBUG semantic results. A purported lazy caller transport that eagerly resolves on every child call fails the PAYG purpose. Other B5 producers continue to accept the original rich caller type.

**GITHUB021 delta:** The previous *physical* requirement that slot 3 always contain a rich `ProtosActivation` on unpublished entries is superseded only for the admitted direct terminal lane. The invariants of **one universal B5 ABI, fixed positions, exact semantic caller and fresh activation identity, same NORMAL/DEBUG behavior, by-reference contexts, continuation and ReturnHome correctness** remain ratified. No new normative semantics or general automatic D013 bypass. I102/#896 was already COMPLETED/CLOSED at `guillermomolina/protos@dc69fd6269a73e0510e90eb0743514eb63e4630a` (`0.3.361-SNAPSHOT`), with maintainer-reported `make test` PASS. The B5+ extension is new I115 work and does not reopen or invalidate I102's historical B5 closure.

**Read-only inspected product:** `guillermomolina/protos@fcc5c405413d70df546c5a9e5fa2c57b78c8ec91`. Ratification is not proof that the extension is implemented, validated or faster.