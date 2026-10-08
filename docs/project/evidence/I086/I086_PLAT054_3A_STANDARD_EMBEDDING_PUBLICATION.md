# I086 / PLAT054-3A — Standard Polyglot embedding with a lazy per-Context Protos Process

Status: **SLICE A PUBLISHED — I086 REMAINS OPEN**

Publication date: 2026-10-08. Product revision: [`3814eca12200f9ed2a6607f0002ef65cce71599e`](https://github.com/guillermomolina/protos/commit/3814eca12200f9ed2a6607f0002ef65cce71599e), commit subject `I086 PLAT054-3A: standard Polyglot embedding with lazy per-Context Process`.

Owning implementation: [I086 / guillermomolina/protos#840](https://github.com/guillermomolina/protos/issues/840). Ratified governing platform decision: [PLAT054 / #838](https://github.com/guillermomolina/protos/issues/838); language specification revision `0.1.449`, normative publication `ab1f2196e6736d4df01e83f685a3fc8aa3f606ac`. Performance consumer: [PERF032 / #831](https://github.com/guillermomolina/protos/issues/831).

## Publication inspection and scope

The exact published commit was inspected through GitHub. It contains **16 paths**: `pom.xml`, `CHANGELOG.md`, four new embedding runtime components, changes to six existing runtime/execution components and `ProtosValueLookup`, adjustments to three existing architecture/scope tests, and one new acceptance test.

New files:
- `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedProcess.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddingException.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosHostBindingsScope.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosHostEvalRootNode.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandardPolyglotEmbeddingTest.java`

Other changed Java files: `ProtosHostExecutableClosure.java`, `ProtosLanguage.java`, `ProtosLanguageContext.java`, `ProtosPolyglotExecutionContext.java`, `ProtosStandaloneHostedExecution.java`, `ProtosValueLookup.java`, and existing `ProtosA4B3ProductionEntryArchitectureTest.java`, `ProtosI026EScopeTest.java`, `ProtosPerf006B6BFinalConformanceArchitectureTest.java`.

**Product version:** `pom.xml` advanced from `0.3.282-SNAPSHOT` to `0.3.283-SNAPSHOT`; the corresponding root `CHANGELOG.md` entry is published **in the same commit**. Contrary to the earlier provisional implementation report, metadata finalization is therefore already complete **for slice A**, not deferred.

## Delivered product behavior

The public standard Polyglot sequence:

```java
try (Context context = Context.newBuilder("protos")
        .option("protos.CoreRoot", corePath).build()) {
    context.eval("truffleRun: () => { 42 }");
    Value run = context.getBindings("protos").getMember("truffleRun");
    int result = run.execute().asInt();
}
```

is now supported **when the caller supplies `protos.CoreRoot`**, or when the runtime's language home already supplies Core. The ultimate required **jar-internal Core fallback** is NOT delivered by slice A. Therefore the completely option-free example is not yet a portable guarantee.

Other slice-A behaviors:
- Process/RootActor/Core initialization waits until the first valid `eval`; pre-evaluation bindings and parse failures do not create the Process.
- One persistent Process per live Context; subsequent evals are separate standalone moduleContexts in the same RootActor; ordinary Actor-local `import` caching is preserved. No claim is made that a host eval source can currently acquire a canonical importable `ModuleKey` from the standard API.
- Truffle bindings form a stable Java-read-only view of the last normally completed entry's own local slots; re-selection preserves retained Closure values.
- Closure extraction is fresh/receiver-bound; host calls reuse the existing PERF033 compact path with no mandatory Task; native Closures use ordinary generic guest invocation.
- Arguments already represented as Protos values are supported; ordinary `Value.execute(2, "x")` host arguments are **NOT yet admitted** (no implicit conversion to Protos Integer/String).
- A fatal escaping RootActor Error terminates the Process without recreation; concurrency into the same RootActor rejects unsafe simultaneous entry; Context close terminates the Process and joins Actor carriers.
- Process args/environment/standard streams come from Truffle Env. The initial module gets `process` only; filesystem and network capabilities are **not** granted in this slice. Driver/session parsing retains its prior execution behavior.

These are product implementation descriptions derived from inspected code and the exact implementation changelog, not an assertion that every edge was independently executed here.

## Validation provenance

The project owner reported after pushing this revision:

> I086 PLAT054-3A: standard Polyglot embedding with lazy per-Context Process, pushed

> el git diff check esta limpio. Todos los tests han pasado en local

The following results are **human-reported**. No raw test logs, CI report, benchmark result or actual test count were provided and none was independently run by ChatGPT.

```text
PROTOS_REVISION=3814eca12200f9ed2a6607f0002ef65cce71599e
PRODUCT_VERSION=0.3.283-SNAPSHOT
SPECIFICATION_REVISION=0.1.449
PUBLISHED_SOURCE_COMMIT=VERIFIED
HUMAN_REPORTED_ALL_LOCAL_TESTS=PASS
HUMAN_REPORTED_GIT_DIFF_CHECK=CLEAN
AGENT_EXECUTED_BUILD_OR_TEST=NO
CHANGED_PATHS=16
SLICE_A=PUBLISHED
I086_COMPLETE=NO
```

## Remaining scoped work

**Slice B — host scalar argument admission**, next: convert supported plain Java integer/string host values passed to embedding `Value.execute(...)` into exact ordinary Protos Integer/String representation before guest argument binding; preserve the original Protos-valued path, ordinary argument/default/rest order, runtime ownership and fatal guest Error behavior. No arbitrary Java object admission, no unapproved numeric narrowing/coercion and no foreign-capability amplification. The source of exact allowed scalar types and numeric range must come from the current Protos normative value contract and existing runtime code.

**Slice C — full distribution/authority/lifecycle closure**: internal packaged Core so `Context.newBuilder("protos").build()` works without a CoreRoot option or language home; bounded host filesystem/network grants; actor spawning under explicit host thread policy; close-with-open-resources; related product/distribution tests; version and changelog update **for changes actually made in slice C**. Do not defer the already published A version.

Potential semantic/authority gates in C: how the host's Polyglot builder IO/socket/thread privileges map to Protos represented capabilities; whether a host-supplied entry Source is admitted to canonical `ModuleKey` through a current resolver; and thread creation policy for embedded Actors. If current normative specifications plus GraalVM API do not determine a safe public behavior, fail closed and request a discrete owner decision; do not silently invent a host authority policy.

## Independence and performance boundary

I086 remains open. No `protos-benchmarks` mutation is part of A, B or C. PERF032 remains responsible for source-preparation parity, Graal node counts and timing analysis *after* a valid product embedding plane exists. This slice neither establishes nor measures those results.

## AI-assistance disclosure

ChatGPT inspected the exact published GitHub commit and prepared this durable evidence record. The product edit, local validation, commit and push were performed by the human executor; no independent local test re-run is claimed.
