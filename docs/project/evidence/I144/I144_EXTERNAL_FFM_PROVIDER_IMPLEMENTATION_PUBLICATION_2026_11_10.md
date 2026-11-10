# I144 — External Java 25 FFM provider implementation, publication and local validation

**Date:** 2026-11-10  
**Formal implementation owner:** [I144 #1050](https://github.com/guillermomolina/protos/issues/1050)  
**Approved design:** [D238 #1047](https://github.com/guillermomolina/protos/issues/1047), [D238-B1-SQL-R1 ratification](../../decisions/language/D238_PROTOS_FIRST_NATIVE_FFI_B1_SQL_R1_RATIFIED.md)  
**PROTOS_REVISION:** `a7d9666aeb29a18905232907b83450fc82d034ec`  
**Published product:** https://github.com/guillermomolina/protos/commit/a7d9666aeb29a18905232907b83450fc82d034ec  
**Implementation version at that revision:** `0.3.460-SNAPSHOT`  
**Documentation follow-up:** [DOC022 #1112](https://github.com/guillermomolina/protos/issues/1112) (open, ready, independent)  

## Verified published product scope

GitHub confirms that commit `a7d9666aeb29a18905232907b83450fc82d034ec` has the subject
`I144: implement external Java FFM provider and pre-entry execution admission`
and changes **34 files** (+4,244/-14). This is a source/implementation publication,
**not** a claim that a new public release has been distributed.

- `providers/ffm/` implements an optional, independently packaged Java 25 FFM
  I085 provider with a trusted-in-process `ffi:sqlite3` target, logical-library
  host authorization, generic ABI descriptors and downcalls, symbol binding,
  per-Actor sessions, resource custody, bounded borrowed-memory copying, and
  explicit release/close. It contains no Java adapter dedicated to a particular
  SQLite C function.
- `dist/build_ffm_provider.py` builds the provider JAR against the portable JVM
  distribution's public SPI. `dist/test_external_foreign_providers.py` extends
  the external-provider acceptance harness with the SQLite provider and
  authorization/denial paths.
- `ProtosForeignValueOperations.acceptsExecution` supplies backward-compatible
  whole-signature pre-entry admission. The projected `call` and
  `std:interop.invoke` routes preserve ordinary pre-entry `Error` versus
  entered-provider `ForeignError`; a C status code remains ordinary data.
- C/Protos/JUnit fixture coverage includes integer/float/UTF-8 ABI,
  `out` parameters, SQLite statements, resource custody on C errors,
  adoption of all owned results before fallible conversion, irreversible
  destructor retirement, stale-reference rejection, bounded borrowed copying
  and epoch invalidation, and provider/session authorization.
- `docs/guide/interop/ffm-provider.md` and `providers/ffm/README.md` document
  the implemented low-level provider and its non-sandbox security boundary.
- The same commit includes a **separate incidental fix**
  in `ProtosStandaloneHostedExecution.java`: a RootActor service bootstrap
  outcome that already reached READY is no longer rejected merely because the
  RootActor terminates before the host rechecks mutable lifecycle state.
  This is not FFM functionality.

At the exact published product revision,
`protos/tests/foreign-provider/ffm-sqlite.protos` supplies
`sqlite3_prepare_v2`'s fifth `pzTail` argument as an explicit `null`
using a private nullable opaque descriptor. A sentence in the initially
published FFM guide still suggests the tests receive it as an opaque output;
[DOC022 #1112](https://github.com/guillermomolina/protos/issues/1112)
owns correcting the teaching prose to match executable source. That
documentation mismatch does not authorize a new ABI or public `std:ffi`.

## Validation and provenance

| Gate | Evidence |
| --- | --- |
| Product commit and changed paths | **VERIFIED** from GitHub commit `a7d9666aeb29a18905232907b83450fc82d034ec` |
| Product Maven version and Java release | **VERIFIED** from the published `pom.xml`: `0.3.460-SNAPSHOT`, Java release 25 |
| `git diff --check` | **CLEAN**, maintainer reported |
| I144 focal Java/SPI/FFM tests | **PASS**, maintainer reported |
| External-provider/CLI harness | **PASS**, maintainer reported; an earlier report noted 18/18 |
| `make check` | **PASS**, maintainer reported |
| `make test` | **PASS after the separate READY race fix**, maintainer reported |
| Earlier parallel `make test` run | **FAILED** one RootActor CLI service READY/termination test (5,204 tests, 1 failure, 6 skipped reported); not silently represented as green |
| Exact final per-suite JUnit counts, skip status and CI run | **NOT INDEPENDENTLY VERIFIED** in this record |
| Native Image or non-Linux FFM integration | **NOT CLAIMED** |
| Docs-repository `git diff --check` | **NOT RUN** through this GitHub connector publication |

The final green local gates are owner/maintainer reports; this record does
not claim that an agent or CI independently reran them. No fixture output,
test counts or platform probe not supplied by the owner has been invented.

## Boundaries and follow-up

I144 is limited to the optional, external low-level FFM provider for the
**Linux x86-64 portable JVM**. `TRUSTED_IN_PROCESS` is not a native sandbox;
ABI declarations are trusted and incorrect C signatures can crash or
corrupt the process. Native Image dynamic external-provider support, native
callbacks, generic native pointer arithmetic and arbitrary-library guest
authority remain excluded.

The public Protos-first `std:ffi` declaration layer belongs to
[LIB028 #1051](https://github.com/guillermomolina/protos/issues/1051);
the idiomatic SQLite connections/statements/transactions library belongs to
[LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052).
The independent guide, progressive tutorials, real examples and executable
learning-material harness belong to [DOC022 #1112](https://github.com/guillermomolina/protos/issues/1112).
None of these future deliverables is falsely claimed as implemented by I144.

```text
FORMAL_IDENTIFIER=I144
PROTOS_REVISION=a7d9666aeb29a18905232907b83450fc82d034ec
IMPLEMENTATION_VERSION=0.3.460-SNAPSHOT
PRODUCT_CHANGED_PATHS=34
PRODUCT_PUBLICATION=PASS_GITHUB_VERIFIED
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
FOCAL_TESTS=PASS_OWNER_REPORTED
EXTERNAL_PROVIDER_HARNESS=PASS_OWNER_REPORTED
MAKE_TEST=PASS_OWNER_REPORTED_AFTER_READY_FIX
MAKE_CHECK=PASS_OWNER_REPORTED
FINAL_EXACT_SKIP_COUNTS=NOT_VERIFIED
NATIVE_IMAGE_FFM=OUT_OF_SCOPE
DOC_FOLLOWUP=DOC022
```
