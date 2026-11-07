# D246-C — JWT comparative security, falsification and owner-approval evidence

**Evidence captured 2026-11-07.** The owner approved **D246-C** in the active conversation by saying **"apruebo D246-C"** after receiving and reviewing the complete C proposal and its exact strict security profile. This record is a retrospective research/approval snapshot, **not a substitute for the normative Protos specification, implementation, executed tests or a recorded native parent relationship**.

- Design [D246 #1072](https://github.com/guillermomolina/protos/issues/1072).
- Ratified decision [D246-C](../../decisions/language/D246_STRICT_HS256_JWT_C_RATIFIED_2026_11_07.md).
- Implementation [I161 #1094](https://github.com/guillermomolina/protos/issues/1094). The project allocated [I160 #1093](https://github.com/guillermomolina/protos/issues/1093) concurrently for D245-C; I161 is the collision-free JWT identifier after post-creation reconciliation.
- Exact inspected product HEAD: [`08c27760eb3f85105277e5b1b1b159abbca67ebe`](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe), Java 25 / 0.3.453-SNAPSHOT; no JWT implementation present at audit.
- External RealWorld reference: [`realworld-apps/realworld@25588846c314b15a5e427ffe816e6b4af7be68c7`](https://github.com/realworld-apps/realworld/commit/25588846c314b15a5e427ffe816e6b4af7be68c7); `specs/api/openapi.yml`, `specs/api/hurl/auth.hurl`.
- Relevant already approved choices: D243-C explicit entropy ([design record](../../decisions/language/D243_SECURE_ENTROPY_EXPLICIT_CAPABILITY_C_RATIFIED.md)); D244-C fixed-key/message-bounded synchronous HMAC with private JCA backend ([record](../../decisions/language/D244_HMAC_SHA256_PRIVATE_BACKEND_C_RATIFIED_2026_11_07.md)); D219 private-facility minimality; D220 explicit trusted-security boundaries.
- User approval does not approve new entropy/clock authority, password hashing/cryptographic provider architecture, OAuth, a general JWS API, or any change to already established designs.

## 1. Normative and repository evidence

Instruction authority inspected: `AGENTS.md`, `AGENTS.work/DESIGN.md` (GITHUB010/GITHUB021), `AGENTS.work/COORDINATION.md` (formal Issue publication, stable identifiers and closure evidence), `AGENTS.work/REFERENCE.md` (design philosophy and role-first durable records), `spec/AGENTS.md`, `protos/AGENTS.md`; implementation follow-up also uses `AGENTS.work/IMPLEMENTATION.md` and `src/AGENTS.md`.

Applicable normative owners/sections inspected: `spec/PROTOS_LANGUAGE_SPEC.md`; `spec/semantics/MODULES.md` (Actor-local MODULE/SOURCE and private init); `spec/semantics/VALUES_AND_COLLECTIONS.md` (Bytes identities, mutation, String/Bytes separation, foreign values); `spec/semantics/ERRORS.md` (fresh standard errors and ordinary library error prototypes); `spec/io/HTTP.md` (§5 byte-valued headers preserving duplicates; §10 per-request limits; §11 HTTP exposure limitation; §17 Router purity); `spec/io/PROCESS_IO.md` (bootstrap-local authority and no implicit grants).

Implementation/guide files inspected: `protos/lib/json/json/module.protos` (JSON node constructors, `parse`, `encode`, duplicate-member rejection, exact-decimal number representation); `protos/lib/http/{router,server,host}/module.protos`; `protos/lib/text/utf8/module.protos`; `protos/lib/crypto/sha256/module.protos`; `protos/lib/datetime/{instant,time}/module.protos`; `docs/guide/library/{README,text,datetime,logging}.md`; `docs/design/STANDARD_LIBRARY_IDEAS.md`; `protos/tools/test/{RepositoryCorpusPlans,RepositorySuite}.protos`; `protos/tests/library/crypto/sha256/{vectors-and-boundaries,ownership-and-domain}.protos`; `src/AGENTS.md`; `spec/PROTOS_SPEC_CHANGELOG.md`; application `README.md`, `TODO.md`.

Observed gaps at audited revision:
- Existing SHA-256 is a digest, not a signing/verification service; its earlier LIB006 scope explicitly excludes HMAC, JWT, entropy and secret-handling guarantees.
- `std:json/json` rejects duplicate members, has a separate JSON node model and supports exact decimal numbers. JWT must **use** rather than replace its parser/encoder.
- No current public ordinary Protos Base64url module was found. Internal Java use of `java.util.Base64` for module keys is not a Standard Library codec.
- `std:datetime/instant` provides exact temporal values but no wall-clock acquisition; `std:logging` explicitly requires a caller-supplied time source. No JWT library should introduce an implicit clock.
- HTTP preserves repeated Authorization headers as separate pairs and raw field values as Bytes. A generic JWT library must not silently impose RealWorld's particular `Authorization: Token ...` scheme.
- I159 HMAC is approved **but not published** at the product revision inspected; independently approved D243/I157 entropy is for key acquisition and must not be needed during ordinary deterministic JWT computation.

## 2. Primary standards and security sources

- [RFC 7515 — JSON Web Signature (JWS)](https://www.rfc-editor.org/rfc/rfc7515): original header/payload compact octets are the MAC input; protected headers; compact vs JSON serializations; HS256 example in Appendix A.1.
- [RFC 7519 — JSON Web Token (JWT)](https://www.rfc-editor.org/rfc/rfc7519): registered claims; NumericDate; issuer/audience semantics; non-unique JSON member security.
- [RFC 7518 — JSON Web Algorithms (JWA)](https://www.rfc-editor.org/rfc/rfc7518): HS256 MAC and secret-key strength; algorithm-specific requirements.
- [RFC 8725 — JSON Web Token Best Current Practices](https://www.rfc-editor.org/rfc/rfc8725): algorithm confusion, key provenance, mutually exclusive validation rules, `typ` and cross-JWT confusion, untrusted `kid`/remote-key concerns.
- [RFC 4648 — Base-N Encodings](https://www.rfc-editor.org/rfc/rfc4648): URL-safe alphabet, padding and canonical final pad bits.
- [PyJWT usage](https://pyjwt.readthedocs.io/en/stable/usage.html) and [API warnings](https://pyjwt.readthedocs.io/en/stable/api.html): explicitly trusted algorithm list; claims verification settings; never derive verifier algorithms from token headers.
- [jose (JavaScript) documentation](https://github.com/panva/jose): separate `SignJWT`/`jwtVerify` from generic JWS.
- [Auth0 java-jwt](https://github.com/auth0/java-jwt): configured algorithm and claim requirements.
- [JJWT](https://github.com/jwtk/jjwt): signed-claims parsing with explicit verification keys.
- [golang-jwt/jwt](https://github.com/golang-jwt/jwt): explicit validation configuration; note that requiring expiration is a policy, not guaranteed by a generic JWT parser.
- [Nimbus JOSE+JWT](https://connect2id.com/products/nimbus-jose-jwt): extensible JOSE processing and key selectors.
- [ruby-jwt](https://github.com/jwt/ruby-jwt): independent Ruby verifier and algorithm-selection precedent.

GraalVM/Truffle relevance: GraalPy can host Python packages such as PyJWT, GraalJS exposes a JavaScript ecosystem (though a Node.js package is not a portable embedded guest primitive), and TruffleRuby exposes the Ruby JWT idiom. Their **guest-library APIs** are useful prior art; they cannot justify bypassing Protos Module/Actor isolation, authority provisioning, Native Image constraints, or dependency independence. This Dxxx does **not** pick a Truffle host-implementation architecture; D244-C separately approves the narrow JCA crypto boundary.

## 3. Meaningful candidate set

| Candidate | Model | Decision |
| --- | --- | --- |
| A | Defer JWT entirely | Rejected: leaves RealWorld without an authentic reusable solution; encourages unsafe private workarounds |
| B | Publish HMAC/Base64url only; application writes JWT/claims policy | Rejected as primary design: policy and algorithm confusion are repeated in every consumer |
| **C** | **Ordinary strict JWT issuance/complete verification, using HMAC and a reusable codec; private JWS composition** | **OWNER APPROVED**; smallest safe current reusable boundary |
| D | Two independent public JWS and JWT layers from the start | Deferred: clean orthogonality, but imposes broad signed-message semantics without current independent need |
| E | Full JOSE/JWK/JWKS/JWE/OAuth/provider abstraction | Rejected as present design: unjustified public complexity, trust acquisition, remote operations and cryptographic attack surface |

The comparison explicitly separates `future-compatible` from `future-preimplemented`: C preserves the cryptographic signing-input boundary that future JWS can expose without paying for it now.

## 4. GITHUB010 twelve-dimension scoring

Scale 1..5 (higher is better); confidence H/M/L. Each cell provides the reason; totals do not override security/foundational constraints.

| Dimension | B — primitives only | **C — JWT** | D — public JWS + JWT |
| --- | --- | --- | --- |
| Correctness/invariants | 2/M — every app repeats critical verification policy | **5/H — signature+claims are one fail-closed operation** | 5/H — verifier composition possible |
| Protos alignment | 4/M — small primitive universe | **5/H — ordinary import and no privilege** | 5/H — orthogonal layers |
| Present proportionality | 3/H — shifts necessary JWT burden to app | **5/H — directly solves current authentic tokens** | 2/H — unused JWS public surface |
| Incremental growth | 3/M — later addition of JWT needed | **4/H — JWS may be added independently later** | 5/M — public general substrate |
| Future-option resilience | 3/M — applications diverge | **4/M — keep original signing-input boundary** | 5/M — broad extension |
| Scalability | 3/M — independently maintained per app | **4/M — Actor-local stateless verifier** | 4/M — likewise Actor-local |
| Conceptual simplicity | 2/H — app duplicates security responsibilities | **5/H — narrow complete contract** | 3/M — two public abstractions |
| Portability | 4/M — byte primitives portable | **5/M — Protos source over D244** | 5/M — same primitives |
| Runtime/resource cost | 3/L — app-specific overhead unknown | **4/M — bounded byte/JSON/HMAC work** | 4/L — similar operational cost |
| Failure/operability | 2/M — scattered policy, inconsistent errors | **5/M — single fail-closed boundary** | 4/M — more call paths |
| Deferral/migration cost | 2/M — app migrations likely | **4/H — generic JWS additive** | 5/M — built in immediately |
| Evidence/implementation risk | 4/H — known primitives | **5/H — many independent JWT systems** | 5/H — mature JOSE precedent |

**Non-compensating flags:** B is underengineered for authenticated-token verification: adding JWT later requires replacing application-private security checks. D and E overengineer the present project because their public surface, algorithm agility and configuration risk have no second proven use. C's strongest downside is absent public JWS/asymmetric support; its additive migration path is concrete.

## 5. Adversarial interoperability and conformance matrix

These are **expected tests to implement, not executed results**. Execute with independently generated fixture bytes/keys, not a mocked verifier.

| Case | Required outcome |
| --- | --- |
| In-profile HS256 JWT roundtrip, original bytes | Verify yields complete trusted JSON claims |
| External HS256 PyJWT, jose, Java and Go fixtures in profile | Verify without requiring same JSON field order/spacing |
| Protos-generated HS256 externally verified | External verifier accepts exact JWS signing input |
| Payload or protected-header byte changed after signing | Reject, no verified claims |
| Tag changed first/middle/last octet; tag truncated; wrong key | Reject, never early-exit comparison in the HMAC primitive |
| `alg:none`, `alg:RS256`, missing or malformed `alg` | Reject, never choose untrusted algorithm |
| Token-supplied `kid`, `jku`, `x5u`, `jwk`, `crit` | Reject; never resolve files, URLs, JWK or trusted keys from token |
| 0/1/2/more-than-2 separators; empty segment | Reject |
| Base64url `+`, `/`, `=`, whitespace, impossible length, nonzero unused bits | Reject noncanonical spelling |
| Duplicate JSON `alg`, `exp`, `iss` and escaped-equivalent keys | Reject |
| Malformed UTF-8, JSON non-object, nesting 17 | Reject |
| Total token 8192 / 8193; decoded header 1024/1025; claims 4096/4097 | Precise permitted/rejected bounds |
| Key 31/32/4096/4097 octets, nonstandard `Bytes` | Enforce JWT minimum and D244 maximum/domain |
| Correct `iss`/`aud`/`typ`, wrong each, `aud` Array with/without expected | Exact policy, no normalization |
| `sub` missing/empty/nonstring | Reject |
| `iat`/`exp` missing, string, float or out-of-order | Reject |
| `now == exp` without skew, `nbf` in future, `iat` in future | Reject at precise temporal boundaries |
| Configured skew 0/300/>300, maximal life exactly/exceeded | Enforce explicit upper bounds |
| `jti` absent/present, additional nonsecurity claims | No implicit RNG or extra privilege |
| Wrong native crypto provider, bad policy, wrong argument kind | Fail closed, no secret-bearing host diagnostics |
| Cross-Actor copies, independent Processes/Contexts and installed Native | No leaked authority/global key/other mutable state |
| Two Authorization headers, `Token` vs `Bearer` | Application HTTP layer decides; JWT never accepts a header or converts it to principal |
| Signed token with deleted account or unauthorized article edit | Cryptographic validity alone does not authorize; app denies |

Negative tests MUST assert absence of a trustworthy claims result, not only "an error was thrown".

## 6. Scalability, future regret and incremental analysis

**Smallest sufficient now:** one strict issuer+verifier, one reusable Base64url codec, existing Protos JSON/UTF-8/Instant, the approved HMAC primitive, tests and guides. Extra public JOSE methods or new host facilities have no present justification.

**Pay for what you need:** ordinary import builds no network, clock, RNG, provider registry, key pool or background tasks. Per-operation CPU and allocation scale with compact bounded token size; the backend cost is wholly delegated to D244's keyed MAC. Tokens larger than admitted bounds fail early.

**Grow as you need:** public generic JWS can be added without changing the signed compact token bytes; asymmetrical keys would be separate explicit trust profiles. Multi-node scaling may pass a small trusted verification configuration or dispatch verification to a dedicated Actor. No implicit key replication across Actors/Contexts.

**Exact cost of deferral:** future JWS requires a new generic API and tests (not breaking existing `Jwt.verify`); future `kid` requires a trusted, bounded in-process key selection contract; RS256/EdDSA requires new crypto primitives and a new key/algorithm security review; revocation requires application session state. These are real costs but not presently demanded.

**Future regret scenario:** third-party OIDC or distributed verification with asymmetric issuer keys could make strict single-HS256 insufficient. Escape: separate additive verifier/profile and public JWS module under a later D decision; old JWT calls and compact signed bytes remain stable. No automatic algorithm reinterpretation.

**Worst argument against C:** an independent public JWS module could be a better orthogonal foundation if many unrelated signed-document consumers materialize immediately. Current evidence identifies only JWT/RealWorld; public generality has not earned its extra concept and misuse surface.

## 7. GITHUB021 approved-invariant/delta audit

| Previously approved invariant | C treatment |
| --- | --- |
| D243-C: only explicit host-granted entropy to initial RootActor; no grant from import | PRESERVED — JWT does not generate randomness or transfer grants |
| D244-C: HMAC is synchronous, supplied Bytes, 0..4096 key and <=16 MiB message; no public key handle | PRESERVED — JWT narrows its key minimum to 32; never expands backend operations |
| D219: do not generalize private facilities speculatively | PRESERVED — no new host JWT facility |
| D220: explicit trust/authority separation | PRESERVED — keys and time supplied by caller |
| `MODULES.md`: per-Actor module instances, no global mutable state | PRESERVED — ordinary module, no key/provider singleton |
| `VALUES_AND_COLLECTIONS.md`: String != Bytes, standard Bytes semantics | PRESERVED — strict explicit UTF-8/Base64url |
| `ERRORS.md`: fresh Error identity and non-Core library prototype | PRESERVED |
| `HTTP.md` §17: Router has no credential authority | PRESERVED — application HTTP extraction outside library |
| Existing `std:datetime/instant` no clock source | PRESERVED — `now` explicit, trusted acquisition remains out of scope |

No conflicting already owner-approved D246 invariant was found in the active Issue (it was an investigation-only intake with no preapproved candidate). The new C contract is the **disclosed exact proposal** approved by the owner. **DECISION_APPROVAL_PROVENANCE=PASS; DECISION_INVARIANT_CONSISTENCY=PASS.**

## 8. Execution/publication honesty

This is a research and owner-approval record. **No tests, benchmarks, fuzzing, interop fixtures, builds, JCA provider checks, native-image runs, Protos spec updates, or actual RealWorld authentication are asserted to have passed by D246 ratification.**

The implementation issue may be allocated and researched after approval; it remains blocked from an integrated green JWT implementation until I159's real HMAC module is present. The trustworthy current-time source for RealWorld remains a separate concern. Publication of this record and an exact-D246 issue closure are independent of those future implementations.

**Historical note:** I161's GitHub native parent relationship is a live Issue coordination postcondition, not provided by a Markdown link or this file. Verify it separately under GITHUB015; do not describe it as established solely from this record.
