# PLAT054-0 — Standard Polyglot bindings and benchmark-harness reconciliation evidence

Date: 2026-10-08
Status: RESEARCH EVIDENCE — NOT A RATIFIED DECISION
Decision owner: https://github.com/guillermomolina/protos/issues/838
Parent causal/performance owner: https://github.com/guillermomolina/protos/issues/831
Related closed callable-interop owner: https://github.com/guillermomolina/protos/issues/832

## Exact source identities inspected

- `guillermomolina/protos@ac1e660cc37f8629852062fee41dcc33cf0758f8` — current product read reference.
- `guillermomolina/protos-benchmarks@e221bb056a208693c9102891df2e13d65e61eefb` — inspected benchmark reference. Benchmark repository was read only.
- `guillermomolina/protos-project-docs@99a80b3923d07de24d66e68528245009a2653c8c` — documentation reference before this evidence was published.

## Concrete asymmetry: preparing `primitive-return-literal`

The benchmark's identical *timed* callable interop entry is `Value.execute()`, but its **preparation** still differs. This must not be confused with an already-proved difference in Graal graphs or with an execution-time `PreparedTopLevel.invoke()` gate:

```text
GraalJS / GraalPy:
  context.eval(source)
  Value run = context.getBindings(language).getMember("truffleRun")
  [timed] run.execute()

Protos canonical (post-PERF033):
  ProtosStandaloneHostedSession.open(...)
  PreparedTopLevel prepared = session.prepareTopLevel("run")
  Value run = prepared.executable()
  [timed] run.execute()
```

Source evidence:

