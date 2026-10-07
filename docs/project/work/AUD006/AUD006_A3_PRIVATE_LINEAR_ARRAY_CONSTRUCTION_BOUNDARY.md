# AUD006-A3 — Private linear Array construction boundary reconciliation

## Status

```text
WORK_ITEM=AUD006
ISSUE=guillermomolina/protos#453
SLICE=AUD006-A3
SLICE_TYPE=INVESTIGATION_AND_OWNER_DECISION
STATUS=COMPLETE_APPROVED
ANALYSIS_HEAD=0d478c5bf4aabaaac781b80cc7f5889c8bbc9160
ANALYSIS_HEAD_SUBJECT=TEST002-A4: reconcile JSON parser semantic test ownership
PROTOS_HEAD_RECONCILED=fc9aca90f479051964d8d56c11a2778386789671
PROTOS_HEAD_SUBJECT=DOC005-F: document Test Tool exact file-backed focal selection
HEAD_RECONCILIATION=PASS
OWNER_APPROVAL_DATE=2026-10-06
OWNER_APPROVAL=AUD006_A3_CONCLUSION_APPROVED
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

This is a durable non-normative project record. It records the result of the
AUD006-A3 architecture investigation and the project owner's explicit approval.
It does not define public Protos language or Standard Library semantics.

AUD006-A1 remains the governing static proof that the current balanced
`CommandLine.parse` accumulation is `Theta(N log N)` under the current
materializing Array construction behavior. AUD006-A2 remains valid negative
evidence that complete-parser timing/allocation scaling is not a discriminating
instrument for this small builder term.

## Moving-HEAD reconciliation

A3's final research was reconciled at:

```text
0d478c5bf4aabaaac781b80cc7f5889c8bbc9160
TEST002-A4: reconcile JSON parser semantic test ownership
```

Before this durable publication, `guillermomolina/protos` advanced to:

```text
fc9aca90f479051964d8d56c11a2778386789671
DOC005-F: document Test Tool exact file-backed focal selection
```

The only intervening product-repository commit changes
`docs/guide/tools/test-tool.md`. It does not modify
`protos/lib/cli/CommandLine.protos`, Array runtime representation or
construction, standard-module provisioning, execution-context semantics, or the
Test Tool facilities used as architectural precedent. Therefore the A3
conclusion remains valid at the reconciled HEAD.

## A3 question

A1 and A2 had provisionally classified the narrow linear construction boundary
as requiring a new platform/runtime decision because guest `CommandLine`
cannot construct an ordinary standard Array linearly with the existing public
Array surface.

A3 therefore investigated whether repairing F1 actually requires a new durable
platform institution, or whether the repository already contains a ratified
architecture that can be reused without changing public semantics.

The research compared the private append/finalize family against exact-size
fill, generalization of internal argument staging, ownership-transfer/zero-copy,
public Array growth, global Array representation changes, Java-owned
CommandLine parsing, weakening D115, and instrumentation-only approaches. It
also reviewed comparable builder/storage patterns in mature Truffle and
non-Truffle runtimes.

The initial A3 recommendation was a private append/finalize builder producing
one ordinary standard Array with a single final O(N) materialization. During
follow-up review, the existing Test Tool architecture supplied a stronger
repository-local precedent that changes the governance classification.

## Existing ratified Test Tool precedent

D101 / #420 ratified Candidate C-prime for Test Tool:

```text
tool policy remains in Protos
host/runtime exposes one narrow bootstrap-local capability
the capability owns only irreducible host mechanics
no broad ambient authority is introduced
the capability does not become public Core or Standard Library API
```

The published implementation includes facilities such as:

- `ProtosTestToolCatalogAcquisitionFacility`;
- `ProtosTestToolFileSelectionFacility`; and
- other bootstrap-local exact-execution/tool facilities.

The directly relevant mechanical precedent is
`ProtosTestToolFileSelectionFacility`:

```text
private host ArrayList<Object>
    -> linear add(...)
    -> one prelude.newFrozenArray(associations)
    -> ordinary Protos Array result
```

This already demonstrates the implementation family needed for AUD006 F1:
temporary host-private linear accumulation followed by one ordinary Protos Array
materialization. The policy that decides what to append remains outside the host
mechanism.

Test Tool does still contain unrelated guest-side balanced-chunk builders in
places such as `Options.protos`, `SuiteGraph.protos`, `Manifest.protos`
and `Runner.protos`. A3 therefore does **not** claim that Test Tool already
provides a reusable generic Array-builder facility. The precedent is the
ratified private-capability and host-private-linear-accumulation architecture,
not an already-existing public/general builder API.

## Standard-module visibility difference

The Test Tool initial module can retain bootstrap-local capabilities because it
is a tool bootstrap module rather than an ordinary importable Standard Library
surface.

`std:cli/CommandLine` is different: its module context is the imported module
instance, so blindly provisioning a helper member and leaving it present would
make the helper structurally observable.

Current execution-context semantics, however, permit an OPEN module context to
remove a PRESENT local slot. A Standard Library module may therefore consume a
temporary bootstrap member during initialization, capture the required
capability in a private lexical closure/activation, and remove the temporary
top-level slot before module initialization completes.

The implementation must ensure that public `parse` and any other surviving
closure capture the private lexical binding that holds the capability, not the
removed module slot itself.

This is a use of already-existing execution-context semantics, not a new
visibility/export mechanism.

## Approved architecture

The project owner explicitly approved the following AUD006-A3 conclusion on
2026-10-06.

```text
SELECTED_REPAIR_FAMILY=
  D101_STYLE_PRIVATE_BOOTSTRAP_CAPABILITY_WITH_LINEAR_HOST_ACCUMULATION

