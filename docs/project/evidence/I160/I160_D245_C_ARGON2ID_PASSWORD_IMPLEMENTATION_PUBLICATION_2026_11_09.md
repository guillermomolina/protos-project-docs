# I160 — D245-C Protos-first Argon2id password implementation and publication

**Date:** 2026-11-09  
**Issue:** [I160 #1093](https://github.com/guillermomolina/protos/issues/1093), implementing owner-ratified [D245-C #1071](https://github.com/guillermomolina/protos/issues/1071).  
**PROTOS_REVISION:** `9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd`  
**Implementation version:** `0.3.458-SNAPSHOT`.  
**Published product commit:** https://github.com/guillermomolina/protos/commit/9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd

This is revision-bound, non-normative implementation and maintainer-reported test evidence. It is not an independent security certification. Observable Password semantics are governed by [`spec/semantics/PASSWORD.md`](https://github.com/guillermomolina/protos/blob/9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd/spec/semantics/PASSWORD.md), not this project record.

## Published implementation

The published product commit modifies **35 files** (+3,790/-67), including:

- Ordinary Protos module [`std:crypto/password`](https://github.com/guillermomolina/protos/blob/9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd/protos/lib/crypto/password/module.protos), private [record codec](https://github.com/guillermomolina/protos/blob/9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd/protos/lib/crypto/password/internal/Record.protos) and `PasswordError`. Public operations are `hash`, `hashWithSalt`, `verify` and `needsRehash`.
- Argon2id v=19 with the selected m=65536 KiB, t=3, p=4 profile, 16-byte salt and 32-byte derived tag. Password Strings are normalized to NFC before strict UTF-8 encoding. Canonical, strictly bounded records preserve normalization and profile identity.
- A private [Argon2 facility](https://github.com/guillermomolina/protos/blob/9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd/src/main/java/com/guillermomolina/protos/execution/facilities/ProtosPasswordArgon2Facility.java) backed by pinned Bouncy Castle `bcprov-jdk18on:1.86`; no public Java crypto or foreign-object surface.
- [Bounded asynchronous KDF executor](https://github.com/guillermomolina/protos/blob/9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd/src/main/java/com/guillermomolina/protos/execution/facilities/ProtosPasswordKdfExecutor.java) and [per-Process admission/custody](https://github.com/guillermomolina/protos/blob/9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd/src/main/java/com/guillermomolina/protos/execution/facilities/ProtosPasswordKdfCustody.java), with Actor-local Future completion and cancellation; expensive derivation does not execute within an Actor turn.
- Standalone Processes reuse a runtime-host-owned lazy executor; embedded Processes own lazy executors. Actor/Process/Context retirement cancels or suppresses outstanding tickets and late publication.
- Password creation using a generated salt requires an **authentic, independently granted** `Entropy` capability. Importing Password never creates or grants entropy. `hashWithSalt` accepts caller-owned actual Bytes with an exact 16-octet salt; it does not fabricate randomness.
- Two library corpus sources, five new Java integration test classes, repository Test Tool registration, SuiteGraph/progress/completeness and native-boundary test updates, a language-guide update and an [executable example](https://github.com/guillermomolina/protos/blob/9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd/protos/examples/crypto/password-argon2id.protos) explicitly requiring `--allow-entropy`.

## Maintainer-executed validation and debugging history

| Gate | Evidence |
| --- | --- |
| Password Argon2 vectors, NFC, input, record, Future, KDF executor and hosted integration | Focused Java integration suites PASS |
| Hosted `hashWithSalt`, explicit-Entropy record, import-without-grant, independent Context state | PASS; Context-close-with-pending-KDF regression included |
| Test Tool/CLI corpus, SuiteGraph, grouping, native boundary and completeness contracts | Seven integration test classes PASS after correcting stale corpus expectations |
| Final `make test` | **PASS, exit 0** (maintainer-reported) |
| Final `make check` | **PASS, exit 0** (maintainer-reported) |
| `git diff --cached --check` | **PASS**; exactly 35 staged paths before publication |
| Publication metadata | `pom.xml` and `CHANGELOG.md` updated once to `0.3.458-SNAPSHOT` **after** full test gates |
| Tests after version bump or product push | **NOT RUN**, as directed by owner |
| Product commit and push | **PASS**, exact SHA verified on `main` |
| Rebuilt/installed Native Image specifically exercising I160 Password/Argon2id | **NOT REPORTED / NOT VERIFIED** |

An initial full `make test` reported 12 Java failures: the Password example lacked the explicit CLI entropy grant in its learning-material harness; exact TEST010/TOOL005/TOOL011 corpus counts and membership had not yet been updated; the native-Closure inventory omitted the private Password facility; and repository completeness lacked the new corpus authority. The subsequent bounded corrections passed the seven affected Java integration classes, followed by final full `make test` and `make check` PASS. Temporary hosted-close diagnostics were removed. This records resolved intermediate failures rather than hiding them.

## Scope boundaries and outstanding acceptance

The implemented hash is an application-reusable Password primitive, not a user/account service, migration engine, JWT key manager or cryptographic certification. No plaintext credential migration, implicit entropy, ambient clock, provider registry, general secret-zeroization guarantee or RealWorld end-to-end authentication deployment is claimed.

**Important:** I160 #1093's original acceptance explicitly calls for rebuilt/installed Native Image verification of the password backend. That specific gate has **not** been evidenced in the supplied I160 validation and is not satisfied by the separately verified [I161 JWT/Base64url Native cases](../I161/I161_NATIVE_IMAGE_INSTALLED_ACCEPTANCE_2026_11_09.md). The product implementation and JVM/full-suite publication are verified, but mark full I160 acceptance **pending Native** unless the owner explicitly revises that acceptance gate. This record does not direct execution of any new tests.

```text
FORMAL_IDENTIFIER=I160
DESIGN=D245-C
PROTOS_REVISION=9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd
VERSION=0.3.458-SNAPSHOT
FILES_CHANGED=35
FOCUSED_INTEGRATION=PASS_OWNER_REPORTED
MAKE_TEST=PASS_EXIT_0_OWNER_REPORTED
MAKE_CHECK=PASS_EXIT_0_OWNER_REPORTED
STAGED_DIFF_CHECK=PASS_OWNER_REPORTED
POST_BUMP_TESTS=NOT_RUN
PRODUCT_PUSH=PASS_VERIFIED
INSTALLED_NATIVE_PASSWORD=NOT_VERIFIED
I160_IMPLEMENTATION_PUBLICATION=PASS
I160_FULL_ACCEPTANCE=PENDING_NATIVE
```