- [Peer JVM runner `executableRun`](https://github.com/guillermomolina/protos-benchmarks/blob/e221bb056a208693c9102891df2e13d65e61eefb/truffle/src/peer/java/com/guillermomolina/protos/benchmarks/truffle/TrufflePeerJvmRunner.java).
- [Protos canonical surface](https://github.com/guillermomolina/protos-benchmarks/blob/e221bb056a208693c9102891df2e13d65e61eefb/truffle/measure/surfaces/canonical/ProtosCanonicalSurface.java).
- [Protos prepared surface](https://github.com/guillermomolina/protos-benchmarks/blob/e221bb056a208693c9102891df2e13d65e61eefb/truffle/measure/surfaces/prepared/ProtosPreparedSurface.java), a **different stronger** session-gated surface, must not be substituted for the canonical one.
- [Protos workload](https://github.com/guillermomolina/protos-benchmarks/blob/e221bb056a208693c9102891df2e13d65e61eefb/truffle/workloads/primitive-return-literal/primitive-return-literal.protos) declares `run: () => { 1 }`; peers expose their function as `truffleRun`. This identifier mismatch is not itself a semantic/performance finding.
- [Graph case selection](https://github.com/guillermomolina/protos-benchmarks/blob/e221bb056a208693c9102891df2e13d65e61eefb/truffle/measure/graphs.json): Protos `canonical` versus JS/Python `executable-value`; primary compilation root matching and inlining parity remain a separate PERF032 causal check.

**Correct interpretation:** PERF033 already converged the timed host-to-guest call on `Value.execute()`, but it did **not** implement the normal Protos `Context.eval + getBindings` preparation. PLAT054 closes that product embedding API gap. It cannot, without new retained tests/measurements, be credited with a graph-node reduction or timing improvement.

## Confirmed product and decision constraints

- [ProtosLanguage](https://github.com/guillermomolina/protos/blob/ac1e660cc37f8629852062fee41dcc33cf0758f8/src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java) currently creates `ProtosLanguageContext` and returns `sourceCompiler.compileBytecode` from `parse`, but does not override `getScope` or `disposeContext`. Ordinary `Context.eval` is not demonstrated to bootstrap Process/RootActor and publish the requested scope.
- [ProtosHostExecutableClosure](https://github.com/guillermomolina/protos/blob/ac1e660cc37f8629852062fee41dcc33cf0758f8/src/main/java/com/guillermomolina/protos/execution/ProtosHostExecutableClosure.java) preserves captured Closure, caller and owning Context with compact direct-call machinery. Its present API accepts only source-backed, zero-argument Closures. General member exposure must not silently promise all argument shapes without a design/implementation contract.
- [PLAT001](https://github.com/guillermomolina/protos-project-docs/blob/main/docs/project/decisions/platform/PLAT001_TRUFFLE_RUNTIME_HOSTING.md) requires one multithread Context per hosted Process without identifying the semantic Process with the Truffle object.
- [PLAT046](https://github.com/guillermomolina/protos-project-docs/blob/main/docs/project/decisions/platform/PLAT046_ORDINARY_HOSTED_SINGLE_ACTOR_CALLER_EXECUTION_BOUNDARY.md) and its 2026-10-07 approved amendment reject universal RootTask/Task/activation/serialization/explicit enter-leave machinery for canonical ordinary Closure interop.
- [PLAT053](https://github.com/guillermomolina/protos-project-docs/blob/main/docs/project/decisions/platform/PLAT053_FOREIGN_PROVIDER_RUNTIME_LIFECYCLE_ARCHITECTURE.md) preserves lazy Process-owned foreign provider compartments and Actor-isolated sessions; ordinary embedding is not an implicit foreign-provider authorization.
- [MODULES.md](https://github.com/guillermomolina/protos/blob/ac1e660cc37f8629852062fee41dcc33cf0758f8/spec/semantics/MODULES.md): module instances are ordinary `moduleContext` objects with Actor-local cache/identity, canonical importable-entry rules and non-importable standalone-entry rules; there is no independent export registry.
- [PROCESS_IO.md](https://github.com/guillermomolina/protos/blob/ac1e660cc37f8629852062fee41dcc33cf0758f8/spec/io/PROCESS_IO.md): `process` and any authorized default `filesystem`/`network` capability are RootActor *initial-module local slots*, not prelude globals or grants to every imported/additional module.
- [ACTORS.md](https://github.com/guillermomolina/protos/blob/ac1e660cc37f8629852062fee41dcc33cf0758f8/spec/concurrency/ACTORS.md) sections 24C and 32: an unhandled RootActor turn Error is fatal and terminates a minimal Process. Persistent Context semantics cannot silently override this.

## Owner-approved direction preserved, not reopened

The eight recorded owner directions in [PLAT054/#838](https://github.com/guillermomolina/protos/issues/838) are carried forward: (1) lazy Process per embedder Context; (2) Core from installed distribution/language home with explicit override; (3) Env-governed stdio/args/environment/filesystem and denied-by-default network; (4) persistent Process and ordinary module-instance eval, not REPL, with bindings of last successful entry; (5) host read-only bindings/ordinary guest mutation; (6) no universal per-call serialization and reject unsafe concurrent entry; (7) captured Closure identity survives later eval; (8) PAY AS YOU GROW with no universal per-execute bootstrap, RootTask, Task, rich activation or scheduler.

Proposed public embedding shape (NOT YET IMPLEMENTED):

```java
try (Context context = Context.newBuilder("protos").build()) {
    context.eval(source);
    Value run = context.getBindings("protos").getMember("truffleRun");
    Value result = run.execute();
}
```

## Normative reconciliation still required under GITHUB010

1. **Unhandled Error versus successive eval:** preserve the normative fatal RootActor/Process outcome; do not invent automatic same-Context Process recreation or reclassify an unhandled guest Error as recoverable. Precisely define host exception propagation, eval failure before bootstrap, handled Error, and `getBindings` behavior after fatal termination. If a different recovery guarantee is desired, surface exact ACTORS replacement for owner approval.
2. **Several direct-entry modules in one RootActor:** specify module identity, cache-before-execute for importable canonical entries, distinct standalone entry instances without fabricated ModuleKeys, bootstrap-local `process`/`filesystem`/`network` slots only where normative, selector update on normal completion and unchanged selector on failed eval. Reconcile `MODULES.md` initial-module wording and `PROCESS_IO.md` before implementation. This newly specified embedding contract requires explicit approval; do not infer approval from permission to coordinate or publish records.
3. **Bindings/interop details:** assess Truffle 25.4.4.1.1 public `getScope` behavior, retained scope object and member visibility after successive evals, read-only Java view versus guest mutation, exact Closure capture, cross-Context use/close failure, native and non-zero-arity Closure behavior, and rejected concurrent unsafe RootActor entry. Do not create a second export registry.
4. **Core/security:** define exactly the public override spelling and resolution precedence and prove no ambient filesystem/network authority enhancement. Context construction and `getBindings` before first eval must not initialize Process/Core/RootActor unnecessarily.
5. **GITHUB010 completion:** full normative and official Truffle versioned API read, adversarial alternative comparison, candidate scoring and explicit owner-approved-invariant/delta review remain part of the approval gate. This evidence file itself is not PLAT054 ratification.

## After ratification

Propose **one** coherent product implementation in `guillermomolina/protos`: lazy bootstrap + `Context.eval` entry + language scope + reusable compact callable interop + Process/Context close + regression tests. Do **not** change `guillermomolina/protos-benchmarks` in this stage. Later compare preparation and selected compiled roots/graphs, then separately measure; equivalence of APIs alone does not prove structural/latency parity.

## Evidence and validation boundaries

```text
PLAT054_0_RESULT=SOURCE_RECONCILIATION_RECORDED
OWNER_DIRECTION_PRESERVED=YES
CANONICAL_TIMED_CALL_EQUALS_VALUE_EXECUTE=YES
PREPARATION_SYMMETRY_ALREADY_EXISTS=NO
RAW_POLYGLOT_EMBEDDING_IMPLEMENTED=NO
GRAPH_PARITY_CLAIM=NO
PERF032_CLOSURE_CLAIM=NO
NORMATIVE_RATIFICATION_COMPLETE=NO
IMPLEMENTATION_AUTHORIZED=NO
NEXT_SLICE=PLAT054_1
NEXT_SLICE_TYPE=INVESTIGATION_RATIFICATION
NEXT_REPOSITORY=NONE
NEW_ISSUE_REQUIRED=NO
```

No product, benchmark, build or test commands were executed for this documentation-only coordination. No `git diff --check` was executed in this interaction, and it is **not** claimed as PASS. Publication via the authorized GitHub repository API records exact repository content; local validation remains unverified.

## AI-assistance disclosure

This evidence record was prepared with AI assistance from ChatGPT by reviewing current GitHub repository source, normative material, ratified architecture and benchmark harness; it does not claim independently executed tests.
