<!--
# Expressions
-->

# 式

<!--
## Complete syntax
-->

## 完全な構文

<!--
Expressions will be explained in more detail below, but just to get it out in the open, the complete syntax for expressions, including the string and nat literal extensions, is as follows:
-->

式の詳細については後述しますが、まずはこれが何であるかを明らかにするために、以下に文字列と nat リテラルの拡張を含む式についての完全な構文を挙げます：

```
Expr ::= 
  | boundVariable 
  | freeVariable 
  | const 
  | sort 
  | app 
  | lambda 
  | forall 
  | let 
  | proj 
  | literal

BinderInfo ::= Default | Implicit | InstanceImplicit | StrictImplicit

const ::= Name, Level*
sort ::= Level
app ::= Expr Expr
-- a deBruijn index
boundVariable ::= Nat
lambda ::= Name, (binderType : Expr), BinderInfo, (body : Expr)
forall ::= Name, (binderType : Expr), BinderInfo, (body : Expr)
let ::= Name, (binderType : Expr), (val : Expr) (body : Expr)
proj ::= Name Nat Expr
literal ::= natLit | stringLit

-- Arbitrary precision nat/unsigned integer
natLit ::= Nat
-- UTF-8 string
stringLit ::= String

-- fvarId can be implemented by unique names or deBruijn levels; 
-- unique names are more versatile, deBruijn levels have better
-- cache behavior
freeVariable ::= Name, Expr, BinderInfo, fvarId
```

<!--
Some notes:
-->

これについて、いくつかの注意点があります：

<!--
+ The `Nat` used by nat literals should be an arbitrary precision natural/bignum.
-->

+ nat リテラルが利用する `Nat` は任意精度の自然数・bignum でなければなりません。

<!--
+ The expressions that have binders (lambda, pi, let, free variable) can just as easily bundle the three arguments (binder_name, binder_type, binder_style) as one argument `Binder`, where a binder is `Binder ::= Name BinderInfo Expr`. In the pseudocode that appears elsewhere I will usually treat them as though they have that property, because it's easier to read.
-->

+ 束縛子（ラムダ・pi・let・自由変数）を持つ式では3つの引数（束縛子の名前・型・スタイル）を1つの引数 `Binder` として簡単にまとめることができます。ここで束縛子は `Binder ::= Name BinderInfo Expr` となります。他の場所で例示する疑似コードでは、読みやすさのためにそのプロパティを持っているかのように常に扱うことにします。

<!--
+ Free variable identifiers can be either unique identifiers, or they can be deBruijn levels.
-->

+ 自由変数の識別子は、一意な識別子にすることも de Bruijn レベルにすることもできます。

<!--
+ The expression type used in Lean proper also has an `mdata` constructor, which declares an expression with attached metadata. This metadata does not effect the expression's behavior in the kernel, so we do not include this constructor.
-->

+ Lean で使用される式の型は `mdata` コンストラクタというものも持っており、これはメタデータを付与した式を宣言します。このメタデータはカーネル内での式の動作には影響しないため、このコンストラクタはここでは取り扱いません。

<!--
## Binder information
-->

## 束縛子の情報

<!--
Expressions constructed with the lambda, pi, let, and free variable constructors contain binder information, in the form of a name, a binder "style", and the binder's type. The binder's name and style are only for use by the pretty printer, and do not alter the core procedures of inference, reduction, or equality checking. In the pretty printer however, the binder style may alter the output depending on the pretty printer options. For example, the user may or may not want to display implicit or instance implicits (typeclass variables) in the output.
-->

ラムダ・pi・let・自由変数コンストラクタで構築された式は、名前・束縛子の「スタイル」・束縛子の型の形式で束縛子の情報を保持します。束縛子の名前とスタイルはプリティプリンタが使用するためだけのものであり、推論・簡約・同値チェックの中核となる処理を変更することはありません。しかし、プリティプリンタでは束縛子のスタイルがプリティプリンタのオプションに応じて出力を変更することがあります。例えば、ユーザは暗黙的または暗黙インスタンス（型クラス変数）を出力に表示したい場合もあれば、したくないこともあるでしょう。

