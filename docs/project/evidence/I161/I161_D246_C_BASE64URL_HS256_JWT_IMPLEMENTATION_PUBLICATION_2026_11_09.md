# I161 — D246-C strict Base64url and HS256 JWT implementation evidence

**Date:** 2026-11-09  
**Work item:** [I161 #1094](https://github.com/guillermomolina/protos/issues/1094), implementation child of [D246 #1072](https://github.com/guillermomolina/protos/issues/1072).  
**Design authority:** [D246-C ratified decision](../../decisions/language/D246_STRICT_HS256_JWT_C_RATIFIED_2026_11_07.md).  
**PROTOS_REVISION:** `a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b`  
**Implementation version:** `0.3.457-SNAPSHOT`; **normative specification revision:** `0.1.500`.  
**Published commit:** https://github.com/guillermomolina/protos/commit/a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b

This is revision-bound non-normative implementation and human-reported validation evidence. It is **not** a new approval, cryptographic certification, or proof of installed Native Image parity. Normative owners are [`spec/auth/JWT.md`](https://github.com/guillermomolina/protos/blob/a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b/spec/auth/JWT.md) and [`spec/encoding/BASE64URL.md`](https://github.com/guillermomolina/protos/blob/a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b/spec/encoding/BASE64URL.md).

## Published scope

The product commit contains **36 changed files** (+2,634/-61), including:

- [`std:text/base64url`](https://github.com/guillermomolina/protos/blob/a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b/protos/lib/text/base64url/module.protos): reusable strict RFC 4648 canonical unpadded `encode(Bytes)` and `decode(String)`, with rejection of padding, invalid alphabets and noncanonical unused bits.
- [`std:auth/jwt`](https://github.com/guillermomolina/protos/blob/a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b/protos/lib/auth/jwt/module.protos): synchronous, Actor-local `issue`, `verify` and `JwtError`, composing private [Compact](https://github.com/guillermomolina/protos/blob/a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b/protos/lib/auth/jwt/internal/Compact.protos), [Policy](https://github.com/guillermomolina/protos/blob/a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b/protos/lib/auth/jwt/internal/Policy.protos) and [Claims](https://github.com/guillermomolina/protos/blob/a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b/protos/lib/auth/jwt/internal/Claims.protos) sources.
- HS256 only; explicit caller-supplied Bytes secret (32–4096), policy and Instant. The signing input uses the original ASCII encoded segments; HMAC verification precedes payload parsing or release of authenticated claims.
- Strict compact alphabet and separator count, token/header/payload size budgets (8192/1024/4096 bytes), JSON depth budget 16, 32-byte MAC, restricted JOSE header and exact NumericDate registered-claim policies.
- No JWT-owned HTTP policy, key discovery, ambient clock, entropy, network, filesystem, Java JWT implementation or Core selector. The preexisting [I159 HMAC](../I159/I159_D244_C_HMAC_SHA256_IMPLEMENTATION_PUBLICATION_2026_11_08.md) is the cryptographic primitive.
- Six suite-native corpora, registry/completeness/SuiteGraph/progress integration, extractor-facing symbol documentation, user guide, tutorial, example, and normative specification links.

## Human-executed validation, before the metadata-only version bump

| Gate | Reported outcome |
| --- | --- |
| Base64url — vectors/canonicality | 5/5 PASS |
| Base64url — domain/roundtrip | 6/6 PASS |
| JWT — HS256 issuance and independent fixture | 4/4 PASS |
| JWT — negative security | 10/10 PASS |
| JWT — claims, NumericDate, limits | 24/24 PASS |
| JWT — identity and Actor boundaries | 3/3 PASS |
| **Total I161 guest cases** | **52/52 PASS** |
| Java integration focal classes (documentation, TEST010, TOOL005, TOOL011) | 52/52 PASS, 0 failed, BUILD SUCCESS |
| Final `make test` | PASS, maintainer reported exit 0 |
| Final `make check` | PASS, maintainer reported exit 0 |
| `git diff --check` and staged diff check | PASS |
| Version bump | `pom.xml` and `CHANGELOG.md` together to `0.3.457-SNAPSHOT`, **after** all tests |
| Post-bump tests | NOT RUN, per project workflow |
| Product commit/push to main | PASS; exact revision verified in GitHub |

An earlier full `make test` run failed with nine integration expectations (extractor symbols, missing TEST010 corpus bindings, TOOL005 29-vs-31 leaves, TOOL011 progress groups). A single integration batch updated the existing regression oracles and added documented public codec slots; **all 52 relevant focal Java tests passed**, followed by the final successful full `make test` and `make check`.

## Implementation debugging and scope limits

- `Compact.split` initially failed on valid three-segment tokens. Replacing a mutable `periods` array with two integer separator indices fixed the issue; the compact smoke test and independent HS256 fixture then passed. The deeper runtime behavior behind the earlier array path was **not** established as a confirmed runtime defect.
- A few negative-test cases returned `INFRA` when combining contextual `for` with `Assertions.signals`. Replacing those loops with explicit same-input assertions removed infrastructure aborts, without relaxing the token checks. No independent underlying engine-cause conclusion was proven.
- Two policy tests failed because their fixture assigned `clockSkewSeconds` onto an object lacking that local slot; adding the slot to the test fixture restored all 24 claim/time cases.
- The independent `clockSkewSeconds`-omitted guest smoke was proposed but no result was supplied; optional default is covered by implementation inspection, **not** claimed as separately executed smoke evidence here.
- **Installed Native Image JWT/Base64url parity and actual rebuilt Native CLI acceptance remain NOT REPORTED / NOT VERIFIED**. I161's original acceptance includes that gate, so publication of this record **does not close I161's full acceptance**.
- No RealWorld backend authentication deployment, application trusted-clock source, side-channel timing measurement, key erasure or crypto certification is claimed. Those are distinct application/host acceptance boundaries.

```text
FORMAL_IDENTIFIER=I161
DESIGN=D246-C
PROTOS_REVISION=a5d8c7a0b8f8dcd3dfc96c9c055cc8b36b16d49b
IMPLEMENTATION_VERSION=0.3.457-SNAPSHOT
SPEC_REVISION=0.1.500
FILES_CHANGED=36
GUEST_CASES=52/52_PASS
JAVA_FOCAL=52/52_PASS
MAKE_TEST=PASS_OWNER_REPORTED
MAKE_CHECK=PASS_OWNER_REPORTED
DIFF_CHECK=PASS_OWNER_REPORTED
PRODUCT_PUSH=PASS_VERIFIED
POST_BUMP_TESTS=NOT_RUN
INSTALLED_NATIVE_PARITY=NOT_VERIFIED
I161_FULL_ACCEPTANCE=PENDING_NATIVE
```
