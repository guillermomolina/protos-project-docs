# TEST009-O — shared host-leaf boundaries and compilerability result

Date: 2026-10-05

Owning work item: `TEST009 / guillermomolina/protos#795`

Preceding policy record:

~~~text
TEST009_N_PROJECT_RECORD_REVISION=6e721a0d99dbc46f41907765d0c11436a7a0cea2
TEST009_N_PROJECT_RECORD_PATH=docs/project/evidence/TEST009/TEST009_N_POST_M_TOO_DEEP_AND_M5_POLICY.md
~~~

## Exact product publication

~~~text
PROTOS_REVISION=53c54bb952354cae61e72db68c4ff8f509c827ed
COMMIT_SUBJECT=TEST009-O: bound shared host leaves exposed by M5 native-body PIC
IMPLEMENTATION_VERSION=0.3.221-SNAPSHOT
PRODUCT_PUSH=COMPLETE
~~~

Maintainer-reported local publication state:

~~~text
GIT_DIFF_CHECK=PASS
LOCAL_TESTS=PASS
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

## Product changes

TEST009-O retained the TEST009-M M5 global exact-native-body PIC and implemented
the three narrow host cuts selected by TEST009-N.

### Encoding host leaves

`ProtosEncodingValue` keeps Protos-facing validation and error policy visible
while bounding portable codec host transformations and one-shot String/byte
assembly.

### Context-local C-prime plan construction

The six Context-local plan caches in `ProtosLanguageContext` keep their volatile
already-built fast paths visible and move only synchronized lazy cache-miss
`createPlan` work behind `@TruffleBoundary`.

### Physical File resource close

`ProtosHostResourceClose.closePhysically` centralizes the physical host close
for the four NIO File resource implementations. Protos close admission,
idempotence, lifecycle state and completion ordering remain outside that
boundary.

The product commit also reconciles TEST009 static PE guard baselines with
current already-safe source identities. Guard risk thresholds were not relaxed.

## Sole final expensive diagnostic

The retained acquisition used one logical Case and one shard worker:

~~~text
CASE_DISPLAY=protos/corpus/conformance:call/closure-call-and-return.protos::plain closure call
SHARD_WORKERS=1
SEMANTIC_CORPUS=PASS
CORPUS_PASSED=1
CORPUS_FAILED=0

COMPILATIONS_DONE=211
COMPILATION_FAILURES=40
SHUTDOWN_CASCADE_FAILURES=0
PERFORMANCE_WARNINGS=5758
PE_CONSTANT_FAILURES=0
OTHER_PERMANENT_FAILURES=40

DIAGNOSTIC_WALL_SECONDS=1500.6
SHARD_WALL_SECONDS=1470.6
TRUFFLE_COMPILATION_DIAGNOSE=FAIL
~~~

The 40 permanent failures classify exactly as:

~~~text
CODE_INSTALLATION_TOO_LARGE=30
TOO_DEEP_INLINING=8
COMPILATION_EXCEEDED_100_SECONDS=2
COMPILER_OOM=0
PE_CONSTANT_FAILURES=0
~~~

The two compilation-timeout failures were compilation ids 1975 and 197. They
are a new permanent-failure class relative to the retained TEST009-M result and
are not reclassified as code-size or TooDeep.

## Comparison with TEST009-M

~~~text
METRIC                              TEST009-M   TEST009-O
COMPILATIONS_DONE                   194         211
COMPILATION_FAILURES                36          40
PERFORMANCE_WARNINGS                4877        5758
PE_CONSTANT_FAILURES                0           0
CODE_INSTALLATION_TOO_LARGE         21          30
TOO_DEEP_INLINING                   15          8
COMPILER_OOM                        0           0
COMPILATION_EXCEEDED_100_SECONDS    0           2
DIAGNOSTIC_WALL_SECONDS             718.3       1500.6
~~~

Directional reading:

~~~text
TOO_DEEP_INLINING=IMPROVED_15_TO_8
CODE_INSTALLATION_TOO_LARGE=REGRESSED_21_TO_30
PERFORMANCE_WARNINGS=REGRESSED_4877_TO_5758
COMPILATION_TIMEOUT_CLASS=NEW_0_TO_2
WALL_TIME=REGRESSED_718.3_TO_1500.6
MONOTONIC_COMPILERABILITY=FAIL
~~~

The TooDeep count was nearly halved, so the O boundaries had material effect.
That improvement is insufficient for acceptance because the slice's explicit
policy required no new permanent family and no material code-too-large
regression.

## Remaining TooDeep classification

The post-O detailed TooDeep traces no longer show the C-prime
`BytecodeSupport.ConstantsBuffer` construction path selected as TEST009-N
family B.

