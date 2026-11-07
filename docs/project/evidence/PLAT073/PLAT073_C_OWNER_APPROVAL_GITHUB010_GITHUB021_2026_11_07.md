# PLAT073-C — Java 25 and FFM GITHUB010 / GITHUB021 evidence

**Date:** 2026-11-07. **Kind:** source-backed architecture research and exact owner-selection record, not executed post-migration tests or a Native security proof. **Protos source:** [`bfb276f9b76c663d3ef1fb68823e13a22ede136c`](https://github.com/guillermomolina/protos/commit/bfb276f9b76c663d3ef1fb68823e13a22ede136c). **Historical PLAT072 project record:** [`8c878bdbf787694fd94a59855d78d0cd5a704c84`](https://github.com/guillermomolina/protos-project-docs/commit/8c878bdbf787694fd94a59855d78d0cd5a704c84). **Decision:** [PLAT073-C ratification](../../decisions/platform/PLAT073_JAVA_25_FFM_AMENDMENT_TO_PLAT072_E.md).

## Approval provenance

The comparative PLAT073 investigation recommended **Candidate C — Java 25 classfile/API baseline and private FFM-only Linux confined-Filesystem bridge, with T01–T16 and actual JVM/Native validation mandatory**. The project owner responded **"aprobado"** in the immediate follow-up on 2026-11-07. Approval is specific to this candidate, not to any future hybrid JNI fallback or weakened containment. Owner also reported `git diff --check` clean and `make test`/`make check` PASS locally on pre-migration work. No new Java 25 or FFM tests were run or independently validated during this decision publication.

## Audited Protos baseline and compatibility

- [`toolchain.json`](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/toolchain.json) sets Java bytecode release 21; pinned GraalVM Community 25.5.5.0.3, JDK 25.0.5.0.3 and Maven minimum 3.9.9. [`pom.xml`](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/pom.xml) explicitly compiles `--release 21`, owns Native profiles and a Graal/Truffle 25.5.5.0.3 dependency set. This mismatch was intentional in [DIST002](https://github.com/guillermomolina/protos/blob/fa001f4f4b15372faf6d808bb6af4ae159747e29/docs/project/DIST002_TOOLCHAIN_ALIGNMENT.md), not a Truffle-imposed Java 21 limitation.
- [`tools/verify_toolchain.py`](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/tools/verify_toolchain.py), [toolchain tests](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/tools/tests/test_verify_toolchain.py), [`bin/protos`](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/bin/protos), [Native image initialization](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/scripts/native-generate-init-args.sh), CI, Dockerfile, distribution builder and manifest need cross-consumer validation.
- [External provider test](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/src/test/java/com/guillermomolina/protos/execution/foreign/api/ProtosExternalForeignProviderTest.java) and [provider HOWTO](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/docs/guide/interop/custom-provider.md) intentionally invoke `javac --release 21`: this should remain an ABI-compatibility test when feasible. Old Java 21 provider bytecode (classfile major 65) may load into pinned Java 25 (classfile major 69), but Protos compiled as classfile 69 cannot run on Java 21. Actual SPI linkage must be verified.
- [Existing FFM libc provider fixture](https://github.com/guillermomolina/protos/blob/bfb276f9b76c663d3ef1fb68823e13a22ede136c/protos/tests/foreign-provider/plugins/libc/src/i085b/libc/LibcProvider.java) uses `java.lang.foreign` for `strlen` on the JVM. This is not an `openat2`/Landlock/Native/security test.
- Current Native release: Linux x86_64, glibc >=2.39 dynamically linked; Native PLAT045 interpreter-only limitation remains.

## Prior art, concrete source and non-equivalence

| System | Investigated mechanism | Application to Protos |
|---|---|---|
| OpenJDK | [JEP 454](https://openjdk.org/jeps/454): FFM finalized JDK 22 | Available on selected Java 25; not a containment mechanism |
| GraalVM Native Image | [Official FFM support](https://github.com/oracle/graal/blob/master/docs/reference-manual/native-image/FFM-API.md): GraalVM 25+, Linux/x64 and Linux/AArch64; `foreign.downcalls` reachability metadata, `captureCallState`, `firstVariadicArg`, native access enabled; default lookup unsupported for fully static binary | Makes FFM feasible but does not prove this Protos binary or syscall |
| GraalPy | [NativeAccessSupport](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/nativeaccess/NativeAccessSupport.java), [NativeContext](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/nativeaccess/NativeContext.java): JDK 22+ FFM, separate unavailable JDK21 branch, cached handles, thread-local call state | Real Truffle prior art, and evidence that retaining Java 21 has ongoing maintenance cost |
| TruffleRuby | [FFMSupport](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/platform/FFMSupport.java), [TrufflePosixNodes](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/extra/TrufflePosixNodes.java), [40.0.0 changelog](https://github.com/truffleruby/truffleruby/blob/master/CHANGELOG.md): raised embedding Java minimum 21→25 and migrated POSIX calls to FFM, reported ~100ns→~12ns in its own benchmark | Direct positive precedent; **not** a Protos performance claim |
| Espresso | [JNI NativeEnv](https://github.com/oracle/graal/blob/master/espresso/src/com.oracle.truffle.espresso/src/com/oracle/truffle/espresso/jni/NativeEnv.java) maintains Java/JNI compatibility | Broad JVM guest JNI needs differ from narrow Linux backend |
| GraalJS | [GraalJS](https://github.com/oracle/graaljs) host/polyglot file-provider integration | Native host calls must not bypass an embedding Context provider |
| Sulong / LLVM | [GraalVM LLVM](https://www.graalvm.org/latest/reference-manual/llvm/) interop substrate | Overengineering for a few privileged Linux kernel operations |
| SimpleLanguage | [SimpleLanguage](https://github.com/graalvm/simplelanguage) minimal Truffle language | No comparable host Filesystem security architecture |
| Apple Pkl | [Pkl threat model](https://pkl-lang.org/threat-model.html) permission-bounded evaluation | Authority precedent, no proof of recursive writable Linux races |
| Linux, BSD, Darwin, Windows | [openat2](https://man7.org/linux/man-pages/man2/openat2.2.html), [landlock](https://man7.org/linux/man-pages/man7/landlock.7.html), [rename](https://man7.org/linux/man-pages/man2/rename.2.html), [unlink](https://man7.org/linux/man-pages/man2/unlink.2.html); OS-specific alternate mechanisms | No false claim that a cross-OS Java API makes Linux confinement portable |

## Candidate set and explicit elimination

- **A Java 21 + JNI/C:** exact previous approved bridge; no migration, old bytecode and two-language maintenance. Alternative if change deferred; not selected.
- **B Java 25 + JNI/C:** conservative source-floor modernization with existing C ABI and packaging obligations. **Strongest challenge to C** if FFM Native ABI fails; not silently selected as fallback.
- **C Java 25 + FFM-only private Linux bridge: SELECTED**, with proof gates.
- **D Java 25 + hybrid FFM/JNI:** duplicates ABIs, lifecycle, build and proof machinery with no demonstrated necessary split; overengineering.
- **E Java 21 multi-release jar + Java 25 FFM classes:** multiple implementations for a Java 21 runtime compatibility promise not actually supported today; reject.
- **F Java 22 interim baseline:** no benefit over pinned runtime Java 25; reject.
- **G defer/keep prior contract:** can avoid migration risk, but preserves unneeded API ceiling and C-shim maintenance; not preferred.

## GITHUB010: 12 required scores (1–5, confidence H/M/L)

Higher score is better; H/M/L is high/medium/low confidence. A design score never establishes an implementation security PASS.

| Axis | A: 21/JNI | B: 25/JNI | C: 25/FFM | D: 25/hybrid |
|---|---|---|---|---|
| 1 Correctness/invariants | 3L JNI mature; T04/T05 open | 3L same security | 3L ABI and security unproven | 3L two ABIs to prove |
| 2 Protos alignment | 3M legacy floor | 4M aligned JDK | **5M** minimal private bridge | 2M duplicated mechanisms |
| 3 Pay for present need | 3M old ceiling | 4M modern floor | **5M** no own C build | 1H two paths |
| 4 Incremental growth | 3M C extension | 4M extend JNI | **5M** add narrow downcalls | 4M two extensible paths |
| 5 Future options | 3M FFM later | 3M C replaceable | 4M backend replaceable | 4M optionality at a cost |
| 6 Scalability | 3L grant workers pending | 3L same workers | 3L same workers | 2L extra native state |
| 7 Simplicity | 3H C/Java bridge | 3H C/Java bridge | **4M** one Java bridge | 1H dual ABI |
| 8 Portability | 3M old classfile, Linux backend | 2M JDK25/Linux | 3M FFM API, Linux ABI | 2M dual deployments |
| 9 Runtime cost | 3L JNI not measured | 3L JNI not measured | 4L optimized handle potential | 2L extra layer |
| 10 Failure/operability | 4M C errno familiar | 4M C errno familiar | 3L arena/errno/metadata pending | 2M two failure surfaces |
| 11 Deferral/reversibility | 4M no migration | 3M JNI later rewrite | 4M private backend swap | 3M difficult simplification |
| 12 Evidence/risk | 4M mature JNI; kernel unproved | 4M mature JNI | 3M production precedents, no Protos Native proof | 3M compound proof |

**Hard gates, not compensating scores:** none passes PLAT072 T04/T05 yet; D is overengineering and keeping Java 21 imposes an unproven compatibility maintenance cost. A/B remain credible if C fails, but approval must be reopened.

## GITHUB021 invariant-by-invariant preservation

| Previously ratified constraint | C assessment |
|---|---|
| D248-B exactly one CLI grant, bootstrap-local RootActor authority; no Actor/Tool/Network/Resolver implicit grants | **PRESERVE** |
| Exact root FD pin, no pathname/mountinfo TOCTOU, four `openat2 RESOLVE_*` flags, bind mount resistance | **PRESERVE** |
| Grant-specific Landlock including `REFER`, `TRUNCATE`, worker confinement | **PRESERVE** |
| Atomic `renameat2`/`unlinkat`; File cursor/resource stability; `entries` single-observation name/kind; cancellation/commit | **PRESERVE** |
| Effective Context `Truffle FileSystem`, Process lifetime/revocation and no bypass | **PRESERVE** |
| T01–T16 with T04/T05 deterministic moved-parent proof, fail closed unsupported hosts | **PRESERVE — hard gate** |
| Java 21 bytecode + narrow JNI/C | **EXPLICITLY REOPENED and replaced** by Java 25 + narrow FFM-only, owner "aprobado" |
| No language/spec API change, no general OS sandbox or generic FFI | **PRESERVE** |

## Exact falsification and acceptance plan

**DIST022 (toolchain/distribution):** `toolchain.json`, `pom.xml`, verifier/test fixtures, provider SPI built `--release 21` loading on actual pinned Java 25, exact CI/devcontainer, portable runtime launcher, source/classfile major 69, full `make test` and `make check` on migrated code, rebuilt installed Native executable with release manifest/dependency identity. Original baseline green tests do **not** satisfy these gates.

**I156 (FFM ABI):** supported glibc symbols and libc `syscall(long,...)`, `Linker.Option.firstVariadicArg`, actual per-architecture syscall number and `open_how` layout, `Linker.Option.captureCallState("errno")` and thread-isolated state, Landlock ABI capabilities, confined worker ownership, native syscall signatures, `foreign.downcalls` reachability metadata and native-access policy in both JVM and Native; prevent descriptor reuse/use-after-close/cancel faults. No Protos-owned C/JNI bridge.

**I156 (PLAT072 adversarial suite):** T01 root path ancestor swap, T02 mount/bind crossing, T03 intermediate/final symlinks; **T04/T05 controlled parent FD moved outside authority before `unlinkat`/`renameat2`, absolutely no out-of-root effect**; T06/T07 createNew/create races; T08 truncate/cancel/commit; T09/T10 entry mutation and symlinks; T11 coherent `entries` name/kind including `DT_UNKNOWN`; T12 stable File; T13 unsupported ABI/LSM/provider/host fail closed **before guest**; T14 multiple Processes/Contexts/Actors no bleed; T15 service/termination/in-flight cancellation; T16 actual installed JVM/Native persistence/security/identity. Include `EAGAIN`, `EINTR`, `ENOSYS`, `EACCES`, `EXDEV`, `ELOOP`, `ENOENT`, `EEXIST`, FD reuse, nonrepresentable names and no unconfined fallback.

**If T04/T05 fails:** stop/reopen a *separate* confinement decision (broker/namespace or safe refusal); JNI cannot rescue it. **If FFM Native/ABI fails:** stop/reopen ABI choice for Java 25 + JNI; never silently add a hybrid. Native Image `Linker.defaultLookup` not available on fully static builds, which are not currently selected.

## Incrementality, stress, deferral, strongest objection

Smallest sufficient: one Java 25 implementation floor, one narrowly scoped Linux FFM adapter activated only by an explicit grant, otherwise no worker/arena/kernel-cost for an ungranted program. Preserve an extensible backend seam without prebuilding generic guest FFI, child-Process policy, extra OS mechanisms or multi-release jars. Future many-Context density/alternative schedulers may justify a broker and worker ownership redesign; alternate platforms need their own security proof, not merely a recompile. Adding JNI later rewrites the private host adapter and packaging, **not** language semantics or File/Path identity. Adding fully static Native or new architectures later needs ABI and release-specific tests.

**Strongest counterargument:** C exposes Linux struct, constant and syscall ABI maintenance in Java plus FFM Native reachability constraints while JNI/C can compile against Linux headers. I156 also has a difficult independent T04/T05 security proof. The conservative Java 25+JNI option B minimizes novelty. This risk is addressed by strict ABI/JVM/Native proof gates and a **mandatory new decision**, not a silent fallback, if C is not viable.

**Audit and nonclaims:** Sources materially inspected include `AGENTS.md`, `AGENTS.work/DESIGN.md`, `REFERENCE.md`, `COORDINATION.md`, `IMPLEMENTATION.md`, normative IO sections, `toolchain.json`, `pom.xml`, verifier/tests, `Makefile`, `bin/protos`, Native arguments, Dockerfile/CI, provider fixture/test/docs and PLAT072/DIST002 records. External source comparisons cover GraalPy, TruffleRuby, GraalVM FFM, Espresso, GraalJS, Sulong, SimpleLanguage, Pkl, OpenJDK, Linux and non-Linux OS limits. No uncommitted I156 patch, new Native image, Java 25 migrated tests or controlled T04/T05 were executed or certified by this documentation publication.
