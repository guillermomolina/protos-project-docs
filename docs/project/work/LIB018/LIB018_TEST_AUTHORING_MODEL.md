# LIB018 — Minimal suite-native test authoring model

Status: **IN_PROGRESS — LIB018-0 C′ RATIFIED**

Owning work item: GitHub Issue `#592` — `LIB018 — Expand Protos testing support beyond the initial Assertions surface`

Research/decision sub-item: GitHub Issue `#593` — `LIB018-0 — Minimal Protos testing suite authoring model audit`

Nature: project Standard Library design record; **non-normative**

Explicit project-owner approval: **2026-09-18**

Protos evidence baseline: `64b21418f19501ff472d03286367b1535fabb3a4`

Upstream architecture:

- D152 / #595 — local-first flat CasePlan Test Tool architecture;
- D153 / #596 — explicit module declaration, authority-free discovery and
  symbolic fresh-Process rematerialization.

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

## Purpose

This record closes the LIB018-0 authoring-model decision and records the selected
initial suite-native Standard Library surface.

The selected architecture is **Candidate C′ — module-as-suite + canonical
`std:test/Test` values**.

The source module is the initial grouping/declaration owner. It exposes one exact
local declaration root named `tests`, whose value is a standard Array of named,
ordinarily invokable Test values.

No separate public `Suite` value is introduced initially.

Conceptually:

```protos
Test: import("std:test/Test")
Assertions: import("std:test/Assertions")

tests: Array(
    Test("first", () => {
        Assertions.require(...)
    }),

    Test("second", () => {
        Assertions.require(...)
    })
)
```

## Approval provenance

The project owner explicitly approved Candidate C′ on 2026-09-18 after the
renewed LIB018-0 packet had re-evaluated the authoring model against D152, D153
and the dependency/library audit.

The exact approval was:

> apruebo C′

```text
DECISION_APPROVAL_PROVENANCE=PASS
```

## Existing boundaries preserved

LIB018-0 does not reopen LIB016 Assertions semantics.

The current assertion surface remains:

```text
std:test/Assertions

Assertions.AssertionFailure
Assertions.require(condition)
Assertions.signals(errorPrototype, body)
```

D152 remains authoritative for:

- one logical Test -> one independently schedulable Case;
- one source -> zero/one/many logical Cases;
- flat inert CasePlan before scheduling;
- global work-conserving `--jobs N`;
- one fresh semantic Protos Process per logical Case;
- case-private stdout/stderr;
- test-neutral lower execution;
- migration ownership and no old/new double execution.

D153 remains authoritative for:

- one explicit authoring-owned declaration root;
- finite canonically ordered logical entries;
- stable unique non-empty String local selectors;
- no arbitrary slot/prefix discovery;
- no ambient mutable registration;
- no live Closure transfer into CasePlan or across Process boundaries;
- fresh-Process rematerialization by selector;
- declaration-signature consistency before body invocation;
- no Java-owned Test/Suite semantics;
- declaration construction before Case execution authority is provisioned.

## Dependency/library audit

The project owner explicitly requested that LIB018 use existing libraries and
determine whether another general-purpose library was actually required.

The audit result is:

```text
NEW_GENERAL_PURPOSE_LIBRARY_REQUIRED=NO
SEPARATE_NEW_LIBxxx_REQUIRED=NO
NEW_TESTING_SPECIFIC_MODULE=std:test/Test
```

The selected model reuses existing facilities.

| Capability | Existing owner | Selected action |
| --- | --- | --- |
| ordered declaration root | Core `Array` | reuse |
| stable test name/selector | Core `String` | reuse |
| body execution | ordinary invocation / Closures | reuse |
| canonical descriptor immutability | ordinary Object + `freeze()` | reuse |
| duplicate detection during discovery | Core `Map` internally | reuse |
| generated Cases | Array/iteration and optional `std:collections/Array.map` | reuse |
| assertion failure | `std:test/Assertions` | reuse |
| ordinary guest failure | Core `Error` | reuse |
| module grouping | module/import semantics | reuse |
| discovery/rematerialization | D153 Test Tool boundary | reuse |
| Case planning/scheduling | D152 Test Tool boundary | reuse |
| file focal selection | D151/TOOL008 | reuse |
| shared preparation | ordinary helper/factory Closures | reuse initially |

No general ordered-entry collection, diff library, temporary-filesystem helper,
subprocess helper, fixture library, matcher framework or parameterization
framework is required to implement the selected initial authoring model.

## Why a Test module is justified

The strongest no-new-library alternative used ordinary structural descriptors:

```protos
tests: Array(
    {
        name: "first"
        call: () => { ... }
    }
)
```

