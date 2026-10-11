# AUD021-B/C — Candidate C owner approval and Path/Filesystem implementation boundary

**Date:** 2026-10-11  
**Status:** OWNER-APPROVED architectural implementation direction; non-normative, research/decision evidence; implementation not yet validated.  
**Formal audit owner:** [AUD021/#874](https://github.com/guillermomolina/protos/issues/874).  
**Implementation owner:** [I094/#875](https://github.com/guillermomolina/protos/issues/875).  
**Product revision inspected:** [guillermomolina/protos@b47be4b698ca27b537898abee89dd6f4c8c49fc3](https://github.com/guillermomolina/protos/commit/b47be4b698ca27b537898abee89dd6f4c8c49fc3).  
**Prepublication docs revision:** [guillermomolina/protos-project-docs@b614a0f6d394d52e24ccca010d4af1fa99593bd8](https://github.com/guillermomolina/protos-project-docs/commit/b614a0f6d394d52e24ccca010d4af1fa99593bd8).  
**Previous snapshot:** [AUD021-A](https://github.com/guillermomolina/protos-project-docs/blob/b614a0f6d394d52e24ccca010d4af1fa99593bd8/docs/project/evidence/AUD021/AUD021_A_PROTOS_FIRST_PATH_BOUNDARY_RESEARCH_CHECKPOINT_2026_10_11.md).

## 1. Exact approval provenance and change from initial hypothesis

On 2026-10-11, the project owner explicitly said **"ok pues apruebo C"**, referring to the exact Candidate C recap after comparing implementations in other languages. This is explicit selection of **Candidate C for implementation**, replacing the earlier approval of Candidate B strictly as a research hypothesis, not as a ratified design. The follow-up owner instruction authorizes GitHub Issue coordination, a durable report in guillermomolina/protos-project-docs, and a subsequent autonomous implementation prompt with separate agent and human-executor responsibilities. Product source changes, tests and product Git publication were NOT authorized for agent direct execution.

The approved C is a deliberate **representation versus semantics** split:
- Physical representation: keep a minimal private authentic immutable Java ProtosPathValue storing an ordered sequence of host Strings. No requirement to instantiate a private guest Array for every Path.
- Guest behavior: implement Path.relative, path.child, structural equality and hash meaningfully as source closures in Path.protos, retaining only narrowly justified intrinsic mint/extract/verify primitives. A source shim that delegates whole behavior back to Java does not meet C.
- Host boundary: snapshot exactly ordered validated Path component names in an inert immutable PathComponents descriptor, without any guest callback. Filesystem flows/backends consume that descriptor, never guest prototypes, Path values, Prelude or Closures.
- Actors/P: exact rematerialization on actual transfer, never an automatic Actor/Task/Process dependency of local Path.
- Security/FS: capability authority, NIO/TruffleFile access, confinement at resource acquisition, asynchronous lifecycle and File custody remain native host responsibilities.

The owner explicitly rejected the general premise that additional Path allocations alone veto a Protos-first design. Choosing C after comparison is an architectural judgment, **not** a benchmark result or an automatic global rule that performance outranks semantic ownership.

## 2. Verified source and normative baseline

At the pinned product revision:
- [Path.protos](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/protos/lib/core/Path.protos) is only an empty Path prototype.
- [ProtosStandardPathProtocol.java](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/execution/ProtosStandardPathProtocol.java) installs native relative, child, equality and hash Closures; the relative factory currently allows an ordinary receiver delegating up to Path, whereas child/equality/hash require a genuine carrier. Preserve this observable distinction unless a separate semantic approval occurs.
- [ProtosPathValue.java](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/runtime/ProtosPathValue.java) is a final represented-value carrier with a prototype and a defensive immutable List<String>. Its child, structurallyEquals and structuralHash currently implement guest policy in Java. Existing structuralHash delegates to Java List.hashCode.
- [ProtosStandardStringProtocol.java](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/execution/ProtosStandardStringProtocol.java) provides String scalar access; [ProtosStandardHashSupport.java](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/execution/ProtosStandardHashSupport.java) defines semantic String.hash using Java String.hashCode. Source implementing Path.hash must keep hash coherence and avoid accidental arbitrary-precision polynomial growth or accidental change of observed hash values if preserving those is required.
- [IpAddress.protos](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/protos/lib/core/IpAddress.protos) and [ProtosStandardIpAddressProtocol.java](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/execution/ProtosStandardIpAddressProtocol.java) provide a concrete source-closure/operator-rename precedent. Unlike mutable, shape-forgeable guest objects, Path's host-authenticity check must reject fabricated instances and not depend on guest sends.
- [ProtosActorValueTransfer.java](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java) remembers a dedicated Path copy with destination Prelude; [ProtosParallelRuntime.java](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java) has a Path transfer branch returning a fresh copy without apparent memoization. The latter is a **potential** shared-reference aliasing defect to validate and correct if confirmed, not a proven test failure.
- Existing host flow/backend seam: ProtosStandardFilesystemProtocol; ProtosFilesystemOpenFlow; ProtosFilesystemNamespaceMutationFlow; ProtosFilesystemTreeObservationFlow; ProtosFilesystemOpenOptions; ProtosEmbeddedFilesystemCustody; ProtosNioConfinedFilesystemBackend; ProtosNioReadOnlyFilesystemBackend; ProtosNioReadOnlyTreeFilesystemBackend; ProtosNioCapturedTreeFilesystemBackend. They currently know ProtosPathValue or its components and need case-by-case decoupling; no security layer can be removed merely because a new descriptor exists.

**Normative authority unchanged:** [spec/io/FILESYSTEM.md](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/spec/io/FILESYSTEM.md), plus spec/io/IO_CORE.md, BYTE_IO.md, PROCESS_IO.md; spec/semantics/OBJECT_MODEL.md and VALUES_AND_COLLECTIONS.md; spec/concurrency/ACTORS.md and PARALLEL_EXECUTION.md. D169/D170 and PLAT051 remain their respective existing authorities. Where historical Issue wording differs from current spec, revalidate and do not claim a new ratification from this document.

## 3. Comparative language evidence, interpretation and limits

The researched examples establish **separability**, not portability of native path string syntax:

| Language | Observed representation/library arrangement | Relevant lesson | What Protos must NOT copy |
| --- | --- | --- | --- |
| Python pathlib PurePath | Python source class stores raw path strings and lazily derives normalized parts, formatted string and hash as needed | Source-level value semantics need not eagerly allocate guest component objects | Python's native POSIX/Windows parser, drive/root, current-directory and path separators |
| Ruby Pathname | Ruby standard-library object wraps a path string and delegates I/O to File/Dir facilities | Small carrier plus language-level methods is viable | Ruby pathname normalization and ambient host path authority |
| Rust Path/PathBuf | Standard types are thin wrappers around OsStr/OsString; components are parsed/iterated according to platform rules | Representation can be compact while rich operations live in its library | Host-specific separators, roots, upward traversal and platform syntax |
| Elixir Path / Erlang filename / Go filepath | Mostly functional APIs manipulating string/list/binary path data | A path manipulation API need not instantiate Actors/Processes, and representation is independent of I/O scheduling | Treating an arbitrary string as an authorized host pathname |

Primary pointers: [CPython pathlib source](https://github.com/python/cpython/blob/main/Lib/pathlib/__init__.py), [Python pathlib reference](https://docs.python.org/3.14/library/pathlib.html), [Rust std::path](https://doc.rust-lang.org/std/path/), [Ruby pathname](https://docs.ruby-lang.org/en/master/Pathname.html), [Elixir Path](https://hexdocs.pm/elixir/Path.html), [Erlang filename](https://www.erlang.org/doc/apps/stdlib/filename.html), [Go filepath](https://pkg.go.dev/path/filepath).

The examples do NOT prove a speedup for C or justify a Protos String-to-Path coercion. Under Protos, child("a/b") appends one semantic component, not two; the backend may reject a name it cannot represent atomically. Path has no authority; Filesystem holds namespace authority. Empty, ".", and ".." component names are not accepted.

## 4. Candidate disposition and necessity classification

- **C — APPROVED:** compact Java immutable payload + real source-backed Path behavior + inert host-only components descriptor. Better aligned with the owner's final simplicity/pay-as-you-grow and Protos-first balance, without allocating a dedicated Protos Array per Path.
- **B — NOT SELECTED:** Protos private Array for component storage plus minimal authenticated carrier. Remains an evaluated alternative, not required future scaffolding or silently approved implementation.
- **A — NOT SELECTED AS TARGET:** keep current native guest semantics and pass guest Path all the way to backend. Useful baseline only.
- **D — NOT SELECTED AS TARGET:** inert host descriptor only while leaving all guest Path semantics in native Java. Useful fallback diagnostic, not the approved end state.

| Mechanism | Result |
| --- | --- |
| Authority-free immutable Path; exactly ordered normal components; structural == and hash | KEEP |
| Filesystem authority, confined resource acquisition, backend security, File custody, Future/commit/cancel | KEEP |
| Compact genuine Java carrier, private immutable components, destination-local prototype | KEEP, MINIMIZE |
| Source-expressible native Java Path semantics | KEEP BEHAVIOR, MOVE RESPONSIBILITY TO PROTOS |
| Backend interfaces accepting guest Path/prototypes | REMOVE COUPLING NOW, replace with inert PathComponents |
| Generic std transfer registry, per-Path actor/scheduler/worker, fake open shape recognition | NOT JUSTIFIED; do not introduce |
| Guest Array per Path, public Path components/recognizer/parser/upward/root API | NOT PART OF APPROVED C; do not introduce |

## 5. Architecture and security contract

**Guest side:** Path.protos must own factory, appending, logical comparison and hashing as actual source closures. Minimal Java intrinsics may unforgeably mint an empty Path/append a validated String, report count and return immutable component Strings; any equivalent minimal bridge must remain private and fail on forged receivers. A native helper implementing the entire equality/hash/child algorithm invalidates the claimed guest ownership. Source operators may require bootstrap aliasing as in IpAddress; inspect actual source syntax. Do not make provider callbacks or closure dispatch part of host argument decoding.

**Boundary:** at each Filesystem invocation, validate the canonical ProtosPathValue and extract a complete inert immutable PathComponents descriptor with defensive, independent value ownership. For replace, capture both source and target before async work. Read open options exactly once and preflight invalid combinations before target/backend effects. Backends receive no ProtosObjectValue, Closure, Prelude, Path prototype or execution context, regardless of provider.

**Host:** retain TruffleFile provider confinement, secure NIO SecureDirectoryStream acquisition, no-follow semantics, read-only/read-write variants, captured immutable trees, race-safe namespace operations, File cursor/commit/cancellation, resource release. Lexical checks alone do not prove confinement if native backend follows links or races. Backends may reject unrepresentable opaque component names; never translate an embedded separator into a structure change.

**Actor/P:** a local Path is a plain lightweight immutable value; creating/manipulating it does not construct RootActor, RootTask or RootProcess. On actual transfer, rematerialize components under target Prelude. Repeated occurrences of the *same original Path reference* in a graph preserve destination alias identity, whereas distinct but equal Paths remain distinct. Filesystem/File capabilities are never implicitly sent with a Path. Context and true Process rematerialization are separate boundaries; do not claim implementation support that has not been established.

**Async:** host callbacks yield inert completed data; guest Arrays, Files and entry descriptors are materialized via the established owning Actor/Context. No new global registry, waiting worker, permanent synchronization or unneeded Task solely for this new value boundary.

## 6. Concrete acceptance and residual risks

1. **Identity / reflection:** Path.relative() and delegated factory receiver behavior, source method ownership, normal arity/error, genuine carrier recognition, frozen prototype, no guest-visible implementation slots, ordinary parent() reflection, == vs !==/===, Map hash.
2. **Components:** empty and nested, invalid scalar/name/domain, slash/backslash/colon and drive/UNC as data, equality independent of host casing/normalization and independent of the particular Filesystem.
3. **Hash:** Path's preexisting Java List.hashCode uses signed 32-bit 31-fold hashing of String.hashCode, while Protos Integer arithmetic may be unbounded; a source implementation must explicitly decide how to preserve existing observable hash outputs and avoid large-Integer growth, without introducing unrelated guest or host work. Match map-key behavior.
4. **Transfer:** Actor and P destination prototype, no capability/Closure transfer; memo identity of duplicated references under nested collections and separate identities for two equal distinct Paths; no eager transfer when not used.
5. **Host:** open/replace/remove/entries/captured-tree and each concrete backend; exact one-time argument snapshot; options preflight and errors, secure confinement at actual acquisition, symlink and TOCTOU resistance, no reinterpreted names, atomic File selection, async cancel/late-release, no callback on poller and guest materialization on owner.
6. **Pay-as-you-grow:** Path-absent/no extra infrastructure; local Path/no actor scheduler; I/O call/one descriptor; actual transfer/on-demand rematerialization. No performance benchmarks were executed; measure hot construction/equality/hash with human if there is concrete regression evidence.
7. **Design gate:** if normal representation forces new public API, observable reflection/identity shift, D169 semantics amendment, generalized value transfer or weakened backend guarantees, STOP and request exact owner D/PLAT approval rather than inventing behavior.

## 7. Execution boundary and next work

The next slice is **I094-A — IMPLEMENTACIÓN** in **guillermomolina/protos**, one grouped implementation across Path source, minimum runtime carrier/bridge, snapshots/backends and Actor/P paths with tests. [I094/#875](https://github.com/guillermomolina/protos/issues/875) owns the complete agent/human-executor contract. The agent must treat the local repo as at actual HEAD, examine current source/spec and relevant tests before editing, preserve concurrent work and never execute validation/build/tests/Git state changes. The human executes the final focused/static and integrated make test check; only after green modifies pom.xml and CHANGELOG.md exactly once just before publication, with no tests after moving those files. The product commit precedes any future revision-coupled implementation evidence.

**Validation performed for this document:** external public references and GitHub source/spec inspection only. No source edit, compiler, test, benchmark or project execution was performed; the suspected P transfer alias bug is not claimed as reproduced. This record provides architectural approval provenance and an executable implementation boundary, not I094 completion.

**Issue/native-parent caveat:** GitHub Issue tooling in this interaction did not expose a native sub-issue linking mutation. Until the exact native AUD021 -> I094 relationship is verified or repaired, hierarchy postcondition is pending and prose links are not a substitute.
