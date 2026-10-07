# AUD009-E3 — Standard Library integer math and deterministic hashing complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#649`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline:
`1b4065f0f79a5c2837b4462f00b3b8e481d11b98`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Initial checkpoint proposal: `guillermomolina/protos#649`, issue comment
`5739876278`.

Owner correction: `guillermomolina/protos#649`, issue comment
`5739881096`.

Owner approval provenance: `guillermomolina/protos#649`, issue comment
`5739883474`, 2026-09-19.

Derived implementation routes: **NONE**

## Purpose

AUD009-E3 reviewed the remaining pure algorithmic Standard Library modules:

```text
std:math/Integer
std:crypto/SHA256
```

The review explicitly tested whether current low/no internal use justified
removal, while applying AUD009's anti-simplification rule against deleting
valuable foundational capability merely because its next consumer has not yet
landed.

AUD009-E3 itself authorizes no normative or implementation change.

## Evidence summary

### std:math/Integer

Current public surface:

```text
gcd(a, b)
lcm(a, b)
factorial(n)
pow(base, exponent)
powMod(base, exponent, modulus)
```

At the evidence baseline, no current production repository source imports
`std:math/Integer`; references are confined to its tests/design and example
planning.

The module is nevertheless very small and institutionally light:

- ordinary Protos source;
- no runtime/native family;
- no authority;
- no global state;
- no host/JVM dependency;
- no implicit numeric widening;
- all behavior is defined over retained exact unbounded Core Integer semantics.

The project owner identified an important requirement missing from the initial
removal proposal: this library is basic foundation expected by upcoming
distribution work.

That changes the interpretation of zero current consumers. It is a timing fact,
not evidence that the capability is premature.

Removing the module and then reconstructing essentially the same basic integer
algorithms for near-term distribution work would be churn without reducing a
meaningful ongoing architectural burden.

### std:crypto/SHA256

Current public surface:

```text
digest(bytes) -> fresh Bytes[32]
```

Current productive consumers include:

```text
protos/tools/package/ProjectMetadata.protos
protos/tools/package/ContentIdentity.protos
protos/tools/package/ResolutionInput.protos
```

These use SHA-256 for canonical project/content/resolution-input identity and
staleness checks.

The module remains a tiny reusable public owner for deterministic unkeyed hashing
without introducing a generic crypto framework.

## Final classification

```text
STDLIB_MATH_INTEGER=KEEP
INTEGER_GCD=KEEP
INTEGER_LCM=KEEP
INTEGER_FACTORIAL=KEEP
INTEGER_POW=KEEP
INTEGER_POW_MOD=KEEP

STDLIB_CRYPTO_SHA256=KEEP
SHA256_DIGEST=KEEP
SHA256_PURE_DETERMINISTIC_CONTRACT=KEEP
```

No E3 mechanism is classified `REMOVE_NOW_RECONSIDER_LATER` or
`REMOVE_PERMANENTLY`.

## std:math/Integer remains

The approved KEEP classification rests on:

- near-term foundational role for distribution work;
- low institutional/maintenance burden;
- ordinary-source implementation over stable Core Integer semantics;
- no duplicate runtime or authority model;
- high likelihood of direct reuse;
- avoiding removal/reconstruction churn.

The existing five-operation surface remains:

```text
gcd
lcm
factorial
pow
powMod
```

E3 does not broaden that surface.

### powMod boundary remains unchanged

`powMod` remains mathematical modular exponentiation only.

It does **not** gain:

```text
constant-time guarantee
cache-oblivious guarantee
side-channel resistance guarantee
secret-key handling guarantee
cryptographic primitive status
```

Keeping the module does not pre-authorize cryptographic reuse beyond its exact
mathematical contract.

Classification: **KEEP**.

## std:crypto/SHA256 remains

The SHA-256 module remains because it already has real Package Tool consumers and
its one-operation public API is the smallest reusable owner for the demonstrated
need.

The current pure-Protos implementation remains implementation detail. Future
acceleration/intrinsification may be considered separately if justified while
preserving the same observable digest contract.

Classification: **KEEP**.

## Deliberate absences remain absent

```text
broad std:math/Math function module           ABSENT / RETAIN ABSENCE
BigInteger library value family               ABSENT / RETAIN ABSENCE
generic Number utility hierarchy              ABSENT / RETAIN ABSENCE
implicit mixed-family math                    ABSENT / RETAIN ABSENCE
cryptographic constant-time integer math      ABSENT / RETAIN ABSENCE

hash algorithm registry                       ABSENT / RETAIN ABSENCE
generic Hash interface/object                 ABSENT / RETAIN ABSENCE
digestHex convenience                         ABSENT / RETAIN ABSENCE
incremental SHA256 state                      ABSENT / RETAIN ABSENCE
stream/File/TextReader hashing                ABSENT / RETAIN ABSENCE
HMAC / KDF / password hashing                 ABSENT / RETAIN ABSENCE
signatures / ciphers                          ABSENT / RETAIN ABSENCE
entropy/randomness                            ABSENT / RETAIN ABSENCE
native/JVM crypto public contract             ABSENT / RETAIN ABSENCE
```

## Strongest attempted removals

```text
remove std:math/Integer because no production consumer exists
    rejected -> owner confirms near-term distribution-foundation role;
                current zero-use is temporal, not structural evidence

keep only a subset of Integer math
    rejected -> arbitrary narrowing with no demonstrated simplification benefit

keep powMod only for future crypto
    rejected as rationale -> its contract is mathematical, not constant-time
                             cryptographic arithmetic

move integer algorithms into Core
    rejected -> expands permanent Core unnecessarily

remove SHA256 and duplicate it privately in Package Tool
    rejected -> destroys reusable ownership while preserving the same cost

replace SHA256 with JVM MessageDigest
    rejected -> unnecessary host/runtime dependency; backend optimization is
                separate from public API ownership

broaden SHA256 into generic crypto/hash registry
    rejected -> speculative institution beyond current evidence
```

## Required routing

No implementation removal/redesign route is required.

```text
REMOVAL_ROUTES=NONE
DERIVED_LIBXXX=NONE
DERIVED_DXXX=NONE
```

Future distribution consumers may use the retained `std:math/Integer` surface
without reopening E3. Any new math operations still require ordinary independent
justification.

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_BASELINE=1b4065f0f79a5c2837b4462f00b3b8e481d11b98

STDLIB_MATH_INTEGER=KEEP
STDLIB_CRYPTO_SHA256=KEEP

REMOVAL_ROUTES=NONE
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_E3_CLASSIFICATION=COMPLETE
```

AUD009-E3 is complete.
