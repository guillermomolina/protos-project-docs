# AUD019-A — Foreign-library interop and polyglot import architecture audit

Slice status: **COMPLETE**

Parent Issue: `guillermomolina/protos#818` (`AUD019`)

Research date: **2026-10-07**

Protos revision audited: `8bd5c37c7cca13599df39d18de2eadf01e4a86eb`

Pinned GraalVM/Truffle line at that revision: `25.4.4.1.1`

Later `main` observed during publication: `ddf59b1b3f5c97758354688088845d641f7a4e04` (`LIB015-C1`). Its delta is confined to `std:logging`, its tests, `pom.xml`, and `CHANGELOG.md`; it does not change the module, interop, Context, Actor/module-cache, Package Tool dependency, or foreign-runtime surfaces on which this slice depends. The audit therefore remains revision-bound to `8bd5c37...` while the later unrelated movement does not invalidate its conclusions.

## Scope and correction

This slice closes only the **base architecture investigation**. It does not close parent `AUD019` and does not authorize implementation.

The Issue-creation references `PLAT410 / #659` and `PLAT411 / #660` are invalid for this subject and were ignored. The verified prior interop authority is `PLAT013 / #250`, which selected semantic-value-native `InteropLibrary` projection for Protos values and explicitly deferred Closure `execute` as a new public foreign-call path.

Project-owner clarification also established:

~~~text
PROTOS_WRITTEN_BINDINGS_REQUIRED=NO
PROTOS_WRITTEN_BINDINGS_ALLOWED=YES
PROTOS_WRITTEN_BINDINGS_PREFERRED=NOT_PRESELECTED
RUNTIME_OR_GENERATED_SHIM_ALLOWED=YES
SYNTHETIC_PROTOS_FACING_FOREIGN_FACADE=FIRST_CLASS_CANDIDATE
~~~

No implementation, test, build, benchmark, local spike, repository mutation, or Issue mutation was performed during the research itself.

## Executive conclusion

The cleanest base architecture is:

~~~text
                    Protos source
                         |
                 import("scheme:target")
                         |
                         v
                ordinary module resolver
                         |
                         v
                ForeignImportProvider
                  /       |       \
              java     python      js ...
                |         |         |
                +---------+---------+
                          |
                acquired foreign value
                          |
              facade/shim only when needed
                          |
                          v
              Protos-facing imported value

std:interop
    |
    +---- uses the same foreign-value substrate
              |
              v
        InteropLibrary / Env / Context
~~~

The central result is:

~~~text
GENERIC FOREIGN VALUE PROTOCOL       YES
GENERIC FOREIGN MODULE IMPORT        NO
~~~

Truffle standardizes operations on foreign **values** through `InteropLibrary`, but it does not standardize a universal operation equivalent to “import module X from language Y”. Java class lookup, Python imports, JavaScript module loading, Ruby `require`/load behavior, and native-library/symbol acquisition remain different acquisition mechanisms.

Therefore the correct model is:

~~~text
ONE_GENERIC_INTEROP_SUBSTRATE
+
ONE_IMPORT_PROVIDER_PER_LANGUAGE_OR_MODULE_SYSTEM
~~~

The recommended representation is **HYBRID with direct-foreign-value bias**:

1. preserve the real foreign value when its `InteropLibrary` shape maps naturally to Protos;
2. use a small provider/runtime-owned facade when module semantics, security metadata, normalization, or deliberate adaptation requires one;
3. permit generated shims as an optimization/adaptation technique rather than a mandatory layer;
4. keep library-specific or Protos-written bindings optional for APIs that benefit from an idiomatic Protos surface.

## Current Protos architecture

### `import(...)` and module identity

Core already separates the exact String module specifier from host-defined resolution. A successful resolver produces a canonical `ModuleKey`; from that point module identity, Actor-local caching, cache-before-execute behavior, initialization, cycles, and failure semantics are governed by Protos.

The implementation boundary is currently:

~~~text
ProtosModuleKey resolve(String exactSpecifier, Optional<ProtosModuleKey> importingModule)
ProtosModuleSource loadSource(ProtosModuleKey key)
~~~

That shape assumes every resolved module ultimately supplies **Protos source**. Foreign imports expose the architectural seam: a Java class or Python module should not have to manufacture fake `.protos` source merely to participate in `import()`.

The existing specifier -> canonical `ModuleKey` boundary should be retained, while the load result eventually needs to represent at least:

~~~text
ResolvedModule
    |
    +-- ProtosSourceModule
    |
    +-- ForeignModule
            foreign value
            provider identity
            lifecycle / authority metadata as required
~~~

This is a platform/runtime architecture change, not evidence that parser syntax must change.

Current module instances are real Actor-local Protos module objects and repeated imports resolving to the same `ModuleKey` return the same module instance within one Actor. That creates unresolved guest-semantic questions for foreign modules: repeated import identity, Actor-local versus Context-local facade identity, and how foreign identity interacts with Protos `===` cannot be selected mechanically.

### Polyglot Context

At the audited revision Protos constructs its Polyglot Context for `ProtosLanguage.ID`, configures input/output/error and bounded I/O, and initializes Protos. It does not enable a general HostAccess policy, host-class lookup, additional guest languages, or general polyglot-language execution for this feature.

`ProtosLanguageContext` owns `TruffleLanguage.Env`, but its current public parse helper rejects non-Protos `Source` values. Therefore general foreign execution is not an already-present hidden feature; Context composition and cross-language acquisition must be deliberately introduced.

### Existing `InteropLibrary` projection

PLAT013 selected semantic-value-native interop: real Protos runtime values are the authoritative interop receivers, with synthetic adapters reserved for genuine non-value/tooling boundaries. Current value families including null, Boolean, numeric values, String, Array, Bytes, and ordinary objects export faithful `InteropLibrary` facets.

PLAT013 explicitly deferred `ProtosClosureValue` executability. Current Closures are invoked by Protos' own execution machinery and do not expose an approved `InteropLibrary.execute` entry point. Consequently foreign -> Protos callbacks are not currently authorized by existing architecture.

## What Truffle actually provides

Primary authority:

- `InteropLibrary` Javadoc: https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/interop/InteropLibrary.html
- `TruffleLanguage.Env` Javadoc: https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/TruffleLanguage.Env.html
- GraalVM JDK 25 embedding guide: https://www.graalvm.org/jdk25/reference-manual/embed-languages/
- GraalVM JDK 25 polyglot programming: https://www.graalvm.org/jdk25/reference-manual/polyglot-programming/

`InteropLibrary` supplies the common value-operation protocol: members, executable/instantiable values, arrays, hash entries, iterators, primitive/string/numeric facets, exceptions, metaobjects, identity, source/language information, and related messages.

`TruffleLanguage.Env` supplies lower-level facilities including public cross-language parsing and authorized host-symbol lookup. In particular, `lookupHostSymbol` returns a Truffle interop representation of an accessible Java class, while `parsePublic(Source, ...)` selects the target language from the `Source` language ID subject to embedder accessibility.

These APIs establish a common **use** substrate. They do not establish a common **module/package acquisition** protocol.

## Java architecture

### Host Java versus Espresso

These are distinct systems and should remain distinct.

For ordinary access such as:

~~~protos
Instant = import("java:java.time.Instant")
now = Instant.now()
~~~

the smallest mechanism is host Java interoperability:

~~~text
java:java.time.Instant
        |
        v
JavaImportProvider
        |
        +-- authority check
        +-- Env.lookupHostSymbol("java.time.Instant")
        |
        v
host-class interop value
~~~

A Java class can therefore be the imported foreign value directly. No per-library binding or wrapper is intrinsically required.

Espresso is guest Java implemented as a Truffle language. If Protos ever exposes it, it should be a distinct provider/authority choice rather than silently redefining `java:` to mean a guest JVM.

