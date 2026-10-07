# D160 — Parallel collection algorithm placement

Status: **RATIFIED — Candidate B (Standard Library policy over retained minimal P)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#625`
Trigger: AUD009-C2 / `guillermomolina/protos#624`
Protos evidence revision: `5ce8e039a69489a49fe446d58de7fb39bcbb278f`
Project-record base: `a1ddaec12b9dc85b6c1e3fd659e1b0c273ef7f8b`

This is a durable non-normative decision record. Observable Protos semantics remain authoritative under `guillermomolina/protos:spec/**`.

## Decision

D160 selects **Candidate B**.

The five current privileged Core Array parallel algorithms move to ordinary Standard Library policy in `std:collections/Array`, implemented over the retained public P/Future kernel where that kernel is sufficient.

```text
Core Array.parallelMap                  REMOVE
Core Array.parallelFilter               REMOVE
Core Array.parallelFindIndex            REMOVE
Core Array.parallelReduce               REMOVE
Core Array.parallelSort                 REMOVE

Closure.parallel                        KEEP
Future / Future.all                     KEEP
isolated process-local P                KEEP
P projection / explicit transfer        KEEP
no public P/worker/executor/task object  KEEP ABSENT
Array.parallelEach                      KEEP ABSENT
writable Array/object partitions        KEEP ABSENT

std:collections/Array.map               KEEP
std:collections/Array.filter            KEEP
std:collections/Array.findIndex         KEEP
std:collections/Array.reduce            KEEP
std:collections/Array.sort              KEEP

std:collections/Array.parallelMap       ADD
std:collections/Array.parallelFilter    ADD
std:collections/Array.parallelFindIndex ADD
std:collections/Array.parallelReduce    ADD
std:collections/Array.parallelSort      ADD
```

The Standard Library owns algorithm policy. Core owns the minimal isolated-execution mechanism.

## Approval provenance and invariant consistency

The complete D160 packet presented to the project owner covered current repository evidence, comparative prior art, Candidates A through E, twelve-dimension scoring, library implementability, deterministic ordering/failure/cancellation tradeoffs, reduction/sort policy, performance risks, deferral/reintroduction cost, compatibility, recommendation, strongest argument against the recommendation, and intentionally deferred questions.

The project owner explicitly approved Candidate B in the active interaction on 2026-09-19:

```text
aprobada
```

Candidate B preserves every fixed D160 authority. It does not remove isolated P, `Closure.parallel`, Future, Future.all, projection/transfer isolation, or process-local execution. It does not introduce public scheduler/worker/task handles, `parallelEach`, or writable Array/object partition authority.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Current repository evidence

At the evidence revision:

- Core still installs all five `Array.parallel*` selectors through `ProtosParallelRuntime`;
- `spec/concurrency/PARALLEL_EXECUTION.md` standardizes operation-specific privileged contracts;
- `std:collections/Array` already owns ordinary source-backed `map`, `filter`, `findIndex`, `reduce`, and `sort`;
- repository-wide Protos-source search finds no production `protos/lib` or `protos/tools` consumer of the five Core `Array.parallel*` selectors;
- current visible use is examples, benchmarks, documentation and conformance/runtime tests;
- PERF001-F previously demonstrated real multicore scaling through parallel array mapping, proving the retained isolated-P kernel has practical multicore value.

Unlike D161 ByteRegion reservations, the current Array parallel family does not impose reservation state/checks on every ordinary Array operation. Its continuing cost is primarily privileged Core API/spec/runtime policy and implementation freedom, not a per-Array runtime tax.

## Comparative evidence

The investigation compared materially different approaches:

- Java places collection/data-parallel algorithm policy in Streams/collections libraries over ForkJoin/runtime execution machinery;
- .NET PLINQ layers ordered/unordered parallel query policy over Task Parallel Library infrastructure;
- Rust Rayon provides parallel iterator algorithms as a library over a scheduling substrate and deliberately leaves reduction grouping freer than D160's current canonical Core tree;
- C++ places parallel algorithms in the Standard Library and parameterizes them with execution policies;
- Scala parallel collections demonstrate that a parallel collection layer can be separated into a module rather than remain fundamental language/Core surface;
- Swift task groups represent a primitive-first approach where higher-level algorithms are composed above the task mechanism.

The cross-system pattern supports a separation between execution substrate and collection-algorithm policy.

## Candidate set and result

### A — retain the current Core family

Rejected. It preserves compatibility and strong deterministic contracts but freezes five algorithm policies as privileged Core semantics and keeps every implementation responsible for them.

### B — Standard Library algorithms over minimal P

**Selected.** It preserves useful parallel collection capability while relocating algorithm policy to the existing `std:collections/Array` owner and keeping Core focused on isolated P plus Future composition.

### C — Standard Library plus narrower privileged helper kernel

Deferred, not rejected permanently. If measured library implementations show material overhead or an impossible required guarantee, a separately justified narrow helper may be introduced. D160 does not pre-authorize one.

### D — remove the five algorithms without replacement

Rejected. This would discard a useful capability even though isolated P has demonstrated real multicore scaling and `std:collections/Array` is already the natural owner of collection algorithms.

### E — retain only a subset in Core

Rejected. Keeping some algorithms privileged and moving others would create an arbitrary ownership boundary without present evidence that any one of the five represents a uniquely fundamental Core capability.

## Comparative scoring

Scores use 1–5; confidence is H/M.

