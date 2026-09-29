# DIST005-A — Native Image release-admission investigation

Status: PUBLISHED  
Owning Issue: `guillermomolina/protos#549` (`DIST005`)  
Slice: `DIST005-A — Native Image release-admission investigation`

## Investigation identity

```text
PROTOS_REVISION=d18822e968a1ee6986d832731054b9b90b332b1d
PROTOS_VERSION=0.3.108-SNAPSHOT
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1

DIST006_C1_PROJECT_RECORD_REVISION=b457b9893281479deab33e724e69b3b632330189
DIST006_C1_PROJECT_RECORD_PATH=docs/project/evidence/DIST006/DIST006_C1_PORTABLE_NATIVE_IMAGE_25_4_CLOSURE.md
```

This investigation was read-only. It did not modify Protos product files, run
builds/tests/project programs, materialize release candidates, create tags or
releases, or change the selected DIST007 portable-JVM release path.

Current `main` remained exactly the DIST006-C1 product revision above during
the investigation.

## Governing authority reconciled

The current architecture is not a blank Native Image design problem.

PLAT038 already ratifies:

- dual JVM + Native implementation forms;
- Native Image as a first-class runtime artifact;
- JVM as the primary implementation/compiler-diagnostic surface;
- preservation of the external `PROTOS_HOME` Protos resource tree;
- no requirement for a complete single-file Protos toolchain;
- required Native guest Truffle runtime compilation;
- Native conformance for Native readiness/distribution/release; and
- reuse of the DIST001 detached release-candidate model.

PLAT039 already ratifies the runtime-compilation boundary: guest-hot semantics
remain visible to Truffle Partial Evaluation while narrow host/cold/tooling work
crosses explicit boundaries. No new Protos semantic redesign is required for the
current Native path.

DIST006-C1 then proves the current 25.4 implementation can build and execute as
Native Image while preserving guest runtime compilation:

```text
NATIVE_BUILD=PASS
NATIVE_VERSION_SMOKE=PASS
NATIVE_HELP_SMOKE=PASS
NATIVE_GUEST_SMOKE=PASS
NATIVE_FORCED_GUEST_JIT=PASS
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURES=0
COMPILATION_FAILURES=0
HELPER_BYTECODE_ROOT_TIER2=PASS
SEMANTIC_BYTECODE_ROOT_TIER2=PASS
```

Those results establish Native implementation viability, not complete
distribution/release admission.

## Current Native build and resource model

Current repository inspection establishes:

- the Maven `native` profile owns Native Image construction;
- `org.graalvm.buildtools:native-maven-plugin:1.1.14` is the current plugin;
- the canonical Native build container is
  `ghcr.io/graalvm/native-image-community:25i4-25.0.4.1.1-ol10`;
- `build/native/generate-init-args.sh` owns the current generated
  Truffle/Bytecode build-time initialization set;
- `build/native/test-native.sh` currently proves only version/help/basic guest
  execution plus forced guest runtime compilation;
- current Native validation points `PROTOS_HOME` at the source repository.

The public CLI remains filesystem-resource based. `ProtosCli.core()` resolves:

```text
$PROTOS_HOME/protos/lib/core
```

and the higher-level surfaces derive the rest of the distribution tree from the
same root, including `protos/lib`, bundled Package/Test Tool sources and the
test corpus where required.

Therefore the current architecture supports a relocatable Native archive of the
form:

```text
archive root/
  bin/protos              native executable or launcher to it
  protos/lib/...
  protos/tools/...
  protos/tests/...        where required by the distributed Test Tool contract
  release/provenance/license metadata
```

A complete single executable is not required by existing architecture.

## Native release-surface audit

The current evidence status is:

| Surface | Current Native evidence | Release-admission requirement |
| --- | --- | --- |
| `protos --version` | PASS | required |
| `protos --help` | PASS | required |
| `protos -e <source>` | basic PASS | representative extracted-distribution proof required |
| `protos <file>` | GAP | required |
| REPL | GAP | required |
| `protos run` | GAP | required |
| `protos package` | GAP | required |
| `protos test` | GAP | required |
| `protos language-server` | GAP | required |
| `protos debug <file>` / DAP | GAP | required |
| optimizing guest Truffle runtime | PASS at DIST006-C1 | must be reproved from packaged Native artifact |

No current GAP is classified as a known incompatibility. These are missing
execution proofs.

## Closed-world and resource risk

No objective closed-world architectural blocker was found.

Production repository inspection found no broad project-owned
`Class.forName`/`java.lang.reflect` mechanism comparable to the reflective
test code. One explicit Protos classpath resource,
`/com/guillermomolina/protos/lexer/unicode17.bin`, is loaded with a constant
class/resource-name path.

Third-party/tooling paths remain material until executed. In particular:

- LSP4J uses reflective service endpoint machinery and dynamic-proxy based
  remote endpoints;
- JLine owns the real interactive terminal path;
- Graal DAP instrumentation must be proven from the Native image, not inferred
  from JVM DAP compatibility;
