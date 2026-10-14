# PLAT059 — GraalPy detailed AST and bytecode-lowering optimization inventory

**Type:** historical upstream static-source audit; non-normative evidence, not an approved Protos platform decision.  
**Source:** [oracle/graalpython](https://github.com/oracle/graalpython/tree/a099d006438942d54c1856c2cce7b6c2839b0c85) at exact commit `a099d006438942d54c1856c2cce7b6c2839b0c85`.  
**Protos owner:** [PLAT059/#894](https://github.com/guillermomolina/protos/issues/894).  
**Companion:** [GraalJS detailed inventory](PLAT059_GRAALJS_DETAILED_OPTIMIZATION_INVENTORY_2026_10_14.md).  
**Protos baseline reviewed separately:** `guillermomolina/protos@8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff`.

## Audit boundary and architecture

GraalPy contains a **real dedicated AST optimization phase**: `BytecodeDSLCompiler` calls `AstOptimizer.optimize(...)` before `RootNodeCompiler` emits Truffle Bytecode DSL operations. Runtime DSL `@Specialization` / quickening / JIT behavior is **downstream** and not evidence of a static folding operation.

The **57 rows** preserve the previous audit's individually named mechanisms and variants. They are **not 57 standalone optimizer passes**; #27/#29 represent closely related `__debug__` behavior at AST and optimization-level emission, and entries marked `SEMANTIC` are primarily efficient, correctness-preserving implementations of Python control semantics rather than independent constant elimination.

Classification: `FOLD`/ `AST` = pre-bytecode semantic rewriting, `OPTIONAL` = compile-level emission behavior, `LOWER`/ `SCOPE`/ `CALL`/ `CONTROL`/ `SPECIALIZE` = static choice of physical Bytecode DSL operations or representations. Each transformation applies only when its source-level guards and Python runtime contract are satisfied.

**Primary code paths:** [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java); [`BytecodeDSLCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/BytecodeDSLCompiler.java); [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java); [`PBytecodeDSLRootNode.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java). **Representative published test:** [`test_parser.py`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python.test/src/tests/test_parser.py) (`test_optimized_ast`, `test_ast_optimizer_code_generation`, future-annotations and docstring cases).

## 1. Constant binary operations in AstOptimizer

| # | Mechanism | Behavior / boundary | Category | Source |
| ---: | --- | --- | --- | --- |
| 1 | Constant addition | Apply Python addition semantics to admitted constant operands. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 2 | Constant subtraction | Calculate subtraction for admitted constant operands. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 3 | Constant multiplication | Calculate multiplication where the result stays within resource limits. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 4 | Constant true division | Compute admissible constant `/` operations using GraalPy semantic nodes. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 5 | Constant floor division | Compute admissible constant `//` operations. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 6 | Constant remainder | Compute admissible `%`, not all string/bytes formatting cases. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 7 | Constant exponentiation | Compute admitted constant powers with size safeguards. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 8 | Constant left shift | Fold safe shifts with size safeguards. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 9 | Constant right shift | Fold admissible constant right shifts. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 10 | Constant bitwise AND | Calculate admitted `a & b` constants. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 11 | Constant bitwise OR | Calculate admitted `a \| b` constants. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 12 | Constant bitwise XOR | Calculate admitted `a ^ b` constants. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |

## 2. Unary folding

| # | Mechanism | Behavior / boundary | Category | Source |
| ---: | --- | --- | --- | --- |
| 13 | Unary numeric minus | Calculate `-c` at AST optimization when valid. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 14 | Unary numeric plus | Calculate `+c` at AST optimization when valid. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 15 | Bitwise invert | Calculate `~c` at AST optimization when valid. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 16 | Logical not | Calculate `not c` for a constant truth value. | `FOLD` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |

## 3. AST-level collection, iterable and expression rewriting

| # | Mechanism | Behavior / boundary | Category | Source |
| ---: | --- | --- | --- | --- |
| 17 | Tuple of constants | Represent eligible tuple literals as a single constant value in read context. | `FOLD/AST` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 18 | Constant tuple subscript | Calculate admissible static indexing, e.g. `('a','b')[1]`. | `FOLD/AST` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 19 | Constant list membership iterable | Use a constant tuple for an eligible list literal used by `in`. | `AST` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 20 | Constant set membership iterable | Use a frozen set for an eligible set literal used by `in`. | `AST` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 21 | List iterable in a `for` | Lower an eligible list expression as a tuple iterable, avoiding unneeded mutable-list construction. | `AST` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 22 | Constant set iterable in a `for` | Lower eligible constant sets to frozen sets for iteration. | `AST` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 23 | Comprehension iterable normalization | Apply iterable rewrites to eligible comprehension iterator expressions. | `AST` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 24 | Invert identity comparison under `not` | Change `not (a is b)` into `a is not b`. | `AST` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 25 | Invert membership under `not` | Change `not (a in b)` into `a not in b`. | `AST` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 26 | Single-element old-style format rewrite | Rewrite supported `%s` / `%r` / `%a` single-tuple forms to a formatted-expression representation. | `AST` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 27 | Resolve `__debug__` | Use a compile-optimization-level-specific boolean value. | `AST` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |

## 4. `-O` / `-OO` emission-time transformations

| # | Mechanism | Behavior / boundary | Category | Source |
| ---: | --- | --- | --- | --- |
| 28 | Remove `assert` at optimization level ≥1 | Do not emit assertion execution when Python optimization mode requires removal. | `OPTIONAL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 29 | Use `__debug__ == False` at optimization level ≥1 | Compile optimization-level boolean semantics; overlaps #27 but records distinct lowering consequence. | `OPTIONAL` | [`AstOptimizer.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java) |
| 30 | Remove docstrings at level 2 | Omit eligible documentation constants/slots in `-OO` bytecode generation. | `OPTIONAL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |

## 5. Literal pool and constant collection emission

| # | Mechanism | Behavior / boundary | Category | Source |
| ---: | --- | --- | --- | --- |
| 31 | Register/deduplicate constants | Use a constant table/map rather than inlining repeated independent constant-construction work. | `LOWER` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 32 | Narrow machine integer constant representation | Emit a smaller integer carrier when a long constant fits an int where the builder permits it. | `LOWER` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 33 | Typed constant tuple construction | Use typed backing arrays in bytecode DSL operations. | `LOWER` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 34 | Typed constant list construction | Use typed backing arrays in bytecode DSL operations while still creating a fresh mutable list. | `LOWER` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 35 | Select primitive backing type | Choose int, long, boolean, double or object storage for eligible literal collection payload. | `LOWER` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 36 | Avoid re-evaluation of constant elements | Use a prebuilt immutable element payload, but do not reuse a single mutable Python list instance. | `LOWER` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |

## 6. Specialized call and method emission

| # | Mechanism | Behavior / boundary | Category | Source |
| ---: | --- | --- | --- | --- |
| 37 | Fixed-arity positional call sites | Select distinct operations for zero through four positional arguments. | `CALL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 38 | Method call receiver preservation | Emit method lookup and call forms that reuse the correct receiver. | `CALL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 39 | Specialized `super()` call patterns | Recognize eligible `super().x` and `super(T,obj).x` forms; retain dynamic validation. | `CALL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 40 | Starred positional argument expansion | Choose specialized variadic-unpack operators rather than always building generic structures. | `CALL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 41 | Single starred positional argument | Use a direct starred-unpack form for the one-star case. | `CALL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 42 | Keyword argument combination | Choose the appropriate keyword/`**kwargs` preparation and merge forms according to shape. | `CALL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |

## 7. Binding, frame and destructuring representation

| # | Mechanism | Behavior / boundary | Category | Source |
| ---: | --- | --- | --- | --- |
| 43 | Direct local name operation | Generate known local load/store rather than general name resolution. | `SCOPE` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 44 | Closure-cell access | Emit a cell operation for statically classified captured variables. | `SCOPE` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 45 | Separate global lookup | Choose a global/builtin-oriented name operation for known global resolution. | `SCOPE` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 46 | Reuse eligible local slots | Avoid unnecessary scratch locals when a target/local can be used safely. | `SCOPE` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 47 | Clear dead temporary slots | Use load-and-clear forms for temporary lifetimes where valid. | `SCOPE` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 48 | Direct tuple/list destructuring into targets | Avoid intermediate aggregate creation when shape and assignment semantics permit stack-based distribution. | `LOWER` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 49 | Two-local unpack operation | Emit `UnpackToLocals2` for an eligible two-target unpack. | `LOWER` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |

## 8. Control-flow shape and expression result emission

| # | Mechanism | Behavior / boundary | Category | Source |
| ---: | --- | --- | --- | --- |
| 50 | Direct `is None` / `is not None` | Emit `IsNone` / `IsNotNone` rather than general identity dispatch for recognized cases. | `SPECIALIZE` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 51 | Avoid redundant truthiness conversion | Skip a `Yes`/boolean-coercion wrapper where the expression already yields a boolean. | `LOWER` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 52 | Short-circuit `and`/`or` op | Use Bytecode DSL short-circuit operations that preserve Python operand-result semantics. | `CONTROL/SEMANTIC` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 53 | `if` without an `else` | Choose the smaller `IfThen` form instead of an unnecessary `IfThenElse`. | `CONTROL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 54 | Comparison-chain operand retention | Avoid evaluating a middle operand twice while preserving Python comparison chaining. | `CONTROL/SEMANTIC` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 55 | One-element joined string | Skip generic string concatenation/build scaffolding where one formatted component suffices. | `CONTROL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 56 | Try statement without handlers | Avoid a superfluous catch/rethrow wrapper when no handler needs it. | `CONTROL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |
| 57 | Bare except without final unmatched rethrow | Omit an unreachable final handler-miss rethrow when an unconditional except covers it. | `CONTROL` | [`RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java) |

## Safety limits and static/dynamic distinction

- `AstOptimizer` folds **values**, not arbitrary calls: it visits child expressions, checks `ExprTy.Constant`, and invokes the semantic GraalPy arithmetic/operation nodes only for admitted constant patterns. This is not general inter-statement propagation.
- Safety controls include `MAX_INT_SIZE = 128` and `MAX_COLLECTION_SIZE = 256` for **selected potentially explosive** powers/multiplication/shifts/sequence repetitions; they are **not** global limits on Python integer magnitude or collection size. Errors in a speculative static operation are allowed to leave the expression for runtime rather than change its observable behavior.
- Python `list` literals are **fresh mutable objects** even when a constant payload is shared; typed `MakeConstant*List` operations do not turn them into singleton lists.
- Rewriting a list/set used as an iterable is contextual: it must preserve Python's observation/evaluation contract; it is not license to replace every list or set by an immutable value.
- Optimized source metadata/annotations: the optimizer respects `from __future__ import annotations`; optimization level affects `__debug__`, `assert`, and docstring handling separately.
- Node specializations in [`PBytecodeDSLRootNode.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java) include primitive iterators, typed accesses, call caches and `@ForceQuickening`. They are **runtime specialization** (not among the 57 static inventory entries). The `@GenerateBytecode` configuration supports trace/profile `@Instrumentation`, primitive boxing elimination and a distinct uncached interpreter.
- **Search negatives in these phases:** no confirmed general local-variable constant propagation, general constant `if`/boolean folding, loop invariant code motion, CFG-based common subexpression elimination, arbitrary Python function inline, or general loop unrolling. This is bounded source evidence, not a categorical claim about Graal final compilation.

## Implications / restrictions for Protos

GraalPy is a useful reference for early literal operations, conservative resource-safe evaluators, representations of immutable fact analyses, and specialized bytecode emission. Its `+` is **not** automatically equivalent to Protos' ordinary, overridable message `+`. A local with statically known `1` does not prove that dispatch of `x + 2` may be replaced by `3`. Retain D013 selection/overrides, D179 binding presence and capture, Context/Prelude authority, numeric overflow and mixed-family rules, ReturnHome/Task suspension and NORMAL/DEBUG semantics. [Ratified PLAT058](https://github.com/guillermomolina/protos/issues/891) additionally requires **independent handwritten Bytecode DSL Builder oracle** comparisons of full instructions, constant operands and root-group topology.

## Verification and limits

This is a **static-source inventory** based on selected compiler and execution source. No GraalPy test/build execution, complete upstream runtime audit, measured Truffle phase attribution, benchmark, Protos implementation or ratification is asserted here. Where the lowering entry is an efficient implementation of Python syntax, it is explicitly categorized rather than being misrepresented as a removed computation.