Primary authority:

- host Java embedding and HostAccess: https://www.graalvm.org/jdk25/reference-manual/embed-languages/
- Espresso interoperability: https://www.graalvm.org/jdk25/reference-manual/espresso/interoperability/

### Generic JAR support

Generic Java interoperability can cover arbitrary already-provisioned JARs without one binding per library. Dependency acquisition must remain separate:

~~~text
Package Tool / project environment
    -> obtains and provisions JAR/class/module path

import("java:com.example.Foo")
    -> resolves an already-available class
~~~

Constructors, static and instance members, fields, arrays, primitive values, exceptions, and overload selection can use the host interop machinery. SAM/interface callbacks depend on a later approved Protos Closure executable projection.

## GraalPy

GraalPy values/functions/classes participate in the common polyglot value protocol, but Python module acquisition is Python semantics. The provider must invoke Python's own import machinery inside the configured Python environment, conceptually:

~~~text
python:numpy
    -> PythonImportProvider
    -> Python import/importlib semantics
    -> GraalPy module object
    -> InteropLibrary
~~~

Python package provisioning and runtime acquisition remain separate. GraalPy documentation also shows that third-party/native packages need a correctly prepared environment/runtime, reinforcing the dependency-acquisition boundary.

Primary authority:

- GraalPy documentation: https://www.graalvm.org/python/docs/
- GraalPy overview/embedding: https://www.graalvm.org/python/

## GraalJS

GraalJS demonstrates the same split. Once JavaScript objects/functions/modules are acquired they are ordinary polyglot foreign values, but module/package loading follows JavaScript-specific rules.

The JDK 25 documentation distinguishes Java `Context` embedding from Node.js: ES modules can be evaluated through module-aware Sources and file resolution, while Node built-ins and ordinary CommonJS `require()` are not universally present in a Java-embedded Context.

Therefore:

~~~text
js:specifier
    -> JavaScriptImportProvider
    -> GraalJS module-resolution semantics
    -> JS module/object foreign value
~~~

Primary authority:

- https://www.graalvm.org/jdk25/reference-manual/js/Modules/

## TruffleRuby

TruffleRuby values likewise participate in polyglot interoperability, while Ruby library acquisition remains governed by Ruby `require`/load paths/gems rather than a Truffle-wide import API.

Thus:

~~~text
ruby:some_library
    -> RubyImportProvider
    -> Ruby require/load semantics
    -> Ruby module/object foreign value
~~~

The public Ruby polyglot documentation also distinguishes language installation/runtime availability from value interoperability, which supports the same provider boundary.

## Sulong / LLVM and native FFI boundary

Sulong/LLVM and NFI-style/native ABI work should remain a distinct acquisition domain. Shared libraries and symbols can ultimately surface values through interop, but they do not establish that high-level language package imports are generic.

Do not collapse:

~~~text
foreign language/module import
~~~

into:

~~~text
native ABI / shared-library FFI
~~~

merely because both can eventually produce interop values.

## `std:interop` responsibility

`std:interop` remains useful and should share exactly the same foreign-value substrate as imported foreign modules.

Likely capability classes include explicit member read/write/invoke, executable invocation, instantiation, array/hash/iterator operations, explicit conversion/inspection, language/metaobject queries, and bounded privileged acquisition helpers where authority permits.

Not every raw Truffle API should automatically become public Standard Library API. Authority-bearing operations such as host type lookup, arbitrary cross-language eval, native access, or provider registration may need to remain privileged behind a controlled runtime/provider boundary.

## Ordinary Protos syntax versus explicit interop

This matrix is an architectural recommendation, not yet approved language semantics.

