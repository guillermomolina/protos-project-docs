# TEST009 — post-O systematic compilerability method correction

Date: 2026-10-05

Owning work item: `TEST009 / guillermomolina/protos#795`

This record corrects the execution method after TEST009-O. It does not change
Protos code, language semantics, specification text, tests, or compiler options.

## Governing systematic procedure

TEST009 had already established the systematic Truffle compilerability procedure:

~~~text
OLD_DYNAMIC_METHOD:
  try boundaries on many methods
  compile/measure
  retain winners

NEW_DYNAMIC_METHOD:
  run strict corpus compilation gate
  fail closed on compiler failure/warnings
  diagnose the exact failing expansion
  classify the remedy
  implement one causally justified repair
~~~

The governing methodological invariant is:

~~~text
DIAGNOSTIC_FAILURE != AUTOMATIC_BOUNDARY_PRESCRIPTION
~~~

A diagnostic identifies the failed compilerability property. The repair class
must then follow from the cause:

~~~text
cold/generic host work expanded into PE
  -> candidate narrow boundary

PE-relevant unresolved polymorphism
  -> specialization/cache/guard/library design

valid but excessively large PE graph
  -> inspect inlining structure / structural cut

required structural operand not PE-constant
  -> cached/constant invariant

unstable state repeatedly invalidates code
  -> state-lifetime investigation

unknown graph owner
  -> expansion diagnostics before source mutation
~~~

## Where TEST009 drifted

TEST009-M M5 addressed unresolved native-body dispatch with an exact-body
specialization PIC:

~~~text
native body identity
  -> nativeDirect specialization
  -> generic nativeCall fallback
~~~

That remedy was global for monomorphic native bodies. The post-M diagnostic
showed:

~~~text
TOO_DEEP_INLINING=1 -> 15
CODE_INSTALLATION_TOO_LARGE~=23 -> 21
COMPILER_OOM=1 -> 0
PERFORMANCE_WARNINGS~=19988 -> 4877
~~~

TEST009-N then selected a falsifiable hypothesis: M5 could remain global if
three shared host seams were cut narrowly.

TEST009-O implemented those cuts and produced:

~~~text
TOO_DEEP_INLINING=15 -> 8
CODE_INSTALLATION_TOO_LARGE=21 -> 30
COMPILATION_EXCEEDED_100_SECONDS=0 -> 2
PERFORMANCE_WARNINGS=4877 -> 5758
DIAGNOSTIC_WALL_SECONDS=718.3 -> 1500.6
~~~

The selected O seams did remove or suppress the observed C-prime plan-construction
and physical NIO-close expansion paths, but additional host/runtime graphs
became dominant, including actor I/O lifecycle Set/Map bookkeeping and logical
test discovery through host filesystem traversal.

Therefore the sequence:

~~~text
observe host expansion A/B/C
  -> add boundaries A/B/C
  -> rerun diagnostic
  -> observe different host expansion D/E/F plus code-size/timeouts
~~~

is the rejected old dynamic method in another form. Continuing that loop is not
a systematic compilerability strategy and has no demonstrated convergence
property.

## Corrected interpretation

The failure is no longer treated as a collection of missing host boundaries.

The architectural question is:

~~~text
What structural authority determines which native-body code is allowed
to become part of the guest partial-evaluation graph?
~~~

M5 changed that frontier globally. The next work must classify that mechanism,
not patch the latest traces one by one.

The burden of proof is now:

~~~text
GLOBAL_NATIVE_BODY_PIC_AS_DEFAULT_POLICY=REJECTED_UNLESS_STRUCTURALLY_JUSTIFIED
BOUNDARY_CHASING=STOP
~~~

This is not a decision to revert every narrow O boundary. O's boundaries are
semantically neutral implementation separations and some targeted paths did
disappear. What is rejected is using successive host-leaf boundaries as the
authority that makes global M5 safe.

## Systematic next investigation

The next slice must inspect the current architecture of:

~~~text
NativeCall
ProtosNativeClosureBody
ProtosSuspensionCapableNativeClosureBody
all native-body construction sites/families
all native-body execution entry points
existing TruffleBoundary and host/runtime service boundaries
existing structured native body classes
existing runtime/guest ownership distinctions
~~~

It must answer whether Protos already has, or can derive without ad-hoc
whitelisting, a compact structural distinction equivalent to:

~~~text
guest execution machinery intended for PE
vs
host/runtime service work not intended to expand into the guest graph
~~~

Acceptable outcomes are only:

~~~text
A)
ARCHITECTURAL_PE_ELIGIBILITY_RULE=EXISTS
-> nativeDirect may be restricted using that independently justified rule

B)
ARCHITECTURAL_PE_ELIGIBILITY_RULE=DOES_NOT_EXIST_OR_IS_NOT_MAINTAINABLE
-> M5 should be reverted, partially or fully as source evidence requires
~~~

The following is not an acceptable outcome:

~~~text
C)
add another batch of boundaries for the latest diagnostic traces
~~~

A marker, annotation, whitelist, class ladder, switch by name, or manually
maintained list is not considered architectural eligibility merely because it
can make the current diagnostic greener. Any eligibility mechanism must express
a pre-existing or independently justified runtime property and remain stable
when new native bodies are added.

## Execution policy for the next slice

~~~text
NEXT_SLICE=TEST009-P
NEXT_SLICE_TYPE=INVESTIGATION
IMPLEMENTATION_AUTHORIZED=NO

LOCAL_COMMANDS=0
BUILDS=0
TESTS=0
BENCHMARKS=0
TRUFFLE_DIAGNOSTICS=0
SOURCE_MUTATIONS=0
COMMITS=0
PUSHES=0

ANOTHER_BOUNDARY_BATCH=FORBIDDEN
CODE_TOO_LARGE_MICRO_REPAIR_BATCH=FORBIDDEN
~~~

The investigation may read the current HEAD of `guillermomolina/protos`,
current durable TEST009 evidence in `guillermomolina/protos-project-docs`,
the live TEST009 Issue, and public Truffle/Graal sources when needed.

No new formal Issue is required. TEST009 remains open.

## Evidence chain

~~~text
SYSTEMATIC_PROCEDURE_RECORD=
docs/project/evidence/TEST009/TEST009_SYSTEMATIC_TRUFFLE_COMPILERABILITY_PROCEDURE.md

TEST009_N_RECORD_REVISION=6e721a0d99dbc46f41907765d0c11436a7a0cea2
TEST009_N_RECORD=
docs/project/evidence/TEST009/TEST009_N_POST_M_TOO_DEEP_AND_M5_POLICY.md

TEST009_O_RECORD_REVISION=772317410a3db37c51cbbbae62fb97e6c57d064e
TEST009_O_RECORD=
docs/project/evidence/TEST009/TEST009_O_SHARED_HOST_LEAF_BOUNDARIES_AND_COMPILERABILITY_RESULT.md

TEST009_O_PRODUCT_REVISION=53c54bb952354cae61e72db68c4ff8f509c827ed
~~~

AI assistance: this durable record was drafted with ChatGPT from the published
TEST009 systematic procedure, the exact TEST009-M/O product and diagnostic
evidence, and the post-O maintainer correction that the boundary-by-boundary
sequence is not converging.
