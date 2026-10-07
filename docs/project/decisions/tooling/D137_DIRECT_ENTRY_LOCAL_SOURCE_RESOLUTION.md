# D137 — Direct-entry source import resolution semantics

Status: **RATIFIED — Candidate B′ selected**

Allocated: **2026-09-16**

Explicit project-owner approval: **2026-09-18**

Decision issue: `guillermomolina/protos#545`

Nature: durable implementation-independent official CLI/filesystem-host module-resolution contract

Research baseline: `guillermomolina/protos@16d4acc0d7d8026b8ce03139f51baabf36c34aad`

Normative Core language effect: **none**.

Package-resolution effect: **none**.

## Decision

`import(String)` means **module resolution**, not filesystem loading.

A resolver may be backed by a filesystem, package graph, Standard Library, network
location, embedded source store, or another future domain, but the semantic module
model remains:

```text
exact semantic String specifier
        +
importing module identity/environment
        |
        v
resolver
        |
        v
canonical ModuleKey
        |
        v
Actor-local module instance
```

A filesystem path, URL, package spelling, or another locator is therefore not
itself the semantic meaning of `import`.

D137 selects one additional resolver policy for the official direct-file and
debug-file CLI hosts. It does not redefine universal Core module semantics.

## Selected candidate — B′

The official direct-file and debug-file CLI hosts provide an
**importer-relative local-source module namespace** for explicit `./` and `../`
specifier spellings.

The namespace is backed by one confined source tree rooted at the directory of
the selected direct entry.

Conceptually:

```text
protos /path/app/main.protos

selected direct entry
        |
        +-- local-source root = /path/app
        |
        +-- canonical initial ModuleKey(main)
        |
        v
execute main through ordinary cache-before-execute module lifecycle
```

The root and physical host paths are resolver/host implementation facts. They do
not become universal Protos language values or semantic module identity.

## Local specifier contract

The initial direct-file local domain recognizes only explicit relative source
spellings beginning with:

```text
./
../
```

The portable separator in this resolver domain is `/`.

The following do not initially acquire local-file meaning:

```text
helper.protos
/helper.protos
C:\\...
file:...
https:...
```

The absence of those meanings is not a permanent prohibition. Additional
resolver domains may be designed later without changing `import(String)`.

## Importer-relative base

A local specifier is resolved relative to the directory of the **importing local
module**.

It is never resolved relative to the process working directory merely because
the official CLI happens to have one.

Therefore source behavior is stable under launcher-CWD changes.

## Local-source root and confinement

The direct entry's selected containing directory defines the initial
local-source root.

Logical normalization of `.` and `..` is permitted, but the normalized target
must remain inside that root.

```text
root/main.protos -> ./lib/a.protos       PASS
root/lib/a.protos -> ../b.protos         PASS
root/main.protos -> ../outside.protos    ERROR
```

This root is a module-source loading authority, not a general guest Filesystem
capability.

The project owner selected this bounded authority model rather than initially
granting arbitrary parent-filesystem module traversal.

## Exact source selection

The v1 direct local-source resolver performs exact source selection:

```text
IMPLICIT_PROTOS_EXTENSION=NO
DIRECTORY_INDEX_SEARCH=NO
SEARCH_PATH_FALLBACK=NO
CWD_FALLBACK=NO
```

A local target must be an exact source file accepted by the direct-file host.

The resolver does not append `.protos`, probe multiple extensions, search an
`index` convention, or fall through to ambient directories.

## Exact names and case

Portable local-source resolution uses exact component names.

A requested component must match the selected directory entry exactly rather
than relying on host case-folding to manufacture alias equivalence.

Host-native filename implementation remains below the resolver boundary.

## Symbolic links, reparse points and uncertain indirection

The initial B′ contract does not add symbolic-link/reparse/other-indirection
traversal to the local module namespace.

Imported local-source traversal must fail closed on such indirection rather than
follow a path whose containment cannot be established under the selected root.

A future secure alias/symlink model remains a separate decision if real use
requires it.

## Canonical direct-entry identity

The direct entry itself is an importable canonical initial module.

Before its first source expression executes, the official direct-file host must
give it the local resolver's canonical `ModuleKey` and place its module instance
into the owning Actor's module cache in `INITIALIZING` state through the existing
canonical initial-module lifecycle.

This preserves normal module-cycle semantics.

```text
cache main#1 / INITIALIZING
execute main#1
    import B
        cache B#1 / INITIALIZING
        execute B#1
            import main
                -> existing main#1
```

