# AUD007 — Parallel patch launcher isolation and publication-safety audit

## Status

```text
WORK_ITEM=AUD007
ISSUE=guillermomolina/protos#452
SLICE=AUD007-A
SLICE_TYPE=INVESTIGATION
STATUS=COMPLETE
INVESTIGATION_EXECUTED_COMMANDS=NO
PRODUCT_REPOSITORY_MODIFIED=NO
INVESTIGATION_HEAD=845a1103b031abcf95d8ba852e0d780ba6b6591a
INVESTIGATION_HEAD_SUBJECT=AUD006-A4: make CommandLine accumulation linear
RECONCILIATION_HEAD=62f3f5710210f247aad8574d3d0d56d3254bfd70
RECONCILIATION_HEAD_SUBJECT=LIB010-E2-A: make TOML temporal fraction encoding linear
ARCHITECTURE_DECISION_REQUIRED=NO
NEXT_SLICE=AUD007-B
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```

This is the durable non-normative record of AUD007-A. It audits the current
parallel patch validation/publication model, shared mutable resources, validation
evidence binding, simultaneous publication races, and the minimum dynamic proof
needed before AUD007 can close. It does not define Protos language or Standard
Library semantics.

The original Issue body used the historical identifier AUD005. GITHUB013 later
reconciled the collision and the live authoritative identifier is AUD007. New
durable material therefore uses `AUD007`.

## Reconciliation through current main

AUD007-A was performed against:

```text
845a1103b031abcf95d8ba852e0d780ba6b6591a
AUD006-A4: make CommandLine accumulation linear
```

Before durable publication, `main` was reconciled through:

```text
62f3f5710210f247aad8574d3d0d56d3254bfd70
LIB010-E2-A: make TOML temporal fraction encoding linear
```

The two intervening publications changed TOML and TEST009-T compilerability
surfaces, including `CHANGELOG.md`, `pom.xml`, TOML source/tests, compilerability
runtime/test sources, and Truffle compilerability tooling.

They did not change the AUD007-A governing publication/validation surfaces:

```text
AGENTS.md
AGENTS.work/AUDIT.md
AGENTS.work/IMPLEMENTATION.md
scripts/publication_validation.py
scripts/validation_impact.py
scripts/test_publication_validation.py
scripts/test_validation_impact.py
.github/workflows/tests.yml
Makefile
toolchain.json
.github/ci-image-lock.json
```

They also did not change the DAP port-allocation test pattern or the Maven local
repository contract underlying the resource-isolation findings below. The A
conclusions therefore remain applicable at the reconciliation head.

## Executive conclusion

The GITHUB003 architecture remains the correct publication model:

- long validation is tied to immutable candidate content rather than requiring
  `main` to remain frozen;
- publication is a short optimistic/fail-closed phase;
- a normal non-force fast-forward publication race does not require a global
  validation lock;
- unrelated `main` movement may preserve expensive validation evidence only
  when the declared dependency/precondition closure remains valid; and
- relevant movement requires revalidation or safe abort.

AUD007-A finds no architectural reason to replace this model and no reason to
serialize long-running validation.

However, the current repository no longer contains one generic retained
end-to-end launcher that mechanically owns materialization, movement
classification, rematerialization, validation-reuse proof, commit creation and
publication as one auditable component. The retained layers are separate:

1. `scripts/validation_impact.py` classifies a definitive commit delta
   conservatively;
2. `scripts/publication_validation.py` validates one exact candidate commit;
3. current human-executor policy leaves final repository mutation/publication to
   the maintainer.

Therefore current static evidence is sufficient to retain the architecture, but
not sufficient to declare every simultaneous-launcher race closed.

## PERF007 reconciliation

The historical PERF007 `FULL:NON_TOOL` quarantine is retired.

At the audited/reconciled state:

- shared, unknown, unmapped, empty and top-level-closure deltas route to
  `FULL`;
- `publication_validation.py` reports Tool tests as included by the selected
  scope;
- the publication-validation parser rejects the retired
  `FULL:NON_TOOL` result; and
- only explicitly bounded Tool-local intermediate routing remains.

AUD007-B must not reconstruct or depend on the retired quarantine.

## Positive existing evidence

Issue #452 already records a real TOOL002-J long FULL run with:

