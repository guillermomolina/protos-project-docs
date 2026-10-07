# PLAT041 — RootTag/materialized-local decision evidence

Date: 2026-10-01

## Identity

~~~text
WORK_ITEM=PLAT041/#759
PARENT=PERF025/#758
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=6811d0cef3735d39ffd3801b3bae6ef48318bb66
PROTOS_VERSION=0.3.131-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
SELECTED_CANDIDATE=C_PRIME
SEMANTIC_CHANGE=NO
~~~

This record retains the evidence supporting
`PLAT041_BYTECODE_ROOT_TAG_MATERIALIZED_LOCAL_GROUPING_BOUNDARY.md`.

## Trigger evidence

PERF025-C1 stopped with a clean working tree before implementation.

The blocking facts established from current product source and generated
Bytecode DSL output were:

1. automatic root tagging is selected at generated-interpreter configuration
   level, not per root;
2. current canonical lowering creates lexical owner roots, nested Closure roots
   and Object-body helper roots inside shared `BytecodeRootNodes` groups;
3. PERF013 deliberately established that shared-generation topology so
   captured accesses can use `MaterializedLocalAccessor`; and
4. tagging the current mixed interpreter would therefore expose helper roots,
   while splitting Object bodies into another generated group would broadly
   lose the same-generation fast path in common Object/method shapes.

No product files were modified by the blocked attempt.

## Truffle 25.4 mechanism evidence

Authoritative API references used by the decision packet:

- `GenerateBytecode`:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/GenerateBytecode.html
- `GenerateBytecodeTestVariants`:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/GenerateBytecodeTestVariants.html
- `BytecodeRootNodes`:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/BytecodeRootNodes.html
- `MaterializedLocalAccessor`:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/MaterializedLocalAccessor.html
- `OperationProxy`:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/OperationProxy.html

Decision-relevant conclusions:

~~~text
AUTOMATIC_ROOT_TAGGING_PER_ROOT_OPT_OUT=NO_CURRENT_SUPPORTED_MECHANISM
GENERATE_BYTECODE_TEST_VARIANTS=TEST_ONLY
TWO_PRODUCTION_CONFIGURATIONS_REQUIRE_TWO_REAL_GENERATED_INTERPRETERS=YES
OPERATION_PROXY_CAN_SHARE_OPERATION_IMPLEMENTATION=YES
OPERATION_PROXY_MERGES_ROOT_GROUP_IDENTITY=NO
MATERIALIZED_LOCAL_ACCESS_REQUIRES_COMPATIBLE_SAME_GENERATION_ROOT_IDENTITY=YES
~~~

PLAT026's existing future escape hatch remains valid: a later supported
per-root automatic classification mechanism could simplify the implementation
without changing semantic/tooling membership.

## Current Protos source evidence

### Root configurations

`ProtosBytecodeRootNode` is currently the full source/helper Bytecode
interpreter and uses:

~~~text
enableYield=true
enableTagInstrumentation=true
enableRootTagging=false
enableRootBodyTagging=false
enableMaterializedLocalAccesses=true
~~~

`ProtosSemanticBytecodeRootNode` is the current small semantic shell and uses
automatic RootTag plus continuation forwarding into a helper target.

### Closure execution

`ProtosBytecodeClosureExecutionPlan` currently builds an untagged activation
root and wraps its target with `ProtosSemanticBytecodeRootNode.wrap(...)`.

That produces two nested Bytecode `CallTarget` invocations for an ordinary
source-backed Closure.

Current BUG008 comments identify this doubled target depth as the reason guest
recursion consumes more host stack and reusable guest execution provisions a
dedicated 64 MiB stack.

### Object construction

Current `PrepareObjectConstruction`:

- creates a new `ProtosObjectValue`;
- creates `ProtosActivation.forObjectConstruction(object, enclosing)`; and
- packages that activation with the Object-body `RootCallTarget`.

`EnterObjectConstruction` then calls the helper root with the construction
activation.

`ProtosActivation.forObjectConstruction` establishes the important semantic
facts independently of the helper root:

- construction context and receiver are the new Object;
- captured lexical contexts come from the enclosing activation's
  `lexicalContextsForClosureCapture()`;
- prelude, return home, method home, module state and execution domain are
  inherited;
- Task identity is attached when present;
- otherwise exact dynamic-control ownership is inherited; and
- `construction=true` prevents the construction context from becoming an
  additional lexical capture scope.

`MaterializeClosure` captures
`activation.lexicalContextsForClosureCapture()`, receiver, method home, return
home and prelude. Therefore using the construction activation as the active
activation while the Object body executes preserves the existing closure
materialization semantics without requiring the Object body itself to be a
root.

