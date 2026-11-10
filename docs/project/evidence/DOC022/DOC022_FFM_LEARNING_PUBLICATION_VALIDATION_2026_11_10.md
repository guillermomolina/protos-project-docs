# DOC022 — Revision-bound FFM tutorials and examples publication evidence

**Date:** 2026-11-10  
**Issue:** [DOC022 #1112](https://github.com/guillermomolina/protos/issues/1112)  
**PROTOS_REVISION:** `053e55ffb0cdee56d71ec38876410e6740d80783`  
**Product commit:** https://github.com/guillermomolina/protos/commit/053e55ffb0cdee56d71ec38876410e6740d80783  
**Prior I144 implementation revision:** `a7d9666aeb29a18905232907b83450fc82d034ec`  
**Maintained work record:** [DOC022 completion](../../work/DOC022/DOC022_FFM_PROVIDER_LEARNING_COMPLETION_2026_11_10.md)

## Independently observed GitHub publication

The product head was fetched from the public GitHub repository. The exact
commit `053e55ffb0cdee56d71ec38876410e6740d80783` has subject
`DOC022: document external FFM provider with runnable SQLite examples`,
parent `a7d9666aeb29a18905232907b83450fc82d034ec`, **12 changed paths, 986 insertions and 6 deletions**.

Published path inventory:

```text
dist/test_external_foreign_providers.py
docs/guide/README.md
docs/guide/interop/README.md
docs/guide/interop/ffm-provider.md
providers/ffm/README.md
protos/examples/README.md
protos/examples/ffi/README.md
protos/examples/ffi/sqlite-version.protos
protos/examples/ffi/sqlite-memory-query.protos
protos/tutorials/README.md
protos/tutorials/08-language-interactions/07-ffi-provider-basics.protos
protos/tutorials/12-system-resources/04-ffi-ownership-and-borrows.protos
```

No product-runtime Java source, normative specification or Maven version
file is in the published diff. This records *learning-material* publication,
not a new implementation or a GitHub Release.

## Human-executed validation, faithfully scoped

The maintainer provided the following outputs from the local checkout:

| Gate | Result | Source of evidence |
| --- | --- | --- |
| `python3 -m py_compile dist/test_external_foreign_providers.py` | PASS (no error) | maintainer console output |
| Initial `make dist` on dirty tree | EXPECTED REFUSAL (requires `--allow-dirty`) | maintainer console output |
| Initial external-provider harness on older `0.3.459-SNAPSHOT` ZIP | 22/22 PASS | maintainer console output, superseded by exact-version run |
| `python3 dist/build_portable.py --allow-dirty` | `BUILD SUCCESS`, `DIST_BUILD: PASS`, classfile major 69, archive CRC/layout/identity checks PASS | maintainer console output |
| Exact-version archive | `target/distributions/protos-0.3.460-SNAPSHOT-posix-jvm.zip` | maintainer console output |
| External-provider harness with explicit exact-version `--archive` | **22/22 PASS**, no unsupported case reported | maintainer console output |
| `I144_TUTORIAL_BASIC` | PASS | exact-version harness output |
| `I144_TUTORIAL_LIFETIME` | PASS | exact-version harness output |
| `I144_EXAMPLE_VERSION` | PASS | exact-version harness output |
| `I144_EXAMPLE_QUERY` | PASS | exact-version harness output |
| `git diff --check` | CLEAN | maintainer final report |
| All local tests (including full `make test`/`make check` as applicable) | PASS, no final per-suite counts supplied | maintainer final report |

The external-provider harness also reported `PROTOS_CLASSFILE_BASELINE=PASS`,
`I144_FFM_PROVIDER_JAR=PASS`, `I144_FFM_SQLITE=PASS`,
`I144_FFM_DENIED=PASS`, `I144_NO_IMPLICIT_AUTHORIZATION=PASS`,
`I144_UNUSED_NO_SESSION=PASS`, `I144_ARBITRARY_ALIAS_DENIED=PASS`,
`DISTRIBUTION_UNMODIFIED=PASS` and the original Java/opaque/provider
acceptance gates PASS.

The exact final full-Java-suite test counts, an independent CI run,
release artifact publication and a Native Image FFM run **were not supplied
and are not claimed**. These local test results are owner reports;
independent verification is of the GitHub commit and its file delta.

## Acceptance meaning

Four complete teaching programs execute real C symbols through
`import("ffi:sqlite3")` using only authorized external provider loading.
They deliberately avoid the ordinary no-provider example sweep.
The documentation is a user-facing explanation of existing I144,
not normative authority for new FFI semantics.

```text
FORMAL_IDENTIFIER=DOC022
PROTOS_REVISION=053e55ffb0cdee56d71ec38876410e6740d80783
PRODUCT_COMMIT=PASS_GITHUB_VERIFIED
PRODUCT_FILES_CHANGED=12
DISTRIBUTION_VERSION=0.3.460-SNAPSHOT
DIST_BUILD=PASS_OWNER_REPORTED
FFM_HARNESS=22_OF_22_PASS_OWNER_REPORTED
FFM_NEW_LEARNING_CASES=4_OF_4_PASS_OWNER_REPORTED
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
MAKE_TEST=PASS_OWNER_REPORTED
MAKE_CHECK=PASS_OWNER_REPORTED
CI_INDEPENDENT_RUN=NOT_VERIFIED
NATIVE_IMAGE_FFM=OUT_OF_SCOPE
```
