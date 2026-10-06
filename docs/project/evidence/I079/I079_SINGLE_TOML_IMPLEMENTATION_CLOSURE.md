# I079 — Single TOML implementation convergence closure

Status: **CLOSED**

Issue: `guillermomolina/protos#806`

Governing decision: D087 / #372, amended 2026-10-06

Triggering audit: AUD017 / #804

Exact Protos revision:
`a61b2c5bfe5a8615fa269d12cf9cfc198b6d6f25`

Implementation version: `0.3.239-SNAPSHOT`

Validation provenance: maintainer-reported local validation on the published
candidate:

```text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
```

## Closure result

I079 implements the owner-approved AUD017/D087 convergence and leaves one TOML
implementation in Protos.

The surviving implementation is:

```text
std:toml/TOML
    -> protos/lib/toml/TOML.protos
```

The public TOML API now provides explicit dialect selection through:

```text
TOML.parseDialect(text, "1.0")
TOML.parseDialect(text, "1.1")
```

while the existing:

```text
TOML.parse(text)
```

retains its TOML 1.1 meaning.

The implementation uses one parser with dialect-conditional rules rather than a
second parser or neutral/private compatibility engine.

## Persisted-tool consumers

At the exact closure revision:

- Package Tool `ManifestSchemaV1` imports `std:toml/TOML` and parses persisted
  manifest text with `TOML.parseDialect(text, "1.0")`.
- Test Tool `ResourceRequirements` imports `std:toml/TOML` and parses D077
  persisted requirements with TOML 1.0.
- Test Tool `ResourceCatalog` imports `std:toml/TOML` and parses D077 persisted
  catalog text with TOML 1.0.
- Package/Test Tool schema validation remains owned by the tools rather than by
  the TOML library.

The current persisted schemas therefore remain explicitly TOML 1.0 without
forking implementation ownership.

## Dialect compatibility retained

The same parser now retains the material TOML 1.0 / TOML 1.1 distinctions that
motivated the migration work:

- TOML 1.1 accepts `\e`; TOML 1.0 rejects it.
- TOML 1.1 accepts `\xHH`; TOML 1.0 rejects it.
- TOML 1.1 accepts multiline inline tables; TOML 1.0 rejects them.
- TOML 1.1 accepts comments inside inline tables; TOML 1.0 rejects them.
- TOML 1.1 accepts inline-table trailing commas; TOML 1.0 rejects them.
- TOML 1.1 accepts temporal values with omitted seconds; TOML 1.0 rejects them.
- TOML 1.0 parsing is no longer artificially limited to signed 64-bit Integers;
  schema owners may still impose their own independent numeric constraints.

The retained conformance coverage includes:

```text
protos/tests/conformance/library/toml/parser-dialects.protos
protos/tests/conformance/library/toml/parser-toml10.protos
```

The former private-parser behavioral corpus was migrated to the public API in
TOML 1.0 mode instead of being discarded.

## Private implementation retirement

The closure revision removes:

```text
protos/tools/shared/Toml10/TomlSyntax.protos
protos/tools/shared/Toml10/TomlDocument.protos
protos/tools/package/TomlSyntax.protos
protos/tools/package/TomlDocument.protos
```

The Package Tool exact-module overlays that existed only to expose those TOML
modules are also removed from `ProtosCli`.

The generic `tool-shared:` resolver namespace remains available for unrelated
future/internal uses, but it no longer owns TOML.

The Package Tool continues to resolve `std:toml/TOML` through bundled-tool
Standard Library resolution rather than through project package resolution.

## Maintained documentation

The implementation reconciles the maintained Package Tool design documents with
the amended D087 architecture, including:

```text
docs/design/PACKAGE_TOOL_ARCHITECTURE.md
docs/design/PACKAGE_MANIFEST_FORMAT.md
docs/design/PACKAGE_MANIFEST_SCHEMA_V1.md
```

Historical records that describe the former private TOML engine remain historical
evidence rather than live architecture.

## Closure matrix

```text
ONE_TOML_IMPLEMENTATION=PASS
PUBLIC_TOML_11_COMPATIBILITY=PASS
PERSISTED_TOOL_TOML_10_COMPATIBILITY=PASS
PACKAGE_TOOL_MIGRATED=PASS
RETAINED_TEST_TOOL_TOML_CONSUMERS_MIGRATED=PASS
PROJECT_PACKAGE_RESOLUTION_BOOTSTRAP_INDEPENDENCE=PASS
TOOL_SHARED_TOML10_PRODUCTION_REFERENCES=0
PRIVATE_TOML_IMPLEMENTATION_REMOVED=PASS
OBSOLETE_TOML_SPECIFIC_RESOLVER_WIRING_REMOVED=PASS
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
I079_STATUS=CLOSED
NEXT_I079_SLICE=NONE
```

No further I079 implementation or investigation slice is required.