No second `main` module instance is created.

```text
DIRECT_ENTRY_CANONICAL_MODULE_IDENTITY=YES
DIRECT_ENTRY_REIMPORT_ALIASES_INITIAL_INSTANCE=YES
```

The ordinary Core rule remains unchanged: another host may still execute a source
as a standalone non-importable initial entry when no importable canonical
identity exists.

## Canonical local ModuleKey

Specifier spelling remains distinct from canonical module identity.

Within one direct local-source resolver, normalized spellings that select the
same exact root-relative namespace entry produce the same canonical ModuleKey.

D137 does not select the internal encoding of that ModuleKey.

Hard-link/inode/file-key equivalence is not part of the v1 contract. Distinct
exact namespace entries remain distinct ModuleKeys even if a particular host
filesystem maps them to the same underlying file object.

## Cyclic module semantics

D137 does not invent circular-import handling.

It relies on the existing Core module lifecycle:

```text
cache-before-execute
INITIALIZING instance is the real module instance
recursive import returns that same instance immediately
no hidden wait for READY
no second instance for the same canonical ModuleKey
```

Thus ordinary cycles such as `A -> B -> A` remain legal.

A module observed during the cycle exposes exactly the slots already created at
that point. Access to a slot that has not yet been created remains an ordinary
lookup Error.

## Resolver composition

The official direct-file resolver environment composes resolver domains rather
than turning all Strings into filesystem paths.

```text
local importer + ./... or ../...
    -> local-source resolver

std:...
    -> Standard Library resolver

self:... / dep:...
    -> not supplied by direct-file mode;
       package/workspace execution remains their owner

local relative import from a non-local importer
    -> Error
```

A Standard Library module therefore does not accidentally acquire the direct
entry's local-source authority merely because it executes in the same Process.

Package/workspace resolution remains separately owned by its existing logical
package model.

## Locationless entries

The following forms receive no implicit local-source base:

```text
protos -e ...
REPL
stdin
generated/in-memory source without an explicitly selected local-source root
```

They receive no CWD fallback.

They may still use resolver domains explicitly supplied by their host.

## Debug-file parity

`protos debug <file>` uses the same local-source resolution and canonical
direct-entry identity contract as ordinary direct-file execution.

## Authority boundary

D137 grants a module-source loading capability to the official direct-file host.

It does **not** grant the guest program a general Filesystem capability.

This separation matters because `import` is an ordinary runtime operation whose
String may be computed dynamically.

## Why B′ was selected

The exhaustive D137 investigation compared Node.js ESM, Deno, Python, Ruby, Lua,
Io, Rust, Go, Java source-file mode, Swift Package Manager, C# script/file-based
execution, and Self.

The recurring design families were:

```text
importer/source-relative canonical modules
ambient/search-path script loading
explicit package/source roots
separate script/file-loading primitives
```

B′ preserves the strongest existing Protos properties:

- ordinary `import(String)`;
- resolver-mediated canonical ModuleKey identity;
- Actor-local module instances;
- cache-before-execute cyclic initialization;
- separation of Standard Library, package and local-source resolver domains;
- absence of ambient CWD/search-path identity;
- absence of a second `include`/load/require-relative primitive; and
- no universal filesystem institution in Core.

## Alternatives considered

### A — host-defined status quo

Rejected because it leaves an already-demonstrated direct-file composition need
undefined.

### P — package-only composition

Rejected because a trivial two-file program would have to pay for package
manifest/workspace concepts.

### B — unrestricted canonical source-relative direct-file resolution

Strong alternative, rejected because arbitrary relative traversal is easy to add
and hard to retract compatibly. A future confinement requirement would break
programs that relied on unrestricted `../../...` loading.

### B″ — explicit source-root opt-in

Retained as a strong additive future option, not selected initially because it
imposes CLI ceremony before a concrete need for roots wider than the entry
directory has been demonstrated.

### C — universal Core source-relative semantics

Rejected because it makes source-location/filesystem-shaped behavior a universal
language concern for an official filesystem-CLI use case.

### D — mandatory abstract Core resolution context

Rejected because the existing `resolve(specifier, importingModuleKey)` boundary
already preserves the needed escape path. A new universal context would
preimplement speculative generality.

## Strongest argument against B′

The entry directory may not be the programmer's intended source root.

```text
project/
├── bin/
│   └── main.protos
└── lib/
    └── parser.protos
```

Running `project/bin/main.protos` cannot import `../lib/parser.protos` under
B′ because the inferred local root is `project/bin`.

