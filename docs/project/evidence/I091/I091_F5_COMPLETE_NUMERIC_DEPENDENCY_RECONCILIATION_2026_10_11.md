# I091-F5 — Complete numeric-dependency reconciliation at `ced3746f9664`

**Audit date:** 2026-10-11 (Europe/Madrid).
**Owning issue:** [I091/#869](https://github.com/guillermomolina/protos/issues/869).
**Current product:** [`ced3746f9664ec5ceb562461c077bdf0627e5663`](https://github.com/guillermomolina/protos/commit/ced3746f9664ec5ceb562461c077bdf0627e5663), `0.3.331-SNAPSHOT`.
**Census baseline:** [`76de43651079172d90be5bc6b1f535336ae47cfb`](https://github.com/guillermomolina/protos/commit/76de43651079172d90be5bc6b1f535336ae47cfb); its production source was unchanged from `eba6250f96aa6873829f7011a3a1aee7b6969887` (I089) and the human reported a full `tools/numeric_dependency_census.py` count of 683.
**Historical baseline fingerprint:** `605cd68c3e75bbc16cab54bbb89770161f2e5f59b21b0c77875e9c01f68f8034`.
**Current operator-reported fingerprint:** `52cb776b3931489ecd237b172ef958d575f580b606bab8e7c74b7beeeba8e817`, from the canonical script executed at the exact published HEAD.
**Previous audit:** [I091 at `ced3746f`](I091_CED3746_BIG_INTEGER_AND_CLOSURE_AUDIT_2026_10_10.md).
**Authority:** [PLAT056 Candidate C](../../decisions/platform/PLAT056_PRIMITIVE_FIRST_NUMERIC_GUEST_OBJECT_ARCHITECTURE.md). Do not implement D197's future public recognition/numeric tower under this evidence.

## Executive result

**All production-source token references are counted and assigned to source owners. This lexical/ownership inventory is NOT proof that all references are architecturally necessary. A subsequent contract-level counterexample confirmed that the public foreign-provider SPI unnecessarily requires `java.math.BigInteger` for *every* integral scalar, including signed-64 values; see the corrective finding below.**

| Audited source-code identifier | Baseline | Product at `ced3746f` | Removed | Reduction |
| --- | ---: | ---: | ---: | ---: |
| `BigInteger` (including `java.math.BigInteger`) | 447 | **73** | 374 | 83.67% |
| `ProtosIntegerValue` | 187 | **59** | 128 | 68.45% |
| `ProtosFloatValue` | 49 | **39** | 10 | 20.41% |
| **Total** | **683** | **171** | **512** | **74.96%** |

- Baseline: **431** `src/main/java/**/*.java` files, **77** with these tokens.
- Current: **436** production Java files, **20** with these tokens.
- The 171 surviving tokens comprise **21 import tokens** (14 BigInteger, 2 Integer carrier, 5 Float carrier) and **150 non-import code tokens** (59 BigInteger, 57 Integer carrier, 34 Float carrier).
- **No reduction quota was approved or used.** A remaining Java `BigInteger` occurrence is not automatically undesirable, and even an exact large guest hash or explicit public Java ABI must retain unbounded precision.

## Reproducible source reconciliation, not indexed search

The canonical repository script `tools/numeric_dependency_census.py` was read and its `mask_noncode` / `PATTERN` token-counting rule reproduced over pinned GitHub Git blobs in read-only connector code. This was the initial audit, **not** a local script invocation by the auditor. **Subsequently, the human executor ran the canonical script at the pinned published HEAD and provided its terminal output; the observed 171-token totals are identical to the audit prediction.** The reported JSON/TSV output files remain in the human's local `target/`; their contents have not been independently ingested or committed to the documentation repository. This distinction preserves validation provenance.

The two immutable recursive production Git trees were compared by exact relative pathname and Git blob SHA:

- **99 changed path names** across the two revisions: **92 modified paths present at both revisions, six added, one removed**.
- Exactly **338 unchanged Java blobs** have byte-identical Git blob IDs; their exact old numeric-symbol counts therefore carry forward.
- All 99 changed path names were visited. For each available pre/post blob, count the three identifiers with the canonical script's word-boundary rule while masking Java comments, strings and characters.
- The old changed-path subset totals **443 / 187 / 49**; the new changed-path subset totals **69 / 59 / 39** (`BigInteger` / `ProtosIntegerValue` / `ProtosFloatValue`).
- Consequently, the unchanged blobs contribute **4 / 0 / 0** at both ends. The four BigInteger occurrences are in `execution/ProtosHostJavaCatalogue.java` (2) and `spi/foreign/ProtosForeignValueClass.java` (2); their exact source was checked.
- Result: **69+4=73, 59+0=59, 39+0=39**, hence **171 total in 20 matching production files**.

This is an exhaustive **revision-pinned differential lexical result** from the prior exhaustive census, now **cross-checked with a canonical Python script run by the human executor**; it is not an estimate based on potentially stale GitHub indexed code searches. It counts source tokens, **not JVM allocations, classes loaded, Truffle graph nodes or runtime cost**. Git commit, per-path blob identities and the script's reported current SHA-256 fingerprint pin the source version.

### Module distribution

| Module | Baseline: Big / Integer / Float | Current: Big / Integer / Float | Baseline total | Current total |
| --- | --- | --- | ---: | ---: |
| `cli` | 0 / 5 / 3 | 0 / 0 / 0 | 8 | 0 |
| `execution` | 213 / 115 / 38 | 10 / 11 / 28 | 366 | 49 |
| `runtime` | 228 / 67 / 8 | 58 / 48 / 11 | 303 | 117 |
| `spi` | 6 / 0 / 0 | 5 / 0 / 0 | 6 | 5 |
| All other modules | 0 / 0 / 0 | 0 / 0 / 0 | 0 | 0 |
| **Total** | **447 / 187 / 49** | **73 / 59 / 39** | **683** | **171** |

## Classified complete remaining file inventory

Every survivor is assigned an ownership classification below. Counts are **all matching tokens (imports included)**; a token in a file inherits the stated numeric ownership, with the explanatory sub-boundaries noted where a file spans more than one responsibility.

`B` = Java `BigInteger`; `I` = `ProtosIntegerValue`; `F` = `ProtosFloatValue`.

| Relative file below `src/main/java/com/guillermomolina/protos/` | B | I | F | Source responsibility / audit disposition |
| --- | ---: | ---: | ---: | --- |
| `runtime/ProtosBinary64Rounding.java` | 14 | 3 | 0 | **JUSTIFIED — EXACT ALGORITHM / HOST PROJECTION.** Arbitrary-precision rounded quotient, exact large-vs-binary64 comparison and binary64 integral extraction; signed-64/signed-64 quotient stays primitive |
| `runtime/ProtosNumericValueSupport.java` | 14 | 42 | 9 | **JUSTIFIED — SEMANTIC NUMERIC OWNER.** Recognizes physical carriers, normalizes signed-64/large Integer, exact large arithmetic, host scalar boundary, hashes and materialization |
| `runtime/ProtosNumberLiteral.java` | 2 | 0 | 0 | **JUSTIFIED — PARSING.** `BigInteger` used only after the primitive literal accumulator cannot represent the magnitude |
| `runtime/ProtosLargeIntegerValue.java` | 5 | 0 | 0 | **JUSTIFIED — RICH CARRIER.** Narrow immutable payload and explicit host BigInteger projection, subject to separate F1/F4 representational acceptance |
| `runtime/ProtosNumericHashKey.java` | 3 | 1 | 0 | **JUSTIFIED — EXACT HASH.** Keeps large semantic Map hash keys exact; no truncation to `long` |
| `runtime/ProtosIntegerValue.java` | 3 | 2 | 0 | **JUSTIFIED — SIGNED-64 CARRIER / INTEROP.** Physical small-Integer class and explicit Truffle `asBigInteger` |
| `runtime/ProtosFloatValue.java` | 2 | 0 | 2 | **JUSTIFIED — FLOAT CARRIER / INTEROP.** Physical Float class and exact-integral host `asBigInteger` projection |
| `runtime/ProtosFixedIntegerInteropValue.java` | 5 | 0 | 0 | **JUSTIFIED — HOST ABI.** Fixed-width unsigned 64-bit carrier with exact arbitrary-precision interop projection |
| `runtime/ProtosSemanticTransferPayload.java` | 10 | 0 | 0 | **JUSTIFIED — PLAT051 TRANSFER CONTRACT.** Inert exact `BigInteger` leaves, including small leaf materialization at an expressly requested transport boundary |
| `execution/CanonicalToBytecodeLowerer.java` | 2 | 0 | 0 | **JUSTIFIED — LARGE LITERAL DESCRIPTOR.** Only truly large constants materialize an arbitrary-precision host descriptor |
| `execution/ProtosSemanticBytecodeRootNode.java` | 1 | 5 | 5 | **JUSTIFIED — SELECTED NUMERIC FAST-PATH / MATERIALIZATION.** Guarded primitive numeric operations, standard method selection and actual rich literal observation; not a generic VM client |
| `execution/ProtosCurrentNumericRelations.java` | 0 | 6 | 7 | **JUSTIFIED — NUMERIC EQUALITY/ORDER.** Compact carrier specialization plus numeric service fallback |
| `execution/ProtosStandardFloatProtocol.java` | 0 | 0 | 6 | **JUSTIFIED — STANDARD FLOAT PROTOCOL.** Numeric receiver validation and Float operations |
| `execution/ProtosStandardIntegerProtocol.java` | 0 | 0 | 6 | **JUSTIFIED — NUMERIC MIXED ARITHMETIC.** D196 ordinary Float operand handling, normal selected method and fallback |
| `execution/ProtosStandardNumericConversionProtocol.java` | 0 | 0 | 4 | **JUSTIFIED — NUMERIC CONVERSION.** Explicit Float recognition and construction at semantic conversion boundary |
| `execution/ProtosForeignPluginValueAdapter.java` | 2 | 0 | 0 | **DESIGN DEFECT — PUBLIC SPI BRIDGE.** The current API forces host BigInteger for every exact integral scalar, including signed-64 values; contrary to pay-as-you-grow; requires compatibility-aware remediation |
| `execution/ProtosHostJavaProvider.java` | 3 | 0 | 0 | **JUSTIFIED — JAVA HOST ABI.** Reflective parameters explicitly declaring `BigInteger.class` and lossless projection |
| `execution/ProtosHostJavaCatalogue.java` | 2 | 0 | 0 | **JUSTIFIED — JAVA HOST CATALOGUE.** Explicit supported Java BigInteger class |
| `spi/foreign/ProtosForeignValueClass.java` | 2 | 0 | 0 | **DESIGN DEFECT — PUBLIC FOREIGN ABI.** `integral(BigInteger)` is the only exact Integer constructor; cannot publish a `Long` without arbitrary-precision boxing |
| `spi/foreign/polyglot/ProtosPolyglotValueOperations.java` | 3 | 0 | 0 | **DESIGN DEFECT / SAFE LARGE-INTEGER FALLBACK.** Polyglot classify calls `asBigInteger()` unconditionally even for signed-64; outbound `convert` and array `index` convert back to primitive; exact out-of-range guards must remain but routine allocations must not |
| **Total** | **73** | **59** | **39** | **171 audited source-token occurrences** |

**Import classification:** 14 B, 2 I, 5 F = 21. All imports belong to the matched source owners above, not to generic filesystem, network, Map or CLI code. The non-import code uses sum to 150. All 171 tokens have **source owners**, but the ABI/SPI design is **not fully accepted**: the foreign-provider integer transport sites identified below are categorized **DESIGN DEFECT**, not `JUSTIFIED`. The stock census tool still prints `CLASSIFICATION: PENDING SOURCE AUDIT` because it does not incorporate this separate semantic review. The stock census script still prints `CLASSIFICATION: PENDING SOURCE AUDIT` because its generated inventory deliberately does not import these source-level justifications; that tool message is expected and does not invalidate this separate completed review.

### F5 architectural conclusions

1. **The earlier widespread Java BigInteger coupling has been contained by source owner, but the surviving public SPI contract still exhibits avoidable `BigInteger` coupling for small integers.** At this revision, the generic File/Byte/Network, Array/Map/IdentityMap, Actor/P/Process and CLI code does not directly mention these audited representation classes except through the owning services; the exact large hash carrier remains intentionally specialized.
2. **Unbounded exactness is not removable.** The truly large Integer payload, correct exact-rational conversion, exact semantic hash key, public Java interop projections and the PLAT051 payload contract are valid reasons for using `java.math.BigInteger`.
3. **Count reduction does not prove performance.** F2 structural bytecode behavior and PERF040 graph/timing studies retain independent acceptance. Do not demand Java BigInteger be eliminated from valid rich/ABI paths.
4. **Representation observability remains separately open.** `ProtosLargeIntegerValue extends ProtosObjectValue` can reach generic reflection/debugger/interop consumers. This requires F1/F4 reachability proof even if the subclass itself uses a legitimate BigInteger payload.
5. **The lexical F5 census requires no mass source replacements.** A separate *targeted* public-SPI contract repair is justified; do not silently break already-compiled external plugin JARs or change D188/D197 normative semantics as part of a cosmetic cleanup.


## Correction — public foreign-provider ABI forces BigInteger for signed-64 scalars

**2026-10-11, subsequent owner counterexample: the earlier blanket `JUSTIFIED` classification was too permissive.** The source-owner census and its 171 reference count are **still exact**; the inference that every remaining occurrence is *necessary* is **retracted**.

**Observed inbound path at pinned product `ced3746f`:**
1. `ProtosPolyglotValueOperations.classify`, `case INTEGRAL`: checks `polyglot.fitsInBigInteger()` then **always** calls `polyglot.asBigInteger()`; does not try `fitsInLong()/asLong()` first.
2. The public `ProtosForeignValueClass.integral(BigInteger)` is the **only** `INTEGER` factory. Its `scalar()` is therefore always `java.math.BigInteger`, even for 42.
3. `ProtosForeignPluginValueAdapter.classify` casts that scalar to `BigInteger` and forwards it to the internal `ProtosForeignAdmissionDescriptor.integral(Number)`.
4. The internal descriptor calls `ProtosNumericValueSupport.normalizedHostInteger`, immediately **demoting** every signed-64 `BigInteger` back to `Long`.
5. `ProtosForeignValueAdmission.admit` calls `ProtosNumericValueSupport.integerFromHost`, retaining the ordinary signed-64 Integer value.

**Observed outbound path:**
1. `ProtosForeignValueAdmission.exportOrNull` calls `ProtosNumericValueSupport.hostInteger`; for a small Integer this is a `Long`.
2. `ProtosForeignPluginValueAdapter.publicArgument` explicitly wraps **every** such `Long` as `BigInteger.valueOf(small)`, because the public `ProtosForeignArgumentValue` contract requires host `BigInteger` for integral values.
3. `ProtosPolyglotValueOperations.convert` **demotes** in-range `BigInteger` to `long`; `index` also assumes a `BigInteger` scalar and rejects anything else. Range checking itself is legitimate; the compulsory intermediate BigInteger is not.

**Decision comparison:**
- [D188](https://github.com/guillermomolina/protos/issues/819) approves **source-semantically classified, losslessly admitted integral values**, not a mandatory Java `BigInteger` scalar at the external SPI.
- [D197](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md) ratifies **guest-visible signed-64 Integer distinct from guest-visible BigInteger**, Fraction and Complex; Java `java.math.BigInteger` is a **backend helper** and never a guest family definition. D197's complete numeric-tower semantics are **not yet installed** in the current runtime.
- [PLAT056](../../decisions/platform/PLAT056_PRIMITIVE_FIRST_NUMERIC_GUEST_OBJECT_ARCHITECTURE.md) explicitly requires primitive pay-as-you-grow operation paths and allows Java BigInteger only when materially justified. An *explicit* host boundary permits necessary rich integer representation but does not make *unconditional* small-integer BigInteger allocation necessary.
- The inline `// D188: the public SPI declares every exact Integer as a host BigInteger` attributes an **implementation choice** to D188; it is **not D188's normative wording**. This comment must not be treated as semantic authority.
- The earlier D188 rule says unbounded Protos Integer; D197 **prospectively supersedes** that family model with a signed-64 Integer and distinct BigInteger guest family. This unresolved future semantic reconciliation is separate from immediately avoiding pointless host allocations in the provider SPI.

**External comparison:** GraalVM [InteropLibrary](https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/interop/InteropLibrary.html) and [polyglot Value](https://www.graalvm.org/truffle/javadoc/org/graalvm/polyglot/Value.html) expose distinct `fitsInLong/asLong` and `fitsInBigInteger/asBigInteger` operations, rather than forcing host BigInteger for every numeric value. GraalVM's [interop migration guidance](https://github.com/oracle/graal/blob/master/truffle/docs/InteropMigration.md) advises selecting compatible primitive channels first and avoiding indiscriminate unboxing.

**Remediation scope recommended for separately authorized implementation:** keep a source-declared exact-integral classification independent of Java carrier; provide **signed-64 `long` transport by default**; allow an exact, separately admitted large-magnitude representation only when the value is actually outside signed-64; preserve large-value fidelity, source family provenance, negative index rejection and the normal Protos Integer/BigInteger guest-family distinction when D197 is later implemented. Address **both inbound and outbound plugin paths**, Polyglot classification/conversion/index, public SPI argument contract, external-JAR compatibility/versioning and source-level black-box tests. No general Java BigInteger substitution, no Fraction/Complex automatic-admission change and no unapproved `Integer.recognizes` semantics.

**Current disposition:** `F5_LEXICAL_CENSUS=PASS`; `F5_SOURCE_OWNERSHIP=COMPLETE`; `F5_ARCHITECTURAL_JUSTIFICATION=REOPENED_FOR_PUBLIC_SPI_SMALL_INTEGER`; `PUBLIC_SPI_BIG_INTEGER_FOR_LONG=DEFECT_CONFIRMED`; `FIX_IMPLEMENTED=NO`; `PRODUCT_TESTS_RUN_IN_THIS_AUDIT=NO`. The previously asserted `UNJUSTIFIED_MATCHES_CONFIRMED=0` is explicitly **withdrawn**.

## F5 disposition, limitations and coordination

**F5-SOURCE-CENSUS=PASS (exact immutable Git-blob differential, whole-tree count).**
**F5-PER-OCCURRENCE-OWNERSHIP=CLASSIFIED (20 files / 171 token sites, by source owner).**
**F5-PER-OCCURRENCE-DESIGN_JUSTIFICATION=FAILED_AT_PUBLIC_SPI_SMALL_INTEGER_BOUNDARY.**
**UNJUSTIFIED_SMALL_INTEGER_BIG_INTEGER_ABI_REQUIREMENT=CONFIRMED.**
**CANONICAL_PYTHON_SCRIPT_EXECUTED_BY_HUMAN=YES; AUDITOR_EXECUTED=NO.**
**CURRENT_SCRIPT_SHA256_FINGERPRINT=52cb776b3931489ecd237b172ef958d575f580b606bab8e7c74b7beeeba8e817 (operator report).**
**SCRIPT_JSON_TSV=REPORTED_CREATED_IN_LOCAL_TARGET; FILE_CONTENTS_NOT_INGESTED.**
**PRODUCT_SOURCE_EDITED=NO; TESTS_OR_BENCHMARKS_RUN_HERE=NO.**

### Canonical-script execution: operator-supplied terminal evidence

Human executor executed the following commands from the root of `guillermomolina/protos`:

```console
$ git rev-parse HEAD
ced3746f9664ec5ceb562461c077bdf0627e5663
$ python3 tools/numeric_dependency_census.py
I091-A NUMERIC DEPENDENCY CENSUS
HEAD: ced3746f9664ec5ceb562461c077bdf0627e5663
Fingerprint: 52cb776b3931489ecd237b172ef958d575f580b606bab8e7c74b7beeeba8e817
Production Java files: 436
Matching files: 20
Total references: 171
  BigInteger: 73
  ProtosFloatValue: 39
  ProtosIntegerValue: 59
Modules:
  execution: 49
  runtime: 117
  spi: 5
JSON: target/i091-numeric-census.json
TSV: target/i091-numeric-census.tsv
CLASSIFICATION: PENDING SOURCE AUDIT
```

**Independent reconciliation outcome:** exact match for HEAD, production file count, matching file count, all three symbol subtotals, total references and module subtotals. No mismatch requiring a source investigation or rerun. The stock script's `PENDING SOURCE AUDIT` denotes the inventory's deliberately unassigned per-occurrence review state; the **completed 20-file, 171-site ownership classification is in this document**, not written into the generated TSV/JSON.

The source fingerprint was emitted by the script as reported by the human. The auditor did not receive the script-generated JSON/TSV contents and therefore does not claim to have opened or hashed those files directly. The user had already reported tests PASS on the published code; the census does not require repeating them when no source changes have been made.

I091 remains **OPEN** until F1/F2/F3/F4 and final acceptance/closure evidence are satisfied. Future visible D197 BigInteger/Fraction/Complex and unresolved `Integer.recognizes` belong to I090, while pinned compiled-graph/timing measurements belong to PERF040. This F5 report does not claim either is completed or unblocked.

The present report is durable audit evidence, not a normative specification amendment or a change in approved PLAT056 Candidate C.
