# I078 — closure reconciliation audit

Date: 2026-10-03

## Work identity

~~~text
WORK_ITEM=I078
PROTOS_ISSUE=guillermomolina/protos#764
DECISION_AUTHORITY=D180/guillermomolina/protos#762
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
DOWNSTREAM_BLOCKER=WEB009/guillermomolina/protos-website#5
~~~

This record is durable non-normative project evidence. It does not redefine
Protos semantics or replace the live GitHub Issue state.

## Published implementation state

The language-visible and repository-owned source migration was published in:

~~~text
I078_IMPLEMENTATION_REVISION=8ea87fb1794599247f0e95fd5570d7dd7e08a52f
I078_IMPLEMENTATION_PARENT=57d8cf4ec195aca3cb5b7c33d37755d965e9f3df
PUBLISHED_COMMIT_SUBJECT=PERF025-C1 checkpoint: inline Object-body execution (C1a+C1b)
~~~

The published commit subject is unrelated to the I078 portion of that commit,
but the delta includes the I078 normative, runtime, library/tool, conformance,
fixture and guide migration.

At the audited current branch head:

~~~text
PROTOS_HEAD=cfc0fb433e82f0478c9fff9cc965c3fc506fabc9
PROTOS_HEAD_SUBJECT=PERF025: specialize residual inline callback Activation consumers
POM_VERSION=0.3.169-SNAPSHOT
~~~

The current product state publishes the canonical standard loop selector as
`whileTrue`, removes the standard `while` alias, and does not publish
`whileFalse`.

## Stale-name audit

A default-branch code search for the old call spelling `.while(` leaves only
two intentional categories:

1. historical specification changelog text under
   `spec/changelog/PROTOS_SPEC_CHANGELOG-0.1.300-0.1.399.md`, which is archived
   evidence and must not be rewritten by ordinary work; and
2. the conformance case in
   `protos/tests/conformance/control/while-basic-and-validation.protos` that
   deliberately defines and calls a user-owned ordinary selector named
   `while` to prove that the removed standard spelling remains available to
   programs as an ordinary member name.

Current conformance and architecture coverage also explicitly assert:

~~~text
STANDARD_WHILETRUE_PRESENT=YES
STANDARD_WHILE_PRESENT=NO
STANDARD_WHILEFALSE_PRESENT=NO
USER_DEFINED_WHILE_REMAINS_ORDINARY=YES
~~~

No stale repository-owned call site using the removed standard selector was
found.

## Normative state

The current normative owners already state the selected D180 surface, including:

- `spec/semantics/EXECUTION_AND_CONTROL.md`: `whileTrue` is the only standard
  pre-test loop selector, with no standard `while` alias and no standard
  `whileFalse`;
- `spec/semantics/CALLABLES.md`: the standard Closure-specific protocol lists
  `whileTrue`;
- `spec/PROTOS_GRAMMAR.md`: examples use `whileTrue` while `while` remains
  non-reserved;
- `spec/concurrency/FUTURES_AND_TASKS.md`: synchronous nested loop activation
  terminology uses `whileTrue`.

However, the live specification changelog still has:

~~~text
CURRENT_SPEC_REVISION=0.1.436
I078_SPEC_CHANGELOG_ENTRY=ABSENT
~~~

Because I078 changed normative files in the published implementation commit,
the specification revision discipline is not yet reconciled.

## Implementation changelog/version state

The I078 implementation commit did not change `pom.xml` or `CHANGELOG.md`.
Subsequent unrelated work advanced the Maven development version to
`0.3.169-SNAPSHOT`, but there is still no implementation changelog entry that
records I078's public protocol rename.

Therefore:

~~~text
CURRENT_IMPLEMENTATION_VERSION=0.3.169-SNAPSHOT
I078_IMPLEMENTATION_CHANGELOG_ENTRY=ABSENT
I078_ATOMIC_VERSION_RECONCILIATION=NOT_RECORDED
~~~

The absence cannot be repaired merely by observing that unrelated later work
consumed higher versions.

## Validation evidence already available

The current exact product head is also the final PERF025 product authority.
Maintainer-reported integrated validation recorded on
`guillermomolina/protos#758` for this exact revision states:

~~~text
PROTOS_REVISION=cfc0fb433e82f0478c9fff9cc965c3fc506fabc9
MAKE_TEST_REPORTED_BY_MAINTAINER=PASS
PROTOS_TESTS=1284_PASSED_0_FAILED
~~~

This full suite includes the migrated standard-loop conformance and Java
architecture/semantic coverage now present on main. I078 therefore does not need
another expensive integrated suite solely to re-prove unchanged product code.

The final metadata/specification reconciliation must still run its applicable
cheap checks after the final edit, including `git diff --check` and the current
specification/version/changelog checks selected by the repository instructions.
Do not repeat the full suite unless the reconciliation changes executable/test
content or another policy trigger requires it.

## Exact remaining reconciliation

I078 is not yet closable at this audit revision.

The remaining product change is bounded to publication metadata and changelog
authority. Immediately before the human-executed product commit, derive moving
version/revision numbers from the then-current `origin/main`.

If no intervening product publication occurs, the expected reconciliation is:

~~~text
NEXT_IMPLEMENTATION_VERSION=0.3.170-SNAPSHOT
NEXT_SPEC_REVISION=0.1.437
~~~

The product reconciliation must:

1. add a `CHANGELOG.md` entry that explicitly records I078 / D180 Candidate B:
   standard Closure `while` renamed to `whileTrue`, no compatibility alias,
   no `whileFalse`, D044 non-name semantics and ordinary lookup preserved;
2. advance the Maven development version atomically with that implementation
   changelog entry;
3. add the next live `spec/PROTOS_SPEC_CHANGELOG.md` revision describing the
   already-published normative rename and naming the affected normative
   documents;
4. run the applicable final metadata/spec checks and `git diff --check`;
5. publish the human-executed Protos reconciliation commit; and
6. record that exact product revision on #764 before closing it completed.

No runtime, library, test, guide, or semantic implementation work is currently
identified as missing.

## WEB009 consequence

WEB009 remains blocked only by this I078 publication-coherence reconciliation.
Once #764 closes on the exact reconciled product revision, WEB009 can re-audit
and select that exact source revision for its one-shot canonical-source refresh.

~~~text
I078_IMPLEMENTATION_SURFACE=COMPLETE
I078_RUNTIME_SOURCE_MIGRATION=COMPLETE
I078_NORMATIVE_TEXT=COMPLETE
I078_STALE_STANDARD_WHILE_AUDIT=PASS
I078_CURRENT_HEAD_FULL_SUITE=PASS
I078_IMPLEMENTATION_CHANGELOG_RECONCILIATION=PENDING
I078_SPEC_REVISION_RECONCILIATION=PENDING
I078_STATE=CLOSURE_PENDING
WEB009_STATE=BLOCKED_BY_I078_RECONCILIATION
~~~