The physical NIO `FileChannelImpl/FileLockTable` close path selected as family
C is also absent from the retained TooDeep traces. However File close still
produces TooDeep through a different path:

~~~text
ProtosFileFlow.close
 -> ProtosIoLifecycle.startRelease
 -> ProtosIoLifecycle.finishClose
 -> ProtosActorExecutionDomain.terminalActorIoLifecycleCleanupForRuntime
 -> HashSet.remove / HashMap.TreeNode.find
~~~

The remaining TooDeep roots demonstrate broader host expansion opened by the
global native-body PIC:

~~~text
ProtosIoLifecycle.beginOperation
 -> HashSet.add / HashMap tree machinery
 -> TextReader readLine
 -> File read
 -> Filesystem open

ProtosIoLifecycle.authorizeFirstCloseLocked
 -> ProtosActorExecutionDomain.registerActorIoLifecycleCleanupForRuntime
 -> TextReader close

ProtosIoLifecycle.finishClose
 -> ProtosActorExecutionDomain.terminalActorIoLifecycleCleanupForRuntime
 -> HashSet.remove / HashMap tree machinery
 -> File close

ProtosTestLogicalCaseDiscoveryFacility.execute
 -> ProtosDirectFileModuleResolver
 -> Files.newDirectoryStream / UnixFileSystemProvider
~~~

The recurring JDK String/Locale/Formatter/reflection shape therefore survives,
but not as only the previously selected Encoding seam. It is reachable through
additional native bodies and lifecycle/discovery host structures.

## Falsification result for the M5 policy

TEST009-N Candidate A was a falsifiable experiment: keep the global PIC and cut
three shared host seams. TEST009-O shows that those three cuts are not a stable
global policy by themselves.

~~~text
M5_GLOBAL_PIC_POLICY_NEEDS_RECONSIDERATION=YES
NEW_PERMANENT_FAILURE_FAMILY_ALLOWED=NO
NEW_PERMANENT_FAILURE_FAMILY_OBSERVED=YES
MONOTONIC_COMPILERABILITY=FAIL
~~~

This does not by itself prove that every O boundary should be reverted. The
boundaries remain narrow, semantically neutral implementation separations and
the targeted C-prime-construction and physical-NIO-close TooDeep paths are no
longer present in the retained acquisition.

What is falsified is the assumption that a global native-body PIC can be made
stable merely by adding the three currently observed host-leaf cuts. Continuing
to chase each newly exposed host graph with another boundary batch would violate
the regression-control rule established by TEST009-N.

The next work must return to the M5 policy itself.

## Diagnostic provenance note

The diagnostic build banner reports:

~~~text
DIAGNOSTIC_MAVEN_VERSION_AT_ACQUISITION=0.3.219-SNAPSHOT
~~~

The final TEST009-O product commit is `0.3.221-SNAPSHOT`. Between acquisition
and publication, concurrent main advanced through
`I070-A1` at `a5d3fe00c941c29f0a4e80eb262be610ff01443b`, whose published commit
states that it removes stale unattached documentation comments and introduces no
executable or semantic change.

The TEST009-O workflow allowed only one expensive diagnostic, so no second
acquisition was manufactured after synchronization and metadata publication.
The maintainer separately reports the final local test suite and
`git diff --check` as passing.

## Next slice

TEST009 remains open.

~~~text
TEST009_COMPLETE=NO

NEXT_SLICE=TEST009-P
NEXT_SLICE_TYPE=INVESTIGATION
IMPLEMENTATION_AUTHORIZED=NO
NEXT_SCOPE=M5_NATIVE_DIRECT_ELIGIBILITY_VS_REVERT_AFTER_O_FALSIFICATION

NEXT_INVESTIGATION_MUST_CLASSIFY:
  - structural eligibility for PE-friendly native bodies;
  - whether host-heavy native bodies can remain on generic nativeCall;
  - whether eligibility can be compact and maintainable;
  - whether the two compilation-timeout failures share the same global-PIC cause;
  - whether any part of M5 should instead be reverted.

ANOTHER_BOUNDARY_BATCH_BEFORE_POLICY_INVESTIGATION=FORBIDDEN
ANOTHER_EXPENSIVE_DIAGNOSTIC_BEFORE_POLICY_INVESTIGATION=FORBIDDEN
CODE_TOO_LARGE_MICRO_REPAIR_BATCH=FORBIDDEN
~~~

No new formal Issue is required for this internal TEST009 slice.

AI assistance: this durable record was drafted with ChatGPT from the exact
published TEST009-O commit, the maintainer-reported local validation, the sole
retained TEST009-O diagnostic acquisition, and the preceding TEST009-N policy
record.
