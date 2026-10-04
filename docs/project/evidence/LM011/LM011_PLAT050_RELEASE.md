# LM011 — PLAT050 release of canonical formatter implementation

Status: **READY FOR IMPLEMENTATION**

Date: **2026-10-04**

Owning workstream: `LM011 / guillermomolina/protos#670`

Released by: `PLAT050 / guillermomolina/protos#792`

Governing public policy: `D183 / guillermomolina/protos#791`

Product authority:

~~~text
PROTOS_REVISION=a38470bc6e2f68e770ddc8054053995bb2477b19
~~~

## Release result

D183 is ratified as Candidate B — fixed structural style + conservative lexical preservation.

PLAT050 is now ratified as Candidate F — on-demand hybrid source-layout view plus exact bundled Protos formatter policy over a tool-neutral host source mechanism.

The two decisions make the next implementation step mechanical.

~~~text
LM011_A=COMPLETE
D183=RATIFIED
PLAT050=RATIFIED

LM011_B_RELEASED=YES
CURRENT_BLOCKER=NONE
NEXT_SLICE=LM011-B1
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~

## LM011-B1 scope

The first implementation slice establishes the source-preservation foundation required by the canonical formatter.

It may implement:

1. on-demand lexer trivia occurrences for horizontal whitespace, line comments and block comments;
2. opt-in parser source facts for semicolon/newline sequence separator origin, single-parameter Closure source form and trailing-Closure origin;
3. an immutable editor-neutral source-layout view over exact source, token occurrences, Surface AST, trivia and parser source facts;
4. deterministic comment attachment with D183's `OWN_LINE`, `END_OF_LINE` and `EMBEDDED_BETWEEN_TOKENS` classes and grammar-significant relationships;
5. source-position-independent preservation projections for the D183 invariants;
6. focal tests proving block-comment newline containment, line-comment terminating-newline behavior, separator-kind preservation, trailing-Closure attachment, exact literal spelling and zero ordinary-execution formatter metadata.

The slice must preserve the ordinary lexer/parser/runtime paths when formatter metadata is not requested.

## Boundaries

LM011-B1 does not authorize:

~~~text
CLI spelling
LSP formatting integration
VS Code integration changes
range formatting
check mode
on-type formatting
parser recovery
partial invalid-source formatting
style configuration
TypeScript formatting semantics
lossless CST
user-module execution
host-owned FormatterPolicy
~~~

Those are either later LM011 slices or explicitly deferred capabilities.

## Later mechanical sequence

After LM011-B1, the ratified architecture permits later LM011-B implementation slices to add:

~~~text
exact bundled Protos D183 formatter policy
whole-document editor-neutral formatting authority
semantic-equivalence projection/check
source-preservation postcondition check
determinism/idempotence/fail-closed corpus
JVM and Native Image evidence
~~~

LM011-C later owns the CLI formatting surface.

LM011-D later owns standard-LSP / VS Code Format Document and format-on-save integration.

LM011-E owns final idempotence, corpus, end-to-end evidence and closure.

No new Dxxx/PLATxxx is required before LM011-B1 unless implementation discovers a concrete contradiction with D183 or PLAT050.

## Result

~~~text
LM011_STATUS=READY
LM011_B_RELEASED=YES
IMPLEMENTATION_AUTHORIZED=YES
NEXT=LM011-B1
REPOSITORY=guillermomolina/protos
OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~
