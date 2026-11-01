# DOC014 — Editing agent: one atomic bitwise learning-material slice

## Task identity and absolute execution boundary

**TYPE: IMPLEMENTATION (DOCUMENTATION AND RUNNABLE PROTOS MATERIAL), NOT INVESTIGATION.**
**PRODUCT REPOSITORY: `guillermomolina/protos` ONLY.**
**FORMAL ISSUE: https://github.com/guillermomolina/protos/issues/1020.**
**BASE: assume the local product checkout is already at its intended current HEAD.** The recorded I137 product commit `c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5` (0.3.431-SNAPSHOT; spec 0.1.486) is historical verified context, not an instruction to checkout or reset. All task-relevant source and normative rules are in the local product checkout. Never inspect another repository, browse the web, or revisit an earlier cross-language design investigation.

**You are an EDIT-ONLY agent:** read local task-owned files and local instruction files; edit or create exactly the files enumerated below; inspect the resulting text. **Never** run shell commands, programs, linters, builds, Maven, tests, formatters, validation scripts, Git status/add/commit/push/pull/fetch/checkout, or deploy. Those actions belong to the human executor. Do not create or update any GitHub issue. Do not request credentials. No external tool/source/repository is needed.

**ONE SINGLE ATOMIC DOC014 SLICE:** make all six content edits together. No DOC014-A/B/C, no separate implementation phase, no intermediate publication, no unfinished choice left for the human. Preserve any unrelated pre-existing local edits; do not replace whole files merely to insert paragraphs. This task teaches the already-shipped language, not adds a language feature.

Read and honor local `AGENTS.md`, `AGENTS.work/DOCUMENTATION.md`, `AGENTS.work/IMPLEMENTATION.md` (runnable learning sources), `protos/AGENTS.md`, `AGENTS.work/REFERENCE.md` (source license), and `docs/guide/SOURCE_STYLE.md`. The authoritative finished behavior is already in `spec/PROTOS_GRAMMAR.md` §21 + symbolic lexing and `spec/semantics/VALUES_AND_COLLECTIONS.md` (Integer bitwise messages). Use those local files as authority without design research or changes. `AGENTS.work/COORDINATION.md` belongs to the coordinating assistant, not to your editing task.

## Approved Protos contract — no unresolved questions

Exactly six expressions using five fixed operator tokens:

| Source | Ordinary message |
| --- | --- |
| `a & b` | `a.bitAnd(b)` |
| `a \| b` | `a.bitOr(b)` |
| `a ~ b` | `a.bitXor(b)` |
| `~a` | `a.bitNot()` |
| `a << n` | `a.shiftLeft(n)` |
| `a >> n` | `a.shiftRight(n)` |

Integer values have mathematical unbounded signed two's-complement bit semantics; `~0 == -1`; `(-3) >> 1 == -2`; `1 << 64 == 18446744073709551616`, with automatic SmallInteger/BigInteger normalization. Bitwise operators use ordinary overridable message dispatch and strict one-time RHS evaluation, including on custom non-Integer receivers implementing the named methods. `&&`/`||` remain lazy, `^` is nonlocal return, and `~` is binary XOR or prefix complement by grammar position. Shift counts must be genuine nonnegative Integers; negative counts signal `Error`, not a shift in the other direction. Fixed-width tasks explicitly mask their results, e.g. `value & 4294967295` for 32 bits.

Operator binding strength high-to-low: postfix > unary `! - ~` > `* / %` > `+ -` > `<< >>` > `&` > binary `~` > `|` > comparisons > equality > `&&` > `||`; new binary forms are left-associative. Maximal-munch rejects `a~~b`, `a&-b`, `a<<-1`, `a>>=b`, `a>>>b`, `a~=b`, and `a|>b` as entire invalid symbolic runs; use `a ~ ~b`, `a & -b`, `a << -1` (the last is syntactically valid but fails on execution). No `^^`, `>>>`, bitwise compound assignments, custom operators, host-width wrapping or Number/Boolean coercion.

## File-by-file edits: six and only six files

### FILE 1 — NEW `protos/tutorials/01-values-and-slots/04-bitwise-operators.protos`

