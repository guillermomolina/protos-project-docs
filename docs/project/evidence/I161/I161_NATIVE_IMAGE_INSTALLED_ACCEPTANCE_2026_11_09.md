# I161 — Rebuilt and installed Native Image acceptance closure

**Date:** 2026-11-09  
**Work item:** [I161 #1094](https://github.com/guillermomolina/protos/issues/1094).  
**Ratified design:** [D246-C](../../decisions/language/D246_STRICT_HS256_JWT_C_RATIFIED_2026_11_07.md).  
**PROTOS_REVISION:** `a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b`  
**Implementation version:** `0.3.457-SNAPSHOT`  
**Published product commit:** https://github.com/guillermomolina/protos/commit/a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b  
**Earlier implementation evidence:** [I161 implementation publication](I161_D246_C_BASE64URL_HS256_JWT_IMPLEMENTATION_PUBLICATION_2026_11_09.md).

This is an **additive later verification record**. The earlier evidence truthfully stated that installed Native acceptance was *not yet verified at that checkpoint*; this record supersedes that limitation without rewriting the historical observation. All executions below were performed by the maintainer in the Protos checkout and their results reported in the active conversation. No new Protos product commit was made for this verification.

## Native executable validation

- Rebuilt Native executable: `target/native/protos`, Linux x86-64 ELF.
- Reported SHA-256 of the rebuild: `60d1002a66e66cc8e59ff678ad2a81e27499bfbbc35231bff5d2b0f18ef9485d`. This is a *binary checksum*, not the Git commit SHA or a separately authenticated archive checksum.
- Executed all six I161 corpus files against that Native executable, using `test --file`; **52/52 cases PASS**, zero failing corpus files.
- `make test-native` completed with **exit 0**, including reported `NATIVE_TLS_PEER_INTEGRATION=PASS` and `NATIVE_TLS_TRUST_PROVENANCE=PASS`. The underlying target rebuilds Native and runs the general Native regression script and Native TLS JUnit integration.
- The existing JVM `make test`, `make check`, 52/52 guest cases and 52/52 focal Java tests were already reported PASS before the `0.3.457-SNAPSHOT` metadata-only version bump, as captured in the earlier record.

## Real installed distribution validation

The maintainer used the standard offline Native distribution packaging and installer:

1. Created the `protos-0.3.457-SNAPSHOT-native-linux-x86_64.zip` development distribution with `dist/build_native.py`.
2. Installed it using `sh dist/install.sh --prefix <isolated temporary prefix> --local-native <archive>`. **INSTALL_EXIT=0**.
3. Ran the installed `bin/protos` with the **six corpus files from the installed distribution tree** under `share/protos/versions/v0.3.457-SNAPSHOT/protos/tests/`. This verifies the deployed library, Test Tool, native executable, and packaged test selection together, without relying on the checkout paths.
4. Observed all six suites PASS, **INSTALLED_NATIVE_CASES=52/52** and **I161_INSTALLED_NATIVE=PASS**.

| Installed corpus | Reported cases |
| --- | ---: |
| Base64url RFC 4648 vectors and canonicality | 5/5 PASS |
| Base64url domain and roundtrip | 6/6 PASS |
| JWT issuance and interoperability | 4/4 PASS |
| JWT negative security | 10/10 PASS |
| JWT claims/time/limits | 24/24 PASS |
| JWT identity and Actor boundaries | 3/3 PASS |
| **Total installed Native** | **52/52 PASS** |

### Diagnosed test-harness mistake, not a JWT regression

An earlier temporary installation executed the same installed `bin/protos` with **absolute checkout test-file paths**, not the installed corpus paths. The Test Tool rejected each selected file with `Error().signal()` at `protos/tools/test/Main.protos:234`, because it could not match the selected path to its installed repository plans. Those results were **0/52** and were not cryptographic assertion failures. The maintainer reran the exact six suites against the installed files, yielding **52/52 PASS**. The transient installed prefix was removed after the test run.

## Scope and acceptance

- **I161 product/JVM/Native executable/installed-development-Native acceptance: PASS**, using the reported local validation results.
- **D246-C design approval remains unchanged**; this record is implementation verification, not a new decision.
- No RealWorld HTTP authentication deployment, production clock provision, published public release, separate cryptographic certification, side-channel timing result, or key zeroization is claimed.
- No post-metadata-bump **JVM** suite was rerun; Native tests were executed retrospectively on the published exact product revision.

```text
FORMAL_IDENTIFIER=I161
PROTOS_REVISION=a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b
VERSION=0.3.457-SNAPSHOT
REBUILT_NATIVE_BINARY_SHA256=60d1002a66e66cc8e59ff678ad2a81e27499bfbbc35231bff5d2b0f18ef9485d
REBUILT_NATIVE_GUEST_CASES=52/52_PASS_OWNER_REPORTED
MAKE_TEST_NATIVE=PASS_EXIT_0_OWNER_REPORTED
INSTALLED_NATIVE=PASS_OWNER_REPORTED
INSTALLED_NATIVE_GUEST_CASES=52/52_PASS_OWNER_REPORTED
ORIGINAL_INSTALLED_NATIVE_ACCEPTANCE=SATISFIED
I161_IMPLEMENTATION_ACCEPTANCE=PASS
```
