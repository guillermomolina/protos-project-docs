# PERF025-D179-A — Stable current lexical-read membership assumptions

## Status

Durable non-normative implementation evidence for PERF025 /
`guillermomolina/protos#758`.

This record captures the bounded D179 lexical-membership specialization
published at exact product revision:

```text
PRODUCT_REVISION=ffc351dca7363bcded452dd4d19d30e787cac391
PRODUCT_PARENT=7d507be8940528b045aa96426c7433ba040dfc83
PRODUCT_VERSION=0.3.166-SNAPSHOT
COMMIT_SUBJECT=PERF025: specialize stable current lexical reads
OWNING_ISSUE=guillermomolina/protos#758
SLICE=PERF025-D179-A
```

No benchmark magnitude is claimed by this record.

## Bounded implementation result

The implementation specializes statically `Resolved` reads owned by the
current genuine lexical root.

Each `ProtosFrameLexicalLayout` now owns one Truffle `Assumption` per
static binding ordinal/name:

```text
PRESENT_CONTINUITY(root-definition, binding-name)
```

The token is optimization metadata only. Semantic presence remains owned by the
existing frame-local cleared state and lexical binding authority.

While the token is valid, an ordinary current-scope `ReadFrameLocal` can read
the proven frame local without first executing `LocalAccessor.isCleared(...)`.
After invalidation it returns to the exact previous presence-aware path.

The root compact-call form is narrower still: a current `Resolved` read in an
unmaterialized compact call reads the accessor directly. At that point the
guest Context for that activation has not become observable, so no D179
structural removal can have made that already-established binding absent.

## Invalidation semantics

For a static frame-backed binding, the first successful
`PRESENT -> ABSENT` removal invalidates the exact root/name token **before**
the physical local is cleared.

```text
successful remove(static name):
    invalidate PRESENT_CONTINUITY(root, name)
    clear frame local
```

The token is deliberately one-way:

- initial establishment does not invalidate it;
- ordinary PRESENT-to-PRESENT assignment does not invalidate it;
- removal of another name does not invalidate it;
- legal D179 recreation at the same static ordinal does not renew it;
- close/freeze do not create a new token.

This preserves D179 C0 semantics while allowing partial evaluation to remove the
presence test only while continuity is still valid.

## Bytecode reparse lifetime

The first implementation draft was reviewed before publication and a
Bytecode-DSL reparse lifetime hazard was found: creating the assumptions in a
fresh layout on every retained-parser replay could let an old escaped authority
invalidate an obsolete token while reparsed instructions trusted a new token.

The published implementation fixes that hazard by retaining one exact
`ProtosFrameLexicalLayout` per `CanonicalLexicalScope` in the lowerer.
`BytecodeLocal` objects remain parse-local, but the logical root's layout and
root/name assumptions survive source/instrumentation reparses.

Every replay validates that declaration-order names are identical before the
existing layout is reused.

Focused evidence explicitly creates a frame authority before
`BytecodeRootNodes.ensureComplete()`, reparses the logical root, and proves
that removal through the pre-reparse authority invalidates the exact assumption
visible after reparse.

## D179 semantics preserved

The slice preserves:

```text
STATIC_LOCAL_EXISTS != SEMANTIC_BINDING_PRESENT
PRESENT(null) != ABSENT

PRESENT -> ABSENT while OPEN = allowed
ABSENT -> PRESENT recreation while OPEN = allowed
removal may reveal lexical/receiver fallback
capture remains by reference
late nearer binding semantics remain unchanged

assignment destination selected before RHS = unchanged
post-RHS selected-destination validation = unchanged
no re-resolution after RHS = unchanged

CLOSED/FROZEN semantics = unchanged
generic fallback = retained
```

The implementation does not add a second presence bitmap or competing lexical
authority.

## Scope deliberately not implemented

This bounded slice does **not** specialize:

- captured owner-presence checks;
- captured nearer-scope scans;
- "no nearer dynamic binding introduced" assumptions;
- current or captured write selection;
- post-RHS write validation;
- binding creation;
- `Candidate` or `Dynamic` resolutions;
- a cyclic/revalidating assumption scheme.

Those surfaces remain governed by the existing exact D179 machinery.

Accordingly, this record advances the residual D179 line but does not claim the
entire broader captured-membership optimization space is closed.

## Product files changed

Exact product commit
`ffc351dca7363bcded452dd4d19d30e787cac391` changes seven files:

```text
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalLayout.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosI075DLexicalAuthorityCurrentBytecodeNodeTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025D179LexicalMembershipStabilityReadTest.java
```

GitHub reports the exact one-commit delta as:

```text
FILES_CHANGED=7
ADDITIONS=564
DELETIONS=22
```

## Validation evidence

Owner-executed validation on the final implementation candidate reported:

```text
MAVEN_COMPILE=PASS

FOCAL_TESTS_AFTER_REPARSE_FIX=4_PASSED_0_FAILED
RELATED_D179_LEXICAL_TESTS=79_PASSED_0_FAILED

FINAL_MAKE_TEST=PASS
FINAL_PROTOS_TESTS=1284_PASSED_0_FAILED
FINAL_GIT_DIFF_CHECK=PASS
```

The focused coverage includes:

- one stable token per root/name;
- independence between names;
- sharing across activations of the same lowered definition;
- assignment not invalidating membership;
- removal invalidating only the removed name;
- remove/recreate keeping the token permanently invalid;
- PRESENT(null) remaining PRESENT;
- resumed reads after invalidation observing current values;
- distinct layouts owning distinct tokens;
- token identity surviving Bytecode reparse;
- a pre-reparse authority invalidating the post-reparse token identity;
- uncached-to-cached Bytecode-node transition compatibility.

## Formal conclusions

```text
PERF025_D179_A_CURRENT_RESOLVED_READS=COMPLETE

CURRENT_RESOLVED_READ_VALID_ASSUMPTION_SKIPS_IS_CLEARED=YES
COMPACT_UNOBSERVED_RESOLVED_READ_SKIPS_IS_CLEARED=YES

ASSUMPTION_GRANULARITY=ROOT_DEFINITION_PLUS_BINDING_NAME
ASSUMPTION_ONE_WAY=YES
FIRST_SUCCESSFUL_STATIC_REMOVAL_INVALIDATES=YES
INVALIDATION_BEFORE_CLEAR=YES
RECREATE_REVALIDATES_ASSUMPTION=NO
UNRELATED_NAME_REMOVAL_INVALIDATES=NO
BYTECODE_REPARSE_RENEWS_ASSUMPTION=NO

SECOND_LEXICAL_AUTHORITY=NO
CANDIDATE_DYNAMIC_CHANGED=NO
CAPTURED_PATH_CHANGED=NO
WRITE_SEMANTICS_CHANGED=NO
CREATION_SEMANTICS_CHANGED=NO

D179_C0_EXACT=YES
PRESENT_NULL_NOT_ABSENT=YES
NO_RE_RESOLUTION_AFTER_RHS=YES
POST_RHS_SELECTED_DESTINATION_VALIDATION=YES
CAPTURE_BY_REFERENCE=YES

OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PLATFORM_DECISION_CHANGE=NO
PERFORMANCE_EFFECT_MEASURED=NO
BENCHMARK_RESULT_CLAIMED=NO

RESIDUAL_D179_LINE=PARTIALLY_ADVANCED
CAPTURED_MEMBERSHIP_SPECIALIZATION=REMAINING_IF_JUSTIFIED
PERF025_STATUS=OPEN
FINAL_BENCHMARK_PENDING=YES
```