COMMANDLINE_POLICY_OWNER=PROTOS
HOST_MECHANISM_SCOPE=ARRAY_CONSTRUCTION_ONLY
HOST_ACCUMULATION=PRIVATE_INVOCATION_LOCAL_LINEAR_BUFFER
FINAL_MATERIALIZATION=ONE_ORDINARY_STANDARD_ARRAY
ZERO_COPY_OWNERSHIP_TRANSFER=NO
PUBLIC_ARRAY_API_CHANGE=NO
ARRAY_REPRESENTATION_CHANGE=NO
COMMANDLINE_PUBLIC_API_CHANGE=NO
COMMANDLINE_OBSERVABLE_SEMANTIC_CHANGE=NO
D111=KEEP
D115=KEEP
D118=KEEP
D119=KEEP
LIB011_ISSUE_428=KEEP_CLOSED
```

The approved implementation envelope is:

1. Protos `CommandLine` continues to own all parse/specification policy,
   traversal, provenance, ordering, positional allocation and result shaping.
2. The runtime may provision a narrow private construction capability only for
   the exact standard `std:cli/CommandLine` module identity.
3. The capability may create invocation-local private builder state backed by a
   host growable buffer such as `ArrayList<Object>`.
4. Append stores exact already-evaluated Protos values in order and performs no
   user dispatch, conversion, parsing or policy.
5. Finalization materializes exactly one fresh ordinary standard Array through
   existing Array/runtime construction infrastructure.
6. The baseline repair must copy into the final Array once. It must not add
   owned-storage transfer, zero-copy adoption or new COW semantics.
7. Builder state is one-shot and private. Append after finish and repeated
   finish must fail closed.
8. No mutable static/global builder registry or parser state is allowed.
9. The temporary bootstrap member must not remain in the final observable
   `CommandLine` module surface.
10. No public `Array.add`, reserve, resize, builder, exact-size-fill or other
    new collection API is introduced.
11. The asymptotic correctness claim must not depend on Truffle partial
    evaluation, JIT compilation, a native-body PIC, or Native Image
    specialization.
12. A focused retained test must be able to prove the private builder lifecycle
    and linear append/finalize accounting without restoring the rejected
    expensive full-parser scaling experiment.

## Governance correction

The repository-local D101/Test Tool precedent means A1/A2's provisional
governance classification was too strong.

```text
PREVIOUS_CLASSIFICATION=NEW_PLATFORM_OR_RUNTIME_DECISION_REQUIRED
A3_RECONCILED_CLASSIFICATION=MECHANICAL_REUSE_OF_RATIFIED_ARCHITECTURAL_PATTERN
PLAT051_REQUIRED=NO
NEW_DXXX_REQUIRED=NO
NEW_LANGUAGE_DECISION_REQUIRED=NO
NEW_STANDARD_LIBRARY_SEMANTIC_DECISION_REQUIRED=NO
```

No `PLAT051` should be allocated for this repair merely to restate the
D101-style private capability pattern.

If implementation exposes a materially different requirement — for example a
new general standard-module authority model, zero-copy ownership transfer,
public Array growth semantics, a new transferable builder value, or a global
runtime registry — the implementation slice must stop and route that exact new
choice through the normal decision process.

## Evidence strategy for the repair

A2 showed that complete-parser timing/allocation scaling is a poor retained
instrument. The approved architecture permits more direct deterministic
evidence.

The implementation slice should retain focused evidence for at least:

```text
BUILDER_STATE=INVOCATION_LOCAL
APPENDS_FOR_N_VALUES=N
FINAL_ARRAY_MATERIALIZATIONS=1
APPEND_AFTER_FINISH=REJECTED
SECOND_FINISH=REJECTED
FINAL_RESULT=ORDINARY_STANDARD_ARRAY
ELEMENT_ORDER=PRESERVED
ELEMENT_IDENTITY=PRESERVED
PUBLIC_BOOTSTRAP_SLOT_AFTER_MODULE_INIT=ABSENT
GLOBAL_MUTABLE_BUILDER_STATE=ABSENT
```

The existing CommandLine semantic corpus remains the authority for unchanged
observable parser behavior.

The old A2 full-parse stress experiment must not be restored or rerun merely to
claim the linear repair.

## Scope intentionally not absorbed

This approval does not authorize opportunistic migration of the similar
balanced-chunk builders elsewhere in Test Tool or TOML tooling. Those sites may
be audited separately if their actual scale requirements justify it.

It also does not authorize:

- a generic public Array builder;
- a Core/prelude builder binding;
- a Java implementation of CommandLine parsing;
- changes to Array COW/snapshot representation;
- zero-copy storage transfer;
- global parser caches/registries;
- weakening D115; or
- reopening LIB011 / #428.

## Next slice

```text
NEXT_SLICE=AUD006-A4
NEXT_SLICE_TITLE=implement private linear CommandLine Array construction
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
AUD006_A4_STATUS=READY
AUD006_B_STATUS=BLOCKED_BY_A4
AUD006_C_STATUS=READY_INDEPENDENT
AUD006_D_STATUS=BLOCKED_BY_A4_B_C
```

AUD006-A4 may implement only the approved narrow repair and its focused retained
evidence. After A4 publication and validation, AUD006-B can proceed with the
independent command-depth robustness work.

AUD006 remains open. LIB011 / #428 remains closed.
