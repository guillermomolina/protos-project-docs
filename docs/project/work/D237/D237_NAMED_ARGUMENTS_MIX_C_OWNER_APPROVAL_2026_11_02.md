# D237 — Owner-approved named-call syntax and MIX-C argument-binding checkpoint

**Record type:** bounded, non-normative design/work evidence; **not** a complete D237 ratification, product specification change or implementation closure.
**Recorded date:** 2026-11-02.
**Authoritative live design issue:** [guillermomolina/protos#1038](https://github.com/guillermomolina/protos/issues/1038).
**Future implementation issue (blocked):** [I143 #1039](https://github.com/guillermomolina/protos/issues/1039).
**Exact product baseline inspected:** [guillermomolina/protos@2e85e8dc0b85548add6962ef6556f24513bc8d05](https://github.com/guillermomolina/protos/commit/2e85e8dc0b85548add6962ef6556f24513bc8d05).
**Normative owners (unchanged at that revision):** `spec/PROTOS_GRAMMAR.md` §16–19, `spec/semantics/CALLABLES.md` (invocation arguments and ordinary `call`), `spec/semantics/EXECUTION_AND_CONTROL.md`, `OBJECT_MODEL.md`, `ERRORS.md`, `VALUES_AND_COLLECTIONS.md`, concurrency/interop owners where relevant.
**Authority:** specification under `guillermomolina/protos/spec/` remains normative; this record preserves exact owner selections and outstanding decisions only.

## 1. Exact human decision provenance

The project owner had already explicitly selected contextual `f(a: 10)` rather than `.a:`, after being shown that this changes existing slot-creation-argument programs and that `f((a: 10))` preserves the old interpretation: **“si lo acepto, de hecho creo que tiene mas sentido el significado que le queremos dar”**. The bounded selection and compatibility disclosure are recorded in [D237 syntax approval comment](https://github.com/guillermomolina/protos/issues/1038#issuecomment-6578019068).

After comparative GITHUB010 investigation and falsification of four positional/named mixing designs, the agent presented **D237-MIX-C** verbatim as the pending contract: *“¿aprobamos D237-MIX-C, con posiciones fijas, orden textual libre, error por colisión y trailing Closures estrictamente posicionales?”* The project owner responded in the same conversation on **2026-11-02**: **“Perfecto, apruebo el contrato, el git diff check esta limpio. make test y make check pasan en local. Actualiza issues en guillermomolina/protos y agrega evidencias en guillermomolina/protos-project-docs.”**

These are explicit, **bounded owner approvals**. They do not approve hidden details of generic custom forwarding, exact Error class, D130 Array literal behavior, unmodified-source migration of all existing code, or an implementation commit. D237 therefore stays **OPEN** until the full normative package is specified, reconciled, separately approved where necessary and durably ratified.

## 2. Approved syntax invariant and compatibility change

- At the top level of an ordinary call-argument item, directly written `identifier: expression` is a **named call argument** whose identifier is a literal parameter label, not a lexical lookup or slot-creation target. No `.identifier:` marker is required.
- Grouping `f((identifier: expression))` yields an **ordinary positional argument** whose value expression performs normal slot creation. Non-argument slot creation and Closure default declarations retain their existing uses of `:`.
- Previously valid `f(identifier: expression)` that relied on creating a local slot and passing its value changes source meaning. That compatibility break was disclosed and **accepted**. Migration is explicit grouping of the inner expression, not automatic runtime interpretation.
- Parse context and syntax choose the interpretation. No callee-signature, mutable-lookup or type-dependent parser heuristics.

These examples are illustrative expected language behavior; none was executed as an implemented D237 test:

~~~protos
f(a: 10)       // named argument a; no slot creation
f((a: 10))     // positional value of a slot creation
a: 10          // normal slot creation
(a: 10) => a   // default parameter declaration
~~~

## 3. Owner-approved D237-MIX-C contract

1. **Order-independent mixing, stable positional ordinals.** Positional and named items may interleave in **any textual order**. The nth supplied positional value continues to target the nth non-rest formal parameter, independent of the number or location of named items. Named items select by exact declared parameter name. Named items **never shift** positional ordinal assignments.
2. **Evaluation order.** Receiver/target and the supplied item expressions are evaluated **once, left to right**; existing positional Array spread expansion retains its established observable behavior. Name association does not reorder effects; ordinary `call` lookup occurs only after argument evaluation.
3. **Collision and unknown-name failures.** Repeated **explicit syntactic labels** are a source error. Runtime association of a named value with a parameter already receiving a positional value (including one produced by spread) is an error. Unknown labels are an error, **not silently ignored**. Dynamic association validation precedes the corresponding target Closure's defaults/body, without undoing already evaluated caller effects. Exact Error taxonomy is a remaining normative-detail checkpoint.
4. **D203 defaults, omission, `null`, order and flexible arity.** Parameters are still bound **in declaration order**, with defaults executed only for absent parameters in the actual activation/context. A supplied `null` is not absence. Forward references in default expressions do not see unbound later formal slots. Excess **positional** values remain subject to D203's established ignore/rest behavior. Do not introduce parameter predeclaration, hoisting, a second Closure kind or `undefined`.
5. **Existing `...spread` and `...rest` remain exclusively positional.** An Array `...spread` contributes positional values at its source position; a trailing `...rest` formal captures only excess positional suffix values. No `**kwargs`, arbitrary name-rest capture, intrinsic `args` resurrection, implicit Map-to-keyword conversion or automatic name spread was approved.
6. **Trailing Closure is strictly positional.** `f(args...) { body }` remains equivalent to a call with an appended **ordinary last positional Closure**. It does **not** search for a free/callback-typed/named formal, and must still signal normal collision when a named argument already assigned the corresponding formal position.
7. **Ordinary dynamic invocation remains authoritative.** D013's receiver-first evaluation, ordinary overridable `call` lookup, `this`, `methodHome`, extracted methods, NLR/ReturnHome and polymorphic invokability are not replaced by privileged direct-Closure selection. The standard `Object.call` is expected to preserve name association through the Closure execution and instance-`init` pathways, but the *exact* behavior for receiver/Closure layers, errors and custom wrappers must be ratified before implementation. A custom `call: (...args)` does not thereby declare semantic parameters to receive arbitrary names.

### Falsification cases defining what was selected

Given `f: (a: 10, b: 20, c: 30) => [a, b, c]`, the expected **proposed** results under selected MIX-C are:

| Source call | Expected association |
| --- | --- |
| `f(c: 99)` | `[10,20,99]` |
| `f(c: 99, 1)` | `[1,20,99]` — same as `f(1,c:99)` |
| `f(...[1,2], c: 99)` | `[1,2,99]` |
| `f(a: 99, 1)` | Dynamic collision on `a`, **not** `[99,1,30]` |
| `f(1, b: 99, 2)` | Dynamic collision on `b` |
| `f(unknown: 99)` | Unknown label error, not ignored |
| `f(a: 1, a: 2)` | Duplicate label source error |

**Counterexample rejecting “fill unassigned holes”:** for `f: (a,b) => [a,b]`, a holes-first policy would allow `f(1,2,a:99)` to yield `[99,1]` and silently discard `2` under D203 excess-argument tolerance. FIXED positional ordinals plus collision detection rejects this; a late label cannot reorder earlier values.

**Trailing Closure counterexample:** with `run: (options, callback) => callback()`, `run(options: config) { work() }` is a collision under MIX-C because its trailing Closure is positional index 0, already assigned by `options:`. A named explicit `callback: () => { work() }` is the unambiguous form. No Swift-style heuristic is approved.

## 4. GITHUB021 reconciliation — accepted, preserved and still pending

| Prior authority / design invariant | Reconciliation |
| --- | --- |
| D203 #901: flexible positional arity, absent/default/explicit null, left-to-right binding and rest | **Preserved** for positional calls. MIX-C extends only supplied-item association by name, with separately approved collision/unknown-name errors. |
| D013 #171: ordinary polymorphic `call` lookup / `this` / `methodHome` | **Preserved in principle.** Precise standard `Object.call` forwarding and custom-call error boundary remain open. D207 #932 is a separate unsettled reassessment: do not preapprove its alternative. |
| D145 #574: `args` is an ordinary identifier, no ambient introspection intrinsic | **Preserved.** Existing `...rest` cannot transport arbitrary names. Generic named forwarding is explicitly not approved. |
| D130: `[a,...]` is exact `Array(a,...)` ordinary-call sugar and uses `argument-items` grammar | **Unresolved observable interaction.** New `[a:1]` grammar behavior must either follow identical named-call rules or require an expressly approved revision of D130. Do not silently claim equivalence. |
| Contextual if/while/for/return grammar | **Requires detailed precedence audit.** Named-argument syntax must not silently reinterpret a contextual binder/condition/return operand; exact collision cases require explicit resolution. |
| Error and non-local control/suspension/Actor isolation | Preserve existing evaluation order, Error precedence and return-home authority where applicable; **new named-validation error ordering and classification must be specified**. |
| B5 compact frame ABI and Truffle/runtime optimization | Not a semantic commitment to physical representation. Preserve pay-as-you-grow as performance objective; measure versus positional benchmark baseline. |

No owner approval of a complete D237 normative delta was represented. The 2026-11-02 local tests report is **not** an I143 test report.

## 5. Precedent and trade-off summary

The comparative packet considered Python (stable positional indexes/keyword collisions and star expansion), Ruby (positional+keyword separation and explicit forwarding), Kotlin (signature-aware named calls), Dart (declaration-separated named parameters), C# (argument order and named/positional constraints), Swift (trailing Closure association rules) and JavaScript (options objects). These precedents are informative but not normative for dynamic Protos.

Surviving mixing candidates: (A) prohibit mixing; (B) pos-first then named; (C) any order with **fixed positional ordinals**; (D) any order filling “free holes”. MIX-C was selected after the adversarial D203 excess-plus-late-name and trailing-Closure counterexamples. It is more general than (B) without the hidden reindexing of (D). The strongest drawback is that `f(a: 1, 2)` is a collision rather than a compact spelling for supplying `b`; the owner explicitly accepted the MIX-C policy. Detailed 12-axis scoring and confidence remained in the discussion; the compact matrix result was C preferred, D rejected for silently shifted ordinals.

**Pay-as-you-grow:** positional-only calls retain the existing semantic pathway and need no guest-visible name Map or mandatory allocation. Name-bearing invocation may use static label descriptors plus dynamic association, but this is a **hypothesis to test**, not proven performance. Alternative VMs and Actors must not inherit a new privileged invocation universe.

## 6. Work disposition and evidence limitations

- **D237 #1038:** keep OPEN pending exact D130/standard-Object.call/contextual-grammar/error/interop decisions and normative ratification, notwithstanding explicit owner approval of syntax and MIX-C.
- **I143 #1039:** independently tracked **implementation** work, blocked by complete D237 approval/publication; must not implement remaining language policy by accident. Native GitHub dependency relationship was not available through the connected actions when created; textual reference is not a native edge.
- **No Protos product edit, version bump, new conformance run or implementation commit** is claimed by this record.
- Human reported local `git diff --check` clean and `make test` / `make check` PASS on 2026-11-02. Those reports are accepted **as human attestation only**; no exact local tested SHA/dirty-state snapshot was provided. The product baseline named at top is the **GitHub HEAD inspected for design**, not independently proven to be the human's local validation target.
- No CLI/build/test/performance measurement specific to D237-MIX-C has been executed. Subsequent I143 evidence requires its **own** revision-bound human tests and publication.

```text
FORMAL_IDENTIFIER=D237
DECISION_SCOPE=SYNTAX_AND_MIX_C_ONLY
OWNER_APPROVAL=EXPLICIT_2026_11_02
GITHUB021_APPROVED_INVARIANTS=RECONCILED_WITH_PENDING_ADJACENT_DELTAS
PRODUCT_REVISION_INSPECTED=2e85e8dc0b85548add6962ef6556f24513bc8d05
PRODUCT_NORMATIVE_DELTA=NONE
PRODUCT_IMPLEMENTATION=NONE
D237_STATE=OPEN
I143_STATE=BLOCKED
TESTS_HUMAN_REPORTED=make_test_PASS;make_check_PASS;git_diff_check_CLEAN
TESTS_TIED_TO_IMPLEMENTATION_REVISION=NO
REQUIRED_FUTURE_DECISIONS=D130;OBJECT_CALL;CONTEXTUAL_PRECEDENCE;ERRORS;INTEROP
```