| Criterion | A | B | C | D | E |
| --- | ---: | ---: | ---: | ---: | ---: |
| Correctness / invariants | 5H | 4H | 5H | 5H | 4H |
| Protos alignment | 3H | 5H | 4H | 4H | 3H |
| Present-need proportionality | 2H | 5H | 3H | 4H | 3H |
| Incremental growth | 3M | 5H | 4H | 3M | 3M |
| Future-option resilience | 3M | 5H | 5H | 4H | 3M |
| Scalability | 5H | 4M | 5H | 2H | 4H |
| Conceptual simplicity | 2H | 5H | 3H | 5H | 2H |
| Portability / implementation freedom | 3H | 5H | 4M | 5H | 3H |
| Runtime / resource cost | 4H | 5H | 4H | 5H | 4H |
| Failure / operability | 5H | 4M | 5H | 5H | 4M |
| Deferral / reversibility / migration | 4M | 5H | 4M | 3M | 3M |
| Evidence maturity / implementation risk | 4H | 4H | 3M | 4H | 3M |
| Total / 60 | 43 | 56 | 49 | 49 | 39 |

The arithmetic total is supporting evidence, not decision authority.

## Guarantees deliberately moved out of Core

The current privileged family specifies more than the retained P/Future substrate intrinsically requires. D160 does not retain the following as Core-level invariants merely because the old algorithms had them:

- one canonical `parallelReduce` logical reduction tree for non-associative reducers;
- one canonical stable merge-sort execution tree for `parallelSort`;
- atomic validation/snapshot of every logical child input before any child becomes eligible;
- operation-specific Core failure topology beyond the retained deterministic Future/P primitives.

The Standard Library implementations must define coherent observable behavior, but they are free to choose library-level contracts that preserve useful determinism without freezing unnecessary Core execution topology.

Existing serial collection behavior and retained P/Future semantics remain authoritative.

## Strongest argument against Candidate B

A privileged runtime implementation can potentially batch, fuse, chunk and coordinate parallel collection work more efficiently than source-level composition of `Closure.parallel` plus Futures. For large arrays with very small callbacks, library composition may allocate more Futures or cross more public abstraction boundaries than a specialized native algorithm.

That risk is real and is the strongest argument for Candidate C.

Candidate B is nevertheless selected because no present measurement demonstrates that five permanent privileged algorithms are required to obtain acceptable performance. The correct incremental path is to implement and benchmark the Standard Library family first. If evidence identifies a bottleneck or an unrepresentable required guarantee, introduce only the narrowest separately justified helper instead of preserving five Core policies in advance.

## Incremental design and future stress

Pay for what you need:

- ordinary Array users do not gain five privileged Core contracts by default;
- parallel collection users opt into `std:collections/Array` and the retained P/Future substrate;
- no speculative private helper is added now.

Grow as you need:

- library algorithms can evolve independently of Core;
- new parallel collection operations can be added in the library without expanding Core;
- measured pressure can later justify a narrow batching/chunking/helper primitive;
- a future helper must be justified by workload evidence and must not silently restore the old five Core selectors.

Future workloads such as large pure transforms, filters, searches, reductions and sorts remain representable through the retained P kernel and Standard Library layer. If source composition becomes the bottleneck, D160 deliberately leaves room for a later optimization decision.

## Compatibility and migration

The observable compatibility break is removal of method-style Core sends such as:

```text
array.parallelMap(worker)
```

The replacement owner is the Standard Library module, following the established module-function style used by serial Array algorithms, for example conceptually:

```text
Arrays: import("std:collections/Array")
Arrays.parallelMap(array, worker)
```

Exact detailed Standard Library argument/error/performance contracts must be reconciled during implementation consistently with this placement decision. No Core compatibility alias is retained merely to preserve the old selector location.

Examples, benchmarks, guide text, conformance tests, native-boundary architecture expectations and normative specifications must be migrated together.

## Intentionally deferred

D160 does not decide:

- a privileged batching/chunking/helper API;
- worker count, chunk size, SIMD width or scheduling strategy;
- whether future library reductions require associativity or expose another reduction-policy contract;
- whether sort/reduce should guarantee exactly the historical Core algorithm trees;
- `Array.parallelEach`;
- writable Array/object partition authority;
- a general parallel-iterator family;
- future auto-parallelization.

Any new Core helper or new fundamental semantic institution requires separate evidence and approval.

## Normative and implementation routing

Candidate B changes observable Core surface and Standard Library surface. A separate implementation owner must:

1. remove all five Core Array parallel selectors and their operation-specific normative Core contracts;
2. add the five parallel facilities to `std:collections/Array` using retained public P/Future mechanisms where sufficient;
3. define coherent Standard Library ordering/failure/cancellation/reduction/sort contracts without silently re-freezing unnecessary Core topology;
4. remove obsolete privileged runtime paths when no retained consumer needs them;
5. preserve the foundational P runtime and `Closure.parallel`;
6. update `PARALLEL_EXECUTION.md`, values/collections/runtime owners, Standard Library documentation, guide/examples/benchmarks/tests and `PROTOS_SPEC_CHANGELOG.md` consistently;
7. benchmark the migrated canonical parallel-map workload so any material regression can be distinguished from semantic correctness;
8. stop and route a new decision if implementation demonstrates a genuine need for a new privileged helper rather than inventing one inside the implementation slice.

```text
D160_STATUS=RATIFIED
SELECTED_CANDIDATE=B
CORE_PARALLEL_ARRAY_ALGORITHMS=REMOVE
STANDARD_LIBRARY_PARALLEL_ARRAY_ALGORITHMS=ADD
FOUNDATIONAL_ISOLATED_P=KEEP
CLOSURE_PARALLEL=KEEP
FUTURE_ALL=KEEP
PRIVILEGED_HELPER=NOT_ADDED_NOW
NORMATIVE_RECONCILIATION_REQUIRED=YES
IMPLEMENTATION_RECONCILIATION_REQUIRED=YES
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
