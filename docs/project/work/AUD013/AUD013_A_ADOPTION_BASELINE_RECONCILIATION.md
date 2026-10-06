# AUD013-A — Adoption-baseline reconciliation

## Status

```text
WORK_ITEM=AUD013/#581
SLICE=AUD013-A
TYPE=INVESTIGATION
RESULT=PASS
SOURCE_SWEEP_EXECUTED=NO
PRODUCT_REPOSITORY_MODIFIED=NO
```

AUD013-A reconciles the owner-declared activation baseline against subsequent
language, library, test, tooling, documentation, platform and audit work before
the first repository-wide source sweep. It is a non-normative audit record and
does not create Protos semantics.

## Activation baseline and investigation head

The historical AUD013 activation anchor remains:

```text
ACTIVATION_BASELINE=80cba12d21647009ece1f9f9a1348c55289766f2
BASELINE_PROTOS_FILES=1690
LANGUAGE_CAPABILITY_SET=OWNER_DECLARED_SETTLED_FOR_THIS_AUDIT
SOURCE_SWEEP_STARTED=NO
```

AUD013-A was reconciled through the then-current Protos head:

```text
RECONCILIATION_HEAD=62f3f5710210f247aad8574d3d0d56d3254bfd70
RECONCILIATION_HEAD_SUBJECT=LIB010-E2-A: make TOML temporal fraction encoding linear
```

The movement of `main` does not replace the historical activation anchor. The
purpose of this slice is to classify post-baseline evolution against that anchor,
not silently rebaseline the audit.

The explicit implementation blockers recorded by AUD013 are closed:

```text
D142/I049=IMPLEMENTED
I049_PUBLICATION=0b6dd12a76818c2490fbe93253daf5a0429808e6

D143/I050=IMPLEMENTED
I050_PUBLICATION=9ffe1e3e75663cac8f262d9f1d9f24185ba9dcf0

AUD013_EXPLICIT_IMPLEMENTATION_BLOCKERS=CLEARED
```

## Reconciliation rules

An adoption obligation survives into the first source sweep only when:

1. an authoritative owner has already settled the capability/rule;
2. maintained source can still contain a semantically classifiable older form;
3. adopting the current form does not require AUD013 to invent semantics; and
4. the owning work has not already performed the repository-wide cutover.

Post-baseline owner work that already completed its own cutover is recorded as
reconciled, not scheduled again.

## Active adoption manifest

### A013-ASSERT-REQ — Boolean assertion duplication

```text
OWNER=TEST003/#562 + TEST003-A/#563
API_OWNER=LIB016/#557
CURRENT_FORM=Assertions.require(condition)
CLASS=REQUIRES_PER_OCCURRENCE_SEMANTIC_CLASSIFICATION
```

Candidate legacy forms are local `require` helpers or equivalent Boolean checks
whose complete relevant contract is "canonical Boolean true succeeds; false is
an assertion failure".

Retain helpers when they encode final-value expectations, manifest/TestPlan
semantics, exact Error identity/category, state, cleanup, bookkeeping or any
other behavior beyond the ratified Assertions contract.

TEST003-D/#566 owns prevention of newly introduced duplicated assertion helpers;
AUD013 consumes that guard rather than creating another prevention institution.

### A013-ASSERT-SIG — expected-error assertion duplication

```text
OWNER=TEST003/#562 + TEST003-A/#563
API_OWNER=LIB016/#557
CURRENT_FORM=Assertions.signals(errorPrototype, body)
CLASS=REQUIRES_PER_OCCURRENCE_SEMANTIC_CLASSIFICATION
```

Candidates include `Error.handle`/flag/`reject` constructions used only to
prove that a body signals the expected Error family. Exact caught-object
identity, custom classification, cleanup, state transitions or a meaningful
Boolean return disqualify mechanical migration.

### A013-MAP-ABSENCE — lazy expected-absence lookup

```text
OWNER=D142/#572 + I049/#598
CURRENT_FORM=Map.atIfAbsent(key, fallback)
IDENTITY_MAP_FORM=IdentityMap.atIfAbsent(key, fallback)
CLASS=REQUIRES_PER_OCCURRENCE_SEMANTIC_CLASSIFICATION
```

