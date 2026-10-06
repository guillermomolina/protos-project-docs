# AUD007-B2 — Final parallel launcher isolation closure

## Status

```text
WORK_ITEM=AUD007
ISSUE=guillermomolina/protos#452
SLICE=AUD007-B2
SLICE_TYPE=IMPLEMENTATION
STATUS=COMPLETE
PROTOS_REVISION=b2b132af1338e474857a0e1c9012f2c32f56e869
PROTOS_REVISION_SUBJECT=AUD007-B2: close parallel launcher isolation defects
IMPLEMENTATION_VERSION=0.3.233-SNAPSHOT
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
SPECIFICATION_CHANGED=NO
PRODUCT_RUNTIME_SEMANTICS_CHANGED=NO
PUBLICATION_MODEL=GITHUB003_OPTIMISTIC_FAIL_CLOSED
GLOBAL_VALIDATION_LOCK=NOT_REQUIRED
ARCHITECTURE_DECISION_PENDING=NO
AUD007_CLOSURE_READY=YES
NEXT_SLICE=NONE
```

This is the final durable non-normative closure record for AUD007. It binds the
resource-isolation repairs from B2 to the publication-race and cleanup evidence
from AUD007-B1 and closes the correctness gaps identified by AUD007-A.

## Published implementation

Exact Protos revision:

```text
b2b132af1338e474857a0e1c9012f2c32f56e869
AUD007-B2: close parallel launcher isolation defects
```

The commit changes thirteen paths:

```text
CHANGELOG.md
Makefile
pom.xml
scripts/maven_local_repository_harness.py
scripts/publication_validation.py
scripts/test_maven_local_repository_harness.py
scripts/test_publication_validation.py
scripts/test_validation_impact.py
scripts/validation_impact.py
src/test/java/com/guillermomolina/protos/execution/ProtosAud007DapPortOwnershipTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosDapTestSupport.java
src/test/java/com/guillermomolina/protos/execution/ProtosI026FDapBehaviorTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosI026FDapTransportTest.java
```

`pom.xml` advances the implementation version from `0.3.232-SNAPSHOT` to
`0.3.233-SNAPSHOT` after the substantive validation work.

No specification file is changed. The commit explicitly records no product
runtime semantic change and retains the GITHUB003 optimistic/fail-closed
publication architecture.

## F1 — Maven local-repository mutable-state isolation

### Defect proof

`scripts/maven_local_repository_harness.py` creates a fully local Maven
experiment using:

- two independent Maven JVMs;
- one loopback HTTP fixture remote;
- one synthetic parent POM;
- one temporary shared local repository; and
- a two-party barrier inside resolution of the same artifact.

The retained test
`test_shared_writable_repository_has_no_multiprocess_synchronization` requires:

```text
SHARED_REPOSITORY_WRITES=YES
CONCURRENT_ARTIFACT_RESOLUTION=YES
ARTIFACT_DOWNLOADS=2
MULTIPROCESS_SYNCHRONIZATION=NOT_PROVEN
```

This proves that the project cannot rely on an unqualified shared writable Maven
local repository for independently running validation JVMs.

### Repair

`publication_validation.py` now gives each validation a uniquely named private
writable Maven local-repository head and uses the configured/shared repository
only as `maven.repo.local.tail`.

The retained isolation test requires:

```text
MAVEN_PROCESS_A_EXIT=0
MAVEN_PROCESS_B_EXIT=0
SHARED_REPOSITORY_WRITES=NO
ARTIFACT_DOWNLOADS=0
PRIVATE_HEADS_DISTINCT=YES
PRIVATE_HEADS_LEFT=0
WRITABLE_STATE_ISOLATED=YES
```

The implementation additionally:

- creates private repositories with `tempfile.mkdtemp`;
- records ownership in `OWNER.json`;
- recovers residue only for dead owners;
- removes the private repository after validation;
- runs validation in its own process group; and
- terminates the group on interruption before cleanup.

Final F1 classification:

```text
F1_MAVEN_SHARED_STATE=REPAIRED
F1_SHARED_REPOSITORY_WRITES_PRE_REPAIR=YES
F1_MULTIPROCESS_SYNCHRONIZATION=NOT_PROVEN
F1_WRITABLE_STATE_ISOLATED=YES
F1_SHARED_TAIL_WRITTEN=NO
```

## F2 — DAP ephemeral-port ownership

### Defect proof