Create a small **standalone-CLI** lesson immediately following `03-assignment.protos` in the existing beginner directory. Begin with the exact, complete APL Part 5 source notice copied from `protos/tutorials/01-values-and-slots/03-assignment.protos` or `protos/examples/numbers/exact-and-complex.protos`, preserving its complete wording; do not shorten the notice. Then use the following **exact program body** (minimal whitespace adjustment consistent with neighboring tutorial files is fine):

```protos
// Six fixed bitwise forms are ordinary Integer message sends.
// Expected output, one value per line: 8, 14, 6, -1, -2, true.

left: 12
andValue: left & 10
namedAnd: left.bitAnd(10)
orValue: 12 | 10
xorValue: 12 ~ 10
complement: ~0
large: 1 << 64
negativeShift: (-3) >> 1

// Integer bits have unlimited sign extension; mask explicitly for 32-bit work.
word32: (~0) & 4294967295

// The & level binds tighter than |, and ~ is both prefix NOT and binary XOR.
mixed: 1 | 6 & 3
doubleTilde: 12 ~ ~10

// Both bitwise & and | evaluate their right argument even for 0 and -1.
// Boolean && is deliberately lazy and does not evaluate the right expression.
calls: 0
andResult: 0 & (calls = calls + 1)
orResult: -1 | (calls = calls + 1)
lazy: false && (calls = calls + 1)

// All six operators send ordinary messages. Receivers need not be Integers.
custom: {
    bitAnd: (argument) => { 40 + argument }
}
customResult: custom & 2

verified:
    (andValue == 8) &&
    (andValue == namedAnd) &&
    (orValue == 14) &&
    (xorValue == 6) &&
    (complement == -1) &&
    (large == 18446744073709551616) &&
    (negativeShift == -2) &&
    (word32 == 4294967295) &&
    (mixed == 3) &&
    (doubleTilde == -7) &&
    (andResult == 0) &&
    (orResult == -1) &&
    (calls == 2) &&
    (lazy === false) &&
    (customResult == 42)

if(!verified) {
    Error().signal()
}

print(andValue)
print(orValue)
print(xorValue)
print(complement)
print(negativeShift)
print(verified)
```

Use only existing source features: `if` is contextual control, `Error().signal()` fails an invalid demo, `print` is provided by the CLI, object literal method `bitAnd` uses ordinary dynamic dispatch. The `namedAnd` binding explicitly checks that operator `&` and named message `.bitAnd(10)` agree. The `andResult` and `orResult` cases independently prove strict RHS evaluation; the `lazy` case proves `&&` does not evaluate its RHS. No import or capability authorization needed. Do not invent a private bitwise library, Java helper, runtime facility, a syntax test inside executable source that fails lexing, or hidden evaluation-dependent side effects.

### FILE 2 — NEW `protos/examples/numbers/bit-flags-and-masks.protos`

Create a **task-oriented** real-world example of permission flag manipulation and explicit 32-bit masks. Begin with the same full APL Part 5 source notice using the comment style of neighboring `protos/examples/numbers/exact-and-complex.protos`. Then use this **exact Protos program body**:

```protos
// A portable permissions word: set, test, clear and toggle bits.
// Integers are unbounded; bitwise operators are ordinary message sends.
// Expected output: six true lines.

readFlag: 1 << 0
writeFlag: 1 << 1
executeFlag: 1 << 2

permissions: readFlag | writeFlag
canWrite: (permissions & writeFlag) != 0

// Set execute; clear write; toggle read; toggle again to restore.
granted: permissions | executeFlag
withoutWrite: granted & (~writeFlag)
toggledRead: permissions ~ readFlag
restored: toggledRead ~ readFlag

// A 32-bit bit pattern needs an explicit mask; ~0 itself is negative.
mask32: 4294967295
all32: (~0) & mask32

// Extract two packed octets with shift and AND rather than host-width rules.
packed: (255 << 8) | 17
highByte: (packed >> 8) & 255
lowByte: packed & 255

verified:
    canWrite &&
    (permissions == 3) &&
    (granted == 7) &&
    (withoutWrite == 5) &&
    ((withoutWrite & writeFlag) == 0) &&
    (toggledRead == writeFlag) &&
    (restored == permissions) &&
    (all32 == mask32) &&
    (highByte == 255) &&
    (lowByte == 17) &&
    ((-3) >> 1 == -2) &&
    (1 << 64 == 18446744073709551616)

if(!verified) {
    Error().signal()
}

print(canWrite)
print(withoutWrite == 5)
print(toggledRead == writeFlag)
print(restored == permissions)
print((highByte == 255) && (lowByte == 17))
print(verified)
```

