# I066-A — canonical numeric IP migration investigation

Date: 2026-10-05

## Work identity

~~~text
WORK_ITEM=I066
RESEARCH_SLICE=I066-A
PROTOS_ISSUE=guillermomolina/protos#669
DECISION_AUTHORITY=D172/guillermomolina/protos#644
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative investigation evidence. It does not replace
the normative Protos specification or the live GitHub Issue state.

## Exact investigated product state

~~~text
PROTOS_REVISION=5a3c5e4a274dee3b815746a21008c4cbedf4c1c1
PROTOS_COMMIT_SUBJECT=BUG016-B: shard Truffle diagnostics by logical Case
D172_SELECTED_CANDIDATE=C
INVESTIGATION_EXECUTION_MODE=READ_ONLY
COMMANDS_EXECUTED=NO
BUILDS_EXECUTED=NO
TESTS_EXECUTED=NO
PRODUCT_MUTATION=NO
~~~

The project owner approved the implementation shape derived by this investigation
on 2026-10-05.

## Investigation verdict

~~~text
I066_A_INVESTIGATION=COMPLETE
D172_CANDIDATE_C_IMPLEMENTABLE_FROM_CURRENT_HEAD=YES
EXISTING_GENERAL_STDLIB_RUNTIME_SEAM=NO
BOUNDED_GENERAL_SEAM_CAN_BE_ADDED_WITHOUT_NEW_DECISION=YES
IP_SPECIFIC_INFRASTRUCTURE_REQUIRED=NO
IMPORT_DURING_TRANSFER_REQUIRED=NO
D048_SEMANTICS_CAN_BE_PRESERVED=YES
ACTOR_MODULE_ISOLATION_CAN_BE_PRESERVED=YES
NEW_LANGUAGE_DECISION_REQUIRED=NO
NEW_PLATFORM_DECISION_REQUIRED=NO
IMPLEMENTATION_READY=YES
~~~

## Current architecture

At the investigated revision, Core bootstrap still creates the canonical
source-backed `IpAddress` and `IpEndpoint` objects from
`protos/lib/core/IpAddress.protos` and
`protos/lib/core/IpEndpoint.protos`. The existing
`ProtosStandardIpAddressProtocol` and `ProtosStandardIpEndpointProtocol`
install D048 construction, exact recognition, structural equality and hashing on
those exact objects, after which `protos/lib/core/prelude.protos` publishes both
families as public Prelude bindings.

The current Standard Library modules consume those Prelude names:

~~~text
std:network/IpAddresses
    -> IpAddress(...)
    -> IpAddress.recognizes(...)

std:network/IpEndpoints
    -> IpEndpoint(...)
    -> IpEndpoint.recognizes(...)
~~~

Network/TCP native consumers also recover the families through
`prelude.bindings().readLocalSlot("IpAddress")` and
`prelude.bindings().readLocalSlot("IpEndpoint")`.

## Existing substrate that makes D172 Candidate C viable

`ProtosPrelude` already retains standard canonical objects that are intentionally
not public Prelude bindings:

~~~text
runtimeBytesPrototype
runtimeActorRefPrototype
runtimeTcpConnectionPrototype
runtimeTcpListenerPrototype
~~~

`TcpConnection` and `TcpListener` are especially relevant: they prove that one
frozen standard protocol identity can remain runtime-owned, be absent from the
public Prelude, participate in transfer/runtime machinery and remain ordinary
observable Protos state through values that delegate to it.

The module runtime separately already proves that module contexts can receive
pre-established local slots before source execution through the RootActor initial
module bootstrap-local path. Ordinary imports do not currently have a general
equivalent seam.

## Required bounded general seam

No existing general Standard-Library/runtime seam currently maps a canonical
standard module identity to immutable initial module members. I066 therefore
needs one small general internal seam rather than IP-specific loader logic.

Recommended internal shape:

~~~text
ModuleKey -> immutable Map<memberName, canonicalStandardObject>
~~~

For I066 the registered data is:

~~~text
std:network/IpAddresses
    IpAddress -> runtimeCanonicalIpAddress

std:network/IpEndpoints
    IpEndpoint -> runtimeCanonicalIpEndpoint
~~~

When a fresh Actor-local module instance is created, those exact immutable
members are installed on its ordinary `moduleContext` before its source body
executes. The existing Actor-local module cache, `ModuleKey` identity,
cache-before-execute lifecycle and module instance ownership remain unchanged.

The mechanism must be applied coherently to every canonical module-instance
creation path used by the runtime, including the ordinary bytecode import path,
the direct canonical module runtime path and canonical initial-module execution.
It must not be encoded as IP-specific conditional logic inside the generic module
lifecycle.

## Canonical identity and Actor ownership

The required model is:

~~~text
one ProtosPrelude/runtime
    -> one frozen canonical IpAddress family
    -> one frozen canonical IpEndpoint family

Actor A
    -> Actor-local std:network/IpAddresses moduleContext
    -> Actor-local std:network/IpEndpoints moduleContext

Actor B
    -> distinct Actor-local std:network/IpAddresses moduleContext
    -> distinct Actor-local std:network/IpEndpoints moduleContext
~~~

The module instances remain Actor-local and distinct. Their canonical family
members may reference the same frozen runtime-owned standard objects because the
existing module/Prelude sharing rules already permit physically shared
semantically immutable standard state.

