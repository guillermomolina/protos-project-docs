# DOC007-G — Final authority and integrity audit

Status: **CLOSED on publication**

Live coordination: GitHub `DOC007 / #532`

Validation class: `DOCUMENTATION_ARCHITECTURE_AND_REPOSITORY_MIGRATION`

## Purpose

DOC007-G performs the final authority and integrity audit after the durable
`docs/project/**` corpus has been extracted into
`guillermomolina/protos-project-docs`, active references have been cut over, the
cross-repository publication contract has been established, and the duplicate
live corpus has been removed from `guillermomolina/protos`.

This phase validates the owner-approved AUD010 Candidate B-prime architecture.
It does not introduce a new governance model.

## Revisions audited

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=756e4cd750ffec6de8e21aa604b21abb546b0a33

PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
PROJECT_RECORD_REVISION_E=ab3e50d914c859960c806047fe67cf0c88798e92
```

`PROTOS_REVISION` is the published DOC007-F source revision that removes the
migrated source corpus.

`PROJECT_RECORD_REVISION_E` is the published DOC007-E durable record revision
that closes the cross-repository publication-contract cutover.

The final revision of this DOC007-G record is captured in live Issue `#532`
after publication because a commit cannot contain its own final SHA.

## 1. Operational authority

Result:

```text
CONTROL_PLANE=guillermomolina/protos
LIVE_WORK_AUTHORITY=protos GitHub Issues
SCHEDULING=Protos Development Project
FORMAL_IDENTIFIER_AUTHORITY=protos
OWNER_APPROVAL_AUTHORITY=protos workflow
RESULT=PASS
```

The documentation repository does not allocate, select, close, or reinterpret
formal Protos identifiers independently.

Publication of a durable record in `protos-project-docs` is not itself owner
approval.

No Issue migration occurred.

No formal-ID federation was introduced.

## 2. Normative authority

Result:

```text
NORMATIVE_AUTHORITY=guillermomolina/protos/spec
PROJECT_RECORDS_NORMATIVE=NO
RESULT=PASS
```

The extracted corpus remains durable and non-normative.

No language or Standard Library semantic authority moved into
`protos-project-docs`.

## 3. Durable project-record authority

Result:

```text
DURABLE_PROJECT_RECORD_AUTHORITY=guillermomolina/protos-project-docs
CANONICAL_CORPUS=docs/project/**
ROLE_FIRST_INFORMATION_ARCHITECTURE=PRESERVED
RESULT=PASS
```

The destination repository preserves the established role-first hierarchy:

```text
docs/project/
  work/
  decisions/
  architecture/
  governance/
  registries/
  evidence/
  history/
```

The destination repository is now the single live canonical location for the
migrated durable project-record corpus.

## 4. Duplicate live corpus removal

DOC007-F published:

```text
PROTOS_REVISION=756e4cd750ffec6de8e21aa604b21abb546b0a33
```

At that revision, the source repository no longer contains live
`docs/project/**`.

Result:

```text
SOURCE_LIVE_DOCS_PROJECT_PRESENT=NO
DESTINATION_CANONICAL_CORPUS_PRESENT=YES
DUPLICATE_AUTHORITATIVE_CORPUS=NO
RESULT=PASS
```

Repository history was not rewritten. Earlier Protos commits and tags naturally
retain the historical in-repository paths that existed at those revisions.

## 5. Maintained cross-repository references

DOC007-D converted maintained references in active product documentation and
policy surfaces to `guillermomolina/protos-project-docs`.

The final source-side audit after staging DOC007-F found remaining
`docs/project/` strings in the following valid classes:

```text
DESTINATION_REPOSITORY_PATH_OR_URL
HISTORICAL_CHANGELOG_OR_SPEC_CHANGELOG
HISTORICAL_RELEASE_ARCHIVE_CONTRACT
TEST_FIXTURE_LOCAL_TEMPORARY_TREE
EXPLICIT_TEXT_DESCRIBING_DESTINATION_ROLE_FIRST_PATHS
```

No remaining inspected occurrence was an active locator that expected the
deleted source `docs/project/**` tree to exist.

Result:

```text
ACTIVE_SOURCE_LOCAL_PROJECT_RECORD_LINKS=NONE_FOUND
MAINTAINED_DESTINATION_LINKS=PASS
HISTORICAL_LITERALS=PRESERVED
RESULT=PASS
```

## 6. Release tooling and artifact metadata

DOC007-E removed the remaining live source-repository coupling required before
source deletion.

Release helpers now require the durable selection path explicitly instead of
defaulting to a source-local `docs/project/**` record.

`dist/README.md` documents a separately selected clean
`protos-project-docs` checkout at an exact project-record revision.

