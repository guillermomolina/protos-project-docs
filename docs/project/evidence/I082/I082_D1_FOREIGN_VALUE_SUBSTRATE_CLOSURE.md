# I082-D1 — D188 foreign-value admission/projection/error substrate closure

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=I082
SLICE=I082-D1
PROTOS_ISSUE=guillermomolina/protos#830
PARENT_WORK=AUD019/guillermomolina/protos#818
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~

This record is durable non-normative project evidence.

## Published product revision

~~~text
STARTING_PROTOS_REVISION=c21ff278e6dca1219429cc7f08568c54af2bd951
STARTING_VERSION=0.3.274-SNAPSHOT

ENDING_PROTOS_REVISION=32f61e4781270e0e5aca29aa1809a53a17f5bb02
ENDING_VERSION=0.3.275-SNAPSHOT
SPECIFICATION_REVISION=0.1.446

COMMIT_SUBJECT=I082-D1: add D188 foreign-value admission/projection/error substrate
~~~

No normative specification file changes.

## Implemented D188 substrate

I082-D1 implements the shared provider-neutral foreign-value substrate required by
D188, except ordinary foreign iteration through `each`, which is explicitly
deferred to I082-D2.

Published implementation includes:

~~~text
SOURCE_CLASSIFIED_FOREIGN_ADMISSION=YES
BOOLEAN_ADMISSION=YES
TRUE_NULL_ADMISSION=YES
VALID_UNICODE_TEXT_ADMISSION=YES
UNBOUNDED_INTEGER_ADMISSION=YES
EXACT_BINARY64_ADMISSION=YES
AMBIGUOUS_OR_LOSSY_VALUES_REMAIN_RAW=YES

RAW_FOREIGN_REFERENCE=YES
RAW_REFERENCE_BOUND_TO_EXACT_SESSION_GENERATION=YES
CLOSED_SESSION_REBIND=NO

PROVIDER_STABLE_FOREIGN_IDENTITY=YES
NO_STABLE_IDENTITY_INDEPENDENT_ADMISSIONS_DISTINCT=YES
RAW_EQUALITY_DEFAULTS_TO_SEMANTIC_IDENTITY=YES
RAW_HASH_USES_SEMANTIC_IDENTITY_HASH=YES
FOREIGN_EQUALS_HASH_IMPORTED=NO

ORDINARY_PROTOS_PROJECTION_FIRST=YES
FAITHFUL_FOREIGN_MEMBER_FALLBACK=YES
RESERVED_INSTITUTION_MEMBER_HIJACKING=NO
FOREIGN_WRITE_REDIRECTION=NO

EXECUTABLE_FOREIGN_CALL_PROJECTION=YES
NON_EXECUTABLE_RAW_INHERITS_OBJECT_CALL=NO
GENERIC_FOREIGN_INSTANTIATION=NO

INDEXED_AT_PROJECTION=YES
INDEXED_ATPUT_PROJECTION=YES
AMBIGUOUS_INDEX_HASH_PROJECTION_REJECTED=YES

FOREIGN_ITERATION_EACH=DEFERRED_TO_I082_D2

FOREIGN_OPERATION_ENTERED_BOUNDARY=YES
PRE_ENTRY_FAILURE_IS_ORDINARY_ERROR=YES
ENTERED_FAILURE_IS_FRESH_FOREIGNERROR=YES
SAFE_FOREIGNERROR_PAYLOAD=YES
CAUSE_PROJECTION_CYCLE_SAFE=YES

ACTOR_TRANSFER_RAW_FOREIGN=NON_TRANSFERABLE
P_TRANSFER_RAW_FOREIGN=NON_PARALLEL
FOREIGN_MODULE_FACADE_USES_SHARED_D188_SUBSTRATE=YES
~~~

## ForeignError reconciliation

The already-normative `ForeignError -> Error` category is now present in the
implementation surfaces that own the closed Core taxonomy:

~~~text
protos/lib/core/error_taxonomy.protos
protos/lib/core/prelude.protos
ProtosCoreErrorTaxonomy
ProtosCoreErrors.StandardError
required Core binding tests
reflection parent tests
test-tool standard-error mapping
~~~

Each entered foreign failure produces a fresh occurrence with exactly the six
safe public payload slots required by D188:

~~~text
language
operation
category
foreignCategory
message
cause
~~~

No host Throwable, PolyglotException, runtime handle, foreign exception object,
host/native stack, session object or provider implementation metadata is made
guest-visible.

## Architecture boundaries preserved

I082-D1 does not implement or grant:

~~~text
D189_CALLBACK_BRIDGE=NO
FOREIGN_ITERATION_EACH=NO
PLAT052_FULL_AUTHORITY_ENFORCEMENT=NO
CONCRETE_PRODUCTION_PROVIDER=NO
STD_INTEROP_PUBLIC_API=NO
HOSTACCESS_BROADENING=NO
POLYGLOTACCESS_BROADENING=NO
IO_NATIVE_PROCESS_NETWORK_AUTHORITY=NO
~~~

The single D188 admission/identity/conversion/failure substrate is used by
import-based foreign values and is the intended substrate for later
`std:interop`, concrete providers and D189 callback arguments.

## Exact product change surface

The published commit changes the Core Error bindings/mappings, the Bytecode and
invocation lookup integration points, Actor/P transfer guards, semantic identity,
the foreign module facade/import acquisition path, and adds the provider-neutral
D188 substrate classes plus focused tests.

Notable new implementation owners include:

~~~text
ProtosForeignAdmissionDescriptor
ProtosForeignArgument
ProtosForeignFailureDescription
ProtosForeignHandle
ProtosForeignOperation
ProtosForeignProjectedOperations
ProtosForeignValueAdapter
ProtosForeignValueAdmission
ProtosForeignProjectedReceiver
ProtosRawForeignValue
~~~

Focused test infrastructure includes:

~~~text
ProtosForeignValueFixture
ProtosForeignValueAdmissionTest
ProtosForeignValueProjectionTest
~~~

## Validation provenance

The maintainer reported after publication:

> el git diff check esta limpio.
>
> Todos los tests han pasado en local

Recorded exactly as:

~~~text
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
LOCAL_TESTS=PASS_MAINTAINER_REPORTED
~~~

No unreported command, test count, duration or suite composition is inferred.

## Slice closure and next routing

The product changelog explicitly identifies the only remaining D188 generic
projection item as foreign pull iteration:

~~~text
Foreign iteration (each) is not projected yet; it follows in I082-D2.
~~~

Therefore D1 closes, while the larger I082-D closure remains pending D2.

~~~text
I082_D1_STATUS=COMPLETED
I082_D_STATUS=IN_PROGRESS
I082_STATUS=IN_PROGRESS

NEXT_SLICE=I082-D2
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
NEXT_SLICE_GOAL=D188_FOREIGN_PULL_ITERATION_EACH
~~~

I082-D2 remains separate from I082-E. D2 is Protos-controlled pull iteration and
never hands the Protos block to the foreign runtime. I082-E is the D189
foreign-to-Protos callback bridge with Actor/Task re-entry, expiry, thread
admission and suspension constraints; combining them would mix two opposite
callback ownership models in one publication.

## AI-assistance disclosure

This closure evidence was materially prepared with AI assistance from ChatGPT
using the exact published I082-D1 product commit, current normative specification,
live GitHub state and maintainer-reported local validation.
