# D140 — Horizontal object composition / traits model

Status: **RATIFIED — Candidate A′ selected**

Approval date: **2026-09-18**
Decision issue: `guillermomolina/protos#568`
Triggering audit: `AUD009-A2` / `guillermomolina/protos#561`
Protos baseline reviewed before approval: `c0a698b0aeffea58bdaa6c50cb9917503659d770`

Nature: durable non-normative decision/rationale record for the Core v0.1
horizontal object-composition model. Observable Protos semantics remain owned by
the applicable normative material under `guillermomolina/protos/spec/`.

## Approval provenance

AUD009-A2 first established an explicit owner invariant:

> Protos must retain horizontal multi-source reuse / trait-like composition as a
> language capability now.

D140 then reopened only the realization of that capability. After the
comparative research, adversarial review, scoring, deliberate limits, and exact
Candidate A′ packet were presented, the project owner explicitly approved:

```text
aprobado A'
```

No additional semantic choice is bundled into this ratification.

## Ratified model

Core v0.1 keeps horizontal multi-source reuse as **ordinary-object structural
composition**.

The retained surface is contextual object-body composition:

```protos
duck: animal {
    ...flyable
    ...swimmable
}
```

A composition source is an ordinary Protos object. There is no distinct
`Trait` value kind, trait declaration, trait identity, trait inheritance graph,
trait-private state namespace, method/data slot distinction, or trait-specific
method-resolution order.

For one reached composition item:

1. evaluate the source expression according to ordinary Protos evaluation;
2. inspect the resulting source object's local slots;
3. exclude names structurally reserved by direct local declarations in the
   receiving object body;
4. validate the complete effective contribution set against the receiver's
   current local structure;
5. if any effective contribution conflicts, signal the composition error without
   installing a partial subset from that item;
6. otherwise install every effective contribution as a local slot of the
   receiving object.

Composition therefore remains shallow structural flattening. It copies local
slot bindings, not object graphs. Stored values keep their ordinary identity and
aliasing semantics.

After successful composition the contributed bindings are ordinary local slots
of the receiver. The receiver's delegation parent is unchanged and no continuing
runtime relationship to the source is created.

## Uniform slot participation

Every local source slot participates under the same rule regardless of whether
its stored value is a Closure, immutable data, mutable state, another object,
`null`, or another Protos value.

D140 deliberately rejects a composition-only distinction between "method slots"
and "state slots". The structural concept remains the ordinary slot.

## Local declaration precedence

Direct local slot declarations in the receiving object body retain structural
precedence over composed sources. Their names are reserved for those declarations
independently of textual position.

The reservation is not a binding and does not make the later declaration visible
before ordinary left-to-right execution reaches it. Its role is only to exclude
that name from composition contributions during construction.

This preserves the trait property that explicit composer-local glue wins without
turning source order into an implicit winner-selection rule.

## Source-source conflicts

When two composition sources would contribute the same non-reserved local name,
source order does not select a winner. The later reached conflicting composition
item signals a composition conflict.

Evaluation itself remains left-to-right. The order-independence rule concerns
winner selection, not source-expression effects.

## Per-item structural atomicity

Each composition item remains atomic with respect to structural changes to the
receiver.

After the source expression has evaluated, the complete effective contribution
set is validated before any of those slots is installed. If one effective
contribution conflicts, none from that composition item is installed.

Effects that occurred while evaluating the source expression are not rolled back.
This is a bounded construction invariant, not a general transaction system.

## Closure, receiver, and `super`

Composition does not clone or rewrite Closure values.

A contributed Closure later follows the ordinary callable rules:

- the dynamic receiver is the final receiving object;
- receiver-bound extraction follows the normal Closure/extraction semantics;
- `methodHome` follows the ordinary lookup origin of the new local slot;
- `super` continues through the receiver's ordinary single delegation chain;
- there is no trait-specific `super`, trait ancestry, or multiple-parent resend
  chain.

Closures remain the single executable value kind.

## `without` and `alias`

`without(name)` and `alias(sourceName, aliasName)` remain ordinary structural
Object operations rather than dedicated trait syntax or a parallel trait algebra.

The retained boundary is important:

- `without` removes one local name from the resulting structural view;
- `alias` adds another local name;
- `alias` does not remove the original name;
- neither operation rewrites Closure bodies, lexical references, receiver
  semantics, or method metadata.