New portable `RUNTIME.txt` metadata uses revision-bound durable evidence in
`protos-project-docs`.

Historical archive-verifier literals remain unchanged where changing them would
falsify an already-persisted release artifact contract.

Result:

```text
LIVE_RELEASE_SELECTION_SOURCE_COUPLING=REMOVED
PORTABLE_RUNTIME_EVIDENCE_SOURCE_COUPLING=REMOVED
HISTORICAL_ARCHIVE_CONTRACT=PRESERVED
RESULT=PASS
```

## 7. Cross-repository publication and closure contract

The operative closure tuple is:

```text
PROTOS_REVISION=<exact SHA>
PROJECT_RECORD_REVISION=<exact SHA>
CROSS_REFERENCES=PASS
REQUIRED_DURABLE_PUBLICATION=PASS
```

For implementation plus durable closure evidence:

```text
product revision publishes
  -> durable record names exact product revision
  -> exact durable record revision is captured
  -> live Issue verifies both revisions
  -> Issue may close
```

If required durable publication is absent or contradictory, the live Issue
remains open.

Result:

```text
MOVING_MAIN_SUFFICIENT_FOR_REVISION_BOUND_EVIDENCE=NO
FAIL_CLOSED_CLOSURE=YES
CROSS_REPOSITORY_REVISION_PAIRING=ACTIVE
RESULT=PASS
```

## 8. Agent discoverability

A fresh agent starting in `guillermomolina/protos` can discover from the root
project policy and maintained documentation that durable project records live in
`guillermomolina/protos-project-docs`.

A fresh agent starting in `guillermomolina/protos-project-docs` is told by its
root `AGENTS.md` that:

- this repository is the canonical durable, non-normative project-record store;
- operational project authority remains in `guillermomolina/protos`;
- substantive project work follows the canonical source-repository policy;
- live Issue, scheduling, priority, assignment, approval and normative authority
  do not move here.

Result:

```text
SOURCE_TO_DESTINATION_DISCOVERABILITY=PASS
DESTINATION_TO_SOURCE_AUTHORITY_DISCOVERABILITY=PASS
DUPLICATE_GOVERNANCE_AUTHORITY=NO
RESULT=PASS
```

## 9. History and migration integrity

The migration preserved the extracted project-document history and did not
rewrite `guillermomolina/protos` history.

Historical path strings whose literal spelling is part of chronology, migration
evidence, snapshots, or exact release verification remain unchanged.

Result:

```text
PROTOS_HISTORY_REWRITE=NO
HISTORY_PRESERVING_EXTRACTION=PASS
HISTORICAL_EVIDENCE_SEMANTIC_REWRITE=NO
RESULT=PASS
```

## 10. AUD010 Candidate B-prime conformance

Final authority model:

```text
CONTROL_PLANE              = guillermomolina/protos
LIVE_WORK_AUTHORITY        = protos GitHub Issues
SCHEDULING                 = Protos Development Project
FORMAL_IDENTIFIERS         = governed from protos
APPROVAL_COORDINATION      = protos Issues / AGENTS policy
NORMATIVE_AUTHORITY        = protos/spec
PRODUCT_IMPLEMENTATION     = owning product repository
DURABLE_PROJECT_RECORDS    = guillermomolina/protos-project-docs
ISSUE_MIGRATION            = NO
FORMAL_ID_FEDERATION       = NO
PROTOS_HISTORY_REWRITE     = NO
```

Result:

```text
AUD010_CANDIDATE_B_PRIME=IMPLEMENTED
RESULT=PASS
```

## Final DOC007 result

```text
DOC007_A_STATUS=CLOSED
DOC007_B_STATUS=CLOSED
DOC007_C_STATUS=CLOSED
DOC007_D_STATUS=CLOSED
DOC007_E_STATUS=CLOSED
DOC007_F_STATUS=CLOSED
DOC007_G_STATUS=CLOSED_ON_PUBLICATION

PROTOS_REVISION=756e4cd750ffec6de8e21aa604b21abb546b0a33
PROJECT_RECORD_REVISION_E=ab3e50d914c859960c806047fe67cf0c88798e92
PROJECT_RECORD_REVISION_G=CAPTURE_IN_LIVE_ISSUE_AFTER_PUBLICATION

CROSS_REFERENCES=PASS
REQUIRED_DURABLE_PUBLICATION=PASS
FINAL_AUTHORITY_AUDIT=PASS
FINAL_INTEGRITY_AUDIT=PASS
DOC007_STATUS=CLOSED_ON_G_PUBLICATION_AND_LIVE_ISSUE_CLOSURE
```

After this record is published and its exact destination revision is captured in
live Issue `#532`, DOC007 satisfies its defined closure criteria and the Issue may
be closed as completed.