That model is technically sufficient but repeats the same testing-specific
descriptor ceremony in every suite-native source and invites local helper
variants.

The repository already experienced that failure mode with local `require` /
`reject` helpers before LIB016.

Therefore one tiny canonical testing-specific constructor is justified by present
authoring pressure:

```text
std:test/Test
```

This module belongs to LIB018 itself. It does not justify a separate general
library.

## Why no first-class Suite value

The earlier pre-D152/D153 LIB018-0 recommendation proposed:

```text
Test
Suite
Suite name
Suite children
```

D153 changes that conclusion.

The source module already owns the explicit declaration root, and D152 already
separates source association from logical Case identity.

The initial group is therefore already:

```text
source module
    -> tests Array
        -> Test A
        -> Test B
        -> Test C
```

A second `Suite("name", tests)` value would duplicate grouping/name structure
without current evidence for:

- multiple separately named suites per source;
- nested groups;
- suite-local lifecycle;
- suite metadata.

A public Suite abstraction remains a future option if genuine grouping or
navigation pressure appears.

## Public module

The canonical new module is:

```text
std:test/Test
```

The imported module is ordinarily invokable as the Test factory:

```protos
Test: import("std:test/Test")

test: Test("name", () => {
    ...
})
```

## Test(name, body) contract

The selected public operation is:

```text
Test(name, body)
```

### Name

`name` must be a non-empty semantic String.

```text
TEST_NAME=NON_EMPTY_SEMANTIC_STRING
```

The exact ordinary Error mechanics for invalid API arguments follow existing
Core/Standard Library validation conventions. If implementation exposes a new
observable invalid-argument taxonomy choice not already determined by existing
rules, the implementation slice must stop at the normal approval gate.

### Returned value

Each successful invocation returns one fresh frozen ordinary object.

```text
TEST_VALUE=FRESH_FROZEN_ORDINARY_OBJECT
```

The Test value has:

- a public local `name` slot containing the exact supplied String;
- ordinary invokable behavior through a local `call` Closure.

The Test value is **not** a privileged runtime type, Core value family, compiler
entity or Java-owned object.

```text
TEST_NOMINAL_RUNTIME_TYPE=NO
TEST_SPECIAL_CORE_VALUE_KIND=NO
TEST_COMPILER_MAGIC=NO
```

### Invocation

Invoking the Test value with zero arguments invokes its captured body with zero
arguments.

```text
TEST_INVOCATION_ARGUMENTS=ZERO
TEST_INVOCATION_RESULT=EXACT_BODY_RESULT
TEST_INVOCATION_ERROR=EXACT_BODY_ERROR_PROPAGATION
```

The library does not catch, aggregate or reclassify the body's ordinary result or
Error merely because the value is a Test.

The canonical Test value is itself ordinarily invokable because the library
creates its local `call` behavior.

D153 discovery therefore needs no new public Core `isCallable` or
Closure-reflection API merely to identify a canonical Test value.

## Declaration root

The exact local declaration-root name is:

```text
tests
```

Its value is a standard Array.

```text
DECLARATION_ROOT_LOCAL_SLOT=tests
DECLARATION_ROOT_KIND=STANDARD_ARRAY
DECLARATION_ORDER=ARRAY_INDEX_ORDER
```

The initial declaration may be empty:

```text
EMPTY_TESTS_ROOT=ALLOWED
```

Each root entry must expose:

- a non-empty String `name`;
- ordinary invokable behavior.

The canonical `std:test/Test` factory is the intended constructor of such
entries, but D153/Test Tool discovery need not impose hidden nominal identity on
them.

## Duplicate names

Two logical entries in the same source declaration may not claim the same local
String selector.

```text
DUPLICATE_TEST_NAME=DISCOVERY_ERROR
```

Because the declaration root is an Array, duplicate generated values remain
observable to D153 discovery and cannot be silently overwritten as they could be
through ordinary Map replacement.

## Module-as-suite grouping

The source module is the initial suite/group.

No initial public suite name exists independently of source/module association.

```text
SEPARATE_SUITE_VALUE_INITIAL=NO
SUITE_NAME_INITIAL=NO
NESTED_SUITE_INITIAL=NO
```

This is an authoring/grouping decision only; the physical file/source remains
distinct from logical Case identity under D152/D151.

## Generated/table-driven Tests

The selected design requires no parameterized-test framework.

Ordinary data and collection helpers may construct Tests.

For example, a source may use existing `std:collections/Array.map` to turn
ordinary case data into Test values.

Generated Tests remain ordinary Test entries whose names must be stable and
unique under D153.

