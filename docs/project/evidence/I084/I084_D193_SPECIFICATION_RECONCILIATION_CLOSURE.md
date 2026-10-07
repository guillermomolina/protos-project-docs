# I084 — D193 normative specification reconciliation closure

Status: **COMPLETE**

Implementation issue: `guillermomolina/protos#837` — I084

Decision authority: `D193 / guillermomolina/protos#836`

Parent library work: `LIB021 / guillermomolina/protos#835`

Publication date: **2026-10-07**

## Published product revision

~~~text
PROTOS_REVISION=6ae0a9b8a992b80d802fe9273a1170687a271edb
COMMIT_SUBJECT=I084: reconcile D193 std:interop public operation contract into normative specification
IMPLEMENTATION_VERSION=0.3.280-SNAPSHOT
SPECIFICATION_REVISION=0.1.448
~~~

The published commit is based directly on the prior I082-G product revision
`c0ac98971df115d64b7bc9f146e8b02e11da30e6`.

## Exact changed paths

I084 changed exactly:

~~~text
spec/PROTOS_SPEC_CHANGELOG.md
spec/semantics/VALUES_AND_COLLECTIONS.md
~~~

No runtime, Standard Library source, tests, root `pom.xml`, root
`CHANGELOG.md`, or archived specification changelog changed.

## Normative result

Specification revision 0.1.448 reconciles the owner-approved D193 A− contract.

The normative public module is:

~~~text
std:interop
~~~

with exactly the initial operations:

~~~text
invoke(target, ...arguments)
instantiate(target, ...arguments)
readMember(target, name)
writeMember(target, name, value)
~~~

The Foreign Values owner now defines:

- `invoke` as explicit foreign execution without ordinary `call` lookup;
- `instantiate` as explicit construction, distinct from `invoke`;
- `readMember` as explicit underlying foreign-member read that bypasses
  ordinary Protos slot/projection precedence only for that explicit operation;
- `writeMember` as explicit foreign-member mutation whose successful normal
  result is the exact original Protos `value`;
- the existing D188 entered/not-entered Error boundary;
- exact D189 callback reuse rather than a second callback model;
- possession-only authority with no discovery, acquisition, session creation,
  Context creation, registration, trusted mode, or dependency acquisition;
- exact existing provider/session/generation lifetime with no rebinding;
- unchanged Actor/P non-transferability.

The normative text also retains the D193 correction that explicit
`invokeMember` is not universally reducible to `invoke(readMember(...))`:
an invocable-but-unreadable foreign member is possible. The operation remains
deferred because no current Protos provider/workload requires it.

## Preserved decisions

~~~text
D188_DELTA=NONE
D189_DELTA=NONE
PLAT052_DELTA=NONE
PLAT053_DELTA=NONE
D192_DELTA=NONE
~~~

Ordinary member lookup/write/invocation, ordinary callability, `at` /
`atPut`, projected foreign `each`, raw foreign identity/equality/hash,
`ForeignError`, callback lifetime, Actor/P boundaries, provider/session
topology, and dependency acquisition are unchanged.

## Validation evidence

The maintainer reported after publication:

~~~text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
~~~

This durable record retains the human-reported result; the documentation
publisher did not rerun the product validation.

The published commit itself confirms that only the two specification paths
above changed and that the Maven implementation version remains
`0.3.280-SNAPSHOT`.

## Release result

~~~text
I084_STATUS=COMPLETE
D193_SPECIFICATION_RECONCILIATION=COMPLETE
LIB021_IMPLEMENTATION_RELEASE=YES
~~~

LIB021 may now implement the four-operation A− surface without reopening D193
unless implementation exposes a genuinely new substantive semantic choice.

The next implementation should remain a slice of LIB021 rather than a new Issue
unless current work later crosses an independent coordination trigger.

## Project-record publication

This closure record was prepared from project-record base:

~~~text
PROJECT_RECORD_BASE_REVISION=27a5c6e3d6cd00301c18697996b8c7abd19a29eb
~~~

## AI-assistance disclosure

This durable closure record was materially prepared with AI assistance from
ChatGPT using the exact published I084 commit, live GitHub coordination, the
ratified D193 contract, and the maintainer-reported validation result. No
independent human review is claimed.
