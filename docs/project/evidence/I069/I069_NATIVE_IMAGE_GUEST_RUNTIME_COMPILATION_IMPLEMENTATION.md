# I069 — Native Image guest runtime compilation implementation evidence

FORMAL_IDENTIFIER=I069
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/711
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
BASE_PROTOS_REVISION=0c9307240796260882ddca27611c91b7ffa4ce3b
PROTOS_REVISION=7f033b059e2e25a1a6d1776e7eb1f960f7ab814e
PROTOS_VERSION=0.3.89-SNAPSHOT
COMMIT_MESSAGE=i069: enable native guest runtime compilation
PLAT038_AUTHORITY=docs/project/decisions/platform/PLAT038_NATIVE_IMAGE_BOOTSTRAP_RUNTIME_RELEASE_BOUNDARY.md@816aeb1d3ab96bffa691dce26c66908d9b8323cd
PLAT039_AUTHORITY=docs/project/decisions/platform/PLAT039_TRUFFLE_RUNTIME_COMPILATION_BOUNDARY.md@485bec973b63bc3c8dd28f4bb184295ab472bbd5
IMPLEMENTATION_STATE=COMPLETE

## Result

I069 implements the PLAT038 Native Image bootstrap/runtime boundary and the
PLAT039 Candidate C PE-visible guest-kernel rule in the product repository.

The published implementation establishes:

```text
STAGE1_JVM_RETAINED=PASS
NATIVE_BUILD_ENTRY_POINT=PASS
NATIVE_BUILD=PASS
NATIVE_EXECUTABLE_SMOKE=PASS
TRUFFLE_RUNTIME_COMPILER_IN_NATIVE=PASS
FORCED_GUEST_JIT=PASS
HELPER_BYTECODE_ROOT_TIER2=PASS
SEMANTIC_BYTECODE_ROOT_TIER2=PASS
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURE=0
BUILD_NATIVE_DIST_RELEASE_SEPARATION=PASS
CHANGELOG_DOES_NOT_TRIGGER_DIST=PASS
CHANGELOG_DOES_NOT_TRIGGER_RELEASE=PASS
NO_OBSERVABLE_PROTOS_SEMANTIC_CHANGE=PASS
FINAL_REQUIRED_VALIDATION=PASS
```

The accepted Native Image is not interpreter-only: forced Truffle compilation
successfully compiles both generated Protos Bytecode DSL roots.

## Product implementation

The product repository now contains a first-class Maven `native` profile that:

- generates Native Image class-initialization arguments before packaging;
- invokes GraalVM Build Tools `native-maven-plugin` 1.1.14;
- builds the Stage1 native `protos` executable;
- preserves the ordinary JVM development path;
- keeps native build separate from distribution and release publication.

Durable native build inputs are:

```text
build/native/Dockerfile
build/native/generate-init-args.sh
```

Investigation-only Native Image graph, blocklist, frontier, policy, and PE-audit
Python programs were intentionally not retained as product tooling.

## Native initialization policy

The accepted initialization policy is deterministic and narrower than
package-wide `--initialize-at-build-time`.

It includes:

- the generated language provider;
- generated Truffle LibraryExports classes;
- helper and semantic Bytecode DSL structural root classes;
- generated helper and semantic operation `*_Node` classes;
- `ProtosTextReaderCPrimeExecution.Advance`;
- `ProtosCoreErrors.StandardError`.

The validated current-tree policy contains:

```text
BUILD_TIME_INITIALIZED_CLASSES=311
DUPLICATES=0
MISSING_EXPECTED_CLASSES=0
UNEXPECTED_CLASSES=0
CURRENT_NATIVE_INIT_POLICY=PASS
```

This policy removes the Native-only generated-root initialization failure
without broad package initialization.

## PLAT039 implementation

The guest PE kernel was audited and adjusted under the ratified
`KEEP_PE | BOUNDARY | SPLIT` rule.

The implementation keeps guest-hot semantics PE-visible while using narrow
host/cold boundaries and structural separation where required. The affected
surfaces include:

- Bytecode send/call/lookup and generated operation execution;
- closure parameter/default/rest binding;
- frame-backed current and captured lexical access;
- object/member and binding traversal;
- module execution-plan cache hits;
- continuation, suspension, Future, Task, Actor, and parallel runtime state;
- standard protocol/native closure paths;
- debugger and interop projections;
- host filesystem/network/cold materialization edges.

PE-sensitive guest paths avoid unsuitable iterator, collection-view, stream,
Optional, or equivalent opaque shapes where the Native runtime compiler could
not retain the required guest structure.

No broad enter/resume boundary was introduced.

## Hosted-constant closure fixes

Standard protocol fast paths that had depended on identity-sensitive native
lambda singletons now use explicit marker `ProtosNativeClosureBody`
implementations for the relevant Object, IdentityMap, Map, and Array paths.

This removed the corresponding hosted-constant closure problem while preserving
the existing protocol semantics.

## Runtime compiler validation

The successful final Native Image reported:

```text
RUNTIME_COMPILED_METHODS=2132
NATIVE_BUILD=PASS
BLOCKLIST_FAILURES=0
LATE_DEOPT_FAILURES=0
FRAME_BUILD_FAILURES=0
```

Forced compilation was then executed against that same image with Truffle
experimental compilation enabled, background compilation disabled, immediate
compilation enabled, and compilation tracing enabled.

Observed acceptance:

```text
FORCED_JIT_OPT_DONE=2
FORCED_JIT_BOTH_ROOTS=TRUE
FORCED_JIT_OPT_FAILED=0
FORCED_JIT_FRAME_FAILURE=FALSE

ROOT=ProtosBytecodeRootNodeGen
TIER=2

ROOT=ProtosSemanticBytecodeRootNodeGen
TIER=2
```

The earlier Native-only error:

```text
FrameWithoutBoxing; should not be materialized
(must not pass virtual object into an invoke that cannot be inlined)
```

is absent from the accepted image's forced guest compilation.

## Final validation evidence

Human-executed validation reported during the implementation session:

```text
NATIVE_VERSION_SMOKE=PASS
NATIVE_HELP_SMOKE=PASS
NATIVE_BASIC_GUEST_EVAL=PASS

NATIVE_FORCED_GUEST_JIT=PASS
FORCED_JIT_BOTH_ROOTS=TRUE
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURE=0

FINAL_MVN_VERIFY=PASS
GIT_DIFF_CHECK=PASS
FINAL_STATIC_REVIEW=PASS
INDEX_CANDIDATE=PASS
```

The immutable publication candidate is:

```text
BASE=0c9307240796260882ddca27611c91b7ffa4ce3b
HEAD=7f033b059e2e25a1a6d1776e7eb1f960f7ab814e
VERSION=0.3.89-SNAPSHOT
```

GitHub `main` was observed at the exact published HEAD before this evidence
record was created.

## Architectural invariants retained

```text
INTERPRETER_ONLY_NATIVE=NO
STAGE1_JVM_REMOVED=NO
BROAD_PACKAGE_BUILD_TIME_INIT=NO
BROAD_ENTER_RESUME_BOUNDARY=NO
SECOND_LEXICAL_BINDING_AUTHORITY=NO
PLAT036_I068_FRAME_ARCHITECTURE_REOPENED=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
DIST_PUBLICATION_PERFORMED=NO
RELEASE_PUBLICATION_PERFORMED=NO
```

## Performance scope

I069 establishes Native Image runtime-compiler correctness and guest JIT
reachability. It does not claim that the previously observed Protos performance
gap is resolved.

Performance attribution and benchmark remediation remain separate work, including
the paused PERF010-A investigation.

## Closure

I069 is complete at Protos revision
`7f033b059e2e25a1a6d1776e7eb1f960f7ab814e`.

The implementation satisfies the PLAT038 requirement that Native Image remain a
first-class build artifact with a real Truffle runtime compiler, and the PLAT039
requirement that the guest kernel remain PE-visible except for evidence-backed
narrow host/cold boundaries or structural splits.
