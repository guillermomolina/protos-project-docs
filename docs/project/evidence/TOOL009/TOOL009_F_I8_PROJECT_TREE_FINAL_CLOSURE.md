# TOOL009-F-I8 — Package project-tree consolidation and final closure

## Identity

- Parent work item: `TOOL009-F / guillermomolina/protos#694`
- Slice: `TOOL009-F-I8`
- Product repository: `guillermomolina/protos`
- Published product revision:
  `016233f90580407ecfd9dadfa19b5f4d5f233369`
- Commit:
  `TOOL009-F-I8: consolidate Package project-tree sources and close migration`

The project owner reported the published F-I8 candidate as pushed and tested.

## F-I8 scope

F-I8 closes the Package Tool project-tree consolidation:

```text
content-identity:       12 -> 6 physical sources / 12 Logical Cases
resolution-input-lock:  2 -> 2 physical sources / 2 Logical Cases
resolution-root:        8 -> 8 physical sources / 8 Logical Cases
execution-plan:        14 -> 11 physical sources / 28 Logical Cases
project-projection:     4 -> 3 physical sources / 4 Logical Cases

TOTAL:                 40 -> 30 physical sources
                       54 -> 54 Logical Cases
```

Final physical bookkeeping:

```text
CREATED_SOURCES=9
REUSED_SOURCES=21
REMOVED_SOURCES=19
```

## Published project-tree manifests

At the product revision:

```text
content-identity/manifest.tsv       = 6 rows
resolution-input-lock/manifest.tsv  = 2 rows
resolution-root/manifest.tsv        = 8 rows
execution-plan/manifest.tsv         = 25 rows
project-projection/manifest.tsv     = 3 rows

TOTAL_PROJECT_TREE_MANIFEST_ROWS=44
```

The 44 project-tree manifest rows expand to 54 Logical Cases because selected
Test selectors are crossed only with their retained project authority.

## Authority x selector reconciliation

The published source shapes are:

```text
content-identity:
  6 authorities x 2 selectors each = 12 Cases

resolution-input-lock:
  2 authorities x 1 selector each = 2 Cases

resolution-root:
  8 authorities x 1 selector each = 8 Cases

execution-plan:
  8 retained one-selector authorities                = 8 Cases
  registry-leaf-v2 x 2 selectors                    = 2 Cases
  f2e3b-transitive x 3 selectors                    = 3 Cases
  f2e3b-build-v2-error x 15 authorities x 1 selector = 15 Cases
                                                        --
                                                        28 Cases

project-projection:
  root-only x 2 selectors = 2 Cases
  member-bytes x 1        = 1 Case
  stale x 1               = 1 Case
                            --
                             4 Cases
```

Therefore:

```text
AUTHORITY_SELECTOR_MATRIX_AFTER=54
MISSING_MATRIX_ENTRIES=0
EXTRA_MATRIX_ENTRIES=0
PROJECT_TREE_MATRIX_RECONCILIATION=PASS
```

The critical 15-authority `f2e3b-build-v2-error.protos` source remains a
single-selector source referenced by 15 distinct project authorities. It was
not collapsed.

## Global TOOL009-F physical reconciliation

The current published corpus composition is:

```text
ordinary conformance manifest sources             171
repository-explicit library sources                23
Process + Actor + Group sources                    10
Package TOML/version/lock/resolution-input         36
Package project-tree sources                       30
                                                   ---
FINAL_SUITE_NATIVE_PHYSICAL_SOURCES=270
```

The repository-explicit library physical count at the final product revision is:

```text
URI=3
CSV=5
CLI=5
Math/Integer=4
SHA-256=2
IpAddresses=3
IpEndpoints=1
TOTAL=23
```

The logical-case total established and preserved across the eight implementation
slices is:

```text
ordinary conformance                               836
repository-explicit libraries                       78
Process + Actor + Group                             36
Package non-project-tree                           259
Package project-tree                                54
                                                  ----
FINAL_LOGICAL_CASES=1263
```

Thus the implementation reaches the Phase E/F target:

```text
INITIAL_SUITE_NATIVE_PHYSICAL_SOURCES=1234
FINAL_SUITE_NATIVE_PHYSICAL_SOURCES=270

INITIAL_LOGICAL_CASES=1263
FINAL_LOGICAL_CASES=1263

LOGICAL_CASE_COVERAGE_CHANGE=NONE
```

## Validation

The project owner reported the final F-I8 candidate as tested after publication:

```text
USER_REPORTED_FINAL_VALIDATION=PASS
```

This is the completion signal for the final F-I8 validation gate. This durable
record does not invent individual command output that was not supplied in the
handoff.

## Eight-slice completion

```text
TOOL009-F-I1=COMPLETE
TOOL009-F-I2=COMPLETE
TOOL009-F-I3=COMPLETE
TOOL009-F-I4=COMPLETE
TOOL009-F-I5=COMPLETE
TOOL009-F-I6=COMPLETE
TOOL009-F-I7=COMPLETE
TOOL009-F-I8=COMPLETE

IMPLEMENTATION_SLICES_COMPLETE=8/8
```

Notable published revisions:

```text
F-I1  343e74eca5870d6d119612be79d968a9a8cce4c9
F-I2  44690b1fc8c9aed023600c6d5731f969c4507e27
       reconciled by 9a7ea87ad0191973f59ee453c6ec67eca7cfc520
F-I3  44382cba6d691a1eae8bf74be6ed3e15eda6f0cd
F-I4  8431be72e1fe5f0c6677eb768e07716bd85d894b
F-I5  f586752d8009a7a27db59a41e5d868d045c29fb3
F-I6  f6daf53bfd803021954681c9ed8307d523711421
F-I7  9ada962e63e7198b8ae7b468b4c8f9d471bb6cff
F-I8  016233f90580407ecfd9dadfa19b5f4d5f233369
```

F-I6's final bookkeeping was corrected during implementation to 12 created,
2 reused and 100 removed because two target paths already existed in the
baseline. This did not change its 102 -> 14 physical-source or 102 -> 102 Case
result.

## Result

```text
TOOL009_F_I8=COMPLETE
TOOL009_F=COMPLETE

PROTOS_REVISION=016233f90580407ecfd9dadfa19b5f4d5f233369

PROJECT_TREE_SOURCES_BEFORE=40
PROJECT_TREE_SOURCES_AFTER=30
PROJECT_TREE_LOGICAL_CASES_BEFORE=54
PROJECT_TREE_LOGICAL_CASES_AFTER=54

FINAL_SUITE_NATIVE_PHYSICAL_SOURCES=270
FINAL_LOGICAL_CASES=1263

PROJECT_TREE_MATRIX_RECONCILIATION=PASS
TOOL009_F_GLOBAL_RECONCILIATION=PASS
USER_REPORTED_FINAL_VALIDATION=PASS

LOGICAL_CASE_COVERAGE_CHANGE=NONE
TEST_TOOL_SEMANTICS_CHANGE=NONE
PACKAGE_TOOL_SEMANTICS_CHANGE=NONE
SPEC_CHANGE=NONE

PARENT_ISSUE_READY_TO_CLOSE=YES
```
