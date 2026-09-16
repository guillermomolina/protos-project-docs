# AUD009-A1 — Matching surface complexity and necessity review

Status: **IN_PROGRESS**

Nature: non-normative retrospective audit evidence

Parent: `AUD009` / `guillermomolina/protos#522`

Semantic decision authority: `D131` / `guillermomolina/protos#503`

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

Specification changed: **NO**

Implementation changed: **NO**

## Purpose

Audit the current Core v0.1 matching surface under the AUD008/AUD009 retrospective methodology and produce evidence-backed proposed classifications without selecting replacement semantics inside AUD009.

D131 remains the semantic decision authority. AUD009-A1 supplies inventory, complexity/necessity evidence, comparative evidence, proposed AUD009 classifications, and follow-up routing.

## Governing outcomes

Every reviewed mechanism will receive one proposed AUD009 outcome:

- `KEEP`
- `REMOVE_NOW_RECONSIDER_LATER`
- `REMOVE_PERMANENTLY`

These are proposals until the project owner explicitly approves the relevant substantive decision through D131 or another required authority.

For `REMOVE_NOW_RECONSIDER_LATER`, record `RECONSIDERATION_TRIGGER` and `RECONSIDERATION_SCOPE` under the approved AUD009 routing contract.

For `REMOVE_PERMANENTLY`, record the rejection rationale and the mechanism/direction that should own related future requirements.

## Initial surface inventory

The audit must cover at least:

1. outer postfix expression-valued `match` / `case` envelope;
2. source-order arm selection and terminal no-selection behavior;
3. ordinary value patterns / `Object.match(subject)`;
4. arbitrary user-defined matcher objects through `pattern.match(subject)`;
5. positional matcher result carrier (`false`, `true`, non-empty Array captures);
6. `@name` binder;
7. `_` wildcard/discard;
8. guards and strict-Boolean guard semantics;
9. guard/body `=>` delimiter rule;
10. fixed Array patterns;
11. terminal Array remainder;
12. middle Array remainder;
13. bare Array remainder discard;
14. Array shallow observation/fresh frozen captured remainder semantics;
15. open/subset Map patterns;
16. exact Map mode;
17. Map remainder capture;
18. bare Map remainder discard;
19. Map snapshot/fresh frozen remainder semantics;
20. alias patterns;
21. OR patterns and first-success commitment;
22. OR binding-interface equivalence;
23. opaque matcher `captures(...)` fixed bindings;
24. opaque matcher dynamic `captures(...rest)`;
25. complete-arm dynamic-rest terminality;
26. static tri-state exhaustiveness/redundancy framework;
27. structural unreachability errors and warning/lint scope.

## Evidence dimensions

For every mechanism record, where applicable:

- current real use / actual consumers;
- programmer/user cognitive cost;
- public syntax/semantic/API surface;
- parser/grammar cost;
- canonical/lowering/runtime cost;
- static-validation/tooling cost;
- test/conformance maintenance cost;
- interaction cost with other match features;
- runtime/resource cost;
- robustness/correctness value;
- pay-for-what-you-need;
- grow-as-you-need;
- concrete deferral/reintroduction cost;
- future-option resilience;
- scalability;
- Protos-philosophy alignment;
- compatibility/migration consequences;
- confidence.

Historical implementation effort is sunk cost and is not itself evidence for KEEP or removal.

## Initial repository observations

Current normative grammar exposes OR, aliases, fixed/dynamic opaque capture interfaces, Array remainder at arbitrary prefix/suffix boundaries, open/exact Map patterns, Map remainder and complete logical binding-interface restrictions.

Current implementation evidence already confirms that `captures(...)` spans parser handling, Bytecode lowering, parser tests and conformance interactions with OR patterns, guards, Array patterns and Map patterns. AUD009-A1 must therefore measure cross-feature interaction complexity rather than evaluating each syntax form only in isolation.

## Comparative research requirement

Consume D131's required comparison set and use primary language documentation/specifications where practical. At minimum compare relevant pattern-matching properties from Rust, Scala 3, Python, C#, Java, Swift, Kotlin, Dart, Ruby, Erlang/Elixir, OCaml/F#, Haskell and useful Racket/Clojure evidence, with Smalltalk/Self/Io as philosophy/control-flow contrasts where materially comparable.

The comparison is evidence, not authority. Protos is not required to imitate prevalence elsewhere.

## Deliverable matrix

The final decision packet must include at least:

```text
FEATURE
CURRENT_ROLE
CURRENT_REAL_USE
PUBLIC_COMPLEXITY
IMPLEMENTATION_COMPLEXITY
INTERACTION_COMPLEXITY
CURRENT_BENEFIT
COST_OF_DEFERRAL
FUTURE_RESILIENCE
PROTOS_PHILOSOPHY
PROPOSED_OUTCOME
CONFIDENCE
FOLLOW_UP_OWNER
RECONSIDERATION_TRIGGER / RECONSIDERATION_SCOPE (when applicable)
REJECTION_RATIONALE / SUPERSEDING_DIRECTION (when applicable)
```

## Boundary

AUD009-A1 does not change `spec/**`, parser/runtime behavior, tests, guide semantics, D131 state, or any prior Dxxx decision. It stops at the complete evidence-backed proposal packet for explicit project-owner selection.
