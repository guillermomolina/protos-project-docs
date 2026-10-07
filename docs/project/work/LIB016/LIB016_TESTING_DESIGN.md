# LIB016 — Protos testing-support Standard Library design

Status: **IN_PROGRESS — LIB016-0 B′ RATIFIED**

Owning work item: GitHub Issue `#557` — `LIB016 — Protos testing support library and Test Tool boundary`

Research/decision sub-item: GitHub Issue `#558` — `LIB016-0 — Testing authoring model and Test Tool boundary comparative audit`

Nature: project Standard Library design record; **non-normative**

Explicit project-owner approval: **2026-09-18**

Protos evidence baseline: `76a8508baca9ce411752d9fb275406db8d24f0b2`

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

## Purpose

This record closes the LIB016-0 comparative research checkpoint and records the
selected initial architecture for reusable test-authoring support in ordinary
Protos source.

The selected architecture is **Candidate B′ — minimal runner-independent
assertion library**.

The core boundary is deliberately small:

```text
ordinary Protos test source
        |
        v
std:test/Assertions
        |
        | ordinary Protos Error/Boolean/equality semantics
        v
ordinary program outcome
        |
        v
TOOL002 / protos test
        |
        +-- corpus/TestPlan acquisition
        +-- Process isolation
        +-- scheduling / --jobs
        +-- execution requirements / resources
        +-- stdout/stderr capture
        +-- aggregation / reporting
        `-- CLI exit classification
```

LIB016 therefore standardizes only the smallest repeated authoring mechanism.
It does not create a testing framework, runner protocol, registration system or
second test-specific runtime universe.

## Current Protos pressure

Current repository tests already expose two valid authoring styles.

### External expectations

The conformance corpus contains ordinary `.protos` programs whose expected
outcomes are described externally through manifest/TestPlan policy interpreted
by TOOL002.

Typical expectations include Boolean, Integer, Error and Future terminal-state
outcomes.

This style remains valid. LIB016 does not replace it.

### Self-checking ordinary Protos programs

Several Standard Library corpora instead perform their own checks in ordinary
Protos source and repeatedly define local helpers such as:

```protos
require: (condition) => {
    condition.ifFalse(() => {
        Error().signal()
    })
}
```

Some families also locally implement expected-Error helpers.

That repetition is concrete authoring pressure, but it does not justify a full
testing framework. LIB016-0 therefore starts from the smallest reusable mechanism
that removes the repeated semantics without absorbing TOOL002 responsibilities.

## Existing TOOL002 boundary

TOOL002 remains the authority for the bundled Test Tool.

Already-selected Test Tool architecture keeps these concerns outside LIB016:

- logical TestPlan / CaseSpec construction;
- corpus/source acquisition;
- stable case identity;
- fresh semantic Protos Process / RootActor execution;
- scheduling and `--jobs` policy;
- execution requirements and resource allocation;
- stdout/stderr capture;
- infrastructure versus semantic outcome separation;
- aggregation and reporting;
- CLI invocation and exit-status policy;
- future hardened/remote execution placement.

The Test Tool's exact-entry backend remains mechanically test-neutral.

The historical TOOL002 architecture explicitly left a later
assertion/helper/self-asserting-test API as a separate decision. LIB016 owns that
public authoring question without reopening TOOL002 execution architecture.

TEST001 remains closed under its runner-cutover meaning. TEST002 remains the
separate owner of legacy Java-to-Protos semantic-test migration.

## Comparative prior-art findings

LIB016-0 compared materially different testing systems to identify semantic and
ownership boundaries rather than copy API spelling.

### Smalltalk / Pharo SUnit

SUnit demonstrates a clean object-oriented separation among TestCase, TestSuite,
TestResult and resources. It also distinguishes assertion failure from an
unexpected error.

The useful lesson for Protos is the distinction between assertion failure and
arbitrary program failure.

The TestCase/Suite/Result framework topology is larger than current Protos
pressure requires and is not selected.

### Prototype-oriented testing

Prototype-oriented systems demonstrate that test support can be implemented with
ordinary objects rather than language-level test syntax or classes.

The useful lesson is that Protos does not need a privileged testing object model.

Magic naming, distinguished framework roots or global registration do not follow
from prototype orientation and are not selected.

### Rust

Rust's `#[test]` items and assertion macros provide strong diagnostics and
integration, but they depend on compiler-recognized attributes/macros and a
generated test harness.

The useful lesson is negative for LIB016: rich assertion diagnostics obtained
through compiler participation are not free library behavior and must not be
smuggled into a Standard Library decision.

### Go

Go's `testing.T` centralizes failure state, logging, cleanup, subtests, skipping
and parallel-test controls in one runner-owned test context.

