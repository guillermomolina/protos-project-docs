# PLAT054-2 — Publication of standard Polyglot embedding normative contract

**Evidence type:** immutable specification-publication snapshot.  
**Status:** PRODUCT NORMATIVE CONTRACT PUBLISHED; runtime implementation pending.  
**Date:** 2026-10-08.

## Exact publication identifiers

- **Product repository:** `guillermomolina/protos`
- **Product revision:** [`ab1f2196e6736d4df01e83f685a3fc8aa3f606ac`](https://github.com/guillermomolina/protos/commit/ab1f2196e6736d4df01e83f685a3fc8aa3f606ac)
- **Commit subject:** `PLAT054-2: publish standard Polyglot embedding normative contract`
- **Global normative specification revision:** `0.1.449` dated 2026-10-08.
- **Decision issue:** [PLAT054 / #838](https://github.com/guillermomolina/protos/issues/838)
- **Performance parent:** [PERF032 / #831](https://github.com/guillermomolina/protos/issues/831)
- **Platform decision:** [PLAT054](../../decisions/platform/PLAT054_STANDARD_POLYGLOT_EMBEDDING_CONTRACT.md)
- **Owner approval / GITHUB010 evidence:** [PLAT054-1](PLAT054_1_OWNER_APPROVAL_NORMATIVE_RECONCILIATION.md).

## Verified source publication

The published GitHub commit contains **exactly four files**, with a direct specification record of the approved effects:

1. [`spec/semantics/MODULES.md`](https://github.com/guillermomolina/protos/blob/ab1f2196e6736d4df01e83f685a3fc8aa3f606ac/spec/semantics/MODULES.md) — new **Host-initiated evaluations in an embedded RootActor** subsection. First entry is RootActor initial module; later entries run in the same live Process without REPL semantics. Source identity/name is not a ModuleKey. Canonical READY module entries reuse instance and retained terminal result; unkeyed entries create fresh standalone moduleContext. Stable read-only host bindings select only local slots of the last normally completed module. Closure member read uses fresh receiver-bound extraction, retained value identity remains fixed, and fatal Process termination revokes live guest entry.
2. [`spec/io/PROCESS_IO.md`](https://github.com/guillermomolina/protos/blob/ab1f2196e6736d4df01e83f685a3fc8aa3f606ac/spec/io/PROCESS_IO.md) — new **Standard Polyglot embedding bootstrap and authority** subsection. Lazy one Process/Context; Context and pre-evaluation binding queries do not initialize Process/RootActor/Core; only initial module receives bootstrap-local slots, with filesystem/network conditioned on host-granted authority. Streams, arguments and environment derive from Truffle Env; Core resource loading grants no filesystem privilege. Same-RootActor unsafe concurrency fails closed, fatal Process does not restart, disposal respects termination and provider-resource custody.
3. [`spec/concurrency/ACTORS.md`](https://github.com/guillermomolina/protos/blob/ab1f2196e6736d4df01e83f685a3fc8aa3f606ac/spec/concurrency/ACTORS.md) — §24C clarification that synchronous host-initiated ordinary RootActor execution is an Actor turn for escaping Error fatality **without mandatory physical RootTask or ProtosTask**.
4. [`spec/PROTOS_SPEC_CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/ab1f2196e6736d4df01e83f685a3fc8aa3f606ac/spec/PROTOS_SPEC_CHANGELOG.md) — new `0.1.449` entry for the three normative owners and preserved invariants.

The revision does not modify Java source, tests, `pom.xml`, root `CHANGELOG.md`, or the benchmark repository. No runtime behavior or benchmark parity is asserted as already implemented.

## Validation provenance

The project owner reported after pushing the revision:

> PLAT054-2: publish standard Polyglot embedding normative contract, pushed.

> el git diff check esta limpio. Todos los tests han pasado en local

Accordingly the **reported** local human-executor state is:

```text
HUMAN_REPORTED_LOCAL_TESTS=PASS
HUMAN_REPORTED_GIT_DIFF_CHECK=CLEAN
AGENT_EXECUTED_TESTS=NO
AGENT_EXECUTED_BUILDS=NO
AGENT_EXECUTED_GIT_OPERATIONS_IN_PROTOS=NO
COMMIT_AND_NORMATIVE_REVISION=VERIFIED_ON_GITHUB
PRODUCT_RUNTIME_IMPL=NOT_YET
BENCHMARK_GRAPH_OR_TIMING_PARITY=NOT_MEASURED
```

No actual test log, precise test count or CI-run ID was supplied; no stronger validation claim is made.

## GITHUB021 invariant consistency and release of next work

The 0.1.449 change has been compared with the seven PLAT054-1 approved clarifications and the eight owner-approved design constraints; the four changed paths introduce the approved host entry semantics without an alternative global namespace, extra ModuleKey, Process-per-eval, or unconditional Task/scheduler allocation. Retained existing module lifecycle, Actor isolation, Process I/O capability semantics, Closure extraction and terminal error classification remain the controlling contracts.

The **normative publication gate** recorded by PLAT054/#838 has been discharged. The next action is grouped product implementation, **PLAT054-3**, in **`guillermomolina/protos`**: standard `Context.eval`/language bindings, lazy Process bootstrap, Core discovery/override, restricted Env authorities, canonical/standalone module lifecycle, Closure interop, Process termination and regression tests. Preserve human-executor execution boundaries.

Do not conflate completion of this specification publication with completion of Java embedding or PERF032 Graal graph parity. The PLAT054 work item may continue tracking the runtime integration until implementation conformance is documented; its platform decision is already ratified.

## Cross-revision verification

```text
PROTOS_REVISION=ab1f2196e6736d4df01e83f685a3fc8aa3f606ac
SPECIFICATION_REVISION=0.1.449
SPEC_FILES_CHANGED=4
OWNER_APPROVAL=EXPLICIT
DOCUMENTATION_RECORD=THIS_FILE
RUNTIME_IMPLEMENTATION=PENDING
FOLLOW_ON_SLICE=PLAT054-3
FOLLOW_ON_REPOSITORY=guillermomolina/protos
```

## AI assistance

ChatGPT prepared this evidence record and inspected the published commit through GitHub. The normative edits, local tests, validation and source-repository publication were performed by the human executor, as reported by the project owner; no separate review of the test logs is claimed.
