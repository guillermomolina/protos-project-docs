# I119 — Agent Editor: autonomous A+B+C source edit handoff

**Date:** 2026-10-21  
**Product:** `guillermomolina/protos`, current local **HEAD** (do not fetch/checkout/rebase).  
**Issue:** [I119/#945](https://github.com/guillermomolina/protos/issues/945).  
**Decision:** [D211 ratified A+B+C](../../decisions/language/D211_CONTEXTUAL_IF_BUNDLE.md).  
**Execution class:** IMPLEMENTATION. Agent edits source files only; no tests/builds/commands that validate, no state-changing Git commands, no product commits or pushes.  
**Operator:** Human executor separately runs diagnostics/tests and publishes after complete A+B+C patch. **No intermediate partial-A or partial-B/C product commit.**

## Self-contained Claude prompt — paste into the editing agent

> You are implementing Protos issue **I119** in **`/workspaces/protos`**. The local `protos` repo **is already on HEAD**. Everything needed is in its local files or in this handoff. **DO NOT** browse the web, inspect any other repository, fetch, checkout, switch branch, install packages, compile, run tests, run validation/benchmark tools, or commit/push. Read root/scoped `AGENTS.md` and implementation work policy locally. Inspect only exact patch-owned files, local dependent implementation/tests, and relevant local normative sections; preserve unrelated changed files. Use `grep`, not `rg`; apply focused checked Python replacements for mechanical edits, direct source file edits for AST/parser/tool work. A human owns all compilation, testing, validation and product Git publication.
>
> **Owner-ratified design is NOT up for investigation or renegotiation:** A+B+C must arrive together in a **single complete product patch**, even though you may edit in P1/P2/P3 batches. A: remove standard `Object.if(block)`/`Object.else(block)` behavior with **no alias**; B: `if(condition) { yes }` is a contextual expression sending ordinary `ifTrue` on the once-evaluated condition; C: `if(condition) { yes } else { no }` is a contextual expression sending ordinary `ifTrueIfFalse`, with Closure arguments for both branches. Existing Boolean standard protocol `ifTrue`, `ifFalse`, `ifTrueIfFalse`, `not`, `and`, `or` and ordinary custom slots are preserved. No new Truffle control opcode, new Java keyword, guest keyword, truthiness, hidden synchronous branch execution, implicit await, or extra runtime control abstraction.
>
> **Grammar recognition:** bare **ungrouped** identifier spelled `if` + exactly one explicit **non-spread** parenthesized condition expression + immediately attached same-continuing-logical-line **parameterless braced trailing Closure**; only such exact source takes priority over a lexical `if` function call. For the optional two-arm shape, recognize bare contextual `else` immediately following the first block on the **same logical line**, with an attached braced parameterless Closure. Do not make either name globally reserved; no `elseIf`/`elif`; never extend newline continuation rules. Ordinary `(if)(x) {body}`, `holder.if(x) {body}`, `if(x)`, `if(a,b) {body}`, `if(...args) {body}`, `if(x)\n{body}`, `else {body}` must remain ordinary grammar forms. Parse both branches **before postfix chaining**, nested conditionals, grouping, assignments, arguments, `^`, comments and source spans must work. A logical NEWLINE between `}` and `else` ends the first expression, not a two-arm conditional; block comments consuming physical newlines do not create NEWLINE tokens.
>
> **Correct evaluation and canonical lowering:** `CanonicalSend(canonicalize(condition), "ifTrue", List.of(canonicalize(yesClosure)), fullSpan)` for B; `CanonicalSend(canonicalize(condition), "ifTrueIfFalse", List.of(canonicalize(yesClosure),canonicalize(noClosure)), fullSpan)` for C. Do NOT wrap the condition in a new Closure: D210 wraps its `while` predicate to reevaluate it, **but D211 if conditions must evaluate exactly once**. Do NOT lower C to chained `.ifTrue(...).ifFalse(...)` or old `.if(...).else(...)`. Exact selected-body normal result for both; false without else returns null; both Closure argument values are created left to right, but only selected callback is invoked under the standard behavior. Preserve ordinary lookup, custom overrides and generic fallback, canonical Boolean standard receiver-domain validation, NLR/home activation, `this`/`super`/`context`, Task/Future/error/cancel/suspend/unwind/ensure and absence of effect replay.
>
> **P1 — replace/remove standard selectors (no tests yet).** Use the checked batch below for the 3 confirmed files when their original anchors match. In `ProtosStandardBooleanProtocol.java`, remove the `StructuredCallbackKind.IF/ELSE` variants, native closure bodies and values, identity admission, installed root selectors, and native `ifMessage`/`elseMessage`; keep six retained selectors and exact canonical dispatch. In `ProtosCoreBootstrap.java`, remove the two root slots from the required publication set and standard validation; do not ban custom `if`/`else`. In `ProtosStructuredControlCalls.java`, remove only IF/ELSE cases from `hasCallback`, selected callback index, skipped/immediate results, and finish-return handling. Search local dependent `ProtosPreparedCalls`, guarded selection, compiler lowering and Java tests only to remove stale enum/selector dependencies without touching unrelated control behavior. Keep all source header licenses.
>
> **P2 — implement paired contextual AST and canonicalization.** Add `src/main/java/com/guillermomolina/protos/parser/ast/SurfaceContextualIf.java` as a proper licensed sealed Surface record with `condition: SurfaceExpression`, `yes: SurfaceClosure`, optional `no: SurfaceClosure` and `span: SourceSpan`, checking invariants in the compact record constructor. Add to `SurfaceExpression.java` permits. In `ProtosParser.java`, build a context-sensitive node in/near `parseCallSuffix` following the D210 `isContextualWhile` pattern. The optional `else` is parsed while owning the first call's exact source shape and before returning into `parsePostfixFoundation`. Consume `else` and second braced body only within approved adjacency; keep the first `if` identifier nonlexical within contextual node but all other occurrences ordinary; include complete two-arm source span. In `Canonicalizer.java` add a `SurfaceContextualIf` switch case and ordinary `CanonicalSend` lowering as above; do not duplicate D210 condition wrapping. In `ProtosParserSourceFacts.java` allow contextual-if branch Closure origin(s) with correct owning node; adapt any enum/capture path necessary for parser facts. Add focused parser and canonical tests for all boundary cases, nesting, nonlocal return and postfix composition.
>
> **P3 — wire tools, fix conformance and normative contract.** Extend exhaustive Surface visitors: `analysis/ProtosStaticDefinitions.java`, `ProtosStaticReferences.java`, `ProtosStaticHover.java`, `ProtosStaticLint.java`, `ProtosDocumentSymbols.java`, `ProtosSourceLayoutView.java`, `ProtosSourceLayoutToolBridge.java` (under `execution/host/tooling/`), and any local switch also requiring the new sealed subtype. **Critical:** the `if` condition is an eager expression in current execution context; DO NOT copy the deferred `SurfaceContextualWhile` condition analysis from `ProtosStaticDefinitions`. Both branch bodies are lexical Closures. Represent `else` as a structural boundary, not lexical reference. Update `protos/tools/formatter/Structural.protos` to print both forms idempotently, with a same-line `} else {` join; preserve tokens/comments/source layout and avoid silently producing `}\nelse {`. Adjust docs/guide `04-control-flow-through-protocols.md`, `spec/semantics/VALUES_AND_COLLECTIONS.md` §16, `spec/PROTOS_GRAMMAR.md` §36/contextual D210 material, and `spec/PROTOS_LANGUAGE_SPEC.md` §33 to remove normative D200 selectors and document exact B/C grammar, results and unavoidable compatibility break. Keep previous D200/D051/D210 changelog entries historically truthful.
>
> **Focal regression updates:** remove or rewrite standard-IF/ELSE assertions in `protos/tests/conformance/boolean/receiver-preserving-if-else.protos`; replace that fixture's semantic intent with a dedicated D211 contextual-if suite (reuse/restructure file as appropriate, keep suite manifest consistent) including true/false result identity, false no-else null, exact once condition evaluation, branch side effects, Future identity, custom `ifTrue`/`ifTrueIfFalse`, Error/NLR/cancel/suspension, child tasks and `Object.hasSlot("if")`/`Object.hasSlot("else")` false. In `protos/tests/conformance/control/control-spelling-is-ordinary.protos` preserve ordinary lexical `if` test via grouped `(if)(123) {42}` and add actual contextual precedence test, preserving ordinary `else` name. Update `ProtosStandardBooleanProtocolTest.java`, `ProtosI072PhaseEStructuredSendConvergenceTest.java`, `ProtosI095BooleanControlInlineTest.java`, `ProtosI095BooleanProtocolInlineTest.java`, `ProtosI101ProfileSpecializedLoweringTest.java`, `ProtosParserTrailingClosureTest.java`, `ProtosParserSourceFactsTest.java`, `CanonicalizerCallTest.java`, formatter/source-layout/analysis tests and other directly matching local tests as needed. Do not mechanistically migrate independent libraries, old release benchmarks or examples not in the implementation path. Include negative tests for literal syntax collisions and for removed public selectors; tests asserting old D200 behavior must not simply be deleted without replacement coverage of retained protocol behavior.
>
> **Final editing boundary:** no `pom.xml`, `CHANGELOG.md`, or `spec/PROTOS_SPEC_CHANGELOG.md` until the human executor reports all final substantive tests/checks green. At that point update those 3 files **once**, based on actual HEAD/metadata, following repository release rules; no tests after those metadata edits per local protocol. Never invent a version. Do not run `git diff --check`, tests, Maven, Make, Podman, or publication commands yourself. At handoff enumerate edited paths, normative effects, the exact human-run focal/broad commands requested, tests not run by agent and remaining caveats. One final human product commit/push for A+B+C only.

## Pre-researched Truffle references: for orientation, not further research

**Nothing from other Truffle languages is necessary to implement D211.** Current Protos's D210 implementation and Boolean protocol are the actual blueprint. These source-verified examples are supplied only to prevent wrong assumptions; do **not** follow them or open outside repositories:

1. Oracle Truffle SimpleLanguage `oracle/graal`, `truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/parser/SLBytecodeParser.java`, `visitIf_statement`: when an `else` exists it builds `b.beginIfThenElse()` / `b.endIfThenElse()` around condition/then/alt; without `else` it builds `b.beginIfThen()` / `b.endIfThen()`. It includes an `SLToBoolean` conversion. **Do not copy** its distinct statement control or truthiness: Protos canonicalizes to **ordinary `ifTrue`/`ifTrueIfFalse` message sends** and strict canonical Boolean standard methods. Reference exact upstream commit: `3a4b3b12e524d49521ff3982a8ff5dd9edd999dd`.
2. GraalJS `oracle/graaljs`, `graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/control/IfNode.java`: `IfNode` owns condition/then/else child nodes; `doBoolean` selects one child and `doObject` uses `JSToBooleanNode`. **Do not copy** independent JS statement AST nodes, JS truthiness, EMPTY result, or its generator node semantics; Protos preserves Closure evaluation and result-producing ordinary messages. Reference exact upstream commit: `107b0a3c87f8f250b48da7c2fb702c6a83a38e50`.
3. GraalPy similarly uses Python statement/truthiness semantics, not Protos's Boolean method/Closure result contract. There is **no GraalPy implementation dependency** for I119. The existing Protos `SurfaceContextualWhile` / `Canonicalizer.lowerContextualWhile` is a stronger exact-domain structural precedent; the one important difference is eager one-shot `if` predicate evaluation.

## P1 checked source-edit batch — exact commands for editor only

This code edits the 3 already source-verified Java files without executing tests or Git modifications. Run **only if** those paths are unmodified and the expected anchors match the local HEAD; do not paste into a checkout with unrelated changes on the same owned files. It is **not** a complete I119 patch; continue P2/P3 before requesting the human to run tests.

```bash
cd /workspaces/protos
python3 - <<'PY'
from pathlib import Path
root = Path("src/main/java/com/guillermomolina/protos")
b = root / "execution/protocols/values/ProtosStandardBooleanProtocol.java"
boot = root / "execution/host/embedding/ProtosCoreBootstrap.java"
ctl = root / "execution/bytecode/activation/structured/ProtosStructuredControlCalls.java"
new = {}

def remove_once(data, old, label):
    if data.count(old) != 1:
        raise RuntimeError(f"{label}: expected exactly one anchor; found {data.count(old)}")
    return data.replace(old, "", 1)

def replace_once(data, old, replacement, label):
    if data.count(old) != 1:
        raise RuntimeError(f"{label}: expected exactly one anchor; found {data.count(old)}")
    return data.replace(old, replacement, 1)

s = b.read_text(encoding="utf-8")
s = replace_once(s, "        OR(1),\n        IF(1),\n        ELSE(1);", "        OR(1);", "enum")
s = remove_once(s, "    private static final ProtosNativeClosureBody STANDARD_IF_BODY =\n            ProtosStandardBooleanProtocol::ifMessage;\n    private static final ProtosNativeClosureBody STANDARD_ELSE_BODY =\n            ProtosStandardBooleanProtocol::elseMessage;\n", "native bodies")
s = remove_once(s, "    private static final ProtosClosureValue STANDARD_IF =\n            standardNative(STANDARD_IF_BODY);\n    private static final ProtosClosureValue STANDARD_ELSE =\n            standardNative(STANDARD_ELSE_BODY);\n", "native closures")
s = replace_once(s,
    "     * original Boolean selectors were already native before PLAT028, and D200 adds if/else\n     * through the same helper; stable Closure/body identities let C-prime recognize the structured callback operations without expanding that boundary.\n",
    "     * original Boolean selectors use stable Closure/body identities that let C-prime\n     * recognize structured callbacks without expanding that boundary.\n",
    "native provider comment")
for old, label in [
    ("        if (body == STANDARD_IF_BODY) return StructuredCallbackKind.IF;\n", "body if"),
    ("        if (body == STANDARD_ELSE_BODY) return StructuredCallbackKind.ELSE;\n", "body else"),
    ("        if (behavior == STANDARD_IF) return StructuredCallbackKind.IF;\n", "selection if"),
    ("        if (behavior == STANDARD_ELSE) return StructuredCallbackKind.ELSE;\n", "selection else"),
    ("        install(object, \"if\", STANDARD_IF);\n", "install if"),
    ("        install(object, \"else\", STANDARD_ELSE);\n", "install else"),
]:
    s = remove_once(s, old, label)
start = s.index("    /*\n     * D200: if/else return the original canonical receiver")
end = s.index("    private static Object requireReceiverAndArity(", start)
s = s[:start] + s[end:]
new[b] = s

s = boot.read_text(encoding="utf-8")
s = remove_once(s, '                        "if",\n                        "else",\n', "root slot set")
s = remove_once(s,
    '        requireStandardBooleanSelection(object, "if",\n                ProtosStandardBooleanProtocol.StructuredCallbackKind.IF);\n        requireStandardBooleanSelection(object, "else",\n                ProtosStandardBooleanProtocol.StructuredCallbackKind.ELSE);\n',
    "root selection validation")
s = replace_once(s, "I095 inlines all eight standard Boolean selectors;",
                 "I095 inlines the standard Boolean selectors;", "root comment")
new[boot] = s

s = ctl.read_text(encoding="utf-8")
for old, label in [
    ("                case IF -> receiver == ProtosBooleanValue.TRUE;\n", "hasCallback if"),
    ("                case ELSE -> receiver == ProtosBooleanValue.FALSE;\n", "hasCallback else"),
    ("                // D200: the callback's normal result is ignored; the receiver is returned.\n                case IF, ELSE -> receiver;\n", "callback result"),
    ("                case IF, ELSE -> receiver;\n", "immediate result"),
]:
    s = remove_once(s, old, label)
s = replace_once(s,
    "                case IF_TRUE, IF_FALSE, AND, OR, IF, ELSE -> 0;",
    "                case IF_TRUE, IF_FALSE, AND, OR -> 0;",
    "selectedIndex")
new[ctl] = s

# Validate every targeted transformation before writing any file.
for path, output in new.items():
    if not output or output == path.read_text(encoding="utf-8"):
        raise RuntimeError(f"{path}: zero/invalid delta")
for path, output in new.items():
    path.write_text(output, encoding="utf-8")
    print("EDITED", path)
PY
```

Do not issue a separate product commit, partial validation claim, or release after P1. If an anchor fails, **do not force** the replacement; inspect the local HEAD and adjust the targeted edit without expanding scope.

## Acceptance handoff for the human

After all P1/P2/P3 edits, provide a precise list of changed paths; explain custom dispatch preservation, newline/else grammar behavior, exact selected-result identity and condition-evaluation proof, and indicate all tests as **NOT EXECUTED BY AGENT**. Human executor contract is the separate `I119_HUMAN_EXECUTOR_2026_10_21.md` document.
