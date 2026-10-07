# AUD019-A — Foreign-library interop and polyglot import base-architecture audit

Status: **COMPLETE — BASE ARCHITECTURE RESEARCH ONLY**

Parent issue: `AUD019 / guillermomolina/protos#818`

Research date: **2026-10-07**

Audited Protos revision:

```text
PROTOS_REVISION=8bd5c37c7cca13599df39d18de2eadf01e4a86eb
PROTOS_VERSION=0.3.256-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
```

This record preserves the AUD019-A investigation. It does not ratify the
recommended architecture, define new Protos semantics, broaden HostAccess or
PolyglotAccess, or authorize implementation.

During later coordination the Protos default branch advanced beyond the audited
revision. The findings below remain revision-coupled to the exact baseline above;
no claim is made that commits published after that revision were re-audited.

## Scope corrections consumed

AUD019-A consumed both owner corrections already recorded on `#818`:

1. `PLAT410 / #659` and `PLAT411 / #660` are invalid references and were ignored.
   The verified prior interop authority is `PLAT013 / #250`.
2. Protos-written bindings are optional. Runtime-owned, generated, synthetic,
   language-provider, direct-foreign-value and hybrid facades were treated as
   first-class candidates.

The investigation therefore optimized for:

```text
USER_SURFACE =
    ordinary import(...) + natural Protos-facing use

IMPLEMENTATION =
    smallest faithful/efficient bridge regardless of implementation language
```

## Executive finding

The fundamental split is:

```text
GENERIC FOREIGN VALUE PROTOCOL
    YES — Truffle InteropLibrary

GENERIC "IMPORT MODULE X FROM LANGUAGE Y" PROTOCOL
    NO
```

Truffle standardizes operations over already-acquired foreign values. It does
not standardize the package/module acquisition semantics of Python, JavaScript,
Ruby, Java host classes, or other guest ecosystems.

The recommended base architecture is therefore:

```text
                         Protos source
                              |
                    import("scheme:target")
                              |
                              v
                     Protos module resolver
                              |
                              v
                    ForeignImportProvider
                 /          |          \
              Java       GraalPy      GraalJS ...
                |            |            |
                +------------+------------+
                             |
                    acquired foreign value
                             |
               adapt/facade only if needed
                             |
                             v
                  Protos-facing imported value

std:interop
     |
     +-------- same foreign-value substrate --------+
                                                     |
                                    InteropLibrary / Env / Context
```

The representation policy is deliberately hybrid:

```text
DIRECT FOREIGN VALUE
    preferred when the foreign value maps cleanly

LANGUAGE-PROVIDER / RUNTIME FACADE
    used when module acquisition or semantic adaptation requires one

GENERATED SHIM
    optional optimization/adaptation mechanism

LIBRARY-SPECIFIC BINDING
    optional idiomatic last mile

PROTOS-SOURCE BINDING
    allowed but not required
```

No implementation is authorized because the investigation exposes both
guest-visible semantic decisions and durable platform/security decisions.

# 1. Current Protos inventory

## 1.1 Module/import semantics

`spec/semantics/MODULES.md` is the normative owner.

Current Core distinguishes:

```text
module specifier
    exact semantic String supplied by the program

ModuleKey
    canonical host-resolved internal identity

module instance
    Actor-local moduleContext returned by import(...)
```

The exact specifier meaning is host-defined. Once resolution produces a
canonical `ModuleKey`, Core owns Actor-local caching, cache-before-execute,
cycles, initialization, repeated-import identity and failure/retry behavior.

Current `import(specifier)` returns the real module instance/moduleContext; there
is no separate namespace wrapper.

This is a useful boundary for foreign import because schemes such as `java:` and
`python:` can remain host/module-resolution policy. The current loader result,
however, is too narrow.

## 1.2 Current resolver interface

At the audited revision:

```java
public interface ProtosModuleResolver {
    ProtosModuleKey resolve(
        String exactSpecifier,
        Optional<ProtosModuleKey> importingModule) throws Exception;

    ProtosModuleSource loadSource(ProtosModuleKey key) throws Exception;
}
```

This assumes every successfully resolved module ultimately provides Protos
source.

Foreign imports need a generalized load result concept, for example:

```text
Resolved/loadable module
    |
    +-- Protos source module
    |
    +-- foreign module acquisition/provider result
```

The exact public/internal type is implementation work after the required design
authority. AUD019-A does not ratify a Java class name or API.

## 1.3 Module identity/cache

Current Core requires:

```text
same Actor
+ same canonical ModuleKey
=
same active module instance
```

Across Actors, module instances are distinct.

Foreign imports create unresolved semantic questions:

- Does repeated `import("python:numpy")` preserve current module-instance
  identity exactly?
- Does each Actor receive a distinct Protos-facing facade over a shared guest
  module?
- Does the guest language itself create distinct module state?
- How does a raw foreign object's identity interact with Protos `===`?

These cannot be selected mechanically.

## 1.4 Standard Library/workspace/package resolver structure

Current runtime already has multiple resolver implementations/domains including:

- Standard Library `std:`;
- bundled tool `self:` resolution;
- workspace/package `self:` and `dep:` resolution;
- direct-file resolution;
- package execution-plan resolution.

That is evidence that scheme/domain routing already belongs at the resolver
boundary rather than the parser.

Foreign providers should extend that architectural boundary rather than invent a
second import language construct.

## 1.5 Current Graal Context

At the audited revision Protos opens the Polyglot Context approximately as:

```java
Context.newBuilder(ProtosLanguage.ID)
    .in(in)
    .out(out)
    .err(err)
    .allowIO(sourceReadability.ioAccess())
    .build();
```

and initializes only the Protos language.

