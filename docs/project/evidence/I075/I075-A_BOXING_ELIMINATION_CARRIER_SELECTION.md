# I075-A — Bytecode DSL boxing-elimination carrier selection

Status: **COMPLETE**

This durable, non-normative record retains the investigation result for
I075-A / guillermomolina/protos#746.

## Identity

```text
DATE=2026-09-30
WORK_ITEM=I075-A
PARENT_WORK_ITEM=I075
GITHUB_ISSUE=guillermomolina/protos#746
PARENT=PERF011 / guillermomolina/protos#693

TYPE=INVESTIGATION_ONLY

PROTOS_REVISION=a89a8897ea20b10344785ee9573ed329d188ef28
PROTOS_VERSION=0.3.124-SNAPSHOT
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
```

The current Protos HEAD at investigation time was still exactly the published
I074 revision above.

No commands, builds, tests, benchmarks, profilers, repository mutations, patches,
commits or pushes were performed in `guillermomolina/protos` during I075-A.

## Exact result

The investigation did **not** find an existing guest-semantic primitive carrier
for Float, Integer or Boolean.

The clean existing primitive carrier is instead implementation-internal
`int` metadata already passed through Bytecode DSL operations:

- positional argument indices;
- positional parameter counts / upper bounds;
- captured lexical depths;
- frame-backed ordinals.

These values are implementation machinery, not Protos guest values, and already
exist independently of boxing elimination.

```text
FIRST_PRIMITIVE_CARRIER=OTHER
BOXING_ELIMINATION_TYPES_INITIAL_SET={int.class}
CURRENT_PRIMITIVE_FLOW_ALREADY_SUFFICIENT=YES
NEW_PRIMITIVE_REPRESENTATION_REQUIRED=NO

SEMANTIC_RISK=LOW
ARCHITECTURE_DECISION_REQUIRED=NO
IMPLEMENTATION_READY=YES
RECOMMENDED_NEXT_SLICE=I075-B
```

## Float / double finding

Current numeric literals are materialized before entering the Bytecode DSL
operand flow.

`CanonicalToBytecodeLowerer` emits:

```text
emitLoadConstant(materialize(literal))
```

and `ProtosNumberLiteral.materialize` constructs Float literals directly as:

```text
new ProtosFloatValue(Double.parseDouble(...))
```

Standard Float arithmetic is implemented by native closures that:

1. require `ProtosFloatValue` receiver and argument;
2. unwrap their internal doubles;
3. perform the Java double operation;
4. immediately construct a new `ProtosFloatValue`.

Therefore the current flow is:

```text
ProtosFloatValue
  -> local Java double arithmetic
  -> ProtosFloatValue
```

not:

```text
double
  -> Bytecode DSL stack/locals/operations
  -> double
```

A `double` carrier would require a new representation/materialization policy
across lookup/send, locals, identity, interop, reflection/tooling and
continuation boundaries. That is not a bounded I075 implementation.

The current Float identity/equality rules further make such a redesign
semantically sensitive: raw-bit identity distinguishes +0.0 from -0.0 while
NaN identity/equality behavior follows the existing Protos rules.

```text
DOUBLE_CURRENT_PRIMITIVE_FLOW=NONE
DOUBLE_BOXING_ELIMINATION_APPLICABLE=NO
DOUBLE_ARCHITECTURAL_COMPLEXITY=HIGH
DOUBLE_SEMANTIC_RISK=HIGH
```

## Integer / long finding

`ProtosIntegerValue` stores a `BigInteger`, literals are created directly as
`BigInteger`, and current Integer arithmetic remains in that representation.

There is no existing guest Integer `long` carrier.

Introducing:

```text
long fast carrier
+ transparent BigInteger promotion on overflow
```

would require a new numeric-representation architecture covering mixed
representations, arithmetic, comparison, identity/hash, dispatch, locals,
interop and materialization. It must not be introduced merely to enable a
Bytecode DSL feature.

```text
LONG_CURRENT_PRIMITIVE_FLOW=NONE
LONG_BOXING_ELIMINATION_APPLICABLE=NO
LONG_REQUIRES_NEW_NUMERIC_REPRESENTATION=YES
LONG_ARCHITECTURAL_COMPLEXITY=HIGH
LONG_SEMANTIC_RISK=HIGH
```

## Boolean finding

Guest Booleans remain the canonical semantic objects:

```text
ProtosBooleanValue.TRUE
ProtosBooleanValue.FALSE
```

The Bytecode DSL backend also contains many Java `boolean` operations, but
these are internal control predicates for structured dispatch, iteration,
continuation state, I/O sequencing and similar implementation machinery.

