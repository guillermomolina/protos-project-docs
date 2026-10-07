# TOOL009 — Package Logical Case Resolver Completion Reconciliation

## Identity

- Parent: `guillermomolina/protos#600`
- Cleanup blocker: `guillermomolina/protos#685`
- Prior readiness investigation: `guillermomolina/protos#692`
- Prior Package Logical Case facility publication: `a32acf925383a2b1748bede12586ff3d320b7bf7`
- Publication-hygiene revision visible before this follow-up: `f8cfb52afb2dd77693595fcb2099a21a9a6cc8e0`
- Resolver-completion Protos revision: `2a5412bd60115d5c0ef928499668dada5cd61094`
- Published version: `0.3.73-SNAPSHOT`
- Product commit: `TOOL009: complete Package Logical Case resolver coverage for TOML module graph`

## Result reported by the Human Executor

The bounded follow-up to complete real Package-module resolution for suite-native Logical Case execution was reported implemented and fully validated.

```text
TOOL009_PACKAGE_RESOLVER_COMPLETION = COMPLETE
NEW_DECISION_REQUIRED = NO
PACKAGE_LOGICAL_CASE_REAL_MODULE_GRAPH = SUPPORTED
GENERIC_CLOSURE_GUARDS = UNCHANGED
PACKAGE_CORPUS_MIGRATED = 0
```

The Human Executor reported all requested validation green, including:

```text
git diff --check = PASS
focused Package Logical Case / registry validation = PASS
make test = PASS
```

These PASS statements are Human Executor execution evidence; they are not inferred from source inspection.

## Defect that required the follow-up

The first Package-flavored suite-native Logical Case facility proved Package-specific execution through an exact `package-runtime-names` overlay, but the real Package TOML corpus imports Package Tool internals from direct-file suite sources.

The real graph is:

```text
self:TomlSyntax
  -> tool-shared:Toml10/TomlSyntax

self:TomlDocument
  -> tool-shared:Toml10/TomlDocument
       -> tool-shared:Toml10/TomlSyntax

self:ManifestSchemaV1
  -> self:TomlDocument
       -> tool-shared:Toml10/TomlDocument
            -> tool-shared:Toml10/TomlSyntax
```

A suite source materialized by `ProtosDirectFileModuleResolver` uses a `direct-file:` ModuleKey. The ordinary Test Tool fallback resolver therefore cannot itself accept these Package `self:` / Tool-shared imports through its bundled-tool closure checks.

## Bounded implementation reported

The reported implementation keeps generic closure semantics unchanged and extends the Package Logical Case exact overlay with the five real corpus specifiers:

```text
self:TomlSyntax
self:TomlDocument
self:ManifestSchemaV1
tool-shared:Toml10/TomlSyntax
tool-shared:Toml10/TomlDocument
```

Reported Package-owned canonical keys:

```text
tool001-package:toml-syntax
tool001-package:toml-document
tool001-package:manifest-schema-v1
```

Reported shared canonical keys preserve the existing bundled-tool shared identity:

```text
bundled-tool-shared:Toml10/TomlSyntax
bundled-tool-shared:Toml10/TomlDocument
```

No generic relaxation of `ProtosBundledToolModuleResolver`, `ProtosDirectFileModuleResolver`, or `ProtosExactModuleOverlayResolver` was reported.

## Focal coverage reported

The Human Executor reported new focal coverage that invokes the real Package modules rather than embedded copies and forces the transitive imports:

- real `TomlSyntax` plus `tool-shared:Toml10/TomlSyntax`;
- real `TomlDocument` plus transitive shared `TomlDocument -> TomlSyntax`;
- real `ManifestSchemaV1` with its lazy `self:TomlDocument` import forced from the selected Test body.

The existing Logical Case invariants remain reported preserved:

```text
selected Test.call() = Case authority
exactly one selected Test execution
fresh Case Process
discovery observational
```

## Reported changed product paths

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/test/java/com/guillermomolina/protos/execution/ProtosPackageTestLogicalCaseExecutionFacilityTest.java
```

The reported implementation version is `0.3.73-SNAPSHOT`.

No file under `protos/tests/package-tool/` was reported changed, so the 102-case corpus remains unmigrated.

## Publication verification state

The product publication is now directly visible and revision-bound:

```text
PROTOS_REVISION=2a5412bd60115d5c0ef928499668dada5cd61094
PARENT_PROTOS_REVISION=f8cfb52afb2dd77693595fcb2099a21a9a6cc8e0
VERSION=0.3.73-SNAPSHOT
PRODUCT_PUBLICATION_VERIFICATION=PASS
```

Direct inspection at that exact revision confirms:

- the five real Package TOML exact overlays are present in `ProtosCli.java`;
- the shared Toml10 overlays preserve the existing `bundled-tool-shared:` canonical keys;
- the generic bundled-tool closure guards remain unchanged;
- the dedicated focal tests exercise the real Package modules and their transitive imports;
- the changelog records this as a resolver-completion prerequisite only;
- no Package corpus fixture is migrated by this commit.

The Human Executor's reported green validation is therefore now bound to a concrete published product revision.

## Current migration consequence

The resolver prerequisite is durably published and `main` is ready for the already-selected 102-case migration as one implementation slice:

```text
102 Package Tool TOML cases
-> suite-native
-> one implementation/publication slice
```

The TomlSyntax (27), TomlDocument (31), and ManifestSchemaV1 (44) groupings remain internal checkpoints only, not separate releases.

TOOL009-B / #685 must not perform final global legacy cleanup until all 102 Package cases are suite-native and global production/tool consumers are reconciled again.