The foreign-import feature therefore does not currently have:

- general host-class lookup enabled for user imports;
- a public `HostAccess` policy selected for this feature;
- a public `PolyglotAccess` policy selected for this feature;
- additional guest languages configured as ordinary Protos runtime
  dependencies.

This is not merely a missing Standard Library wrapper.

## 1.6 `TruffleLanguage.Env`

`ProtosLanguageContext` owns the current `TruffleLanguage.Env`.

Current implementation uses it for Protos file/source materialization and
`env.parsePublic`. The current `parsePublic` helper rejects source whose language
ID is not Protos.

The Env is therefore present as internal platform substrate, but cross-language
acquisition/evaluation is not an existing Protos feature.

## 1.7 Existing `InteropLibrary` projection

`PLAT013 / #250` selected:

```text
semantic-value-native InteropLibrary projection
+
synthetic/contextual adapters only at genuine non-value boundaries
```

Important constraints include:

- real Protos values are authoritative interop receivers;
- no universal wrapper graph;
- ordinary Protos object interop inspection exposes local slots only;
- interop inspection must not invoke Closure-valued members;
- Protos identity/equality remains Protos-defined;
- unsupported facets fail closed;
- `Closure` `InteropLibrary.execute` was explicitly deferred.

Current runtime value classes such as String, Boolean, Null, numbers and Array
export faithful InteropLibrary facets directly.

## 1.8 Closure/callback state

`ProtosClosureValue` is not currently a generally executable Truffle foreign
value. Protos invokes Closures through its own invocation machinery.

Therefore:

```text
FOREIGN -> PROTOS CLOSURE CALLBACK
    not currently authorized/implemented

PROTOS -> FOREIGN VALUE PASSING
    substrate partially exists through PLAT013 projections,
    but the cross-language execution path is not wired
```

Adding `InteropLibrary.execute` to Closure would establish a new public entry
path into Protos execution and cannot be treated as a mechanical PLAT013
extension.

## 1.9 Ordinary operations over foreign values

Current ordinary Protos send/call/index semantics are not a generic
InteropLibrary-consuming foreign-value surface.

The target-language side of interop is therefore missing.

## 1.10 Actor / Task / P

Current Protos has strong Actor/Process/Context ownership and isolation rules.
Foreign objects can carry:

- mutable guest state;
- Context ownership;
- Java classloader references;
- thread-affine state;
- native handles;
- resources;
- callbacks into Protos.

No existing authority establishes that arbitrary foreign values may cross Actor
or P boundaries.

## 1.11 Native Image

The audited POM includes a Native Image path and PLAT045 fallback-runtime policy.
Foreign language components and reflective/host access therefore have
distribution/AOT consequences. Support must remain pay-as-you-grow rather than
silently bundling every guest runtime.

# 2. Upstream Truffle/GraalVM facts

Primary upstream authority for this audit is the GraalVM/Truffle line pinned by
the audited Protos HEAD: `25.4.4.1.1`.

## 2.1 InteropLibrary: value protocol, not module resolver

`InteropLibrary` standardizes capabilities on values, including materially:

- primitive/null/string/number facets;
- members;
- executability;
- instantiation;
- arrays;
- hash entries;
- iterators;
- exceptions;
- metaobjects;
- identity/language/display/source-location facets.

Representative messages include:

```text
readMember / writeMember / invokeMember
execute
instantiate
readArrayElement / writeArrayElement
readHashValue / writeHashEntry
getIterator / iteratorNextElement
isIdentical / identityHashCode
```

Truffle's model is explicitly two-sided:

```text
source language/value
    exports interop messages

target language
    decides how those messages participate in its own semantics
```

This is exactly why a Protos foreign-value receive/dispatch layer is still
needed.

Primary source:

- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/interop/InteropLibrary.html

## 2.2 `TruffleLanguage.Env`

The Env provides host and cross-language platform facilities, including host
symbol lookup and public/internal parsing where access policy permits.

These primitives are sufficient to implement provider acquisition mechanics,
but they do not define a universal package/module system.

Primary source:

- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/TruffleLanguage.Env.html

## 2.3 Polyglot Context access is separately configured

Polyglot embedding separates:

- language availability;
- `HostAccess`;
- host-class lookup;
- I/O/native access;
- cross-language/polyglot access;
- Context lifetime.

Primary source:

- https://www.graalvm.org/jdk25/reference-manual/embed-languages/

That separation matters because `import("java:...")` must not imply
`allowAllAccess(true)`.

# 3. Java

## 3.1 Host Java is the smallest backend for `java:`

For a conceptual import:

```protos
Instant = import("java:java.time.Instant")
Instant.now()
```

the smallest platform mechanism is authorized host-Java class lookup, not an
Espresso guest JVM.

Conceptual provider route:

```text
java:java.time.Instant
    ->
JavaImportProvider
    ->
authority check
    ->
Env.lookupHostSymbol("java.time.Instant")
    ->
host-class interop value
```

A Java class may therefore be returned as the acquired foreign value directly.
A universal Protos wrapper is not required merely to access ordinary Java
members.

## 3.2 Espresso is a distinct model

Espresso represents Java as a Truffle guest language with its own guest-JVM
semantics. It should not silently become the meaning of ordinary `java:` host
library import.

If exposed later, host Java and guest Java should remain distinguishable.

Primary source:

- https://www.graalvm.org/jdk25/reference-manual/espresso/interoperability/

## 3.3 Generic JAR consumption is feasible

Generic Java interop should not require one handwritten binding per library.

Once a JAR is already provisioned on the runtime class/module path and authority
permits the class, generic lookup/member/call/instantiate interop can cover:

- classes;
- static fields/methods;
- constructors;
- instance fields/methods;
- arrays;
- primitive/wrapper/String/null values;
- enums;
- overloaded methods/constructors and varargs through host interop rules;
- functional-interface/SAM adaptation only after Protos callback executability
  is legitimately defined.

Library-specific Protos bindings remain useful for idiomatic adaptation, not as
a prerequisite for basic Java library access.

External precedent:

- Clojure Java interop:
  https://clojure.org/reference/java_interop

# 4. Other Truffle languages

## 4.1 GraalPy

Once a Python value is acquired, its functions/classes/modules/objects can use
the generic InteropLibrary value protocol.

Acquiring `numpy`, however, is Python import semantics. A Python provider must
perform language-specific acquisition, conceptually through Python's module
loader/import machinery.

Therefore:

```text
python:numpy
    ->
PythonImportProvider
    ->
Python module acquisition
    ->
GraalPy module object
    ->
generic foreign-value substrate
```

Package/environment provisioning remains a separate concern.

## 4.2 GraalJS

A JavaScript object/function returned from cross-language evaluation is a generic
foreign value.

ES module/package acquisition remains GraalJS/JavaScript-specific. Embedded JS
module resolution and Node package/builtin semantics are not one universal
Truffle import API.

Primary module source:

- https://www.graalvm.org/jdk25/reference-manual/js/Modules/

## 4.3 TruffleRuby

Ruby values can participate in generic polyglot interop once acquired. Ruby
library loading remains Ruby-specific (`require`/load-path/gem semantics).

Primary source:

- https://www.graalvm.org/jdk24/reference-manual/ruby/Polyglot/

The versioned documentation URL is older than the Protos GraalVM line; it is
used here only for the stable architectural distinction between polyglot value
exchange and Ruby's own library-loading semantics, not to assert a
25.4.4.1.1-specific API signature.

## 4.4 Sulong/LLVM/native boundaries

LLVM/NFI/native symbol acquisition is materially different from high-level
language module import. Native ABI interop must remain a separate architecture
question even if resulting values ultimately participate in a common foreign
substrate.

FFM/native precedent:

- https://www.graalvm.org/jdk25/reference-manual/native-image/native-code-interoperability/ffm-api/

# 5. Generic substrate vs generic import

Explicit answer:

```text
GENERIC_FOREIGN_VALUE_SUBSTRATE=YES

GENERIC_ALL_TRUFFLE_MODULE_IMPORT=NO

PER_LANGUAGE_IMPORT_PROVIDER_REQUIRED=YES
```

The right reusable unit is:

```text
ONE_GENERIC_INTEROP_SUBSTRATE
+
ONE_IMPORT_PROVIDER_PER_LANGUAGE_OR_MODULE_SYSTEM
```

not:

```text
ONE_GENERIC_TRUFFLE_IMPORT_PROVIDER
```

# 6. Facade/shim architecture

## Candidate representation A — direct foreign object

```text
import(...)
    -> provider
    -> actual foreign value
```

Strengths:

- minimal allocation;
- preserves foreign identity;
- avoids wrapper cache;
- lets Truffle specialize directly on the receiver;
- scales well as new foreign object kinds appear.

Weaknesses:

- ordinary Protos operations need a defined foreign branch;
- identity/equality/mutation/error semantics cannot be inferred automatically;
- some foreign APIs remain non-idiomatic.

Result: **preferred default substrate**.

## Candidate representation B — runtime-owned proxy/facade

```text
import(...)
    -> ProtosForeignModuleFacade
         -> foreign target
```

Strengths:

- can normalize module-facing behavior;
- can hide provider machinery;
- can carry canonical provider/module metadata;
- can deliberately adapt awkward foreign semantics.

Weaknesses:

- new wrapper identity questions;
- additional allocations/cache/lifetime;
- risks recreating a universal parallel object model;
- can obscure actual foreign identity.

Result: **use only when module/adaptation semantics require it**.

## Candidate representation C — generated shim

A provider can generate specialized adapter data/AST/bytecode/runtime objects
from foreign metadata.

Potential advantages:

- stable member surfaces may specialize well;
- can create idiomatic mappings where metadata is strong.

Costs:

- generation/cache/versioning complexity;
- Native Image/dynamic-code concerns;
- not required for basic interop.

Result: **optional optimization/adaptation technique, not foundational**.

## Candidate representation D — language-provider facade

Each language provider may acquire and optionally adapt its native module/type
model behind a common Protos-facing result contract.

Result: **required at acquisition boundary; facade itself optional per provider**.

## Candidate representation E — library-specific binding

Useful only when a library's API cannot map reasonably or idiomatically through
the generic foreign surface.

Result: **optional last mile**.

## Candidate representation F — Protos-written binding

Useful for idiomatic adaptation and potentially some provider logic once a
minimal privileged bridge exists.

Result:

```text
PROTOS_WRITTEN_BINDINGS_REQUIRED=NO
PROTOS_WRITTEN_BINDINGS_ALLOWED=YES
PROTOS_WRITTEN_BINDINGS_PREFERRED=NOT_PRESELECTED
```

## Recommended representation — hybrid

```text
PROTOS_FACING_FOREIGN_FACADE=HYBRID
```

Rule:

> Preserve the actual foreign value when Truffle already exposes the structure
> Protos needs. Introduce a facade only where module acquisition or deliberate
> semantic adaptation requires one.

# 7. `std:interop`

`std:interop` remains useful even when ordinary Protos syntax handles natural
foreign operations automatically.

Potential explicit classes of operation include:

- member introspection/read/write/invoke;
- execute/instantiate;
- arrays and hashes;
- iteration;
- metaobject/language discovery;
- explicit primitive conversion;
- explicit host-type lookup;
- privileged guest-language parse/eval/acquisition primitives where public
  authority eventually permits them.

