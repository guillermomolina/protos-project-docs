# AUD013-B1 — Library-test Assertions adoption evidence

## Status

```text
WORK_ITEM=AUD013/#581
SLICE=AUD013-B1
TYPE=IMPLEMENTATION
PRODUCT_REPOSITORY=guillermomolina/protos
PUBLISHED_REVISION=c8f61ae87ef6a26f8525b9d24f5ad466b665a29c
SUBJECT=AUD013-B1: adopt Assertions in library tests
RESULT=PASS
```

AUD013-B1 consumes the TEST003 / LIB016 assertion-adoption handoff inside the
maintained Standard Library test partition. This record is non-normative and
does not extend the public Assertions contract.

## Published change

The published Protos commit modifies 21 files, all under:

```text
protos/tests/library/**/*.protos
```

No production source, `pom.xml`, or `CHANGELOG.md` is part of the commit.

The exact changed files are:

```text
protos/tests/library/cli/closure.protos
protos/tests/library/cli/help.protos
protos/tests/library/cli/parse.protos
protos/tests/library/cli/specification-and-result.protos
protos/tests/library/cli/subcommands.protos
protos/tests/library/csv/encode.protos
protos/tests/library/csv/integration-and-scale.protos
protos/tests/library/csv/parse.protos
protos/tests/library/csv/row-parser.protos
protos/tests/library/csv/text-adapter.protos
protos/tests/library/math/integer/factorial.protos
protos/tests/library/math/integer/gcd-lcm.protos
protos/tests/library/math/integer/integrated-closure.protos
protos/tests/library/math/integer/power.protos
protos/tests/library/network/ip-addresses/parse-format.protos
protos/tests/library/network/ip-addresses/surface.protos
protos/tests/library/network/ip-addresses/validation.protos
protos/tests/library/network/ip-endpoints/parse-format-and-validation.protos
protos/tests/library/uri/format.protos
protos/tests/library/uri/parse.protos
protos/tests/library/uri/resolve.protos
```

Diff-level reconciliation of the published commit records:

```text
REMOVED_LOCAL_REQUIRE_DEFINITIONS=68
REMOVED_LOCAL_REJECT_DEFINITIONS=35
ADDED_ASSERTIONS_REQUIRE_CALLS=655
ADDED_ASSERTIONS_SIGNALS_CALLS=304
```

The removed helpers are the TEST003-A-proven legacy shapes: trivial Boolean
`require` helpers and expected-Error `reject` helpers consumed only through
the Boolean assertion compound. Their call sites now use the ratified
`std:test/Assertions` surface.

The migration preserves intentional test oracles such as explicit null checks,
identity checks, exact values, formatting results and domain predicates; those
oracles are now arguments to `Assertions.require` rather than being rewritten
into unrelated language features.

## Authority consumed

```text
ASSERTIONS_API_OWNER=LIB016/#557
ASSERTION_ADOPTION_POLICY_OWNER=TEST003/#562
SEMANTIC_CLASSIFICATION_CONTRACT=TEST003-A/#563
NEW_DUPLICATION_PREVENTION=TEST003-D/#566
LEGACY_MIGRATION_OWNER=AUD013/#581
```

The ratified public surface remains:

```text
std:test/Assertions
Assertions.AssertionFailure
Assertions.require(condition)
Assertions.signals(errorPrototype, body)
```

No richer testing API, framework, fixture model, registry, Test Tool semantic
change, language syntax or runtime behavior is introduced.

## Validation

The maintainer reports for the published candidate:

```text
GIT_DIFF_CHECK=PASS
FULL_LOCAL_REQUIRED_TESTS=PASS
PUBLICATION=PUSHED
```

No exact test-count claim is added here because the maintainer handoff supplied
the pass result but not a numeric suite count.

## AUD013 closure attempt

AUD013-B1 is a successful bounded implementation slice, but it is not sufficient
to close AUD013/#581.

The parent closure contract requires all maintained Protos source surfaces to be
audited. B1 modified only the Standard Library test partition. Current HEAD
still contains maintained source in other required surfaces, including:

```text
protos/lib/**
protos/tools/**
protos/tests/conformance/**
protos/tests/tooling/**
protos/tests/package-tool/**
protos/examples/**
other maintained Protos source discovered by the repository inventory
```

Current post-B1 search evidence also shows still-unclassified candidates outside
the B1 partition, for example:

```text
protos/lib/collections/Range.protos
    contains the historical Integer-domain probe spelling div(1)

protos/tools/test/**
    contains Map containsKey usage requiring semantic classification

protos/tests/tooling/tool002-lib011-options-adoption.protos
    contains legacy require/reject-shaped text requiring classification

protos/tests/conformance/network/ip-address.protos
protos/tests/conformance/network/ip-endpoint.protos
    contain reject-shaped expected-error machinery requiring classification

protos/examples/collections/maps.protos
    contains containsKey usage requiring classification
```

These are discovery candidates, not declarations that the code is wrong.
Each must be classified under the AUD013 manifest before the parent can claim:

```text
MAINTAINED_PROTOS_SOURCE_SURFACES=AUDITED
BASELINE_OBLIGATIONS=CHECKED_IN_SINGLE_INTEGRATED_PASS
FINAL_RESCAN=NO_UNCLASSIFIED_LEGACY_PATTERNS
CURRENT_PROTOS_BASELINE=ESTABLISHED
```

Therefore:

```text
AUD013_B1=COMPLETE
TEST003_ASSERTION_ADOPTION_LIBRARY_TEST_PARTITION=COMPLETE
AUD013_PARENT_CLOSURE=NOT_YET_JUSTIFIED
ISSUE_581_MUST_REMAIN_OPEN=YES
```

## Recommended next slice

The next bounded surface should be the maintained Standard Library production
source because AUD013-A already identified concrete active-manifest candidates
there.

```text
NEXT_SLICE=AUD013-B2
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
SURFACE=protos/lib/**/*.protos
PURPOSE=integrated Standard Library production-source adoption sweep
```

AUD013-B2 must evaluate all active manifest entries per source file, not run a
feature-by-feature whole-tree migration. In particular it must semantically
classify the retained `Range.protos` domain probes and any D142/D143/D144
candidates, while preserving genuine membership checks, validation branches,
custom access patterns and other still-canonical forms.