No host-dependent word sizes, logical unsigned `>>>`, Boolean truthiness, external commands, crypto dependencies or network authority. Do not register this in the Protos Test Tool conformance manifest. Its intended regression is automatic CLI learning-material discovery.

### FILE 3 — `protos/tutorials/README.md`

Near the introductory progression (after “Each source file focuses on one language mechanism”), add a short passage linking to:

`01-values-and-slots/04-bitwise-operators.protos`

Explain that it demonstrates the new fixed bitwise sugar, strict vs lazy evaluation, unbounded Integer semantics and an ordinary object overriding `bitAnd`; requires only the standalone CLI. Keep the 14-directory progression and HTTP/system-authority paragraphs unchanged. Do not add a 15th directory.

### FILE 4 — `protos/examples/README.md`

In the existing cookbook table, immediately after the `numbers/exact-and-complex.protos` row, add:

`| Set, clear, test and toggle Integer bit flags with an explicit 32-bit mask | \`numbers/bit-flags-and-masks.protos\` |`

Use the exact table style of surrounding rows and an actual relative link or inline code consistent with that table. In the nearby standalone-CLI guidance mention the new example needs no host authority or imports and prints six checked Boolean lines. Do not touch Rosetta Code catalog: no external Rosetta task or publication exists for this source.

### FILE 5 — `docs/guide/05-values-identity-equality-and-collections.md`

Edit only the existing **“### Bitwise operators on Integer”** subsection (present after exact Integer division). Keep its existing six-operator table, sample results, unbounded Integer/shift/Error paragraph, `&&`/`||` contrast and nonlocal `^` statement unchanged.

Immediately after those existing paragraphs, add a small instructional block (not a second normative owner):

- Explain grouping from tighter to looser: unary `~`, arithmetic `* / %` then `+ -`, shifts `<< >>`, `&`, XOR `~`, `|`, then comparisons and equality, followed by Boolean `&&` and `||`. For ambiguous source, recommend parentheses; all added infix levels associate left.
- Give exactly these idiomatic snippets with explanatory comments:
  `1 | 6 & 3       // 3 = 1 | (6 & 3)`
  `12 ~ ~10       // -7: binary XOR of 12 and complement of 10`
  `(1 << 8) | 17  // 273: combine two fields`
  `(~0) & 4294967295 // 4294967295: explicit 32-bit mask`
- Explain the maximal-symbol-run trap in prose/code-span, not a runnable fenced `protos` example: `a ~ ~b` is valid, `a~~b` invalid; `a & -b` valid, `a&-b` invalid. `a << -1` parses but signals the standard Error.
- Explain a non-Integer object can implement `bitAnd` and be used as `object & argument` because lowering is ordinary message dispatch; do not imply all non-Integer builtins support the operations.
- Add two correct relative links, from `docs/guide/`, to `../../protos/tutorials/01-values-and-slots/04-bitwise-operators.protos` and `../../protos/examples/numbers/bit-flags-and-masks.protos`; describe the first as step-by-step tutorial and the second as cookbook permissions/masks.
- Do not alter any other value-family section, original headings, Math/Range/Vector text, or global guide structure.

### FILE 6 — `docs/guide/README.md`

Under “How to use the learning material” or near the “Values, identity, equality, and collections” chapter entry, add one short navigation sentence to the pre-existing guide bitwise subsection and cross-link the new executable tutorial and cookbook example. Use relative links:
- `05-values-identity-equality-and-collections.md#bitwise-operators-on-integer`
- `../../protos/tutorials/01-values-and-slots/04-bitwise-operators.protos`
- `../../protos/examples/numbers/bit-flags-and-masks.protos`

Leave the rest of the guide navigation, historical DOC001 status and release instructions intact.

## Protos-first, validation and file exclusions