The public Standard Library API should not necessarily mirror every raw
InteropLibrary message one-for-one. Authority-bearing primitives may need to
remain runtime/provider internal.

Architectural conclusion:

```text
STD_INTEROP_AND_IMPORT_SHARE_SUBSTRATE=YES
```

# 8. Ordinary syntax vs explicit interop

This is an architectural recommendation pending semantic authority, not a
ratified language contract.

| Operation | AUD019-A recommended class |
| --- | --- |
| `foreign.member` | automatic foreign interop where member semantics map |
| `foreign.member(...)` | automatic foreign interop |
| `foreign(...)` | automatic when executable, pending call semantics |
| foreign construction | facade/explicit interop unless ordinary construction semantics are approved |
| `foreign[index]` | automatic where array/hash semantics align |
| `foreign[index] = value` | semantic decision required |
| `foreign.member = value` | semantic decision required |
| iteration | automatic foreign interop when iterator protocol exists |
| map/hash access | automatic only where Protos key semantics remain sound; otherwise explicit |
| equality | semantic decision required |
| identity / `===` | semantic decision required |
| comparison/order | explicit/provider-specific unless separately defined |
| String/number conversion | ordinary Protos conversion rules first; explicit for ambiguous/lossy conversion |
| exception handling | provider/runtime translation into approved Protos Error contract |
| metaobject/language inspection | explicit `std:interop` |

# 9. Callbacks

Truffle already has a generic executable-value protocol, but Protos Closure
executability was deliberately deferred by PLAT013.

To support:

```text
Java/Python/JS/etc.
    -> callback
    -> Protos Closure
```

Protos needs an approved foreign-call entry contract covering at least:

- owning Actor;
- Task/invocation context;
- receiver;
- return behavior;
- Error translation;
- reentrancy;
- callback from a foreign-created thread;
- suspension/cancellation;
- Context entry/lifetime;
- captured Protos object lifetime.

Therefore:

```text
CALLBACK_PROTOS_TO_FOREIGN=PARTIAL
CALLBACK_FOREIGN_TO_PROTOS=NOT_CURRENTLY_POSSIBLE
```

This is a real Dxxx/PLATxxx gate, not mechanical implementation.

# 10. Identity, lifetime, mutation and concurrency

The following are **SEMANTIC_DECISION** questions:

- repeated foreign import identity;
- relationship of foreign identity to Protos `===`;
- observable wrapper/facade identity;
- ordinary equality/comparison;
- ordinary foreign mutation;
- Error/failure identity;
- guest-visible transfer restrictions.

The following are primarily **PLATFORM_DECISION** questions:

- Context ownership/lifetime;
- provider registry ownership;
- classloader lifetime;
- guest-language runtime lifetime;
- thread affinity;
- callback entry machinery;
- blocking-call integration;
- foreign async/future integration;
- cancellation plumbing;
- Native Image language/resource composition.

The following become **MECHANICAL_IMPLEMENTATION** only after those authorities:

- specialized InteropLibrary dispatch nodes;
- provider lookup caches;
- class lookup caches;
- concrete module load-result classes;
- diagnostics formatting;
- generated adapter internals consistent with the approved contract.

# 11. Dependency acquisition boundary

Keep these problems separate:

```text
DEPENDENCY ACQUISITION
    Package Tool / project/environment provisioning

IMPORT / BINDING
    runtime provider resolves an already-provisioned class/module
```

Examples:

```text
Maven/package layer
    -> makes jackson.jar available

import("java:com.fasterxml.jackson.databind.ObjectMapper")
    -> Java class lookup
```

```text
Python environment provisioning
    -> installs numpy

import("python:numpy")
    -> Python module acquisition
```

A future Package Tool provider model for Maven/PyPI/npm/RubyGems is plausible,
but AUD019-A does not design it.

```text
PACKAGE_TOOL_FOREIGN_DEPENDENCY_EXTENSION_REQUIRED=DEFER
```

# 12. Security / authority

A naive foreign import can bypass Protos capability boundaries completely.

Examples include:

```text
java.io.File
java.nio.file.Files
ProcessBuilder
System.getenv
System.getProperties
reflection
ClassLoader
native loading

Python os/subprocess
Node filesystem/child_process
Ruby File/system
```

GraalVM separates HostAccess, host-class lookup, I/O, native access and
cross-language configuration for good reason.

The security decision must cover at least:

- enabled guest-language set;
- allowed Java classes/packages;
- HostAccess;
- host-class lookup;
- class loading;
- native access;
- filesystem/process/network exposure;
- guest standard-library authority;
- reflection;
- environment/system properties;
- resource/sandbox policy.

Do not implement foreign import by enabling unrestricted access.

```text
SECURITY_AUTHORITY_DECISION_REQUIRED=YES
```

# 13. Optimization / cost

`InteropLibrary` is a Truffle Library intended for cached/specialized dispatch.

That favors:

```text
Protos foreign-aware operation node
    -> cached InteropLibrary
    -> actual foreign receiver
```

over a universal:

```text
foreign receiver
    -> generic wrapper graph
    -> synthetic Closure/member table
    -> generic Protos send
```

Base recommendation:

- preserve raw foreign values on the common path;
- cache provider/module acquisition separately;
- specialize stable foreign operation sites;
- avoid mandatory wrappers;
- allow generated shims only where evidence proves value;
- load/bundle guest languages only when selected.

No quantitative performance claim is made; AUD019-A ran no benchmark or spike.

# 14. External architecture lessons

## Clojure/JVM