This is powerful, but it creates a substantial test-specific institution.
Current Protos tests do not require such a context object.

### Python `unittest` and pytest

`unittest` provides explicit TestCase/TestSuite/TestResult layering. pytest
shows excellent authoring ergonomics through assertion rewriting, discovery,
fixture injection/scopes and plugin machinery.

Both systems are useful evidence for mature testing needs, but pytest in
particular demonstrates how much hidden machinery may sit behind apparently
simple assertions.

LIB016 does not adopt assertion rewriting, implicit fixture injection or a global
plugin/registration universe.

### JUnit with Surefire / Gradle

JUnit plus external execution infrastructure is strong evidence that authoring
semantics and physical execution/forking policy are separable concerns.

LIB016 keeps that separation but does not copy Java annotation/class institutions.

### Elixir ExUnit

ExUnit is valuable concurrency prior art because tests compose with BEAM process
isolation and asynchronous execution.

Its assertion layer is useful evidence for ordinary authoring support. Its macro,
registration, setup-context and async-case framework is not required by current
Protos pressure.

### Swift Testing / XCTest

Swift Testing demonstrates modern structured tests, parameterization, traits and
rich expectation diagnostics.

Its ergonomics rely significantly on compiler/macro support and test metadata.
Those facilities are intentionally outside the initial Protos assertion library.

### Node

Node provides the strongest direct ownership precedent for LIB016:
`node:assert` is an assertion library distinct from `node:test`, and ordinary
program execution can use the assertion library without participating in the test
runner.

This maps closely to the selected Protos boundary: assertions are reusable,
runner-independent ordinary library behavior.

### QuickCheck / Hypothesis

Property testing adds generators, replay/seeds, shrinking, corpus/database
policy and failure minimization. It is a distinct testing model rather than a
small extension to example assertions.

Property/generative testing remains deferred.

## Candidate architectures

### Candidate A — no public testing library initially

Keep external expectations and local helper definitions.

This has zero new public surface and remains technically safe because a library
could be added later without a foundational rewrite.

The continuing cost is duplicated authoring helpers and no dedicated distinction
between assertion failure and an arbitrary Error.

### Candidate B′ — minimal runner-independent assertion library — SELECTED

Publish one ordinary Standard Library module with only:

```text
std:test/Assertions

Assertions.AssertionFailure
Assertions.require(condition)
Assertions.signals(errorPrototype, body)
```

The module has no runner registration, suite identity, fixture graph or ambient
authority.

### Candidate C — public first-class Test / Case / Suite values

Represent test identity, grouping and metadata as public Standard Library values.

Rejected for the initial design because TOOL002 already owns internal CaseSpec /
TestPlan execution planning and no current authoring requirement proves the need
for a second public structural model.

This candidate carries an overengineering red flag.

### Candidate D — public TestResult / failure-value protocol

Make tests return structured pass/fail values consumed by TOOL002.

Rejected for the initial design because it creates a second failure universe
beside ordinary Core Error signaling and publicly couples the library to the
runner without a current requirement for aggregate/non-unwinding assertion
results.

### Candidate E — tool-private assertion helpers

Place reusable assertion helpers under TOOL002 rather than `std:`.

Rejected because assertions require no Test Tool authority and are useful for
ordinary direct execution. Tool-private ownership would make TOOL002 a special
authoring dependency.

### Full framework / registration model

A larger framework containing test registration, suites, fixtures, lifecycle
hooks, tags, skip/only, parameterization and related policy is deliberately not
part of the initial contract.

These capabilities may be valuable later, but current evidence does not justify
paying their public and conceptual cost now.

### Language/compiler-integrated assertions

Special test syntax, annotations, compiler test mode, source rewriting, macros or
privileged call-site AST reflection are not selected.

Current requirements can be satisfied through ordinary library mechanisms.

## AGENTS.md candidate evaluation

Scores are 1–5. Arithmetic is an aid, not decision authority.

| Criterion | A none | B′ minimal assertions | C Test/Case/Suite | D result protocol | E tool-private |
| --- | ---: | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 5.0 | **5.0** | 4.0 | 3.5 | 4.5 |
| Protos alignment | 4.5 | **5.0** | 2.5 | 2.5 | 3.0 |
| Present-need proportionality | 3.0 | **5.0** | 2.0 | 2.0 | 4.0 |
| Incremental growth | 5.0 | **5.0** | 3.0 | 3.5 | 3.0 |
| Future-option resilience | 4.5 | **5.0** | 3.5 | 3.5 | 3.5 |
| Scalability | 4.0 | **5.0** | 4.0 | 4.0 | 4.0 |
| Conceptual simplicity | **5.0** | 4.5 | 2.5 | 2.5 | 4.0 |
| Portability / implementation freedom | 5.0 | **5.0** | 4.0 | 4.5 | 3.0 |
| Runtime / resource cost | 5.0 | **5.0** | 4.0 | 4.5 | **5.0** |
| Failure / operability | 2.5 | **4.5** | 4.0 | 3.0 | 3.5 |
| Deferral / reversibility / migration | **5.0** | **5.0** | 2.5 | 3.0 | 3.5 |
| Evidence maturity / implementation risk | 4.0 | **5.0** | 4.5 | 3.5 | 4.0 |

