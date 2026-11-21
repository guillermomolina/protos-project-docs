# D259 — Explicit trusted native FFM library authorization (B2+A)

**Decision status:** OWNER-APPROVED, RATIFIED DESIGN SELECTION (2026-11-21); **NOT** a claim that the Protos normative specification, CLI, provider implementation, or tests for D259 have been published.
**Canonical live issue:** [D259 #1139](https://github.com/guillermomolina/protos/issues/1139).
**Owner approval provenance:** The owner first selected candidate B (reusable generic trusted-in-process FFM provider), then approved the B2 host-configuration direction; expressly clarified that permitting a native library means accepting **all risks** of executing it in process; after a comparison of the remaining A/B/C configuration alternatives, explicitly replied **“si apruebo A”** to option A — an explicit `alias=native-target` mapping. These are separate, cumulative approvals, not approval of automatic naming or blanket unrestricted libraries.
**Inspected Protos baseline:** [`2df251847ead94012f8d5a87c2b81d2d981882c7`](https://github.com/guillermomolina/protos/commit/2df251847ead94012f8d5a87c2b81d2d981882c7) on `main` (2026-11-21). The baseline still has the SQLite-specific FFM provider policy and **does not implement D259**.
**Revision-bound evidence and handoff:** [D259 approval, source, and handoff audit](../../evidence/D259/D259_B2_A_OWNER_APPROVAL_SOURCE_AND_HANDOFF_2026_11_21.md).
**Authority:** This record is a durable **non-normative design decision**. The normative Protos contract must be published under `guillermomolina/protos/spec/`, with a new specification changelog entry, before production implementation is treated as conformant.

## 1. Exact approved choice

Choose **B2 + option A**: reuse the existing optional I085 external-provider SPI, trusted-in-process admission, I144 Java 25 FFM backend, D188 foreign-value boundary, and Protos-first `std:ffi` declarations; replace the hardcoded single SQLite library policy with an **explicit host-supplied map of logical aliases to exact native loader targets**. No default library is authorized.

The selected CLI surface is a **repeatable** `--allow-library <alias>=<target>`. Examples:

```sh
protos --foreign-provider-path ./protos-ffm-provider.jar \
  --allow-library sqlite3=libsqlite3.so.0 \
  --allow-library math=libm.so.6 \
  app.protos

protos --foreign-provider-path ./protos-ffm-provider.jar \
  --allow-library sqlite3=/opt/native/libsqlite3.so.0 app.protos
```

Guest source uses the stable logical identifier `import("ffi:sqlite3")`, **never** a native loader path or a permission-provisioning API. The first-party `std:ffi` module remains an authority-free ordinary import.

Embedder configuration uses the already present `ProtosForeignProviderConfiguration.trustedInProcess(providerPaths).withProviderOptions("protos-ffm", Map.of("library.sqlite3", "libsqlite3.so.0", ...))`. Option keys follow `library.<alias>`; values are exact host-selected targets. CLI and Java embedding must build equivalent policy; no new public provider SPI, global ambient library registry, or guest authorization channel is selected.

- Alias: exact, case-sensitive, valid C-identifier-style token matching the existing `FfmTypes.requireIdentifier` shape, `[A-Za-z_][A-Za-z0-9_]{0,127}`.
- Target: a nonempty exact native loader name such as `libsqlite3.so.0` **or** an absolute filesystem path supplied by the host. Relative paths are invalid. No inferred `lib` prefix, guessed extension, auto-version discovery, name aliases or fallback search policy in Protos.
- Empty allowlist: no native libraries available; provider JAR selection alone no longer authorizes SQLite. Repeated/conflicting definitions of the same alias are configuration errors; two distinct aliases may intentionally refer to the same native target.
- Native file identity is **not** a sandbox or pinning promise. Loader search, symlinks, replacement, dependencies, constructors, transitive symbols, and subsequent native-side loads remain risks that the host has agreed to accept.
- The option applies to application execution (file, `-e`, REPL and the applicable `run`/`debug` application commands), not to bundled Tools. CLI routing must preserve the existing ordinary application-permission rules.

## 2. Owner-selected trust and isolation semantics

**Authorizing a library is the host's explicit decision to accept all risks of that library executing trusted inside the operating-system process.** This includes host filesystem/network/system effects beyond Protos guest capabilities, C process-global state shared across Actors/Contexts/Processes co-located in that OS process, blocking and thread-affinity requirements, unsafe ABI declarations, memory corruption, native initializers, library dependencies and OS-process termination.

The allowlist checks **which aliases the FFM provider is allowed to acquire**. It is not an OS sandbox, whole-process confinement facility, native behavior analyzer, symbol sandbox, native-libraries-only execution domain or proof of safety. Do not invent per-library locks, worker isolation, IPC, Landlock, audit/proof mechanisms, library-specific permissions, or automatic global-state proxies. A trusted Java provider plugin is itself executable host-trusted Java code and is not constrained by this FFM-provider-specific allowlist.

Existing `--allow-filesystem`, `--allow-network`, `--allow-resolver`, `--allow-entropy`, `--allow-clock` and other ordinary Protos capability grants remain **independent**: allowing a native library does not enable, revoke, widen or disable those guest permissions. Protos-managed objects, Actor state, module facades, resource generations and non-transferability continue to follow their existing rules.

**Explicit normative delta required:** `spec/concurrency/ACTORS.md` §24K currently forbids ordinary cross-Actor access to mutable foreign globals except through an explicitly defined semantic boundary. For the host-trusted FFM facility, the authorization itself selects an explicitly **unsafe external native capability boundary**: Protos does not promise isolation of the native library's global state and does not impose native concurrency coordination. This does **not** make Protos mutable heap objects cross-Actor or native resources automatically P-transferable. Before implementing the behavior, amend §24K and `spec/semantics/FFI.md` to delimit this exception expressly, not silently contradict Core.

Synchronous native calls preserve `ACTORS.md` §24J: no implicit Protos suspension or Actor-local reentry. A native operation can block indefinitely. Callback/upcall and arbitrary C++ ABI support are **not** newly approved.

## 3. Acquisition, errors and lifecycle

- `canonicalTarget(exactSpecifier)` remains a stable logical-identifier check, independent of embedder options; do not expand the SPI to expose host configuration at that stage merely to reject imports earlier.
- Authorize `alias` using the immutable per-provider host option map **before** constructing native `Linker`/`Arena` or calling `SymbolLookup.libraryLookup`. Unknown aliases may cause a lightweight host session to exist but must **not** trigger native library load.
- Preserve lazy acquisition; the FFM JAR's presence and `std:ffi` import alone must not initialize native FFM or acquire an OS library.
- Preserve Actor-local foreign MODULE cache-before-initialize, same-key identities, failed-initialization cache removal and retry, provider-session binding, resource retirement and non-transferability.
- Invalid host CLI syntax, malformed alias/target and duplicates: usage/configuration error. Invalid logical import identifier: ordinary resolution error. A syntactically valid unlisted alias fails as existing entered `ForeignError` category `unauthorized-library`. Authorized but missing native target: existing `ForeignError` `library-unavailable`. Existing preentry argument rejection and entered-foreign-error distinctions are unchanged. Native crashes are not recoverable Protos Errors.
- Migrate current implicit `sqlite3` support and legacy `sqlite3.path` option to explicit `library.sqlite3`; update examples, CLI usage, tests and provider documentation rather than keeping a second implicit-authority path.
- Limit the current executable FFM ABI to already specified Linux x86-64 LP64 / supported JDK 25 JVM profile. Approval of a C++ or Qt/Boost **library name** does not promise support for C++ ABI, C structs, callbacks, variadics, pointers, cross-platform Native Image plugin loading or untested systems.

## 4. Alternatives, comparative basis, and rejection rationale

The comparison covered: GraalPy's `NativeContext.loadLibrary(String,int)` and context-managed native libraries (direct native loading with explicit runtime ownership); GraalPy's `NativePosixSupport.loadNativeLibrary(PythonContext)` (explicit runtime-selected path, `@TruffleBoundary`); the official Java 25 `SymbolLookup.libraryLookup(String|Path,Arena)` (FFM name/path loader); Python `ctypes.CDLL` (language-level direct arbitrary library selection); Rust `libloading::Library` and .NET `NativeLibrary.Load` (host-side explicit loading); and Truffle NFI/Sulong patterns (foreign native loading by an interop runtime). These systems are evidence for **loading and resource-boundary mechanisms**, not authority for Protos's specific security decisions. The contrast spans runtime-managed native loading, language-directed dynamic loading, and host/embedding-authorized lookup.

Three materially plausible B2 configuration forms were evaluated:

- **A (SELECTED):** repeatable `--allow-library alias=target` mapped to existing embedder options. Explicit, deterministic, names distinct from loader targets, no new provider SPI.
- **B (REJECTED):** `--allow-library sqlite3` with automatic platform-specific target mapping. Superficially convenient, but introduces unapproved search/filename heuristics (e.g. `sqlite3` is not necessarily a valid loader name for `libsqlite3.so.0`) and OS/version ambiguity.
- **C (REJECTED FOR R1):** generic `--provider-option protos-ffm:library.sqlite3=...`. Extensible but exposes a general provider configuration CLI and extra public surface not justified by the current requirement.
- **Defer/do nothing (REJECTED):** retains SQLite hardcoding and cannot admit unrelated authorized C libraries without editing/repackaging the provider.

Comparative scores, 1 (weak) to 5 (strong), **H** = high confidence, **M** = medium confidence:

| AGENTS GITHUB010 axis | A explicit map | B inferred name | C generic option |
| --- | --- | --- | --- |
| Correctness/invariants | 5H | 2H | 4H |
| Protos alignment | 5H | 2H | 3H |
| Proportionality/pay only when used | 5H | 4M | 3H |
| Incremental growth | 5H | 2H | 5H |
| Future options | 4H | 2H | 5H |
| Scalability | 5H | 3M | 5H |
| Conceptual simplicity | 5H | 4M | 2H |
| Portability/freedom | 4H | 1H | 4H |
| Runtime/resource cost | 5H | 5H | 5H |
| Failure/operability | 5H | 2H | 3M |
| Deferral/reversibility/migration | 4H | 3M | 4H |
| Evidence maturity/implementation risk | 5H | 2H | 4H |

**Strongest argument against A:** an explicit grant can execute code that compromises or crashes the entire hosting OS process, including unrelated Protos Contexts; even absolute paths do not supply a hardened identity guarantee. This is not concealed or mitigated by unsupported per-library safety machinery; it is the precise **owner-accepted trusted-in-process** model. Host process-level isolation, if ever desired, is independent future work and must never be presumed from this allowlist.

## 5. GITHUB021 owner-invariant/delta audit

| Previous approval or owner correction | B2+A result |
| --- | --- |
| D238 Protos-first `std:ffi`, D188/I085 single boundary | PRESERVED; no parallel import/foreign universe |
| D238 host-authorized aliases and trusted FFM | PRESERVED; generalized beyond SQLite |
| Owner's B2 selection (not deferral) | PRESERVED; one generic host-selected allowlist |
| Owner's full-risk acceptance for allowed native code | PRESERVED; no native behavioral confinement |
| Independent Protos guest capability grants | PRESERVED; no capability auto-grant/auto-revoke |
| Native Actor-local managed references and non-transferability | PRESERVED; only native library internals have no isolation guarantee |
| Prior §24K Core no unrestricted foreign mutable globals | **EXPLICITLY DELIMITED** by owner-approved unsafe/trusted native exception; normative amendment required before implementation |
| Host mapping A rather than automatic target or generic provider CLI | PRESERVED; exact alias→target, CLI and embedder |
| Existing ABI/platform/lifecycle restrictions | PRESERVED |
| D256 SQLite durable authority, PLAT074, LIB031 | **NOT** implicitly reopened, solved, ratified, unblocked or implemented |

**GITHUB021 conclusion:** exact selected owner invariants are consistent **for this approved design**. The only boundary adjustment — trusted native library state versus §24K — has been explicitly surfaced in the decision packet and approved through the owner's native-risk clarification; it remains to be rendered as a coherent normative delta in the product specification. No unapproved native confinement requirements are added.

## 6. Falsification and implementation handoff

The implementing `Ixxx` must cover (at minimum): empty allowlist and implicit-SQLite removal; repeated and malformed CLI syntax; invalid alias/relative path; unknown target before any native loading; two independent libraries in the same provider; matching CLI/embedding policies; lazy no-provider/no-import operation; same target through distinct aliases; process-global native state visible through explicitly authorized C calls across Actors; preserved Protos Actor/resource rules; native I/O independent of Protos guest I/O grants; provider load errors and D188 error projection; multi-Context authorization separation; closure/termination and generation custody; distribution-specific Linux x86-64/JDK25 and unaffected baseline tests.

The implementation must reuse the current `FfmLibraries`, `FfmForeignProvider`, `FfmSession`, `ProtosForeignProviderConfiguration`, `ProtosForeignModuleImports`, `ProtosCli` and I144 test seams. Justify each production Java modification by a **specific method, annotation, and equivalent verified behavior in GraalPy or another Truffle language**; do not invent an additional Java architecture. The `FFI.md` C-ABI descriptor `i32` refers to the foreign C ABI and is **not** an `int` primitive in Protos (whose primitive numbers use `long`/`double`).

This selection **does not prove implementation**. At the inspected revision, no D259 source/tests had been published. The user reported `git diff --check` clean and all local tests passing (2026-11-21); no command logs, test counts, or D259 implementation results were supplied, and no new D259 implementation tests were run by the agent. Publication of this record is a separate project-documentation event.

```text
FORMAL_IDENTIFIER=D259
FAMILY=D
OWNER_APPROVAL=EXPLICIT_2026-11-21_B2_PLUS_A
DECISION_SELECTED=GENERIC_HOST_AUTHORIZED_TRUSTED_FFM_ALIAS_MAP
GITHUB010=RESEARCH_AND_COMPARISON_RECORDED
GITHUB021_INVARIANT_CHECK=PASS_FOR_APPROVED_DESIGN
PROTOS_REVISION=2df251847ead94012f8d5a87c2b81d2d981882c7
PROTOS_REVISION_ROLE=INSPECTION_ONLY
PROJECT_RECORD_REVISION=THIS_RECORD_PUBLICATION_COMMIT
NORMATIVE_SPEC_PUBLISHED=NO
IMPLEMENTATION_PUBLISHED=NO
D259_SPECIFIC_TESTS_RUN=NO
```