Generic Java classes, methods and constructors can be consumed without one
binding per library. Optional wrappers provide idiomatic APIs where desired.

Lesson: generic platform interop plus optional high-level binding scales better
than binding-per-library as the foundation.

## JRuby/Jython/Kotlin/Scala

Hosted-language ecosystems similarly distinguish generic platform objects from
library-specific idiomatic facades. Their exact mechanics are not copied because
Protos/Truffle has a different object model.

## .NET/DLR

Shared dynamic-dispatch infrastructure does not imply one module/package system.
Languages retain language-owned acquisition and semantics.

## Python/Ruby native integration

Native-extension systems demonstrate the opposite end: library-specific binding
is still appropriate when a foreign ABI cannot expose a rich generic dynamic
object protocol.

Conclusion: keep library bindings possible, but do not make them the default for
Java/Truffle object ecosystems.

# 15. Base-architecture candidates

## A — `std:interop` only

Explicit low-level interop, no ordinary foreign import schemes.

Reject as target because it does not satisfy requested import ergonomics.

## B — hard-coded per-language imports in runtime

Mechanically feasible but scales poorly and tangles module resolution with every
foreign ecosystem.

## C — common substrate + runtime-owned providers

Strong candidate. Separates common foreign operations from language-specific
acquisition.

## D — common substrate + generated facades everywhere

Potentially ergonomic but imposes wrapper/generation cost and identity
complexity even when raw values are already sufficient.

## E — direct foreign values + interop-aware semantics only

Efficient value layer but incomplete by itself because language-specific module
acquisition remains necessary.

## F — Protos-written providers/bindings as architectural requirement

Reject as requirement. It turns implementation-language preference into
architecture and does not remove privileged acquisition/authority needs.

## G — hybrid

```text
common foreign substrate
+
per-language acquisition provider
+
direct foreign value by default
+
facade/generated shim only when adaptation requires it
+
optional library-specific/Protos binding
```

**Recommended, pending project-owner approval.**

# 16. Twelve-dimension comparative scorecard

Scores are 1–5. Confidence reflects evidence quality, not approval.

Abbreviations:

1. Correctness/invariants
2. Protos alignment
3. Present-need proportionality
4. Incremental growth
5. Future-option resilience
6. Scalability
7. Conceptual simplicity
8. Portability/implementation freedom
9. Runtime/resource cost
10. Failure/operability
11. Deferral/reversibility
12. Evidence maturity

| Candidate | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| A interop-only | 4 | 3 | 3 | 2 | 2 | 3 | 4 | 4 | 4 | 4 | 2 | 5 |
| B hard-coded imports | 3 | 2 | 3 | 2 | 2 | 2 | 3 | 2 | 3 | 3 | 2 | 4 |
| C substrate + runtime providers | 4 | 5 | 5 | 5 | 5 | 5 | 4 | 4 | 5 | 4 | 5 | 5 |
| D generated facade everywhere | 3 | 3 | 2 | 3 | 4 | 3 | 2 | 3 | 2 | 3 | 3 | 3 |
| E direct foreign values only | 3 | 4 | 5 | 4 | 4 | 5 | 4 | 4 | 5 | 3 | 4 | 5 |
| F Protos-written required | 3 | 3 | 3 | 3 | 3 | 3 | 3 | 4 | 4 | 3 | 3 | 3 |
| G hybrid | 5 | 5 | 5 | 5 | 5 | 5 | 4 | 5 | 5 | 5 | 5 | 4 |

### Score justifications and confidence

#### A — `std:interop` only

- Correctness 4/HIGH: minimal semantic surface and explicit operations fail
  closed, but does not satisfy import ergonomics.
- Protos alignment 3/HIGH: mechanism-oriented but forces foreignness into user
  code even where transparent use is natural.
- Present need 3/HIGH: cheap technically, insufficient functionally.
- Incremental growth 2/MEDIUM: adding ergonomic imports later requires another
  parallel layer.
- Future resilience 2/MEDIUM: preserves low-level access but not requested
  module architecture.
- Scalability 3/MEDIUM: value operations scale; library acquisition does not.
- Simplicity 4/HIGH: conceptually small.
- Portability 4/HIGH: explicit abstraction can be reproduced elsewhere.
- Runtime cost 4/MEDIUM: little overhead when unused.
- Operability 4/MEDIUM: explicit failures are diagnosable.
- Reversibility 2/MEDIUM: adding import later changes the user model
  substantially.
- Evidence 5/HIGH: explicit interop layers are mature prior art.

#### B — hard-coded runtime imports

- Correctness 3/MEDIUM: can be made correct per language but central runtime
  branches accumulate.
- Protos alignment 2/HIGH: adds privileged language-specific institutions.
- Present need 3/MEDIUM: expedient for first languages but charges maintenance
  immediately.
- Incremental growth 2/HIGH: each new language edits central machinery.
- Future resilience 2/HIGH: tightly couples supported ecosystems.
- Scalability 2/HIGH: poor provider/ecosystem scaling.
- Simplicity 3/MEDIUM: initially simple, globally complex later.
- Portability 2/HIGH: tends to bind platform mechanisms directly into runtime.
- Runtime cost 3/MEDIUM: can be lazy but central infrastructure grows.
- Operability 3/MEDIUM: diagnostics become language-branch-specific.
- Reversibility 2/MEDIUM: extracting providers later is architectural churn.
- Evidence 4/MEDIUM: feasible but not the cleanest architecture.

#### C — common substrate + runtime-owned providers

- Correctness 4/HIGH: cleanly separates acquisition and value use; semantic
  decisions still needed.
- Protos alignment 5/HIGH: one mechanism, local providers, no unnecessary
  binding-per-library institution.