`ProtosAud007DapPortOwnershipTest` retains a deterministic reproduction of the
old reserve-close-rebind pattern:

1. reserve an ephemeral loopback port;
2. capture its number;
3. close the reservation;
4. give the exact released port to a foreign listener; and
5. show that the real Graal DAP cannot own that captured port.

The proof does not rely on a timing race.

### Repair

The two real-DAP test families no longer preallocate a port number.

`ProtosDapTestSupport` defines the only allowed test address as:

```text
127.0.0.1:0
```

The real DAP instrument binds the OS-assigned ephemeral port and publishes the
actual endpoint through `ProtosGraalDapReadinessAdapter`. The client connects to
that already-bound endpoint; there is deliberately no client overload that
accepts a preselected port number.

The retained ownership test additionally walks the Java test tree and rejects
DAP address configuration that does not use the shared ephemeral-loopback
constant.

The new ownership regression joins the serial Java lane with the existing real
DAP behavior test.

Final F2 classification:

```text
F2_DAP_PORT_TOCTOU=REPAIRED
F2_RESERVE_CLOSE_REBIND_REMAINING=0
F2_DAP_PORT_OWNER=OPERATING_SYSTEM_UNTIL_BOUND_ENDPOINT_PUBLICATION
```

## F3 — untracked candidate evidence binding

### Defect proof

Before B2, candidate verification ignored untracked paths by using tracked-only
Git status. The retained publication-validation tests preserve this historical
premise and demonstrate that an untracked executable input can be absent from
the candidate SHA while the old tracked-only state check remains clean.

### Repair

`publication_validation.py` now:

1. parses tracked and untracked worktree state separately;
2. fails closed on tracked changes after the candidate SHA;
3. loads the candidate's own `validation_impact.py` taxonomy without writing
   bytecode;
4. classifies untracked paths through the same validation-observability rules;
5. explicitly scans ignored-but-observable `.mvn/` inputs that Maven reads; and
6. reports the offending paths in the fail-closed diagnostic.

`validation_impact.py` now exposes the shared observability primitives. Only
neutral paths are treated as unobservable; unknown executable roots fail closed.
It also preserves the leading dot of dot-directory paths, preventing `.mvn/`
from aliasing another root during normalization.

Retained tests cover at least:

- untracked Java production source;
- untracked Java test source;
- untracked Protos executable/library/tool input;
- untracked script/validation input;
- unknown executable root;
- ignored `.mvn/maven.config`;
- clean exact candidate;
- tracked dirty candidate;
- HEAD mismatch; and
- unrelated `docs/` / scratch input that remains preserved rather than poisoning
  candidate evidence.

Final F3 classification:

```text
F3_UNTRACKED_CANDIDATE_INPUT=REPAIRED
F3_RELEVANT_UNTRACKED_FAIL_CLOSED=YES
F3_IGNORED_MAVEN_INPUT_FAIL_CLOSED=YES
F3_UNRELATED_UNTRACKED_PRESERVED=YES
```

## B1 publication-race and cleanup evidence retained

AUD007-B1 is published in Protos as:

```text
ef8d43e86902b1edec99365bfd7952b29563c598
AUD007-B1: add deterministic publication race and cleanup harness
```

Its durable record remains:

`docs/project/work/AUD007/AUD007_B1_PUBLICATION_RACE_AND_CLEANUP_EVIDENCE.md`.

B1 already closed the dynamic publication and interruption matrix:

```text
P1_DISJOINT_REUSE=PASS
P2A_DIRECT_OVERLAP=SAFE_ABORT
P2B_DEPENDENCY_CLOSURE_MOVEMENT=SAFE_ABORT
P3_SIMULTANEOUS_PUBLICATION=PASS
P4_BOUNDED_RETRY=SAFE_ABORT

CLEANUP_DURING_VALIDATION=PASS
CLEANUP_AFTER_REMATERIALIZATION=PASS
CLEANUP_BEFORE_PUBLICATION=PASS
CLEANUP_GRACEFUL_SIGTERM=PASS
CRASH_RESIDUE_RECOVERY=PASS
```

B2 does not replace those proofs; it completes the resource/evidence-binding
half that B1 intentionally left open.

## Final shared-resource reconciliation

### Git/publication state

