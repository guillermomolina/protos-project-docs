# BUG031 — Exact Fraction and Complex REPL diagnostic rendering (0.3.424)

**Date:** 2026-10-30. **Role:** immutable, non-normative implementation-publication and validation evidence for [BUG031 #997](https://github.com/guillermomolina/protos/issues/997).

## Published identity

```text
WORK_ITEM=BUG031
ISSUE=guillermomolina/protos#997
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=80b2604f4dfd82e3442e72b1a97dd86a10fd3aff
IMPLEMENTATION_VERSION=0.3.424-SNAPSHOT
PRODUCT_CHANGED_PATH_COUNT=5
SPECIFICATION_CHANGE=NO
DURABLE_RECORD_DECISION=REQUIRED_BY_OWNER_REQUEST
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

The [exact published product commit](https://github.com/guillermomolina/protos/commit/80b2604f4dfd82e3442e72b1a97dd86a10fd3aff), message `BUG031: render Fraction and Complex values in the REPL diagnostic inspector (0.3.424)`, was verified through GitHub. Its five changed paths are:

1. `src/main/java/com/guillermomolina/protos/cli/ProtosDiagnosticInspector.java`
2. `src/test/java/com/guillermomolina/protos/cli/ProtosDiagnosticInspectorTest.java`
3. `src/test/java/com/guillermomolina/protos/cli/ProtosReplTest.java`
4. `pom.xml`
5. `CHANGELOG.md`

The product changelog at that exact revision records `0.3.424-SNAPSHOT`. No `spec/**`, `protos/lib/**`, Core numeric support or `ProtosValueRenderer` source was modified.

## Defect and exact repair

The REPL evaluated `1/5` and `Complex(0,1)` successfully but displayed `<value>`, because the bounded diagnostic inspector recognized Integer and Float but omitted the canonical Fraction and Complex families.

The published inspector authenticates Fraction with `ProtosNumericValueSupport.isFraction`, reads the canonical numerator and denominator through existing private-family accessors, and renders exact decimal `n/d`. It authenticates Complex with `ProtosComplexNumericSupport.isComplex` and recursively renders the existing exact real-number scalar components as `Complex(real, imaginary)`. Examples: `1/5`, `-2/3`, `Complex(0, 1)`, `Complex(1/2, -0.0)` and `Complex(NaN, Infinity)`. An exactly integral Fraction result remains an Integer.

The new numeric branches use the existing bounded `RenderState`; they do not introduce a formatting protocol, guest message invocation, Float conversion of exact components, host reflection or a new Core value kind. D063's `inspect != print != serialize` separation remains intact. Unrecognized sealed families and authority-bearing capabilities remain opaque; ordinary objects delegating to numeric prototypes are not misidentified as genuine numbers.

## Published regression coverage

- `ProtosDiagnosticInspectorTest`: canonical Fraction signs/reduction and large Integer components; Complex exact components, signed Float zero, NaN and infinities; Array and local Object slot embedding; maximum structural depth, cycles and inherited-slot exclusion; negative forged/delegating and unrelated sealed-family cases; explicit Core scalar coverage and intentionally opaque capabilities.
- `ProtosReplTest`: actual REPL result sequence `0`, `1/5`, `1`, `Complex(0, 1)`, `Complex(1/2, -0.0)`; nested Arrays and Maps; ordinary structural rendering for `IpAddress`, `IpEndpoint` and RFC 3986 URI objects, plus quoted URL Strings. A stable marker isolates the tested value output without assuming object-slot order or compact layout.

The Core's numeric family authentication remains distinct from Standard Library object presentation. The patch does not add IP dotted-decimal or reconstructed URI rendering.

## Human validation and provenance

| Gate | Reported result | Provenance |
| --- | --- | --- |
| Focused `ProtosDiagnosticInspectorTest,ProtosReplTest` | **PASS** | Maintainer reported focal PASS during implementation; subsequent final marker edit was covered by the full-suite report |
| `git diff --check` | **CLEAN** | Maintainer report for the pushed publication |
| `make test` | **PASS** | Maintainer report, local integrated suite |
| `make check` | **PASS** | Maintainer report, local |
| Exact product commit | **VERIFIED** | GitHub commit `80b2604f4dfd82e3442e72b1a97dd86a10fd3aff`, five paths and exact changelog |

The evidence-recording agent did not run Maven, `make`, or the REPL independently. No CI, separate native-image test, or unreported run is claimed. This is a publication snapshot and not an alternative normative specification or live status ledger.

## Authority and closure scope

- [D063 #314](https://github.com/guillermomolina/protos/issues/314): bounded non-evaluating CLI inspection, independent of program output and serialization.
- [BUG031 #997](https://github.com/guillermomolina/protos/issues/997): acceptance and live closure coordination.
- Normative numeric family behavior stays in `spec/semantics/VALUES_AND_COLLECTIONS.md` of the product repository; no normative change is made here.

The final Issue closure comment records this evidence publication's exact `PROJECT_RECORD_REVISION`. This record deliberately carries the immutable product SHA rather than a moving `main` reference.