Therefore:

~~~text
IpAddresses@A !== IpAddresses@B

IpAddresses@A.IpAddress
    === IpAddresses@B.IpAddress
    === runtimeCanonicalIpAddress
~~~

does not create shared mutable module state.

## Transfer result

Actor and P transfer currently preserve IP family membership because the
canonical families are public Prelude values and therefore treated as shared
standard anchors. Removing those bindings without replacement would cause the
delegation parents to be copied and would break exact canonical-parent
recognition.

I066 must therefore extend the existing runtime-standard anchor test to the
private canonical IP families. Transfer remains ordinary value copying:

~~~text
source IP occurrence !== destination IP occurrence
semantic state is preserved
destination parent === canonical runtime family
~~~

No module import, source execution or module-cache mutation is required during
Actor or P transfer.

## Network/TCP result

The following current Prelude-binding dependencies must be changed to runtime-only
canonical-family access:

- `ProtosStandardNetworkProtocol`;
- `ProtosNioNetworkHost`;
- `ProtosStandardTcpConnectionProtocol`;
- `ProtosStandardTcpListenerProtocol`.

`ProtosNioNetworkBackend` already works substantially from explicitly supplied
prototype identities and can retain that architecture.

The D048 protocol bridges themselves require no semantic redesign.

## Public placement result

I066 implementation should remove:

~~~text
Prelude.IpAddress
Prelude.IpEndpoint
~~~

and expose the same canonical family identities through:

~~~text
std:network/IpAddresses.IpAddress
std:network/IpEndpoints.IpEndpoint
~~~

while retaining the existing helpers:

~~~text
IpAddresses.v4
IpAddresses.v6
IpAddresses.parse
IpAddresses.format

IpEndpoints.parse
IpEndpoints.format
~~~

The Standard Library source algorithms need no semantic rewrite; their bare
family lookups can resolve to their module's pre-provisioned canonical member.

## Preservation requirements

~~~text
D048 construction                       PRESERVE
fresh frozen IP values                  PRESERVE
exact canonical immediate parent        PRESERVE
exact-own-state recognition             PRESERVE
callback-free recognition               PRESERVE
structural equality/hash                PRESERVE
Map-key behavior                        PRESERVE
Actor transfer                          PRESERVE
P transfer                              PRESERVE
Network/TCP consumption                 PRESERVE
numeric IPv4/IPv6                       PRESERVE
numeric endpoints                       PRESERVE
no DNS                                  PRESERVE
no Network authority                    PRESERVE
no mapped-v6 normalization              PRESERVE
Actor-local module instances            PRESERVE
~~~

## Required focused evidence for implementation

I066-B must cover at least:

- Prelude absence of `IpAddress` and `IpEndpoint`;
- exact identity exposure through both `std:network` modules;
- direct numeric construction through those exposed canonical families;
- existing IPv4/IPv6 parse/format behavior;
- D048 recognition negative cases;
- equality/hash/Map-key behavior;
- fresh Actor-transfer identity with canonical family membership;
- fresh P-transfer identity with canonical family membership;
- proof that transfer performs no hidden module import/source execution;
- Network connect/listen validation;
- TCP local/remote endpoint recognition;
- NIO endpoint materialization;
- architecture guards against an IP-specific loader/cache/value family.

## Normative reconciliation

`spec/io/NETWORK.md` currently describes `IpAddress` and `IpEndpoint` as
standard frozen Prelude factory/prototypes. I066-B must move that public placement
to the approved `std:network` module members without weakening any D048
construction, recognition, equality/hash, transfer or authority rule.

`spec/concurrency/ACTORS.md`, `spec/concurrency/PARALLEL_EXECUTION.md`,
`spec/semantics/MODULES.md` and the informative
`spec/runtime/ABSTRACT_RUNTIME.md` already contain the ownership/isolation rules
needed by the proposed implementation and do not require a new language or
platform decision.

Historical specification changelog records must remain historical; D172 is a new
placement reconciliation, not a rewrite of the D048 record.

## Materially inspected product files

- `AGENTS.md`
- `AGENTS.work/IMPLEMENTATION.md`
- `protos/lib/core/IpAddress.protos`
- `protos/lib/core/IpEndpoint.protos`
- `protos/lib/core/prelude.protos`
- `protos/lib/network/IpAddresses.protos`
- `protos/lib/network/IpEndpoints.protos`
- `src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosModuleRuntime.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosCanonicalInitialModuleExecution.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardIpAddressProtocol.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardIpEndpointProtocol.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardNetworkProtocol.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosNioNetworkHost.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosNioNetworkBackend.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardTcpConnectionProtocol.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardTcpListenerProtocol.java`
- `spec/io/NETWORK.md`
- `spec/semantics/MODULES.md`
- `spec/concurrency/ACTORS.md`
- `spec/concurrency/PARALLEL_EXECUTION.md`
- `spec/runtime/ABSTRACT_RUNTIME.md`
- current IP construction, Map-key, Actor-transfer, P-transfer and
  Standard-Library network conformance tests.

## Research closure

~~~text
I066_A_RESEARCH=COMPLETE
OWNER_APPROVAL=YES
NEXT_SLICE=I066-B
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
ISSUE_669_MUST_REMAIN_OPEN=YES
ISSUE_669_STATUS=READY
I066_CLOSURE_AUTHORIZED=NO
~~~