Confidence is HIGH for the B′ ownership boundary and initial authoring need.
Confidence is MEDIUM where richer framework candidates depend on currently absent
future requirements.

## Incremental-design analysis

### Smallest sufficient solution

The smallest useful standardization supported by current repository evidence is:

```text
require(Boolean)
signals(ErrorPrototype, Closure)
```

plus one distinguishable assertion-failure category.

No current evidence requires public Test, Case, Suite, fixture, registration or
property-testing abstractions.

### Pay for what you need

B′ imposes no conceptual/runtime cost on programs that do not import it.

Tests that do import it pay only for ordinary local library calls and ordinary
Error occurrences. There is no global registry, scheduler, worker, filesystem,
network, clock or background-task cost.

### Grow as you need

Future equality-specific helpers, richer diagnostics, fixtures, parameterization,
property testing and runner adapters can be added without changing the meaning of
the initial three public bindings.

The design is future-compatible without preimplementing those capabilities.

### Cost of deferral

Deferring convenience assertions, messages, fixtures, suites, tags and property
testing is additive and low-cost.

Deferring LIB016 entirely would also be technically safe; Candidate A's cost is
continued duplication rather than architectural rewrite.

One boundary should not be deferred once a public assertion API exists:
assertion-failure identity. Changing later from generic `Error` to a distinct
failure parent would alter observable handler matching. Therefore B′ fixes
`AssertionFailure` now.

## Ratified public identity

The canonical initial module identity is:

```text
std:test/Assertions
```

No broader `std:test` framework namespace behavior is implied by this module
identity.

## Ratified public surface

The initial public bindings are exactly:

```text
Assertions.AssertionFailure
Assertions.require(condition)
Assertions.signals(errorPrototype, body)
```

No additional assertion helper is authorized by LIB016-0.

## AssertionFailure contract

`AssertionFailure` is an ordinary Error prototype whose immediate delegation
parent is Core `Error`.

It has no required own payload slots in the initial contract.

Each assertion failure occurrence manufactured by the library is a fresh ordinary
Error occurrence whose immediate delegation parent is `AssertionFailure`, under
the existing Core Error freshness/identity rules.

No singleton failure instance is introduced.

## require contract

Conceptually:

```text
Assertions.require(true)  -> null
Assertions.require(false) -> signal fresh AssertionFailure
```

The condition is a canonical Boolean.

A non-Boolean argument is API misuse and is **not** an assertion failure. Its exact
ordinary argument/domain failure mechanics remain governed by already-applicable
Standard Library/Core rules; if bounded implementation exposes a new observable
choice not already determined there, implementation must stop at the explicit
approval gate rather than inventing it.

`require` does not define alternate equality, identity, truthiness or predicate
semantics.

Callers use ordinary Protos operations explicitly, for example:

```protos
Assertions.require(actual == expected)
Assertions.require(actual === expected)
```

## signals contract

Conceptually:

```text
Assertions.signals(errorPrototype, body)
```

composes ordinary Core Error semantics.

The approved behavior is:

- `body` is invoked under matching based on ordinary Error delegation;
- if `body` signals a matching Error, `signals` returns the exact caught Error
  object;
- if `body` completes normally, `signals` signals a fresh
  `AssertionFailure`;
- if `body` signals a non-matching Error, that exact Error propagates unchanged;
- invalid arguments are ordinary API misuse, not a successful expected-error
  assertion.

No second hidden error taxonomy or matching channel is introduced.

Returning the exact caught Error allows callers to perform any additional
ordinary assertions themselves:

```protos
error: Assertions.signals(InvalidThing, () => {
    operation()
})

Assertions.require(error.parent() === InvalidThing)
```

## Diagnostics boundary

LIB016-0 does not promise pytest/Rust/Swift-style expression decomposition.

Those ecosystems obtain rich diagnostics through source rewriting, macros,
compiler integration or dedicated metadata. Protos currently has no approved
general mechanism exposing assertion call-site expression structure to an
ordinary library.

`AssertionFailure` therefore has no required own diagnostic payload in the
initial contract.

A future general diagnostics/source-location facility may be designed separately
and may later enrich testing support without retroactively making LIB016 a
parser/compiler feature.

