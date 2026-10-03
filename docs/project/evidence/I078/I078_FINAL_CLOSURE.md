# I078 — final closure evidence

Date: 2026-10-03

## Work identity

~~~text
WORK_ITEM=I078
PROTOS_ISSUE=guillermomolina/protos#764
DECISION_AUTHORITY=D180/guillermomolina/protos#762
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
DOWNSTREAM_CONSUMER=WEB009/guillermomolina/protos-website#5
~~~

This record is durable non-normative project evidence. It does not redefine
Protos semantics or replace the live GitHub Issue state.

## Final publication

The final publication-reconciliation commit is:

~~~text
PROTOS_REVISION=5b2c7d5baa402677f8fe6bf2ca3a19859d4bc475
PROTOS_PARENT=cfc0fb433e82f0478c9fff9cc965c3fc506fabc9
COMMIT_SUBJECT=I078: reconcile whileTrue publication metadata
IMPLEMENTATION_VERSION=0.3.170-SNAPSHOT
SPECIFICATION_REVISION=0.1.437
~~~

The final commit changes exactly:

~~~text
CHANGELOG.md
pom.xml
spec/PROTOS_SPEC_CHANGELOG.md
~~~

No runtime, library, tool, benchmark, conformance, Java test, guide or historical
specification-archive file is changed by the closure commit.

## Reconciled implementation publication

The implementation changelog now records D180 Candidate B / I078 explicitly:

- the standard Closure pre-test loop selector is `whileTrue`;
- the standard `while` selector is removed with no compatibility alias;
- no standard `whileFalse` selector is added;
- every non-name D044 semantic remains unchanged;
- ordinary lookup, shadowing, extraction and override remain ordinary;
- user-defined selectors named `while` remain valid ordinary members;
- no dedicated loop syntax, truthiness or selector intrinsic is introduced.

The Maven development version advances atomically from
`0.3.169-SNAPSHOT` to `0.3.170-SNAPSHOT`.

## Reconciled normative publication

The live specification changelog advances from `0.1.436` to `0.1.437`.

Revision `0.1.437` records the already-published D180 normative change in the
affected domains:

- `spec/semantics/EXECUTION_AND_CONTROL.md`;
- `spec/semantics/CALLABLES.md`;
- `spec/PROTOS_GRAMMAR.md`;
- `spec/concurrency/FUTURES_AND_TASKS.md`.

It records exactly one standard pre-test loop selector,
`condition.whileTrue(body)`, with no standard `while` alias and no standard
inverse `whileFalse`. It also records that `while` and `whileFalse` remain
ordinary non-reserved member names.

No archived file under `spec/changelog/` was modified.

## Substantive implementation authority

The substantive I078 migration was already present before the closure commit.
The earlier audit established that current product state had:

~~~text
STANDARD_WHILETRUE_PRESENT=YES
STANDARD_WHILE_PRESENT=NO
STANDARD_WHILEFALSE_PRESENT=NO
USER_DEFINED_WHILE_REMAINS_ORDINARY=YES
STALE_STANDARD_DOT_WHILE_CALLS=NONE
~~~

The remaining old `.while(` occurrences were intentional historical archive
text and the conformance case that deliberately defines a user-owned ordinary
`while` selector.

The final closure commit does not alter executable or test content, so it does
not invalidate that substantive evidence.

## Validation provenance

Before this metadata/specification-only reconciliation, exact parent revision
`cfc0fb433e82f0478c9fff9cc965c3fc506fabc9` was the final PERF025 product
authority and had maintainer-reported integrated validation:

~~~text
MAKE_TEST=PASS
PROTOS_TESTS=1284_PASSED_0_FAILED
~~~

The I078 final reconciliation changed only the three publication metadata /
changelog files listed above. The maintainer reviewed, committed and pushed the
final diff successfully. No executable or test content changed, so repeating the
full integrated suite was not required solely for this closure commit.

## Closure

The publication-coherence blockers recorded by the earlier
`I078_CLOSURE_RECONCILIATION_AUDIT.md` are now resolved:

~~~text
IMPLEMENTATION_CHANGELOG_RECONCILIATION=PASS
IMPLEMENTATION_VERSION_RECONCILIATION=PASS
SPEC_CHANGELOG_RECONCILIATION=PASS
SPEC_REVISION_RECONCILIATION=PASS
RUNTIME_OR_SEMANTIC_WORK_PENDING=NO
TEST_WORK_PENDING=NO
GUIDE_WORK_PENDING=NO
HISTORICAL_SPEC_ARCHIVES_CHANGED=NO
I078_CLOSURE_AUTHORIZED=YES
~~~

I078 / #764 can therefore close completed at exact Protos revision
`5b2c7d5baa402677f8fe6bf2ca3a19859d4bc475`.

WEB009 no longer has an I078 source-coherence blocker. Its next implementation
may re-audit current Protos state and select an exact source revision according
to its own acceptance criteria.

~~~text
I078_STATUS=CLOSED_COMPLETED
FINAL_PRODUCT_REVISION=5b2c7d5baa402677f8fe6bf2ca3a19859d4bc475
IMPLEMENTATION_VERSION=0.3.170-SNAPSHOT
SPEC_REVISION=0.1.437
WEB009_I078_BLOCKER=RESOLVED
~~~
