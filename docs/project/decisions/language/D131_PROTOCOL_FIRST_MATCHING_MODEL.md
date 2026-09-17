# D131 — Protocol-first Core v0.1 matching model

Status: **RATIFIED — Candidate C selected**

Approval date: **2026-09-17**
Decision issue: `guillermomolina/protos#503`
Related audit: `AUD009-A1` / `guillermomolina/protos#535`
Exact pre-ratification candidate: `docs/project/work/D131/D131_PROTOCOL_FIRST_BONFIRE_CANDIDATE_CHECKPOINT.md`
Final comparative audit: `docs/project/work/D131/D131_FINAL_COMPARATIVE_AUDIT.md`
Product baseline reviewed before approval: `6ccd8b91ca5446958841db125990e4e0756c3dd0`

Nature: durable non-normative decision/rationale record for normative matching semantics that must be reconciled under `guillermomolina/protos/spec/` and then implemented mechanically.

## Approval provenance

After the exact Candidate C checkpoint, differential GITHUB010 comparison, feature matrix, invariant review, strongest counterarguments, regret/recovery paths, and explicit surfacing of the two least-reversible commitments, the project owner explicitly approved the candidate with:

```text
APROBADO
```

The approval question explicitly included these two commitments:

1. ordinary standard Array and Map values gain structural `match` behavior distinct from their ordinary equality behavior;
2. `value.caseOf(...)` initially uses a normal insertion-ordered Map as the case carrier and deliberately inherits that Map's ordinary hash/equality/unique-key semantics.

No additional semantic choice is bundled into this ratification beyond the exact candidate checkpoint and final comparative audit.

## Ratified foundation

Core v0.1 retains exactly one public recognition authority:

```text
pattern.match(subject)
```

The inherited root behavior remains:

```text
Object.match(subject) -> exactly one ordinary this == subject
```

The normal matcher-result carrier remains exact:

```text
false        -> mismatch
true         -> successful recognition with zero captures
[x, ...]     -> successful recognition with positional captures
```

No second matcher authority, capture sink, callback matcher path, hidden pattern family, or parallel binding ABI is introduced.

## Ratified direct structural matching

### Array

Ordinary standard Array values are structural matchers through their ordinary `match` behavior.

Initial Core v0.1 contract:

- the matcher receiver must own standard Array indexed state; selecting the standard Array behavior for an ineligible receiver signals ordinary invalid-receiver `Error`;
- an ineligible/non-Array subject returns canonical `false`;
- recognition is fixed and exact-length only;
- one shallow observation of matcher Array and subject Array is fixed before any child matcher executes;
- child matchers execute left-to-right through ordinary `child.match(childSubject)`;
- each reached child is invoked exactly once;
- captures compose positionally under the retained matcher-result carrier;
- captured aggregate values are never recursively flattened.

Array remainder/rest is not part of initial Core v0.1.

### Map

Ordinary normal standard Map values are structural matchers through their ordinary `match` behavior.

Initial Core v0.1 contract:

- the matcher receiver must own normal standard Map keyed-entry state; selecting the standard Map behavior for an ineligible receiver signals ordinary invalid-receiver `Error`;
- an ineligible/non-Map subject returns canonical `false`;
- recognition is open/subset only;
- one shallow stable observation of matcher requirements and subject associations is fixed for the attempt;
- matcher requirements are considered in matcher insertion order;
- required subject associations are resolved with the existing normal Map key-search law: ordinary `hash` plus query-side `==` under the standard Map contract;
- a missing required association is canonical `false`;
- unrelated subject associations are ignored;
- required subject association/value references are fixed before nested mapped-value child matching begins;
- mapped-value children execute in requirement order through ordinary `match`;
- captures compose positionally under the retained matcher-result carrier.

Exact Map and Map remainder are not part of initial Core v0.1.

## Ratified initial standard matcher objects

The initial standard helper set is exactly:

```text
Any.match(subject)     -> true
Capture.match(subject) -> [subject]
```

`Any` and `Capture` are ordinary matcher objects in the standard language environment, not keywords and not dedicated pattern syntax.

The following are deliberately not standardized initially:

```text
Or(...)
Guard(...)
Identity(...)
Array remainder combinators
exact Map
Map remainder
```

Their absence preserves future option space; it is not a prohibition on later explicit decisions.

## Ratified ordinary selection surface

The Core v0.1 multi-way selection surface is the ordinary message:

```protos
value.caseOf(cases)
```

`caseOf` is an ordinary selector, not a keyword or dedicated grammar form.

The approved case representation is an ordinary insertion-ordered normal standard Map:

```text
matcher -> callable
```

Its normal Map properties are deliberate semantic consequences of the representation:

