# DOC008 — Source documentation coverage and hygiene

## Record purpose

This is the durable, non-normative work record for DOC008 / guillermomolina/protos#560.

Operational status, scheduling, and closure remain owned by the GitHub Issue in
`guillermomolina/protos`. Normative Protos semantics remain owned by `spec/`
in that repository.

## DOC008-A — mechanical inventory and triage

### Status

```text
DOC008_A_STATUS=COMPLETE
DOC008_A_TYPE=INVESTIGATION
DOC008_A_VERDICT=READY_FOR_IMPLEMENTATION
PROTOS_HEAD=fc9aca90f479051964d8d56c11a2778386789671
TOOL007_BASELINE=6c2ffc8412889bb9d26d08714f2ce56307239d95
TOOL007_IMPLEMENTATION_VERSION=0.3.34-SNAPSHOT
```

The inventory was first assembled against
`0d478c5bf4aabaaac781b80cc7f5889c8bbc9160`. During the investigation,
`main` advanced by the direct child
`fc9aca90f479051964d8d56c11a2778386789671`
(`DOC005-F: document Test Tool exact file-backed focal selection`).
That commit changes only `docs/guide/tools/test-tool.md`; it does not change
`protos/lib/**`, `protos/tools/**`, or the D138/D067 documentation
machinery. The DOC008-A inventory was therefore reconciled to the latter exact
revision without changing its source-coverage findings.

### TOOL007/D138 baseline

The relevant documentation machinery is byte-for-byte unchanged between the
TOOL007 closure revision and the DOC008-A product revision:

```text
ProtosSourceDocumentation.java                         6207e135e2825dcc56193e1a3b6aee35d4c2fef1
ProtosStandardLibraryDocumentationExtractor.java       76b2fd44afd0ca0b47e458cab73c000f5094ceb3
ProtosDocumentationModel.java                          7033280678eca5809dfcb3ab998556c7fe810e8c
ProtosDocumentationJson.java                           50648a49274182d325104aa76afe61ec51899add
ProtosSourceDocumentationTest.java                     0fde11e650a3b02060fdaa393a334f785ddb6eb7
ProtosStandardLibraryDocumentationExtractorTest.java   0c61b352f4a4e50533d311af25eaa0898b661189
```

DOC008 therefore continues to use TOOL007's final D138 source-owner and
association semantics directly. No parallel comment scanner or ownership model
is needed.

### Effective production-source boundary

The exact repository tree contained 680 `.protos` files:

```text
protos/lib/**                         52   production library/runtime
protos/tools/**                       47   production bundled tools/applications
protos/tests/**                      486   test source, out of coverage scope
protos/examples/**                    28   examples, out of coverage scope
protos/tutorials/**                   26   tutorials, out of coverage scope
protos/benchmarks/**                  23   benchmarks, out of coverage scope
src/test/resources/**/*.protos        18   fixtures/test resources, out of scope

PRODUCTION_SOURCE_FILES=99
NON_PRODUCTION_SOURCE_FILES=581
```

Within `protos/lib/**`:

```text
protos/lib/core/**=29
protos/lib/** non-core=23
```

The 29 Core files remain production D138 source units but are not importable
D067 `std:` modules. The 23 non-Core files are the exact current D067 module
set.

### Current D067 coverage

The static HEAD inventory established:

```text
MODULE_SOURCE_OWNERS=99
MODULES_DOCUMENTED=11
MODULES_UNDOCUMENTED=88

NAMED_SLOT_OWNERS=NOT_STATICALLY_DETERMINED_WITH_REQUIRED_D138_FIDELITY
NAMED_SLOT_OWNERS_DOCUMENTED=28
NAMED_SLOT_OWNERS_UNDOCUMENTED=NOT_STATICALLY_DETERMINED_WITH_REQUIRED_D138_FIDELITY

D067_MODULES=23
D067_MODULES_DOCUMENTED=5
D067_MODULES_UNDOCUMENTED=18

D067_TOP_LEVEL_SYMBOLS=122
D067_DOCUMENTED_SYMBOLS=21
D067_UNDOCUMENTED_SYMBOLS=101

MECHANICAL_HYGIENE_FINDINGS=0
STALE_OR_CONTRADICTORY_SAFE_FINDINGS=0
SEMANTIC_BLOCKERS=2
```

The total D138 named-slot-owner count was deliberately not guessed from text.
Correct nested/member-target/grouped ownership requires the existing parser-based
`ProtosSourceDocumentation` authority.

