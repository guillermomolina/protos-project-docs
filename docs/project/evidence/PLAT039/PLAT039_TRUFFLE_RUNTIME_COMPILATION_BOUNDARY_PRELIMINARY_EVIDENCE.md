# PLAT039 — Truffle runtime-compilation boundary preliminary evidence

## Status

```text
FORMAL_IDENTIFIER=PLAT039
STATUS=OPEN_RESEARCH
RATIFIED=NO
OWNER_APPROVAL=NOT_YET_REQUESTED
IMPLEMENTATION_READY=NO

PROTOS_REVISION=d4ac1c7be600c00c785dc9742dc7aeaeab17eaec
PROTOS_VERSION=0.3.87-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.3.4.1
TRIGGER_ISSUE=guillermomolina/protos#716
IMPLEMENTATION_CONSUMER=guillermomolina/protos#711
```

This is a **preliminary evidence checkpoint**, not the GITHUB010 exhaustive
decision packet and not a ratification record. It records the concrete source
findings established while investigating the Native Image runtime-compilation
failure so that the next PLAT039 research pass does not have to rediscover them.

## Trigger

I069 retains the PLAT038 requirement that the Native Image artifact keeps the
Truffle runtime compiler available. PLAT039 was allocated because Native Image
analysis then found many blocklisted Java methods reachable from
runtime-compilable guest roots.

The raw number of reported blocklist violations is not treated as a defect count
or progress metric. Changing one reachability cut changes the graph explored by
Native Image. The relevant unit of analysis is therefore the Protos gateway that
makes a Java/host family reachable from guest runtime compilation.

## GraalVM 25.3.4.1 runtime-compilation constraint

Inspection of the GraalVM 25.3.4.1 Native Image Truffle feature establishes
these relevant constraints:

- methods annotated with `@TruffleBoundary` are deliberately outside runtime
  compilation;
- the runtime-compilation blocklist includes broad Java families relevant to
  current Protos code, including `BigInteger`, `Collection`, `List`,
  `Map`, `HashMap`, `IdentityHashMap`, `Iterable`, `Iterator`,
  `Supplier`, `Function`, `BiFunction`, and related functional interfaces;
- Native Image determines runtime-compilable reachability from Truffle guest
  roots, so ordinary Java helper reachability matters even when the helper is
  semantically implementation-only;
- GraalVM has temporary allowlisting for a limited subset of collection and
  functional-interface methods, but that is not a durable architectural
  contract Protos should rely on.

Reference:

- GraalVM 25.3.4.1
  `substratevm/src/com.oracle.svm.truffle/src/com/oracle/svm/truffle/TruffleFeature.java`.

## Current Protos source findings

### 1. Dynamic source compilation is reachable from a Bytecode guest operation

`ProtosSourceCompiler.compileBytecode()` currently performs:

```text
Source
  -> ProtosParser
  -> Canonicalizer
  -> CanonicalToBytecodeLowerer
  -> RootCallTarget
```

without a Truffle compilation boundary.

`ProtosModuleRuntime.prepareBytecodeCanonicalModule()` reaches that compiler on
a module-cache miss. The caller is reachable from the Bytecode DSL structured
import path. A cache hit, however, already has an existing module record and
should not pay for the cold source-loading/compilation path.

Preliminary classification:

```text
module cache hit / cached target execution = KEEP_PE
module resolution/load/materialize/parse/canonicalize/lower miss path = SPLIT
cold source compiler helper = BOUNDARY
guest target execution after compilation = KEEP_PE
```

The boundary must not wrap the later guest `CallTarget` execution.

### 2. Integer arithmetic combines two blocklisted mechanisms

`ProtosStandardIntegerProtocol` installs arithmetic using
`BiFunction<BigInteger, BigInteger, BigInteger>`, including
`BigInteger::add`, `subtract`, and `multiply`.

This puts both a generic Java functional interface and `BigInteger` on an
ordinary guest arithmetic path.

SimpleLanguage provides a directly relevant pattern: keep the guest
specialization visible while placing the individual `BigInteger` operation
behind a small leaf `@TruffleBoundary(allowInlining = true)` helper.

Preliminary classification:

```text
arity/type/semantic checks = KEEP_PE
generic BiFunction dispatch = REMOVE_FROM_HOT_PATH
individual BigInteger operation = NARROW_LEAF_BOUNDARY
ProtosIntegerValue construction/result = KEEP_PE
```

`ProtosBinary64Rounding.divideExactIntegers()` also requires review because it
contains substantial `BigInteger` work.

### 3. Physical I/O and host integration are boundary work

The NIO filesystem backends reach host APIs such as:

- `Files`;
- `SecureDirectoryStream`;
- `SeekableByteChannel`;
- `ByteBuffer`;
- host `OutputStream` routing.

These are physical host effects, not guest computation that benefits from
partial evaluation.

The portable Protos lifecycle/Future/state-machine semantics remain distinct
from those physical host calls. Broadly boundary-wrapping the complete Protos
Future or C-prime orchestration would hide guest/runtime semantics that still
need to compose correctly.