Migration is permitted only when it preserves one logical lookup, lazy fallback
invocation on absence, exact present values including `null` and `false`, the
existing key law and non-mutating behavior.

Presence queries, duplicate validation, get-or-insert behavior, branches whose
presence itself is observable, and mutating logic remain distinct.

### A013-MULTISLOT — fixed-prefix Array multiple-slot creation

```text
OWNER=D143/#573 + I050/#599
CURRENT_FORM=(a, b): source
CLASS=REQUIRES_PER_OCCURRENCE_SEMANTIC_CLASSIFICATION
```

Candidates are two or more consecutive local slot creations from fixed initial
positions of the same standard Array.

Migration must preserve one RHS evaluation, standard-Array eligibility,
fixed-prefix observation, current-context ordinary slot creation, left-to-right
creation and the published non-transactional failure boundary.

Sparse/reordered indexing, assignment, accessor helpers, assertions and
structures requiring rest/nesting/Map/object destructuring are not D143
adoption.

Representative current-source candidates observed during reconciliation include
fixed-prefix extraction in `protos/lib/crypto/SHA256.protos` and
`protos/tools/test/Manifest.protos`.

### A013-RECOGNIZES — exact semantic-family validation

```text
OWNER=D147/#577 + I045/#584
CURRENT_FORMS=String.recognizes,Integer.recognizes,Float.recognizes,Array.recognizes
CLASS=REQUIRES_PER_OCCURRENCE_SEMANTIC_CLASSIFICATION
```

Only a discarded operation whose sole purpose is exact family validation may be
replaced. The caller's existing domain-specific failure behavior must be
preserved.

The recognized domains remain exactly those approved by D147/I045. AUD013 must
not add recognizers by symmetry.

A concrete retained probe at the reconciliation head is
`start.div(1)` / `stop.div(1)` in
`protos/lib/collections/Range.protos`; this is a candidate for semantic
classification, not automatic replacement.

### A013-NULL-CONTROL — null-aware Object control

```text
OWNER=D144/#575 + I048/#589
CURRENT_FORMS=Object.ifNull,Object.ifNotNull
CLASS=REQUIRES_PER_OCCURRENCE_SEMANTIC_CLASSIFICATION
```

Only present-only action/transform or null-fallback logic with exactly equivalent
evaluation, callback reachability and result/control semantics is eligible.

Explicit null checks used for EOF, parser state, initialization state,
validation, test assertions or another domain distinction remain valid and are
not modernization debt.

## Reconciled obligations that require no new AUD013 sweep

### Standard prelude fail

```text
OWNER=D146/#576 + I046/#585
AUD013_CLASS=ALREADY_SATISFIED_NO_SWEEP
```

I046-B2 already removed 46 exact trivial local `fail` wrappers across 29 files
while preserving stateful/custom helpers, direct `Error().signal()`,
exact-object `error.signal()`, local shadowing and assertion semantics.

### Suite-native Test authoring

```text
OWNER=LIB018/#592 + TOOL009/#600
AUD013_CLASS=ALREADY_SATISFIED_NO_SWEEP
GLOBAL_PRODUCTION_LEGACY_CASES=0
REPOSITORY_EXECUTION_LOGICAL_ONLY=YES
```

TOOL009 completed the production migration. AUD013 may use `std:test/Test`
when structurally appropriate, but there is no remaining repository-wide
"convert every test" obligation.

### Source documentation

```text
OWNER=DOC008/#560
AUD013_CLASS=ALREADY_SATISFIED_NO_SWEEP
```

DOC008 already performed the horizontal maintained-source documentation pass.
The deliberately unresolved `TOML::array` and `TOML::table` semantics remain
owned by AUD005/#451 and LIB010/#418. AUD013 must not duplicate or preempt that
work.

### Protocol-first matching and removed matching syntax

```text
OWNER=D131/#503 + I041/#550
AUD013_CLASS=ALREADY_SATISFIED_NO_SWEEP
```

I041 already performed the cutover and repository-wide reconciliation.
Negative/historical fixtures remain intentional evidence.

### Standard loop selector rename

```text
OWNER=D180/#762 + I078/#764
AUD013_CLASS=ALREADY_SATISFIED_NO_SWEEP
STANDARD_SELECTOR=whileTrue
STALE_STANDARD_DOT_WHILE_CALLS=NONE
```

