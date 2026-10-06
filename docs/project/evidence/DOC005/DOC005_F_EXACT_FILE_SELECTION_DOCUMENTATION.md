# DOC005-F — Test Tool exact file-backed focal-selection documentation

Date: 2026-10-06

## Work identity

~~~text
PARENT_WORK_ITEM=DOC005
PARENT_ISSUE=guillermomolina/protos#448
SLICE=DOC005-F
SLICE_ISSUE=guillermomolina/protos#597
SOURCE_IMPLEMENTATION=TOOL008/guillermomolina/protos#591
DESIGN_AUTHORITY=D151/guillermomolina/protos#590
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative documentation evidence. It does not replace
the live GitHub coordination state or any owning Test Tool/design authority.

## Exact published product state

~~~text
PROTOS_REVISION=fc9aca90f479051964d8d56c11a2778386789671
COMMIT_SUBJECT=DOC005-F: document Test Tool exact file-backed focal selection
PRODUCT_PUBLICATION=PUSHED
CHANGED_PRODUCT_PATH=docs/guide/tools/test-tool.md
SPECIFICATION_CHANGE=NO
IMPLEMENTATION_CHANGE=NO
~~~

The published change extends the maintained Test Tool guide rather than creating
a second documentation surface.

## Published documentation outcome

The guide now documents the current exact file-backed focal-selection surface:

~~~text
bin/protos test --file FILE
~~~

The published explanation establishes that:

- the selected public spelling is the separate-token `--file FILE` form;
- a file path is an invocation-local locator, not logical Case, corpus, suite,
  execution-requirement, persistence, or remote-worker identity;
- relative paths are interpreted against the invocation working directory and
  absolute paths are accepted;
- file selection filters already-authoritative Test Tool plans and does not turn
  an arbitrary existing `.protos` source into a test;
- zero authoritative matches fail before scheduling;
- one matching Case selects that Case;
- several matching Cases for one source are all retained as distinct logical
  Cases;
- repeated `--file FILE` occurrences are supported by the current Test Tool;
  each occurrence must match, their matches form a union, duplicate selection
  does not duplicate execution, and canonical plan order is preserved;
- selected Cases retain their existing expectations, resources, execution
  authority, fresh-Process isolation, result classification, progress, and exit
  behavior; and
- later `--directory`, `--case`, and `--list-cases` surfaces are acknowledged
  as separate current capabilities rather than incorrectly described as absent.

The executable example documented by the published revision is:

~~~text
bin/protos test --file protos/tests/library/uri/parse.protos
~~~

That source currently contributes several logical Cases, which makes the
physical-file-versus-logical-Case distinction visible in one concrete example.

## Reconciliation with the original DOC005-F allocation

The original #597 text reflected the earlier TOOL008 publication boundary and
listed multiple-file and directory/Case selection as deferred.

By the time DOC005-F was implemented, later Test Tool work had already extended
that surface. The documentation therefore reconciled against current published
behavior instead of copying the historical negative list.

This is a documentation reconciliation only:

~~~text
TOOL008_D151_FILE_LOCATOR_INVARIANT=PRESERVED
CURRENT_REPEATABLE_FILE_SELECTION=DOCUMENTED
LATER_SELECTOR_SURFACES_REDESIGNED_BY_DOC005_F=NO
TEST_TOOL_SEMANTICS_CHANGE=NO
~~~

## Validation and publication evidence

After the product commit was pushed, the maintainer reported:

~~~text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
PRODUCT_PUBLICATION=PUSHED
~~~

The available exact-commit combined-status interface exposes no status entries
for this documentation commit:

~~~text
CI_HEAD=fc9aca90f479051964d8d56c11a2778386789671
COMBINED_STATUS_ENTRIES=0
~~~

DOC005-F is documentation-only and its Issue does not require an independent
remote-CI publication gate after the documented command/current behavior and
documentation checks are satisfied.

## Closure assessment

The published change plus maintainer-reported validation satisfies the DOC005-F
closure criteria:

~~~text
TEST_TOOL_FILE_SELECTION_DOCUMENTED=PASS
LOCAL_FILE_VS_TEST_IDENTITY_DISTINCTION=PASS
ZERO_ONE_MANY_CASE_BEHAVIOR=PASS
CURRENT_REAL_EXAMPLE=PASS
CURRENT_SELECTOR_SURFACE_RECONCILED=PASS
DOCUMENTATION_VALIDATION=PASS
SPECIFICATION_CHANGE=NO
IMPLEMENTATION_CHANGE=NO
DOC005_F_COMPLETE=YES
ISSUE_597_CLOSURE_AUTHORIZED=YES
NEXT_DOC005_F_SLICE=NONE
~~~

The parent DOC005 remains independently open. TOOL006/#473 has already restored
the resource-requirements wiring that had blocked DOC005-C, so DOC005-C is
released for its documentation work; DOC005-E remains dependent on completing C
and the final current-surface consistency pass.

AI assistance: this durable closure evidence was drafted with ChatGPT from the
exact published Protos commit, the live DOC005-F/DOC005/TOOL006 GitHub state,
and maintainer-reported local validation.
