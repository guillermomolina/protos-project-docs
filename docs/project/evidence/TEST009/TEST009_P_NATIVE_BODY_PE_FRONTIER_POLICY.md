# TEST009-P — native-body partial-evaluation frontier policy

Date: 2026-10-06

Owning work item: `TEST009 / guillermomolina/protos#795`

Investigation type: **read-only architecture / compilerability policy investigation**.

## Exact analyzed state

The investigation itself analyzed the then-current product revision:

~~~text
ANALYZED_PROTOS_HEAD=53c54bb952354cae61e72db68c4ff8f509c827ed
ANALYZED_PROTOS_VERSION=0.3.221-SNAPSHOT
COMMIT=TEST009-O: bound shared host leaves exposed by M5 native-body PIC

ANALYZED_PROJECT_DOCS_HEAD=efe398522afef71f26e11e729b0672b1f671903f
~~~

Before durable publication, `guillermomolina/protos` advanced by one concurrent
I070 commit:

~~~text
CURRENT_PROTOS_HEAD_AT_PUBLICATION=7f398f623a210b2bc3837d37dd3e5980dd726e75
CURRENT_PROTOS_VERSION_AT_PUBLICATION=0.3.222-SNAPSHOT
CONCURRENT_COMMIT=I070-A: eliminate handwritten main compilation warnings
~~~

That delta changes handwritten Java/Truffle warning cleanup, canonical `@Bind`
forms, DSL metadata and serialization warnings. It does not remove, restrict or
otherwise change the TEST009-M exact-native-body PIC: both
`ProtosBytecodeRootNode.EnterClosureCall` and
`ProtosSemanticBytecodeRootNode.EnterClosureCall` still contain the
`nativeDirect` exact-body specialization plus the generic `nativeCall`
fallback. Therefore the P architectural conclusion remains applicable to the
current product HEAD.

## Governing systematic procedure

P reapplies the previously published compilerability rule:

~~~text
DIAGNOSTIC_FAILURE != AUTOMATIC_BOUNDARY_PRESCRIPTION
~~~

The M5 problem is not treated as a request to find another set of
`@TruffleBoundary` methods. The question is whether Protos has a structural,
diagnostic-independent authority that determines which native bodies are allowed
to become part of the guest partial-evaluation graph.

## Current native-body architecture

The current representation does not encode such an authority.

`ProtosNativeClosureBody` is only a functional interface:

~~~text
Object execute(ProtosActivation activation, List<?> supplied)
~~~

The same representation is used across heterogeneous families including
primitive/value operations, collections, standard Object/Error/control helpers,
encoding, byte/text I/O, filesystem, network/TCP, process services,
Actor/Future/parallel runtime services, import/module facilities, CLI and Test
Tool facilities.

`ProtosClosureValue.nativeClosure` accepts that representation generically,
stores it, and preserves it across binding/projection. A prepared native
invocation becomes `NativeCall`; M5 then caches the exact body identity at
`EnterClosureCall.nativeDirect`.

The only important existing nominal subcategory is:

~~~text
ProtosSuspensionCapableNativeClosureBody
~~~

That interface represents PLAT019 suspension capability. It is the authority for
ordinary versus continuation-capable native entry; it does **not** represent PE
eligibility and cannot be reused as such without changing its architectural
meaning.

Likewise, `StructuredCallCapabilities` and the nominal/identity categories for
specific standard operations represent already-established structured guest
dispatch semantics. They do not partition the general native-body population
into PE-friendly versus host/runtime-service bodies.

## Eligibility result

No current type, interface, execution-plan kind, node kind, call target,
protocol family, context ownership rule or other existing invariant means:

~~~text
this ordinary native body is intended to be visible to guest partial evaluation
~~~

A new marker or factory introduced solely now would have to be assigned
body-by-body to lambdas, method references or nominal bodies according to a
human decision about whether each implementation should be PE-visible.

That would make the effective rule:

~~~text
mark this body because current compiler diagnostics say it is safe/useful
~~~

which is diagnostic-derived manual tagging, not structural eligibility.

Therefore:

~~~text
ARCHITECTURAL_PE_ELIGIBILITY_RULE=
  DOES_NOT_EXIST_OR_IS_NOT_MAINTAINABLE

ELIGIBILITY_IS_INDEPENDENT_OF_CURRENT_DIAGNOSTIC=NO
NEW_NATIVE_BODY_POLICY_IS_BY_CONSTRUCTION=NO
WHITELIST_OR_MANUAL_TAGGING_REQUIRED=YES
~~~

## M5 decision

The global exact-native-body PIC is not structurally justified.

~~~text
GLOBAL_NATIVE_BODY_PIC_AS_DEFAULT_POLICY=
  NOT_STRUCTURALLY_JUSTIFIED

