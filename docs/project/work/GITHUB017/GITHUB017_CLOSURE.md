# GITHUB017 — CI reactivation closure

Status: **CLOSED**

Owning live Issue: `guillermomolina/protos` GitHub #492.

This is the durable closure record for GITHUB017, published retrospectively
under the general closure-evidence contract introduced by GITHUB020 / #536.

It is a non-normative project record. Live coordination remains in the owning
GitHub Issue, and executable CI behavior remains defined by the published
`guillermomolina/protos` repository state.

## Outcome

GITHUB017 restored automatic repository CI after the temporary suspension used
while the Test Tool execution path and repository-suite latency were being
reconciled.

The final GITHUB017 product publication is:

```text
PROTOS_REVISION=b616caebde59ea1aa95c3a208182c1c9cef1b8f6
COMMIT=GITHUB017: streamline automatic CI
```

At that revision:

- push to `main` runs repository CI;
- pull requests run repository CI;
- `workflow_dispatch` remains available for explicit confirmation runs;
- the canonical repository test entry point is the Makefile;
- CI executes in the repository development-container environment;
- Java/JUnit and Protos Test Tool validation remain enabled; and
- routine automatic CI no longer repeats portable-distribution validation that
  already had independent TEST001-H evidence.

The canonical CI command is:

```text
make test JAVA_TEST_JOBS=4 PROTOS_TEST_JOBS=4
```

## Final CI topology

The restored workflow uses the repository development-container definition and
the GHCR CI image/cache identity:

```text
ghcr.io/guillermomolina/protos-devcontainer:ci
```

The repository Makefile remains the single source of test-execution policy.

Conceptually:

```text
push / pull request / manual dispatch
        |
        v
development-container CI environment
        |
        v
make test JAVA_TEST_JOBS=4 PROTOS_TEST_JOBS=4
        |
        +-- test-java
        |
        `-- test-protos
```

Portable-distribution validation remains available as an explicit validation
surface but is not repeated on every ordinary push merely to duplicate already
established distribution evidence.

## Live closure evidence

Two successful GitHub Actions runs on the same final GITHUB017 product revision
provide the live restoration and confirmation evidence.

### Push restoration run

```text
RUN_NUMBER=1820
RUN_ID=35064998115
EVENT=push
HEAD_SHA=b616caebde59ea1aa95c3a208182c1c9cef1b8f6
CONCLUSION=success
```

### Manual confirmation run

```text
RUN_NUMBER=1821
RUN_ID=35068351353
EVENT=workflow_dispatch
HEAD_SHA=b616caebde59ea1aa95c3a208182c1c9cef1b8f6
CONCLUSION=success
```

The second run confirmed that the restored CI remained green independently of
the initial push-triggered execution.

Stable GitHub run identities are sufficient closure evidence here; raw Actions
logs are intentionally not copied into `docs/project/evidence/GITHUB017/`.

## Deferred optimization

The remaining hosted-runner latency is not a GITHUB017 correctness blocker.

Future work is owned separately by:

```text
GITHUB019 / #534
Decouple CI execution from devcontainer image builds
```

That work may evaluate a prebuilt CI image, separate Java/Protos Actions steps,
Maven dependency caching, and cold/warm latency improvements. It does not reopen
GITHUB017.

## GITHUB020 reconciliation

GITHUB020 generalized the formal-work closure-evidence rule after GITHUB017 had
already closed.

The governing Protos publication for that rule is:

```text
CLOSURE_POLICY_REVISION=5ed6e25b1e3a10946331c8fe1b4995f488d1ff12
```

GITHUB017 is therefore the first retrospective reconciliation case under the new
contract.

Its closure decision is:

```text
ISSUE_CLOSURE_COMMENT=PASS
CLOSURE_EVIDENCE_IDENTIFIED=PASS
DURABLE_RECORD_DECISION=REQUIRED
PROTOS_REVISION=b616caebde59ea1aa95c3a208182c1c9cef1b8f6
CLOSURE_POLICY_REVISION=5ed6e25b1e3a10946331c8fe1b4995f488d1ff12
EVIDENCE_PATH_ROLE_FIRST=PASS
RAW_LOG_ARCHIVAL_REQUIRED=NO
GITHUB_PROJECT_IS_EVIDENCE_AUTHORITY=NO
```

The exact `PROJECT_RECORD_REVISION` is recorded back on GITHUB017 / #492 after
this file is published, because a commit cannot embed its own final SHA as an
input to itself.

## Closure statement

GITHUB017 is complete.

Automatic test CI is active, the canonical test entry point is explicit, two
live successful workflow runs prove the restored path, redundant routine
distribution validation has been removed without weakening its independent
evidence, and remaining CI-performance work has a separate formal owner.