Preliminary classification:

```text
portable Future/lifecycle/C-prime state transitions = KEEP_PE_OR_EXPLICIT_RUNTIME_STATE
physical NIO/socket/stream operation = BOUNDARY
host resource acquisition/release adapter = BOUNDARY
guest callback execution after completion = KEEP_PE / MUST_NOT_RUN_INSIDE_BOUNDARY
```

### 4. Call preparation still contains JDK collection machinery in the hot path

`ProtosBytecodeRootNode` currently contains hot-path structures such as:

- `PreparedArgumentVector` backed by `ArrayList<Object>`;
- `List.copyOf` snapshots;
- `PreparedClosureCall.supplied` as `List<?>`;
- `List.of(supplied)` for variadic arguments;
- spread handling through `addAll(array.indexedSnapshot())`;
- structured Map/IdentityMap/Array match and iteration snapshots based on
  `List`/`ArrayList`.

These are on closure-call, method-send, structured collection, and argument
preparation paths. A broad Truffle boundary around them would cut exactly the
guest call/dispatch work that partial evaluation should optimize.

Preliminary classification:

```text
call/send preparation semantics = KEEP_PE
ArrayList/List representation on hot path = SPLIT/REPLACE, NOT BROAD_BOUNDARY
cold snapshot/debug/interop conversion = BOUNDARY_CANDIDATE
```

A PE-friendly fixed representation such as an `Object[]`-based or
Truffle-specific carrier should be compared in the exhaustive decision packet;
no representation is selected here.

### 5. Ordinary object slots remain map-backed

`ProtosMapBackedLexicalBindingAuthority` stores ordinary local slots in:

```text
Map<String, Object> bindings = new LinkedHashMap<>();
```

Ordinary lookup reaches `containsKey`, `get`, `put`, and `remove` through
that authority.

I068 has already moved eligible execution-context lexical bindings onto
frame-backed storage, but ordinary object-slot lookup still has this Java Map
representation.

Preliminary classification:

```text
ordinary guest slot lookup/read/write = KEEP_PE
LinkedHashMap representation under ordinary object slots = SPLIT/REPRESENTATION_REVIEW
boundary around ProtosValueLookup.lookup() = REJECT_PRELIMINARILY
```

This is directly relevant to the PERF010 slot-read/method-call/monomorphic
dispatch problem and should be evaluated as a hot representation question, not
only as a Native Image acceptance problem.

### 6. Generic Java continuation/rematerialization carriers require structural review

Current runtime structures include Java functional carriers such as
`Supplier<Object>`, `Supplier<ProtosClosureExecutionPlan>`, and `Runnable`
around suspension, host execution, cancellation/release, and deferred execution
plan materialization.

Because `Supplier` and related functional interfaces are blocklisted, the
decision packet must distinguish:

- host-only callbacks that can be cut off safely;
- explicit runtime state that should replace a generic Java closure;
- guest continuation semantics that must remain visible and must **not** be
  hidden behind a broad boundary.

In particular, a boundary that encloses guest resumption or calls back into
guest `CallTarget` execution would defeat the objective.

Preliminary classification:

```text
host wait/resource callback = BOUNDARY_CANDIDATE
generic Supplier/Runnable crossing guest-hot path = SPLIT
guest continuation/resume decision = KEEP_PE
guest execution after resume = KEEP_PE
```

### 7. Existing `ProtosValueLookup.materializeMemberRead()` boundary

The only production `@TruffleBoundary` found in the inspected Protos code is
the helper that materializes a member read and binds a Closure to receiver/home.

Further source inspection shows that ordinary prepared message send uses
`ProtosValueLookup.lookup()` and carries `home` separately; it does not need
to materialize the bound Closure through this helper on the normal send path.

`ProtosClosureValue.bindMethod()` itself copies local slots through a map
snapshot, so the existing boundary has a plausible cold/materialization role.

Preliminary classification:

```text
ProtosValueLookup.lookup() = KEEP_PE
materializeMemberRead()/bindMethod materialization = KEEP_EXISTING_BOUNDARY_PROVISIONALLY
ordinary send must not be routed through materializeMemberRead merely for Native Image
```

## Preliminary gateway matrix

| Gateway / surface | Preliminary class | Reason |
|---|---|---|
| Bytecode send/call/lookup | KEEP PE | Core guest optimization surface |
| Frame-backed lexical read/write | KEEP PE | I068 optimization/semantic authority |
| Structured control and C-prime guest state transitions | KEEP PE | Guest/runtime semantics |
| Module cache hit + cached target execution | KEEP PE | Hot reusable guest path |
| Module miss load/parse/canonicalize/lower | SPLIT + BOUNDARY | Cold compiler/host work |
| BigInteger arithmetic operation | NARROW BOUNDARY | Graal blocklisted leaf operation |
| BiFunction-based arithmetic dispatch | SPLIT/REMOVE | Generic blocklisted functional carrier on hot path |
| Physical NIO/filesystem/socket/stream access | BOUNDARY | Host effect |
| Argument List/ArrayList preparation | SPLIT/REPLACE | Hot guest path; broad boundary would damage PE |
| Ordinary object LinkedHashMap slots | SPLIT/REPRESENTATION REVIEW | Hot lookup; broad boundary is wrong |
| Supplier/Runnable continuation carriers | SPLIT | Separate host callback from guest continuation |
| Debug/snapshot/diagnostic Java collection conversion | BOUNDARY CANDIDATE | Cold/non-semantic materialization |
| Existing member-read materialization boundary | KEEP PROVISIONALLY | Not ordinary send path |