User-defined ordinary selectors named `while` remain valid and are not legacy
merely because their spelling matches the removed standard selector.

### AUD009 removals and relocations

AUD009 implementation owners already reconciled maintained source for removed or
relocated APIs such as the removed public `identityHash` selector, fixed-width
Core numeric placement, `Future.detach`, Path capability removals, filesystem
append/capture-tree changes, and parallel-Array placement.

Negative tests proving an old public API is absent are intentional retained
forms, not adoption debt.

### D179 supersession

The historical C3 monotonic execution-context membership obligation is
superseded by the owner-approved C0/E reconsideration and I071/#717. It must not
survive as an AUD013 modernization rule.

### Formatter/tooling and runtime/platform evolution

D183/D184/LM011 formatter work and later TEST/PERF/PLAT/BUG/runtime evolution do
not by themselves create a repository-wide Protos authoring transformation for
AUD013. The formatter remains its own tooling authority; AUD013 is not a global
reformat campaign.

The reconciliation-head LIB010-E2-A change is an internal complexity repair with
no specification, public API or encoded-spelling change and therefore creates no
new adoption rule.

## Source-surface taxonomy for the integrated sweep

The future source sweep uses these deterministic surface classes:

```text
STANDARD_LIBRARY
TOOLS_SUPPORT
TEST_CORPORA
EXAMPLES
PACKAGE_MODULE_FIXTURES_CURRENT
OTHER_MAINTAINED_PROTOS_SOURCE

NEGATIVE_SYNTAX_FIXTURE
HISTORICAL_FIXTURE
COMPATIBILITY_FIXTURE
GENERATED
CORPUS_ONLY
DOCUMENTATION_HISTORICAL_EXAMPLE
OTHER_INTENTIONAL_NON_CURRENT_SOURCE
```

A file is in scope when the repository executes, imports, tests, benchmarks or
presents it as maintained current Protos source. An individual occurrence may
still be intentionally negative, historical, generated or compatibility-owned.

Every candidate occurrence must end in exactly one AUD013 classification:

```text
ALREADY_CURRENT
MIGRATE_SEMANTICS_PRESERVING
KEEP_INTENTIONAL_LEGACY_FIXTURE
KEEP_DISTINCT_OR_STILL_CANONICAL
ROUTE_TO_EXISTING_OWNER
ROUTE_TO_NEW_DESIGN_OR_LIBRARY_WORK
OUT_OF_SCOPE_GENERATED_OR_HISTORICAL
```

## Remediation boundary

No active manifest family is safe as a blind textual replacement campaign.

A concrete edit becomes mechanical only after its occurrence has satisfied the
semantic preconditions of the corresponding manifest entry.

If a source occurrence would require richer Assertions, a new recognizer,
get-or-insert Map behavior, richer destructuring, altered null semantics, a new
test authoring institution or any other unratified capability, AUD013 routes it
instead of implementing that capability.

## Baseline verdict

```text
ACTIVATION_BASELINE_RECONCILED=YES
POST_BASELINE_RELEVANT_EVOLUTION_CLASSIFIED=YES
ADOPTION_MANIFEST=COMPLETE
TEST003_HANDOFF_CONSUMED=YES
D142_D143_HANDOFFS_CONSUMED=YES
LIB018_TOOL009_HANDOFF_RECONCILED=YES
DOC008_WORK_NOT_DUPLICATED=YES
AUD009_POST_BASELINE_EFFECTS_RECONCILED=YES
SOURCE_SURFACE_TAXONOMY=DEFINED
NO_NEW_SEMANTICS_INVENTED=YES
SOURCE_SWEEP_EXECUTED=NO
SOURCE_SWEEP_READY=YES
```

No owner-review gate is required before the first integrated sweep.

## Next slice

```text
NEXT_SLICE=AUD013-B1
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
SURFACE=protos/tests/library/**/*.protos
PURPOSE=integrated library-test source sweep
```

AUD013-B1 must inspect each file once against all active manifest entries, apply
only proven semantics-preserving migrations, retain/justify distinct forms, and
route missing capabilities rather than redesigning them.
