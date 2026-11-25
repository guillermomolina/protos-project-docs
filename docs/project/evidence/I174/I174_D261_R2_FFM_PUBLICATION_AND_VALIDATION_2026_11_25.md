# I174 / D261-R2 — Generic FFM publication and validation record

Date: 2026-11-25.

This is durable non-normative publication evidence, not a new design decision or
a public release. Canonical semantics remain in the product specification.
Implementation owner: [I174 #1166](https://github.com/guillermomolina/protos/issues/1166).
Design owner: [D261 #1158](https://github.com/guillermomolina/protos/issues/1158).
The exact P4-C candidate was explicitly selected in the
[owner approval](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7129273923) and
[ratified decision record](https://github.com/guillermomolina/protos-project-docs/blob/89e99f2ee37c586bb82b0630655bd47420a603b2/docs/project/decisions/language/D261_R2_P4_C_EXACT_PUBLIC_FFM_CONTRACT_SELECTED_2026_11_24.md).
The original [exact contract and GITHUB010/GITHUB021 review](https://github.com/guillermomolina/protos-project-docs/blob/e1b668d37f8e3973dab5081adec52f290fd28880/docs/project/evidence/D261/D261_R2_P4_EXACT_PUBLIC_CONTRACT_GITHUB010_GITHUB021_REVIEW_2026_11_24.md)
remain unchanged.

## 1. Verified publication identities

- PROTOS_REVISION: `ac4fa8d1ec29d7d30f741d03fb499165feaf2f93`.
- Baseline: `510a2c71e9c62cc3c147dea447c70d26e18d9d36`.
- [Exact seven-commit comparison](https://github.com/guillermomolina/protos/compare/510a2c71e9c62cc3c147dea447c70d26e18d9d36...ac4fa8d1ec29d7d30f741d03fb499165feaf2f93):
  64 changed paths, 12,555 additions and 410 deletions (GitHub compare totals).
- Remote main compared with the final SHA: identical, zero ahead/behind.
- Implementation version: `0.4.220-SNAPSHOT`; specification revision: `0.1.515`.
- JDK release 25; configured GraalVM version `25.5.5.0.3`.
- Both changelogs were read at the exact final SHA; each of the seven slices
  has matching implementation and normative revision entries.
- GitHub native parent and child endpoints both confirm I174 -> D261.
  This supersedes the pending hierarchy note at allocation.

| Slice | Exact published commit | Implementation | Spec | Delivered scope |
| --- | --- | --- | --- | --- |
| A | [`5951d3d3`](https://github.com/guillermomolina/protos/commit/5951d3d37ca31f52053820b74e7ebb6ba61b6298) | 0.4.214-SNAPSHOT | 0.1.509 | Fixed-width scalars, declarative layouts, prototype association, preserving bindings and RootActor authority. |
| B | [`8e2b2796`](https://github.com/guillermomolina/protos/commit/8e2b27968057b26ac0e4bf6659ed8b44c1a9e501) | 0.4.215-SNAPSHOT | 0.1.510 | Typed pointers, nullable/borrowed pointer results and original-owner provenance. |
| C | [`51bc7357`](https://github.com/guillermomolina/protos/commit/51bc735793bf22e2c4d9cbd93c3060955b1e47e9) | 0.4.216-SNAPSHOT | 0.1.511 | Bounded native memory, typed regions, slices, views, cType and trusted extent assertions. |
| D | [`e8a39c58`](https://github.com/guillermomolina/protos/commit/e8a39c58bfe50cd6d6cc9c798008c1b7f8b4cc18) | 0.4.217-SNAPSHOT | 0.1.512 | By-value aggregate ABI, independently owned group results and exact Native registration. |
| E | [`c78e5d38`](https://github.com/guillermomolina/protos/commit/c78e5d382a314491cae0ceacd076c24f41c60400) | 0.4.218-SNAPSHOT | 0.1.513 | Explicit union selection, Map/Array projection and staged writes. |
| F | [`86d01f21`](https://github.com/guillermomolina/protos/commit/86d01f216c9897d1c94f8a3c59f74b48bc1dc9c9) | 0.4.219-SNAPSHOT | 0.1.514 | Ordinary prototype snapshots and writeback, canonical ordinary reflection and complete focal coverage. |
| G | [`ac4fa8d1`](https://github.com/guillermomolina/protos/commit/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93) | 0.4.220-SNAPSHOT | 0.1.515 | Integrated portable/Native CLI conformance, Actor snapshots, SQLite installed validation, normative and guide reconciliation. |

## 2. Product sources and normative evidence

All links below are pinned to the final published product SHA:

- [FFI normative owner](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/spec/semantics/FFI.md),
  [SQLite owner](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/spec/semantics/SQLITE.md),
  [value/collection owner](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/spec/semantics/VALUES_AND_COLLECTIONS.md),
  [module owner](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/spec/semantics/MODULES.md),
  [error owner](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/spec/semantics/ERRORS.md),
  [Actor owner](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/spec/concurrency/ACTORS.md) and
  [P owner](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/spec/concurrency/PARALLEL_EXECUTION.md).
- [Implementation changelog](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/CHANGELOG.md),
  [specification changelog](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/spec/PROTOS_SPEC_CHANGELOG.md),
  [Maven metadata](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/pom.xml).
- [Public Protos module](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/protos/lib/ffi/module.protos),
  [generic FFM backend](https://github.com/guillermomolina/protos/tree/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/providers/ffm),
  [exact physical Native metadata](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/src/main/resources/META-INF/native-image/com.guillermomolina/protos/reachability-metadata.json).
- [Public FFI guide](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/docs/guide/library/ffi.md),
  [provider guide](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/docs/guide/interop/ffm-provider.md),
  [SQLite guide](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/docs/guide/library/sqlite.md).

## 3. Delivered contract and invariant reconciliation

The final normative inventory contains 13 scalar descriptors and 29 ordinary
Closures (42 public members). It retains R1 and adds the selected exact R2
surface. The reviewed source/test changes cover:

- G2 / D259: RootActor-only native authority; inert standard-module import;
  lazy, independently host-authorized library acquisition; trusted in-process
  C, not a sandbox. Declarative layouts need no native library; native memory
  can acquire an available provider session without acquiring a library.
- P2 / C1: independent immutable layout descriptors and exact prototype identity;
  indexed native regions versus ordinary slots; no implicit native field routing
  or per-C-struct Java classes.
- M3: independently owned bounded roots, views/slices, explicit trusted extent
  and access mode, original-source provenance, permanent revocation and borrow
  epoch; native root release does not advance unrelated borrow epochs.
- S2 / P4-C: exact signed/unsigned widths, explicit f32 narrowing, actual C
  layout and verified aggregate-by-value calls, explicit union paths, detached
  Map/Array/object snapshots, local-slot reflection and prepared writeback.
  Canonical ordinary method admission rejects incompatible overrides; no
  privileged read/write of object fields is added by the snapshot implementation.
- N1: exact finite physical Native Image registrations independently of nominal
  layout and library authorization; a separately authorized library may reuse
  an already registered shape; absent registration fails before C entry.
- R1 and SQLite: ordinary fresh bind Closures, output ordering, ownership,
  adoption and cleanup rules, synchronous calls and existing private SQLite
  length-carrying and borrowed-copy adaptation remain. No new SQLite-specific
  Java backend.
- Out of scope: C upcalls/retained callbacks, numeric address forging, arbitrary
  pointer arithmetic, packed/bitfield/flexible-array forms, direct C array
  by value, variadic ABI, long double and C++ class ABI, unverified platforms,
  dynamic provider JARs in Native Image, native sandbox guarantees.

GITHUB021 provenance is the approved P4-C packet; the published reconciliation
introduces no separately selected design candidate. This record does not
independently redefine semantics or claim exhaustive new proof of every invariant.

## 4. Validation evidence and provenance

The project owner explicitly reported on 2026-11-25 that the whole branch with
all commits was pushed to main, git diff check was clean, and all tests pass
locally. This is the final human-executor summary, received after the requested
I174-G validation sequence. Remote publication was independently checked as
described above. Test commands were not independently executed by the
coordinating agent.

Earlier focal I174-F consolidation output supplied by the owner directly showed
`ProtosI144FfmFixtureAbiTest`: **46 tests, 0 failures, 0 errors, 0 skipped,
BUILD SUCCESS**, Maven total time 38.433 s (2026-11-25 06:02:52 UTC).
This is a pre-final-slice observed log, not an invented final-G count.
The earlier D slice native report was also positive; no earlier run count is
substituted for a final-G run.

| Gate | Evidence classification |
| --- | --- |
| Clean git diff check | PASS, owner-reported |
| Local test suite | PASS, owner-reported final summary |
| Focal FFM 46/46 | PASS, directly supplied F-consolidation output |
| Python validator tests, portable JVM CLI, Native Image and installed Native distribution | Implemented gates; covered by the owner's general local-green report, not separately enumerated with final command transcripts or archive digests |
| Public release, artifact identity and installed archive digest | Not asserted; no release requested |
| Performance measurement | Not requested or claimed |

The requested final validation sequence included Python validator unittest,
`make test`, `make test-native`, and build/validation of a fresh Native
archive. No final per-command transcript, exact archive path, digest,
case count or timings were supplied. In particular, the broad owner summary
is not an independently observed `DIST005_NATIVE_COMPLETE_ADMISSION: PASS`
marker. Keep that distinction when interpreting acceptance.

## 5. Integrated test surface

- [JVM FFM fixture class](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/src/test/java/com/guillermomolina/protos/execution/foreign/api/ProtosI144FfmFixtureAbiTest.java):
  actual C layouts/scalars and group ABI, ownership and close diagnostics,
  pointers and bounded memory, Map/object snapshots and staged failures,
  no-library fixture and exact public inventory.
- [C and Protos fixtures](https://github.com/guillermomolina/protos/tree/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/protos/tests/foreign-provider):
  scalar/layout, typed pointers, native memory, group ABI, missing registration,
  RootActor/Actor/P isolation, no-library and R1 regression. G adds positive
  detached snapshot transfer and rejection of pointer-containing snapshots.
- [Portable provider runner](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/dist/test_external_foreign_providers.py),
  [Native smoke runner](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/scripts/native-test.sh),
  [installed Native validator](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/dist/validate_native.py):
  shared temporary R1 import adaptation, independently authorized two-library
  checks, exact C marker-file validation, no library acquisition memory case,
  SQLite query/TEXT/BLOB/local transactions and independent seed/reopen calls,
  missing-shape and no-bundled-fixture checks. Native no-library-acquisition
  uses an unused host alias; it is not an assertion of an empty CLI allowlist.
- [Python validator tests](https://github.com/guillermomolina/protos/blob/ac4fa8d1ec29d7d30f741d03fb499165feaf2f93/dist/tests/runtime/test_validate_native_ffm.py):
  classification, temporary source preservation, packaging guards and command
  wiring. Mocks establish wiring, not native execution.

The G publication changes tests, validators, documentation and normative
reconciliation, not Java or Protos production implementation, SPI or Native
signature registrations. The generic provider fixture relocation is included
in its published implementation changelog (directory-file limit).

## 6. Coordination and remaining evidence boundaries

Product publication is complete and local tests are green per the owner.
The exact selected design and the seven publication slices are linked above.
No new formal work, priority, milestone, release or semantic amendment is
allocated by this record. Live Issue/Project state remains in the product
repository. The final issue work log must name the exact published
PROJECT_RECORD_REVISION (the Git commit containing this record); this file
cannot embed its own commit hash. Any separately required installed-distribution
acceptance detail should be linked from that work log rather than fabricating
an artifact or execution result.

No independent final execution, performance result, or public release is
claimed. The human published product commits; the owner explicitly authorized
this bounded project-docs publication.