## Interaction with I068 and PERF010

This evidence changes the interpretation of the recent Bytecode DSL migration
without invalidating it.

I068 moved a substantial part of lexical execution state into Bytecode
DSL/frame-backed authority. That is aligned with the PE-safe direction, but it
does not by itself remove:

- JDK collections from call argument preparation;
- ordinary-object map-backed slot storage;
- generic Java functional carriers;
- cold compiler/host work accidentally reachable from guest roots.

Therefore the absence of an automatic large performance gain after I068 is not
evidence that the Bytecode DSL migration was the wrong architectural direction.
It is evidence that DSL adoption was necessary but did not complete the
runtime-compilation/representation boundary.

PERF010 should consume PLAT039 results as architecture evidence, but PLAT039
must not claim that any one boundary leak explains a measured attributable
fraction of PERF010 overhead without a causal performance experiment.

## Relevant precedent already inspected

The preliminary pass inspected or used these Truffle precedents:

- SimpleLanguage: `println`, generic `eval`, and `BigInteger` leaf boundaries;
- GraalJS: parsing/source-compilation boundaries;
- GraalPy: compile/parsing and host resource boundaries;
- Espresso: filesystem/socket/read/write host boundaries;
- Sulong/LLVM: dependency load/parse cold boundary;
- TruffleSqueak: file, external-call, image and primitive boundaries;
- TruffleRuby: host/process/native utility boundaries.

This is **not yet sufficient for GITHUB010 completion**. The final PLAT039
packet must still perform the required broad Truffle survey, explicitly include
Apple Pkl when materially comparable (or record why it is not), add useful
outside-Truffle evidence, construct the full candidate set, perform the required
1–5 scoring with confidence, run the anti-overengineering/adversarial gates, and
present an exact recommendation pending project-owner approval.

## Preliminary architecture rule

The strongest rule established so far is:

> A method should not become a Truffle boundary merely because it uses Java.
> Use a boundary when the hidden work is an opaque host/cold effect whose
> internals do not need guest partial evaluation. If the method performs guest
> dispatch, slot access, call preparation, continuation composition, or calls
> back into guest code, split the host mechanism from the guest operation
> instead of hiding the entire path.

A corollary is that a Truffle boundary should normally be narrow and should not
enclose a guest callback or guest `CallTarget` invocation.

## Candidate directions to carry into the exhaustive packet

These are research candidates, not selected architecture:

1. **A — Status quo / ad-hoc annotation repair.**
   Add boundaries only where the current Native Image build complains.
   Preliminary concern: unstable reachability-driven whack-a-mole and high risk
   of cutting guest-hot paths.

2. **B — Host/cold leaf boundaries only.**
   Isolate parsing, physical I/O, host services and leaf blocklisted primitives,
   while leaving current hot representations unchanged.
   Preliminary benefit: smallest safe change; preliminary concern: hot JDK
   collections and functional carriers remain structural liabilities.

3. **C — Explicit PE-visible guest kernel plus host gateways and hot-carrier
   refactoring.**
   Combine narrow host/cold boundaries with targeted replacement/splitting of
   blocklisted structures that lie on ordinary guest dispatch/call/slot paths.
   This is the strongest preliminary direction, but it is **not selected** until
   GITHUB010 research/scoring and owner approval are complete.

4. **D — Broad native/continuation boundary.**
   Place broad boundaries around `enterNative`, resume, parameter binding or
   similar gateways.
   Preliminary concern: hides guest computation and callback composition from
   PE; likely incompatible with the optimization objective.

5. **E — Disable Truffle runtime compilation in Native Image.**
   This conflicts with ratified PLAT038 and is therefore not an admissible
   PLAT039 candidate unless PLAT038 is explicitly reopened.

## Next research gate

The next PLAT039 pass must complete the GITHUB010 exhaustive decision packet
before implementation boundaries are changed:

```text
TRUFFLE_RUNTIME_COMPILER_IN_NATIVE=REQUIRED
EXHAUSTIVE_TRUFFLE_SURVEY=PENDING
APPLE_PKL_SURVEY=PENDING
OUTSIDE_TRUFFLE_SURVEY=PENDING
COMPLETE_GATEWAY_INVENTORY=PENDING
CANDIDATE_SCORING=PENDING
INVARIANT_DELTA_CHECK=PENDING
RECOMMENDED_CANDIDATE=PENDING
OWNER_APPROVAL=PENDING
IMPLEMENTATION_READY=NO
```

No production runtime-boundary implementation is authorized by this checkpoint.
