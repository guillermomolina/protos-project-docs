# I159 — D244-C HMAC-SHA256 implementation publication evidence

**Date:** 2026-11-08
**Work item:** [I159 #1092](https://github.com/guillermomolina/protos/issues/1092), native child of [D244 #1070](https://github.com/guillermomolina/protos/issues/1070).
**Ratified authority:** [D244-C](../../decisions/language/D244_HMAC_SHA256_PRIVATE_BACKEND_C_RATIFIED_2026_11_07.md).
**PROTOS_REVISION:** `52b6da5da98e7eedb525ab4eb5111258c136193e`
**Version:** `0.3.456-SNAPSHOT`; **normative spec revision:** `0.1.499`.
**Published commit:** https://github.com/guillermomolina/protos/commit/52b6da5da98e7eedb525ab4eb5111258c136193e

This record is non-normative, revision-bound implementation and verification evidence, not a new design approval. The normative owner is [`spec/semantics/CRYPTO.md`](https://github.com/guillermomolina/protos/blob/52b6da5da98e7eedb525ab4eb5111258c136193e/spec/semantics/CRYPTO.md).

## Published functionality

- [`protos/lib/crypto/hmac_sha256/module.protos`](https://github.com/guillermomolina/protos/blob/52b6da5da98e7eedb525ab4eb5111258c136193e/protos/lib/crypto/hmac_sha256/module.protos) exports ordinary Actor-local `digest(key, message)` and `verify(key, message, tag)` for genuine Protos `Bytes`.
- A private, deferred [JCA HMAC-SHA256 facility](https://github.com/guillermomolina/protos/blob/52b6da5da98e7eedb525ab4eb5111258c136193e/src/main/java/com/guillermomolina/protos/execution/facilities/ProtosHmacSha256Facility.java) is installed only for the canonical module. It uses independent MAC invocations, exact octet snapshots, and no ambient authority or public Core native selector.
- Digests are fresh open mutable 32-octet `Bytes`. Verification accepts only complete 32-octet tags, uses a comparison without content-dependent early exit, and returns `false` for a wrong well-typed tag.
- Approved bounds: keys 0–4096 bytes, messages 0–16,777,216 bytes. Invalid inputs and provider faults signal ordinary Protos errors. There is no implicit entropy, secret erasure promise, JWT, password hashing or generic crypto registry.
- The same product commit includes RFC 4231 vectors, ownership/boundary/privacy regressions, Test Tool corpus registration, Native Image regression integration, Java architecture inventory, programmer guide, executable tutorial and example, and the normative changelog.

## Human-executed validation

The maintainer reported these validations on the implementation checkout **before** the final metadata-only version bump:

| Validation | Reported outcome |
| --- | --- |
| Focused Java facility, integration and Test Tool tests | 48 PASS |
| HMAC Protos suites | 24/24 PASS (10 vectors and boundaries, 8 verification/ownership, 6 invalid/private boundary) |
| Tutorial and example | PASS; tutorial prints `32` and three `true` values; example prints four `true` values |
| `make test-native` | PASS; Native Image build and 38/38 Native TLS JUnit tests successful; HMAC suites registered in the Native test driver |
| `make check` | PASS |
| Final `make test` | PASS; Protos test runner reports 3,708 passed, 0 failed |
| `git diff --check` | PASS; push of `52b6da5da98e7eedb525ab4eb5111258c136193e` independently verified |
| Tests after version bump | NOT RUN, per owner workflow |

The product commit advances `pom.xml` and `CHANGELOG.md` together from `0.3.455-SNAPSHOT` to `0.3.456-SNAPSHOT` and specifies `0.1.499` in the normative specification changelog.

## Validation findings and limits

- An initial nested-iteration guest case for invalid tag lengths caused an infrastructure abort. Replacing it with five explicit checks preserved all original lengths (0, 1, 16, 31, 33), and the full verification suite passed 8/8. The precise cause of the original abort remains unknown.
- Initial educational sources accidentally inherited unrelated bitwise-example code from license-header extraction; removing that prefix restored successful tutorial/example execution.
- The audited non-Core native-Closure inventory was updated to include the one private HMAC provider.
- Two different Workspace CLI service tests failed intermittently under cross-class parallel execution; the complete class passed 13/13 in isolation. Moving the class to the existing serial Java lane allowed `make test` to pass without deleting tests or weakening assertions. This is a scheduling mitigation, **not** an identified underlying concurrency root cause.
- The provided Native output proves the integrated Native target succeeded but does not separately enumerate per-case HMAC results in the excerpt. Do not claim independent HMAC timing-side-channel measurement, memory zeroization or cryptographic certification.

The final source commit was pushed by the maintainer; there were no post-version-bump tests.

```text
FORMAL_IDENTIFIER=I159
DURABLE_RECORD_DECISION=REQUIRED
PROTOS_REVISION=52b6da5da98e7eedb525ab4eb5111258c136193e
IMPLEMENTATION_VERSION=0.3.456-SNAPSHOT
SPEC_REVISION=0.1.499
HMAC_GUEST_CASES=24/24_PASS
PROTOS_SUITE=3708_PASS_0_FAILED
MAKE_TEST=PASS_OWNER_REPORTED
MAKE_CHECK=PASS_OWNER_REPORTED
MAKE_TEST_NATIVE=PASS_OWNER_REPORTED
NO_TESTS_AFTER_BUMP=PASS
```