RECOMMENDED_M5_ACTION=FULL_REVERT
~~~

The minimum causal revert is the M5 exact-body specialization mechanism itself:

- remove `nativeDirect` from the Bytecode `EnterClosureCall`;
- remove the symmetric `nativeDirect` specialization from Semantic Bytecode;
- restore one generic native-call entry in both;
- remove M5-only exact-body access/helper machinery only when it becomes unused;
- retain the single ordinary-versus-suspension-capable authority in
  `NativeCall`;
- retain M1-M4;
- do not introduce an eligibility marker, whitelist, name switch or growing
  `instanceof` ladder.

This is a compilerability-frontier change only. It changes no observable Protos
language semantics and requires no specification change.

## Convergence property

~~~text
CONVERGENCE_PROPERTY=YES
~~~

The property belongs to the recommended revert policy: new ordinary native bodies
again inherit the same generic native-dispatch frontier automatically. Adding a
new filesystem, network, test-tool or runtime-service body does not make its
exact implementation PE-constant merely because a call site is monomorphic.

This removes the M5-specific cycle:

~~~text
new native body
 -> diagnostic exposes new host graph
 -> add another boundary
~~~

It does not claim all remaining TEST009 compilerability debt disappears.

## TEST009-O boundaries

P classifies the three O cuts independently from M5:

~~~text
O_ENCODING_BOUNDARY=INDEPENDENTLY_CORRECT_BOUNDARY
O_CPRIME_PLAN_BOUNDARY=INDEPENDENTLY_CORRECT_BOUNDARY
O_PHYSICAL_CLOSE_BOUNDARY=INDEPENDENTLY_CORRECT_BOUNDARY
~~~

Each isolates host/runtime work while leaving its Protos semantic authority on
the visible side of the cut, and remains architecturally meaningful even if M5
is removed.

## Remaining O diagnostic observations

The existing evidence does not prove exact causality for the two new
>100-second compilations, so P does not overclaim:

~~~text
POST_O_COMPILATION_TIMEOUTS=POSSIBLY_RELATED
~~~

The code-installation-too-large increase is compatible with the broader frontier
expansion but also belongs to pre-existing compilerability debt:

~~~text
POST_O_CODE_TOO_LARGE_REGRESSION=PARTIALLY_RELATED
~~~

No micro-repair is selected for either family in P.

## Upstream consistency

The previously pinned Truffle/Graal sources support the architectural direction:

- Truffle PE specializes code reachable from compiled roots when stable
  interpreter structure becomes PE-constant.
- GraalJS and GraalPython represent builtins through Truffle node categories
  rather than arbitrary generic functional-interface bodies.
- TruffleRuby likewise expresses core execution through dedicated node
  categories and places host/runtime work behind explicit boundaries where
  appropriate.
- Truffle documentation treats runtime code unsuitable for PE as an explicit
  architectural boundary decision.

This corroborates, but does not independently define, the Protos result: PE
participation should follow an execution structure, not a diagnostics-derived
exception list.

## Result

~~~text
TEST009_P_INVESTIGATION=COMPLETE

SYSTEMATIC_PROCEDURE_REAPPLIED=YES

GLOBAL_NATIVE_BODY_PIC_AS_DEFAULT_POLICY=
  NOT_STRUCTURALLY_JUSTIFIED

ARCHITECTURAL_PE_ELIGIBILITY_RULE=
  DOES_NOT_EXIST_OR_IS_NOT_MAINTAINABLE

CONVERGENCE_PROPERTY=YES
RECOMMENDED_M5_ACTION=FULL_REVERT

BYTECODE_SEMANTIC_BYTECODE_SYMMETRY=REQUIRED
ORDINARY_VS_SUSPENSION_AUTHORITY=KEEP_SINGLE_OWNER

BOUNDARY_CHASING_ALLOWED=NO
CODE_TOO_LARGE_MICRO_REPAIR_ALLOWED=NO
EXPENSIVE_DIAGNOSTICS_DURING_P=0

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO

TEST009_COMPLETE=NO

NEXT_SLICE=TEST009-Q
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_IMPLEMENTATION_REPOSITORY=guillermomolina/protos
NEXT_SCOPE=MINIMUM_CAUSAL_FULL_REVERT_OF_TEST009_M_M5_NATIVE_BODY_PIC
~~~

No new formal Issue is required for Q.

AI assistance: this durable record was drafted with ChatGPT from the exact
TEST009-M/N/O and post-O evidence chain, the current Protos native-body
architecture, the pinned upstream Truffle-language references already recorded
by TEST009, and a publication-time reconciliation against current Protos HEAD.
