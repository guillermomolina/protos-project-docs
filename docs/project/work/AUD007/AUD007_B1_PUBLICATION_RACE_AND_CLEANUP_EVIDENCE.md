# AUD007-B1 — Deterministic publication race and cleanup evidence

## Status

```text
WORK_ITEM=AUD007
ISSUE=guillermomolina/protos#452
SLICE=AUD007-B1
SLICE_TYPE=IMPLEMENTATION
STATUS=COMPLETE
PRODUCT_REVISION=ef8d43e86902b1edec99365bfd7952b29563c598
PRODUCT_REVISION_SUBJECT=AUD007-B1: add deterministic publication race and cleanup harness
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
PARENT_STATUS=OPEN
NEXT_SLICE=AUD007-B2
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```

This record captures the retained dynamic evidence added by AUD007-B1 for
publication/reconciliation races and cleanup. It is non-normative project
evidence and does not alter Protos language or Standard Library semantics.

## Published implementation

Exact Protos revision:

```text
ef8d43e86902b1edec99365bfd7952b29563c598
AUD007-B1: add deterministic publication race and cleanup harness
```

The commit adds exactly:

```text
scripts/publication_race_harness.py
scripts/test_publication_race_harness.py
```

The harness is explicitly a local test fixture rather than a production patch
launcher. It uses a local bare Git repository, independent temporary publisher
roots, explicit checkpoints/barriers, non-force pushes, bounded retries and
owned-resource cleanup.

## Publication-race evidence

### P1 — disjoint movement with validation reuse

The retained test forces movement after the expensive-validation token is
created. The movement is outside the publisher's patch-owned paths and declared
dependency closure.

The publisher then:

1. fetches the new local fixture head;
2. rematerializes the same patch over that head;
3. proves the patch-owned blob identities are unchanged;
4. proves the rematerialized delta did not escape patch ownership;
5. reuses the existing validation token; and
6. publishes with a normal non-force update.

Expected retained marker:

```text
AUD007_RESULT P1_DISJOINT_REUSE=PASS
```

This closes the dynamic proof gap for unrelated-main movement and bounded
validation reuse in the harness model.

### P2a — direct patch overlap

The retained test forces another publisher/controller to modify B's owned path
after validation.

Expected result:

```text
VALIDATION_REUSE=REJECTED
PUBLICATION=SAFE_ABORT_OR_REVALIDATION_REQUIRED
AUD007_RESULT P2A_DIRECT_OVERLAP=SAFE_ABORT
```

No push is attempted after the overlap is detected.

### P2b — dependency-closure movement

The retained test moves a path that is disjoint from B's patch-owned paths but
inside B's declared validation dependency closure.

Expected result:

```text
VALIDATION_REUSE=REJECTED
REVALIDATION_REQUIRED=YES
AUD007_RESULT P2B_DEPENDENCY_CLOSURE_MOVEMENT=SAFE_ABORT
```

This proves that direct path overlap is not the only invalidation condition.

### P3 — simultaneous publication

Two publishers start from the same fixture base and are released from an
explicit barrier immediately before publication.

The retained assertions require:

- one and only one publisher to succeed;
- the losing publisher to observe a failed ordinary push;
- no force option;
- no merge/rebase/pull/cherry-pick/am recovery;
- remote main to remain on the winner's valid commit; and
- all recorded remote updates to remain fast-forward-only.

Expected marker:

```text
AUD007_RESULT P3_SIMULTANEOUS_PUBLICATION=PASS
```

This closes the simultaneous-publication correctness gap without introducing a
global publication lock.

### P4 — bounded retry

The controller advances the fixture remote immediately before every publication
attempt.

The retained harness declares an explicit retry limit and proves that repeated
movement terminates in a safe abort rather than livelock.

Expected evidence:

```text
RETRY_COUNT <= DECLARED_LIMIT
FINAL_RESULT=SAFE_ABORT
AUD007_RESULT P4_BOUNDED_RETRY=SAFE_ABORT
```

## Cleanup and interruption evidence

### Catchable interruption

The retained tests inject deterministic interruption:

- during validation;
- after rematerialization; and
- immediately before publication.

For each applicable case they assert:

- no publication occurred;
- remote main remains valid;
- the publisher-owned validation child is stopped;
- its loopback listener is gone;
- publisher-owned temporary state is removed;
- foreign sibling state is preserved; and
- cleanup can be called again safely.

Expected markers:

```text
AUD007_RESULT CLEANUP_DURING_VALIDATION=PASS
AUD007_RESULT CLEANUP_AFTER_REMATERIALIZATION=PASS
AUD007_RESULT CLEANUP_BEFORE_PUBLICATION=PASS
```

### Real SIGTERM

The harness also launches a held publisher in a separate process/session and
delivers real SIGTERM.

The signal is converted into a catchable interruption, after which the retained
assertions require:

- process result 130;
- child validation process terminated;
- child listener gone;
- owned work directory cleaned; and
- remote main unchanged.

Expected marker:

```text
AUD007_RESULT CLEANUP_GRACEFUL_SIGTERM=PASS
```

### SIGKILL crash residue

AUD007-B1 correctly does not claim that cleanup code can run after SIGKILL.

Instead it proves a crash-recovery invariant:

1. SIGKILL leaves an owned work root carrying `OWNER.json`;
2. remote main is unchanged;
3. recovery scans only correctly prefixed roots with valid owner metadata;
4. a dead owner's process group is terminated when applicable;
5. only the dead publisher's owned residue is removed;
6. unrelated/foreign state is preserved;
7. recovery is idempotent.

Expected marker:

```text
AUD007_RESULT CRASH_RESIDUE_RECOVERY=PASS
```

This closes the prior ambiguity between graceful signal cleanup and
non-catchable crash recovery.

## Git safety evidence retained by the harness

The harness records Git commands issued by each publisher and rejects:

```text
merge
rebase
pull
cherry-pick
am
-f
--force
--force-with-lease
+refspec
```

The local bare fixture also sets:

```text
receive.denyNonFastForwards=true
receive.denyDeletes=true
core.logAllRefUpdates=always
```

and the tests verify every recorded update of fixture `main` is a
fast-forward.

Therefore B1 dynamically supports the GITHUB003 architecture retained by
AUD007-A:

```text
GLOBAL_VALIDATION_LOCK=NOT_REQUIRED
PUBLICATION_MODEL=OPTIMISTIC_FAIL_CLOSED
FORCE_PUBLICATION=FORBIDDEN
AUTOMATIC_MERGE_REBASE=FORBIDDEN
RETRY=FINITE
```

## Maintainer validation

The maintainer reported after publication:

```text
git diff --check = PASS
all local tests = PASS
```

No contrary test evidence is recorded for this revision.

## Parent closure state

AUD007-B1 closes the publication-race and cleanup portion of AUD007.

It does **not** exercise the three resource/evidence-binding findings left by
AUD007-A:

```text
F1_MAVEN_SHARED_STATE=OPEN
F2_DAP_PORT_TOCTOU=OPEN
F3_UNTRACKED_CANDIDATE_INPUT=OPEN
```

Accordingly AUD007 / #452 cannot yet close.

The next productive slice should handle all three remaining findings together,
rather than creating three tiny slices:

```text
NEXT_SLICE=AUD007-B2
GOAL=deterministic resource/evidence-binding proof and bounded repairs
OWNER_DECISION_REQUIRED_BEFORE_START=NO
```

If B2 proves a Maven architecture choice genuinely requires owner selection,
that decision is surfaced only then. DAP and untracked-candidate repairs should
be made in B2 when the deterministic proof confirms the already identified
defect and the repair follows directly from the established invariant.