```text
PARAMETERIZATION_FRAMEWORK=NO
GENERATED_TESTS=ORDINARY_PROTOS_COMPOSITION
```

## Assertions

LIB018-0 adds no new assertion helper.

```text
ASSERTIONS=EXISTING_std:test/Assertions
NEW_ASSERTION_API_INITIAL=NO
```

Repository evidence shows many equality-oriented assertions, so richer
expected/actual diagnostics may be valuable later.

That evidence does not make a diff library, matcher DSL or equality-specific
assertion a dependency of the initial Test authoring model.

Such additions remain separate later work if migration evidence justifies them.

## Shared preparation

The initial mechanism is ordinary helpers/factories inside the source module.

```protos
makeFixture: () => {
    ...
}

tests: Array(
    Test("first", () => {
        fixture: makeFixture()
        ...
    }),

    Test("second", () => {
        fixture: makeFixture()
        ...
    })
)
```

D152's fresh Process per logical Test gives each Case independent semantic state.

LIB018-0 does not create lifecycle hooks merely to shorten ordinary helper calls.

## Composition

Tests compose through ordinary Arrays and modules.

A source may concatenate/import arrays of ordinary Test values or generate them
with collection functions where appropriate.

The initial model does not require recursive Suite composition.

## Explicitly not selected

LIB018-0 does not select:

- a public `Suite` type/value;
- nested suites;
- setup / teardown / beforeEach / afterEach;
- fixture scopes or shared fixture lifecycle;
- TestResult values;
- parameterization framework;
- tags / skip / only / todo;
- timeout / retry;
- matcher DSL;
- structural diff library;
- mocks / stubs;
- snapshots / golden-file framework;
- property/generative testing;
- plugin/extension engine;
- compiler annotations/macros/assertion rewriting;
- global registration;
- source-name prefix discovery;
- Java-owned Test/Suite semantics.

Every deferred capability requires independent evidence and approval when
substantive.

## Test Tool relationship

The library does not schedule or aggregate Tests.

```text
LIBRARY_RUN_METHOD=NO
LIBRARY_RESULT_AGGREGATOR=NO
```

D153 discovers the `tests` Array, projects inert names/source association into
the D152 CasePlan, discards discovery-time live bodies and later reconstructs the
source in each selected fresh Process.

One Test therefore maps to one authoritative logical Case.

```text
ONE_TEST=ONE_LOGICAL_CASE
ONE_TEST=ONE_FRESH_PROCESS
ONE_SOURCE_TO_N_TESTS=YES
PHYSICAL_PATH_IS_IDENTITY=NO
```

## D151 / TOOL008

`--file FILE` remains a source-backed focal selector.

For a suite-native source:

```text
FILE
    -> Test A
    -> Test B
    -> Test C
```

file selection selects all authoritative logical Cases associated with the exact
source.

The file is not itself the logical Case identity.

## Migration

No flag day is required.

```text
not-yet-migrated source
    -> incumbent production Test Tool ownership

migrated suite-native source
    -> tests Array
    -> D153 discovery
    -> N D152 Cases
```

The same bounded migration change removes old authoritative ownership for a
migrated production test/source so D152's no-double-execution invariant remains
true.

Multi-scenario files are natural early migration candidates. Single-purpose
one-source/one-case tests need not be rewritten merely for aesthetic uniformity
unless later adoption work demonstrates value.

## Dependency conclusion

```text
NEW_GENERAL_PURPOSE_LIBRARY_REQUIRED=NO
SEPARATE_NEW_LIBxxx_REQUIRED=NO

NEW_TESTING_SPECIFIC_MODULE=std:test/Test

REUSED=
    Core Array
    Core Map
    Core String
    Core Error
    ordinary Object/freeze
    ordinary invocation/Closures
    modules/import
    std:collections/Array where useful
    std:test/Assertions
    D151/D152/D153 Test Tool boundaries
```

## Comparative result

The renewed candidate set included:

- A — Assertions only;
- B — Map as declaration root;
- C — plain structural Test descriptors in Array;
- C′ — canonical `std:test/Test` values in Array;
- D — explicit Test + Suite descriptors;
- E — local builder registration;
- F — fixture-aware model.

The current-AGENTS informational totals were:

| Candidate | Score / 60 |
| --- | ---: |
| A — Assertions only | 42 |
| B — Map root | 53 |
| C — plain structural Array | 55 |
| **C′ — Test + Array** | **57** |
| D — Test + Suite | 51 |
| E — builder registration | 47 |
| F — fixture-aware | 40 |

The selection was not arithmetic-only.

C′ was selected because it is the smallest model that:

- satisfies D152/D153;
- removes real authoring duplication;
- preserves generated duplicate detection;
- leaves grouping to the source module and ordering to Array;
- requires no new general-purpose library;
- avoids a second Suite institution;
- stays ordinary, structural and test-neutral below the authoring layer.

## Ratified contract

```text
LIB018_0_STATUS=RATIFIED
LIB018_0_SELECTED_CANDIDATE=C_PRIME_MODULE_AS_SUITE_CANONICAL_TEST

PUBLIC_MODULE_NEW=std:test/Test

TEST_FACTORY_CALL=Test(name, body)
TEST_NAME=NON_EMPTY_SEMANTIC_STRING
TEST_VALUE=FRESH_FROZEN_ORDINARY_OBJECT
TEST_LOCAL_NAME_SLOT=YES
TEST_ORDINARILY_INVOKABLE=YES
TEST_INVOCATION_ARGUMENTS=ZERO
TEST_INVOCATION_RESULT=EXACT_BODY_RESULT
TEST_INVOCATION_ERROR=EXACT_BODY_ERROR_PROPAGATION

TEST_NOMINAL_RUNTIME_TYPE=NO
TEST_SPECIAL_CORE_VALUE_KIND=NO
TEST_COMPILER_MAGIC=NO

DECLARATION_ROOT_LOCAL_SLOT=tests
DECLARATION_ROOT_KIND=STANDARD_ARRAY
DECLARATION_ORDER=ARRAY_INDEX_ORDER
EMPTY_TESTS_ROOT=ALLOWED

ROOT_ENTRY_REQUIRED_NAME=NON_EMPTY_STRING
ROOT_ENTRY_REQUIRED_BEHAVIOR=ORDINARILY_INVOKABLE
DUPLICATE_TEST_NAME=DISCOVERY_ERROR

SEPARATE_SUITE_VALUE_INITIAL=NO
SUITE_NAME_INITIAL=NO
NESTED_SUITE_INITIAL=NO

ASSERTIONS=EXISTING_std:test/Assertions
NEW_ASSERTION_API_INITIAL=NO

SETUP_TEARDOWN_INITIAL=NO
FIXTURE_MODEL_INITIAL=NO
PARAMETERIZATION_FRAMEWORK=NO
TEST_RESULT_LIBRARY_VALUE=NO
TAGS_SKIP_ONLY=NO
TIMEOUT_RETRY=NO

NEW_GENERAL_PURPOSE_LIBRARY_REQUIRED=NO
SEPARATE_NEW_LIBxxx_REQUIRED=NO

TOOL_DISCOVERY=D153
TOOL_CASEPLAN=D152
ONE_TEST=ONE_LOGICAL_CASE
ONE_TEST=ONE_FRESH_PROCESS
FILE_IS_CASE_ID=NO
```

## Invariant/delta consistency

The selected C′ contract was checked against explicit D152/D153 invariants, the
owner-requested dependency/library audit and current Protos main before durable
publication.

```text
D152_LOGICAL_CASE=PASS
D152_SOURCE_CAN_HAVE_N_CASES=PASS
D152_FRESH_PROCESS_PER_CASE=PASS
D152_TEST_NEUTRAL_EXECUTOR=PASS

D153_EXPLICIT_DECLARATION_ROOT=PASS
D153_STABLE_STRING_SELECTOR=PASS
D153_NO_PREFIX_SCAN=PASS
D153_NO_GLOBAL_REGISTRY=PASS
D153_NO_LIVE_CLOSURE_TRANSFER=PASS
D153_FRESH_PROCESS_REMATERIALIZATION=PASS

DEPENDENCY_LIBRARY_AUDIT=PASS
NEW_GENERAL_PURPOSE_LIBRARY_REQUIRED=NO

NEW_UNSURFACED_ARCHITECTURAL_CONSEQUENCE=NO
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Closure / implementation gate

LIB018-0 is now ratified.

LIB018 parent work may decompose implementation, Test Tool integration and
migration/adoption slices against this exact contract.

Bounded implementation must stop at the normal approval gate if it exposes a new
substantive choice not already fixed above, including any new behavior around:

- invalid-argument taxonomy;
- declaration-root acquisition semantics beyond D153;
- CaseId encoding;
- case-specific execution environment;
- result classification/exit codes;
- fixtures/lifecycle;
- diagnostics;
- parameterization;
- nested grouping;
- migration compatibility.

```text
SPECIFICATION_CHANGED=NO
LANGUAGE_SEMANTICS_CHANGED=NO
STANDARD_LIBRARY_IMPLEMENTATION_CHANGED=NO
TEST_TOOL_IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
