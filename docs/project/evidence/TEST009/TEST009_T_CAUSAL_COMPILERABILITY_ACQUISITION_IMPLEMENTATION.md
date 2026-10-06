# TEST009-T — causal compilerability acquisition implementation

Date: 2026-10-06

Owning work item: `TEST009 / guillermomolina/protos#795`

Implementation repository: `guillermomolina/protos`

This record captures the published TEST009-T implementation selected by TEST009-S.
It is non-normative project evidence. It records the diagnostic acquisition
machinery only; it does not classify the residual `CodeTooLarge` cause and does
not authorize a compilerability repair.

## Exact published product identity

~~~text
PROTOS_REVISION=b6f62a1f52a4d26cd026beb69d3c28a6730c83b7
COMMIT_SUBJECT=TEST009-T: add causal compilerability acquisition
PROTOS_VERSION=0.3.229-SNAPSHOT
~~~

Published paths:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosCompilerabilityJson.java
src/main/java/com/guillermomolina/protos/execution/ProtosCompilerabilityListenerBridge.java
src/main/java/com/guillermomolina/protos/execution/ProtosCompilerabilityRecorder.java
src/main/java/com/guillermomolina/protos/execution/ProtosCompilerabilityRootIdentity.java
src/main/java/com/guillermomolina/protos/execution/ProtosCompilerabilityTrace.java
src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosCompilerabilityTraceTest.java
tools/test_truffle_compilation_gate.py
tools/truffle_compilation_gate.py
tools/truffle_compilerability_causal.py
~~~

## Maintainer-reported validation

The maintainer reports after publication:

~~~text
MAINTAINER_REPORTED_GIT_DIFF_CHECK=PASS
MAINTAINER_REPORTED_ALL_LOCAL_TESTS=PASS
PUBLICATION=PUSHED
~~~

These are maintainer-reported results; the documentation agent did not rerun the
product validation.

## Implemented diagnostic boundary

TEST009-T preserves the TEST009-S requirement that the new machinery is
observation-only.

Activation is private and diagnose-only:

~~~text
PROPERTY=protos.compilerability.causalTrace
PROPERTY_VALUE=true
ORDINARY_RUNTIME_ENABLED=NO
STRICT_SYNC_ENABLED=NO
STRICT_BACKGROUND_ENABLED=NO
DIAGNOSE_ENABLED=YES
~~~

`tools/truffle_compilation_gate.py` adds the property only to
`DIAGNOSTIC_OPTIONS`. The strict SYNC/BACKGROUND option sets remain unchanged.

The Truffle optimizing runtime listener is integrated reflectively. The existing
Maven dependency plane remains unchanged:

~~~text
TRUFFLE_RUNTIME_SCOPE=runtime
COMPILE_SCOPE_PROMOTION=NO
DEPENDENCY_VERSION_CHANGE=NO
~~~

The listener is installed once per JVM on first Protos Context creation when the
private property is enabled. Missing/incompatible runtime listener contracts
produce machine-readable installation errors instead of silently degrading the
acquisition.

## Causal event stream

The runtime emits one prefixed JSON line per compilation lifecycle callback:

~~~text
CAUSAL_EVENT_SCHEMA=protos.compilerability.causal/v1
CAUSAL_PREFIX=[protos-compilerability]
~~~

The recorded lifecycle is:

~~~text
start
  -> truffle_tier
  -> graal_tier
  -> success | failure
~~~

Run-local target identity is used only inside the acquisition to correlate
callbacks. It is not a durable regression identity.

Each start record includes the stable root identity and run-local engine/target
ids required to join the same compilation to the existing engine trace.

Truffle-tier records include:

~~~text
TRUFFLE_TIER_GRAPH_NODE_COUNT
TOP_NODE_TYPES
INLINING_CALL_COUNT
INLINED_CALL_COUNT
INLINED_TARGET_DURABLE_KEYS
~~~

Graal-tier records include:

~~~text
GRAAL_TIER_GRAPH_NODE_COUNT
TOP_NODE_TYPES
~~~

Success records additionally expose, where the runtime provides them:

~~~text
COMPILATION_ID
TARGET_CODE_SIZE
TOTAL_FRAME_SIZE
EXCEPTION_HANDLER_COUNT
INFOPOINT_COUNT
~~~

Failure records retain:

~~~text
FAILURE_PHASE_REACHED
TIER
BAILOUT
PERMANENT_BAILOUT
FAILURE_REASON
~~~

