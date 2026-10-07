# DOC007-E — Cross-repository publication contract cutover

Status: **CLOSED on publication**

Live coordination: GitHub `DOC007 / #532`

Validation class: `DOCUMENTATION_ARCHITECTURE_AND_REPOSITORY_MIGRATION`

## Purpose

DOC007-E operationalizes the cross-repository publication contract selected in
DOC007-A after the durable project-record corpus was extracted and bootstrapped
in `guillermomolina/protos-project-docs`.

This phase does not change the authority model. Operational work remains
coordinated from `guillermomolina/protos`; this repository remains the durable,
non-normative project-record authority.

## Product revision covered by this record

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=a798e6199d2bfe30fca748b7536afe54ea79d8f1
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
```

`PROTOS_REVISION` is the exact product-side cutover revision covered by this
record. A moving `main` reference is not sufficient revision-bound evidence.

The exact `PROJECT_RECORD_REVISION` is captured in live Issue `#532` after this
record publishes, because a Git commit cannot contain its own final commit SHA.

## Source-side publication-contract cutover

### E1 — operational policy

Published source revision:

```text
a80a98a3127b3d56807d0efbd2642ef5d75be1ca
```

`AGENTS.md` now requires separate durable publication when work needs a project
record and uses the following closure tuple:

```text
PROTOS_REVISION=<exact SHA>
PROJECT_RECORD_REVISION=<exact SHA>
CROSS_REFERENCES=PASS
REQUIRED_DURABLE_PUBLICATION=PASS
```

For implementation plus durable closure evidence, product publication occurs
first, the durable record names that exact product revision, the project-record
revision is then captured, and the live Issue verifies both sides before
closure.

A failed durable publication does not roll back an already-published product
commit. The live Issue remains open and required durable publication remains
unsatisfied.

### E2A — release-selection input decoupling

Published source revision:

```text
801cbd129f96761cf722d368c0b2cacfa47a1b7c
```

The following release helpers no longer assume that the DIST001 selection record
exists inside the Protos checkout:

```text
dist/commit_release_candidate.py
dist/materialize_release_candidate.py
dist/prepare_release_candidate_worktree.py
dist/transition_release_candidate_version.py
dist/verify_release_candidate_lineage.py
```

Their `--selection` argument is now mandatory. The caller supplies the durable
record path explicitly.

Validation:

```text
FOCAL_TESTS=PASS (36/36)
SELECTION_CONTRACT=PASS
DIFF_CHECK=PASS
```

### E2B — external durable-record checkout documentation

Published source revision:

```text
f649887b5c4a2b1b9419075427c79ae4724ebdf8
```

`dist/README.md` now documents an explicitly selected
`guillermomolina/protos-project-docs` checkout. It requires:

```text
PROJECT_RECORDS_ROOT=<explicit checkout>
PROJECT_RECORD_REVISION=<exact SHA>
PROJECT_RECORD_WORKTREE=CLEAN
```

The documented release commands consume durable record paths from that verified
checkout. They do not assume an in-tree `docs/project/**` corpus, a fixed sibling
checkout, or a moving `main` revision.

### E2C — portable runtime-evidence locator

Published source revision:

```text
a798e6199d2bfe30fca748b7536afe54ea79d8f1
```

Newly generated portable-distribution `RUNTIME.txt` metadata no longer points
`runtime_evidence` at a source-repository-local project path.

The live locator is revision-bound to the durable DIST002 record:

```text
https://github.com/guillermomolina/protos-project-docs/blob/ee0da2bfae717299600e369b4f11998f3395f42f/docs/project/work/DIST002/DIST002_TOOLCHAIN_ALIGNMENT.md
```

The `RUNTIME.txt` schema is unchanged.

The historical DIST001-E4C2 archive verifier intentionally retains the old
`docs/project/PERF002_TRUFFLE_COMPILABILITY.md` literal because it verifies the
exact metadata embedded in an already-persisted historical candidate archive.
That compatibility evidence is not a current locator and must not be rewritten.

Validation:

```text
PY_COMPILE=PASS
RUNTIME_EVIDENCE_AST_CHECK=PASS
DIFF_CHECK=PASS
HISTORICAL_E4C2_LITERAL=PRESERVED
RUNTIME_TXT_SCHEMA_CHANGED=NO
```

## Post-bootstrap reconciliation

After source-side reference and publication-contract cutover, comparison through:

```text
PROTOS_REVISION=a798e6199d2bfe30fca748b7536afe54ea79d8f1
```

shows no modification under source `docs/project/**`.

Therefore:

```text
POST_BOOTSTRAP_PROJECT_RECORD_DELTA=NONE
SOURCE_DESTINATION_RECONCILIATION=PASS
```

No additional source project record needs to be copied before DOC007-F removes
the source corpus.

## Historical-reference preservation

DOC007 continues to apply the DOC007-A classification:

```text
ACTIVE_MAINTAINED_REFERENCE -> REWRITE_AT_CUTOVER
HISTORICAL_EVIDENCE_STRING  -> PRESERVE_LITERAL
AMBIGUOUS_REFERENCE          -> FAIL_CLOSED_AND_CLASSIFY
```

Old paths preserved in changelogs, migration records, frozen evidence, or
historical release contracts remain truthful descriptions of earlier repository
states.

There is no repository-wide blind replacement of `docs/project/`.

## Cross-repository publication contract

### Decision / approval

```text
owner approval
  -> durable project decision record publishes
  -> exact PROJECT_RECORD_REVISION is verified
  -> dependent product work may publish
```

### Product implementation plus durable closure evidence

```text
product implementation publishes
  -> exact PROTOS_REVISION exists
  -> durable closure/evidence record publishes and names that SHA
  -> exact PROJECT_RECORD_REVISION exists
  -> live Issue verifies both sides
  -> Issue may close
```

### Required closure tuple

```text
PROTOS_REVISION=<exact SHA>
PROJECT_RECORD_REVISION=<exact SHA>
CROSS_REFERENCES=PASS
REQUIRED_DURABLE_PUBLICATION=PASS
```

## Authority invariants after E

```text
CONTROL_PLANE=guillermomolina/protos
LIVE_WORK_AUTHORITY=protos GitHub Issues
SCHEDULING=Protos Development Project
FORMAL_IDENTIFIER_AUTHORITY=protos
NORMATIVE_AUTHORITY=protos/spec
DURABLE_PROJECT_RECORD_AUTHORITY=guillermomolina/protos-project-docs
ISSUE_MIGRATION=NO
FORMAL_ID_FEDERATION=NO
PROTOS_HISTORY_REWRITE=NO
```

## Phase result

```text
DOC007_E_STATUS=CLOSED_ON_PUBLICATION
PROTOS_REVISION=a798e6199d2bfe30fca748b7536afe54ea79d8f1
PROJECT_RECORD_REVISION=CAPTURE_IN_LIVE_ISSUE_AFTER_PUBLICATION
CROSS_REPOSITORY_PUBLICATION_CONTRACT=ACTIVE
LIVE_RELEASE_SELECTION_SOURCE_COUPLING=REMOVED
PORTABLE_RUNTIME_EVIDENCE_SOURCE_COUPLING=REMOVED
POST_BOOTSTRAP_PROJECT_RECORD_DELTA=NONE
SOURCE_DESTINATION_RECONCILIATION=PASS
DOC007_F_STATUS=READY_AFTER_PROJECT_RECORD_REVISION_CAPTURE
```

DOC007-F may remove the live source `docs/project/**` corpus after this durable
record is published and its exact destination revision is captured in live Issue
`#532`.