Thus an internal boolean primitive flow exists, but it is not a primitive
representation of guest Boolean values.

Upstream explicitly warns that boolean boxing elimination can incur additional
instruction cost and is not automatically profitable. Therefore
`boolean.class` is not included in the initial Protos set without independent
evidence.

```text
BOOLEAN_CURRENT_PRIMITIVE_FLOW=EXISTS_INTERNAL_ONLY
BOOLEAN_GUEST_PRIMITIVE_FLOW=NONE
BOOLEAN_BOXING_ELIMINATION_APPLICABLE=PARTIAL
BOOLEAN_INITIAL_ADOPTION=NO
```

## Selected int carrier

The current lowering already emits boxed constants that feed operation
parameters declared as primitive `int`, including:

```text
positionalIndex
positionalParametersBeforeRest
maximumPositionalArguments
lexicalDepth
frameOrdinal
```

Representative existing lowering patterns are:

```text
emitLoadConstant(positionalIndex)
emitLoadConstant(captured.lexicalDepth())
emitLoadConstant(frameBackedOrdinal(...))
```

and the receiving operations declare primitive `int` operands.

These values have no guest identity, delegation, reflection or interop
semantics. Enabling `int.class` therefore exposes an already legitimate
internal primitive flow to the Bytecode DSL without inventing any new Protos
representation.

```text
INT_CURRENT_PRIMITIVE_FLOW=PARTIAL
INT_BOXING_ELIMINATION_APPLICABLE=YES
INT_GUEST_SEMANTIC_OBJECT_REQUIRED_AT=NOT_APPLICABLE
INT_NEW_REPRESENTATION_REQUIRED=NO
INT_ARCHITECTURAL_COMPLEXITY=LOW
INT_SEMANTIC_RISK=LOW
```

The expected value is intentionally narrow: reduce cached-interpreter
boxing/unboxing and generic Object operand handling for existing internal int
metadata. No guest numeric-performance claim is made.

## Upstream authority

Current Graal/Truffle Bytecode DSL authority establishes:

- `boxingEliminationTypes` is best-effort primitive load/store specialization
  in the cached interpreter;
- supported primitive classes are `boolean`, `byte`, `int`, `float`,
  `long` and `double`;
- LoadConstant, LoadArgument, built-in local loads/stores and materialized local
  loads/stores have boxing-elimination specialization paths;
- uncached interpreter mode is compatible but is not itself where boxing
  elimination delivers its cached-interpreter benefit;
- yield/return boundaries do not retain primitive boxing elimination;
- built-in local operations integrate automatically with boxing elimination,
  while custom accessor paths do not become primitive merely because the
  facility is enabled.

SimpleLanguage remains an example/reference implementation using:

```text
{long.class, boolean.class}
```

That configuration is not copied to Protos.

Current GraalPy provides a production-language precedent for an existing
internal primitive carrier with:

```text
boxingEliminationTypes = {int.class}
```

The precedent supports the technique, not the value estimate for Protos.

## Preserved semantic invariants

I075-A requires no relaxation of any Protos invariant:

```text
Everything remains an object at guest semantic level.
Boolean remains canonical TRUE/FALSE objects.
No truthiness is introduced.
Integer remains unbounded.
No SmallInteger/BigInteger guest category is introduced.
Float semantics remain unchanged.
Lookup/delegation remain unchanged.
Identity/equality remain unchanged.
Reflection and interop remain unchanged.
Execution-context and lexical semantics remain unchanged.
Captured/materialized local semantics remain unchanged.
```

## I075-B implementation boundary

The selected implementation scope is deliberately minimal:

```text
ProtosBytecodeRootNode:
  add boxingEliminationTypes = {int.class}

Reuse only existing internal int flows.
Do not add double.class.
Do not add long.class.
Do not add boolean.class.
Do not introduce primitive guest-value representations.
Do not redesign numeric representation.
```

I075-B should validate generated/compiled Bytecode DSL acceptance, semantic
equivalence and the integrated validation required by the repository's current
implementation policy.

The PERF011 lightweight policy remains authoritative:

```text
PERF020_REQUIRED_FIRST=NO
PROTOS_BENCHMARKS_REQUIRED_FIRST=NO
```

No bespoke benchmark harness is required and no performance claim should be made
from noisy single-run suite timing.

## Routing

```text
I075-A=COMPLETE
I075/#746=OPEN
NEXT_SLICE=I075-B
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos

FIRST_PRIMITIVE_CARRIER=OTHER
BOXING_ELIMINATION_TYPES_INITIAL_SET={int.class}
IMPLEMENTATION_READY=YES
ARCHITECTURE_DECISION_REQUIRED=NO
```
