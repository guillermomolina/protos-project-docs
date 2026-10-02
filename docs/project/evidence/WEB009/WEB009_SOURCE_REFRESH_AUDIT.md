# WEB009 — public-source refresh preimplementation audit

Date: 2026-10-02

## Work identity

~~~text
WORK_ITEM=WEB009
WEBSITE_ISSUE=guillermomolina/protos-website#5
WEBSITE_REPOSITORY=guillermomolina/protos-website
PROTOS_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative project evidence. It does not redefine
Protos language semantics, website architecture, release authority, or the
current live state of either Issue tracker.

## Website state audited

~~~text
WEBSITE_REVISION=1834b97c4f2e208bb6335f90383d61ac791bb8b6
WEBSITE_SOURCE_LOCK=4a6df50e8577ebb0f3fc352df13dcbab368f955a
WEBSITE_SOURCE_CONTRACT=EXACT_SHA
WEBSITE_ARCHITECTURE=ASTRO_STarlight_STATIC
~~~

At the audited website revision, `protos-source.lock.json` consumes canonical
Protos content from exact revision
`4a6df50e8577ebb0f3fc352df13dcbab368f955a`.

The website materializer consumes, among other canonical paths:

~~~text
README.md
docs/guide
docs/news
docs/design
docs/assets/branding
spec
protos/tutorials
protos/examples
protos/lib
LICENSE.TXT
pom.xml
src/main
~~~

The repository's `prepare-protos.mjs` contract fetches the exact locked SHA,
materializes canonical guide/tutorial/example/spec/library/news content, and
supports a `--check` mode that verifies generated website state against that
lock.

## Current Protos source state

At this audit, the current published Protos branch head is:

~~~text
PROTOS_HEAD=8ea87fb1794599247f0e95fd5570d7dd7e08a52f
PROTOS_HEAD_SUBJECT=PERF025-C1 checkpoint: inline Object-body execution (C1a+C1b)
PROTOS_HEAD_PARENT=57d8cf4ec195aca3cb5b7c33d37755d965e9f3df
~~~

The subject is not a reliable description of the head commit's delta.

The original retained PERF025-C1 checkpoint remains the earlier ancestor:

~~~text
PERF025_C1_REVISION=595d547b2e9714a185a3cfceadf74565229f43e7
~~~

and must not be rewritten as though the later head were the original C1
publication.

Between that C1 checkpoint and the current head, the repository contains the
published PLAT042/PLAT043 cutovers, including:

~~~text
PERF025_C1C_PLAT042_REVISION=0a5115caddba8ebb7bb4275ce32441ab90938d3d
PERF025_C2B_PLAT043_REVISION=57d8cf4ec195aca3cb5b7c33d37755d965e9f3df
~~~

PERF025 remains open. The post-C2B unchanged-workload stack gate remains pending,
and later PERF026/PLAT044 work may still affect final carrier/open-coding
architecture. No website claim may state that PERF025, carrier retirement, or
the broader historical performance gap is complete.

## Current-head discrepancy relevant to WEB009

The current head commit
`8ea87fb1794599247f0e95fd5570d7dd7e08a52f` changes the public loop selector
surface and repository-owned sources from `while` to `whileTrue`, matching
the implementation scope of I078 / `guillermomolina/protos#764`.

However, at audit time:

~~~text
I078_STATE=OPEN
I078_LABEL_STATUS=status:ready
I078_COMMENTS=0
IMPLEMENTATION_VALIDATION_EVIDENCE=ABSENT_FROM_ISSUE

POM_VERSION_AT_57d8=0.3.134-SNAPSHOT
POM_VERSION_AT_8ea8=0.3.134-SNAPSHOT
IMPLEMENTATION_CHANGELOG_CHANGED_BY_8ea8=NO
SPEC_CHANGELOG_CHANGED_BY_8ea8=NO
~~~

I078's own acceptance criteria require specification and implementation
changelog/version reconciliation and required validation before closure.

Therefore the current head cannot yet be treated by WEB009 as an
unambiguously reconciled canonical publication point merely because it is the
moving branch head.

This is a source-coherence blocker for a one-shot WEB009 refresh, not a language
design blocker.

## News/materialization consequence

At current Protos head, the canonical `docs/news` set contains no new article
after the existing 2026-09-29 item.

Therefore WEB009 should not invent an independent website news article merely to
announce PERF025 progress.

The useful website work is exact-source synchronization of canonical guide,
reference, design, examples, library and existing news material.

## WEB009 disposition

~~~text
WEB009_STATUS=BLOCKED_PENDING_SOURCE_RECONCILIATION
BLOCKER=I078/#764
BLOCKER_KIND=PUBLIC_SOURCE_COHERENCE

SAFE_LAST_PERF025_CHECKPOINT=57d8cf4ec195aca3cb5b7c33d37755d965e9f3df
CURRENT_MOVING_HEAD=8ea87fb1794599247f0e95fd5570d7dd7e08a52f

PIN_8ea87fb_NOW=NO
PIN_MOVING_MAIN=NO
CREATE_WEBSITE_LOCAL_CANONICAL_NEWS=NO
CHANGE_RELEASE_LOCK=NO
~~~

A refresh to `57d8cf4...` would be internally coherent and much newer than the
website's current pin, but would immediately require another publication once
the already-published `whileTrue` source state is reconciled. WEB009 therefore
defers the source-lock update rather than intentionally publishing a short-lived
intermediate pin.

## Unblock condition

WEB009 may resume its one-shot source refresh when the Protos source selected for
publication has a coherent revision-bound authority state.

For the current head path, that means I078 must at minimum reconcile:

- the published `whileTrue` implementation/specification delta;
- implementation version/changelog requirements;
- specification changelog/revision requirements;
- required validation evidence;
- live Issue state.

The source refresh should then select the exact resulting Protos revision, not a
moving branch reference.

## Required WEB009 implementation after unblock

The website implementation should:

1. update `protos-source.lock.json` to the selected exact Protos SHA;
2. run the repository-owned source materializer;
3. verify canonical links and generated guide/tutorial/example/spec/library/news
   content against the lock;
4. keep `protos-release.lock.json` independent and unchanged unless release
   authority independently changes;
5. build the static site;
6. inspect homepage and canonical-derived pages for stale revision references;
7. avoid unsupported PERF025/carrier/performance claims; and
8. publish/verify the website through its normal human-executed workflow.

## Cross references

- WEB009: `guillermomolina/protos-website#5`
- PERF025: `guillermomolina/protos#758`
- I078: `guillermomolina/protos#764`
- D180: `guillermomolina/protos#762`
- PERF025-C1 evidence:
  `docs/project/evidence/PERF025/PERF025_C1_INLINE_OBJECT_BODY_CHECKPOINT.md`
- PERF025-C2B evidence:
  `docs/project/evidence/PERF025/PERF025_C2B_PLAT043_BOOLEAN_OWNERSHIP_CUTOVER.md`

~~~text
WEB009_AUDIT=COMPLETE
WEBSITE_PUBLICATION=NOT_STARTED
SOURCE_LOCK_UPDATE=DEFERRED
DURABLE_BLOCKER_EVIDENCE=YES
~~~