<!--
### Sort
-->

### ソート

<!--
`sort` is simply a wrapper around a level, allowing it to be used as an expression.
-->

`sort` は単にレベルのラッパーであり、式として使用することができます。

<!--
### Bound variables
-->

### 束縛変数

<!--
Bound variables are implemented as natural numbers representing [deBruijn indices](https://en.wikipedia.org/wiki/De_Bruijn_index).
-->

束縛変数は [de Bruijn インデックス](https://ja.wikipedia.org/wiki/%E3%83%89%E3%83%BB%E3%83%96%E3%83%A9%E3%82%A6%E3%83%B3%E3%83%BB%E3%82%A4%E3%83%B3%E3%83%87%E3%83%83%E3%82%AF%E3%82%B9) を表す自然数として実装されています。

<!--
### Free variables
-->

### 自由変数

<!--
Free variables are used to convey information about bound variables in situations where the binder is currently unavailable. Usually this is because the kernel has traversed into the body of a binding expression, and has opted not to carry a structured context of the binding information, instead choosing to temporarily swap out the bound variable for a free variable, with the option of swapping in a new (maybe different) bound variable to reconstruct the binder. This unavailability description may sound vague, but a literal explanation that might help is that expressions are implemented as trees without any kind of parent pointer, so when we descend into child nodes (especially across function boundaries), we end up just losing sight of the elements above where we currently are in a given expression.
-->

自由変数は束縛子が一時的に利用できない状況で、束縛変数に関する情報を伝達するために使用されます。これは自由変数について、新しい（異なる場合もある）束縛変数に入れ替えて束縛子を再構築する上で束縛変数を一時的に自由変数に置き換える代わりに、カーネルが束縛式の本体を走査し、束縛情報の構造化されたコンテキストを保持しないことを通常選択するためです。この利用できないことの説明は曖昧に聞こえるかもしれませんが、役立つように文字通りに説明すると、式は親ノードのポインタを持たない木として実装されているため、子ノードに降りてくると（特に関数の境界を超える場合）、ある式の現在地より上の要素を見失ってしまうことになってしまいます。

<!--
When an open expression is closed by reconstructing binders, the bindings may have changed, invalidating previously valid deBruijn indices. The use of unique names or deBruijn levels allow this re-closing of binders to be done in a way that compensates for any changes and ensures the new deBruijn indices of the re-bound variables are valid with respect the reconstructed telescope (see [this section](./kernel_concepts.md#implementing-free-variable-abstraction)).
-->

束縛子を再構築して開いた式を閉じる時、束縛子が変更されることで以前に有効だった de Bruijn インデックスが無効になることがあります。一意な名前か de Bruijn レベルを使用することで、あらゆる変更を補正し、再束縛された変数の新しい de Bruijn インデックスが再構築されたテレスコープに対して有効であることを保証するようにこの束縛子を再度閉じることができます（[この節](./kernel_concepts/instantiation_and_abstraction.md#自由変数の抽象化の実装) を参照してください [^fn1]）

<!--
Going forward, we may use some form of the term "free variable identifier" to refer to the objects in whatever scheme (unique IDs or deBruijn levels) an implementation may be using.
-->

今後、実装がどのようなスキーム（一意なIDか de Bruijn レベル）を使っているかに関わらず、何らかの形で「自由変数の識別子（free variable identifier）」という用語を使うかもしれません。

<!--
### `Const`
-->

### `const`

<!--
The `const` constructor is how an expression refers to another declaration in the environment, it must do so by reference. 
-->

`const` コンストラクタは、ある式が環境内の別の宣言を参照するための方法です。これは参照として行われなければなりません。

<!--
In example below, `def plusOne` creates a `Definition` declaration, which is checked, then admitted to the environment. Declarations cannot be placed directly in expressions, so when the type of `plusOne_eq_succ` invokes the previous declaration `plusOne`, it must do so by name. An expression is created: `Expr.const (plusOne, [])`, and when the kernel finds this `const` expression, it can look up the declaration referred to by name, `plusOne`, in the environment:
-->

以下の例では、`def plusOne` が `Definition` 宣言を作成し、これがチェックされたのちに環境に受け入れられます。こうした宣言は式の中に直接置くことはできないため、`plusOne_eq_succ` の型が前の宣言 `plusOne` を呼び出すときは名前を指定しなければなりません。こうして `Expr.const (plusOne, [])` という式が作成され、カーネルがこの `const` 式を見つけると、`plusOne` という名前で参照されている宣言をこの環境で探すことができます：

```
def plusOne : Nat -> Nat := fun x => x + 1

theorem plusOne_eq_succ (n : Nat) : plusOne n = n.succ := rfl 
```

<!--
Expressions created with the `const` constructor also carry a list of levels which are substituted into any unfolded or inferred declarations taken from the environment by looking up the definition the `const` expression refers to. For example, inferring the type of `const List [Level.param(x)]` involves looking up the declaration for `List` in the current environment, retrieving its type and universe parameters, then substituting `x` for the universe parameter with which `List` was initially declared.
-->

`const` コンストラクタで作成された式は、`const` 式が参照する定義を検索することで、環境から取得した宣言の展開や推論に代入されるレベルのリストも保持します。例えば、`const List [Level.param(x)]` の型を推測するには、現在の環境で `List` の宣言を検索し、その型と宇宙パラメータを取得し、そして `List` が最初に宣言された宇宙パラメータに `x` を代入します。

<!--
### Lambda, Pi
-->

### ラムダ式、pi

<!--
`lambda` and `pi` expressions (Lean proper uses the name `forallE` instead of `pi`) refer to function abstraction and the "forall" binder (dependent function types) respectively. 
-->

`lambda` と `pi` 式（Lean では `pi` の代わりに `forallE` を使用します）は、それぞれ関数の抽象化と「全ての」束縛子（依存関数型）を意味します。

<!--
```
  binderName      body
      |            |
fun (foo : Bar) => 0 
            |         
        binderType    

-- `BinderInfo` is reflected by the style of brackets used to
-- surround the binder.
```
-->

```
   束縛子名        本体
      |            |
fun (foo : Bar) => 0 
            |         
         束縛子の型

-- `BinderInfo` は束縛子を囲む括弧のスタイルに反映されています。
```

<!--
### Let
-->

### let 式

<!--
`let` is exactly what it sounds like. While `let` expressions are binders, they do not have a `BinderInfo`, their binder info is treated as `Default`.
-->

`let` はまさにその名前の通りのものです。`let` 式は束縛子ですが、`BinderInfo` を持たず、束縛子の情報は `Default` として扱われます。

<!--
```
  binderName      val
      |            |
let (foo : Bar) := 0; foo
            |          |
        binderType     .... body
```
-->

```
   束縛子名        値
      |            |
let (foo : Bar) := 0; foo
            |          |
        束縛子の型      .... 本体
```


<!--
### App
-->

### 適用

<!--
`app` expressions represent the application of an argument to a function. App nodes are binary (have only two children, a single function and an single argument), so `f x_0 x_1 .. x_N` is represented by `App(App(App(f, x_0), x_1)..., x_N)`, or visualized as a tree:
-->

`app` 式は関数への引数の適用を表します。適用のノードはバイナリ（2つの子、つまり1つの関数とその1つの引数のみを持つ）であるため、`f x_0 x_1 .. x_N` は `App(App(App(f, x_0), x_1)..., x_N)` として表され、以下のような木として視覚化されます：

```
                App
                / \
              ...  x_N
              /
            ...
           App
          / \
       App  x_1
       / \
     f  x_0

```

<!--
An exceedingly common kernel operation for manipulating expressions is folding and unfolding sequences of applications, getting `(f, [x_0, x_1, .., x_N])` from the tree structure above, or folding `f, [x_0, x_1, .., x_N]` into the tree above.
-->

カーネルが提供する式の操作は非常に一般的で、適用の列を畳み込んだり展開したりすることで、上の木構造から `(f, [x_0, x_1, .., x_N])` を取得したり、`f, [x_0, x_1, .., x_N]` を上の木へと畳み込んだりすることができます。

<!--
### Projections
-->

### 射影

<!--
The `proj` constructor represents structure projections. Inductive types that are not recursive, have only one constructor, and have no indices can be structures.
-->

`proj` コンストラクタは構造体の射影を表します。再帰的でなく、コンストラクタが1つしかなく、添字を持たない帰納型は構造体になることができます。

<!--
The constructor takes a name, which is the name of the type, a natural number indicating the field being projected, and the actual structure the projection is being applied to.
-->

このコンストラクタは当該の型の名前と、射影されるフィールドを示す自然数、そして射影が適用される実際の構造体を取ります。

<!--
Be aware that in the kernel, projection indices are 0-based, despite being 1-based in Lean's vernacular, where 0 is the first non-parameter argument to the constructor.
-->

ここでカーネルでは射影のインデックスは 0 始まりであるのに対して Lean の用語では 1 始まりであることに注意してください。Lean サイドでは 0 はそのコンストラクタのパラメータではない最初の引数に使われます。

<!--
For example, the kernel expression `proj Prod 0 (@Prod.mk A B a b)` would project the `a`, because it is the 0th field after skipping the parameters `A` and `B`.
-->

例えば、`proj Prod 0 (@Prod.mk A B a b)` というカーネルの式は、パラメータ `A` と `B` をスキップした後の0番目のフィールドとして `a` を射影します。

<!--
While the behavior offered by `proj` can be accomplished by using the type's recursor, `proj` more efficiently handles frequent kernel operations.
-->

`proj` が提供する動作は、型の再帰子を使用することで実現できますが、`proj` は頻繁に行われるカーネルの操作をより効率的に処理します。

<!--
### Literals
-->

### リテラル

<!--
Lean's kernel optionally supports arbitrary precision Nat and String literals. As needed, the kernel can transform a nat literal `n` to `Nat.zero` or `Nat.succ m`, or convert a string literal `s` to `String.mk List.nil` or `String.mk (List.cons (Char.ofNat _) ...)`.
-->

Lean のカーネルはオプションとして任意精度の Nat リテラルと String リテラルをサポートしています。必要に応じてカーネルは自然数リテラル `n` を `Na.zero` または `Nat.succ m` に変換したり、文字列リテラル `s` を `String.mk List.nil` や `String.mk (List.cons (Char.ofNat _) ...)` に変換したりすることができます。

<!--
String literals are lazily converted to lists of characters for testing definitional equality, and when they appear as the major premise in reduction of a recursor.
-->

文字列リテラルは、定義上の同値をテストするためか、再帰子の簡約において主要な対象として現れる時に文字のリストに遅延変換されます。

<!--
Nat literals are supported in the same positions as strings (definitional equality and major premises of a recursor application), but the kernel also provide support for addition, multiplication, exponentiation, subtraction, mod, division, as well as boolean equality and "less than or equal" comparisons on nat literals.
-->

自然数リテラルは文字列を同じ立ち位置（定義上の同値と再帰子の適用の主要な対象）でサポートされていますが、カー熱は加算・乗算・指数・減算・剰余・割り算のほか、自然数リテラルに対する真偽値の同値と「以下」の比較もサポートしています。

[^fn1]: 訳注：原文では`./kernel_concepts.md#implementing-free-variable-abstraction`へリンクが貼られていましたがリンクが切れていたので対応するリンクに更新しています。