- insertion order determines case order;
- matcher keys participate in ordinary normal-Map `hash` / `==` semantics;
- equal/duplicate matcher keys cannot coexist as distinct cases;
- there is no matching-specific parallel identity/equality institution for case keys.

At the beginning of one `caseOf` attempt, the current case associations are observed shallowly in insertion order. Later mutation of the cases Map does not rewrite the already-observed sequence for that attempt.

For each reached case:

```text
result = matcher.match(value)

false        -> continue
true         -> invoke callable()
[captures]   -> invoke callable(...captures)
other normal -> Error
```

The first successful matcher commits. The selected callable's ordinary result is returned unchanged, including `null`.

If no matcher succeeds, `caseOf` signals one fresh ordinary `Error`.

There is no eager whole-table validation institution. Unreached matchers/actions do not fail merely because they would be invalid if reached; ordinary matching and callable behavior is consumed when that case is actually reached/selected.

## Capture naming

Capture names belong to ordinary callable/Closure parameters, not to matching syntax.

For example:

```protos
value.caseOf(%{
    [1, Capture]: x => x
    Any: () => null
})
```

A successful capture carrier is supplied through ordinary call spread and ordinary callable parameter binding. This supersedes the need for a matching-specific selected-arm binding ABI.

## Core v0.1 feature outcome

The complete feature-by-feature rationale and consequences are retained in `D131_FINAL_COMPARATIVE_AUDIT.md`. The ratified direction is:

### KEEP_IN_CORE_V0_1

- `pattern.match(subject)`;
- inherited `Object.match(subject) -> this == subject`;
- exact `false | true | non-empty Array` outcome carrier;
- positional capture composition rules needed by standard composite matchers;
- source-order/first-success multi-way selection semantics, rehomed in ordinary `caseOf`;
- terminal no-selection as fresh ordinary `Error`;
- fixed exact-length standard Array structural recognition, rehomed in `Array.match`;
- standard-Array receiver/subject eligibility distinction and shallow attempt observation;
- open/subset normal standard Map structural recognition, rehomed in `Map.match`;
- normal-Map receiver/subject eligibility distinction, stable association observation, normal Map query law, and deterministic child order;
- ordinary callable invocation/spread as the consumer of positional captures;
- standard `Any` and `Capture` matcher objects.

### REMOVE_FROM_CORE_V0_1

- postfix `subject match { ... }` grammar;
- `case` arm grammar;
- `@name` binder syntax;
- `_` wildcard syntax;
- matching-owned duplicate/linear binding rules;
- current guard capability and `when` / guard-arrow syntax as an initial standard institution;
- dedicated Array-pattern grammar;
- terminal, middle, and bare Array remainder forms and their residual aggregate institution;
- dedicated Map-pattern grammar;
- exact Map mode;
- Map remainder forms and residual aggregate institution;
- alias pattern syntax;
- OR pattern capability / `p1 | p2` as an initial standard institution;
- OR binding-name/interface equivalence machinery;
- fixed and dynamic `captures(...)` source forms;
- D103 complete-arm dynamic-rest terminality machinery;
- matching-specific selected-arm binding ABI;
- dedicated static coverage/exhaustiveness/redundancy framework and pattern-arm structural source-error machinery.

`REMOVE_FROM_CORE_V0_1` means actual despecification/deimplementation in the resulting reconciliation work. It does not mean dormant parser/runtime/test support hidden behind a future-version label.

## Future growth rule

D131 removes the current dedicated pattern language because it is not required by the smallest coherent Core v0.1 model. It does **not** permanently reject future matching capability or syntax.

Future explicit decisions may add ordinary matcher/combinator behavior such as:

```text
Or(...)
Guard(...)
Identity(...)
Array remainder / head-tail recognition
exact Map
Map remainder
```

If repeated real use demonstrates sufficient ergonomic value, future syntax may also be considered, including `|`, binder/wildcard sugar, or compact match/case-like forms. Preferred growth order is ordinary semantic capability first, then optional syntax that lowers to the established protocol rather than recreating an independent pattern institution.

## GITHUB021 invariant/delta consistency check

The final approved candidate preserves every owner-reviewed invariant recorded before ratification:

```text
PRESERVE protocol-first foundation                                      PASS
PRESERVE single pattern.match(subject) recognition authority            PASS
PRESERVE Closure/callable parameters own capture names                  PASS
PRESERVE value.caseOf(...) ordinary selection spelling                  PASS
PRESERVE normal insertion-ordered Map case carrier                      PASS
PRESERVE ordinary Map hash/==/unique-key case semantics                 PASS
PRESERVE no-selection -> fresh ordinary Error                           PASS
PRESERVE selected callable may return null successfully                 PASS
PRESERVE initial helper set exactly Any + Capture                       PASS
PRESERVE ineligible inherited Array/Map match receiver -> Error         PASS
PRESERVE non-Array/non-Map subject -> mismatch                          PASS
PRESERVE shallow matcher/subject/case observation                       PASS
PRESERVE removed features are actually removed from Core v0.1           PASS
PRESERVE future explicit reconsideration remains possible               PASS

WATCHPOINT direct standard Array/Map structural match commitment         EXPLICITLY_APPROVED
WATCHPOINT normal Map as exact initial case representation               EXPLICITLY_APPROVED

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

No hidden semantic delta remains bundled into Candidate C.

## Strongest counterarguments and regret paths

The strongest counterargument is ergonomics: mature matching languages provide concise syntax, guards, OR and remainder patterns immediately, while `caseOf` plus ordinary matcher values is more explicit. D131 intentionally accepts that cost because dedicated sugar can be added later at bounded cost after ordinary protocol use identifies the recurring source patterns worth shortening.

The second counterargument is reversibility: direct structural `Array.match` / `Map.match` and normal-Map `caseOf` are stronger public commitments than separate wrapper matchers/case descriptors. These were surfaced as explicit watchpoints before approval and accepted deliberately.

Recovery paths remain bounded:

- frequent guards -> standardize an ordinary `Guard(...)` matcher, then consider sugar separately;
- frequent OR -> standardize ordered `Or(...)` returning the first successful carrier unchanged, then consider `|` sugar separately;
- frequent head/rest decomposition -> add an explicit Array remainder matcher/combinator with newly decided residual semantics;
- frequent exact Map validation -> add an ordinary exactness matcher/combinator;
- frequent residual Map extraction -> add a Map remainder matcher with explicit residual semantics;
- ordinary source proves too verbose -> design sugar over the ratified protocol rather than restoring the old pattern language by inertia.

## Ordered reconciliation / deimplementation plan

D131 authorizes follow-up implementation work; this ratification publication itself changes no product specification or implementation.

The follow-up must preserve a clearly identified transition until final cutover and end with no accidental compatibility support for removed behavior.

Recommended execution order:

1. **Normative specification cutover**
   - rewrite `spec/semantics/MATCHING.md` around the ratified protocol-first model;
   - reconcile `spec/PROTOS_GRAMMAR.md` and top-level specification references;
   - specify `caseOf`, `Any`, `Capture`, direct `Array.match` and direct `Map.match` exactly from this record;
   - remove/supersede normative authority for rejected D088/D090/D092/D093/D095/D096/D100/D103 institutions as applicable while preserving still-retained D071-D073/D081/D083 semantics and the retained correctness portions of D084/D086.

2. **Add the new ordinary runtime surface**
   - publish standard `Any` and `Capture`;
   - publish standard `caseOf` ordinary behavior;
   - move fixed Array structural recognition to standard `Array.match`;
   - move open/subset Map structural recognition to standard `Map.match`;
   - preserve exact matcher-result validation, ordering, shallow-observation, Error/control, and capture-composition semantics.

3. **Remove dedicated pattern parser / AST / lowering institutions**
   - remove postfix match/case/when syntax acceptance;
   - remove dedicated pattern AST/canonical nodes and matching-specific lowering paths;
   - remove binder/wildcard, remainder, exact Map, alias, OR and `captures(...)` source machinery;
   - remove static coverage/unreachability machinery whose authority depended on the dedicated pattern algebra.

4. **Reconcile tests and conformance**
   - convert retained semantic coverage to direct matcher/`caseOf` tests;
   - delete tests whose only purpose is removed capability;
   - add rejection tests where useful to prove removed syntax is no longer accepted;
   - add receiver eligibility, snapshot mutation, Map key-law, case-carrier uniqueness/order, no-selection Error, null-result, and invalid reached matcher/action coverage.

5. **Reconcile programmer documentation and implementation records**
   - replace the current dedicated matching-expression guide with protocol-first examples;
   - remove obsolete syntax/checklists and coverage promises;
   - document future-growth boundaries without presenting removed forms as dormant features;
   - update implementation status/changelog and all D131/AUD009 cross-references.

6. **Final cutover audit**
   - repository-wide search for removed syntax/institutions;
   - verify no parser/runtime/conformance path preserves rejected behavior accidentally;
   - run focal and full tests under the current project validation policy;
   - record exact specification and implementation revisions that realize D131.

Any follow-up slice that discovers a semantic choice not fixed by this record must stop and route that choice through the normal explicit design-approval process rather than deciding it locally.

## Publication boundary

This ratification record is non-normative governance documentation. At publication time:

```text
D131_STATUS=RATIFIED
SELECTED_CANDIDATE=C_PROTOCOL_FIRST_BONFIRE
SPECIFICATION_CHANGED=NO
IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
VALIDATION_CLASS=GOVERNANCE_DOCUMENTATION_ONLY
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

Normative and executable changes begin only in the ordered reconciliation work above.