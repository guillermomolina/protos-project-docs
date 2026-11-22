# PLAT077 — Candidate A research, GITHUB021 consistency and owner approval (2026-11-22)

**Kind:** Durable, non-normative design/evidence record; AI-assisted drafting. **State:** Exact design selected and ratified; product implementation not yet produced.
**Decision:** [PLAT077 #1146](https://github.com/guillermomolina/protos/issues/1146); [canonical approved architecture](../../decisions/platform/PLAT077_GENERIC_NATIVE_IMAGE_FFM_PROVIDER_A_RATIFIED_2026_11_22.md). **Executable implementation:** [I170 #1148](https://github.com/guillermomolina/protos/issues/1148).
**Exact inspected product revision:** [`guillermomolina/protos@1fa0c41728792596a1e4862f75298a2532b56bb1`](https://github.com/guillermomolina/protos/commit/1fa0c41728792596a1e4862f75298a2532b56bb1).
**Approval provenance:** Project owner, 2026-11-22 conversation, following a presented 5-candidate/12-criterion GITHUB010 comparison and explicit recommendation A: **"ok apruebo la sugerencia"**. The request authorized this documentation's commit/push. The owner's separate statements **"el git diff check esta limpio"** and **"todos los tests pasan en local"** were not accompanied by a run log or a new I170 product diff; preserve only as **owner-reported pre-implementation baseline**, not FFM-in-Native validation.

## Decision scope and owner constraint history (GITHUB021)

The 2026-11-22 owner corrections materially constrain the decision, in this sequence:

1. The goal is installed Native Protos execution of `protos --allow-library sqlite3=libsqlite3.so.0 app.protos`, without separate provider JAR/JDK/GraalVM on the consumer.
2. The same executable must also run `protos app.protos` without acquiring FFI resources or loading third-party libraries in the no-FFI case. Excluding *all compiled FFM implementation* was initially expressed as a preference, not a demonstrated feasibility condition.
3. The owner expressly refined this: **do not discard build-time inclusion**, because a dynamically injected FFM module might be impossible or disproportionately costly. Allow **minimum generic FFM infrastructure** to be compiled into Protos if necessary; do **not include the external library** itself (SQLite or another native C library), and do not integrate speculative per-library adapters.
4. The presented recommendation selected **A** (existing generic FFM provider included at Native build time; explicit lazy host authorization and library acquisition; finite documented Native ABI shape coverage; ordinary Protos FFI API and JVM path unchanged). The owner expressly approved that recommendation.

Consistency audit: (1) and (2) are satisfied by a single Native executable with provider registration only on authorized native hosting and library lookup delayed until `ffi:` acquisition; (3) is satisfied by existing provider sources and build metadata but **no SQLite shared object or Java bridge**; (4) is the architecture now ratified. No earlier statement is silently interpreted as prohibiting generic provider inclusion, and no owner-approved D259 or D256 invariant is replaced.

New consequence disclosed **before approval:** a finite Native ABI shape envelope may be narrower than JVM's dynamically composable signature set and requires rebuild to expand. The decision does **not** approve a new public `std:ffi` descriptor limit or an alternative error prototype. Unsupported Native shapes must fail closed through the existing admission/error semantics before native entry. If that requires an observable semantic amendment, I170 must stop for a separate owner decision; no speculative compatibility claim is authorized.

## Direct Protos source and normative floor

| Inspected source | Concrete finding / constraint |
| --- | --- |
| `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/COORDINATION.md` | Owner design gate, GITHUB010 twelve criteria, GITHUB021 invariant comparison, human-executor mode, durable role-first evidence, I-family scope and no cross-repository product publication |
| `spec/semantics/FFI.md` | Ordinary Protos `std:ffi`, exact ABI descriptors, default deny, no authority from import, `TRUSTED_IN_PROCESS`, pre-entry admission, Actor-local ownership |
| `spec/semantics/SQLITE.md` | Ordinary Protos SQLite declarations/policy, no Java SQLite backend, synchronous operations, durable lifecycle and copied results |
| `protos/lib/ffi/module.protos`; `protos/lib/sqlite/abi.protos`; `protos/lib/sqlite/internal/Borrowed.protos` | Runtime-composed physical signature shapes, SQLite TEXT/BLOB private input descriptors, borrowed pointer copying, owned destructor |
| `providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmForeignProvider.java` | Existing `ProtosForeignProviderPlugin`; construction does not load library/create linker/session; identity `protos-ffm`, route `ffi:` |
| `providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmSession.java` | `acquireModule`: authorize **before** `Linker.nativeLinker()`, `Arena.ofShared()`, `SymbolLookup.libraryLookup()`; Actor-local lifetime |
| `providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmBinding.java` | `Signature.descriptor`: runtime `FunctionDescriptor` construction; constructor uses `linker.downcallHandle(address, descriptor)` then `asType`/`asSpreader`; actual Native support must be proved |
| `providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmTypes.java` | Physical layouts `ValueLayout.JAVA_INT`, `JAVA_LONG`, `JAVA_DOUBLE`, `ADDRESS`. The C ABI `i32` does **not** introduce a Protos guest `int` primitive |
| `src/main/java/com/guillermomolina/protos/execution/foreign/api/ProtosForeignPluginProvider.java`; `.../providers/ProtosExternalProviderPluginLoader.java` | Reuse `trustedDescriptor` and existing adapter; external JVM provider loader uses explicit paths and a URLClassLoader, not generic classpath discovery |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosPolyglotRuntimeHost.java`; `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java` | Empty provider paths currently force an empty registry even if `--allow-library` was given. Both ordinary and debug runtime-host paths need Native built-in registration |
| `pom.xml`, `scripts/native-generate-init-args.sh`, `src/main/resources/META-INF/native-image/com.guillermomolina/protos/reachability-metadata.json` | Existing Native FFM reachability `foreign.downcalls` for Filesystem; additive metadata and Native-only build staging can reuse it |
| `dist/build_ffm_provider.py`, `dist/build_native.py`, `dist/validate_native.py`, `dist/install.sh` | Portable provider remains a separate JAR; installed Native profile has fixed runtime/glibc/ISA/release assumptions; do not silently change them |
| `src/test/java/com/guillermomolina/protos/execution/foreign/api/ProtosI144FfmFixtureAbiTest.java`; `protos/tests/foreign-provider/ffm-public-api.protos`; `protos/tests/foreign-provider/ffm-abi.protos` | Existing independent C fixture covers C primitives, strings, outs, borrowed/owned, isolation and failure paths; tests are *not* Native Image acceptance yet |

Preceding D238/I085/I144/D259/I169/D256/LIB031 decisions are already published. This platform selection extends Native availability **without reopening their semantics**. The normative `spec/` files remain unchanged by the ratification.

## Physical ABI inventory and Native AOT barrier

Every `FfmBinding.Signature` can be reduced to four parameter layouts: `I` (`jint`), `L` (`jlong`), `D` (`jdouble`), `P` (`void*`). Returns are `void`, `I`, `L`, `D`, or `P`. Outputs are pointer positions; borrowed/owned/nullable references, `utf8z`, private TEXT/BLOB input and sentinel copy-destructor all reduce to `P`, while their Protos admission/lifetime behavior remains unchanged.

Exact distinct *physical* forms used by `protos/lib/sqlite/abi.protos` plus its `sqlite/internal/Borrowed.protos`, simplified by carrier:

```text
I()            I(P)             I(P,I)         I(P,P)
I(P,I,L)       I(P,I,D)         L(P,I)         D(P,I)
I(P,P,I,P)     I(P,P,I,P,P)     I(P,I,P,I,P)   P(P,I)
```

These **12** shapes cover the declared SQLite functions and destructors, not the full possible generic R1 signature universe. A second independent library can use `D(D,D)` (for example the C `hypot` function where available); a fixture can independently verify provider portability. The chosen Native envelope must be defined and validated by the actual source/tests, not by hard-coding symbol names or taking this inventory as a perpetual closed table.

Unbounded independent choice of four argument carrier types creates `(4^(N+1)-1)/3` shapes up to arity N *per fixed return type*. For five return forms, max arity 5 means 6,825 potential downcalls; arity 8 means 436,905; neither is justified by present need. The current public R1 signature composition has no comparable published maximum. Native Image requires build-time reachability registration for the actual physical descriptors; absence may raise `MissingForeignRegistrationError` at runtime. No claim that a dynamic FFM descriptor is automatically supported can be made without testing the exact Native build.

Native Image uses `foreign.downcalls` with `returnType`, `parameterTypes`, and optional linker `options`. Match `FfmTypes.layout` **exactly**; the existing Filesystem entries (including capture-call-state and variadic metadata) must remain intact. A missing native shape is a **pre-entry failure**, not reason to call with an approximate ABI or try another execution backend. New native pointers/arenas/handles cannot enter image-heap snapshots.

## Verified comparable implementation precedents

| Implementation | Exact implementation method / mechanism | Applicable behavior and limits |
| --- | --- | --- |
| [GraalPy `GenerateNativeDowncallsProcessor`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python.processor/src/com/oracle/graal/python/processor/GenerateNativeDowncallsProcessor.java) | `generateInvoker`, `collectDowncalls`, generated `ensureLibrary()` | Per-signature call preparation and library acquisition separately; the compiler knows a concrete finite set |
| [GraalPy `NativeDowncallMethodHandleGenerator`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python.processor/src/com/oracle/graal/python/processor/NativeDowncallMethodHandleGenerator.java) | `emitMethodHandleField`, generated `NativeAccessSupport.createDowncallHandle` | Physical signature-to-handle mapping; **not** a mandate to create per-SQLite generated Protos Java classes |
| [GraalPy `NativeAccessSupport`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/nativeaccess/NativeAccessSupport.java) | `createImpl` uses `ImageInfo.inImageCode()` to select Native-specific host behavior | Verified precedent for an explicit Native-vs-JVM boundary without a new Protos interpreter |
| [TruffleRuby `FFMSupport`](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/platform/FFMSupport.java) | `functionDescriptor`, `createDowncallHandle`, `@TruffleBoundary` | Carrier signatures mapped to FFM descriptors, independently of loaded library identity |
| [TruffleRuby `NativeLibrary`](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/platform/NativeLibrary.java) | `open`, `lookupSymbol` and lazy `ensureFunctionsResolved` | Library opening/symbol resolution separated from signature stubs; native addresses resolved at runtime |
| [Espresso `NFINativeAccess`](https://github.com/oracle/graal/blob/master/espresso/src/com.oracle.truffle.espresso/src/com/oracle/truffle/espresso/ffi/nfi/NFINativeAccess.java) | `createNFISignature` calling `env.parseInternal`, `getOrCreateNFISignature` signature cache | Alternative for dynamic signatures, but different NFI backend, not reuse of the current FFM provider |
| [GraalVM 25 Native Image FFM](https://www.graalvm.org/jdk25/reference-manual/native-image/native-code-interoperability/ffm-api/) | `SymbolLookup.libraryLookup`, `foreign.downcalls` registration | Direct support for runtime shared-library selection when builder generated downcall code; no general dynamic-JAR loading |
| [GraalVM 25 reachability metadata](https://www.graalvm.org/jdk25/reference-manual/native-image/metadata/) | `MissingForeignRegistrationError` contract | Explicit metadata coverage is mandatory for dynamic declaration workloads |

Survey exclusions/non-leading precedents: GraalJS offers host-resource/interop precedent but no smaller direct public solution to Java FFM AOT descriptors; Sulong/LLVM is a separate guest native substrate; SimpleLanguage has no general FFM provider mechanism; Apple Pkl uses host authority boundaries but has no comparable generic native C ABI surface, so it informs host-policy separation only. Mature external C `dlopen/dlsym` plus `libffi` solves runtime signature flexibility but constitutes a different backend, while HotSpot/JDK 25's flexible runtime `Linker.downcallHandle` lacks Native Image's exact AOT constraints. Linux/Windows/macOS native library loader/ABIs differ, so only the existing Linux x86-64 LP64 profile is in scope.

**Production code rule:** Every **new** Java class/method must be justified by a concrete analogous method/annotation/behavior among these inspected sources or by direct reuse of an already ratified Protos SPI method. Do not embed other-language citations, external source names, issue numbers, or migration labels into new production Java identifiers/comments.

## GITHUB010 decision-space scoring

Scores 1 (worst) to 5 (best); `H` = high-confidence, `M` = medium, `L` = low. A = integrated original provider + finite metadata; B = combinatorial bounded envelope; C = NFI/libffi backend; D = external Native module; E = JVM-only status quo. Every cell includes its comparison rationale. These are qualitative design scores, not benchmarks.

| Required dimension | A | B | C | D | E |
| --- | --- | --- | --- | --- | --- |
| 1. Correctness / invariants | 4M: existing SPI; finite-gap risk | 3M: new arbitrary ceiling | 2L: ABI/ownership rewrite | 2L: new cross-image ABI | 1H: fails Native objective |
| 2. Protos alignment | 5H: existing ordinary Protos API | 4M: extra AOT inventory | 2M: parallel native backend | 2M: external bridge universe | 3H: preserves JVM-only API |
| 3. Pay only for need | 5H: only necessary stubs | 2M: speculative stub set | 1H: wholesale backend | 1H: new loader/bridge | 3H: zero new code, no feature |
| 4. Incremental growth | 4M: add descriptor rows | 3M: rechoose arity ceiling | 4M: runtime dynamic signatures | 3L: plugin protocol growth | 1H: cannot solve Native |
| 5. Future options | 4M: compatible Native evolution | 3M: bound may calcify | 3M: dynamic but couples NFI | 4L: decoupled if proven | 3H: no implementation lock-in |
| 6. Scalability | 4M: independent Actor sessions | 2M: combinatorial metadata growth | 3L: unknown many-context cost | 3L: extra shared-module lifecycle | 2M: JVM-only throughput/cost |
| 7. Simplicity | 5H: reuse SPI + metadata | 3M: generator/inventory | 1H: second call subsystem | 1H: cross-image object protocol | 5H: status quo simplest |
| 8. Portability/freedom | 3M: existing Linux ABI | 3M: existing Linux ABI | 3L: NFI backend constraints | 2L: native dynamic linking | 4H: JVM generic runtime |
| 9. Runtime/resources | 4M: no C load on idle path | 2L: large image/build cost | 2L: libffi/native backend | 2L: extra bridge cost | 5H: no new Native footprint |
| 10. Failure/operability | 3M: explicit shape rejection | 3L: boundary compatibility | 2L: new NFI faults | 2L: cross-image failures | 4H: known JVM failures only |
| 11. Reversibility/deferral | 4H: metadata easy to extend | 3M: shrink/migrate bound | 2L: expensive to undo | 2L: expensive bridge migration | 1H: delays real adoption |
| 12. Evidence maturity | 4H: GraalPy/Ruby + existing FFM | 3M: AOT supported, cost unknown | 3M: Espresso NFI precedent | 2L: no matching drop-in Java SPI | 5H: existing known JVM mode |

**Non-compensating red flags:** C and D impose a second execution/lifetime boundary not justified by the present Native use case; B charges current builds for unused signature combinations; E fails the motivating feature. A's own underengineering risk is unsupported new signatures, but the change is additive by physical metadata rather than a rewrite of public Protos values/ownership. A would be disqualified if the real Native FFM adapter cannot preserve the approved source/ABI/error semantics or cannot prove the mandatory Native compilation/installation gates.

**Pay as needed:** A's unused-process tax is compiled provider code + finite stubs; no eager FFM session, Linker, Arena or shared-library load. This tax must be measured against a pre-I170 binary and reported honestly, not asserted zero. B/C/D bring additional speculative build/runtime cost.

**Grow as needed:** New C libraries with existing physical signatures need no Java changes or rebuilt Protos; a new physical signature needs an additive Native metadata/binary release. Future truly unbounded runtime ABI would be a separate architecture choice. The cost of deferring NFI is low until that need is demonstrated; conversely deferring Native FFM entirely preserves the present installed-application blocker.

**Strongest counterargument against A:** JVM can compose unrestricted valid R1 signatures at run time whereas Native AOT cannot. Finite coverage can surprise code that is semantically valid on JVM. The selected mitigations are an explicitly described Native availability envelope, deterministic safe rejection and independent-library validation. No assertion of full JVM parity.

## Future-scenario stress / adversarial matrix

| Scenario | Required behavior / proof |
| --- | --- |
| Ordinary Native without `--allow-library` | No built-in provider registered and no FFM session/native `.so` acquired |
| `--allow-library` provided but no `ffi:` import | Authorization map exists, but no native `libraryLookup` or shared-library initializer |
| Import `std:ffi` or `std:sqlite` only | No foreign library acquired by the import |
| Authorized SQLite | Memory and persistent reopen/WAL, TEXT/BLOB, prepare/step/columns, resource teardown |
| Another C library | Different authorized `.so` and signature, same generic bind machinery |
| Unauthorized alias / invalid target / bad symbol | Existing explicit errors; no alternate search, host-path disclosure or privilege |
| Valid R1 signature omitted from Native AOT envelope | Fail before C, using compatible existing error projection; never `MissingForeignRegistrationError` escape, no ABI coercion |
| Owned failure, out value, borrowed-region lifetime | Existing per-Actor custody, bounded copy, early revocation, exactly-once release |
| Multi-Actor/Context, blocked C call or cancellation | No global session registry, no native callback/reentrancy innovation, existing host cancellation semantics |
| JVM provider with explicit JAR | Current ServiceLoader/URLClassLoader and provider policy unchanged |
| Installed Native with no Java tools | Runs through `dist/install.sh` with only OS shared-library dependencies available |
| ELF and compiled Tier-2 guest JIT | No mandatory SQLite `DT_NEEDED`; release validators and optimizer gates unchanged |

## Publication and validation classification

- **Design owner approval:** VERIFIED from the explicit 2026-11-22 conversation for Candidate A, including finite build-time signatures, minimal generic provider, default-off/no-bundled-libraries and unchanged JVM route.
- **GITHUB021 invariant consistency:** PASS for this **design**. D238/D259/D256, I085/I144/I169, PLAT038 and Actor custody are preserved; no conflicting previously approved invariant or implicit override identified.
- **Protos baseline:** `1fa0c41728792596a1e4862f75298a2532b56bb1` inspected; no implementation revision or new I170 source has been published.
- **User-reported tests and `git diff --check`:** PASS locally **before I170 implementation**, exact commands/logs independently unavailable; not evidence of Native FFM.
- **New Native FFM JVM tests, Native image build/installer, release validation, ELF, size/performance, unsupported ABI proof:** NOT RUN / NOT ATTESTED.
- **Change in `spec/`:** NONE at ratification. A semantic conflict or new public error family requires a separate owner-approved normative decision.
- **Execution policy:** Human executes product build, tests, validation, Git and release; docs-only publication is authorized explicitly. No code or speculative CHANGELOG/version bump is produced by these records.