- Present need 5/HIGH: installs only selected providers/languages.
- Incremental growth 5/HIGH: new language is additive.
- Future resilience 5/HIGH: provider boundary survives additional ecosystems.
- Scalability 5/HIGH: acquisition complexity remains local.
- Simplicity 4/HIGH: one extra provider abstraction is justified by real
  language differences.
- Portability 4/MEDIUM: provider concept is portable although individual
  providers are platform-specific.
- Runtime cost 5/MEDIUM: can remain lazy/pay-as-you-grow.
- Operability 4/MEDIUM: provider boundary offers clear failure attribution.
- Reversibility 5/MEDIUM: providers/facades can evolve independently.
- Evidence 5/HIGH: directly matches observed Truffle/value-vs-module split.

#### D — generated facade everywhere

- Correctness 3/MEDIUM: possible but identity/mutation forwarding is delicate.
- Protos alignment 3/MEDIUM: appealing Protos-facing shape but creates a broad
  synthetic layer.
- Present need 2/HIGH: pays wrapper/generation cost even when unnecessary.
- Incremental growth 3/MEDIUM: metadata quality varies by language.
- Future resilience 4/LOW: could adapt broadly, but depends on reliable
  introspection.
- Scalability 3/MEDIUM: cache/generation lifecycle becomes significant.
- Simplicity 2/HIGH: duplicates value surface mechanically.
- Portability 3/MEDIUM: generator strategy may depend on current Truffle/JVM.
- Runtime cost 2/MEDIUM: extra allocation/generation/cache.
- Operability 3/MEDIUM: facade adds another diagnostic boundary.
- Reversibility 3/MEDIUM: raw escape path can mitigate but wrapper identity may
  become compatibility surface.
- Evidence 3/MEDIUM: feasible technique, not proven necessary for base design.

#### E — direct foreign values only

- Correctness 3/HIGH: preserves actual foreign identity but cannot solve module
  acquisition or mismatched semantics alone.
- Protos alignment 4/HIGH: minimal mechanism, no wrapper universe.
- Present need 5/HIGH: smallest value-level substrate.
- Incremental growth 4/HIGH: provider/adaptation layer can be added.
- Future resilience 4/MEDIUM: relies on keeping explicit escape/adaptation.
- Scalability 5/HIGH: Truffle value protocol scales naturally.
- Simplicity 4/HIGH: small value model, but target-language dispatch branches
  remain.
- Portability 4/MEDIUM: semantic concept is portable though implementation uses
  Truffle today.
- Runtime cost 5/HIGH: minimum wrapper overhead.
- Operability 3/MEDIUM: raw foreign errors/shape need translation policy.
- Reversibility 4/MEDIUM: facades can later wrap only specific boundaries.
- Evidence 5/HIGH: directly supported by Truffle architecture and PLAT013's
  wrapper-avoidance precedent.

#### F — Protos-written providers/bindings required

- Correctness 3/MEDIUM: still needs privileged bridge and authority.
- Protos alignment 3/MEDIUM: self-hosting is attractive but should not override
  mechanism quality.
- Present need 3/HIGH: creates source-layer obligations not required by raw
  interop.
- Incremental growth 3/MEDIUM: extensible, but security/provider registration
  remains hard.
- Future resilience 3/MEDIUM: language code can evolve but cannot own all host
  integration.
- Scalability 3/MEDIUM: per-provider Protos code can proliferate.
- Simplicity 3/MEDIUM: moves rather than removes complexity.
- Portability 4/MEDIUM: high-level bindings can be portable.
- Runtime cost 4/LOW: may specialize well but not proven.
- Operability 3/MEDIUM: two-layer failures remain.
- Reversibility 3/MEDIUM: can move logic runtime-side later.
- Evidence 3/MEDIUM: feasible in principle, not necessary as architecture.

#### G — hybrid

- Correctness 5/MEDIUM: permits direct identity where faithful and adaptation
  where required; final score depends on later D/PLAT choices.
- Protos alignment 5/HIGH: mechanisms over institutions; wrapper only when
  necessary.
- Present need 5/HIGH: raw path is small; optional complexity is demand-driven.
- Incremental growth 5/HIGH: providers, facades and bindings are additive.
- Future resilience 5/HIGH: does not commit all ecosystems to one acquisition
  or one facade technology.
- Scalability 5/HIGH: language-specific complexity stays local.
- Simplicity 4/HIGH: slightly more conceptual pieces than raw-only, but each
  removes a demonstrated trade-off.
- Portability 5/MEDIUM: durable split is not inherently JVM-only.
- Runtime cost 5/MEDIUM: direct path stays cheap; heavier adaptation is opt-in.
- Operability 5/MEDIUM: provider/facade boundaries give clear attribution if
  errors are normalized deliberately.
- Reversibility 5/MEDIUM: direct/facade decisions can evolve per provider without
  replacing the substrate.
- Evidence 4/HIGH: architecture follows strong evidence; exact Protos semantic
  integration remains unproven until later decisions/spikes.

# 17. Anti-overengineering gate for the recommendation

## Pay for what you need

Programs that never use foreign interop should not load additional language
runtimes, allocate wrapper graphs, or enable broader authority.

A selected provider/runtime component pays its own startup/resource cost.

## Grow as you need

The design can begin with:

```text
common foreign substrate
+ Java provider
+ one representative guest-language provider
```

and later add JS/Ruby/etc. without changing the fundamental model.

Generated shims, library-specific bindings and ecosystem dependency providers
can remain absent until justified.

## Cost of deferral

Deferring exact JavaScript/Ruby providers is cheap: add providers later.

Deferring the provider abstraction itself is expensive if early implementation
hard-codes Java/Python into central module machinery.

