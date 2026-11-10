# DOC022 — Protos-first FFM provider learning-material completion

**Date:** 2026-11-10  
**Formal work:** [DOC022 #1112](https://github.com/guillermomolina/protos/issues/1112)  
**Published implementation prerequisite:** [I144 #1050](https://github.com/guillermomolina/protos/issues/1050), product [`a7d9666aeb29a18905232907b83450fc82d034ec`](https://github.com/guillermomolina/protos/commit/a7d9666aeb29a18905232907b83450fc82d034ec)  
**PROTOS_REVISION:** `053e55ffb0cdee56d71ec38876410e6740d80783`  
**Product commit:** https://github.com/guillermomolina/protos/commit/053e55ffb0cdee56d71ec38876410e6740d80783  
**Product development version:** `0.3.460-SNAPSHOT` (not a tagged release)  
**Validation record:** [DOC022 publication/acceptance evidence](../../evidence/DOC022/DOC022_FFM_LEARNING_PUBLICATION_VALIDATION_2026_11_10.md)

## Separate documentation ownership

DOC022 is the independently completed educational follow-up to the I144
optional I085/Java 25 FFM provider. Its scope is maintained guides,
runnable tutorials and cookbook programs, plus execution of those materials
in the existing external-provider harness. It neither defines FFI semantics
nor adds native provider machinery, public `std:ffi` or a higher-level SQLite
library. The approved architecture is [D238-B1-SQL-R1](../../decisions/language/D238_PROTOS_FIRST_NATIVE_FFI_B1_SQL_R1_RATIFIED.md).

## Published teaching deliverables

- [FFM provider guide](https://github.com/guillermomolina/protos/blob/053e55ffb0cdee56d71ec38876410e6740d80783/docs/guide/interop/ffm-provider.md):
  end-to-end build/authorization HOWTO, descriptors, Protos-first C signatures,
  status/out results, resource custody, bounded borrowed memory,
  `preserving` declarations, ordinary pre-entry `Error` vs post-entry
  `ForeignError`, platform limits, trust boundary, and the explicit
  `pzTail=null` SQLite lifetime-safe pattern.
- [Provider README](https://github.com/guillermomolina/protos/blob/053e55ffb0cdee56d71ec38876410e6740d80783/providers/ffm/README.md):
  separate build and authorization steps, exact CLI quickstart and links into
  lessons and examples.
- [Basic FFI tutorial](https://github.com/guillermomolina/protos/blob/053e55ffb0cdee56d71ec38876410e6740d80783/protos/tutorials/08-language-interactions/07-ffi-provider-basics.protos):
  `sqlite3_libversion_number`, `sqlite3_complete`,
  `sqlite3_strnicmp`, scalar C returns and rejection of wrong arity,
  type, range and interior NUL before entry.
- [Ownership and borrowing tutorial](https://github.com/guillermomolina/protos/blob/053e55ffb0cdee56d71ec38876410e6740d80783/protos/tutorials/12-system-resources/04-ffi-ownership-and-borrows.protos):
  owned connection/statement, `out` result, prepare with `pzTail=null`,
  borrowed column text, preserving byte count, bounded copy,
  epoch invalidation and reverse release.
- [SQLite version smoke example](https://github.com/guillermomolina/protos/blob/053e55ffb0cdee56d71ec38876410e6740d80783/protos/examples/ffi/sqlite-version.protos):
  one C symbol, zero arguments, explicit provider path, no output on success.
- [In-memory SQLite query example](https://github.com/guillermomolina/protos/blob/053e55ffb0cdee56d71ec38876410e6740d80783/protos/examples/ffi/sqlite-memory-query.protos):
  declare the native C signatures in Protos, run `SELECT`, read i64/f64,
  UTF-8 `café` and an embedded-NUL BLOB, rebuild ordinary Bytes in Protos,
  finalize the statement then close the connection.
- [FFM cookbook README](https://github.com/guillermomolina/protos/blob/053e55ffb0cdee56d71ec38876410e6740d80783/protos/examples/ffi/README.md):
  prerequisites, archive/provider build, invocation, safety and scope.
- [External-provider harness](https://github.com/guillermomolina/protos/blob/053e55ffb0cdee56d71ec38876410e6740d80783/dist/test_external_foreign_providers.py):
  four distinct learning-material cases validating a silent, assertion-driven
  success under the explicitly authorized JAR and actual shared SQLite.
- Guide, interop, examples and tutorials navigation indices link these paths.

The four Protos programs deliberately contain no `print(`: the generic
standalone tutorial/example discovery runs without authorizing the external
provider. The FFM harness instead runs them from their real paths with
`--foreign-provider-path` and requires exit 0, empty standard output and
no unexpected diagnostics.

## Accepted outcome and non-goals

Human-reported local validation after rebuilding the development distribution
`protos-0.3.460-SNAPSHOT-posix-jvm.zip` with `--allow-dirty`:
`DIST_BUILD=PASS`, `DIST_CLASSFILE_MAJOR=69`, all **22/22 external-provider
cases PASS**, including `I144_TUTORIAL_BASIC`,
`I144_TUTORIAL_LIFETIME`, `I144_EXAMPLE_VERSION`, and
`I144_EXAMPLE_QUERY`. The maintainer also reports a clean
`git diff --check` and all local tests PASS. This is owner-provided
execution evidence, **not** independent reproduction by the documentation
publisher; see the [exact gate log summary](../../evidence/DOC022/DOC022_FFM_LEARNING_PUBLICATION_VALIDATION_2026_11_10.md).

The product commit was independently inspected through GitHub: it has one
parent (`protos@a7d9666aeb29a18905232907b83450fc82d034ec`), 12 changed paths (+986/−6), and no changes to
`src/main/`, `spec/`, `pom.xml`, `CHANGELOG.md`, or the FFM provider's
Java source. No version bump belongs to this documentation-only work.

The provider remains Linux x86-64/JDK 25 **portable JVM only**,
`TRUSTED_IN_PROCESS` (not a sandbox). An incorrect C ABI declaration can
crash or corrupt the host process. Future [LIB028 #1051](https://github.com/guillermomolina/protos/issues/1051)
owns a public `std:ffi` Protos API; [LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052)
owns an idiomatic SQLite library. Neither is delivered by DOC022.

```text
FORMAL_IDENTIFIER=DOC022
PROTOS_REVISION=053e55ffb0cdee56d71ec38876410e6740d80783
I144_BASE_REVISION=a7d9666aeb29a18905232907b83450fc82d034ec
PRODUCT_VERSION=0.3.460-SNAPSHOT
FILES_CHANGED=12
ADDITIONS=986
DELETIONS=6
PROTOS_TUTORIALS_ADDED=2
PROTOS_EXAMPLES_ADDED=2
FFM_HARNESS_NEW_CASES=4
DIST_BUILD=PASS_OWNER_REPORTED
FFM_HARNESS=22_OF_22_PASS_OWNER_REPORTED
MAKE_TEST=PASS_OWNER_REPORTED
MAKE_CHECK=PASS_OWNER_REPORTED
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
PRODUCT_PUBLICATION=PASS_GITHUB_VERIFIED
RUNTIME_SPEC_VERSION_CHANGE=NO
```