D137 nevertheless selects B′ because the escape path is additive:

```text
future:
protos --source-root project project/bin/main.protos
```

or the program may graduate to package/workspace execution.

Adding wider authority later does not invalidate B′ programs or Core semantics.
Starting with unrestricted authority and later revoking it would be a
compatibility break.

## Incremental-design conclusion

B′ is the smallest solution that simultaneously:

1. supports ordinary neighboring/subtree direct-source composition;
2. keeps resolution importer-relative rather than CWD-relative;
3. preserves one canonical direct-entry identity under cycles;
4. avoids requiring Package Tool for a two-file program;
5. keeps filesystem semantics outside universal Core;
6. avoids granting arbitrary parent-filesystem module authority; and
7. retains additive escape paths for wider future source layouts.

Capabilities deliberately deferred include:

```text
absolute local-file imports
file: resolver domain
https:/network resolver domains
git: resolver domain
implicit .protos extension
directory/index imports
configurable search paths
multiple local roots
explicit wider --source-root
CWD fallback
symlink/reparse imports
hard-link/inode identity
local imports for REPL/-e/stdin
package-relative ./ semantics
general resource loading
```

## Invariant consistency

```text
ordinary import(...)                         PASS
semantic String argument                     PASS
specifier spelling != ModuleKey              PASS
canonical host ModuleKey                     PASS
Actor-local module cache                     PASS
cache-before-execute                         PASS
standalone Core entry remains possible       PASS
std: remains resolver policy                 PASS
self:/dep: remain package policy             PASS
no second local-file loading primitive       PASS
no implicit global namespace                 PASS
no universal filesystem institution in Core  PASS
resolver failures -> Protos Error             PASS
```

```text
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Exact selected contract

```text
D137_SELECTED_CANDIDATE=B_PRIME

IMPORT_MEANING=MODULE_RESOLUTION_NOT_FILESYSTEM_LOADING

OFFICIAL_DIRECT_FILE_LOCAL_IMPORTS=YES
LOCAL_SPECIFIERS=EXPLICIT_DOT_RELATIVE
LOCAL_SEPARATOR=SLASH
LOCAL_BASE=IMPORTING_LOCAL_SOURCE_DIRECTORY
LOCAL_ROOT=SELECTED_DIRECT_ENTRY_DIRECTORY

ROOT_ESCAPE=ERROR
CWD_FALLBACK=NO
IMPLICIT_EXTENSION=NO
DIRECTORY_INDEX_LOOKUP=NO
SEARCH_PATH_FALLBACK=NO

IMPORTED_SYMLINK_REPARSE_TRAVERSAL=NO
EXACT_COMPONENT_CASE=YES
HARDLINK_INODE_IDENTITY=NO

DIRECT_ENTRY_CANONICAL_MODULE_IDENTITY=YES
DIRECT_ENTRY_REIMPORT_ALIASES_INITIAL_INSTANCE=YES

LOCATIONLESS_LOCAL_BASE=NO
DEBUG_FILE_CONTRACT=SAME_AS_DIRECT_FILE

GENERAL_FILESYSTEM_CAPABILITY_GRANTED=NO

STD_RESOLUTION_CHANGED=NO
PACKAGE_RESOLUTION_CHANGED=NO
CORE_MODULE_SEMANTICS_CHANGED=NO

FUTURE_RESOLVER_DOMAINS=DEFERRED_ADDITIVE
```

## Approval provenance

The project owner explicitly approved **D137 Candidate B′** on **2026-09-18**
after the exhaustive comparison and clarification of the module-vs-filesystem
boundary.

The exact owner statement was:

> Aprobada D137 Candidate B′

GitHub Issue `guillermomolina/protos#545` records the exhaustive packet and
approval coordination event.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Governance consequence

D137 is an implementation-independent **tooling/host contract**.

It does not alter normative Core module semantics and does not alter package
semantics.

The consuming implementation must be routed separately and must not broaden,
reinterpret, or silently weaken this selected contract.

If implementation exposes a materially new semantic or architecture choice, that
choice crosses the normal explicit decision gate.

## Ratification closure contract

```text
D137_STATUS=RATIFIED
D137_SELECTED_CANDIDATE=B_PRIME

PROTOS_REVISION=16d4acc0d7d8026b8ce03139f51baabf36c34aad
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs

SPECIFICATION_CHANGED=NO
IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
PACKAGE_SEMANTICS_CHANGED=NO
STANDARD_LIBRARY_CHANGED=NO

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

D137 ratification is governance/documentation-only.
