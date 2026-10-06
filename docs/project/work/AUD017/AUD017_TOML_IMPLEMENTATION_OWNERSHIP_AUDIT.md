# AUD017 — Package Tool and public TOML implementation-ownership audit

Status: **COMPLETE — consolidation selected**

Issue: `guillermomolina/protos#804`

Audited Protos revision:
`61c475650bf5b5d385e9482b639d310c682e2b0d`

Owner direction recorded: **2026-10-06**

## Question

AUD017 asked whether Package Tool should continue using the private
`tool-shared:Toml10` implementation or converge with the public
`std:toml/TOML` facility.

## Current implementation map

At the audited revision, Package Tool parses through:

```text
protos package manifest
  -> ManifestMain
  -> ManifestCommand
  -> ManifestSchemaV1
  -> self:TomlDocument
  -> tool-shared:Toml10/TomlDocument
  -> tool-shared:Toml10/TomlSyntax
```

The Package-local `TomlDocument` and `TomlSyntax` modules are adapters. The
private parser/document implementation lives under
`protos/tools/shared/Toml10/**`.

The public stack is:

```text
std:toml/TOML
  -> protos/lib/toml/TOML.protos
```

and contains the public semantic constructors, parser/document assembly,
duplicate/conflict handling, TOML 1.1 value handling, temporal model, floats and
encoder.

## Duplication finding

The two implementations materially duplicate common TOML machinery, including:

- lexical scanning and comments;
- quoted strings and escapes;
- numeric/integer tokenization;
- booleans;
- arrays and inline tables;
- tables, dotted keys and array-of-tables;
- document assembly; and
- duplicate/conflict detection.

Package schema projection is not duplication: `ManifestSchemaV1` owns Package
Tool semantics and remains separate.

Public-only features such as encoding, floats and temporal semantic values also
do not justify a second parser.

Historical hardening already demonstrates drift. Public TOML fixes for common
parser concerns do not automatically reach `tool-shared:Toml10`, including
comment/control handling and recursion/stack-safety work.

```text
DUAL_IMPLEMENTATION_DUPLICATION=MATERIAL
```

## Bootstrap finding

The strongest historical reason for D087's private engine no longer describes
current source.

`ProtosBundledToolModuleResolver`:

- resolves `self:` inside the bundled tool;
- resolves `tool-shared:` through the private shared root;
- delegates `std:` directly to `ProtosStandardLibraryModuleResolver`; and
- does not consult project manifests, lock files, package stores, CWD-based
  package discovery, or project package resolution to do so.

Package Tool already consumes Standard Library modules in this bootstrap model.

Therefore:

```text
STD_AVAILABLE_TO_BUNDLED_PACKAGE_TOOL=YES
STD_REQUIRES_PROJECT_PACKAGE_RESOLUTION=NO
DIRECT_STD_IMPORT_CREATES_PROJECT_RESOLUTION_CYCLE=NO
BOOTSTRAP_PREVENTS_ONE_TOML_IMPLEMENTATION=NO
```

The remaining bootstrap requirement is ordinary self-hosting/toolchain bootstrap:
an installation/build may provide the already-built compiler and Standard
Library it needs, or embed those assets. That does not justify maintaining a
second TOML implementation.

## Dialect and schema boundary

The public library currently targets TOML 1.1 while Package Manifest v1 and D077
persisted schemas are pinned to TOML 1.0.

That compatibility constraint remains real but is **not** an implementation
ownership argument.

The selected direction is:

- one parser/library implementation: `std:toml/TOML`;
- existing public TOML 1.1 behavior remains available unchanged;
- persisted tool schemas retain explicit dialect/version pinning;
- Package Manifest v1 and current D077 generations continue to request TOML 1.0
  unless independently changed;
- any TOML 1.0 mode needed by those consumers is implemented inside the same
  public TOML parser, not through a second engine;
- Package/Test Tool schema validation remains outside the TOML library.

## Owner decision

The project owner explicitly rejected keeping two TOML libraries and rejected
introducing a neutral/private/public hybrid merely to preserve the old bootstrap
split.

The owner also clarified the intended bootstrap model: requiring an earlier
Protos installation, an OS-installed toolchain component, or an embedded
Standard Library is ordinary bootstrap and is not itself an architectural
problem.

AUD017 therefore does not schedule another A-F architecture investigation.

The original D087 comparison already researched the relevant ownership families.
AUD017 supplies the changed current-repository evidence that invalidates its
earlier public-`std:toml` rejection, and the owner has selected the new
invariant.

## Audit classification

```text
CURRENT_DUAL_IMPLEMENTATION=REMOVE_PERMANENTLY
CONSOLIDATION_RECOMMENDED=PACKAGE_TOOL_USES_PUBLIC_STD_TOML
ONE_PUBLIC_TOML_IMPLEMENTATION=REQUIRED
PACKAGE_SCHEMA_SEPARATION=PRESERVED
PROJECT_PACKAGE_RESOLUTION_BOOTSTRAP_INDEPENDENCE=PRESERVED
PUBLIC_TOML_API_EXISTENCE=NO_LONGER_HYPOTHETICAL
PRIVATE_TOOL_SHARED_TOML=TO_REMOVE_AFTER_MIGRATION
AUD017_STATUS=COMPLETE
```

"Remove permanently" applies to the **duplicated independent TOML engine**, not
to TOML 1.0 compatibility itself. A single implementation may support multiple
explicit TOML dialects.

## Implementation routing

The next executable work belongs in `guillermomolina/protos`.

It must:

1. preserve existing Package Manifest v1 and D077 schema semantics;
2. preserve existing public TOML 1.1 behavior;
3. add/use the minimum explicit TOML 1.0 parsing mode required by persisted tool
   schemas inside the one public TOML implementation;
4. migrate Package Tool and every remaining bundled-tool TOML consumer from
   `tool-shared:Toml10` to `std:toml/TOML`;
5. retain schema-specific diagnostics/validation at the tool layer;
6. remove the private TOML implementation, obsolete adapters, tests and
   TOML-specific resolver wiring when no consumer remains;
7. retain or adapt conformance/regression coverage so common parser fixes have
   one implementation owner; and
8. demonstrate that bundled Package Tool execution still resolves the Standard
   Library without invoking project package resolution.

No further AUD017 research slice is required.
