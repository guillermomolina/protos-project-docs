# PLAT055 — Java-supplied application-module catalog for standard Polyglot embedding

**Status: RATIFIED PLATFORM ARCHITECTURE — PRODUCT IMPLEMENTATION PENDING.**  
**Owner approval:** 2026-10-08, exact reply **“apruebo A1”** to the full I087-1 candidate packet.  
**Decision Issue:** [PLAT055/#844](https://github.com/guillermomolina/protos/issues/844)  
**Implementation consumer:** [I087/#841](https://github.com/guillermomolina/protos/issues/841) (native child of completed I086/#840).  
**Product revision at approval:** [`protos@6dab6ecc08c9a2102a388e00908c710a15cf2a4c`](https://github.com/guillermomolina/protos/commit/6dab6ecc08c9a2102a388e00908c710a15cf2a4c), implementation `0.3.298-SNAPSHOT`, global normative specification `0.1.451`.  
**Evidence:** [I087-1 approval, comparisons and GITHUB021 review](../../evidence/I087/I087_1_A1_APPROVAL_AND_INVARIANT_RECONCILIATION.md).

## Problem and selected public Java surface

The standard `Context.eval` embedding from PLAT054 creates its module runtime with `ProtosStandardLibraryModuleResolver` alone. It can import `std:` modules, but application-defined source cannot currently serve as an `Actor.spawn` destination without a host-authorized resolver.

**Selected: Candidate A1, a Java-supplied immutable exact module catalog owned by one existing Polyglot Context.** Its minimal *host Java* API has this contract:

~~~java
import org.graalvm.polyglot.Context;
import java.util.Map;
import com.guillermomolina.protos.execution.ProtosEmbeddedModules;

try (Context context = Context.newBuilder("protos")
        .allowCreateThread(true)
        .build()) {
    ProtosEmbeddedModules.install(
            context,
            Map.of("app:workers", "<host-supplied Protos source characters>"));
    context.eval("protos", "<main Protos source using app:workers>");
}
~~~

The names and signatures above are selected as part of A1; this illustrative snippet is **not** runnable Protos code until I087 implements the API and supplies valid Protos source. `allowCreateThread` serves Actor carrier policy, **not catalog registration**. Installation must work with the default `Context.newBuilder("protos").build()` where no Actors are used.

### Lifecycle and ownership

- The Java method operates on the **same** ordinary public Polyglot `Context`, not an internal second Context, a benchmark driver, a specialized bootstrap or a guest setup Source.
- The implementation may use the documented `Context.initialize("protos")`, `Context.enter()`, `Context.leave()` and the internal `ProtosLanguageContext` reference to access Context-owned state. GraalVM exposes no `Context.setModuleCatalog`; this is a **Protos-defined public Java method**. The implementation is responsible for safe entry/leave and exception cleanup.
- Defensively copy the `Map<String,String>` into an immutable Context-local snapshot. Entries contain Protos source characters; do not hold guest module objects, closures, Actor contexts or a mutable host registry.
- Exactly one successful catalog installation per Context, including an empty catalog. Installation may occur after `Context.initialize` or read-only binding inspection, provided the first standard Process bootstrap **has not begun**. Initialization/bindings alone must not create Process/Core/RootActor.
- Installation concurrent with Process bootstrap must be linearized under one Context-local synchronization boundary. After that bootstrap begins, reject installation. A failed invalid catalog should leave no partially installed catalog and must not accidentally bootstrap a Process.
- Fail host configuration with Java `IllegalArgumentException` for invalid inputs (null, invalid/empty specifier, null source, reserved domain) and `IllegalStateException` for duplicate/late/closed registration, subject to Polyglot's own closed-Context state errors. Do not introduce guest-visible configuration primitives or new Error prototypes.
- The catalog is reachable only through its owner Context's private runtime state and is released with Context disposal; Process termination does not resurrect it or make it visible to guests.

### Source keys and loading

- One catalog entry maps an **exact** String specifier `app:<nonempty suffix>` to immutable Protos source characters. The exact specifier is the canonical internal `ProtosModuleKey` identifier, with no aliases, path normalization, implicit extension, relative resolution or filesystem/package discovery.
- `app:` is a host-resolver namespace, **not** a new Core syntax, import intrinsic, guest authority, nor a filesystem URI. Even path-looking suffixes remain opaque exact strings; never traverse a host path from them.
- Reject attempts to define `std:` keys or reserved `foreign:v1:` keys; the existing `std:` library keeps its own resolver and foreign keys remain exclusively owned by foreign-provider routing.
- For catalog code, obtain `ProtosModuleSource.fromCharacters(key, chars)` and compile it in the entered owner Context. No general FileSystem/Network grant is implied by private host-supplied characters.
- At the lazy standard embedding bootstrap, compose the exact catalog resolver with the existing `ProtosStandardLibraryModuleResolver` and hand the **same** resolver to the existing `ProtosCoreBootstrap` → `ProtosModuleRuntime`. Do not introduce a second Actor loader or a new guest import path.
- Unmatched `app:` imports fail via the normal Protos `Error` contract, with no ambient fallback to cwd, host classpath, system package registries, OS filesystem or another Context's catalog.

### Guest semantics unchanged

- `import()` and `Actor.spawn()` share the resolver and canonical ModuleKey. Actor.spawn resolves in the creator **before creation cutover**; the destination independently loads its own Actor-local module instance and invokes the destination module's **own** bootstrap binding. Creator closures, live module contexts and arbitrary capabilities are never transferred implicitly.
- Actor-local cache-before-execute (`INITIALIZING`, `READY`), cycles, partially initialized modules, failed-initialization eviction and subsequent retry are unchanged. Across Actors or Contexts, source/code may be shared only as semantically invisible immutable artifacts; mutable Protos state never is.
- Ordinary `Context.eval(Source)` never fabricates importable identity by matching `Source` name, URI, contents or repeated evaluation against catalog entries. Standalone entries remain distinct per evaluation.
- Public `getBindings("protos")` and `Value.execute()` behavior remains PLAT054 as implemented; no new host scope, exports registry, writable binding or Process bootstrap through scope lookup.
- Failure resolving `Actor.spawn` specifier prevents creating the Actor; failure of destination bootstrap after creation follows the existing Actor lifecycle and does not retroactively fail `spawn`.

### Capability and pay-as-you-grow constraints

Code availability is **not** authority. Catalog registration grants no guest filesystem, network, host access, thread creation, foreign provider, Process capability, or cross-Actor mutable sharing. Core internal resource loading remains distinct from guest Filesystem.

Without an installed catalog, preserve `std:`-only standard embedding, lazy bootstrap and the absence of any mandatory catalog lookup/registry/provider allocation, Task/RootTask, scheduler, host-entry serialization or per-call setup. Only when imported/used is application source compiled/cached under the ordinary Actor-local module runtime. Closing or cancelling Context preserves I086/PLAT054 lifecycle behavior.

## Alternatives and reversibility

- **A1 selected:** exact Java catalog, no filesystem requirement, narrow synchronized Context-local registration.
- **B deferred:** `protos.AppRoot` / physical authorized root. Requires provider confinement, path/symlink/platform policy and adds filesystem deployment coupling. May be added later as another explicit resolver without changing Actor identity semantics.
- **C deferred:** custom Truffle FileSystem resource provider. Useful for broader resource ecosystems, unnecessary for explicit in-memory sources.
- **D rejected:** Polyglot bindings as private module registry; those are an inter-language shared data surface.
- **E rejected:** mutable Engine-wide/global catalog; violates Context ownership and accidental-sharing constraints.
- **F rejected:** continuing `std:` only does not address I087.
- Deliberately no dynamic post-bootstrap updates, hot reload, alias framework, package discovery or new persistent semantic module identity.

See the linked evidence for complete twelve-dimension GITHUB010 score justification and risks. Portability and Native Image remain product acceptance claims requiring tests, not consequences of approval.

## GITHUB021 approved-invariant/delta reconciliation

The complete I087-1 proposal, including the `app:` namespace and one-time Java API, was selected explicitly by the owner. Rechecked against:
1. PLAT054 one lazy Process/RootActor per Polyglot Context and stable read-only host bindings: **PRESERVED**.
2. PLAT054 CoreRoot/home/packaged Core precedence: **PRESERVED**.
3. PLAT054 Truffle Env and effective Filesystem/Network grants: **PRESERVED**.
4. MODULES host-only canonical resolver identity and Actor-local cache/standalone Sources: **PRESERVED**.
5. ACTORS §8 one creator resolution, destination-local load, own bootstrap binding, no carried closures/capabilities: **PRESERVED**.
6. PROCESS_IO bounded host authority, no private Core-to-guest authority leak: **PRESERVED**.
7. Protos PAY AS YOU GROW and independent Actor/P progress: **PRESERVED**.
8. PLAT053 foreign-key source/foreign exclusivity and provider isolation: **PRESERVED**.

**New consequences explicitly included in A1 approval:** the Java `install` host API, exact `app:` key scheme, immutable defensive snapshot, single-install-before-bootstrap timing, Java configuration errors and exclusive domains. They do not change Core syntax/module identity semantics. No previously approved invariant has been changed or silently narrowed; no deferred alternative is selected. **GITHUB021 invariant/delta review: PASS for the exact A1 contract, not for hypothetical later refinements.**

## Authority and implementation handoff

**Non-normative host platform decision.** In product specification `0.1.451`, `spec/semantics/MODULES.md` already assigns specifier resolution to host policies, while `ACTORS.md` §8 and `PROCESS_IO.md` govern runtime and host authority. No additional normative Core semantic revision is required for the approved exact host-resolver catalog. If implementation uncovers guest-visible behavior not uniquely resolved there, stop the affected change and obtain explicit further owner approval before updating normative spec.

I087/#841 owns one coherent implementation slice **I087-2 / IMPLEMENTATION** in `guillermomolina/protos`. Its agent is a preparer; the human executes all tests/builds, product Git mutation and product publication. Required coverage includes registration timing/races/errors, canonical identity, import caching/cycles/failed retries, Actor.spawn load and own-slot bootstrap, cross-Actor/Context isolation, unavailable modules, no ambient I/O, `std:` precedence, idle/no-catalog PAY AS YOU GROW, Context close, portable-JAR conformance and explicit Native Image scope. New product release metadata is modified **only after** human green tests and immediately before commit; no tests after metadata edits.

No product implementation, tests, Native Image success, or performance parity is claimed by this architecture ratification.

```text
DECISION=PLAT055
STATUS=RATIFIED_OWNER_EXACT_A1_2026_10_08
GITHUB010=COMPARED_A1_B_C_D_E_F
GITHUB021=PASS_WITHIN_A1_SCOPE
PRODUCT_HEAD_AT_APPROVAL=6dab6ecc08c9a2102a388e00908c710a15cf2a4c
SPEC_BASELINE=0.1.451
SPEC_CHANGE_REQUIRED=NO_FOR_APPROVED_HOST_RESOLVER_CONTRACT
IMPLEMENTATION_OWNER=I087/#841
IMPLEMENTATION_STATUS=NOT_STARTED
```
