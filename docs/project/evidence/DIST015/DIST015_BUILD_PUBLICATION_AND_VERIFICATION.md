# DIST015 — Protos 0.3.312 build, publication and independent verification

Status: **COMPLETE**

DIST015 (guillermomolina/protos#849) built, validated and published the
owner-authorized Protos 0.3.312 `JVM_PLUS_NATIVE` prerelease through the
maintained DIST005 pipeline (`dist/prepare_release.py`,
`dist/publish_release.py`), then re-read the public GitHub state.

## Native admission failure and DIST015-FIX

The first candidate, built from the originally approved baseline
`d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae`, failed the Native full Test Tool
admission: the run stalled in suite-native discovery of `library/regex`
(single guest thread blocked, no workers started). The Native Image did not
embed the ICU4J Unicode data that `std:regex/Regex` reads at module load
(LIB014-1); the JVM distribution reads it from the JAR and was unaffected.

The fix, `5c4e2ea8a2fff7b2750e4182de41db8754ce4286` (`DIST015-FIX: embed
ICU4J Unicode data in the Native Image`), only adds
`com/ibm/icu/impl/data/icudata/*.icu` and `com/ibm/icu/ICUConfig.properties`
to the Native Image reachability metadata, plus its CHANGELOG entry. It keeps
the project version at `0.3.312-SNAPSHOT`, so the authorized release version
and tag are unchanged; the selection record was rebaselined onto it.

A diagnostic Native build of the fix ran the full bundled suite with
`--jobs 16`: `2581 passed, 0 failed`.

## Exact release identity

```text
RELEASE_BASELINE_REVISION=5c4e2ea8a2fff7b2750e4182de41db8754ce4286
RELEASE_BASELINE_VERSION=0.3.312-SNAPSHOT
RELEASE_VERSION=0.3.312
RELEASE_TAG=v0.3.312
SPECIFICATION_REVISION=0.1.451
CANDIDATE_SOURCE_REVISION=f8ff34498f3a193c9bfe15f215a181c23504a6ad
```

The candidate is a single-parent release-only commit over the baseline that
changes only the root `pom.xml` version `0.3.312-SNAPSHOT -> 0.3.312`. `main`
was not modified by the release.

## Validation

`dist/prepare_release.py` passed Native complete admission (ELF/glibc/ISA,
relocation, no Java/Maven, Package Tool, Test Tool focal and full suite, REPL,
LSP, DAP, PLAT045 interpreter-only), Portable JVM complete admission, and
multi-asset envelope generation plus independent verification.

Product tests were not re-run for the release-only version change.

## Artifacts

```text
protos-0.3.312-native-linux-x86_64.zip  70f893d225419713b312054f3558e2e4b3f4169e8e7bd37e015aff92157d067b  recommended-first-run
protos-0.3.312-posix-jvm.zip            ab9cc9de494cc9189eb29ad2f2f110e53997c49adcb1d863814ddd5b89ce6d29  compatibility-fallback
RELEASE_MANIFEST.txt                    a1e1db7a1b2a6aa4c7587c136820830c78a08f67ca6fe57fd9fc4a8a8e2727f2
RELEASE_NOTES.md                        696a5363a591bff02c3b246d1d02a0ff8a95cc0bd27d9c8db4bd601b921a516e
```

Native: GraalVM Native Image 25.4.4.1.1, Linux x86_64, glibc 2.39+, dynamic,
compatibility ISA, no external JDK, guest JIT unavailable (PLAT045,
oracle/graal#14579). Portable JVM: GraalVM Community Edition 25.4.4.1.1 for
JDK 25.0.4.1.1, Truffle 25.4.4.1.1, `HotSpotTruffleRuntime`.

## Publication and public verification

`dist/publish_release.py` reported `PUBLIC_TAG_IDENTITY`,
`GITHUB_PRERELEASE_METADATA`, `PUBLISHED_ASSET_SET`,
`PUBLISHED_ASSET_DIGESTS` and `RELEASE_MANIFEST_IDENTITY` all `PASS`.

Independent re-read of the public state:

- `refs/tags/v0.3.312` on origin -> `f8ff34498f3a193c9bfe15f215a181c23504a6ad`;
- Release 407135378, prerelease, not draft, published 2026-10-08T18:20:47Z:
  https://github.com/guillermomolina/protos/releases/tag/v0.3.312
- exactly five assets whose GitHub SHA-256 digests equal the local envelope.

## Follow-ups

- `dist/validate_native.py` captures the full Test Tool output until it
  ends, so a long run is indistinguishable from a stall; it should stream
  progress.
- A Native failure during suite-native discovery stalled the Test Tool
  instead of reporting an error.
- DOC010 can synchronize documentation and web with public version 0.3.312.

Machine-readable identity: `DIST015_PUBLICATION_IDENTITY.txt`.