The **only** new executable behavior is in two ordinary Protos learning sources, not in Java. Existing `src/test/java/com/guillermomolina/protos/cli/ProtosCliLearningMaterialsTest.java` scans `protos/examples` and `protos/tutorials` recursively for `.protos` files containing `print(`, and runs them unchanged as standalone CLI programs with nonempty stdout/blank stderr/exit code zero. The two new files deliberately contain `print(`, require no authority, and self-check results with `Error().signal()`. **Do not edit this Java test**, duplicate it, or add entries to `protos/tests/conformance/manifest.tsv`, `protos/tools/test/RepositoryCorpusPlans.protos`, or any other registry.

Do not edit `src/`, `spec/`, `protos/lib/`, `protos/tools/`, `pom.xml`, `CHANGELOG.md`, scripts, Makefile, root README, `docs/guide/SOURCE_STYLE.md`, or unrelated tutorial/example files. DOC014 does not introduce a new implementation version or spec revision: the educational material uses already shipped I137 semantics. Preserve the existing `LICENSE.TXT`; exact complete APL Part 5 notice in both new source files. No speculative release or Native Image assertion.

**External precedent packet, already resolved; use solely to avoid misleading prose, no research needed:**
- Lua 5.4 official manual §3.4.2/§3.4.8 uses `& | ~ << >>` and unary `~`, but has finite-width/coercion, zero-filled right shifts and opposite-direction negative counts; never teach those as Protos. Primary authority: `https://www.lua.org/manual/5.4/manual.html`.
- Python official reference (§6.8–6.9) defines right shift by floor division by a power of two and distinct AND > XOR > OR precedence. GraalPy is a Truffle implementation of Python, but Python spells XOR `^` and Protos `^` is non-local return. Authorities: `https://docs.python.org/3/reference/expressions.html` and `https://www.graalvm.org/reference-manual/graalpy/`. No claim is made that GraalPy's implementation internals were inspected.
- Ruby official `Integer` documentation defines symbolic methods (including `Integer#&`, `Integer#^`, `Integer#|`, unary `~`) and sign-extended complement; TruffleRuby follows Ruby language semantics. Protos instead dispatches to the *six exact named message selectors*, not Ruby's symbolic selectors. Authorities: `https://docs.ruby-lang.org/en/master/Integer.html`, `https://www.truffleruby.org/`. This is a semantic contrast, not a claim to have audited TruffleRuby implementation source.
- GraalJS source `oracle/graaljs:graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/binary/JSBitwiseXorNode.java` has `doDouble` via `JSToInt32Node`, a distinct `doBigInt` and `doInteger`. `JSRightShiftNode.java` separately handles JavaScript Number shifts and BigInt. Protos has **neither implicit int32 truncation nor JS Number/BigInt coercion**; never copy those nodes. Browser semantic reference: `https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_operators`.
- Truffle SimpleLanguage source `graalvm/simplelanguage:language/src/main/java/com/oracle/truffle/sl/parser/SLNodeParser.java` uses `createBinary` to assemble left-associative expressions; `language/src/main/java/com/oracle/truffle/sl/nodes/expression/SLAddNode.java` uses `@Specialization` over long/BigInteger. Unlike a new SimpleLanguage operator implementation, Protos I137 already lowers to ordinary `CanonicalSend`; learning material must not suggest extra Truffle DSL nodes.
- Espresso (Java bytecode) and Sulong (LLVM) are not Protos syntax authorities.
- The contrast has been completed using the official Lua/Python/Ruby/JavaScript references and actual repository files from GraalJS and Truffle SimpleLanguage. **Do not browse** these URLs or look up other Truffle sources: every consequence relevant to DOC014 is fixed above. Espresso/Sulong remain outside the Protos syntax authority boundary.

## Completion response from editing agent

After all six files are created/updated, review the exact resulting text for accidental Markdown joins or missing spaces. **Do not execute anything.** Return:
1. The precise list of six changed paths (two new, four modified) and the didactic additions in each.
2. Confirmation that only current, already-shipped Protos bitwise behavior is documented, both new programs contain the complete APL notice and no JS/Java-style width/coercion was introduced.
3. Confirmation that the existing CLI learning-material test will discover both files, without falsely claiming a PASS.
4. Human-only focal and publication validation commands (see companion human executor).
5. Any genuine blocked edit, with exact source/line and no invented alternative.

No investigation report, no alternative proposals, no version bump, no execution, no code architecture change, no Git operations. Complete **one single DOC014 slice**.