Structural aliasing is not source rewriting.

## Deliberate limit: independent homonymous state

If two composition sources both define a local slot named `count` and their
Closures refer to the ordinary name `count`, the ratified model does not create
two hidden private trait-state cells.

The receiving object may deliberately provide one local `count`, thereby
resolving the collision and sharing that state among the composed behavior.

If the intended design requires two independent same-named private state cells,
current ordinary-object composition does not provide that hidden namespace.
`alias` cannot transparently solve it because aliasing a slot does not rewrite
bare-name references inside already-created Closures.

This is a deliberate Core v0.1 boundary, not an unspecified accident.

## Deliberate limit: repeated common origin

Ordinary-object flattening retains no invisible trait provenance.

If two later composition sources independently contain same-named local slots
that historically came from a common earlier source, a new composition treats
those slots as ordinary same-named contributions. Core v0.1 has no automatic
"same original trait contribution" diamond exception.

Adding such an exception would require provenance, a privileged Trait identity
or graph, a method/data distinction, or another explicit mechanism. D140 does
not install that machinery speculatively.

## Reconsideration triggers

Reopen richer composition only if real Protos library or Tool code demonstrates
recurring material need for either:

```text
INDEPENDENT_HOMONYMOUS_TRAIT_STATE
COMMON_ORIGIN_NESTED_COMPOSITION_CONFLICT_SUPPRESSION
```

A future decision must start from then-current evidence and need. It must not
assume that hidden provenance or stateful Traits are automatically the right
solution.

## Comparative research summary

D140 compared materially different approaches rather than treating the current
implementation as authority.

- Classic Smalltalk Traits: flattening, composer-local precedence, explicit
  conflict resolution, exclusion and aliasing.
- Stateful Traits: trait-local state plus state mapping/merge; strong evidence
  that the independent-state problem is real, but also evidence of the extra
  semantic machinery required.
- Self and Io: multiple-parent/prototype lookup; useful prototype-language
  precedent but incompatible with Protos's single-parent invariant.
- Ruby modules and Scala traits: ordered ancestry/linearization participates in
  precedence and `super`, introducing an additional lookup-order institution.
- Go embedding: preserves an embedded subobject and receiver relationship rather
  than flattening behavior into the outer object's ordinary slots.
- Rust traits: primarily a nominal/static interface and implementation contract,
  substantially larger than the Protos horizontal-reuse requirement.
- JavaScript-style sequential object copying: mechanically small, but source
  order becomes last-writer-wins precedence.

Primary references considered include the original Traits work, Stateful Traits,
the Self Handbook, Io documentation, Ruby documentation, Scala trait design
material, the Go specification, the Rust Reference, and ECMAScript object-copy
semantics.

## Final candidate comparison

Three candidate families survived to final review:

- **A′ — ordinary-object uniform structural flattening**;
- **B — generalized structural composition protocol** with broader dynamic
  structural mutation/transaction machinery;
- **C — first-class/stateful Trait model** with explicit
  identity/state/provenance/composition algebra.

Required 1–5 scoring, with confidence H/M:

| Dimension | A′ | B | C |
| --- | ---: | ---: | ---: |
| Correctness / invariant preservation | 5/H | 4/M | 5/M |
| Protos alignment | 5/H | 4/M | 2/H |
| Present-need proportionality | 4/H | 2/M | 2/H |
| Incremental growth | 4/M | 5/M | 5/H |
| Future-option resilience | 4/M | 5/M | 5/H |
| Scalability | 4/M | 4/M | 5/M |
| Conceptual simplicity | 4/H | 3/M | 2/H |
| Portability / implementation freedom | 5/H | 5/H | 4/H |
| Runtime / resource cost | 5/H | 3/M | 3/M |
| Failure / operability | 5/H | 3/M | 4/M |
| Deferral / reversibility / migration | 4/M | 3/M | 2/M |
| Evidence maturity / implementation risk | 4/M | 3/M | 3/M |

The arithmetic total was not used as decision authority.

Candidate B was rejected because it requires a broader dynamic
structural-mutation and transaction surface than the present need justifies.
Candidate C was rejected because it preimplements provenance/private-state
machinery before concrete Protos code demonstrates that requirement.