## TOOL002 consequences

LIB016-0 changes no TOOL002 semantics.

TOOL002 continues to:

- execute ordinary Protos test source;
- observe normal semantic completion/failure;
- retain external manifest/TestPlan expectations;
- own scheduling, resource, capture, reporting and exit policy.

A self-checking source using `std:test/Assertions` remains an ordinary Protos
program from the executor's perspective.

Existing manifest-based conformance cases require no migration.

## TEST002 consequences

TEST002 may later use LIB016 when migrating suitable legacy Java/JUnit semantic
tests, but LIB016 does not become a prerequisite for all migration work and
TEST002 does not gain authority over the public assertion API.

Migration convenience cannot expand LIB016 by transitivity.

## Future-scenario stress

### Large and highly parallel suites

The assertion library has no shared mutable registry or scheduler state. Each
case remains independently schedulable by TOOL002.

### Tasks, Futures and Actors

Assertions use ordinary Error control and do not introduce a test-specific task,
Actor or Future model.

Asynchronous case completion remains ordinary Protos/TOOL002 execution policy.

### Distributed or hardened execution

`AssertionFailure` remains a semantic Error occurrence inside the case Process.
TOOL002's existing semantic-outcome versus infrastructure-outcome separation is
unchanged.

### Alternate runtimes

The selected API depends only on ordinary Protos values, Boolean semantics and
Core Error signaling/handling. It does not depend on JVM, Truffle, threads, OS
processes or host test frameworks.

### Direct execution

The module remains useful outside TOOL002. A source may import Assertions and be
run directly through the ordinary Protos execution path.

### Future fixtures or property testing

Those layers may be added separately. B′ reserves no lifecycle, sharing,
generator, shrinking, seed or corpus semantics.

## Strongest argument against B′

Candidate A remains genuinely credible.

The current local `require` helper is only a few lines, and adding a public
library later would not require foundational redesign. B′ therefore creates a
compatibility promise mainly to remove already-observed repository duplication.

B′ is nevertheless selected because the pressure is concrete across independent
Standard Library corpora and the reusable solution remains exceptionally small,
pure, authority-free and runner-independent. It solves demonstrated friction
without installing framework machinery in advance.

## Intentionally deferred

LIB016-0 deliberately does not select:

- `equal`, `same`, `notEqual` or approximate-comparison convenience APIs;
- custom assertion-message APIs;
- source-expression introspection;
- source-location payload semantics;
- Test / Case / Suite public values;
- setup / teardown or fixture scopes;
- registration or discovery conventions;
- tags, skip, only or todo;
- timeout or cancellation declarations;
- execution-resource declarations;
- parameterized-test registration;
- snapshots / golden files;
- mocking / stubbing frameworks;
- property / generative testing and shrinking;
- fuzzing;
- coverage APIs;
- benchmark APIs;
- IDE-specific test protocols;
- compiler macros, annotations or assertion rewriting;
- any change to TOOL002 scheduler, execution or reporting semantics.

Every deferred capability requires its own evidence and explicit approval where
the choice is substantive.

## Implementation gate after ratification

Bounded implementation may now realize only the ratified B′ contract.

Expected implementation characteristics are:

- ordinary Protos Standard Library code;
- no Java production bridge;
- no special parser/compiler/runtime support;
- focused conformance through ordinary Protos source;
- existing TOOL002 behavior unchanged unless a separately approved integration
  slice later justifies a change.

If implementation exposes a substantive choice about invalid-argument taxonomy,
diagnostic payload, public representation, source location, additional helpers or
another observable behavior not fixed by B′ or already-authoritative rules, the
affected implementation slice must stop and cross the normal explicit approval
gate.

## Ratification summary

```text
LIB016_0_STATUS=RATIFIED
LIB016_0_SELECTED_CANDIDATE=B_PRIME
PUBLIC_MODULE=std:test/Assertions
INITIAL_SCOPE=MINIMAL_ASSERTIONS_ONLY
ASSERTION_FAILURE_PARENT=Error
ASSERTION_FAILURE_REQUIRED_OWN_PAYLOAD=NONE
REQUIRE=YES
SIGNALS=YES
RUNNER_REGISTRATION=NO
TEST_CASE_TYPE=NO
TEST_SUITE_TYPE=NO
FIXTURES=NO
GLOBAL_REGISTRY=NO
SPECIAL_SYNTAX=NO
COMPILER_ASSERTION_REWRITE=NO
PROPERTY_TESTING=DEFERRED
RICH_DIAGNOSTIC_MODEL=DEFERRED
TOOL002_SEMANTICS_CHANGED=NO
CORE_SEMANTICS_CHANGED=NO
SPECIFICATION_CHANGED=NO
IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
```
