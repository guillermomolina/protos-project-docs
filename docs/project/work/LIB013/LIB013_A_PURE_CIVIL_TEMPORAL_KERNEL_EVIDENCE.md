# LIB013-A — Pure civil temporal kernel implementation evidence

Status: **COMPLETE**

Owning work item: `guillermomolina/protos#430`

Slice: `LIB013-A — Pure civil temporal kernel`

Nature: implementation/validation evidence; **non-normative**

Product revision: `b0bf563c30a612210cd67c6a0bbea01f44617c3c`

Product version: `0.3.242-SNAPSHOT`

Commit subject:

~~~text
LIB013-A: add pure civil temporal kernel
~~~

Ratified design owner: LIB013-0 Candidate A — Pure Values First + Explicit Authorities.

## Publication result

LIB013-A is published in `guillermomolina/protos` at the exact product revision
above.

The published delta is exactly one commit ahead of the LIB013-0 implementation
baseline `e1904bb01b61c6ee0da5fd279da0f7feb143229e`.

The implementation adds the initial pure civil temporal kernel without adding
clock, timezone, parsing/formatting, Duration/Period, Offset/Instant, or runtime
authority.

## Implemented Standard Library surface

New public modules:

~~~text
std:datetime/Date
std:datetime/Time
std:datetime/LocalDateTime
~~~

### Date

`Date(year, month, day)` implements:

- proleptic Gregorian calendar semantics;
- astronomical year numbering including year `0`;
- supported year range `-9999 .. 9999`;
- exact month/day validation;
- proleptic Gregorian leap-year rules including negative years;
- explicit `Error` on invalid/out-of-range arguments;
- frozen ordinary-object values;
- semantic `==` implemented through the established `equals` alias pattern;
- coherent `hash` suitable for normal Map keys;
- ordinary object identity under `===`;
- `Date.compare(left, right)` returning `-1`, `0`, or `1` in chronological order.

### Time

`Time(hour, minute, second, nanosecond)` implements:

- hour range `0..23`;
- minute range `0..59`;
- second range `0..59`;
- nanosecond range `0..999999999`;
- explicit rejection of leap-second label `second == 60`;
- explicit `Error` on invalid arguments;
- frozen ordinary-object values;
- semantic `==`, coherent `hash`, normal Map-key behavior and ordinary `===`;
- `Time.compare(left, right)` in local time-of-day order.

### LocalDateTime

`LocalDateTime(date, time)`:

- composes one validated Date and one validated Time exactly;
- preserves the exact component objects;
- carries no offset, zone or clock state;
- is frozen;
- defines structural semantic equality and coherent hashing;
- retains ordinary object identity;
- exposes `LocalDateTime.compare(left, right)`, ordering by Date then Time.

## Validation evidence retained in product tests

The new suite-native `library/datetime` corpus contains:

- `construction.protos`;
- `equality.protos`;
- `ordering.protos`.

The tests cover, among other cases:

- minimum/maximum supported years;
- year zero and negative years;
- Gregorian leap-year and century rules;
- invalid months/days and unsupported years;
- time field extrema and invalid values;
- leap-second-label rejection;
- exact LocalDateTime composition;
- absence of offset/zone state;
- semantic equality and ordinary identity distinction;
- coherent hashes and Map-key behavior;
- frozen value behavior;
- Date, Time and LocalDateTime comparison laws;
- foreign-argument rejection for comparison.

The commit also registers the new corpus through the current Test Tool/CLI
repository corpus wiring.

## Transferability / current runtime boundary

LIB013-A intentionally did **not** add a datetime-specific runtime value kind or
special transfer rule.

The published values are frozen ordinary Protos objects, but their local
`equals` / `hash` Closures mean they remain non-transferable across Actors
under the current general pass-by-value rules.

This is retained as a current implementation/runtime boundary, not silently
"fixed" by adding a special-case datetime authority or identity mechanism.

~~~text
SPECIAL_DATETIME_RUNTIME_VALUE=NO
DATETIME_TRANSFER_EXCEPTION=NO
CURRENT_ACTOR_TRANSFERABILITY=NON_TRANSFERABLE_DUE_TO_LOCAL_CLOSURES
~~~

This limitation does not invalidate the pure-value semantics implemented by
LIB013-A and does not block LIB013-B.

## Explicitly not implemented

~~~text
Duration=NO
Period=NO
Offset=NO
Instant=NO
OffsetDateTime=NO
ZoneId=NO
ZoneDatabase=NO
ZonedDateTime=NO
WallClock=NO
MonotonicClock=NO
Parsing=NO
Formatting=NO
LocaleFormatting=NO
TimersOrScheduling=NO
TOMLIntegration=NO
~~~

## Exact product changed-path set

The exact `e1904bb0..b0bf563c` delta contains:

~~~text
CHANGELOG.md
pom.xml
protos/lib/datetime/Date.protos
protos/lib/datetime/LocalDateTime.protos
protos/lib/datetime/Time.protos
protos/tests/library/datetime/construction.protos
protos/tests/library/datetime/equality.protos
protos/tests/library/datetime/ordering.protos
protos/tools/test/RepositoryCorpusPlans.protos
protos/tools/test/RepositorySuite.protos
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/main/java/com/guillermomolina/protos/cli/ProtosTestCorpusRegistry.java
src/test/java/com/guillermomolina/protos/cli/ProtosTestToolCorpusRegistryTest.java
src/test/java/com/guillermomolina/protos/cli/ProtosTestToolFileSelectionWiringTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolRepositoryCorpusPlansTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSuiteGraphTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolTool011ProgressGroupingTest.java
~~~

## Maintainer-reported validation

After publication, the maintainer reported:

~~~text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

No validation result is inferred from source inspection.

## Specification and license result

The product CHANGELOG records:

~~~text
SPECIFICATION_CHANGE=NO
~~~

All newly added Protos-owned `.protos` source/test files in the exact commit
carry the project's current Adaptive Public License Part 5 notice.

No dependency, host-time, timezone-database, scheduler or locale authority was
introduced.

## Next slice

The ratified dependency order continues with:

~~~text
NEXT_SLICE=LIB013-B
NEXT_SLICE_NAME=Temporal amounts and calendar arithmetic
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
~~~

LIB013-B owns `Duration`, `Period`, signed temporal-amount arithmetic,
overflow/range failure, and the already-ratified calendar clamp semantics.

LIB013-C and LIB013-D remain later slices.

## Closure state

~~~text
LIB013_A_STATUS=COMPLETE
PRODUCT_REVISION=b0bf563c30a612210cd67c6a0bbea01f44617c3c
PRODUCT_VERSION=0.3.242-SNAPSHOT
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
SPECIFICATION_CHANGED=NO
NEW_UNAPPROVED_SEMANTICS=NO

DATE_IMPLEMENTED=YES
TIME_IMPLEMENTED=YES
LOCALDATETIME_IMPLEMENTED=YES

TZDB_ADDED=NO
CLOCK_AUTHORITY_ADDED=NO
PARSING_FORMATTING_ADDED=NO
DURATION_PERIOD_ADDED=NO
INSTANT_OFFSET_ADDED=NO

PARENT_ISSUE_CLOSED=NO
NEXT_SLICE=LIB013-B
~~~

## AI-assistance disclosure

This durable evidence record was materially prepared with AI assistance from
ChatGPT using the exact published product commit, the ratified LIB013-0 record,
current repository policy, and maintainer-reported validation evidence. No
independent human review is claimed by this record.