### Canonical grouping

`CanonicalToBytecodeLowerer.lowerNestedObjectBodyRoot(...)` currently lowers
Object bodies with:

~~~text
genuineExecutionContextRoot=false
~~~

PERF013 A3 placed that helper root in the same physical generated group as the
lexical owner and nested Closures. The helper remained explicitly non-lexical.

This establishes that removing the physical helper root does not remove a
semantic lexical owner.

### Current activation load surface

The decision investigation found roughly 70 direct
`builder.emitLoadArgument(0)` activation loads in the canonical lowerer.

Candidate C′ therefore selects one current-activation emission authority rather
than a scattered per-operation Object-body rewrite.

## Infrastructure-root inventory

At the ratification baseline, production uses of
`ProtosBytecodeRootNodeGen.create(...)` outside
`CanonicalToBytecodeLowerer` occur in six classes:

~~~text
ProtosTaskCPrimeEntryExecution
ProtosTextWriterCPrimeExecution
ProtosTextReaderCPrimeExecution
ProtosIoReleaseCPrimeExecution
ProtosBufferedByteReaderCPrimeExecution
ProtosBufferedByteWriterCPrimeExecution
~~~

The union of builder operation names used by those six builders is 52 at this
baseline, including ordinary Bytecode DSL built-ins such as Root, Block,
LoadArgument, LoadLocal, StoreLocal, While, TryCatch, TryFinally, Yield and
Return.

This falsifies the assumption that a truthful tagged source interpreter requires
a second complete untagged source-language interpreter after Object-body roots
are removed.

## PERF013 retained evidence

The durable PERF013 cross-generation record establishes:

~~~text
same BytecodeRootNodes generation
+ owner BytecodeLocal
+ matching retained MaterializedFrame/FrameDescriptor identity
    -> MaterializedLocalAccessor fast path

cross-generation reconstructed group
+ old retained captured frame
    -> runtime lexical-authority fallback
~~~

A fresh structurally equivalent group cannot safely use its new accessor against
the old retained frame.

Candidate C′ therefore keeps lexical owners and nested Closures in the same
semantic source generation rather than attempting cross-generation repair.

## Comparative runtime evidence

PLAT041 reuses the already-ratified PLAT026 comparative tooling survey and
PERF013 lexical-lifetime survey because the same root/tooling and retained-frame
questions are being combined, not replaced.

Relevant patterns retained in the decision packet:

- Oracle SimpleLanguage: semantic function roots are the Bytecode root units;
  RootTag and RootBody boundaries are independently configurable.
- GraalPy: Bytecode roots represent Python code units suitable for semantic
  instrumentation.
- GraalJS: internal physical execution wrappers need not become debugger-visible
  guest function roots; closures retain lexical environment state.
- TruffleRuby: Proc/executable state retains its declaration frame without
  promoting runtime plumbing to guest root identity.
- Apple Pkl: function values retain their executable/root and enclosing
  materialized environment; tooling is exposed selectively.
- Espresso, Sulong and GraalWasm: semantic method/function roots are separated
  from runtime/helper machinery.
- TruffleSqueak and Enso: physical execution machinery and debugger-visible
  semantic units are not required to have one-to-one identity.

The portable conclusion is to shape physical execution so guest root identity
remains truthful instead of exposing helpers merely to satisfy backend layout.

## Candidate comparison

### A — two full tagged/untagged source interpreters

~~~text
PLAT026=PASS
PERF013_FAST_PATH=BROAD_REGRESSION
GENERATED_CODE_COST=HIGH
NATIVE_IMAGE_FOOTPRINT_COST=HIGH
SELECTED=NO
~~~

Object/method boundaries split generations and lose the primary A3 benefit.

### B — current semantic shell + untagged mixed interpreter

~~~text
PLAT026=PASS
PERF013=PASS
BUG008_DOUBLE_TARGET=RETAINED
IMPLEMENTATION_RISK=LOW
SELECTED=NO
~~~

Safe baseline, but it leaves PERF025-C's known source of stack amplification in
place.

### C′ — inline Object body + semantic source roots + compact helpers

~~~text
PLAT026=PASS
PERF013_FAST_PATH=PRESERVED
PLAT014=PASS
OBJECT_HELPER_ROOT=REMOVED
DOUBLE_SOURCE_ROOT_WRAPPER=REMOVABLE
FULL_INTERPRETER_DUPLICATION=NO
SELECTED=YES
~~~

### D — future per-root automatic RootTag classifier

~~~text
ARCHITECTURAL_FIT=EXCELLENT
CURRENT_TRUFFLE_25_4_SUPPORT=NO
SELECTED=NO_CURRENTLY
~~~