```text
validation_impact=FULL
affected_test_set=ALL
full_tests=1906
full_failures=0
full_errors=0
origin_main_changed_during_validation=YES
CALLER_GIT_FETCHED_BY_LAUNCHER=NO
CALLER_WORKTREE_CREATED_BY_LAUNCHER=NO
CALLER_BRANCH_CREATED_BY_LAUNCHER=NO
CALLER_WORKTREE_TOUCHED_BY_LAUNCHER=NO
ORIGIN_MAIN_WRITTEN_BY_VALIDATION_PHASE=NO
SHARED_MAVEN_REPOSITORY_WRITTEN=NO
CLEANUP_RUNTIME=PASS
```

That run took approximately nine minutes while `origin/main` advanced. It is
positive evidence that long validation need not freeze `main`, but it does not
prove simultaneous publication, relevant dependency movement, Maven writes,
resource-name collisions, signal cleanup or generic launcher conformance.

AUD007-B must reuse this evidence rather than repeat another long FULL merely to
re-prove the same property.

## Retained publication and validation model

### Candidate validation

`scripts/publication_validation.py`:

- resolves the requested candidate as a commit;
- requires the worktree `HEAD` to equal that exact commit;
- rejects tracked dirtiness after candidate creation;
- runs source-style and legacy-execution prevention guards;
- invokes `scripts/validation_impact.py` for `base..candidate`; and
- executes either canonical `make test` for `FULL` or the selected complete
  Tool-local Maven test set.

It does not:

- fetch or refresh `origin/main`;
- materialize the patch;
- classify post-validation movement for reuse;
- rematerialize on a new base;
- prove patch-byte equivalence;
- create the publication commit;
- publish the candidate; or
- coordinate competing publishers.

### Adaptive impact routing

`scripts/validation_impact.py` is fail-closed:

- shared production/library/runtime/compiler/parser/build/test infrastructure
  routes to `FULL`;
- unknown/unmapped executable paths route to `FULL`;
- empty definitive deltas route to `FULL`;
- cross-tool deltas route to `FULL`;
- top-level executable closure forces `FULL`; and
- only explicitly mapped bounded Package Tool or Test Tool intermediate deltas
  may use reduced profiles.

## Shared-resource inventory

### Git and repository state

| Resource | Classification | AUD007-A result |
|---|---|---|
| caller checkout | private by policy | must remain untouched outside owned paths |
| candidate HEAD/index | private when isolated | suitable validation identity |
| Git object database | shared mutable, Git-managed | correctness-safe under ordinary Git operations; possible contention |
| refs including local remote-tracking refs | shared mutable | never treat moving `origin/main` as immutable identity |
| remote `main` | shared mutable publication point | optimistic non-force fast-forward remains preferred |
| force push | forbidden | retain prohibition |
| merge/rebase conflict repair | forbidden for automatic publication reconciliation | retain fail-closed behavior |
| cleanup metadata | launcher/process owned | abrupt-interruption behavior still requires dynamic proof |

### Build and test state

| Resource | Classification | AUD007-A result |
|---|---|---|
| candidate `target/` trees | private when candidate roots differ | safe |
| Surefire reports | candidate-local | safe |
| slow-test state/logs | candidate-local under `target/` | safe |
| generated sources | candidate-local under `target/` | safe |
| Truffle compilation reports | candidate-local under `target/` | safe |
| JVM/Engine in-process caches | process-private | safe across independent JVMs |
| CPU/RAM | shared performance resource | may slow concurrent FULL; not by itself correctness failure |
| Maven local repository | shared mutable host state | correctness proof missing for multi-process writes |
| Maven metadata/`.lastUpdated` | shared mutable host state | same missing proof |
| HOME/global user configuration | shared external input | not bound by publication validation |
| Java/Python temporary roots | host-shared namespace | most retained APIs use unique names; targeted collision proof still needed |

### Runtime and external resources

| Resource | Classification | AUD007-A result |
|---|---|---|
| production debug endpoint | OS-assigned loopback ephemeral port | good: direct `127.0.0.1:0` binding |
| DAP behavior/transport tests | shared TCP namespace | defect candidate: reserve-close-rebind TOCTOU |
| ordinary TCP test listeners | OS-assigned ephemeral ports in inspected paths | generally safe |
| Unix-domain socket fixtures | fixture-local path | generally safe |
| language server transport | stdio | no shared TCP listener found |
| runtime executors/hosts | process-private with explicit close paths | normal cleanup evidence positive |
| abrupt signal/crash cleanup | host resource lifecycle | dynamic proof still required |

## Evidence-binding matrix

