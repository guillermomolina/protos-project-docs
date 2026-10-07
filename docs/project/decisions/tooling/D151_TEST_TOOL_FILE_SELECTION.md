# D151 — Test Tool exact file-backed focal selection contract

Status: **RATIFIED — Candidate B′ selected**

Allocated: **2026-09-18**

Explicit project-owner approval: **2026-09-18**

Decision issue: `guillermomolina/protos#590`

Research baseline: `guillermomolina/protos@d1bbab2c1c1023e980b43ca01e7b2adafcac05f8`

Primary architectural authority: TOOL002, D122, D123, D125, D126, D132,
D133 and the current Test Tool implementation.

Nature: implementation-independent Test Tool focal-selection and CLI contract.

Normative language effect: **none**.

## Decision

The Test Tool gains one explicit **exact local-file focal selector**:

```text
protos test --file FILE
```

`FILE` is an invocation-local developer locator. It is not Test Tool semantic
identity and it does not create test membership.

The selector resolves against the current invocation's authorized corpus/source
bindings and then selects **every already-authoritative CaseSpec associated with
that exact source file** from the materialized TestPlans.

Conceptually:

```text
local FILE locator
        |
        | controller/invocation-local resolution only
        v
authorized corpus source entry
        |
        v
authoritative SuiteGraph + CorpusBindings
        |
        v
materialized TestPlans + resource joins
        |
        v
all existing CaseSpecs associated with that file
        |
        v
ordinary D108 scheduling / execution / reporting
```

The file selector therefore chooses existing tests; it never defines tests.

## Approval provenance

The project owner approved **D151 Candidate B′** on 2026-09-18 after the
decision packet had been refined to preserve future one-file-to-many-case unit
testing.

The exact owner approval was:

> apruebo B'

Immediately before that approval, B′ had been clarified so that:

```text
one source file -> zero, one, or many authoritative CaseSpecs

--file FILE
    -> selects all authoritative CaseSpecs associated with FILE
```

rather than requiring exactly one CaseSpec per file.

```text
DECISION_APPROVAL_PROVENANCE=PASS
```

## Problem being solved

The repository already orders executable validation from cheaper, higher-signal
checks toward broader and more expensive validation. Java/JUnit can mechanically
run one focal test class or method, but the public Protos Test Tool currently
executes suite/corpus plans and exposes no supported way to request one known
file-backed focal test source.

This causes a concrete development cost: an agent or developer changing one
Protos regression source may have to run a substantially broader corpus merely
to obtain the first executable signal.

D151 adds the missing selection surface without changing any publication rule.
Passing a focal selection does not authorize skipping broader validation that
the repository impact matrix still requires.

## Existing identity model

D151 preserves the already-ratified separation:

```text
local source file locator
    != CaseId
    != CorpusId
    != SuiteId
    != ExecutionRequirementId
```

This distinction is observable in the current Test Tool.

For some retained three-column manifest corpora, the initial CaseId happens to
match the validated manifest path. That is not a universal identity rule.

D132 explicitly ratified logical CaseIds for two-column corpora as:

```text
caseNamespace + relative-case-key
```

and fixed:

```text
CaseId != filesystem path
```

D133 further keeps physical case/project-tree paths out of CaseId identity.

Therefore D151 does not reinterpret a path as a logical case identifier.

## File selector semantics

### Exact CLI form

Generation 1 selects only the exact separate-token form:

```text
protos test --file FILE
```

The option may occur zero or one time.

```text
--file FILE       selected
--file=FILE       not an alias initially
```

The latter remains outside the selected Test Tool projection under the existing
unknown-argument compatibility policy.

D151 does not use a positional file selector.

### Locator role

`FILE` is a controller/invocation-local locator used only to find source entries
already represented by authoritative Test Tool plans.

It is not:

- a CaseId;
- a CorpusId;
- a SuiteId;
- an ExecutionRequirementId;
- persisted suite/corpus metadata;
- a remote-worker identity;
- a cache identity;
- an implicit test declaration.

The physical locator terminates at selection. Ordinary logical CaseSpec data
continues through scheduling/execution.

### Relative and absolute input

An explicit relative `FILE` is interpreted relative to the launcher invocation
working directory.

An explicit absolute `FILE` is allowed as an invocation-local locator.

These rules do not make CWD or an absolute path part of Test Tool semantic
identity. A different checkout location does not rename any CaseId, CorpusId or
SuiteId.

### Exact source selection

Generation 1 adds no path search language.