### E — promote Object helpers to semantic roots

~~~text
PLAT026_CONFLICT=YES
DEBUGGER_TRUTHFULNESS=FAIL
SELECTED=NO
~~~

## GITHUB010 scorecard

Scores 1-5; H/M is confidence.

| Criterion | A | B | C′ | D future API | E |
| --- | ---: | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 4/H | 5/H | 5/M | 5/H conceptually | 2/H |
| Protos alignment | 3/H | 4/H | 5/H | 5/H | 1/H |
| Future-option resilience | 3/M | 3/H | 5/H | 5/H | 2/H |
| Scalability | 2/H | 3/H | 5/M | 5/H | 3/M |
| Conceptual simplicity | 2/H | 5/H | 4/M | 5/H | 4/H |
| Portability / implementation freedom | 3/M | 3/H | 5/M | 4/M | 2/H |
| Runtime / resource cost | 2/H | 3/H | 5/M | 5/H | 4/M |
| Failure / operability | 3/M | 5/H | 4/M | 5/H | 2/H |
| Reversibility / migration cost | 3/M | 5/H | 4/M | 5/H | 2/H |
| Evidence maturity / implementation risk | 3/M | 5/H | 3/M | 1/H current | 3/M |
| **Total / 50** | **28** | **41** | **45** | **45 concept / unavailable** | **25** |

D is not a selectable current architecture because the required API is absent.

### Owner-focus scoring

| Candidate | Future endurance | Scalability | Protos philosophy | Mean |
| --- | ---: | ---: | ---: | ---: |
| A | 6.0 | 4.5 | 6.0 | 5.50 |
| B | 7.0 | 6.0 | 7.5 | 6.83 |
| **C′** | **9.5** | **9.0** | **10.0** | **9.50** |
| D future API | 10.0 | 10.0 | 10.0 | 10.0, unavailable |
| E | 4.0 | 6.0 | 2.0 | 4.00 |

## GITHUB021 invariant/delta check

### PLAT026

~~~text
SEMANTIC_ROOT_AUTOMATIC_ROOT_TAG=PRESERVED
OBJECT_HELPER_ROOT_TAG_EXCLUSION=PRESERVED
ROOT_BODY_TAG_DEFERRED=PRESERVED
STATEMENT_CALL_EXPRESSION_MEMBERSHIP=PRESERVED
NO_GLOBAL_ROOT_REGISTRY=PRESERVED
PLAT026_INVARIANT_DELTA=NONE
~~~

### PLAT014

~~~text
SUSPENSION_OWNERSHIP=PRESERVED
CONTINUATION_COMPOSITION=PRESERVED
NO_REPLAY=PRESERVED
PLAT014_INVARIANT_DELTA=NONE
~~~

### PERF013

~~~text
PERF013_PHYSICAL_A3_DELTA=OBJECT_HELPER_ROOT_REMOVED
SAME_GENERATION_OWNER_CLOSURE_FAST_PATH=PRESERVED
CROSS_GENERATION_RUNTIME_AUTHORITY_FALLBACK=PRESERVED
CAPTURE_BY_REFERENCE=PRESERVED
~~~

### Result

~~~text
DECISION_INVARIANT_CONSISTENCY=PASS
NEW_PROTOS_SEMANTICS=NO
~~~

## Approval provenance

The project owner explicitly approved the exact recommended candidate in the
active interaction on 2026-10-01:

~~~text
Apruebo C′ para PLAT041
~~~

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
APPROVED_CANDIDATE=C_PRIME
~~~

## Released implementation sequence

~~~text
PERF025-C1a = current-activation lowering seam
PERF025-C1b = inline Object-body execution
PERF025-C1c = semantic source RootTag cutover + compact helper interpreter
PERF025-C2  = BUG008 carrier retirement and before/after measurement
~~~

Each slice remains bounded. C1a deliberately changes no physical root topology.

## Cross references

- Product revision: `6811d0cef3735d39ffd3801b3bae6ef48318bb66`
- PLAT041: `guillermomolina/protos#759`
- PERF025: `guillermomolina/protos#758`
- PLAT026 follow-up: `guillermomolina/protos#690`
- PERF013: `guillermomolina/protos#724`
- BUG008: `guillermomolina/protos#681`
- Durable PLAT026 decision:
  `docs/project/decisions/platform/PLAT026_BYTECODE_PRODUCTION_ROOT_TAG_COMPATIBILITY_BOUNDARY.md`
- PERF013 platform evidence:
  `docs/project/evidence/PERF013/PERF013_SLICE_C_CROSS_GENERATION_PLATFORM_BOUNDARY.md`