Deferring guest-visible identity/equality/callback semantics is necessary, not
cost-saving: implementing before those decisions would accidentally establish
public semantics.

## Smallest sufficient solution

The smallest architecture that satisfies the known target is:

```text
one generic foreign-value substrate
+
one language-provider acquisition boundary
+
direct foreign values by default
+
explicit std:interop escape hatch
```

No generated universal wrapper, dependency manager for every ecosystem, or
library-specific binding framework is required now.

# 18. Strongest argument against the recommendation

The hybrid makes ordinary Protos operations permanently aware that a receiver can
be either native Protos or foreign.

That can create:

- larger dispatch logic;
- identity/equality distinctions;
- different mutation behavior;
- error-translation complexity;
- debugging edge cases;
- provider-specific thread/lifetime restrictions.

A universal Protos facade superficially creates a simpler language-facing story.

The counterargument is that a universal facade does not eliminate those
differences. It must still preserve or translate foreign identity, mutation,
callbacks, exceptions, arrays, overloads, lifetime and authority. It therefore
moves complexity into wrapper/cache machinery while adding allocation and
identity-forwarding cost.

# 19. Regret scenario and escape path

## Regret scenario

After implementation, ordinary direct foreign values prove too semantically
irregular for users: Java overloaded members, Python descriptors, JS module
namespaces and Ruby objects require substantially different behavior, and
foreign-aware ordinary syntax becomes difficult to reason about.

## Escape path

The provider contract can return a provider-owned facade for those ecosystems
without replacing the common substrate:

```text
provider
    raw foreign value
        -> optional stable facade
```

`std:interop` retains access to explicit low-level operations.

Because the base design does not promise that every acquired object must be
wrapped or must remain raw, adaptation can grow where evidence demands it.

The high-cost regret would instead be ratifying raw foreign identity/equality
semantics too early. That is why those questions remain blocked behind Dxxx.

# 20. Required follow-up authority

AUD019-A does not authorize implementation.

At minimum, later authority must cover:

## Guest semantic decision (Dxxx)

- ordinary foreign member lookup/invocation;
- executable/constructible behavior;
- indexing and mutation;
- equality/identity;
- repeated foreign import/module identity;
- Error/exception projection;
- callbacks as guest-visible behavior;
- Actor/P transfer restrictions observable by Protos code.

## Platform/runtime decision (PLATxxx)

- Context language composition;
- provider registry/ownership;
- Java host lookup backend;
- HostAccess/PolyglotAccess/host-class lookup;
- security/authority enforcement;
- Context/thread/lifetime rules;
- callback entry machinery;
- module/provider caching;
- Native Image/runtime component provisioning.

A later LIBxxx can define the public `std:interop` API only after the underlying
semantics/platform authority is sufficient.

A later TOOLxxx/Package Tool item for Maven/PyPI/npm/RubyGems acquisition should
be allocated only when dependency provisioning is actually being designed.

# 21. Smallest sufficient future implementation sequence

This sequence is **not authorized yet**; it is the implementation route that
becomes plausible after D/PLAT decisions:

1. platform substrate/load-result boundary capable of source or foreign module;
2. one generic foreign-operation dispatch substrate;
3. authorized Java host-class provider;
4. one representative non-Java provider, preferably GraalPy, proving that module
   acquisition is provider-specific while value use is generic;
5. minimal `std:interop` surface over the same substrate;
6. callback support only after the Closure foreign-call contract is approved;
7. optional provider facades where concrete API mismatch is observed;
8. additional languages and dependency ecosystems incrementally.

# 22. Future proof/spike matrix

No spike was executed during AUD019-A.

Later implementation must prove at least:

1. Java class lookup, static call, constructor and instance call;
2. real import of one non-Java guest module;
3. ordinary Protos member/call/index/iteration behavior selected by Dxxx;
4. equivalent explicit `std:interop` operations;
5. Protos callback round-trip after callback authority exists;
6. primitive/String/null/array/map/error round-trips;
7. missing class/language/module and authority-denied failures;
8. selected module-cache/identity semantics;
9. selected Actor/P/Task behavior;
10. HostAccess restrictions cannot be bypassed through import;
11. existing ordinary Protos imports remain unchanged;
12. JVM and applicable Native Image/distribution gates.

# 23. Source map

## Protos audited revision

- `pom.xml`
  - https://github.com/guillermomolina/protos/blob/8bd5c37c7cca13599df39d18de2eadf01e4a86eb/pom.xml
- `spec/semantics/MODULES.md`
  - https://github.com/guillermomolina/protos/blob/8bd5c37c7cca13599df39d18de2eadf01e4a86eb/spec/semantics/MODULES.md
- `src/main/java/com/guillermomolina/protos/execution/ProtosModuleResolver.java`
  - https://github.com/guillermomolina/protos/blob/8bd5c37c7cca13599df39d18de2eadf01e4a86eb/src/main/java/com/guillermomolina/protos/execution/ProtosModuleResolver.java
- `src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotExecutionContext.java`
  - https://github.com/guillermomolina/protos/blob/8bd5c37c7cca13599df39d18de2eadf01e4a86eb/src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotExecutionContext.java
- `src/main/java/com/guillermomolina/protos/execution/ProtosLanguageContext.java`
  - https://github.com/guillermomolina/protos/blob/8bd5c37c7cca13599df39d18de2eadf01e4a86eb/src/main/java/com/guillermomolina/protos/execution/ProtosLanguageContext.java
- `src/main/java/com/guillermomolina/protos/runtime/ProtosClosureValue.java`
  - https://github.com/guillermomolina/protos/blob/8bd5c37c7cca13599df39d18de2eadf01e4a86eb/src/main/java/com/guillermomolina/protos/runtime/ProtosClosureValue.java