| Evidence dimension | Classification | Reason |
|---|---|---|
| candidate commit SHA | BOUND | validator resolves commit and requires exact HEAD |
| tracked candidate bytes | BOUND | immutable commit plus tracked-clean check |
| untracked candidate inputs | NOT_BOUND | validator explicitly ignores untracked files in candidate-state check |
| base SHA | BOUND_AS_ARGUMENT | selector/guards consume it, but caller must supply the intended publication base |
| definitive changed paths | DERIVED_AND_RECHECKED | Git diff of exact base and candidate |
| patch-owned path set | NOT_BOUND_GENERICALLY | no retained independent ownership manifest |
| validation dependency closure | NOT_BOUND_GENERICALLY | no retained generic closure manifest/proof |
| semantic/governance preconditions | ASSUMED_EXTERNALLY | slice/launcher responsibility |
| selector/guard code | CANDIDATE_TREE_BOUND | executed from candidate tree |
| effective JDK/GraalVM/Maven runtime | NOT_BOUND_BY_PUBLICATION_VALIDATOR | repository contract exists, but this helper does not attest runtime identity |
| HOME/Maven settings/cache state | NOT_BOUND | host environment |
| final rematerialized patch bytes after reuse | NOT_BOUND_BY_VALIDATOR | belongs to publication/reconciliation layer |
| final published commit vs validated evidence | NOT_BOUND_BY_VALIDATOR | publication layer responsibility |

## Confirmed static isolation defects / correctness risks

### F1 — shared Maven local repository has no explicit multi-process proof

Protos does not force a per-candidate Maven local repository for ordinary
validation and does not currently retain a project-specific proof that
independent Maven processes sharing one local repository have the required
multi-process synchronization semantics for all writes relevant to concurrent
launchers.

The TOOL002-J run observed no shared Maven-repository writes, which is valuable
for that run but cannot establish safety for executions that do write or
download.

AUD007-B must deterministically exercise this case before choosing between:

- retaining a shared local repository with demonstrated/configured
  multi-process locking; or
- isolating the local repository per candidate.

No architecture decision is required before that evidence exists.

### F2 — DAP tests use reserve-close-rebind port allocation

`ProtosI026FDapTransportTest` and `ProtosI026FDapBehaviorTest` reserve an
ephemeral port with `ServerSocket(0)`, close that socket, then later ask the
Graal DAP server to bind the numerical port.

Another process can claim the released port in that interval. This is a
deterministic TOCTOU surface under concurrent test launchers.

The production debug runtime uses `127.0.0.1:0` directly and therefore delegates
uniqueness to the OS without the same reservation gap.

AUD007-B should prove F2 with a controlled port-theft barrier; the expected repair
family is to preserve OS-owned ephemeral binding instead of pre-reserving and
releasing a number.

### F3 — publication validation ignores untracked candidate inputs

The candidate-state check uses tracked-only status inspection. Therefore
untracked files are not part of the candidate commit identity and do not fail
candidate verification.

An untracked executable/test input located inside a build/test source root can
therefore potentially affect validation without being represented by the
candidate SHA.

AUD007-B should construct a bounded fixture proving whether the current build can
observe such an input. If yes, publication validation must fail closed on
relevant untracked inputs or otherwise prove they cannot affect the selected
validation.

## Race matrix

| Scenario | Static result | AUD007-B obligation |
|---|---|---|
| same base, disjoint deltas, A publishes first | architecture supports reuse after proof | dynamic proof |
| same base, overlapping deltas | must fail closed | dynamic proof |
| governance/docs movement during B FULL | long-validation phase already has positive evidence | cheap reconciliation proof only |
| dependency-relevant movement during B FULL | reuse forbidden without revalidation | dynamic proof |
| two simultaneous publishers | normal non-force publication gives one winner; loser handling unproven | dynamic proof |
| repeated `main` movement | retry must be finite | dynamic proof |
| interruption during validation/rematerialization/publication | fail-safe cleanup required | dynamic proof |
| colliding temp/worktree/branch/container/port names | DAP port defect identified; other namespaces require bounded proof | dynamic proof |
| background state left alive | normal cleanup positive; abrupt termination unproven | dynamic proof |
| long immutable validation + short reconciliation | TOOL002-J proves long phase, not whole transaction | do not repeat FULL; prove short phase |

## Publication model comparison

### A — optimistic fast-forward / compare-and-reconcile

Selected recommendation: KEEP.

Properties:

- no global validation serialization;
- remote branch update is the short shared commit point;
- losing publishers refresh and classify movement;
- unrelated movement may reuse expensive evidence only under an explicit proof;
- relevant movement revalidates or aborts;
- no global lock state needs crash recovery.

