# I086 / PLAT054-3C — JAR-packaged Core and option-free standard Polyglot embedding

**Status:** SLICE C PUBLISHED; I086/#840 remains OPEN for host authority, Actor/thread and resource-lifecycle completion.  
**Date:** 2026-10-08  
**Exact product revision:** [`2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3`](https://github.com/guillermomolina/protos/commit/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3)  
**Commit subject:** `I086 PLAT054-3C: package Core in the JAR for option-free Polyglot embedding`  
**Issue:** [I086/#840](https://github.com/guillermomolina/protos/issues/840)  
**Ratified design:** [PLAT054/#838](https://github.com/guillermomolina/protos/issues/838)  
**Performance follow-on:** [PERF032/#831](https://github.com/guillermomolina/protos/issues/831)  
**Product version:** `0.3.286-SNAPSHOT`; **normative spec:** `0.1.449`, unchanged.

## Exact publication and concurrent history

The GitHub product HEAD was inspected and equals the published slice-C commit `2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3`. It contains **10 changed paths**:

- New `src/main/java/com/guillermomolina/protos/execution/ProtosCoreResource.java`.
- New `src/test/java/com/guillermomolina/protos/execution/ProtosPackagedCoreEmbeddingTest.java`.
- New `dist/Plat054EmbeddingProbe.java` and `dist/smoke_polyglot_embedding.sh`.
- Updated `src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java`, `ProtosEmbeddedProcess.java`.
- Updated `dist/validate_portable.sh` and `dist/README.md`.
- Updated `pom.xml` and root `CHANGELOG.md` atomically with the product publication, including version `0.3.286-SNAPSHOT`.

A concurrent unrelated I085-B product commit `4f459f2de119d956b8671ad1c1da7b9fa6bcfa17` introduced `0.3.285-SNAPSHOT` between I086-B's `0.3.284-SNAPSHOT` and I086-C. The precise published version for C is **0.3.286**, not 0.3.285.

## Verified implementation structure

`ProtosLanguage` registers `ProtosCoreResource` as a Truffle `internalResources` class. Its implementation uses Truffle internal-resource APIs to expand a bundled `protos/lib` directory, including a file list and SHA-256 content hash, under the Truffle resource cache. The `pom.xml` resource processing packs `protos/lib` under `META-INF/resources/protos/core/lib`, with `files` and `sha256` metadata. This is a private host/runtime location, **not** a guest `filesystem` or `network` capability and not an unpack-per-Context operation.

`ProtosEmbeddedProcess.resolveCoreRoot` now uses:

1. Explicit `protos.CoreRoot`: valid override wins; invalid override fails explicitly without fallback.
2. A language home that actually contains `protos/lib/core`.
3. The Core bundled inside the JAR via `Env.getInternalResource(ProtosCoreResource.class)`.

A language home without Core is not itself a valid Core origin and falls through to the packaged resource, as expressly approved in the owner/assistant implementation decision while C was being validated. If Core is found in the language home but bootstrap is defective, existing bootstrap failure semantics are preserved; no silent fallback hides a failed initialization.

The existing Core bootstrap, standard-library resolver, Actor-local canonical `std:` `ModuleKey` semantics, and module cache remain in use. The host-supplied first `eval` still lazily constructs one Process/RootActor in a Context, and no Core or Process is created just by `Context.newBuilder(...).build()` or reading language bindings.

**Public API now supported without a CoreRoot option:**

```java
try (Context context = Context.newBuilder("protos").build()) {
    context.eval("truffleRun: () => { 42 }");
    Value run = context.getBindings("protos").getMember("truffleRun");
    assert run.execute().asInt() == 42;
}
```

`ProtosPackagedCoreEmbeddingTest` checks Core's real resolved cache location, independence from `protos/lib` in the repository, option-free execution, two independent contexts sharing one cached unpacked resource, standard module import/cache/identity, no default filesystem/network slots, Context close effects on the cache, and retention of Java scalar argument admission and no-Task ordinary calls.

The smoke `dist/smoke_polyglot_embedding.sh`, now called by `dist/validate_portable.sh`, extracts the actual portable distribution ZIP, copies only the product/runtime JARs, removes the extracted distribution tree, and runs a compiled Java probe from an unrelated empty project directory. It validates packaged Core presence, option-free execution, standard import, language-home precedence, explicit override precedence and invalid-override failure. This is **real JAR/distribution coverage**, not merely a classpath test against the source checkout.

## Validation provenance and limits

The project owner reported for published slice C:

> I086 PLAT054-3C: package Core in the JAR for option-free Polyglot embedding, pushed.

> el git diff check esta limpio. Todos los tests han pasado en local

The product commit, build/test sources and distribution smoke wiring were inspected through GitHub. **All-tests PASS and clean diff check are owner-reported**, not independent re-execution. Exact commands completed, CI logs, test counts, native-image run results and Native validation exit codes were not supplied. Consequently Native Image compatibility and cross-runtime packaging are not independently certified by this record, even though internal-resource source declares its intended Native behavior.

```text
SLICE=I086_PLAT054_3C
PRODUCT_REVISION=2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3
PRODUCT_VERSION=0.3.286-SNAPSHOT
SPEC_REVISION=0.1.449_UNCHANGED
SOURCE_COMMIT_VERIFIED=YES
COMMIT_CHANGED_PATHS=10
POM_CHANGELOG_ATOMIC_PUBLICATION=YES
PUBLIC_OPTION_FREE_EMBEDDING=IMPLEMENTED
CORE_RESOLUTION_ORDER=CORE_ROOT_OVERRIDE;VALID_LANGUAGE_HOME;PACKAGED_TRUFFLE_RESOURCE
EXPLICIT_INVALID_OVERRIDE=FAIL_CLOSED
EMPTY_LANGUAGE_HOME=FALLBACK_TO_PACKAGED
PACKAGED_CORE_TESTS_AND_PORTABLE_SMOKE=PUBLISHED
HUMAN_ALL_LOCAL_TESTS=PASS_REPORTED
HUMAN_GIT_DIFF_CHECK=CLEAN_REPORTED
RAW_TEST_LOGS_OR_CI=NOT_RECEIVED
NATIVE_IMAGE_GATE=NOT_INDEPENDENTLY_VERIFIED
AGENT_BUILDS_TESTS_PRODUCT_COMMITS=NONE
I086_ISSUE=OPEN
```

## Remaining I086 scope — next bounded implementation D

I086-D needs to establish the **explicit Polyglot host authority mapping** (default Filesystem and Network only when the host has granted corresponding authority, never merely from packaged-Core files), **Actor/thread policy** under the Context's thread-creation permission, and **Context close resource/provider cleanup** with actual open resources. The published slice A uses `ProtosStandaloneProcessBootstrap.create(..., null, null)` for default capabilities and creates Actor carrier threads lazily through `Env.newTruffleThreadBuilder`; the full positive/negative grants and thread-permission cases need product-level acceptance and may require implementation. `IO_CORE.md`, `FILESYSTEM.md`, `NETWORK.md`, `PROCESS_IO.md` and Actor contracts govern behavior. Existing resource/provider ownership must be audited before implementation; no silent new authority, false positive from just `allowAllAccess`, or eager setup.

If the normative contracts and present GraalVM APIs don't determine an exact safe authority mapping, stop **only the affected authority decision** and seek explicit ratification; independently grounded thread/lifecycle tests and corrections can still be completed as a bounded implementation slice. Do not create artificial micro-issues.

**Separate consumer:** PERF032/#831 graph-size or speed parity remains completely unproven and must be measured on a matched execution plane; completion of I086-C alone does not make that performance claim.

## AI assistance and executor

ChatGPT prepared the project evidence from the exact public commit and owner report. The implementation, validation, product commit and push were performed by the human executor. Publication of this evidence in `guillermomolina/protos-project-docs` is explicitly agent-authorized.
