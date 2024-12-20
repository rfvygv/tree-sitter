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

建议是尽可能全面地添加测试。如果它是一个可见的节点，请将其添加到 `test/corpus` 目录中的测试文件中。通常，测试每种语言结构的所有排列组合是个好主意。这不仅增加了测试覆盖率，还让读者更熟悉如何检查预期输出并了解语言的"边界"。

#### 属性

测试可以用一些`attributes`进行注释。属性必须放在标题中，位于测试名称下方，并以`:`开头。
有些属性还需要参数，这些参数需要使用括号括起来。

**注意**：如果你想提供多个参数，例如在多个平台上运行测试或测试多种语言，你可以在新行上重复该属性。

可用的属性如下：

- `:skip` — 这个属性将在运行 tree-sitter test 时跳过测试。
  当你想临时禁用运行某个测试而不删除它时，这个属性很有用。
- `:error` — 这个属性将断言解析树包含错误。这对验证某些输入无效很有用，而无需显示整个解析树，因此你应该在`---`行下省略解析树。
- `:fail-fast` — 如果带有此属性的测试失败，将停止测试其他测试。
- `:language(LANG)` — 这个属性将使用指定语言的解析器运行测试。对于多解析器仓库（如 XML 和 DTD，或 TypeScript 和 TSX）很有用。默认解析器将是根 `package.json` 中 `tree-sitter` 字段的第一个条目，因此能够选择第二或第三个解析器是有用的。
- `:platform(PLATFORM)` — 这个属性指定测试运行的平台。它对于测试平台特定的行为很有用（例如，Windows 的换行符与 Unix 不同）。 此属性必须与 Rust 的 [`std::env::consts::OS`](https://doc.rust-lang.org/std/env/consts/constant.OS.html)相匹配。

使用属性的示例：

```text
=========================
将被跳过的测试
:skip
=========================

int main() {}

-------------------------

====================================
将在 Linux 或 macOS 上运行的测试

:platform(linux)
:platform(macos)
====================================

int main() {}

------------------------------------

========================================================================
预期有错误的测试，如果没有解析错误将快速失败
:fail-fast
:error
========================================================================

int main ( {}

------------------------------------------------------------------------

=================================================
将使用 TypeScript 和 TSX 解析的测试
:language(typescript)
:language(tsx)
=================================================

console.log('Hello, world!');

-------------------------------------------------
```

#### 自动编译

你可能会注意到，在重新生成解析器后第一次运行 `tree-sitter test` 时，会花费一些额外的时间。这是因为 Tree-sitter 会自动将你的 C 代码编译成一个可动态加载的库。每当你通过重新运行 `tree-sitter generate` 更新解析器时，它会根据需要重新编译你的解析器。

#### 语法高亮测试

`tree-sitter test` 命令还会运行 `test/highlight` 文件夹中的任何语法高亮测试（如果存在）。有关语法高亮测试的更多信息，请参见[the syntax highlighting page][syntax-highlighting-tests]。

### 命令: `parse`

你可以使用 `tree-sitter parse` 在任意文件上运行解析器。这将打印出结果语法树，包括节点的范围和字段名称，如下所示：

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

你可以将任意数量的文件路径和全局模式传递给 `tree-sitter parse`，它会解析所有给定的文件。如果发生任何解析错误，该命令将以非零状态码退出。你还可以使用 `--quiet` 标志阻止语法树的打印。此外，`--stat` 标志会打印出所有处理文件的聚合解析成功/失败信息。这使得 `tree-sitter parse` 可用作一种辅助测试策略：你可以检查大量文件解析是否没有错误：

```sh
tree-sitter parse 'examples/**/*.go' --quiet --stat
```

### 命令: `highlight`

你可以使用 `tree-sitter highlight` 在任意文件上运行语法高亮。这可以直接使用 ansi 转义码将颜色输出到终端，或者如果传递 `--html` 标志，则生成 HTML。更多信息请参见[the syntax highlighting page][syntax-highlighting]。

### 语法 DSL

以下是你在 `grammar.js` 中定义规则时可以使用的内置函数的完整列表。一些这些函数的用例将在后续部分中更详细地解释。

* **Symbols (the `$` object)** - Every grammar rule is written as a JavaScript function that takes a parameter conventionally called `$`. The syntax `$.identifier` is how you refer to another grammar symbol within a rule. Names starting with `$.MISSING` or `$.UNEXPECTED` should be avoided as they have special meaning for the `tree-sitter test` command.
* **符号（`$` 对象）** - 每个语法规则都写成一个 JavaScript 函数，接受一个惯例上称为 `$` 的参数。在规则中使用 `$.identifier` 来引用另一个语法符号。名称以 `$.MISSING` 或 `$.UNEXPECTED` 开头的符号应避免使用，因为它们对 `tree-sitter test` 命令有特殊意义。
* **String and Regex literals** - The terminal symbols in a grammar are described using JavaScript strings and regular expressions. Of course during parsing, Tree-sitter does not actually use JavaScript's regex engine to evaluate these regexes; it generates its own regex-matching logic as part of each parser. Regex literals are just used as a convenient way of writing regular expressions in your grammar.
* **字符串和正则表达式文字** - 语法中的终端符号使用 JavaScript 字符串和正则表达式来描述。当然，在解析期间，Tree-sitter 实际上并不使用 JavaScript 的正则表达式引擎来评估这些正则表达式；它会为每个解析器生成自己的正则表达式匹配逻辑。正则表达式文字仅用于在语法中方便地编写正则表达式。
* **Regex Limitations** - Currently, only a subset of the Regex engine is actually
supported. This is due to certain features like lookahead and lookaround assertions
not feasible to use in an LR(1) grammar, as well as certain flags being unnecessary
for tree-sitter. However, plenty of features are supported by default:
* **正则表达式的限制** - 目前，实际上只支持正则表达式引擎的一个子集。这是因为某些功能（如前瞻和环视断言）在 LR(1) 语法中不可行，以及某些标志对于 Tree-sitter 来说是多余的。然而，许多功能是默认支持的：

  * Character classes
  * Character ranges
  * Character sets
  * Quantifiers
  * Alternation
  * Grouping
  * Unicode character escapes
  * Unicode property escapes
  * 字符类
  * 字符范围
  * 字符集
  * 量词
  * 交替
  * 分组
  * Unicode 字符转义
  * Unicode 属性转义

* **Sequences : `seq(rule1, rule2, ...)`** - This function creates a rule that matches any number of other rules, one after another. It is analogous to simply writing multiple symbols next to each other in [EBNF notation][ebnf].
* **序列 : `seq(rule1, rule2, ...)`** - 该函数创建一个匹配任意数量其他规则的规则，一个接一个。它类似于 [EBNF notation][ebnf] 中简单地写多个符号。
* **Alternatives : `choice(rule1, rule2, ...)`** - This function creates a rule that matches *one* of a set of possible rules. The order of the arguments does not matter. This is analogous to the `|` (pipe) operator in EBNF notation.
* **替代 : `choice(rule1, rule2, ...)`** - 该函数创建一个规则，匹配一组可能规则中的一个。参数的顺序无关紧要。这类似于 EBNF 记法中的 `|`（管道）运算符。
* **Repetitions : `repeat(rule)`** - This function creates a rule that matches *zero-or-more* occurrences of a given rule. It is analogous to the `{x}` (curly brace) syntax in EBNF notation.
* **重复 :  `repeat(rule)`** - 该函数创建一个规则，匹配零次或多次给定规则。它类似于 EBNF 记法中的 {x}（花括号）语法。
* **Repetitions : `repeat1(rule)`** - This function creates a rule that matches *one-or-more* occurrences of a given rule. The previous `repeat` rule is implemented in terms of `repeat1` but is included because it is very commonly used.
* **重复 : `repeat1(rule)`** - 该函数创建一个规则，匹配一次或多次给定规则。前面的 `repeat` 规则是以 repeat1 为基础实现的，但它很常用，所以被单独列出。
* **Options : `optional(rule)`** - This function creates a rule that matches *zero or one* occurrence of a given rule. It is analogous to the `[x]` (square bracket) syntax in EBNF notation.
* **可选 : `optional(rule)`** - 该函数创建一个规则，匹配零次或一次给定规则。它类似于 EBNF 记法中的 `[x]`（方括号）语法。
* **Precedence : `prec(number, rule)`** - This function marks the given rule with a numerical precedence which will be used to resolve [*LR(1) Conflicts*][lr-conflict] at parser-generation time. When two rules overlap in a way that represents either a true ambiguity or a *local* ambiguity given one token of lookahead, Tree-sitter will try to resolve the conflict by matching the rule with the higher precedence. The default precedence of all rules is zero. This works similarly to the [precedence directives][yacc-prec] in Yacc grammars.
* **优先级 : `prec(number, rule)`** - 该函数为给定规则标记一个数值优先级，该优先级将在解析器生成时用于解决 `[LR(1) 冲突][lr-conflict]`。当两个规则以一种代表真正的二义性或给定一个前瞻令牌的局部二义性的方式重叠时，Tree-sitter 将尝试通过匹配优先级较高的规则来解决冲突。所有规则的默认优先级为零。这类似于 Yacc 语法中的`[优先级指令][yacc-prec]`。
* **Left Associativity : `prec.left([number], rule)`** - This function marks the given rule as left-associative (and optionally applies a numerical precedence). When an LR(1) conflict arises in which all of the rules have the same numerical precedence, Tree-sitter will consult the rules' associativity. If there is a left-associative rule, Tree-sitter will prefer matching a rule that ends *earlier*. This works similarly to [associativity directives][yacc-prec] in Yacc grammars.
* **左结合 : `prec.left([number], rule)`** - 该函数将给定规则标记为左结合（并可选应用数值优先级）。当出现所有规则具有相同数值优先级的 LR(1) 冲突时，Tree-sitter 将参考规则的结合性。如果存在左结合规则，Tree-sitter 将优先匹配较早结束的规则。这类似于 Yacc 语法中的`[结合性指令][yacc-prec]`。
* **Right Associativity : `prec.right([number], rule)`** - This function is like `prec.left`, but it instructs Tree-sitter to prefer matching a rule that ends *later*.
* **右结合 : `prec.right([number], rule)`** - 该函数类似于 `prec.left`，但它指示 Tree-sitter 优先匹配较晚结束的规则。
* **Dynamic Precedence : `prec.dynamic(number, rule)`** - This function is similar to `prec`, but the given numerical precedence is applied at *runtime* instead of at parser generation time. This is only necessary when handling a conflict dynamically using the `conflicts` field in the grammar, and when there is a genuine *ambiguity*: multiple rules correctly match a given piece of code. In that event, Tree-sitter compares the total dynamic precedence associated with each rule, and selects the one with the highest total. This is similar to [dynamic precedence directives][bison-dprec] in Bison grammars.
* **动态优先级 : `prec.dynamic(number, rule)`** - 该函数类似于 `prec`，但给定的数值优先级在`运行时`应用，而不是在解析器生成时。只有在使用语法中的 `conflicts` 字段动态处理冲突并且存在真正的二义性时才需要：多个规则正确匹配给定代码片段。在这种情况下，Tree-sitter 将比较与每个规则相关的总动态优先级，并选择总优先级最高的规则。这类似于 Bison 语法中的`[动态优先级指令][bison-dprec]`。
* **Tokens : `token(rule)`** - This function marks the given rule as producing only
a single token. Tree-sitter's default is to treat each String or RegExp literal
in the grammar as a separate token. Each token is matched separately by the lexer
and returned as its own leaf node in the tree. The `token` function allows you to
express a complex rule using the functions described above (rather than as a single
regular expression) but still have Tree-sitter treat it as a single token.
The token function will only accept terminal rules, so `token($.foo)` will not work.
You can think of it as a shortcut for squashing complex rules of strings or regexes
down to a single token.
* **标记 : `token(rule)`** - 该函数将给定规则标记为仅产生一个标记。Tree-sitter 的默认设置是将语法中的每个字符串或正则表达式文字视为单独的标记。每个标记由词法分析器单独匹配，并在树中返回为其自己的叶节点。token 函数允许你使用上述函数（而不是单个正则表达式）来表达复杂规则，但仍然让 Tree-sitter 将其视为单个标记。token 函数只接受终端规则，因此 `token($.foo)` 不会工作。你可以将其视为将字符串或正则表达式的复杂规则压缩为单个标记的快捷方式。
* **Immediate Tokens : `token.immediate(rule)`** - Usually, whitespace (and any other extras, such as comments) is optional before each token. This function means that the token will only match if there is no whitespace.
* **即时标记 : `token.immediate(rule)`** - 通常，空白（以及任何其他附加内容，如注释）在每个标记之前是可选的。此函数意味着标记将仅在没有空白的情况下匹配。
* **Aliases : `alias(rule, name)`** - This function causes the given rule to *appear* with an alternative name in the syntax tree. If `name` is a *symbol*, as in `alias($.foo, $.bar)`, then the aliased rule will *appear* as a [named node][named-vs-anonymous-nodes-section] called `bar`. And if `name` is a *string literal*, as in `alias($.foo, 'bar')`, then the aliased rule will appear as an [anonymous node][named-vs-anonymous-nodes-section], as if the rule had been written as the simple string.
* **别名 : `alias(rule, name)`** - 该函数使给定规则在语法树中以备用名称*出现*。如果 `name` 是*符号*，如 `alias($.foo, $.bar)`，则别名规则将以名为 bar 的[命名节点][named-vs-anonymous-nodes-section]形式*出现*。如果 name 是字符串文字，如 `alias($.foo, 'bar')`，则别名规则将以[匿名节点][named-vs-anonymous-nodes-section]形式出现，就好像规则被写成简单的字符串一样。


* **Field Names : `field(name, rule)`** - This function assigns a *field name* to the child node(s) matched by the given rule. In the resulting syntax tree, you can then use that field name to access specific children.

In addition to the `name` and `rules` fields, grammars have a few other optional public fields that influence the behavior of the parser.
* **字段名称 : `field(name, rule)`** - 该函数为给定规则匹配的子节点分配一个*字段名称*。在生成的语法树中，你可以使用该字段名称访问特定的子节点。

除了 `name` 和 `rules` 字段外，语法还具有一些影响解析器行为的其他可选公共字段。

* **`extras`** - an array of tokens that may appear *anywhere* in the language. This is often used for whitespace and comments. The default value of `extras` is to accept whitespace. To control whitespace explicitly, specify `extras: $ => []` in your grammar.
* **`extras`** - 一个可能出现在语言中的任何地方的标记数组。这通常用于空白和注释。extras 的默认值是接受空白符。如果需要显式控制空白符，可以在语法中指定 `extras: $ => []`。
* **`inline`** - an array of rule names that should be automatically *removed* from the grammar by replacing all of their usages with a copy of their definition. This is useful for rules that are used in multiple places but for which you *don't* want to create syntax tree nodes at runtime.
* **`inline`** - 一个规则名称的数组，这些规则会自动从语法中移除，并用其定义的副本替换所有使用它们的地方。这对于在多个地方使用但不希望在运行时创建语法树节点的规则非常有用。
* **`conflicts`** - an array of arrays of rule names. Each inner array represents a set of rules that's involved in an *LR(1) conflict* that is *intended to exist* in the grammar. When these conflicts occur at runtime, Tree-sitter will use the GLR algorithm to explore all of the possible interpretations. If *multiple* parses end up succeeding, Tree-sitter will pick the subtree whose corresponding rule has the highest total *dynamic precedence*.
* **`conflicts`** - 一个包含规则名称数组的数组。每个内部数组代表在语法中预期存在的LR(1) 冲突的规则集合。当这些冲突在运行时发生时，Tree-sitter 将使用 GLR 算法探索所有可能的解释。如果`多个`解析成功，Tree-sitter 将选择对应规则具有最高总`动态优先级`的子树。
* **`externals`** - an array of token names which can be returned by an [*external scanner*](#external-scanners). External scanners allow you to write custom C code which runs during the lexing process in order to handle lexical rules (e.g. Python's indentation tokens) that cannot be described by regular expressions.
* **`externals`** - 可以由[*external scanner*](#external-scanners)返回的标记名称数组。外部扫描器允许你编写在词法分析过程中运行的自定义 C 代码，以处理无法用正则表达式描述的词法规则（例如 Python 的缩进标记）。
* **`precedences`** - an array of array of strings, where each array of strings defines named precedence levels in descending order. These names can be used in the `prec` functions to define precedence relative only to other names in the array, rather than globally. Can only be used with parse precedence, not lexical precedence.
* * **`precedences`** - 一个字符串数组的数组，每个字符串数组定义降序排列的命名优先级级别。这些名称可以在 `prec` 函数中使用，以仅相对于数组中的其他名称定义优先级，而不是全局定义。只能用于解析优先级，不能用于词法优先级。
* **`word`** - the name of a token that will match keywords for the purpose of the [keyword extraction](#keyword-extraction) optimization.
* **`word`** - 将匹配关键字以进行关键字提取优化的标记名称。
* **`supertypes`** an array of hidden rule names which should be considered to be 'supertypes' in the generated [*node types* file][static-node-types].
* **`supertypes`** - 一个隐藏规则名称的数组，这些规则在生成的[节点类型文件][static-node-types]中应被视为'超类型'。

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

#### 其他外部扫描器细节

如果在 `externals` 数组中的一个标记在解析的特定位置是有效的，那么在执行任何其他操作之前，将首先调用外部扫描器。这意味着外部扫描器函数作为 Tree-sitter 词法分析行为的强大覆盖，并且可以用于解决无法用普通的词法、解析或动态优先级解决的问题。

如果在常规解析过程中遇到语法错误，Tree-sitter 在错误恢复期间的第一步操作将是使用所有标记为有效的标记调用外部扫描器的 `scan` 函数。扫描器应该检测到这种情况并适当地处理它。一个简单的检测方法是在 externals 数组的末尾添加一个未使用的标记，例如 `externals: $ => [$.token1, $.token2, $.error_sentinel]`，然后检查该标记是否标记为有效，以确定 Tree-sitter 是否处于错误校正模式。

如果将终结关键字放在 externals 数组中，例如 `externals: $ => ['if', 'then', 'else']`，那么每当这些终结关键字在语法中出现时，它们都将由外部扫描器进行标记。这类似于在语法中编写 `externals: [$.if_keyword, $.then_keyword, $.else_keyword]` 然后在语法中使用 `alias($.if_keyword, 'if')`。

如果在 `externals` 数组中使用文字关键字，那么词法分析将分为两个步骤，首先将调用外部扫描器，如果它设置了一个结果标记并返回 `true`，那么该标记将被视为已识别，Tree-sitter 将转到下一个标记。但是，外部扫描器也可能返回 `false`，在这种情况下，Tree-sitter 将回退到内部词法分析机制。

如果在 externals 数组中定义了一些以 `$.if_keyword` 形式引用的关键字，并且在语法规则中没有对该规则进行额外的定义，例如 `if_keyword: $ => 'if'`，那么回退到内部词法分析器是不可能的，因为 Tree-sitter 不知道实际的关键字，完全是外部扫描器的责任来识别这样的标记。

外部扫描器是导致无限循环的常见原因。在从外部扫描器发出零宽标记时要非常小心，并且如果在循环中消耗字符，请务必使用 eof 函数来检查是否已到达文件的末尾。

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