### B — brief publication lock

Safe but not currently justified. A lock does not remove the need to refresh and
classify `main`; it only adds lock ownership/recovery state around a problem the
remote ref update already arbitrates.

### C — publisher queue

Potentially useful only if future sustained contention requires fairness or
central ordering. Current evidence does not justify it.

### D — global validation/publication lock

Rejected. It would serialize long independent validation and directly defeat the
parallelism goal exposed by TOOL002-J.

## Invariant classification after AUD007-A

| Invariant | Classification |
|---|---|
| validation is tied to one exact tracked candidate commit | RATIFIABLE_FROM_CURRENT_EVIDENCE |
| `main` may move while long validation runs | RATIFIABLE_FROM_CURRENT_EVIDENCE |
| unrelated movement does not automatically invalidate correct work | RATIFIABLE_FROM_CURRENT_EVIDENCE |
| long validation should not hold a global publication lock | RATIFIABLE_FROM_CURRENT_EVIDENCE |
| publication remains non-force/fail-closed | RATIFIABLE_FROM_CURRENT_EVIDENCE |
| relevant movement forces revalidation or safe abort | REQUIRES_DYNAMIC_PROOF |
| simultaneous losing publisher handles rejection safely | REQUIRES_DYNAMIC_PROOF |
| retry under repeated movement is finite | REQUIRES_DYNAMIC_PROOF |
| abrupt-interruption cleanup is idempotent/safe | REQUIRES_DYNAMIC_PROOF |
| shared Maven state cannot corrupt validation | NEEDS_CHANGE_OR_EXPLICIT_PROOF |
| temporary/runtime resource naming is collision-safe | NEEDS_CHANGE for DAP reservation pattern |
| validation evidence excludes uncommitted executable inputs | NEEDS_CHANGE_OR_EXPLICIT_PROOF |

## AUD007-B dynamic proof plan

AUD007-B is implementation work in `guillermomolina/protos`. It should add a
small deterministic harness/tests, not another broad architecture investigation.

### Publication/reconciliation harness

Use a local bare Git remote and independent local candidate roots. Add explicit
barriers so interleavings are selected rather than timing-dependent.

Required cases:

1. disjoint movement after B's expensive-gate token:
   - A publishes;
   - B refreshes;
   - closure remains unchanged;
   - B rematerializes equivalent patch-owned bytes;
   - cheap gates rerun;
   - expensive evidence is reused.
2. direct overlap:
   - reuse is rejected and publication safely aborts/revalidates.
3. dependency-closure-only movement:
   - changed path is disjoint from patch ownership but inside declared
     validation closure;
   - reuse is rejected.
4. simultaneous normal publication:
   - release A and B at the final publication barrier;
   - exactly one update succeeds;
   - losing publisher does not force, merge or silently overwrite.
5. repeated movement:
   - a controller advances the fixture remote after each reconciliation;
   - retry reaches a finite declared bound and aborts safely.

### Resource harness

Required cases:

1. two Maven processes/candidates using one controlled temporary local
   repository, forcing a write/resolution path and retaining lock/write/failure
   evidence;
2. deterministic DAP port theft between reservation release and server bind;
3. untracked executable/test input placed inside a fixture source root before
   publication validation.

### Cleanup harness

Inject interruption at deterministic barriers:

- during validation;
- after rematerialization; and
- immediately before publication.

After each case verify:

- caller state unchanged;
- remote `main` valid;
- no partial/force publication;
- no child process/listener survives;
- owned temporary state is cleaned or explicitly recoverable; and
- cleanup is safe to invoke again.

## AUD007-B cost boundary

AUD007-B must not repeat the existing approximately nine-minute TOOL002-J FULL
run. Use a cheap deterministic stand-in/token for the already-proven
long-validation phase and test the race/reconciliation mechanics around it.

Only run the repository validation required by the actual AUD007-B changed-file
delta at final candidate time.

## Decision state

```text
OWNER_DECISION_REQUIRED_BEFORE_AUD007_B=NO
GITHUB003_ARCHITECTURE=RETAIN
GLOBAL_VALIDATION_LOCK=REJECT
PERF007_FULL_NON_TOOL_QUARANTINE=RETIRED
MAVEN_ISOLATION_CHOICE=DEFER_UNTIL_DYNAMIC_EVIDENCE
AUD007_B_READY=YES
```

AUD007 remains open after A. Closure requires the deterministic B evidence and
any repairs that B proves necessary.
