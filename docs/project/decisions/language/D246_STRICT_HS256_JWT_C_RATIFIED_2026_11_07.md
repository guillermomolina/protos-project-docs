# D246 — Reusable strict HS256 JWT with explicit trust and time (Candidate C)

**Status: RATIFIED DESIGN — exact owner-approved D246-C on 2026-11-07.** This durable record is **non-normative**: it records the approved design, but does not claim that the corresponding Protos specification or library is implemented or that the native distribution has passed tests.

- Design issue: [D246 #1072](https://github.com/guillermomolina/protos/issues/1072).
- Implementation issue: [I161 #1094](https://github.com/guillermomolina/protos/issues/1094). This is the surviving identifier after a concurrent I160 allocation by D245-C [#1093](https://github.com/guillermomolina/protos/issues/1093).
- Motivating application: [I153 RealWorld #1068](https://github.com/guillermomolina/protos/issues/1068), but the JWT module is reusable and does not implement HTTP or user-account semantics.
- Inspected Protos source baseline: [`08c27760eb3f85105277e5b1b1b159abbca67ebe`](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe), Java 25 / 0.3.453-SNAPSHOT, before JWT/HMAC implementation.
- External RealWorld reference: [`realworld-apps/realworld@25588846c314b15a5e427ffe816e6b4af7be68c7`](https://github.com/realworld-apps/realworld/commit/25588846c314b15a5e427ffe816e6b4af7be68c7), particularly `specs/api/openapi.yml` and `specs/api/hurl/auth.hurl`.
- Detailed research, adversarial matrix, GITHUB010 comparative scoring, exact owner approval and GITHUB021 consistency review: [D246 evidence](../../evidence/D246/D246_C_COMPARATIVE_SECURITY_OWNER_APPROVAL_2026_11_07.md).
- Explicit approval provenance: in the active 2026-11-07 D246 conversation, following the complete alternatives A–E, the C proposal, the strict profile in sections 6–7 and the requested exact-choice approval, the project owner said **"apruebo D246-C"** and explicitly requested Issue updates, durable publication and a file-edit-only implementation prompt. The approval applies to C **as presented**, not to speculative JOSE features, a clock capability or independent crypto decisions.

## 1. Approved boundary

**C** is an opt-in, ordinary, Protos-first **JWT issuance and complete verification library** for a strictly bounded compact **HS256** profile. The library composes existing/reusable Protos `std:json/json`, UTF-8 and Bytes conversions, a reusable public Base64url codec and the independently ratified D244-C HMAC-SHA256 operation. It does **not** define a new Core family, special syntax, prelude binding, ambient security authority, HTTP authentication policy or general-purpose public JWS architecture.

The library may implement a private JWS-compact composition internally, but **JWS is not independently exported** in this initial scope. `std:auth/jwt` was discussed as a candidate Standard Library path; use it as the concrete provisional implementation path in I161 unless an existing canonical namespace rule requires an equivalent correction. Namespace placement is not license to enlarge the protocol.

The conceptual public operations are:

```text
Jwt.issue(claims, signingKey, policy, now) -> String
Jwt.verify(token, verificationKey, policy, now) -> verifiedClaims
```

`claims` is structured JSON data using the established `std:json/json` node model, not arbitrary attacker-provided objects converted by reflection. `verifiedClaims` is ordinary Protos JSON data made available **only after signature, format, type, claim and temporal-policy verification all succeed**. The library can expose an ordinary library `JwtError` prototype for invalid tokens; no `JwtError` is introduced into Core or the prelude. Exact method/slot names beyond this conceptual API belong to the spec/implementation reconciliation of I161, without creating new observable semantics.

## 2. Trust and cryptography

- The sole initially recognized JWS algorithm is **`HS256`**; verification never follows an untrusted `alg` to select a cryptographic algorithm, provider or key. `none`, RS/ES/EdDSA, token-supplied `jwk`, `jku`, `x5u`, `kid`, `crit`, `b64=false`, JWE and JWS JSON serialization are excluded/rejected by this profile.
- Only an explicitly supplied **genuine standard `Bytes`** secret of **32 through 4096 octets** may be used for JWT HS256 signing or verification. The 256-bit minimum is a JWT-specific security requirement; D244-C itself permits shorter keys for its general HMAC primitive. No String/password-to-key coercion.
- D244-C's already approved `std:crypto/hmac_sha256.digest(key, message) -> Bytes[32]` and `verify(key, message, tag) -> Boolean` are the crypto boundary. They are synchronous and operate over bytes. Reuse their full-length, no-early-exit comparison; do not build an independent secret-comparison loop in JWT.
- The signature covers **the exact ASCII octets of the two original compact segments**: `BASE64URL(protectedHeaderBytes) + "." + BASE64URL(payloadBytes)`. Verification must not re-serialize parsed JSON before MAC verification. Multiple equivalent JSON serializations must interoperate.
- The Base64url encoding uses the RFC 4648 URL-safe alphabet **without padding**; the strict decoder rejects mixed alphabets, whitespace, extra characters, padding, impossible lengths and non-canonical unused pad bits.
- Parse UTF-8 strictly, reject duplicate JSON names, and never admit a merely decoded or partially parsed token as authenticated.
- HMAC neither obtains entropy nor reads a clock. The JWT library also never mints keys, reads environment, filesystem, Network, Process, host Java security APIs or current time.

## 3. Registered claims and explicit policy

A trusted ordinary `policy` and explicit `now` are supplied to both operations. The issuer and verifier must validate the profile, rather than merely serialize/parse claims.

| Member / boundary | Approved profile |
| --- | --- |
| `iss` | Required String; exact, case-sensitive configured issuer match |
| `aud` | Required String or Array of Strings; must include the exactly configured audience |
| `sub` | Required nonempty String representing an application-selected stable identity |
| `exp` | Required exact Integer seconds (no Float rounding) |
| `iat` | Required exact Integer seconds |
| `nbf` | Optional; if present must be valid exact Integer seconds and enforced |
| `typ` | Protected-header type required and matched to explicitly configured type; `realworld-access+jwt` is an illustrative application value, not a library-global type |
| `jti` | Optional String/value as application policy permits; no implicit random identifier generation |
| Additional claims | Allowed only as structured JSON; do not bypass the required profile |

Time comparison uses a supplied `std:datetime/instant` and exact Integer/nanosecond arithmetic, with no implicit host clock and no Float. At zero skew, equality of `now` with `exp` is expired. Require `exp > iat`, enforce a trusted positive `maxLifetimeSeconds`, reject future `iat` beyond skew and future `nbf` beyond skew. Skew defaults to **0 seconds**; configured skew cannot exceed **300 seconds**. A malformed or missing claim cannot be changed into a default permissive value.

The issue and verifier do not invent a global clock, default issuer, audience, expiration or authorized user. Claim issuance is an explicit operation over supplied data, key and time; omission of required claims is an error rather than a silent guess. Subject-to-account resolution, current account state, password verification, revocation and permissions remain application logic. Tokens remain valid signatures even if a subject is no longer authorized; the application must reject that subject.

## 4. Budgets, failure and isolation

- Maximum complete compact JWT: **8192 ASCII octets**.
- Maximum decoded protected header: **1024 octets**.
- Maximum decoded claims JSON: **4096 octets**.
- Maximum supported JSON nesting: **16 levels**.
- HS256 signature: **exactly 32 octets**; no truncated tags.

Reject over-budget input before expensive parsing or allocation. The boundaries are independent of existing HTTP header and body budgets.

Malformed, unauthenticated, wrongly signed, wrong-type, wrong-claim, expired, future/not-yet-valid or unsupported-feature tokens fail as invalid tokens (a fresh ordinary library Error occurrence, proposed `JwtError`) with **no verified claims**. Configuration/type/provider failures fail closed; never silently downgrade crypto, expose secret-bearing provider exceptions, return `null` as successful verification or treat malformed tokens as anonymous-but-authenticated. HMAC provider failures must not be concealed as ordinary invalid signature unless independently specified.

Module instances and semantic mutable state remain Actor-local; Protos standard transfer/isolation rules apply. There is no global key registry, cached mutable `Mac`, implicit Actor inheritance, general capability delegation, task creation or new suspension point in the JWT module. JVM and installed Native Image must have the same externally visible contract, with any private HMAC provider implementation governed by D244-C/I159.

## 5. HTTP composition and RealWorld boundary

The RealWorld header convention is exactly `Authorization: Token <jwt>`, not JWT compact syntax. `std:http/router`, `std:http/server` and the JWT library remain orthogonal: HTTP transport returns ordered, duplicate-preserving headers with Byte-valued fields; the application rejects ambiguous/multiple Authorization values, extracts the scheme safely, invokes full JWT verification, resolves `sub` against its current stable identity store and applies permission rules. Errors become appropriate application HTTP responses (usually 401) **outside JWT**.

No JWT module operation accepts a request object, produces HTTP status codes, accesses Network/Filesystem or declares a user authenticated.

## 6. Explicitly excluded and escape path

Deferral is deliberate: a **public generic JWS** API; asymmetric RS/ES/EdDSA; encryption/JWE; JWS JSON formats; externally fetched JWKS; key-provider/HSM registry; automatic `kid` discovery; OAuth/OIDC; revocation/session storage; `AuthenticatedUser` Core category; portable wall-clock acquisition; HTTP middleware; app-specific account records.

A later independently required generic signed-message consumer may justify exposing JWS while preserving the existing compact signed-byte format and strict JWT operation. A later multi-node/asymmetric requirement may justify new explicit keys and issuer verification profiles, but does not automatically grant authority or weaken C's HS256-safe defaults. Explicit key rotation can initially be performed using trusted application-owned verification configurations and error handling; it does not require accepting `kid` from the token.

## 7. Invariant check and publication boundary

The proposal preserves all already approved relevant constraints: D243-C explicit entropy (no auto key generation); D244-C ordinary synchronous HMAC over Bytes, key/message admission and no global handles; D219 no unjustified generic private runtime facility; D220 explicit security/trust composition; module Actor locality; strict String/Bytes separation; fresh ordinary errors; and D222 HTTP Router purity. No existing owner-approved D246 invariant is known to be contradicted. **GITHUB021 invariant/delta consistency: PASS for the exact approved candidate.**

D246-C ratification does **not** implement it. I161 owns the normative `spec/` owner, pure Protos public modules/codec, tests, documentation and JVM/Native acceptance, with I159 actual HMAC publication as its implementation prerequisite. Existing `std:datetime/instant` is a value type without an ambient time source; a trusted server clock remains a separate design/host dependency for actual RealWorld deployment, not a hidden side effect of JWT import.

**No Protos source or test change is represented by this record.**