- Package/Test Tool source and filesystem discovery must be exercised from the
  extracted Native distribution tree.

GraalVM Native Image closed-world reachability can therefore still expose a
bounded metadata/configuration defect when one of these unexecuted paths is
reached. Current evidence neither proves nor identifies such a defect.

```text
CLOSED_WORLD_BLOCKER_PRESENT=UNKNOWN
SEMANTIC_REDESIGN_REQUIRED=NO
```

## Native LSP admission gate

A release-grade Native LSP proof must run the distributed
`protos language-server` over stdio and complete a real protocol session.

Minimum required sequence:

```text
initialize
  -> valid InitializeResult/capabilities
initialized
textDocument/didOpen
  -> representative publishDiagnostics
one representative query
  -> documentSymbol, definition/references, or workspace/symbol
shutdown
exit
  -> clean process termination
```

This must use the exact extracted Native artifact and an actual project/source
fixture so JSON transport, reflective/proxy machinery, Protos static analysis and
filesystem/project binding are all reached.

## Native Debug/DAP admission gate

The Native debugger proof must use the real
`ProtosPolyglotRuntimeHost.openDebug()` path and the existing public
`PROTOS_DEBUG_READY` launcher contract.

Minimum required sequence:

```text
protos debug <known file>
  -> receive one valid PROTOS_DEBUG_READY record
  -> connect to reported loopback TCP endpoint
  -> DAP initialize/attach
  -> install representative source breakpoint
  -> configurationDone
  -> observe stopped event at expected Protos source
  -> continue
  -> guest completion
  -> clean DAP/process shutdown
```

JVM DAP evidence is not Native DAP evidence.

## No-Java / relocation admission gate

The Native distribution proof must exercise an extracted archive with:

```text
JAVA_HOME unset
java absent from PATH
mvn absent from PATH
no Protos repository checkout
no external GraalVM
no network dependency for ordinary execution
```

It must run from an unrelated caller working directory and should repeat the
same exact archive under a second arbitrary extraction path.

The real REPL path must use a PTY/terminal-capable test rather than relying only
on piped stdin because production interactive REPL setup enters JLine's system
terminal path.

## Optimizing-runtime release invariant

Native Image host AOT compilation and Truffle guest runtime compilation remain
separate requirements.

A Native release candidate must preserve the DIST006-C1 forced-JIT invariants
from the packaged/extracted artifact:

```text
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURES=0
COMPILATION_FAILURES=0
HELPER_BYTECODE_ROOT_TIER2>=1
SEMANTIC_BYTECODE_ROOT_TIER2>=1
```

A successful Native executable that runs only in an interpreter/fallback mode
is not release-admissible under PLAT038.

## Platform and artifact identity

Current evidence is insufficient to advertise a Native platform matrix.

The first proof may intentionally select one canonical target, but it must record
the actual artifact identity rather than infer it from container naming:

```text
OS
architecture
libc/ABI assumptions
effective dynamic-library requirements
GraalVM/Native Image version
JDK version
native compiler/toolchain
effective CPU ISA requirement
exact source candidate SHA
archive checksum
```

Multi-platform Native publication is not required for the first bounded proof.

## Release-machinery impact

DIST001 release identity remains valid:

```text
selected V-SNAPSHOT baseline
  -> detached release-only V candidate
  -> one or more validated artifacts from that exact candidate
  -> tag vV / one GitHub prerelease
```

The current release metadata machinery is mechanically singular, however: it
models one `portable_archive`, one checksum and one JVM runtime metadata set.
A future JVM + Native prerelease therefore needs a bounded multi-asset envelope
extension with per-artifact kind/platform/runtime/checksum identity.

This is a release-engineering extension, not a second release identity model.

## Artifact-model findings

```text
JVM_ONLY_TECHNICALLY_VALID=YES
JVM_PLUS_NATIVE_TECHNICALLY_VALID=CONDITIONAL
NATIVE_ONLY_TECHNICALLY_VALID=CONDITIONAL
```

The investigation does not select one of these as the final DIST005 product
architecture.

The current JVM artifact remains the best-proven broad distribution. Native can
satisfy the strict no-external-Java first-run goal if the bounded release proof
closes the remaining tooling/closed-world/platform gaps. Removing the JVM
artifact before those proofs exist would make the release depend on unproven
Native surfaces.

## DIST007 relationship

DIST007 remains independent and unblocked.

Its portable JVM prerelease has its own already-approved scope and explicitly
does not require Native Image or a self-contained claim. DIST005 must not delay
DIST007 for symmetry.

```text
DIST007_BLOCKED_BY_DIST005=NO
```

## Next bounded work

The next executable unit is:

```text
DIST005-B — Native self-contained distribution proof
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
```

It should implement the smallest distribution/validation machinery needed to
assemble a relocatable Native archive and execute the full no-Java admission
matrix on one explicitly identified canonical target.

It must not:

- select the final DIST005 JVM-only/JVM+Native/Native-only product model;
- publish a public release or tag;
- change Protos observable semantics;
- weaken the guest runtime-compilation requirement;
- embed/virtualize the Protos resource tree merely for single-file aesthetics;
- add a second Graal/Truffle dependency authority; or
- block DIST007.

If the proof discovers a fundamental Native incompatibility or a new durable
architecture choice not already governed by PLAT038/PLAT039, stop that affected
path and route it through the appropriate design authority instead of silently
changing architecture.

## Investigation conclusion

```text
DIST005_A_STATUS=READY

PRODUCT_REVISION=d18822e968a1ee6986d832731054b9b90b332b1d
PRODUCT_VERSION=0.3.108-SNAPSHOT
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1

CURRENT_NATIVE_BUILD=PASS
CURRENT_NATIVE_BASIC_GUEST_EXECUTION=PASS
CURRENT_NATIVE_RUNTIME_GUEST_COMPILATION=PASS

NATIVE_RELOCATABLE_ARCHIVE_PLAUSIBLE=YES
NATIVE_REQUIRES_EXTERNAL_JAVA=NO
NATIVE_REQUIRES_EXTERNAL_PROTOS_TREE=YES
NATIVE_ARCHIVE_CAN_CARRY_REQUIRED_PROTOS_TREE=YES

NATIVE_VERSION_SURFACE=PASS
NATIVE_SOURCE_EXECUTION_SURFACE=PARTIAL
NATIVE_REPL_SURFACE=GAP
NATIVE_RUN_SURFACE=GAP
NATIVE_PACKAGE_SURFACE=GAP
NATIVE_TEST_SURFACE=GAP
NATIVE_LSP_SURFACE=GAP
NATIVE_DEBUG_DAP_SURFACE=GAP
NATIVE_OPTIMIZING_TRUFFLE_SURFACE=PASS

CLOSED_WORLD_BLOCKER_PRESENT=UNKNOWN
SEMANTIC_REDESIGN_REQUIRED=NO
PRODUCT_IMPLEMENTATION_CHANGE_REQUIRED=YES
RELEASE_MACHINERY_CHANGE_REQUIRED=YES

MINIMUM_NATIVE_TARGET=UNSELECTED
MULTI_PLATFORM_REQUIRED_FOR_FIRST_NATIVE_PRERELEASE=NO

JVM_ONLY_TECHNICALLY_VALID=YES
JVM_PLUS_NATIVE_TECHNICALLY_VALID=CONDITIONAL
NATIVE_ONLY_TECHNICALLY_VALID=CONDITIONAL

NATIVE_CANDIDATE_ADMISSIBLE=CONDITIONAL
NATIVE_IMPLEMENTATION_PROOF_JUSTIFIED=YES

DURABLE_ARCHITECTURE_DECISION_REQUIRED=YES
PLAT_DECISION_SHOULD_BE_ALLOCATED_NEXT=NO

DIST007_BLOCKED_BY_THIS_WORK=NO

NEXT_ACTION=Implement one bounded DIST005 Native distribution proof slice that assembles a relocatable archive and executes the full no-Java release-admission matrix on one explicitly selected target.
```

## Materially inspected sources

Product repository at the exact revision above:

- `AGENTS.md`;
- `AGENTS.work/COORDINATION.md`;
- `AGENTS.work/IMPLEMENTATION.md`;
- `AGENTS.work/RELEASE.md`;
- `AGENTS.work/DESIGN.md`;
- `pom.xml`;
- `toolchain.json`;
- `build/native/Makefile`;
- `build/native/Dockerfile`;
- `build/native/generate-init-args.sh`;
- `build/native/test-native.sh`;
- `bin/protos`;
- `dist/build_portable.py`;
- `dist/validate_portable.sh`;
- `dist/verify_portable.py`;
- `dist/release_identity.py`;
- `dist/prepare_release_metadata.py`;
- `dist/verify_release_metadata.py`;
- `dist/validate_release_candidate.py`;
- `ProtosCli`, `ProtosPolyglotRuntimeHost`, `UnicodeData17`, and the current
  LSP entry/server/workspace/document-service implementation.

Durable/live project authorities:

- DIST001 release policy;
- DIST003 retained release path;
- DIST005 / #549;
- DIST006 / #733 and DIST006-C1 retained evidence;
- DIST007 / #737;
- PLAT033;
- PLAT038 / #710;
- PLAT039 / #716;
- I069 / #711.

External documentation consulted only for current Native Image/LSP/DAP
constraints:

- GraalVM JDK 25 Native Image reachability/dynamic-feature metadata;
- Native Build Tools Maven plugin 1.1.14 documentation;
- GraalVM Native Image platform/linking/CPU-target documentation;
- GraalVM Truffle embedding/runtime-optimization documentation;
- GraalVM DAP documentation;
- Eclipse LSP4J JSON-RPC documentation.

No external source was treated as Protos semantic authority.
