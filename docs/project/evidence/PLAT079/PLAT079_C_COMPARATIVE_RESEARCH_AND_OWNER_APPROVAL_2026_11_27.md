# PLAT079-C — Comparative research and owner approval evidence

**Date:** 2026-11-27. **State:** direction selected, research/contract not complete.
**Issue:** [PLAT079 #1206](https://github.com/guillermomolina/protos/issues/1206).
**Owner approval:** "APRUEBO PLAT079-C" in the active interaction after the seven-candidate proposal and explicit incomplete-research disclosure. No public chat permalink is available; this is a bounded record of the actual owner statement, not a fabricated GitHub owner comment or verbatim chat transcript.

## Revision-bound observations

- Protos [facde992e6f6d9f11ae1a3ed64e9140a70976e50](https://github.com/guillermomolina/protos/tree/facde992e6f6d9f11ae1a3ed64e9140a70976e50).
- Website [67d8e311452b98fa997d738cf5852c312e539e01](https://github.com/guillermomolina/protos-website/tree/67d8e311452b98fa997d738cf5852c312e539e01).
- Website lock pins Protos sources at 9c56aba2c8b84a9c44440cb361db85b53fcf30b4 and [extension grammar at 82ceb83872b3ef25362b4b817352b088d21cee59](https://github.com/guillermomolina/protos-vscode-extension/blob/82ceb83872b3ef25362b4b817352b088d21cee59/syntaxes/protos.tmLanguage.json); grammar blob 7bb5fcce1fa11abccdfd20a9fb282dff60de40ce. These are distinct from the inspected Protos main revision, not deployment equivalence.
- [Research checkpoint](https://github.com/guillermomolina/protos/issues/1206#issuecomment-7208663520) and [corpus checkpoint](https://github.com/guillermomolina/protos/issues/1206#issuecomment-7208676131).

Inspected Protos paths include root AGENTS.md; AGENTS.work/{DESIGN,REFERENCE,COORDINATION}.md; src/AGENTS.md, protos/AGENTS.md and spec/AGENTS.md; spec/PROTOS_GRAMMAR.md (targeted lexical/contextual sections), spec/PROTOS_LANGUAGE_SPEC.md and spec/semantics/VALUES_AND_COLLECTIONS.md (targeted String sections); docs/guide/library/{regex,text}.md; protos/lib/regex/regex/module.protos; tools/generate_unicode17_data.py; src/main/java/com/guillermomolina/protos/lexer/{ProtosLexer,Token,TokenType,TokenOccurrence,UnicodeXid,UnicodeData17}.java; source/SourceSpan.java; parser/ProtosParser.java and parser/ProtosParserSourceFacts.java search results; analysis/ProtosSourceLayoutView.java; execution/host/tooling/{ProtosSourceLayoutToolBridge,ProtosFormatterToolBootstrap}.java; protos/tools/formatter/main/module.protos; and lexical tests ProtosLexerTest, ProtosLexerSourceSpanTest, UnicodeXid17ConformanceTest and UnicodeNfc17ConformanceTest. The full applicable normative audit is not claimed complete.

ProtosLexer is handwritten, independent of Truffle, position-independent, with strict tokenization and opt-in trivia. Java raw spans are UTF-16 offsets; guest String indices are Unicode scalars. ProtosSourceLayoutView calls a complete parser, and the formatter-specific guest bridge projects raw token type/spelling, comments and structural forms, not a general tolerant lexer API. This is a credible reuse alternative, not already an ordinary guest lexer. Contextual match/if/while/for/until/return remain identifiers at lexing; only the parser resolves contextual constructs. Unicode 17 XID and NFC cannot be replaced with regex word categories. Existing generated Unicode tables and conformance tests provide a data source; exact guest reuse strategy is open. D187 public regex dialect rejects lookaround/backreferences and does not advertise XID properties. String.each may avoid nonconstant scalar indexing costs; measure sequential traversal and materialization rather than assuming linear wall-clock performance.

Website paths inspected: AGENTS.md, package.json, astro.config.mjs, source locks, scripts/{prepare-vscode-grammar,materialize-protos-browser-grammar}.mjs, src/scripts/live-docs-{highlight,markdown}.js, tests/live-docs-highlight.test.mjs. The live viewer uses Shiki 4.4.3 JavaScript engine, github-dark, 32,768 UTF-16 unit limit per block, 64-block selection limit, inert text nodes, literal hex color check and exact textContent comparison before replacement. The locked TextMate grammar contains 18 lookaround assertions and no numeric backreferences detected. Existing AGENTS requires the locked extension grammar and no website fork. The selected direction proposes prospective reconciliation for Protos rendering only.

Complete non-truncated Git tree: 91 Markdown blobs under the inspected roots (including historical/AGENTS files), 1,025 .protos files under lib/tutorials/examples/tools/tests. These are tree counts, not website admissions. Eight representative Markdown files, scanned for top-level fence markers, contain 161 protos, 226 js and 37 ebnf fences; the values/collections guide has 68 protos fences. Some normative examples label Protos as js. This is not a CommonMark AST inventory. Full corpus attempt exceeded connector per-script call cap; batches remain possible, so coverage is pending, not inaccessible. No Shiki visual differential run.

## Prior art

Source-verified Truffle references:
- [GraalPy Tokenizer.java](https://github.com/oracle/graalpython/blob/d2b24d62615009e2413acc6ca3f15a537458af88/graalpython/com.oracle.graal.python.pegparser/src/com/oracle/graal/python/pegparser/tokenizer/Tokenizer.java) and [PTokenizerIter.java](https://github.com/oracle/graalpython/blob/d2b24d62615009e2413acc6ca3f15a537458af88/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/tokenize/PTokenizerIter.java): EXTRA_TOKENS, getNextToken/getTokenCodePoints with @TruffleBoundary; [TokenizerIterBuiltins.java](https://github.com/oracle/graalpython/blob/d2b24d62615009e2413acc6ca3f15a537458af88/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/tokenize/TokenizerIterBuiltins.java): NextNode.next @Specialization, @Slot tp_iternext; getCodePoints sequential TruffleString iterator; [tokenize.py](https://github.com/oracle/graalpython/blob/d2b24d62615009e2413acc6ca3f15a537458af88/graalpython/lib-python/3/tokenize.py) delegates to tokenizer. B is therefore credible, not approved.
- [GraalJS Lexer.java](https://github.com/oracle/graaljs/blob/7bf9a14ed8eda1ac8070b61518922242712ec11f/graal-js/src/com.oracle.js.parser/src/com/oracle/js/parser/Lexer.java): handwritten scanner, source/token stream and state separation.
- [Apple Pkl lexer](https://github.com/apple/pkl/blob/c254bfbabeaed4d013009a7a3609a7f09d3eafb8/pkl-parser/src/main/java/org/pkl/parser/Lexer.java) and [parser](https://github.com/apple/pkl/blob/c254bfbabeaed4d013009a7a3609a7f09d3eafb8/pkl-parser/src/main/java/org/pkl/parser/Parser.java): independent module, spans/snapshots and separate module/expression/REPL parse operations.
- [SimpleLanguage grammar](https://github.com/oracle/graal/blob/4945f9d84667430d17a1b76f59483cf9dd0efc73/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/parser/SimpleLanguage.g4) (ANTLR); [Espresso classfile parser](https://github.com/oracle/graal/blob/4945f9d84667430d17a1b76f59483cf9dd0efc73/espresso-shared/src/com.oracle.truffle.espresso.classfile/src/com/oracle/truffle/espresso/classfile/ClassfileParser.java) and [Sulong LLVMScanner](https://github.com/oracle/graal/blob/4945f9d84667430d17a1b76f59483cf9dd0efc73/sulong/projects/com.oracle.truffle.llvm.parser/src/com/oracle/truffle/llvm/parser/scanner/LLVMScanner.java) are binary-input analogues, not textual lexer precedents.

Other comparison families discussed: [TruffleRuby/Prism](https://github.com/oracle/truffleruby), [Go scanner](https://pkg.go.dev/go/scanner) and [bootstrap](https://go.dev/doc/install/source), [Rust compiler](https://github.com/rust-lang/rust), [rust-analyzer](https://github.com/rust-lang/rust-analyzer), [Python tokenize](https://docs.python.org/3/library/tokenize.html), [Pygments](https://pygments.org/docs/), [Tree-sitter](https://tree-sitter.github.io/tree-sitter/) and [Shiki](https://shiki.style/). These links identify comparison systems; the non-pinned survey is directional, not a complete source-equivalence audit. Pygments-style lexer/presentation separation and Go-style guest tooling illustrate different architectural approaches; lossless CST/recovery in editor tooling is a possible future consumer, not a present obligation.

## Candidate matrix (directional, 1–5)

A current browser Shiki; B existing Java lexer reuse from guest; C0 approximate Protos scanner; C authentic guest lexer and separate highlighter; D full guest lexer/parser; E guest TextMate engine; F external TextMate/FFI. Scores below are architectural estimates, not measured outcomes. Confidence medium except existing source observations high and performance/scale estimates medium-low.

| Axis | A | B | C0 | C | D | E | F |
|---|---:|---:|---:|---:|---:|---:|---:|
| Correctness | 3 | 5 | 3 | 4 | 4 | 3 | 4 |
| Protos alignment | 2 | 4 | 3 | 5 | 4 | 2 | 2 |
| Present proportionality | 4 | 5 | 5 | 4 | 1 | 2 | 2 |
| Incremental growth | 2 | 4 | 2 | 5 | 3 | 2 | 2 |
| Future resilience | 2 | 3 | 2 | 5 | 5 | 2 | 2 |
| Scalability | 3 | 4 | 4 | 4 | 3 | 2 | 3 |
| Simplicity | 3 | 4 | 4 | 4 | 2 | 1 | 2 |
| Portability | 3 | 3 | 5 | 5 | 5 | 4 | 2 |
| Runtime/resource cost | 3 | 5 | 4 | 3 | 2 | 2 | 3 |
| Failure/operability | 3 | 4 | 3 | 4 | 3 | 2 | 3 |
| Reversibility/deferral | 4 | 4 | 2 | 5 | 3 | 2 | 3 |
| Evidence maturity | 5 | 4 | 4 | 3 | 2 | 2 | 4 |
| **Total** | **37** | **49** | **41** | **51** | **37** | **26** | **32** |

A is mature and reversible but browser-bound and does not advance INC005. B minimizes grammar drift and execution cost by sharing the current lexer but requires a guest access boundary and keeps Java ownership. C0 is small but risks a third divergent recognizer and a future replacement. C advances INC005 and serves WEB016 without host infrastructure, but duplicates lexical implementation until future cutover; maturity and performance are unmeasured. D pays for an unneeded parser and recovery/AST maintenance now. E builds a general grammar engine rather than the language frontend; F adds engine and deployment dependencies. Red flags: D overengineering, C0 future rework and E/F disproportionate machinery. Strongest counterargument against C is B's single implementation and existing GraalPy precedent. C was recommended because the owner wants the real lexer as a self-hosting increment, not merely colors.

## Gates and adversarial considerations

- Use strict lexical acceptance and separate presentation recovery; test maximal symbolic runs, exact six reserved words, Unicode 17 XID/NFC, mixed CR/LF/CRLF, comments, escaped and triple quoted strings, raw spelling vs decoded values, malformed fragments, EOF, monotonic complete source coverage and UTF-16/scalar conversion.
- Never evaluate the input. Preserve exact source data and escape text through D267; specify unavoidable HTML representation differences (e.g. NUL, physical newline normalization) and bounded safe plain-text fallback. Reconcile D266/I183 code fences, D267/I179 exact safe serializer and WEB016 current TextMate authority/limits without relabeling arbitrary JS.
- Compare existing Java lexical tests, Unicode official conformance fixtures, negative fixtures and the full applicable .protos and Markdown corpus. Independently arbitrate any Java/spec mismatch by normative authority.
- Measure cold/warm JVM and installed Native amd64, source sizes, per-operation allocation, throughput, growth on adversarial input, JIT tiers and distribution footprint where relevant. No claim that a guest implementation is currently fast, compiled or feasible. No new Java infrastructure.
- Use local operation state, no cross-Context sharing, global mutable cache, new scheduler or speculative AST persistence. Future Actors, cancellation and unwind must compose with existing mechanisms; verify actual workload behavior. Future LSP incremental parsing, compiler/backend migration, alternate runtime and bootstrap can add layers without imposing their cost now; exact interfaces and costs remain unmeasured.
- The current smallest WEB016-only solution could be B or C0. C earns its larger scope via the INC005 goal; deferring parser avoids two complete parser implementations now. If C later proves too costly or grammar drift is unacceptable, existing Java P0 remains a reversible producer. Do not select an unmeasured runtime or public API on the basis of scores.

## Status

GITHUB021 direction consistency reviewed in the [selected direction](../../decisions/platform/PLAT079_C_PROTOS_NATIVE_LEXER_DIRECTION_SELECTED_2026_11_27.md). GITHUB010 final completeness, exact contract and executable gates remain OPEN. No product edits, builds, tests, benchmarks, visual comparison, Native Image or CI evidence were produced. No I-family issue allocated yet: next work is the remaining PLAT079 investigation, not production implementation. No priority/assignee/milestone or native hierarchy inferred. No repository-content/specification change in Protos or website.
