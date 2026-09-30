# I062 — Core Path relative/downward simplification implementation and closure evidence

Date: 2026-09-30

## Identity

```text
WORK_ITEM=I062/#665
DECISION=D169/#640
DECISION_STATUS=RATIFIED
SELECTED_CANDIDATE=B

PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=738e2b9f5d8101f4229542afbaf8f8689785f680
COMMIT=I062: simplify Core Path to relative components (D169)

IMPLEMENTATION_VERSION=0.3.122-SNAPSHOT
SPECIFICATION_REVISION=0.1.436

CI_RUN=36672533523
CI_RUN_NUMBER=2040
CI_RESULT=PASS
```

This record retains the published implementation, normative reconciliation and
full-CI closure evidence for I062. D169 remains the decision authority; this
record does not redefine Path or Filesystem semantics.

## Selected semantic result

I062 implements D169 Candidate B:

```text
Path                                           KEEP
Path.relative()                                KEEP
Path.child(name)                               KEEP
structural equality/hash                       KEEP
Filesystem explicit authority                  KEEP
Filesystem confinement                         KEEP

Path.rooted()                                  REMOVE
rooted/relative Path flag                      REMOVE
Path.parentComponent()                         REMOVE
Parent component kind                          REMOVE
Core file-URL -> Path semantics                REMOVE
conceptual filesystem.pathFromURL(...)         REMOVE
```

Core Path is now one immutable ordered sequence of normal component Strings.
The empty Path denotes the interpreting Filesystem base. Every component is
downward from that base; there is no rootedness dimension and no upward Parent
component.

## Runtime implementation

`ProtosPathValue` now stores only:

```text
delegation prototype
ordered List<String> components
```

The rooted flag and the `Component` / `Normal` / `Parent` representation
families were removed. Structural equality and hash depend only on the ordered
component String sequence.

Core `Path` exposes the retained native selector set:

```text
relative
child
==
hash
```

The removed selectors are absent:

```text
rooted
parentComponent
```

Actor and parallel/P transfer preserve Path values by copying the ordered
component sequence with the destination Path prototype.

## Filesystem preservation

The read-only, confined, read-only-tree and captured-tree filesystem backends
were reconciled to the new representation.

Obsolete rooted/Parent rejection branches were removed while retaining:

```text
explicit Filesystem authority
configured-base interpretation
direct-child checks
portable child-name validation
empty-Path behavior
no-follow/symlink confinement
authority-boundary failure
no ambient host filesystem fallback
```

No host-path parser, implicit `..` normalization, cwd semantics, drive/UNC
interpretation or ambient authority was introduced.

## Normative reconciliation

The implementation advances the global specification revision to `0.1.436`.

`spec/io/FILESYSTEM.md` now specifies:

- Path as one ordered sequence of normal component Strings;
- `Path.relative()` as the empty Path at the interpreting Filesystem base;
- `child(name)` as one-component extension;
- rejection of `""`, `"."` and `".."` as child names;
- no rooted Path form;
- no parent component;
- structural, filesystem-independent equality/hash over component Strings only;
- no Core file-URL -> Path conversion contract.

The former conceptual `filesystem.pathFromURL(url)` contract is removed.
Path and URL/URI data remain inert and grant no Filesystem or network
authority.

D037 remains authoritative for structural equality in principle; D169
supersedes only the removed rootedness and Parent-component dimensions.

The maintained Filesystem/authority guide, examples and conformance material
were reconciled with the simplified model.

## Published change surface

The implementation commit contains:

```text
201 additions
256 deletions
457 total changed lines
```

Representative changed paths include:

```text
CHANGELOG.md
pom.xml
spec/PROTOS_SPEC_CHANGELOG.md
spec/io/FILESYSTEM.md
spec/io/IO_CORE.md
docs/guide/11-process-io-filesystems-and-authority.md
protos/examples/paths/portable-paths.protos
protos/tests/conformance/path/equality-structure-and-hash.protos
protos/tests/conformance/path/receiver-and-factory.protos

src/main/java/com/guillermomolina/protos/runtime/ProtosPathValue.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandardPathProtocol.java
Actor/P transfer and Filesystem backend paths
Path/Filesystem architecture and regression tests
```

## Validation

The exact published revision passed the repository's full GitHub Actions CI:

```text
CI_RUN_ID=36672533523
CI_RUN_NUMBER=2040
CI_HEAD_SHA=738e2b9f5d8101f4229542afbaf8f8689785f680
CI_CONCLUSION=success

TOOLCHAIN_VERIFICATION=PASS
MAVEN_CACHE_RESTORE=PASS
RUN_REPOSITORY_TESTS=PASS

MAIN=837/837 passed
PROCESS_SNAPSHOT=15/15 passed
ACTOR=11/11 passed
GROUP=10/10 passed
PACKAGE_TOML=102/102 passed
URI=11/11 passed
CSV=17/17 passed
CLI=19/19 passed
MATH_INTEGER=11/11 passed
CRYPTO_SHA256=10/10 passed
NETWORK_IP_ADDRESSES=8/8 passed
NETWORK_IP_ENDPOINTS=2/2 passed
PACKAGE_TOOL_VERSION=74/74 passed
PACKAGE_TOOL_LOCK=71/71 passed
PACKAGE_TOOL_RESOLUTION_INPUT=12/12 passed
PACKAGE_TOOL_CONTENT_IDENTITY=12/12 passed
PACKAGE_TOOL_RESOLUTION_INPUT_LOCK=2/2 passed
PACKAGE_TOOL_RESOLUTION_ROOT=8/8 passed
PACKAGE_TOOL_EXECUTION_PLAN=28/28 passed
PACKAGE_TOOL_PROJECT_PROJECTION=4/4 passed

PROTOS_TOTAL=1264 passed, 0 failed
PROTOS_TESTS_REPORTED_TIME=216s
CI_REPOSITORY_TESTS=PASS
```

## Closure

```text
D169_PATH_ROOTED_REMOVAL=PASS
D169_PARENT_COMPONENT_REMOVAL=PASS
D169_FILE_URL_SPEC_REMOVAL=PASS
D037_STRUCTURAL_EQUALITY_PRESERVED=PASS
FILESYSTEM_AUTHORITY_PRESERVED=PASS
NORMATIVE_SPEC_RECONCILIATION=PASS
FOCUSED_VALIDATION=PASS
REQUIRED_FULL_VALIDATION=PASS
PUBLICATION_VALIDATION=PASS
CI_REQUIRED_FOR_CLOSURE=GREEN
I062_STATUS=CLOSED_COMPLETE
```