- `src/main/java/com/guillermomolina/protos/runtime/ProtosActorModuleState.java`
  - https://github.com/guillermomolina/protos/blob/8bd5c37c7cca13599df39d18de2eadf01e4a86eb/src/main/java/com/guillermomolina/protos/runtime/ProtosActorModuleState.java
- resolver implementations:
  `ProtosStandardLibraryModuleResolver`,
  `ProtosWorkspacePackageModuleResolver`,
  `ProtosPackageExecutionPlanV2ModuleResolver`,
  `ProtosBundledToolModuleResolver`,
  `ProtosDirectFileModuleResolver`.
- representative InteropLibrary value implementations:
  `ProtosObjectValue`, `ProtosStringValue`, `ProtosBooleanValue`,
  `ProtosNullValue`, numeric value classes, `ProtosArrayValue`,
  `ProtosBytesValue`.

## GitHub project authority

- AUD019 / #818
  - https://github.com/guillermomolina/protos/issues/818
- PLAT013 / #250
  - https://github.com/guillermomolina/protos/issues/250

The #818 comments correcting the invalid #659/#660 references and the
Protos-binding preference are part of the audit input.

## Upstream

- InteropLibrary
  - https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/interop/InteropLibrary.html
- TruffleLanguage.Env
  - https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/TruffleLanguage.Env.html
- Embedding languages
  - https://www.graalvm.org/jdk25/reference-manual/embed-languages/
- Espresso interoperability
  - https://www.graalvm.org/jdk25/reference-manual/espresso/interoperability/
- GraalJS modules
  - https://www.graalvm.org/jdk25/reference-manual/js/Modules/
- TruffleRuby polyglot guide
  - https://www.graalvm.org/jdk24/reference-manual/ruby/Polyglot/
- Native Image FFM/native interop
  - https://www.graalvm.org/jdk25/reference-manual/native-image/native-code-interoperability/ffm-api/
- Clojure Java interop
  - https://clojure.org/reference/java_interop

# 24. Explicit AUD019-A result

```text
AUD019_A_STATUS=COMPLETE

FIRST_CLASS_STD_INTEROP_RECOMMENDED=YES
FOREIGN_IMPORT_SCHEMES_RECOMMENDED=YES

GENERIC_FOREIGN_VALUE_SUBSTRATE=YES

GENERIC_ALL_TRUFFLE_IMPORT_MECHANISM=NO
GENERIC_ALL_TRUFFLE_MODULE_IMPORT=NO

PER_LANGUAGE_IMPORT_PROVIDER_REQUIRED=YES

JAVA_IMPORT_MECHANISM=
    authorized host-Java class lookup through the Truffle/Polyglot host
    interop substrate over already-provisioned classes; Espresso remains a
    distinct optional guest-Java model

JAVA_IMPORT_BACKEND=
    HOST_JAVA_INTEROP_BY_DEFAULT

TRUFFLE_LANGUAGE_IMPORT_MECHANISM=
    common cross-language/value substrate plus provider-specific module
    acquisition using each language/module system's own semantics

PROTOS_FACING_FOREIGN_FACADE=HYBRID

PROTOS_SOURCE_BINDING_REQUIRED=NO
PROTOS_WRITTEN_IMPORT_PROVIDERS_FEASIBLE=PARTIAL
PROTOS_WRITTEN_LIBRARY_BINDINGS_FEASIBLE=YES

STD_INTEROP_AND_IMPORT_SHARE_SUBSTRATE=YES

ORDINARY_PROTOS_SYNTAX_OVER_FOREIGN_VALUES=PARTIAL
ORDINARY_PROTOS_SYNTAX_CAN_CONSUME_FOREIGN_VALUES=PARTIAL
EXPLICIT_STD_INTEROP_REMAINS_REQUIRED=YES

CALLBACK_PROTOS_TO_FOREIGN=PARTIAL
CALLBACK_FOREIGN_TO_PROTOS=NOT_CURRENTLY_POSSIBLE

IMPORT_RESOLUTION_SEPARATE_FROM_DEPENDENCY_ACQUISITION=YES
PACKAGE_TOOL_FOREIGN_DEPENDENCY_EXTENSION_REQUIRED=DEFER

SECURITY_AUTHORITY_DECISION_REQUIRED=YES
GUEST_SEMANTIC_DECISION_REQUIRED=YES
PLATFORM_ARCHITECTURE_DECISION_REQUIRED=YES

RECOMMENDED_BASE_ARCHITECTURE=
    Preserve import(String) -> canonical ModuleKey resolution; generalize
    module loading so a resolved import can yield Protos source or a
    provider-acquired foreign module value; use one common foreign-value
    substrate with per-language/module-system acquisition providers; retain
    raw foreign values when faithful; introduce provider/runtime facades only
    where module semantics or adaptation require them; expose std:interop as
    an explicit lower-level API over the same substrate; keep generated and
    Protos-written bindings optional.

STRONGEST_ARGUMENT_AGAINST_RECOMMENDATION=
    Ordinary Protos operations acquire a permanent native-versus-foreign
    dispatch boundary and require explicit policy for identity, equality,
    mutation, errors, Actor isolation and callbacks. A universal facade looks
    simpler at the language boundary, but it moves rather than removes those
    distinctions while adding wrapper identity, allocation and cache cost.

NEXT_SLICE=
    AUD019-B — construct the exact semantic/platform decision packets required
    before implementation: guest-visible foreign-value/module semantics,
    callback and isolation behavior, and platform/security authority. Do not
    implement or ratify those decisions in the slice.

NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos

IMPLEMENTATION_AUTHORIZED=NO
```