The 28 existing productive `///` associations and the 11 productive `//!`
module blocks inspected in DOC008-A are mechanically valid under TOOL007. No
misplaced, duplicated, inline, cross-scope, blank-line-separated, or
wrong-owner documentation block was found.

### Exact D067 gap inventory

Documented modules at the DOC008-A baseline:

```text
std:cli/CommandLine
std:collections/Range
std:math/Integer
std:test/Test
std:uri
```

The remaining D067 modules and their top-level symbol gaps were:

```text
std:collections/Array
  module
  map filter findIndex reduce sort
  parallelMap parallelFilter parallelFindIndex parallelReduce parallelSort
  (the five parallel* symbols were already documented)

std:collections/IdentitySet
  module
  call contains size add remove each union intersection difference
  sameMembers isSubset isSuperset isDisjoint

std:collections/Set
  module
  call contains size add remove each union intersection difference
  sameMembers isSubset isSuperset isDisjoint

std:crypto/SHA256
  module
  digest

std:csv/CSV
  module
  rowParser parse encode readRows writeRows

std:io/BufferedReader
  module
  no source-created D067 slot

std:io/BufferedWriter
  module
  no source-created D067 slot

std:io/Files
  module
  readAllBytes writeAllBytes readAllText writeAllText

std:io/ProcessStreams
  module
  stdinReader stdoutWriter stderrWriter

std:json/JSON
  module
  nullValue boolean string number array object parse encode
  eventParser eventWriter readEvents writeEvents

std:network/IpAddresses
  module
  v4 v6 parse format

std:network/IpEndpoints
  module
  parse format

std:test/Assertions
  module
  AssertionFailure require signals

std:text/Latin1
  module
  encode decode reader owningReader writer owningWriter

std:text/UTF16BE
  module
  encode decode reader owningReader writer owningWriter

std:text/UTF16LE
  module
  encode decode reader owningReader writer owningWriter

std:text/UTF8
  module
  encode decode reader owningReader writer owningWriter

std:toml/TOML
  module
  string integer float boolean localDate localTime localDateTime offsetDateTime
  array table encode parse

std:uri
  already fully documented
```

The totals reconcile exactly to 23 modules and 122 mechanically observable
top-level D067 symbols.

### Safe corrections and routed blockers

DOC008-A classified the 18 missing D067 module descriptions as safe to add from
already-authoritative local semantics.

Of the 101 missing D067 symbol descriptions, 99 are safe for DOC008 to author
from existing normative specifications, ratified decisions, tests, and current
implementation contracts without choosing new semantics.

Two symbol gaps are explicitly not owned by DOC008:

```text
std:toml/TOML::array
std:toml/TOML::table
```

AUD005 / #451 records the unresolved composite-constructor validation question
for these two constructors. Observable-behavior changes remain owned by
LIB010 / #418 and, if necessary, a separate Dxxx decision. DOC008 must not
silently select deep-constructor validation semantics through documentation.

The rest of `protos/lib/core/**` and undocumented Tool-internal source owners
are retained in the production inventory but are not converted into artificial
coverage failures merely because D138 makes them documentable. D067's
documented/undocumented publication obligation is specific to the importable
Standard Library surface.

`std:io/BufferedReader` and `std:io/BufferedWriter` are also an important
D138/D067 distinction: their current source files have module-source owners but
no source-created top-level slot owner for the runtime-installed factory.
DOC008 may document the module source unit with `//!`; it must not invent a
`///` owner that does not exist.

### DOC008-A validation provenance

DOC008-A itself made no product-repository changes.

The maintainer subsequently reported:

```text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

These results are retained as repository-state validation evidence, not as a
claim that DOC008-A introduced executable changes.

## Next implementation boundary

The next slice is one complete implementation publication, not a family of
sub-issues:

```text
SLICE_ID=DOC008-B
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
GOAL=complete all safe DOC008 D067 source-documentation corrections
EXPECTED_FINAL_D067_MODULE_COVERAGE=23/23
EXPECTED_FINAL_D067_SYMBOL_COVERAGE=120/122
REMAINING_ROUTED_SYMBOL_GAPS=std:toml/TOML::array,std:toml/TOML::table
```

DOC008-B must preserve all existing valid documentation, add the 18 missing
module `//!` blocks and the 99 safe missing top-level `///` blocks, leave the
two TOML blockers undocumented and explicitly routed, and make no executable or
semantic changes.

After DOC008-B is published and validated, DOC008 should need only closure
reconciliation/evidence, not another implementation subdivision, unless the
current HEAD exposes a new falsifying finding.