```text
DIRECTORY_SELECTOR=NO
GLOB_SELECTOR=NO
REGEX_SELECTOR=NO
LINE_SELECTOR=NO
IMPLICIT_EXTENSION=NO
SEARCH_PATH_FALLBACK=NO
```

Symlink/reparse aliases are not selected initially. The implementation must fail
closed rather than manufacture equivalence through uncertain physical
indirection.

A later explicit decision may broaden locator convenience without changing the
logical Test Tool identity model.

## Selection cardinality

The selected contract deliberately does **not** require one file to correspond
to exactly one CaseSpec.

Instead:

```text
FILE_SELECTION =
    ALL_AUTHORITATIVE_CASES_ASSOCIATED_WITH_EXACT_FILE

ZERO_MATCHES =
    TEST_TOOL_SELECTION_ERROR

ONE_MATCH =
    RUN_ONE_CASE

MULTIPLE_MATCHES =
    RUN_ALL_MATCHING_CASES_IN_CANONICAL_PLAN_ORDER
```

This preserves a future unit-testing design in which one source file may declare
or own several logical test cases:

```text
math.protos
    +-- CaseId A
    +-- CaseId B
    `-- CaseId C

protos test --file math.protos
    -> A + B + C
```

D151 does not itself define such a unit-testing library, discovery mechanism or
CaseSpec-generation model. It only prevents the focal file selector from
freezing the current common `1 file -> 1 CaseSpec` representation.

If an explicitly composed suite graph causes the same physical source to
participate in multiple authoritative suite/corpus plan positions, file
selection preserves those authoritative plan positions rather than silently
deduplicating them by physical path.

## Selection authority and ordering

Test membership remains authoritative before focal selection.

The selected semantic order is:

```text
1. resolve and validate the explicit SuiteGraph
2. resolve/validate all required CorpusBindings
3. resolve/validate all required ExecutionBindings
4. materialize the authoritative TestPlans
5. attach/validate applicable resource requirements and case metadata
6. resolve the local --file locator against authorized corpus source authority
7. select all authoritative CaseSpecs associated with that exact source entry
8. preserve canonical suite/plan order for the selected subset
9. run the ordinary D108 scheduler/execution path
10. use the ordinary aggregation/reporting/outcome model
```

The concrete host/controller mechanism that maps a local path to authorized
corpus source entries is implementation detail as long as it preserves this
contract and does not persist physical paths as logical identities.

No case is selected merely because a file exists.

## Metadata preservation

A selected CaseSpec retains its existing semantics unchanged, including where
present:

- CaseId;
- source/case path association;
- expectation and expected value;
- resource requirements;
- execution requirement;
- CorpusId/suite context;
- CaseAuthorityDescriptor;
- capability/provisioning requirements;
- failure classification;
- deterministic reporting position.

The selector does not reconstruct or synthesize those fields.

```text
CASE_METADATA_PRESERVED=EXACT
EXPECTATION_PRESERVED=YES
RESOURCE_REQUIREMENTS_PRESERVED=YES
EXECUTION_REQUIREMENT_PRESERVED=YES
CASE_AUTHORITY_PRESERVED=YES
```

## Failure model

The following are Test Tool selection/configuration failures, not guest test
failures:

- `--file` with no following value;
- duplicate `--file`;
- invalid or unsupported locator;
- selected file outside all authorized corpus source bindings;
- selected source file with zero authoritative CaseSpecs;
- source association that cannot be resolved deterministically;
- invalid authoritative plan/case metadata encountered before scheduling.

An ordinary expectation mismatch in a selected case remains an ordinary completed
test failure.

An infrastructure failure remains an infrastructure-aborted outcome according to
the existing D108/D114 contracts.

D151 introduces no new guest failure category.

## Existing CLI interaction

### `--jobs`

D069 remains unchanged.

For a selection containing one case:

```text
--jobs N
    -> logical capacity N
    -> only one selected case can be admitted
```

For a future file associated with several CaseSpecs, ordinary bounded scheduling
may admit those cases subject to their existing constraints.

### `--resource-catalog`

D099/D101 remain unchanged. A selected case retains its resource requirements and
uses the ordinary supplied catalog.

### Unknown arguments

The historical TOOL002 projection policy remains unchanged except that the exact
token `--file` becomes one newly recognized Test Tool option.

D151 does not use this work to redesign all argument compatibility.

## Future unit-testing library compatibility

D151 intentionally separates Test Tool selection from a future unit-testing
library.

A future library may decide how multiple logical unit tests are authored inside
one source/module. If that future architecture emits or owns several
authoritative CaseSpecs associated with one source file, D151 already gives the
natural file-level focal behavior:

```text
protos test --file FILE
    -> all authoritative tests associated with FILE
