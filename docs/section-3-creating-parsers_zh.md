---
title: 创建解析器
permalink: creating-parsers
---

# 创建解析器

开发 Tree-sitter 语法可能有一个困难的学习曲线，但一旦你掌握了它，它就会很有趣，甚至像禅宗一样。 本文档将帮助您入门并建立有用的思维模型。

## Getting Started

### Dependencies

为了开发 Tree-sitter 解析器，您需要安装两个依赖项：

* **Node.js** - Tree-sitter 语法是用 JavaScript 编写的，Tree-sitter 使用 [Node.js][node.js] 来解释 JavaScript 文件。 它要求 `node` 命令位于 [`PATH`][path-env] 中的目录之一中。 您需要 Node.js 6.0 或更高版本。
* **C 编译器** - Tree-sitter 创建的解析器是用 C 语言编写的。为了运行和测试这些解析器，您需要使用`tree-sitter parse` 或 `tree-sitter test`命令，必须安装一个 C/C++ 编译器。Tree-sitter 会尝试在每个平台的标准位置查找这些编译器。

### 安装

要创建 Tree-sitter 解析器，您需要使用 [`tree-sitter` CLI][tree-sitter-cli]。 您可以通过几种不同的方式安装 CLI：

* 使用 Rust 包管理器 [`cargo`][cargo] 从源代码构建 `tree-sitter-cli` [Rust crate][crate]。 这适用于任何平台。 有关更多信息，请参阅[贡献文档](./contributing#developing-tree-sitter)。
* 使用 Node 包管理器 [`npm`][npm] 安装 `tree-sitter-cli` [Node.js 模块][node-module]。 这种方法速度很快，但仅适用于某些平台，因为它依赖于预先构建的二进制文件。
* 从 [最新的 GitHub 发布][releases] 页面下载适用于您的平台的二进制文件，并将其放入 `PATH` 环境变量中的一个目录中。

### 项目设置

首选的命名约定是将解析器仓库命名为 "tree-sitter-"，后跟语言的名称。

```sh
mkdir tree-sitter-${YOUR_LANGUAGE_NAME}
cd tree-sitter-${YOUR_LANGUAGE_NAME}
```

您可以使用 `npm` 命令行工具创建一个描述项目的 `package.json` 文件，从而使您的解析器可以从 Node.js 使用。

```sh
# 这将提示您输入信息
npm init

# 这将安装一个小模块，使您的解析器可以从 Node.js 使用
npm install --save nan

# 这将安装 Tree-sitter CLI
npm install --save-dev tree-sitter-cli
```

最后一个命令会将 CLI 安装到工作目录中的 node_modules 文件夹中。一个名为 tree-sitter 的可执行程序会被创建在 `node_modules/.bin/` 里面。您可能希望遵循 Node.js 的惯例，将该文件夹添加到您的 `PATH` 中，这样在该目录工作时就可以轻松运行这个程序。

```sh
# 在您的 shell 配置文件中添加以下行
export PATH=$PATH:./node_modules/.bin
```

安装 CLI 后，创建一个名为 `grammar.js` 的文件，并包含以下内容：

```js
module.exports = grammar({
  name: 'YOUR_LANGUAGE_NAME',

  rules: {
    // TODO: 添加实际的语法规则
    source_file: $ => 'hello'
  }
});
```

然后运行以下命令：

```sh
tree-sitter generate
```

这将生成解析该简单语言所需的 C 代码，以及编译和加载该原生解析器作为 Node.js 模块所需的几个文件。

您可以通过创建一个内容为 "hello" 的源文件并解析它来测试这个解析器：

```sh
echo 'hello' > example-file
tree-sitter parse example-file
```

或者，在 Windows PowerShell 中：

```pwsh
"hello" | Out-File example-file -Encoding utf8
tree-sitter parse example-file
```

这应该会打印以下内容：

```text
(source_file [0, 0] - [1, 0])
```

现在您有了一个可用的解析器。

## 工具概述

让我们详细了解 tree-sitter 命令行工具的所有功能。

### 命令：generate

最重要的命令是 tree-sitter generate。此命令会读取当前工作目录中的 grammar.js 文件，并创建一个名为 src/parser.c 的文件，该文件实现了解析器。在对语法进行更改后，只需再次运行 tree-sitter generate 即可。

第一次运行 tree-sitter generate 时，它还会生成其他几个文件，以支持以下语言的绑定：

#### C/C++

* `Makefile` - 这个文件告诉 `make` 如何编译您的语言。
* `bindings/c/tree-sitter-language.h` - 这个文件提供了您的语言的 C 接口。
* `bindings/c/tree-sitter-language.pc` - 这个文件提供了关于您的语言的 C 库的 pkg-config 元数据。
* `src/tree_sitter/parser.h` - 这个文件提供了一些基本的 C 定义，这些定义在生成的 parser.c 文件中使用。
* `src/tree_sitter/alloc.h` - 这个文件提供了一些内存分配宏，如果您有外部扫描器，则在其中使用。
* `src/tree_sitter/array.h` - 这个文件提供了一些数组宏，如果您有外部扫描器，则在其中使用。

#### Go

* `bindings/go/binding.go` - 这个文件将您的语言包装在一个 Go 模块中。
* `bindings/go/binding_test.go` - 这个文件包含了对 Go 包的测试。

#### Node

* `binding.gyp` - 这个文件告诉 Node.js 如何编译您的语言。
* `bindings/node/index.js` - 这是 Node.js 使用您的语言时初始加载的文件。
* `bindings/node/binding.cc` - 这个文件将您的语言包装在一个用于 Node.js 的 JavaScript 模块中。

#### Python

* `pyproject.toml` - 这个文件是 Python 包的清单。
* `setup.py` - 这个文件告诉 Python 如何编译您的语言。
* `bindings/python/binding.c` - 这个文件将您的语言包装在一个 Python 模块中。
* `bindings/python/tree_sitter_language/__init__.py` - 这个文件告诉 Python 如何加载您的语言。
* `bindings/python/tree_sitter_language/__init__.pyi` - 这个文件在 Python 中使用您的解析器时提供类型提示。
* `bindings/python/tree_sitter_language/py.typed` - 这个文件在 Python 中使用您的解析器时提供类型提示。
  
#### Rust

* `Cargo.toml` - 这个文件是 Rust 包的清单。
* `bindings/rust/lib.rs` - 这个文件在 Rust 中使用时将您的语言包装在一个 Rust crate 中。
* `bindings/rust/build.rs` - 这个文件包装了 Rust crate 的构建过程。

#### Swift

* `Package.swift` - 这个文件告诉 Swift 如何编译您的语言。
* `bindings/swift/TreeSitterLanguage/language.h` - 这个文件在 Swift 中使用时将您的语言包装在一个 Swift 模块中。

如果您的语法中存在歧义或局部歧义，Tree-sitter 会在解析器生成过程中检测到，并会以 `Unresolved conflict` 错误消息退出。有关这些错误的更多信息，请参见下文。

### 命令：`build`

`build` 命令将您的解析器编译成一个可动态加载的库，可以是共享对象（`.so`、`.dylib` 或 `.dll`）或 WASM 模块。

您可以通过 `CC` 环境变量更改编译器可执行文件，并通过 `CFLAGS` 添加额外的标志。对于 macOS 或 iOS，您可以分别设置 `MACOSX_DEPLOYMENT_TARGET` 或 `IPHONEOS_DEPLOYMENT_TARGET` 以定义最低支持版本。

您可以使用 --wasm/-w 标志指定是否将其编译为 wasm 模块，还可以使用 --docker/-d 标志选择使用 docker 或 podman 提供 emscripten，从而无需在本地机器上安装 emscripten。

您可以使用 `--output/-o` 标志指定共享对象文件（native或 WASM）的输出位置，该标志接受绝对路径或相对路径。请注意，如果不提供此标志，CLI 将尝试根据父目录确定语言名称（例如在 `tree-sitter-javascript` 目录中构建将解析为 `javascript`），以便用于输出文件。如果无法确定，它将默认使用 `parser`，从而在当前工作目录中生成 `parser.so` 或 `parser.wasm`。

最后，您还可以指定实际语法目录的路径，以防您当前不在该目录中。这可以通过在命令中提供路径作为第一个*位置*参数来完成。

Example:

```sh
tree-sitter build --wasm --output ./build/parser.wasm tree-sitter-javascript
```

请注意，`tree-sitter-javascript` 参数是第一个位置参数。

### 命令： `test`

`tree-sitter test` 命令使您能够轻松测试解析器是否正常工作。

对于添加到语法中的每个规则，您应首先创建一个*测试*，描述解析该规则时语法树应该如何。 这些测试是使用解析器根文件夹中的 `test/corpus/` 目录内的特定格式的文本文件编写的。

例如，您可能有一个名为 `test/corpus/statements.txt` 的文件，其中包含如下内容的条目：

```text
==================
Return statements
==================

func x() int {
  return 1;
}

---

(source_file
  (function_definition
    (identifier)
    (parameter_list)
    (primitive_type)
    (block
      (return_statement (number)))))
```

* The **name** of each test is written between two lines containing only `=` (equal sign) characters.
* 每个测试的**名称**写在两行仅包含 `=`（等号）字符之间。
* Then the **输入源代码** is written, followed by a line containing three or more `-` (dash) characters.
* 然后是**输入源代码**，接着是一行包含三个或更多 `-`（破折号）字符的行。
* 接着是**预期的输出语法树**，以 [S-表达式][s-exp] 的形式编写。S-表达式中的空白字符的确切位置无关紧要，但理想情况下，语法树应该易于阅读。注意，S-表达式不会显示语法节点（如 `func`、`(` 和 `;`），这些节点在语法中表示为字符串和正则表达式。它只显示*命名*的节点，如[解析器使用页面][named-vs-anonymous-nodes-section]中的此部分所述。

  预期输出部分还可以*可选地*显示与每个子节点关联的[字段名称][field-names-section]。要在测试中包含字段名称，您可以在 S-表达式中的节点之前写上字段名称，后跟一个冒号：
  
```text
(source_file
  (function_definition
    name: (identifier)
    parameters: (parameter_list)
    result: (primitive_type)
    body: (block
      (return_statement (number)))))
```

* 如果您的语言语法与 `===` 和 `---` 测试分隔符冲突，您可以选择添加任意相同的后缀（在下面的示例中为 `|||`）以消除歧义：

```text
==================|||
Basic module
==================|||

---- MODULE Test ----
increment(n) == n + 1
====

---|||

(source_file
  (module (identifier)
    (operator (identifier)
      (parameter_list (identifier))
      (plus (identifier_ref) (number)))))
```

这些测试非常重要。它们作为解析器的 API 文档，并且每次更改语法时都可以运行这些测试，以验证一切是否仍能正确解析。

默认情况下，`tree-sitter test` 命令会运行 `test/corpus/` 文件夹中的所有测试。要运行特定的测试，可以使用 `-f` 标志：

```sh
tree-sitter test -f 'Return statements'
```

The recommendation is to be comprehensive in adding tests. If it's a visible node, add it to a test file in your `test/corpus` directory. It's typically a good idea to test all of the permutations of each language construct. This increases test coverage, but doubly acquaints readers with a way to examine expected outputs and understand the "edges" of a language.
建议是尽可能全面地添加测试。如果它是一个可见的节点，请将其添加到 `test/corpus` 目录中的测试文件中。通常，测试每种语言结构的所有排列组合是个好主意。这不仅增加了测试覆盖率，还让读者更熟悉如何检查预期输出并了解语言的"边界"。

#### Attributes

Tests can be annotated with a few `attributes`. Attributes must be put in the header, below the test name, and start with a `:`.
A couple of attributes also take in a parameter, which require the use of parenthesis.

**Note**: If you'd like to supply in multiple parameters, e.g. to run tests on multiple platforms or to test multiple languages, you can repeat the attribute on a new line.

The following attributes are available:

- `:skip` — This attribute will skip the test when running `tree-sitter test`.
  This is useful when you want to temporarily disable running a test without deleting it.
- `:error` — This attribute will assert that the parse tree contains an error. It's useful to just validate that a certain input is invalid without displaying the whole parse tree, as such you should omit the parse tree below the `---` line.
- `:fail-fast` — This attribute will stop the testing additional tests if the test marked with this attribute fails.
- `:language(LANG)` — This attribute will run the tests using the parser for the specified language. This is useful for multi-parser repos, such as XML and DTD, or Typescript and TSX. The default parser will be the first entry in the `tree-sitter` field in the root `package.json`, so having a way to pick a second or even third parser is useful.
- `:platform(PLATFORM)` — This attribute specifies the platform on which the test should run. It is useful to test platform-specific behavior (e.g. Windows newlines are different from Unix). This attribute must match up with Rust's [`std::env::consts::OS`](https://doc.rust-lang.org/std/env/consts/constant.OS.html).

Examples using attributes:

```text
=========================
Test that will be skipped
:skip
=========================

int main() {}

-------------------------

====================================
Test that will run on Linux or macOS

:platform(linux)
:platform(macos)
====================================

int main() {}

------------------------------------

========================================================================
Test that expects an error, and will fail fast if there's no parse error
:fail-fast
:error
========================================================================

int main ( {}

------------------------------------------------------------------------

=================================================
Test that will parse with both Typescript and TSX
:language(typescript)
:language(tsx)
=================================================

console.log('Hello, world!');

-------------------------------------------------
```

#### Automatic Compilation

You might notice that the first time you run `tree-sitter test` after regenerating your parser, it takes some extra time. This is because Tree-sitter automatically compiles your C code into a dynamically-loadable library. It recompiles your parser as-needed whenever you update it by re-running `tree-sitter generate`.

#### Syntax Highlighting Tests

The `tree-sitter test` command will *also* run any syntax highlighting tests in the `test/highlight` folder, if it exists. For more information about syntax highlighting tests, see [the syntax highlighting page][syntax-highlighting-tests].

### Command: `parse`

You can run your parser on an arbitrary file using `tree-sitter parse`. This will print the resulting the syntax tree, including nodes' ranges and field names, like this:

```text
(source_file [0, 0] - [3, 0]
  (function_declaration [0, 0] - [2, 1]
    name: (identifier [0, 5] - [0, 9])
    parameters: (parameter_list [0, 9] - [0, 11])
    result: (type_identifier [0, 12] - [0, 15])
    body: (block [0, 16] - [2, 1]
      (return_statement [1, 2] - [1, 10]
        (expression_list [1, 9] - [1, 10]
          (int_literal [1, 9] - [1, 10]))))))
```

You can pass any number of file paths and glob patterns to `tree-sitter parse`, and it will parse all of the given files. The command will exit with a non-zero status code if any parse errors occurred. You can also prevent the syntax trees from being printed using the `--quiet` flag. Additionally, the `--stat` flag prints out aggregated parse success/failure information for all processed files. This makes `tree-sitter parse` usable as a secondary testing strategy: you can check that a large number of files parse without error:

```sh
tree-sitter parse 'examples/**/*.go' --quiet --stat
```

### Command: `highlight`

You can run syntax highlighting on an arbitrary file using `tree-sitter highlight`. This can either output colors directly to your terminal using ansi escape codes, or produce HTML (if the `--html` flag is passed). For more information, see [the syntax highlighting page][syntax-highlighting].

### The Grammar DSL

The following is a complete list of built-in functions you can use in your `grammar.js` to define rules. Use-cases for some of these functions will be explained in more detail in later sections.

* **Symbols (the `$` object)** - Every grammar rule is written as a JavaScript function that takes a parameter conventionally called `$`. The syntax `$.identifier` is how you refer to another grammar symbol within a rule. Names starting with `$.MISSING` or `$.UNEXPECTED` should be avoided as they have special meaning for the `tree-sitter test` command.
* **String and Regex literals** - The terminal symbols in a grammar are described using JavaScript strings and regular expressions. Of course during parsing, Tree-sitter does not actually use JavaScript's regex engine to evaluate these regexes; it generates its own regex-matching logic as part of each parser. Regex literals are just used as a convenient way of writing regular expressions in your grammar.
* **Regex Limitations** - Currently, only a subset of the Regex engine is actually
supported. This is due to certain features like lookahead and lookaround assertions
not feasible to use in an LR(1) grammar, as well as certain flags being unnecessary
for tree-sitter. However, plenty of features are supported by default:

  * Character classes
  * Character ranges
  * Character sets
  * Quantifiers
  * Alternation
  * Grouping
  * Unicode character escapes
  * Unicode property escapes

* **Sequences : `seq(rule1, rule2, ...)`** - This function creates a rule that matches any number of other rules, one after another. It is analogous to simply writing multiple symbols next to each other in [EBNF notation][ebnf].
* **Alternatives : `choice(rule1, rule2, ...)`** - This function creates a rule that matches *one* of a set of possible rules. The order of the arguments does not matter. This is analogous to the `|` (pipe) operator in EBNF notation.
* **Repetitions : `repeat(rule)`** - This function creates a rule that matches *zero-or-more* occurrences of a given rule. It is analogous to the `{x}` (curly brace) syntax in EBNF notation.
* **Repetitions : `repeat1(rule)`** - This function creates a rule that matches *one-or-more* occurrences of a given rule. The previous `repeat` rule is implemented in terms of `repeat1` but is included because it is very commonly used.
* **Options : `optional(rule)`** - This function creates a rule that matches *zero or one* occurrence of a given rule. It is analogous to the `[x]` (square bracket) syntax in EBNF notation.
* **Precedence : `prec(number, rule)`** - This function marks the given rule with a numerical precedence which will be used to resolve [*LR(1) Conflicts*][lr-conflict] at parser-generation time. When two rules overlap in a way that represents either a true ambiguity or a *local* ambiguity given one token of lookahead, Tree-sitter will try to resolve the conflict by matching the rule with the higher precedence. The default precedence of all rules is zero. This works similarly to the [precedence directives][yacc-prec] in Yacc grammars.
* **Left Associativity : `prec.left([number], rule)`** - This function marks the given rule as left-associative (and optionally applies a numerical precedence). When an LR(1) conflict arises in which all of the rules have the same numerical precedence, Tree-sitter will consult the rules' associativity. If there is a left-associative rule, Tree-sitter will prefer matching a rule that ends *earlier*. This works similarly to [associativity directives][yacc-prec] in Yacc grammars.
* **Right Associativity : `prec.right([number], rule)`** - This function is like `prec.left`, but it instructs Tree-sitter to prefer matching a rule that ends *later*.
* **Dynamic Precedence : `prec.dynamic(number, rule)`** - This function is similar to `prec`, but the given numerical precedence is applied at *runtime* instead of at parser generation time. This is only necessary when handling a conflict dynamically using the `conflicts` field in the grammar, and when there is a genuine *ambiguity*: multiple rules correctly match a given piece of code. In that event, Tree-sitter compares the total dynamic precedence associated with each rule, and selects the one with the highest total. This is similar to [dynamic precedence directives][bison-dprec] in Bison grammars.
* **Tokens : `token(rule)`** - This function marks the given rule as producing only
a single token. Tree-sitter's default is to treat each String or RegExp literal
in the grammar as a separate token. Each token is matched separately by the lexer
and returned as its own leaf node in the tree. The `token` function allows you to
express a complex rule using the functions described above (rather than as a single
regular expression) but still have Tree-sitter treat it as a single token.
The token function will only accept terminal rules, so `token($.foo)` will not work.
You can think of it as a shortcut for squashing complex rules of strings or regexes
down to a single token.
* **Immediate Tokens : `token.immediate(rule)`** - Usually, whitespace (and any other extras, such as comments) is optional before each token. This function means that the token will only match if there is no whitespace.
* **Aliases : `alias(rule, name)`** - This function causes the given rule to *appear* with an alternative name in the syntax tree. If `name` is a *symbol*, as in `alias($.foo, $.bar)`, then the aliased rule will *appear* as a [named node][named-vs-anonymous-nodes-section] called `bar`. And if `name` is a *string literal*, as in `alias($.foo, 'bar')`, then the aliased rule will appear as an [anonymous node][named-vs-anonymous-nodes-section], as if the rule had been written as the simple string.
* **Field Names : `field(name, rule)`** - This function assigns a *field name* to the child node(s) matched by the given rule. In the resulting syntax tree, you can then use that field name to access specific children.

In addition to the `name` and `rules` fields, grammars have a few other optional public fields that influence the behavior of the parser.

* **`extras`** - an array of tokens that may appear *anywhere* in the language. This is often used for whitespace and comments. The default value of `extras` is to accept whitespace. To control whitespace explicitly, specify `extras: $ => []` in your grammar.
* **`inline`** - an array of rule names that should be automatically *removed* from the grammar by replacing all of their usages with a copy of their definition. This is useful for rules that are used in multiple places but for which you *don't* want to create syntax tree nodes at runtime.
* **`conflicts`** - an array of arrays of rule names. Each inner array represents a set of rules that's involved in an *LR(1) conflict* that is *intended to exist* in the grammar. When these conflicts occur at runtime, Tree-sitter will use the GLR algorithm to explore all of the possible interpretations. If *multiple* parses end up succeeding, Tree-sitter will pick the subtree whose corresponding rule has the highest total *dynamic precedence*.
* **`externals`** - an array of token names which can be returned by an [*external scanner*](#external-scanners). External scanners allow you to write custom C code which runs during the lexing process in order to handle lexical rules (e.g. Python's indentation tokens) that cannot be described by regular expressions.
* **`precedences`** - an array of array of strings, where each array of strings defines named precedence levels in descending order. These names can be used in the `prec` functions to define precedence relative only to other names in the array, rather than globally. Can only be used with parse precedence, not lexical precedence.
* **`word`** - the name of a token that will match keywords for the purpose of the [keyword extraction](#keyword-extraction) optimization.
* **`supertypes`** an array of hidden rule names which should be considered to be 'supertypes' in the generated [*node types* file][static-node-types].

## Writing the Grammar

Writing a grammar requires creativity. There are an infinite number of CFGs (context-free grammars) that can be used to describe any given language. In order to produce a good Tree-sitter parser, you need to create a grammar with two important properties:

1. **An intuitive structure** - Tree-sitter's output is a [concrete syntax tree][cst]; each node in the tree corresponds directly to a [terminal or non-terminal symbol][non-terminal] in the grammar. So in order to produce an easy-to-analyze tree, there should be a direct correspondence between the symbols in your grammar and the recognizable constructs in the language. This might seem obvious, but it is very different from the way that context-free grammars are often written in contexts like [language specifications][language-spec] or [Yacc][yacc]/[Bison][bison] parsers.

2. **A close adherence to LR(1)** - Tree-sitter is based on the [GLR parsing][glr-parsing] algorithm. This means that while it can handle any context-free grammar, it works most efficiently with a class of context-free grammars called [LR(1) Grammars][lr-grammars]. In this respect, Tree-sitter's grammars are similar to (but less restrictive than) [Yacc][yacc] and [Bison][bison] grammars, but *different* from [ANTLR grammars][antlr], [Parsing Expression Grammars][peg], or the [ambiguous grammars][ambiguous-grammar] commonly used in language specifications.

It's unlikely that you'll be able to satisfy these two properties just by translating an existing context-free grammar directly into Tree-sitter's grammar format. There are a few kinds of adjustments that are often required. The following sections will explain these adjustments in more depth.

### The First Few Rules

It's usually a good idea to find a formal specification for the language you're trying to parse. This specification will most likely contain a context-free grammar. As you read through the rules of this CFG, you will probably discover a complex and cyclic graph of relationships. It might be unclear how you should navigate this graph as you define your grammar.

Although languages have very different constructs, their constructs can often be categorized in to similar groups like *Declarations*, *Definitions*, *Statements*, *Expressions*, *Types*, and *Patterns*. In writing your grammar, a good first step is to create just enough structure to include all of these basic *groups* of symbols. For a language like Go, you might start with something like this:

```js
{
  // ...

  rules: {
    source_file: $ => repeat($._definition),

    _definition: $ => choice(
      $.function_definition
      // TODO: other kinds of definitions
    ),

    function_definition: $ => seq(
      'func',
      $.identifier,
      $.parameter_list,
      $._type,
      $.block
    ),

    parameter_list: $ => seq(
      '(',
       // TODO: parameters
      ')'
    ),

    _type: $ => choice(
      'bool'
      // TODO: other kinds of types
    ),

    block: $ => seq(
      '{',
      repeat($._statement),
      '}'
    ),

    _statement: $ => choice(
      $.return_statement
      // TODO: other kinds of statements
    ),

    return_statement: $ => seq(
      'return',
      $._expression,
      ';'
    ),

    _expression: $ => choice(
      $.identifier,
      $.number
      // TODO: other kinds of expressions
    ),

    identifier: $ => /[a-z]+/,

    number: $ => /\d+/
  }
}
```

Some of the details of this grammar will be explained in more depth later on, but if you focus on the `TODO` comments, you can see that the overall strategy is *breadth-first*. Notably, this initial skeleton does not need to directly match an exact subset of the context-free grammar in the language specification. It just needs to touch on the major groupings of rules in as simple and obvious a way as possible.

With this structure in place, you can now freely decide what part of the grammar to flesh out next. For example, you might decide to start with *types*. One-by-one, you could define the rules for writing basic types and composing them into more complex types:

```js
{
  // ...

  _type: $ => choice(
    $.primitive_type,
    $.array_type,
    $.pointer_type
  ),

  primitive_type: $ => choice(
    'bool',
    'int'
  ),

  array_type: $ => seq(
    '[',
    ']',
    $._type
  ),

  pointer_type: $ => seq(
    '*',
    $._type
  )
}
```

After developing the *type* sublanguage a bit further, you might decide to switch to working on *statements* or *expressions* instead. It's often useful to check your progress by trying to parse some real code using `tree-sitter parse`.

**And remember to add tests for each rule in your `test/corpus` folder!**

### Structuring Rules Well

Imagine that you were just starting work on the [Tree-sitter JavaScript parser][tree-sitter-javascript]. Naively, you might try to directly mirror the structure of the [ECMAScript Language Spec][ecmascript-spec]. To illustrate the problem with this approach, consider the following line of code:

```js
return x + y;
```

According to the specification, this line is a `ReturnStatement`, the fragment `x + y` is an `AdditiveExpression`, and `x` and `y` are both `IdentifierReferences`. The relationship between these constructs is captured by a complex series of production rules:

```text
ReturnStatement          ->  'return' Expression
Expression               ->  AssignmentExpression
AssignmentExpression     ->  ConditionalExpression
ConditionalExpression    ->  LogicalORExpression
LogicalORExpression      ->  LogicalANDExpression
LogicalANDExpression     ->  BitwiseORExpression
BitwiseORExpression      ->  BitwiseXORExpression
BitwiseXORExpression     ->  BitwiseANDExpression
BitwiseANDExpression     ->  EqualityExpression
EqualityExpression       ->  RelationalExpression
RelationalExpression     ->  ShiftExpression
ShiftExpression          ->  AdditiveExpression
AdditiveExpression       ->  MultiplicativeExpression
MultiplicativeExpression ->  ExponentiationExpression
ExponentiationExpression ->  UnaryExpression
UnaryExpression          ->  UpdateExpression
UpdateExpression         ->  LeftHandSideExpression
LeftHandSideExpression   ->  NewExpression
NewExpression            ->  MemberExpression
MemberExpression         ->  PrimaryExpression
PrimaryExpression        ->  IdentifierReference
```

The language spec encodes the twenty different precedence levels of JavaScript expressions using twenty levels of indirection between `IdentifierReference` and `Expression`. If we were to create a concrete syntax tree representing this statement according to the language spec, it would have twenty levels of nesting, and it would contain nodes with names like `BitwiseXORExpression`, which are unrelated to the actual code.

### Using Precedence

To produce a readable syntax tree, we'd like to model JavaScript expressions using a much flatter structure like this:

```js
{
  // ...

  _expression: $ => choice(
    $.identifier,
    $.unary_expression,
    $.binary_expression,
    // ...
  ),

  unary_expression: $ => choice(
    seq('-', $._expression),
    seq('!', $._expression),
    // ...
  ),

  binary_expression: $ => choice(
    seq($._expression, '*', $._expression),
    seq($._expression, '+', $._expression),
    // ...
  ),
}
```

Of course, this flat structure is highly ambiguous. If we try to generate a parser, Tree-sitter gives us an error message:

```text
Error: Unresolved conflict for symbol sequence:

  '-'  _expression  •  '*'  …

Possible interpretations:

  1:  '-'  (binary_expression  _expression  •  '*'  _expression)
  2:  (unary_expression  '-'  _expression)  •  '*'  …

Possible resolutions:

  1:  Specify a higher precedence in `binary_expression` than in the other rules.
  2:  Specify a higher precedence in `unary_expression` than in the other rules.
  3:  Specify a left or right associativity in `unary_expression`
  4:  Add a conflict for these rules: `binary_expression` `unary_expression`
```

For an expression like `-a * b`, it's not clear whether the `-` operator applies to the `a * b` or just to the `a`. This is where the `prec` function [described above](#the-grammar-dsl) comes into play. By wrapping a rule with `prec`, we can indicate that certain sequence of symbols should *bind to each other more tightly* than others. For example, the `'-', $._expression` sequence in `unary_expression` should bind more tightly than the `$._expression, '+', $._expression` sequence in `binary_expression`:

```js
{
  // ...

  unary_expression: $ => prec(2, choice(
    seq('-', $._expression),
    seq('!', $._expression),
    // ...
  ))
}
```

### Using Associativity

Applying a higher precedence in `unary_expression` fixes that conflict, but there is still another conflict:

```text
Error: Unresolved conflict for symbol sequence:

  _expression  '*'  _expression  •  '*'  …

Possible interpretations:

  1:  _expression  '*'  (binary_expression  _expression  •  '*'  _expression)
  2:  (binary_expression  _expression  '*'  _expression)  •  '*'  …

Possible resolutions:

  1:  Specify a left or right associativity in `binary_expression`
  2:  Add a conflict for these rules: `binary_expression`
```

For an expression like `a * b * c`, it's not clear whether we mean `a * (b * c)` or `(a * b) * c`. This is where `prec.left` and `prec.right` come into use. We want to select the second interpretation, so we use `prec.left`.

```js
{
  // ...

  binary_expression: $ => choice(
    prec.left(2, seq($._expression, '*', $._expression)),
    prec.left(1, seq($._expression, '+', $._expression)),
    // ...
  ),
}
```

### Hiding Rules

You may have noticed in the above examples that some of the grammar rule name like `_expression` and `_type` began with an underscore. Starting a rule's name with an underscore causes the rule to be *hidden* in the syntax tree. This is useful for rules like `_expression` in the grammars above, which always just wrap a single child node. If these nodes were not hidden, they would add substantial depth and noise to the syntax tree without making it any easier to understand.

### Using Fields

Often, it's easier to analyze a syntax node if you can refer to its children by *name* instead of by their position in an ordered list. Tree-sitter grammars support this using the `field` function. This function allows you to assign unique names to some or all of a node's children:

```js
function_definition: $ => seq(
  'func',
  field('name', $.identifier),
  field('parameters', $.parameter_list),
  field('return_type', $._type),
  field('body', $.block)
)
```

Adding fields like this allows you to retrieve nodes using the [field APIs][field-names-section].

## Lexical Analysis

Tree-sitter's parsing process is divided into two phases: parsing (which is described above) and [lexing][lexing] - the process of grouping individual characters into the language's fundamental *tokens*. There are a few important things to know about how Tree-sitter's lexing works.

### Conflicting Tokens

Grammars often contain multiple tokens that can match the same characters. For example, a grammar might contain the tokens (`"if"` and `/[a-z]+/`). Tree-sitter differentiates between these conflicting tokens in a few ways.

1. **Context-aware Lexing** - Tree-sitter performs lexing on-demand, during the parsing process. At any given position in a source document, the lexer only tries to recognize tokens that are *valid* at that position in the document.

2. **Lexical Precedence** - When the precedence functions described [above](#the-grammar-dsl) are used *within* the `token` function, the given explicit precedence values serve as instructions to the lexer. If there are two valid tokens that match the characters at a given position in the document, Tree-sitter will select the one with the higher precedence.

3. **Match Length** - If multiple valid tokens with the same precedence match the characters at a given position in a document, Tree-sitter will select the token that matches the [longest sequence of characters][longest-match].

4. **Match Specificity** - If there are two valid tokens with the same precedence and which both match the same number of characters, Tree-sitter will prefer a token that is specified in the grammar as a `String` over a token specified as a `RegExp`.

5. **Rule Order** - If none of the above criteria can be used to select one token over another, Tree-sitter will prefer the token that appears earlier in the grammar.

If there is an external scanner it may have [an additional impact](#other-external-scanner-details) over regular tokens defined in the grammar.

### Lexical Precedence vs. Parse Precedence

One common mistake involves not distinguishing *lexical precedence* from *parse precedence*. Parse precedence determines which rule is chosen to interpret a given sequence of tokens. *Lexical precedence* determines which token is chosen to interpret at a given position of text and it is a lower-level operation that is done first. The above list fully captures Tree-sitter's lexical precedence rules, and you will probably refer back to this section of the documentation more often than any other. Most of the time when you really get stuck, you're dealing with a lexical precedence problem. Pay particular attention to the difference in meaning between using `prec` inside of the `token` function versus outside of it. The *lexical precedence* syntax is `token(prec(N, ...))`.

### Keywords

Many languages have a set of *keyword* tokens (e.g. `if`, `for`, `return`), as well as a more general token (e.g. `identifier`) that matches any word, including many of the keyword strings. For example, JavaScript has a keyword `instanceof`, which is used as a binary operator, like this:

```js
if (a instanceof Something) b();
```

The following, however, is not valid JavaScript:

```js
if (a instanceofSomething) b();
```

A keyword like `instanceof` cannot be followed immediately by another letter, because then it would be tokenized as an `identifier`, **even though an identifier is not valid at that position**. Because Tree-sitter uses context-aware lexing, as described [above](#conflicting-tokens), it would not normally impose this restriction. By default, Tree-sitter would recognize `instanceofSomething` as two separate tokens: the `instanceof` keyword followed by an `identifier`.

### Keyword Extraction

Fortunately, Tree-sitter has a feature that allows you to fix this, so that you can match the behavior of other standard parsers: the `word` token. If you specify a `word` token in your grammar, Tree-sitter will find the set of *keyword* tokens that match strings also matched by the `word` token. Then, during lexing, instead of matching each of these keywords individually, Tree-sitter will match the keywords via a two-step process where it *first* matches the `word` token.

For example, suppose we added `identifier` as the `word` token in our JavaScript grammar:

```js
grammar({
  name: 'javascript',

  word: $ => $.identifier,

  rules: {
    _expression: $ => choice(
      $.identifier,
      $.unary_expression,
      $.binary_expression
      // ...
    ),

    binary_expression: $ => choice(
      prec.left(1, seq($._expression, 'instanceof', $._expression))
      // ...
    ),

    unary_expression: $ => choice(
      prec.left(2, seq('typeof', $._expression))
      // ...
    ),

    identifier: $ => /[a-z_]+/
  }
});
```

Tree-sitter would identify `typeof` and `instanceof` as keywords. Then, when parsing the invalid code above, rather than scanning for the `instanceof` token individually, it would scan for an `identifier` first, and find `instanceofSomething`. It would then correctly recognize the code as invalid.

Aside from improving error detection, keyword extraction also has performance benefits. It allows Tree-sitter to generate a smaller, simpler lexing function, which means that **the parser will compile much more quickly**.

### External Scanners

Many languages have some tokens whose structure is impossible or inconvenient to describe with a regular expression. Some examples:

* [Indent and dedent][indent-tokens] tokens in Python
* [Heredocs][heredoc] in Bash and Ruby
* [Percent strings][percent-string] in Ruby

Tree-sitter allows you to handle these kinds of tokens using *external scanners*. An external scanner is a set of C functions that you, the grammar author, can write by hand in order to add custom logic for recognizing certain tokens.

To use an external scanner, there are a few steps. First, add an `externals` section to your grammar. This section should list the names of all of your external tokens. These names can then be used elsewhere in your grammar.

```js
grammar({
  name: 'my_language',

  externals: $ => [
    $.indent,
    $.dedent,
    $.newline
  ],

  // ...
});
```

Then, add another C or C++ source file to your project. Currently, its path must be `src/scanner.c` or `src/scanner.cc` for the CLI to recognize it. Be sure to add this file to the `sources` section of your `binding.gyp` file so that it will be included when your project is compiled by Node.js and uncomment the appropriate block in your `bindings/rust/build.rs` file so that it will be included in your Rust crate.

> **Note**
>
> C++ scanners are now deprecated and will be removed in the near future.
> While it is currently possible to write an external scanner in C++, it can be difficult
> to get working cross-platform and introduces extra requirements; therefore it
> is *greatly* preferred to use C.

In this new source file, define an [`enum`][enum] type containing the names of all of your external tokens. The ordering of this enum must match the order in your grammar's `externals` array; the actual names do not matter.

```c
#include "tree_sitter/parser.h"
#include "tree_sitter/alloc.h"
#include "tree_sitter/array.h"

enum TokenType {
  INDENT,
  DEDENT,
  NEWLINE
}
```

Finally, you must define five functions with specific names, based on your language's name and five actions: *create*, *destroy*, *serialize*, *deserialize*, and *scan*. These functions must all use [C linkage][c-linkage], so if you're writing the scanner in C++, you need to declare them with the `extern "C"` qualifier.

#### Create

```c
void *tree_sitter_my_language_external_scanner_create(void) {
  // ...
}
```

This function should create your scanner object. It will only be called once anytime your language is set on a parser. Often, you will want to allocate memory on the heap and return a pointer to it. If your external scanner doesn't need to maintain any state, it's ok to return `NULL`.

#### Destroy

```c
void tree_sitter_my_language_external_scanner_destroy(void *payload) {
  // ...
}
```

This function should free any memory used by your scanner. It is called once when a parser is deleted or assigned a different language. It receives as an argument the same pointer that was returned from the *create* function. If your *create* function didn't allocate any memory, this function can be a noop.

#### Serialize

```c
unsigned tree_sitter_my_language_external_scanner_serialize(
  void *payload,
  char *buffer
) {
  // ...
}
```

This function should copy the complete state of your scanner into a given byte buffer, and return the number of bytes written. The function is called every time the external scanner successfully recognizes a token. It receives a pointer to your scanner and a pointer to a buffer. The maximum number of bytes that you can write is given by the `TREE_SITTER_SERIALIZATION_BUFFER_SIZE` constant, defined in the `tree_sitter/parser.h` header file.

The data that this function writes will ultimately be stored in the syntax tree so that the scanner can be restored to the right state when handling edits or ambiguities. For your parser to work correctly, the `serialize` function must store its entire state, and `deserialize` must restore the entire state. For good performance, you should design your scanner so that its state can be serialized as quickly and compactly as possible.

#### Deserialize

```c
void tree_sitter_my_language_external_scanner_deserialize(
  void *payload,
  const char *buffer,
  unsigned length
) {
  // ...
}
```

This function should *restore* the state of your scanner based the bytes that were previously written by the `serialize` function. It is called with a pointer to your scanner, a pointer to the buffer of bytes, and the number of bytes that should be read.
It is good practice to explicitly erase your scanner state variables at the start of this function, before restoring their values from the byte buffer.

#### Scan

```c
bool tree_sitter_my_language_external_scanner_scan(
  void *payload,
  TSLexer *lexer,
  const bool *valid_symbols
) {
  // ...
}
```

This function is responsible for recognizing external tokens. It should return `true` if a token was recognized, and `false` otherwise. It is called with a "lexer" struct with the following fields:

* **`int32_t lookahead`** - The current next character in the input stream, represented as a 32-bit unicode code point.
* **`TSSymbol result_symbol`** - The symbol that was recognized. Your scan function should *assign* to this field one of the values from the `TokenType` enum, described above.
* **`void (*advance)(TSLexer *, bool skip)`** - A function for advancing to the next character. If you pass `true` for the second argument, the current character will be treated as whitespace; whitespace won't be included in the text range associated with tokens emitted by the external scanner.
* **`void (*mark_end)(TSLexer *)`** - A function for marking the end of the recognized token. This allows matching tokens that require multiple characters of lookahead. By default (if you don't call `mark_end`), any character that you moved past using the `advance` function will be included in the size of the token. But once you call `mark_end`, then any later calls to `advance` will *not* increase the size of the returned token. You can call `mark_end` multiple times to increase the size of the token.
* **`uint32_t (*get_column)(TSLexer *)`** - A function for querying the current column position of the lexer. It returns the number of codepoints since the start of the current line. The codepoint position is recalculated on every call to this function by reading from the start of the line.
* **`bool (*is_at_included_range_start)(const TSLexer *)`** - A function for checking whether the parser has just skipped some characters in the document. When parsing an embedded document using the `ts_parser_set_included_ranges` function (described in the [multi-language document section][multi-language-section]), the scanner may want to apply some special behavior when moving to a disjoint part of the document. For example, in [EJS documents][ejs], the JavaScript parser uses this function to enable inserting automatic semicolon tokens in between the code directives, delimited by `<%` and `%>`.
* **`bool (*eof)(const TSLexer *)`** - A function for determining whether the lexer is at the end of the file. The value of `lookahead` will be `0` at the end of a file, but this function should be used instead of checking for that value because the `0` or "NUL" value is also a valid character that could be present in the file being parsed.

The third argument to the `scan` function is an array of booleans that indicates which of external tokens are currently expected by the parser. You should only look for a given token if it is valid according to this array. At the same time, you cannot backtrack, so you may need to combine certain pieces of logic.

```c
if (valid_symbols[INDENT] || valid_symbols[DEDENT]) {

  // ... logic that is common to both `INDENT` and `DEDENT`

  if (valid_symbols[INDENT]) {

    // ... logic that is specific to `INDENT`

    lexer->result_symbol = INDENT;
    return true;
  }
}
```

#### External Scanner Helpers

##### Allocator

Instead of using libc's `malloc`, `calloc`, `realloc`, and `free`, you should use the versions prefixed with `ts_` from `tree_sitter/alloc.h`.
These macros can allow a potential consumer to override the default allocator with their own implementation, but by default will use the libc functions.

As a consumer of the tree-sitter core library as well as any parser libraries that might use allocations, you can enable overriding the default allocator and have it use the same one as the library allocator, of which you can set with `ts_set_allocator`.
To enable this overriding in scanners, you must compile them with the `TREE_SITTER_REUSE_ALLOCATOR` macro defined, and tree-sitter the library must be linked into your final app dynamically, since it needs to resolve the internal functions at runtime. If you are compiling
an executable binary that uses the core library, but want to load parsers dynamically at runtime, then you will have to use a special linker flag on Unix. For non-Darwin systems, that would be `--dynamic-list` and for Darwin systems, that would be `-exported_symbols_list`.
The CLI does exactly this, so you can use it as a reference (check out `cli/build.rs`).

For example, assuming you wanted to allocate 100 bytes for your scanner, you'd do so like the following example:

```c
#include "tree_sitter/parser.h"
#include "tree_sitter/alloc.h"

// ...

void *tree_sitter_my_language_external_scanner_create(void) {
  return ts_calloc(100, 1); // or ts_malloc(100)
}

// ...

```

##### Arrays

If you need to use array-like types in your scanner, such as tracking a stack of indentations or tags, you should use the array macros from `tree_sitter/array.h`.

There are quite a few of them provided for you, but here's how you could get started tracking some . Check out the header itself for more detailed documentation.

**NOTE**: Do not use any of the array functions or macros that are prefixed with an underscore and have comments saying that it is not what you are looking for.
These are internal functions used as helpers by other macros that are public. They are not meant to be used directly, nor are they what you want.

```c
#include "tree_sitter/parser.h"
#include "tree_sitter/array.h"

enum TokenType {
  INDENT,
  DEDENT,
  NEWLINE,
  STRING,
}

// Create the array in your create function

void *tree_sitter_my_language_external_scanner_create(void) {
  return ts_calloc(1, sizeof(Array(int)));

  // or if you want to zero out the memory yourself

  Array(int) *stack = ts_malloc(sizeof(Array(int)));
  array_init(&stack);
  return stack;
}

bool tree_sitter_my_language_external_scanner_scan(
  void *payload,
  TSLexer *lexer,
  const bool *valid_symbols
) {
  Array(int) *stack = payload;
  if (valid_symbols[INDENT]) {
    array_push(stack, lexer->get_column(lexer));
    lexer->result_symbol = INDENT;
    return true;
  }
  if (valid_symbols[DEDENT]) {
    array_pop(stack); // this returns the popped element by value, but we don't need it
    lexer->result_symbol = DEDENT;
    return true;
  }

  // we can also use an array on the stack to keep track of a string

  Array(char) next_string = array_new();

  if (valid_symbols[STRING] && lexer->lookahead == '"') {
    lexer->advance(lexer, false);
    while (lexer->lookahead != '"' && lexer->lookahead != '\n' && !lexer->eof(lexer)) {
      array_push(&next_string, lexer->lookahead);
      lexer->advance(lexer, false);
    }

    // assume we have some arbitrary constraint of not having more than 100 characters in a string
    if (lexer->lookahead == '"' && next_string.size <= 100) {
      lexer->advance(lexer, false);
      lexer->result_symbol = STRING;
      return true;
    }
  }

  return false;
}

```

#### Other External Scanner Details

If a token in the `externals` array is valid at a given position in the parse, the external scanner will be called first before anything else is done. This means the external scanner functions as a powerful override of Tree-sitter's lexing behavior, and can be used to solve problems that can't be cracked with ordinary lexical, parse, or dynamic precedence.

If a syntax error is encountered during regular parsing, Tree-sitter's first action during error recovery will be to call the external scanner's `scan` function with all tokens marked valid. The scanner should detect this case and handle it appropriately. One simple method of detection is to add an unused token to the end of the `externals` array, for example `externals: $ => [$.token1, $.token2, $.error_sentinel]`, then check whether that token is marked valid to determine whether Tree-sitter is in error correction mode.

If you put terminal keywords in the `externals` array, for example `externals: $ => ['if', 'then', 'else']`, then any time those terminals are present in the grammar they will be tokenized by the external scanner. It is similar to writing `externals: [$.if_keyword, $.then_keyword, $.else_keyword]` then using `alias($.if_keyword, 'if')` in the grammar.

If in the `externals` array use literal keywords then lexing works in two steps, the external scanner will be called first and if it sets a resulting token and returns `true` then the token considered as recognized and Tree-sitter moves to a next token. But the external scanner may return `false` and in this case Tree-sitter fallbacks to the internal lexing mechanism.

In case of some keywords defined in the `externals` array in a rule referencing form like `$.if_keyword` and there is no additional definition of that rule in the grammar rules, e.g., `if_keyword: $ => 'if'` then fallback to the internal lexer isn't possible because Tree-sitter doesn't know the actual keyword and it's fully the external scanner resposibilty to recognize such tokens.

External scanners are a common cause of infinite loops.
Be very careful when emitting zero-width tokens from your external scanner, and if you consume characters in a loop be sure use the `eof` function to check whether you are at the end of the file.

[ambiguous-grammar]: https://en.wikipedia.org/wiki/Ambiguous_grammar
[antlr]: https://www.antlr.org
[bison-dprec]: https://www.gnu.org/software/bison/manual/html_node/Generalized-LR-Parsing.html
[bison]: https://en.wikipedia.org/wiki/GNU_bison
[c-linkage]: https://en.cppreference.com/w/cpp/language/language_linkage
[cargo]: https://doc.rust-lang.org/cargo/getting-started/installation.html
[crate]: https://crates.io/crates/tree-sitter-cli
[cst]: https://en.wikipedia.org/wiki/Parse_tree
[dfa]: https://en.wikipedia.org/wiki/Deterministic_finite_automaton
[ebnf]: https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form
[ecmascript-spec]: https://262.ecma-international.org/6.0/
[ejs]: https://ejs.co
[enum]: https://en.wikipedia.org/wiki/Enumerated_type#C
[glr-parsing]: https://en.wikipedia.org/wiki/GLR_parser
[heredoc]: https://en.wikipedia.org/wiki/Here_document
[indent-tokens]: https://en.wikipedia.org/wiki/Off-side_rule
[language-spec]: https://en.wikipedia.org/wiki/Programming_language_specification
[lexing]: https://en.wikipedia.org/wiki/Lexical_analysis
[longest-match]: https://en.wikipedia.org/wiki/Maximal_munch
[lr-conflict]: https://en.wikipedia.org/wiki/LR_parser#Conflicts_in_the_constructed_tables
[lr-grammars]: https://en.wikipedia.org/wiki/LR_parser
[multi-language-section]: ./using-parsers#multi-language-documents
[named-vs-anonymous-nodes-section]: ./using-parsers#named-vs-anonymous-nodes
[field-names-section]: ./using-parsers#node-field-names
[nan]: https://github.com/nodejs/nan
[node-module]: https://www.npmjs.com/package/tree-sitter-cli
[node.js]: https://nodejs.org
[static-node-types]: ./using-parsers#static-node-types
[non-terminal]: https://en.wikipedia.org/wiki/Terminal_and_nonterminal_symbols
[npm]: https://docs.npmjs.com
[path-env]: https://en.wikipedia.org/wiki/PATH_(variable)
[peg]: https://en.wikipedia.org/wiki/Parsing_expression_grammar
[percent-string]: https://docs.ruby-lang.org/en/2.5.0/doc/syntax/literals_rdoc.html#label-Percent+Strings
[releases]: https://github.com/tree-sitter/tree-sitter/releases/latest
[s-exp]: https://en.wikipedia.org/wiki/S-expression
[syntax-highlighting]: ./syntax-highlighting
[syntax-highlighting-tests]: ./syntax-highlighting#unit-testing
[tree-sitter-cli]: https://github.com/tree-sitter/tree-sitter/tree/master/cli
[tree-sitter-javascript]: https://github.com/tree-sitter/tree-sitter-javascript
[yacc-prec]: https://docs.oracle.com/cd/E19504-01/802-5880/6i9k05dh3/index.html
[yacc]: https://en.wikipedia.org/wiki/Yacc