No exact failed-install target-code size is fabricated. A `CodeTooLarge`
record can therefore retain only the proven relationship:

~~~text
INSTALL_SIZE_RELATION=GREATER_THAN_JVMCI_NMETHOD_SIZE_LIMIT
~~~

## Durable root identity

TEST009-T explicitly rejects object addresses, generated node ids, lambda
addresses and other process-local labels as durable identities.

Generated continuations normalize to their source root and retain the resume
bytecode index.

Semantic generated roots are keyed from the lowerer's stable source facts:

~~~text
ROOT_FAMILY
SEMANTIC_ROOT_KIND
SOURCE_URI
SOURCE_START
SOURCE_LENGTH
CONTINUATION_RESUME_BCI_WHEN_APPLICABLE
~~~

The source span is recorded only while the causal trace is enabled, avoiding
source-information reparsing on ordinary execution.

Untagged structured-dispatch/C-prime roots use a normalized instruction-stream
fingerprint because they have no semantic source identity.

A root without enough stable metadata fails closed instead of falling back to a
physical object identity.

## Harness integration

A new parser:

~~~text
tools/truffle_compilerability_causal.py
~~~

joins the machine-readable lifecycle stream with the already-enabled textual
compiler diagnostics:

~~~text
TraceCompilation
TraceMethodExpansion=truffleTier
MethodExpansionStatistics=truffleTier
TraceNodeExpansion=truffleTier
NodeExpansionStatistics=truffleTier
TraceInlining=true
~~~

The diagnose report advances to:

~~~text
DIAGNOSE_REPORT_SCHEMA=protos.truffle-compilation.diagnose/v3
~~~

Each shard retains its full raw log and gains:

~~~text
causal_compilations
CAUSAL_ACQUISITION
CAUSAL_COMPILATIONS
CODE_TOO_LARGE_CAUSAL_RECORDS
SEMANTIC_CODE_TOO_LARGE_CAUSAL_RECORDS
~~~

Method/node expansion evidence is attributed only while exactly one compilation
window is active. Malformed lifecycle events, duplicate/missing phases,
unattributed expansion traces, overlapping/ambiguous text windows, missing
durable keys, missing required tier events and durable-key collisions make the
causal acquisition `INCOMPLETE`.

No "most probable" attribution is allowed.

## Tests

The published change adds focal Java/JUnit coverage for:

- diagnose-property activation and disabled-by-default behavior;
- exactly-once activation under concurrent Context creation;
- exact reflective runtime listener contract;
- machine-readable fail-closed installation errors;
- success and `CodeTooLarge` lifecycle correlation;
- correlation violations;
- deterministic source-based durable keys;
- continuation normalization;
- roots without stable metadata failing closed;
- instruction-name normalization;
- JSON escaping and single-line event encoding.

The Python harness tests were also extended for the causal parser and
diagnose-report behavior.

## Semantic / compiler-policy audit

~~~text
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
COMPILER_POLICY_CHANGE=NO
DEPENDENCY_CHANGE=NO

NEW_TRUFFLE_BOUNDARY_BATCH=NO
GRAPH_LIMIT_CHANGE=NO
INLINING_BUDGET_CHANGE=NO
SPLITTING_POLICY_CHANGE=NO
NATIVE_BODY_PIC_REINTRODUCTION=NO
WARNING_ALLOWLIST_CHANGE=NO
~~~

TEST009-T therefore implements the observer selected by S without selecting or
attempting the eventual `CodeTooLarge` remedy.

## Next causal step

TEST009 remains open.

The implementation is now capable of acquiring the missing evidence selected by
TEST009-S. The next bounded step is one real causal diagnostic acquisition on the
post-T code, followed by classification of the acquired records against the S
hypothesis matrix.

No compilerability source repair is authorized before that acquisition is
interpreted.

~~~text
TEST009_T=COMPLETE
TEST009_COMPLETE=NO

NEXT_SLICE=TEST009-U
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SCOPE=ONE_REAL_CAUSAL_DIAGNOSTIC_ACQUISITION_AND_EVIDENCE_CAPTURE
NEXT_REPOSITORY=guillermomolina/protos

SOURCE_REPAIR_AUTHORIZED=NO
TRIAL_AND_ERROR_ALLOWED=NO
~~~

AI assistance: this durable record was drafted with ChatGPT from the exact
published TEST009-T product commit and the maintainer-reported validation state.
