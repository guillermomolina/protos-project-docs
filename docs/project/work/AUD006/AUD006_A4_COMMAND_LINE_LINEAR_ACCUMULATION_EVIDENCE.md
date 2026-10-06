# AUD006-A4 — CommandLine linear accumulation implementation evidence

## Status

```text
WORK_ITEM=AUD006
ISSUE=guillermomolina/protos#453
SLICE=AUD006-A4
SLICE_TYPE=IMPLEMENTATION
STATUS=COMPLETE
PRODUCT_REVISION=845a1103b031abcf95d8ba852e0d780ba6b6591a
PRODUCT_REVISION_SUBJECT=AUD006-A4: make CommandLine accumulation linear
IMPLEMENTATION_VERSION=0.3.228-SNAPSHOT
SPECIFICATION_CHANGED=NO
PUBLIC_API_CHANGED=NO
OBSERVABLE_COMMANDLINE_SEMANTICS_CHANGED=NO
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

This is durable non-normative implementation evidence for AUD006-A4. It records
the published repair of AUD006 finding F1. It does not define Protos language or
Standard Library semantics.

## Governing decision

AUD006-A3 was explicitly approved by the project owner before implementation.
The approved repair family was:

```text
D101_STYLE_PRIVATE_BOOTSTRAP_CAPABILITY_WITH_LINEAR_HOST_ACCUMULATION
```

Its required invariants were:

- CommandLine parsing and policy stay in Protos;
- the host mechanism is limited to private Array construction;
- each builder owns invocation-local private accumulation state;
- append is linear/amortized O(1) and performs no Array prefix
  rematerialization;
- finish performs one O(N) ordinary standard Array materialization;
- no zero-copy ownership transfer is introduced;
- no public Array API or representation changes;
- the bootstrap capability is not left on the final observable CommandLine
  module surface;
- D111, D115, D118 and D119 remain unchanged;
- LIB011 / #428 remains closed; and
- no PLAT051 or new Dxxx is required unless implementation exposes a materially
  different architecture.

A3 durable evidence:

`guillermomolina/protos-project-docs@3584a822b70cd900b8a7af9375bc39bd40794027`

`docs/project/work/AUD006/AUD006_A3_PRIVATE_LINEAR_ARRAY_CONSTRUCTION_BOUNDARY.md`

## Published implementation

Product revision:

```text
845a1103b031abcf95d8ba852e0d780ba6b6591a
AUD006-A4: make CommandLine accumulation linear
```

The commit changes exactly:

```text
CHANGELOG.md
pom.xml
protos/lib/cli/CommandLine.protos
src/main/java/com/guillermomolina/protos/execution/ProtosCommandLineArrayConstructionFacility.java
src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java
src/test/java/com/guillermomolina/protos/execution/ProtosCommandLineArrayConstructionFacilityTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosCoreNativeBoundaryArchitectureTest.java
```

No specification file is modified.

The implementation version advances to:

```text
0.3.228-SNAPSHOT
```

## F1 repair

The previous guest-side balanced builder inside `CommandLine.parse` used
singleton Arrays, recursive carry merging and final chunk collection:

```text
Array(value)
Array(...left, ...chunk)
Array(...result, ...chunk)
```

AUD006-A1 proved that this causes `Theta(N log N)` element-reference copying
under the ordinary materializing Array constructor.

A4 removes that mechanism from `CommandLine.parse`.

The new internal wrappers are:

```text
newBuilder()        -> arrayBuilderFactory()
appendBuilt(b, v)   -> b.append(v)
finishBuilder(b)    -> b.finish()
```

The existing logical consumers remain unchanged:

- option occurrence accumulation;
- raw positional accumulation; and
- final positional occurrence accumulation.

The supplied-arguments snapshot remains the ordinary one-shot
`Array(...suppliedArguments)`; it was not part of F1 and is not redesigned.

## Private construction facility

A4 adds:

`ProtosCommandLineArrayConstructionFacility`.

The facility defines the exact standard module identity:

```text
std:cli/CommandLine
```

and a temporary bootstrap slot:

```text
_commandLineArrayBuilderFactory
```

The shared factory:

- is one frozen ordinary Protos object;
- exposes only `call`;
- owns no mutable per-parse state; and
- is registered only as an initial member of the exact CommandLine ModuleKey.

Each zero-argument factory call creates a fresh ordinary builder object and one
fresh invocation-local `BuilderState`.

The builder exposes only the private native closures:

```text
append(value)
finish()
```

Its state is:

```text
ArrayList<Object> values
OPEN while values != null
CONSUMED after successful finish
```

There is no global builder registry, parser cache, or static mutable
per-invocation state.

## Linear accumulation proof

For one builder receiving N values:

```text
append:
  ArrayList.add(exactValue)
  no Array construction
  amortized O(1) per append
  O(N) total

finish:
  capture accumulated ArrayList
  mark builder CONSUMED by clearing the state reference
  prelude.newArray(accumulated)
  one ordinary standard Array construction
  O(N)

total:
  O(N) + O(N) = O(N)