| Operation | Recommended allocation |
| --- | --- |
| `foreign.member` | automatic foreign interop where member semantics align |
| `foreign.member(...)` | automatic foreign interop where callable/member semantics align |
| `foreign(...)` | automatic when the receiver is `isExecutable` |
| construction | facade/explicit interop until Protos syntax/semantics are approved |
| `foreign[index]` | automatic where array/hash semantics align |
| indexed/member mutation | semantic decision required before automatic exposure |
| iteration | automatic where iterator semantics align |
| hash/map access | automatic only where key/equality rules are compatible; otherwise explicit interop |
| string/numeric conversion | preserve Protos conversion rules; use explicit interop for lossy/ambiguous cases |
| comparison/order | explicit/provider-specific unless later semantics approve automatic mapping |
| equality/identity | semantic decision required |
| exception handling | runtime/provider translation into the Protos Error contract |
| language/metaobject inspection | explicit `std:interop` |

## Callback direction

Truffle already has a generic executable-value protocol, so the technical substrate for a future Protos Closure callback exists. Existing Protos authority does not permit it yet.

A later semantic/platform decision must answer at least:

- which Actor owns a foreign-triggered callback;
- which Task/execution context runs it;
- whether it may suspend or re-enter;
- receiver/return-home behavior;
- calls from foreign threads;
- cancellation and error propagation;
- callback lifetime and Process/Context escape.

Therefore foreign -> Protos callback support is a real design checkpoint rather than mechanical implementation.

## Identity, lifetime, Actor, Task, and P questions

Foreign values may carry mutable state, external resources, guest Context references, Java classloader references, native handles, or thread affinity. AUD019-A does not silently select transfer rules.

The following require later authority:

~~~text
repeated import identity
foreign module cache identity
foreign object identity
Protos === over foreign values
facade/wrapper identity
foreign mutation
foreign references retained by Protos
Protos references retained by foreign code
cycles across runtimes
Context and classloader lifetime
Task/Actor/P transfer
parallel execution / thread affinity
blocking calls
foreign Promise/Future adaptation
cancellation
exception identity and stack presentation
~~~

## Security / authority

Foreign import is capable of becoming a complete capability bypass if implemented naively.

Examples include host access to filesystem/process/environment/reflection/classloader APIs and guest-language standard libraries such as Python `os`/`subprocess` or JavaScript/Ruby equivalents. Native extensions can further weaken sandbox assumptions.

The embedder therefore must make explicit decisions for at least:

~~~text
allowed language set
HostAccess policy
host-class lookup policy
class loading
native access
filesystem/process/network exposure
reflection
resource/sandbox limits
provider authority
~~~

No implementation should broaden the current Context authority merely to make an interop demo work.

## Context, threading, Native Image, and distribution

Additional languages are part of the Engine/Context runtime plane rather than ordinary Protos libraries loaded independently of it. The provider architecture should therefore be Context/Process-owned, not backed by a global mutable registry/cache.

Protos also allows multiple carrier threads to enter its hosted Context. Foreign providers must not assume every target language/value is safe on every carrier; provider/runtime policy needs room to enforce Context/thread affinity or serialization where required.

GraalVM's JDK 25 embedding guide recommends selecting only the guest languages actually needed for production deployment. Protos should follow that pay-as-you-grow rule rather than automatically bundling GraalPy, GraalJS, TruffleRuby, Espresso, LLVM, and other runtimes into every distribution.

Native Image support must likewise be evaluated per selected language/provider, including reflection/configuration and language-runtime resources.

## Dependency acquisition boundary

Keep these separate:

~~~text
DEPENDENCY ACQUISITION
        !=
IMPORT / BINDING
~~~

Examples:

~~~text
Package Tool -> Maven/JAR provisioning
import("java:...") -> class lookup from the configured runtime

Package/environment -> PyPI/GraalPy provisioning
import("python:numpy") -> Python import from the configured Python runtime
~~~

A future Package Tool may gain ecosystem-specific dependency providers (Maven, PyPI, npm, RubyGems, etc.), but AUD019-A does not design them.

## Candidate comparison

Scale: 1 poor, 3 acceptable, 5 strong.

