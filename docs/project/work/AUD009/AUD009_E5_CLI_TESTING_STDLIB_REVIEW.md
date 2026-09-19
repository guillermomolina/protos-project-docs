# AUD009-E5 — Standard Library CLI and testing support complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#651`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline:
`1b4065f0f79a5c2837b4462f00b3b8e481d11b98`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#651`, issue comment
`5739957804`.

Owner approval provenance: `guillermomolina/protos#651`, issue comment
`5739959948`, 2026-09-19.

Derived implementation routes: **NONE**

## Scope

E5 reviewed the remaining optional Standard Library domains:

```text
std:cli/CommandLine
std:test/Assertions
std:test/Test
```

`protos/lib/core/**` is not optional Standard Library breadth. It is
distributable Core implementation written in Protos and remains owned by the
Core audit partitions.

## Final classification

```text
STDLIB_CLI_COMMAND_LINE=KEEP
COMMAND_LINE_OPTION=KEEP
COMMAND_LINE_POSITIONAL=KEEP
COMMAND_LINE_COMMAND=KEEP
COMMAND_LINE_PARSE=KEEP
COMMAND_LINE_RENDER_HELP=KEEP

STDLIB_TEST_ASSERTIONS=KEEP
ASSERTION_FAILURE=KEEP
ASSERTIONS_REQUIRE=KEEP
ASSERTIONS_SIGNALS=KEEP

STDLIB_TEST_TEST=KEEP
TEST_FACTORY=KEEP
MODULE_AS_SUITE_MODEL=KEEP
```

No E5 mechanism is classified `REMOVE_NOW_RECONSIDER_LATER` or
`REMOVE_PERMANENTLY`.

## std:cli/CommandLine

The current public surface remains:

```text
option(descriptor)
positional(descriptor)
command(descriptor)
parse(specification, arguments)
renderHelp(rootSpec, commandPath)
```

LIB011 deliberately selected an explicit-data architecture:

```text
explicit arguments
+ inspectable frozen command specification
+ lossless inert parse result
+ pure deterministic help interpretation
```

The Test Tool currently consumes `std:cli/CommandLine` through
`protos/tools/test/Options.protos`, demonstrating real reuse beyond a
hypothetical future domain.

Keeping the module avoids returning to per-Tool parser duplication and preserves
a coherent future growth boundary without moving optional CLI policy into Core or
a host parser.

AUD006 / #453 remains the independent owner of known post-closure
scalability/hardening work. Its corrective findings do not invalidate the
LIB011 architecture or public API.

Classification: **KEEP**.

## std:test/Assertions

The current public surface remains:

```text
Assertions.AssertionFailure
Assertions.require(condition)
Assertions.signals(errorPrototype, body)
```

LIB016 selected a minimal runner-independent assertion library after repeated
repository-local assertion helpers demonstrated real duplication pressure.

The library remains:

- ordinary Protos;
- authority-free;
- independent of TOOL002;
- based on ordinary Boolean/Error/control semantics;
- free of registration, scheduling and global state.

Current repository policy directs new Protos tests toward these canonical
helpers rather than recreating equivalent local mechanisms.

Classification: **KEEP**.

## std:test/Test

The current public surface remains:

```text
Test(name, body)
```

The module creates one fresh frozen callable named test descriptor.

LIB018 selected the module-as-suite model:

```text
source module = initial grouping/suite owner
Test value    = one named logical case
Array         = ordering/composition
Assertions    = assertion support
TOOL002       = discovery/planning/execution/scheduling/reporting owner
```

This supports the ratified requirement that one source may contain multiple
logical named tests that remain independently schedulable, without introducing a
separate public Suite institution.

Classification: **KEEP**.

## Deliberate absences remain absent

```text
CLI typed decoder/default projection          ABSENT / RETAIN ABSENCE
CLI env/config implicit sources               ABSENT / RETAIN ABSENCE
CLI callbacks/command execution framework     ABSENT / RETAIN ABSENCE
CLI terminal-width probing                    ABSENT / RETAIN ABSENCE
CLI shell completion execution                ABSENT / RETAIN ABSENCE
CLI global parser registry                    ABSENT / RETAIN ABSENCE

public Test Suite value                       ABSENT / RETAIN ABSENCE
nested suites                                 ABSENT / RETAIN ABSENCE
fixture lifecycle hooks                       ABSENT / RETAIN ABSENCE
global test registry                          ABSENT / RETAIN ABSENCE
test-specific scheduler in std:test           ABSENT / RETAIN ABSENCE
property testing                              ABSENT / RETAIN ABSENCE
mocking framework                             ABSENT / RETAIN ABSENCE
snapshot framework                            ABSENT / RETAIN ABSENCE
special assertion syntax/compiler rewriting  ABSENT / RETAIN ABSENCE
```

## Audit lesson applied

E5 preserves the corrected E2/E4 audit principle:

> Current implementation thinness is not sufficient removal evidence when a
> module identity was deliberately selected as the coherent growth boundary for
> a real domain.

That principle is relevant to both `std:test/Test` and the staged
`Assertions`/`Test` testing-support boundary.

## Required routing

No E5 implementation removal/redesign route is required.

```text
REMOVAL_ROUTES=NONE
DERIVED_LIBXXX=NONE
DERIVED_DXXX=NONE
```

Independent existing owners such as AUD006 remain unchanged.

## AUD009-E partition closure

With E5 complete, the optional Standard Library partition has been fully audited:

```text
E1 collections                         COMPLETE
E2 text codecs + data formats          COMPLETE (reconciled)
E3 integer math + deterministic hash   COMPLETE
E4 I/O + numeric networking            COMPLETE
E5 CLI + testing support               COMPLETE

AUD009_E_PARTITION=COMPLETE
```

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_BASELINE=1b4065f0f79a5c2837b4462f00b3b8e481d11b98

STDLIB_CLI_COMMAND_LINE=KEEP
STDLIB_TEST_ASSERTIONS=KEEP
STDLIB_TEST_TEST=KEEP

REMOVAL_ROUTES=NONE
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_E5_CLASSIFICATION=COMPLETE
AUD009_E_PARTITION=COMPLETE
```

AUD009-E5 is complete.