```

The final `ProtosArrayValue` constructor copies the accumulated references into
its own storage once. The host `ArrayList` is not adopted as the Array backing,
so the approved no-zero-copy/no-ownership-transfer boundary is preserved.

The asymptotic claim does not depend on Truffle partial evaluation, JIT
compilation, a native-body PIC, or Native Image specialization.
`append` and `finish` are explicitly host boundaries and the structural cost
remains linear when interpreted.

Final F1 classification:

```text
F1_BALANCED_CHUNK_BUILDER_REMOVED=YES
APPEND_PREFIX_ARRAY_REMATERIALIZATION=NO
COMMANDLINE_BUILDER_APPEND_COST=AMORTIZED_O1
COMMANDLINE_BUILDER_N_APPENDS=O_N
COMMANDLINE_BUILDER_FINISH=O_N
COMMANDLINE_BUILDER_TOTAL=O_N
FINAL_ARRAY_MATERIALIZATIONS_PER_BUILDER=1
ZERO_COPY_OWNERSHIP_TRANSFER=NO
D115_F1_LINEAR_ACCUMULATION_TARGET=RESTORED
F1_STATUS=RESOLVED
```

## Lifecycle and failure behavior

The builder is one-shot.

`BuilderState.append` rejects a consumed builder.

`BuilderState.finish` changes the state to CONSUMED before returning the
result. A second finish is rejected through the private closure boundary.

The private native closures also require:

- the exact builder receiver;
- exactly one argument for `append`;
- exactly zero arguments for `finish`; and
- non-null stored host references.

Invalid private-facility use signals ordinary Protos Error through the existing
runtime error mechanism.

The builder result is an ordinary open standard Array. Existing CommandLine
source remains responsible for freezing result Arrays at the same semantic
points as before, preserving the previous observable lifecycle.

## Bootstrap visibility

`ProtosCoreBootstrap.standardModuleMembers(...)` registers the frozen factory
only under:

`ProtosCommandLineArrayConstructionFacility.MODULE_KEY`.

During `CommandLine.protos` initialization, `parse` is created through a
private lexical capture of the temporary factory.

After that capture, module initialization executes:

```text
context.removeSlot("_commandLineArrayBuilderFactory")
```

Therefore later parse calls use the captured capability rather than looking up
the removed module binding.

The published module surface remains exactly:

```text
option
positional
command
parse
renderHelp
```

Final visibility classification:

```text
FACTORY_PROVISIONED_TO_EXACT_COMMANDLINE_KEY_ONLY=YES
PARSE_CAPTURES_PRIVATE_CAPABILITY=YES
PARSE_DEPENDS_ON_REMOVED_MODULE_SLOT=NO
FINAL_COMMANDLINE_BOOTSTRAP_SLOT_PRESENT=NO
COMMANDLINE_PUBLIC_SURFACE_CHANGED=NO
```

## Retained evidence

A4 adds
`ProtosCommandLineArrayConstructionFacilityTest`.

It directly covers:

- frozen/stateless factory placement at the exact CommandLine ModuleKey;
- no factory member on an unrelated standard-module key;
- fresh independent builder per factory call;
- exact value identity and order preservation;
- ordinary standard Array parentage;
- open result before the existing CommandLine freeze point;
- repeated finish rejection;
- append-after-finish rejection;
- exact receiver and arity validation;
- N direct state appends with zero materializations before finish;
- exactly one materialization after successful finish;
- consumed state after finish;
- imported CommandLine module absence of the bootstrap slot;
- unchanged exact public module surface; and
- successful real CommandLine positional parsing through the captured facility.

The evidence uses a bounded deterministic value count rather than restoring the
rejected A2 full-parser allocation/timing stress experiment.

`ProtosCoreNativeBoundaryArchitectureTest` is also reconciled. The new facility
is explicitly counted as an audited non-Core native-Closure provider with three
native closure construction sites. The existing Core native-provider inventory
is not expanded by silently classifying the facility as Core.

## Observable semantics and architecture

A4 changes only accumulation machinery.

```text
D111=KEEP
D115=KEEP
D118=KEEP
D119=KEEP
COMMANDLINE_POLICY_OWNER=PROTOS
PUBLIC_ARRAY_API_CHANGE=NO
ARRAY_REPRESENTATION_CHANGE=NO
COMMANDLINE_PUBLIC_API_CHANGE=NO
COMMANDLINE_OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_GLOBAL_MUTABLE_STATE=NO
PLAT051_REQUIRED=NO
NEW_DXXX_REQUIRED=NO
LIB011_ISSUE_428=KEEP_CLOSED
```

No public Array append/reserve/resize/builder/exact-size-fill API is introduced.
No Java CommandLine parser is introduced. No Test Tool/TOML balanced builder is
migrated by this slice.

## Validation

After publication, the maintainer reported:

```text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

This record does not invent individual command-level validation that the
maintainer did not report separately.

## AUD006 state after A4

```text
AUD006_STATUS=IN_PROGRESS
AUD006_A1_STATUS=COMPLETE
AUD006_A2_STATUS=NEGATIVE_EVIDENCE_COMPLETE
AUD006_A3_STATUS=COMPLETE_APPROVED
AUD006_A4_STATUS=COMPLETE
AUD006_A_STATUS=COMPLETE
AUD006_F1_STATUS=RESOLVED

AUD006_B_STATUS=READY
AUD006_C_STATUS=READY_INDEPENDENT
AUD006_D_STATUS=BLOCKED_BY_B_C

LIB011_ISSUE_428=KEEP_CLOSED
```

AUD006 remains open because F2 depth/robustness work and the independent
documentation/governance work F3-F6 remain outstanding before the final D
closure re-audit.