| Candidate | Cognitive/API cost | Maintenance | Pay-as-you-grow | Robustness | Scalability | Future resilience | Protos fit | Total | Confidence |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | --- |
| A — `std:interop` only | 2 | 4 | 4 | 4 | 2 | 3 | 2 | 21 | High |
| B — hard-coded runtime imports | 3 | 2 | 3 | 3 | 2 | 2 | 2 | 17 | High |
| C — common substrate + runtime providers | 5 | 4 | 5 | 4 | 5 | 5 | 5 | 33 | High |
| D — generated facade/shim everywhere | 4 | 2 | 2 | 3 | 3 | 3 | 3 | 20 | Medium |
| E — raw foreign values only | 4 | 5 | 5 | 3 | 5 | 5 | 3 | 30 | High |
| F — Protos-written providers required | 3 | 2 | 4 | 2 | 3 | 4 | 4 | 22 | Medium |
| G — providers + direct values + optional facade/binding | 5 | 4 | 5 | 5 | 5 | 5 | 5 | **34** | High |

Candidate G combines the necessary per-language acquisition provider with the low-cost direct-value representation and adds facade/binding machinery only when adaptation is actually needed.

## Strongest argument against the recommendation

The hybrid design makes Protos permanently aware of two receiver categories: native Protos values and foreign interop values. Send/call/index machinery must branch or specialize accordingly, and identity, equality, mutation, errors, Actor isolation, and callbacks need explicit policy.

A universal Protos facade appears simpler because everything imported would outwardly be a normal Protos object. The problem is that the facade still has to preserve foreign identity, mutation, exceptions, arrays, iteration, overloads, resources, and callbacks. It moves the semantic differences into wrapper/cache machinery while adding allocation and identity complexity. That cost is not justified as the default.

## Regret scenario and escape path

The main regret scenario is discovering that exposing raw foreign values through ordinary Protos operations produces too many semantic exceptions or unstable language-specific edge cases.

The recommended architecture keeps an escape path: providers can increase their use of runtime-owned or generated facades without replacing the common foreign-value substrate or changing dependency acquisition. Conversely, a library can publish an idiomatic Protos binding above the provider. This means the project can move toward more adaptation later without committing today to a universal wrapper graph.

The opposite migration is harder: if Protos begins with mandatory wrappers everywhere, removing them later would require unwinding wrapper identity, cache, and compatibility behavior. This asymmetry favors the direct-value-biased hybrid.

## Follow-up decisions required before implementation

### Guest semantic decision

A Dxxx-class decision is required for guest-visible behavior, at minimum:

~~~text
foreign participation in member lookup/send
foreign executable call semantics
construction surface
foreign indexing and mutation
iteration/hash semantics
foreign equality and identity
foreign Error projection
foreign-module repeated-import identity
callback semantics
Actor/P transfer semantics
~~~

### Platform/security architecture decision

A PLATxxx-class decision is required for runtime architecture and authority, at minimum:

~~~text
Context language composition
ForeignImportProvider lifecycle/registration
host Java lookup backend
PolyglotAccess / HostAccess / host-class lookup
provider and module cache ownership
Context/thread affinity
foreign callback entry machinery
Native Image / runtime-language provisioning
security enforcement boundary
~~~

AUD019-A intentionally does not allocate or ratify those formal identifiers. The next slice must turn these discovered questions into explicit candidate decision packets and verify whether one or more separate decisions are required before any implementation authorization.

## Smallest-sufficient first implementation route after authority exists

No implementation is authorized yet. If the later Dxxx/PLATxxx decisions approve the recommended architecture, the smallest proof-oriented implementation sequence should be:

1. introduce the internal common foreign-value/provider substrate without broadening guest semantics beyond the approved decision;
2. implement one authorized Java host-class provider using an already-provisioned class such as `java.time.Instant`;
3. prove static call, construction/instance call as applicable, missing-class and denied-class failures;
4. route the same acquired value through the approved explicit `std:interop` primitive surface;
5. only then add one non-Java provider, preferably GraalPy or GraalJS, to prove the provider boundary is genuinely language-specific while the value substrate remains shared;
6. add callback support only after its dedicated semantics/platform authority is in force;
7. extend Package Tool dependency ecosystems separately from runtime import resolution.

This sequence falsifies the architecture cheaply without committing to every guest language or ecosystem at once.

## Final AUD019-A result block

~~~text
AUD019_A_STATUS=COMPLETE

GENERIC_FOREIGN_VALUE_SUBSTRATE=YES

GENERIC_ALL_TRUFFLE_IMPORT_MECHANISM=NO

PER_LANGUAGE_IMPORT_PROVIDER_REQUIRED=YES

JAVA_IMPORT_MECHANISM=
    authorized host-Java class lookup over already-provisioned JVM classes;
    Espresso remains a distinct optional guest-Java provider

TRUFFLE_LANGUAGE_IMPORT_MECHANISM=
    common cross-language/value substrate plus one acquisition provider per
    language or module system, each invoking that language's import semantics

PROTOS_FACING_FOREIGN_FACADE=HYBRID

PROTOS_SOURCE_BINDING_REQUIRED=NO

STD_INTEROP_AND_IMPORT_SHARE_SUBSTRATE=YES

ORDINARY_PROTOS_SYNTAX_OVER_FOREIGN_VALUES=PARTIAL

CALLBACK_PROTOS_TO_FOREIGN=PARTIAL

CALLBACK_FOREIGN_TO_PROTOS=NOT_CURRENTLY_POSSIBLE

IMPORT_RESOLUTION_SEPARATE_FROM_DEPENDENCY_ACQUISITION=YES

SECURITY_AUTHORITY_DECISION_REQUIRED=YES

GUEST_SEMANTIC_DECISION_REQUIRED=YES

PLATFORM_ARCHITECTURE_DECISION_REQUIRED=YES

RECOMMENDED_BASE_ARCHITECTURE=
    retain the existing import(specifier) -> canonical ModuleKey boundary;
    generalize module loading so a key can yield Protos source or a
    provider-acquired foreign value; share one foreign-value substrate between
    import and std:interop; use per-language acquisition providers; preserve
    direct foreign values when they map naturally; add facades/shims only where
    adaptation is actually required; keep Protos/library-specific bindings
    optional

STRONGEST_ARGUMENT_AGAINST_RECOMMENDATION=
    ordinary Protos dispatch becomes permanently foreign-aware and therefore
    requires explicit identity/equality/mutation/error/isolation/callback policy;
    a universal facade would look simpler but would move those differences into
    wrapper identity/cache machinery rather than remove them

NEXT_SLICE=
    prepare the explicit guest-semantic and platform/security decision packets
    discovered by AUD019-A, including dependency ordering and the minimum
    authority needed before implementation may start

NEXT_SLICE_TYPE=INVESTIGATION

NEXT_SLICE_REPOSITORY=guillermomolina/protos
~~~

## Material Protos sources inspected

- `AGENTS.md`
- `AGENTS.work/AUDIT.md`
- `AGENTS.work/COORDINATION.md`
- `pom.xml`
- `spec/semantics/MODULES.md`
- `src/main/java/com/guillermomolina/protos/execution/ProtosModuleResolver.java`
- current module resolver family (`std:`, workspace/package, direct-file, bundled-tool)
- `src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotExecutionContext.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosLanguageContext.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosActorModuleState.java`
- current represented-value `InteropLibrary` exports
- `src/main/java/com/guillermomolina/protos/runtime/ProtosClosureValue.java`
- PLAT013 / GitHub #250
- AUD019 / GitHub #818 and its scope-correction comments

## Publication validation

This record is documentation-only. Before publication its prepared Markdown was checked with `git diff --no-index --check` against an empty baseline; no whitespace errors were reported.
