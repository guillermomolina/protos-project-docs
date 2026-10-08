# LIB021-A — explicit std:interop operation surface implementation

Status: **SUBSTANTIVE IMPLEMENTATION COMPLETE — ISSUE CLOSURE BLOCKED ON PUBLICATION METADATA REPAIR**

Issue: `guillermomolina/protos#835`

Decision authority: `D193 / guillermomolina/protos#836`

Normative reconciliation: `I084 / guillermomolina/protos#837`

Publication date: **2026-10-08**

## Published product revision

~~~text
PROTOS_REVISION=5decd57c4297481d838908ae9c47ba18de3b742b
COMMIT_SUBJECT=LIB021-A: add explicit std:interop operation surface
PARENT_REVISION=6ae0a9b8a992b80d802fe9273a1170687a271edb
IMPLEMENTATION_VERSION_IN_POM=0.3.281-SNAPSHOT
SPECIFICATION_REVISION=0.1.448
~~~

The product commit is current `main` at the time of this evidence capture.

## Implemented public surface

The ordinary Standard Library module:

~~~text
std:interop
~~~

publishes exactly:

~~~text
invoke(target, ...arguments)
instantiate(target, ...arguments)
readMember(target, name)
writeMember(target, name, value)
~~~

The module source is `protos/lib/interop.protos`. It captures the private
runtime-provisioned `_interopFacility`, publishes the four ordinary Protos
wrappers, and removes the bootstrap slot before the module surface is observed.

## Runtime implementation

LIB021-A adds the private frozen/stateless `ProtosForeignInteropFacility` and
reuses the existing D188/D189 substrate rather than introducing a second foreign
value model.

The implementation preserves:

~~~text
D188_DELTA=NONE
D189_DELTA=NONE
PLAT052_DELTA=NONE
PLAT053_DELTA=NONE
D192_DELTA=NONE
~~~

Notable implementation details retained by the published commit:

- `invoke` uses the existing provider-neutral `execute` operation and
  `EXECUTABLE` classification without ordinary Protos `call` lookup;
- `instantiate` adds the provider-neutral explicit instantiation operation and
  requires `INSTANTIABLE` before provider entry;
- private `MEMBER_READ` and `MEMBER_WRITE` capabilities distinguish explicit
  member families from ordinary projection fidelity;
- `readMember` bypasses ordinary member projection/fidelity only for the
  explicit operation, allowing protected names such as `call`, `at`,
  `atPut`, `each`, `==`, and `hash` to be reached explicitly;
- `writeMember` returns the exact original Protos `value` after successful
  foreign mutation, with no readback or result readmission;
- raw foreign references and attached foreign module facades use the same exact
  `ProtosForeignHandle`, provider, session, and generation;
- outbound arguments and Closure callbacks reuse
  `ProtosForeignProjectedOperations.Outbound`, D188 admission, D189 callback
  scope/lifetime, and `ProtosForeignOperation` entered/not-entered failure
  handling;
- importing `std:interop` itself creates no provider/session/Context,
  discovery, acquisition, or authority;
- the restricted host-Java provider gains only explicit member-read capability
  over its already-catalogued members; no reflection/discovery/authority surface
  is published.

Deferred D193 families remain absent:

~~~text
invokeMember
public capability predicates
explicit indexed/hash APIs
iterator/cursor APIs
explicit conversion
metaobject/type/language/source/display metadata
provider/language discovery
foreign acquisition
retained/asynchronous callbacks
~~~

## Test evidence in the published commit

The commit adds `ProtosForeignInteropTest` and extends the existing foreign
fixtures/tests rather than introducing a parallel test universe.

The retained tests cover at least:

- exact four-slot public module surface and hidden bootstrap facility;
- zero-use import with no foreign provider/session activity;
- distinct explicit `invoke` and `instantiate`, including a target supporting
  both while ordinary D188 callability remains unchanged;
- explicit protected-name reads and reads that ordinary fidelity rejects;
- explicit member writes, exact original-value result identity, and no readback;
- raw-reference and attached-facade operation targets;
- non-foreign target, wrong arity, non-String member name, unsupported
  capability, unexportable outbound value, and closed-generation pre-entry
  rejection;
- fresh `ForeignError` after entered failures for all four operation families;
- D189 synchronous Closure callback reuse, exact Error round-trip, and expiry of
  a callback retained physically by a provider after `writeMember`.

The maintainer reported for the exact published work:

~~~text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
~~~

This record retains the human-reported validation and does not claim that the
documentation publisher reran product tests.

## Publication metadata defect

The substantive implementation is complete, but the published commit does not
satisfy the repository's implementation-version/changelog atomicity rule.

At exact product revision
`5decd57c4297481d838908ae9c47ba18de3b742b`:

~~~text
pom.xml = 0.3.281-SNAPSHOT
root CHANGELOG.md newest section = 0.3.280-SNAPSHOT
root CHANGELOG.md 0.3.281-SNAPSHOT section = ABSENT
~~~

`AGENTS.work/IMPLEMENTATION.md` requires every commit changing `src/` or
`protos/lib/` to contain both the one implementation-version increment and a
matching root `CHANGELOG.md` section for that exact version. It further states
that late metadata materialization does not weaken same-final-commit atomicity.

Because the commit is already published, this historical atomicity omission
cannot be made true retroactively without rewriting published history. The safe
repair is a bounded follow-up commit that adds only the missing
`0.3.281-SNAPSHOT` changelog entry and explicitly records that it documents the
already-published LIB021-A implementation.

No second version bump is justified by that metadata-only repair: the product
implementation is already identified as `0.3.281-SNAPSHOT`.

## Routing

~~~text
LIB021_A_SUBSTANTIVE_IMPLEMENTATION=COMPLETE
LIB021_ISSUE_CLOSURE=BLOCKED

NEXT_SLICE=LIB021-B
NEXT_SLICE_TYPE=IMPLEMENTATION_METADATA_REPAIR
NEXT_REPOSITORY=guillermomolina/protos
NEW_ISSUE_REQUIRED=NO

LIB021_B_SCOPE=
  add missing root CHANGELOG.md section for 0.3.281-SNAPSHOT;
  describe LIB021-A accurately;
  no src/protos/spec/test/pom delta;
  no additional implementation version bump

BEHAVIORAL_TEST_RERUN=
  NOT_REQUIRED_BY_THE_METADATA_CHANGE_ITSELF;
  reassess only if current policy or concurrent executable/dependency movement
  invalidates the already-green evidence
~~~

After LIB021-B is published and the live `pom.xml`/root `CHANGELOG.md`
metadata agree on `0.3.281-SNAPSHOT`, LIB021 can close without another runtime
slice.

## Project-record publication

This record was prepared from project-record base:

~~~text
PROJECT_RECORD_BASE_REVISION=77319b1d61a30ce0e841bbf579189123fe773bc5
~~~

## AI-assistance disclosure

This durable evidence record was materially prepared with AI assistance from
ChatGPT using the exact published LIB021-A commit, current normative/spec state,
live Issue coordination, repository implementation-finalization rules, and the
maintainer-reported validation result. No independent human review is claimed.