```

An eventual exact logical selector such as:

```text
protos test --case CASE_ID
```

remains an additive future option for selecting one logical case inside such a
file. D151 neither requires nor predefines that option.

## Portable distribution and remote execution

The file locator is resolved only at the local/controller selection boundary.

It is not required to travel to a local OS worker or future remote worker.

After selection, workers consume the existing logical/inert case plan:

```text
SuiteId
CorpusId
CaseId
CaseSpec metadata
resource/provisioning requirements
...
```

rather than the launcher's local absolute path.

Therefore:

```text
CHECKOUT_LOCATION_CHANGES_CASE_ID=NO
PORTABLE_DISTRIBUTION_CHANGES_CASE_ID=NO
REMOTE_PLACEMENT_CHANGES_CASE_ID=NO
FILE_LOCATOR_IS_REMOTE_IDENTITY=NO
```

A distribution/environment that cannot map the supplied local `FILE` to an
authorized current-invocation corpus source entry fails the file selector rather
than changing test identity.

## Comparative research

The D151 audit compared more than five credible systems across materially
different selection models.

### pytest

pytest supports file/module selection directly and uses collected node IDs to
refine selection down to class/function/parameterized cases. This demonstrates
that a file locator and a more precise logical/collected identity can coexist
without requiring the file to be the ultimate test identity.

Primary source:
https://docs.pytest.org/en/stable/example/markers.html

### Elixir ExUnit / Mix

`mix test` supports direct test-file selection and `FILE:LINE` rerun
ergonomics. This is strong evidence for file-based focal workflow as a developer
convenience while test names/modules remain separate semantic entities.

Primary source:
https://hexdocs.pm/mix/Mix.Tasks.Test.html

### Vitest

Vitest explicitly recommends narrowing by file to avoid loading unrelated test
files and separately supports test-name and file/line filtering. This reinforces
the performance/developer-ergonomics value of file-level selection.

Primary source:
https://vitest.dev/guide/filtering

### Rust Cargo/libtest

Cargo/libtest primarily filters logical test names and offers exact-name
selection independently of package/target selection. This is strong evidence for
a future logical `CaseId` selector, but it does not erase the immediate value of
a file locator for Protos' current source-oriented developer workflow.

Primary sources:
https://doc.rust-lang.org/cargo/commands/cargo-test.html
https://doc.rust-lang.org/rustc/tests/

### Go testing

Go's `-run` model filters hierarchical logical test/subtest names, showing a
mature one-container-to-many-logical-tests model and reinforcing why D151 must
not freeze one file to one case.

Primary sources:
https://pkg.go.dev/testing
https://go.dev/blog/subtests

### JUnit Platform

JUnit deliberately separates discovery selectors, including source/file-oriented
selectors and unique logical identifiers. This is the closest transferable
precedent for D151's core distinction:

```text
developer locator != durable logical test identity
```

Primary source:
https://docs.junit.org/current/running-tests/console-launcher.html

### Bazel

Bazel separates target selection from framework-owned internal test filtering.
This reinforces Protos' existing separation between explicit suite/corpus
membership and later focal case selection.

Primary source:
https://bazel.build/docs/user-manual

## Candidate comparison

The final surviving candidates were:

- A — positional exact file selector;
- B′ — explicit `--file FILE` locator over authoritative CaseSpecs;
- C — exact logical `--case CaseId` selector as the initial feature;
- F — status quo / no public focal selector.

Candidate D (mandatory SuiteId + CaseId qualification) was rejected as
disproportionate to the current need. Candidate E (arbitrary source becomes a
test) conflicts with explicit Test Tool membership and metadata authority.

Scores are 1–5. Confidence is H/M/L.

| Criterion | A | B′ | C | F |
| --- | ---: | ---: | ---: | ---: |
| correctness / invariant preservation | 4/M | **5/H** | **5/H** | **5/H** |
| Protos alignment | 4/M | **5/H** | **5/H** | 4/H |
| present-need proportionality | **5/H** | 4/H | 3/H | 2/H |
| incremental growth | 3/M | **5/H** | **5/H** | 3/M |
| future-option resilience | 3/M | **5/H** | **5/H** | 4/H |
| scalability | **5/H** | **5/H** | **5/H** | 2/M |
| conceptual simplicity | **4/H** | 4/H | 4/H | **5/H** |
| portability / implementation freedom | 3/M | 4/M | **5/H** | **5/H** |
| runtime / resource cost | **5/H** | **5/H** | **5/H** | 1/H |
| failure / operability | 3/M | **5/H** | **5/H** | **5/H** |
| deferral / reversibility / migration | 3/M | **5/H** | 4/H | 2/H |
| evidence maturity / implementation risk | **5/H** | **5/H** | **5/H** | **5/H** |
| informational total / 60 | 47 | **57** | 56 | 43 |

Arithmetic did not decide the result. The decisive distinction is architectural
and ergonomic:

- A is shorter but would reinterpret the Test Tool's historically ignored
  positional namespace.
- C is logically pure but forces the developer/agent who already has a source
  path to translate that path into current CaseId conventions, which are not
  uniform across all corpora.
- B′ spends one explicit option name to keep local locator syntax separate from
  durable identity and preserves an additive path to a future `--case`
  selector.

## Smallest-sufficient analysis

B′ adds exactly:

- one option;
- one exact local-file locator;
- one authoritative case-source resolution step;
- one selected subset of already-built plans.

It does not add:

- globbing;
- regex filtering;
- directory selection;
- tag expressions;
- file/line selection;
- watch mode;
- failed-test history;
- test-result cache;
- new discovery;
- a new runner;
- a generalized selector DSL.

This satisfies the current demonstrated need while preserving additive future
growth.

## Strongest argument against B′

The strongest objection is that Protos deliberately invested in logical
`SuiteId`, `CorpusId` and `CaseId` separation; accepting a filesystem path
could appear to reintroduce physical identity into the Test Tool.

B′ answers that objection by terminating the path at the controller-local
selection boundary. The path is a locator only. It is neither persisted nor
transported as Test Tool identity.

A future `--case CaseId` remains the cleaner portable exact-logical selector
when such a need becomes concrete.

## Regret scenario and escape path

A future Test Tool may contain many cases that are generated, parameterized,
remote-only, database-backed, or otherwise not naturally selectable by one
physical source file.

B′ does not claim universality. In that future:

- `--file` remains useful for file-backed cases;
- `--case CaseId` can be added for exact logical selection;
- suite-qualified selectors can be added if real ambiguity appears;
- source-free/generated cases remain selectable through their logical identity.

No existing CaseId, CorpusId or SuiteId needs to change.

## Anti-overengineering and underengineering checks

```text
PAY_FOR_WHAT_YOU_NEED=PASS
GROW_AS_YOU_NEED=PASS
STATUS_QUO_COMPARED=PASS
SMALLEST_SUFFICIENT=PASS
FUTURE_STRESS_TEST=PASS
REVERSIBILITY_ANALYZED=PASS
STRONGEST_COUNTERARGUMENT_RECORDED=PASS
```

B′ is not underengineered because it does not derive membership from file
existence, does not drop metadata, and does not make physical paths semantic
identity.

B′ is not overengineered because richer filters and future logical selectors are
deliberately deferred.

## Invariant/delta consistency

The selected B′ contract was checked against the applicable owner-approved
invariants.

```text
D122_EXPLICIT_MEMBERSHIP=PASS
D122_RECURSIVE_AUTO_DISCOVERY_NO=PASS
D122_PLAN_BEFORE_SCHEDULING=PASS
D123_SUITE_ID_NOT_PATH=PASS
D125_EXECUTION_REQUIREMENT_SEPARATION=PASS
D126_CORPUS_ID_NOT_PATH=PASS
D126_CORPUS_BINDING_SOURCE_AUTHORITY=PASS
D126_ALL_PLANS_BEFORE_SCHEDULING=PASS
D132_CASE_ID_NOT_FILESYSTEM_PATH=PASS
D133_PHYSICAL_PATH_NOT_CASE_ID=PASS
D133_CASE_AUTHORITY_LIFECYCLE=PASS
D069_JOBS_UNCHANGED=PASS
D099_D101_RESOURCE_CATALOG_UNCHANGED=PASS
D108_D114_FAILURE_MODEL_UNCHANGED=PASS
FULL_VALIDATION_POLICY_UNCHANGED=PASS
```

The refinement from "exactly one CaseSpec" to "all authoritative CaseSpecs
associated with the exact file" introduces no contradiction with prior approved
invariants. It removes an unnecessary assumption about future test-container
cardinality.

```text
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Exact selected contract