Order-sensitive copying, MRO/linearization, and multiple-parent delegation were
also rejected because they replace explicit conflict semantics with implicit
precedence or contradict the single-parent invariant.

## Anti-overengineering and growth check

A′ is the smallest sufficient current solution:

```text
ordinary object source
+ shallow local-slot flattening
+ local declaration precedence
+ explicit source-source conflict
+ per-item structural atomicity
+ ordinary without/alias views
```

Programs that do not use composition do not pay for Trait identity, provenance,
private state, MRO, multiple inheritance, or structural transactions.

A later richer layer remains possible if real code establishes the need. That is
future-compatible rather than future-preimplemented.

## Strongest counterargument and regret path

The strongest counterargument is expected Tool growth: deep compositions could
make common-origin conflicts noisy, and reusable components could need independent
same-named state.

A stateful/provenance-aware Trait abstraction could solve those cases earlier.

D140 accepts that future migration risk because solving it now requires exactly
the kinds of 0-to-1 semantic categories Protos avoids introducing without
evidence. The two concrete reconsideration triggers above preserve a clear
regret/recovery path.

## GITHUB021 invariant/delta consistency check

The exact approved Candidate A′ was checked against the owner-approved D140
invariants before durable ratification:

```text
PRESERVE horizontal composition capability remains available now              PASS
PRESERVE prototype/delegation model; no classes                                PASS
PRESERVE exactly one ordinary delegation parent                                PASS
PRESERVE ordinary objects/messages over a privileged Trait institution         PASS
PRESERVE reads may delegate; ordinary writes do not mutate ancestors            PASS
PRESERVE Closures remain the single executable value kind                       PASS
PRESERVE implementation convenience is not semantic authority                   PASS
PRESERVE historical implementation effort is sunk cost                          PASS

RETAIN contextual ...source ordinary-object composition                         EXPLICITLY_APPROVED
RETAIN uniform local-slot flattening                                             EXPLICITLY_APPROVED
RETAIN structural local-declaration precedence / reservation                     EXPLICITLY_APPROVED
RETAIN explicit source-source conflicts; no source-order winner                  EXPLICITLY_APPROVED
RETAIN per-item structural atomicity after source evaluation                     EXPLICITLY_APPROVED
RETAIN ordinary without / alias structural views                                EXPLICITLY_APPROVED
RETAIN alias adds a name and does not rewrite Closure bodies                     EXPLICITLY_APPROVED

NEW Trait identity/category                                                      NO
NEW trait-private state namespace                                                NO
NEW provenance/common-origin diamond exception                                  NO
NEW method/data slot distinction                                                 NO
NEW multiple-parent lookup or MRO                                                NO
NEW generic dynamic structural-mutation protocol                                NO

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

No hidden semantic delta remains bundled into A′.

## Specification and implementation consequence

D140 was opened to re-evaluate an already specified and implemented composition
model. Candidate A′ retains that model's observable semantic core.

At this ratification boundary:

```text
SPECIFICATION_CHANGED=NO
IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
```

Current normative owners remain authoritative:

- `spec/semantics/OBJECT_MODEL.md`;
- `spec/semantics/CALLABLES.md`;
- `spec/PROTOS_GRAMMAR.md`.

A later audit may improve wording or discover an actual specification
contradiction, but this durable record does not independently change language
semantics.

## Relationship to AUD009-A2

AUD009-A2 may now classify:

```text
HORIZONTAL_COMPOSITION_CAPABILITY = KEEP
SELECTED_MODEL = D140_A_PRIME_ORDINARY_OBJECT_UNIFORM_FLATTENING
CURRENT_REALIZATION = KEEP
```

The composition question is therefore closed for A2 unless later audit evidence
exposes a contradiction requiring an explicit D140 reopening.

## Publication boundary

This publication changes only durable non-normative project documentation in
`guillermomolina/protos-project-docs`.

```text
D140_STATUS=RATIFIED
SELECTED_CANDIDATE=A_PRIME_ORDINARY_OBJECT_UNIFORM_FLATTENING
SPECIFICATION_CHANGED=NO
IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
VALIDATION_CLASS=GOVERNANCE_DOCUMENTATION_ONLY
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
REQUIRED_DURABLE_PUBLICATION=THIS_FILE
```

The exact `PROJECT_RECORD_REVISION` is recorded in the D140 GitHub Issue closure
comment after publication and re-read.