```text
CALLER_CHECKOUT_ISOLATION=PASS
IMMUTABLE_CANDIDATE_IDENTITY=PASS
DISJOINT_MOVEMENT_REUSE=PASS
DIRECT_OVERLAP_FAIL_CLOSED=PASS
DEPENDENCY_CLOSURE_MOVEMENT_FAIL_CLOSED=PASS
SIMULTANEOUS_PUBLICATION_ARBITRATION=PASS
FORCE_PUBLICATION=FORBIDDEN
AUTOMATIC_MERGE_REBASE=FORBIDDEN
RETRY_BOUND=FINITE
```

### Build/test state

```text
CANDIDATE_TARGET_OUTPUTS=PRIVATE_BY_CANDIDATE_ROOT
SUREFIRE_REPORTS=CANDIDATE_LOCAL
GENERATED_OUTPUTS=CANDIDATE_LOCAL
TRUFFLE_COMPILER_OUTPUTS=CANDIDATE_LOCAL
MAVEN_WRITABLE_LOCAL_REPOSITORY=PRIVATE_PER_VALIDATION
MAVEN_SHARED_LOCAL_REPOSITORY=READ_ONLY_TAIL
UNTRACKED_OBSERVABLE_INPUTS=FAIL_CLOSED
```

### Runtime/external state

```text
DAP_PORT_ALLOCATION=OS_OWNED_EPHEMERAL_LOOPBACK
LSP_TRANSPORT=STDIO
ORDINARY_EPHEMERAL_TCP_BINDING=OS_ASSIGNED
UNIX_SOCKET_FIXTURES=FIXTURE_LOCAL
TEMPORARY_VALIDATION_ROOTS=UNIQUE
BACKGROUND_VALIDATION_PROCESS_GROUP=CLEANED_ON_INTERRUPTION
CRASH_RESIDUE=IDENTIFIED_AND_RECOVERABLE
```

CPU, RAM and read-only cache contention remain performance characteristics, not
correctness-isolation defects. AUD007 found no evidence requiring a global
validation lock or a serialized publisher queue.

## Final evidence-binding matrix

```text
CANDIDATE_COMMIT_SHA=BOUND
TRACKED_CANDIDATE_BYTES=BOUND
RELEVANT_UNTRACKED_INPUTS=BOUND_BY_FAIL_CLOSED_CHECK
IGNORED_MAVEN_INPUTS=BOUND_BY_FAIL_CLOSED_CHECK
BASE_SHA=BOUND_AS_VALIDATION_ARGUMENT
DEFINITIVE_DELTA=DERIVED_FROM_EXACT_BASE_AND_CANDIDATE
PATCH_OWNED_BYTES_ON_REUSE=B1_PROVEN
DEPENDENCY_CLOSURE_MOVEMENT=B1_FAIL_CLOSED
SIMULTANEOUS_PUBLICATION=B1_FAIL_CLOSED
MAVEN_WRITABLE_STATE=B2_PRIVATE_PER_VALIDATION
DAP_LISTENER_IDENTITY=B2_OS_OWNED_UNTIL_PUBLISHED
```

## Maintainer validation

After publishing B2, the maintainer reported:

```text
git diff --check = PASS
all local tests = PASS
```

Validation provenance is therefore maintainer-reported for the exact published
B2 revision. The committed retained tests encode the F1/F2/F3 proof obligations
and the prior B1 publication/cleanup obligations.

## AUD007 closure matrix

```text
AUD007_A_STATIC_AUDIT=COMPLETE
AUD007_B1_PUBLICATION_RACE_PROOF=PASS
AUD007_B1_CLEANUP_PROOF=PASS
AUD007_B2_F1_MAVEN=REPAIRED
AUD007_B2_F2_DAP=REPAIRED
AUD007_B2_F3_UNTRACKED_BINDING=REPAIRED

SHARED_MUTABLE_RESOURCE_INVENTORY=COMPLETE
RACE_MATRIX=COMPLETE
EVIDENCE_BINDING_MATRIX=COMPLETE
CRITICAL_INVARIANTS_RETAINED=YES

GITHUB003_CONTRACT=RETAINED
PERF007_FULL_NON_TOOL_QUARANTINE=RETIRED
GLOBAL_VALIDATION_LOCK=NOT_REQUIRED
ARCHITECTURE_DECISION_PENDING=NO
CORRECTNESS_DEFECTS_UNOWNED=0
AUD007_FOLLOWUPS_REQUIRED=NONE
AUD007_CLOSURE_READY=YES
NEXT_SLICE=NONE
```

AUD007 has satisfied its success criterion. No further AUD007 implementation or
research slice is required.