```text
D151_SELECTED_CANDIDATE=B_PRIME

CLI_FORM=protos test --file FILE
FILE_SELECTOR_OCCURRENCES=0_OR_1
ATTACHED_FILE_OPTION_FORM=NOT_SELECTED_INITIAL
POSITIONAL_FILE_SELECTOR=NO

SELECTOR=EXACT_LOCAL_FILE_LOCATOR
FILE_IS_TEST_IDENTITY=NO
FILE_IS_CASE_ID=NO
FILE_IS_CORPUS_ID=NO
FILE_IS_SUITE_ID=NO
FILE_IS_EXECUTION_REQUIREMENT_ID=NO
FILE_IS_PERSISTED=NO
FILE_IS_REMOTE_WORKER_IDENTITY=NO

SELECTION_AUTHORITY=EXISTING_AUTHORITATIVE_CASESPEC
ARBITRARY_SOURCE_CREATES_TEST=NO

FILE_SELECTION=ALL_AUTHORITATIVE_CASES_ASSOCIATED_WITH_EXACT_FILE
ZERO_MATCHES=TEST_TOOL_SELECTION_ERROR
ONE_MATCH=RUN_ONE_CASE
MULTIPLE_MATCHES=RUN_ALL_MATCHING_CASES_IN_CANONICAL_PLAN_ORDER
PHYSICAL_PATH_DEDUPLICATION=NO

RELATIVE_FILE_BASE=INVOCATION_CWD
ABSOLUTE_FILE=ALLOWED
DIRECTORY_SELECTOR=NO
GLOB_SELECTOR=NO
REGEX_SELECTOR=NO
LINE_SELECTOR=NO
IMPLICIT_EXTENSION=NO
SEARCH_PATH_FALLBACK=NO
SYMLINK_REPARSE_ALIAS_INITIAL=NO

SUITE_GRAPH_RESOLUTION=UNCHANGED
ALL_REQUIRED_BINDINGS_VALIDATED=YES
TESTPLANS_MATERIALIZED_BEFORE_SCHEDULING=YES
RESOURCE_REQUIREMENTS_ATTACHED_BEFORE_SELECTION=YES
SELECTION_BEFORE_CASE_SCHEDULING=YES

CASE_METADATA_PRESERVED=EXACT
EXPECTATION_PRESERVED=YES
RESOURCE_REQUIREMENTS_PRESERVED=YES
EXECUTION_REQUIREMENT_PRESERVED=YES
CASE_AUTHORITY_PRESERVED=YES

JOBS_SEMANTICS_CHANGED=NO
RESOURCE_CATALOG_SEMANTICS_CHANGED=NO
RESULT_CLASSIFICATION_CHANGED=NO
UNKNOWN_ARGUMENT_POLICY=UNCHANGED_EXCEPT_EXACT_--file

MULTIPLE_CASES_PER_FILE_SUPPORTED_BY_SELECTION_MODEL=YES
MULTIPLE_FILE_SELECTION=DEFERRED
CASE_ID_SELECTOR=DEFERRED_ADDITIVE
SUITE_QUALIFIER=DEFERRED_ADDITIVE
FILE_LINE_SELECTOR=DEFERRED_ADDITIVE

FULL_VALIDATION_POLICY_CHANGED=NO
LANGUAGE_SEMANTICS_CHANGED=NO
STANDARD_LIBRARY_SEMANTICS_CHANGED=NO
```

## Implementation routing

D151 authorizes implementation of this exact contract only after durable
ratification is complete.

Implementation belongs in a separate Tool work item. The implementation must
reuse the existing Test Tool planning/scheduling/execution pipeline rather than
create a second runner.

The consuming work may add the smallest required host/controller file-resolution
mechanism, Test Tool option projection, plan-subset filtering and tests. If
implementation exposes a materially new architecture axis, it must stop at the
normal decision gate.

Any AGENTS.md guidance teaching agents to use `protos test --file FILE` as a
focal gate must wait until that executable surface is actually published.

## Closure contract

```text
D151_STATUS=RATIFIED
D151_SELECTED_CANDIDATE=B_PRIME

PROTOS_REVISION=d1bbab2c1c1023e980b43ca01e7b2adafcac05f8
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs

SPECIFICATION_CHANGED=NO
IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
STANDARD_LIBRARY_CHANGED=NO

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

D151 ratification is governance/documentation-only.
