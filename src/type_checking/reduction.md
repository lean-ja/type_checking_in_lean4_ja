<!--
# Reduction
-->

# 簡約

<!--
Reduction is about nudging expressions toward their [normal form](https://en.wikipedia.org/wiki/Normal_form_(abstract_rewriting)) so we can determine whether expressions are definitionally equal. For example, we need to perform beta reduction to determine that `(fun x => x) Nat.zero` is definitionally equal to `Nat.zero`, and delta reduction to determine that `id List.nil` is definitionally equal to `List.nil`. 
-->

簡約とは、式が定義上等しいかどうかを判定てきるように式を [正規形](https://ja.wikipedia.org/wiki/%E6%AD%A3%E8%A6%8F%E5%8C%96_(%E9%A0%85%E6%9B%B8%E3%81%8D%E6%8F%9B%E3%81%88)) に近づけることです。例えば、`(fun x => x) Nat.zero` が `Nat.zero` と定義上等しいと判断するためにβ簡約を行い、`id List.nil` が `List.nil` と定義上等しいと判断するためにδ簡約を行う必要があります。

<!--
Reduction in Lean's kernel has two properties that introduce concerns which sometimes go unaddressed in basic textbook treatments of the topic. First, reduction in some cases is interleaved with inference. Among other things, this means reduction may need to be performed with open terms, even though the reduction procedures themselves are not creating free variables. Second, `const` expressions which are applied to multiple arguments may need to be considered together with those arguments during reduction (as in iota reduction), so sequences of applications need to be unfolded together at the beginning of reduction. 
-->

Lean のカーネルにおける簡約には2つの性質がありますが、この分野を扱う基本的な教科書では触れられない恐れがあります。まず、場合によっては簡約と推論が同時に行われることがあります。とりわけ、たとえ簡約によって自由変数が生成されなくとも、これは開いた項で簡約を実行する必要があるかもしれないことを意味しています。第二に、複数の引数に適用される `const` 式は（ι簡約のように）簡約を行う中でそれらの引数を一緒に考慮する必要がある場合があり、簡約開始時に一連の適用を一緒に展開する必要があります。

<!--
## Beta reduction 
-->

## β簡約

<!--
Beta reduction is about reducing function application. Concretely, the reduction:
-->

β簡約は関数適用についての簡約です。具体的には、次の簡約です：

```
(fun x => x) a    ~~>    a
```

<!--
An implementation of beta reduction must despine an expression to collect any arguments in `app` expressions, check whether the expression to which they're applied is a lambda, then substitute the appropriate argument for any appearances of the corresponding bound variable in the function body:
-->

β簡約の実装は `app` 式の引数を集め、それらが適用される式がラムダ式であるかどうかをチェックし、適切な引数を関数本体の対応する束縛変数の出現に置換するために式を手入れしなければなりません：

```
betaReduce e:
  betaReduceAux e.unfoldApps

betaReduceAux f args:
  match f, args with
  | lambda _ body, arg :: rest => betaReduceAux (inst body arg) rest
  | _, _ => foldApps f args
```

<!--
An important performance optimization for the instantiation (substitution) component of beta reduction is what's sometimes referred to as "generalized beta reduction", which involves gathering the arguments that have a corresponding lambda and substituting them in all at once. This optimization means that for `n` sequential lambda expressions with applied arguments, we only perform one traveral of the expression to substitute the appropriate arguments, instead of `n` traversals.
-->

β簡約のインスタンス化（置換）コンポーネントの重要なパフォーマンス最適化は、「generalized beta reduction」と呼ばれるもので、対応するラムダ式を持つ引数を集め、それらを一度に置換する方法です。この最適化は、適用された引数を持つ `n` 個の連続したラムダ式に対して、`n` 回の走査ではなく、適切な引数を代入するための1回の走査のみを実行することを意味します。

```
betaReduce e:
  betaReduceAux e.unfoldApps []

betaReduceAux f remArgs argsToApply:
  match f, remArgs with
  | lambda _ body, arg :: rest => betaReduceAux body rest (arg :: argsToApply)
  | _, _ => foldApps (inst f argsToApply) remArgs
```

<!--
## Zeta reduction (reducing `let` expressions)
-->

## ζ簡約（`let` 式の簡約）

<!--
Zeta reduction is a fancy name for reduction of `let` expressions. Concretely, reducing 
-->

ζ簡約は `let` 式の簡約を表す大仰な名前です。具体的には、次の簡約です：

```
let (x : T) := y; x + 1    ~~>    (y : T) + 1
```

<!--
An implementation can be as simple as:
-->

実装はシンプルであり、ほぼ次と同じです：

```
reduce Let _ val body:
  instantiate body val
```

<!--
## Delta reduction (definition unfolding)
-->

## δ簡約（定義の展開）

<!--
Delta reduction refers to unfolding definitions (and theorems). Delta reduction is done by replacing a `const ..` expr with the referenced declaration's value, after swapping out the declaration's generic universe parameters for the ones that are relevant to the current context. 
-->

δ簡約は定義（と定理）を展開することを指します。δ簡約は `const ..` 式について、その定義の多相宇宙パラメータを現在のコンテキストに関連した値で交換した後に、参照されている定義の値で置き換えることによって行われます。

<!--
If the current environment contains a definition `x` which was declared with universe parameters `u*`and value `v`, then we may delta reduce an expression `Const(x, w*)` by replacing it with `val`, then substituting the universe parameters `u*` for those in `w*`.
-->

現在の環境に定義 `x` があり、その定義が宇宙パラメータ `u*` と値 `v` で宣言されている場合、式 `Const(x, w*)` を `val` で置き換えてδ簡約し、宇宙パラメータ `u*` を `w*` に置き換えます。

```
deltaReduce Const name levels:
  if environment[name] == (d : Declar) && d.val == v then
  substituteLevels (e := v) (ks := d.uparams) (vs := levels)
```

<!--
If we had to remove any applied arguments to reach the `const` expression that was delta reduced, those arguments should be reapplied to the reduced definition.
-->

δ簡約された `const` 式に到達するために、適用された引数を簡約しなければならなかった場合、それらの引数は簡約された定義に再適用されなければなりません。

<!--
## Projection reduction
-->

## 射影の簡約

<!--
A `proj` expression has a natural number index indicating the field to be projected, and another expression which is the structure itself. The actual expression comprising the structure should be a sequence of arguments applied to `const` referencing a constructor.
-->

`proj` 式は射影されるフィールドを表す自然数のインデックスと、構造体そのものを示す式を持ちます。構造体を構成する実際の式は、コンストラクタを参照する `const` に適用される一連の引き数でなければなりません。

<!--
Keep in mind that for fully elaborated terms, the arguments to the constructor will include any parameters, so an instance of the `Prod` constructor would be e.g. `Prod.mk A B (a : A) (b : B)`.
-->

ここで心に留めてもらいたいこととして、完全にエラボレートされた項に対して、コンストラクタの引数は任意のパラメータを含むため、`Prod` コンストラクタのインスタンスは例えば `Prod.mk A B (a : A) (b : B)` となります。

<!--
The natural number indicating the projected field is 0-based, where 0 is the first *non-parameter* argument to the constructor, since a projection cannot be used to access the structure's parameters.
-->

射影されたフィールドを示す自然数は0始まりであり、0はコンストラクタの最初の **非パラメータ** 引数です。というのも射影は構造体のパラメータのアクセスに使うことができないからです。

<!--
With this in mind, it becomes clear that once we despine the constructor's arguments into `(constructor, [arg0, .., argN])`, we can reduce the projection by simply taking the argument at position `i + num_params`, where `num_params` is what it sounds like, the number of parameters for the structure type.
-->

これを念頭に置けば、コンストラクタの引数を `(constructor, [arg0, .., argN])` へと手直しすれば、射影の簡約は単純に `i + num_params` の位置にある引数を取るだけであることが明白になるでしょう。ここで `num_params` はその名前の通りのもので、その構造体型のパラメータの数を表します。

```
reduce proj fieldIdx structure:
  let (constructorApp, args) := unfoldApps (whnf structure)
  let num_params := environment[constructorApp].num_params
  args[fieldIdx + numParams]

  -- Following our `Prod` example, constructorApp will be `Const(Prod.mk, [u, v])`
  -- args will be `[A, B, a, b]`
```

<!--
### Special case for projections: String literals
-->

### 射影の特殊なケース：文字列リテラル

<!--
The kernel extension for string literals introduces one special case in projection reduction, and one in iota reduction. 
-->

カーネルは文字列リテラルについて拡張を行っており、これによって射影の簡約で1つ、ι簡約で1つの特殊なケースが導入されます。

<!--
Projection reduction for string literals: Because the projection expression's structure might reduce to a string literal (Lean's `String` type is defined as a structure with one field, which is a `List Char`)
-->

文字列リテラルに対する射影の簡約：なぜなら構造体の射影の式を簡約すると文字列リテラルになる可能性があるからです（Lean の `String` 型は `List Char` のフィールドを1つ持つ構造体として定義されています）。

<!--
If the structure reduces as a `StringLit (s)`, we convert that to `String.mk (.. : List Char)` and proceed as usual for projection reduction.
-->

もしその構造体が `StringLit (s)` に簡約される場合、これを `String.mk (.. : List Char)` に変換し、いつものように射影の簡約が処理されます。

<!--
## Nat literal reduction
-->

## 整数リテラルの簡約

<!--
The kernel extension for nat literals includes reduction of `Nat.succ` as well as the binary operations of addition, subtraction, multiplication, exponentiation, division, mod, boolean equality, and boolean less than or equal.
-->

整数リテラル用のカーネル拡張には加算・減算・乗算・指数・除算・剰余・真偽値の同値・真偽値の以下の二項演算に加えて `Nat.succ` の簡約が含まれます。

<!--
If the expression being reduced is `Const(Nat.succ, []) n` where `n` can be reduced to a nat literal `n'`, we reduce to `NatLit(n'+1)`
-->

簡約される式が `Const(Nat.succ, []) n` で `n` が整数リテラル `n` に簡約できる場合、`NatLit(n'+1)` に簡約されます。

<!--
If the expression being reduced is `Const(Nat.<binop>, []) x y` where `x` and `y` can be reduced to nat literals `x'` and `y'`, we apply the native version of the appropriate `<binop>` to `x'` and `y'`, returning the resulting nat literal.
-->

もし式が `Const(Nat.<binop>, []) x y` へと簡約され `x` と `y` がそれぞれ整数リテラル `x'` と `y'` に簡約される場合、適切な `<binop>` のネイティブ版を `x'` と `y'` に適用し、結果の整数リテラルを返します。

<!--
Examples:
-->

例：
```
Const(Nat.succ, []) NatLit(100) ~> NatLit(100+1)

Const(Nat.add, []) NatLit(2) NatLit(3) ~> NatLit(2+3)

Const(Nat.add, []) (Const Nat.succ [], NatLit(10)) NatLit(3) ~> NatLit(11+3)
```

<!--
## Iota reduction (pattern matching)
-->

## ι簡約（パターンマッチ）

<!--
Iota reduction is about applying reduction strategies that are specific to, and derived from, a given inductive declaration. What we're talking about is application of an inductive declaration's recursor (or the special case of `Quot` which we'll see later).
-->

ι簡約とは与えられた帰納的宣言に特有でかつそこから派生した簡約戦略を適用することです。これはまさに帰納的宣言の再帰子（または後で説明する `Quot` の特別な場合）の適用です。

<!--
Each recursor has a set of "recursor rules", one recursor rule for each constructor. In contrast to the recursor, which presents as a type, these recursor rules are value level expressions showing how to eliminate an element of type `T` created with constructor `T.c`. For example, `Nat.rec` has a recursor rule for `Nat.zero`, and another for `Nat.succ`.
-->

各再帰子は「再帰子ルール」を持っており、各コンストラクタごとに1つの再帰子ルールがあります。型として表示される再帰子とは対照的に、これらの再帰子ルールはコンストラクタ `T.c` で作成された `T` 型の要素をどのように除去するかを示す値レベルの式です。例えば、`Nat.rec` は `Nat.zero` と `Nat.succ` それぞれに対する再帰子ルールを持ちます。

<!--
For an inductive declaration `T`, one of the elements demanded by `T`'s recursor is an actual `(t : T)`, which is the thing we're eliminating. This `(t : T)` argument is known as the "major premise". Iota reduction performs pattern matching by taking apart the major premise to see what constructor was used to make `t`, then retrieving and applying the corresponding recursor rule from the environment.
-->

帰納的宣言 `T` に対して、`T` の再帰子が要求する要素の1つは実際の `(t : T)` であり、これは除去する対象です。この `(t : T)` 引数は「major premise」と呼ばれます。ι簡約は major premise を分解することで `t` の構築にあたってどのコンストラクタが使われたかを確認し、対応する再帰子ルールを環境から取得して適用することでパターンマッチを行います。

<!--
Because the recursor's type signature also demands the parameters, motives, and minor premises required, we don't need to change the arguments to the recursor to perform reduction on e.g. `Nat.zero` as opposed to `Nat.succ`.
-->

再帰子の型シグネチャは必要なパラメータ・motive・minor premiseも要求するため、例えば `Nat.succ` とは対照的に `Nat.zero` に対して簡約を実行するために再帰子の引数を変更する必要はありません。

<!--
In practice, it's sometimes necessary to do some initial manipulation to expose the constructor used to create the major premise, since it may not be found as a direct application of a constructor. For example, a `NatLit(n)` expression will need to be transformed into either `Nat.zero`, or `App Const(Nat.succ, []) ..`. For structures, we may also perform structural eta expansion, transforming an element `(t : T)` into `T.mk t.1 .. t.N`, thereby exposing the application of the `mk` constructor, permitting iota reduction to proceed (if we can't figure out what constructor was used to create the major premise, reduction fails).
-->

実際には major premise を作成するために使用されたコンストラクタを公開するための初期化的な手動操作が必要になることがあります。というのもこのコンストラクタがコンストラクタを直接適用したものとして見つからないことがあるからです。例えば、`NatLit(n)` 式は `Nat.zero` か `App Const(Nat.succ, []) ..` のいずれかに変換する必要があります。構造体では、構造体のη展開を行う場合もあります。要素 `(t : T)` を `T.mk t.1 .. t.N` に変換することで、`mk` コンストラクタの適用が明らかになり、ι簡約を進めることができます（major premise を作成するために使用されたコンストラクタが分からない場合、簡約は失敗します）。

## List.rec type

```
forall 
  {α : Type.{u}} 
  {motive : (List.{u} α) -> Sort.{u_1}}, 
  (motive (List.nil.{u} α)) -> 
  (forall (head : α) (tail : List.{u} α), (motive tail) -> (motive (List.cons.{u} α head tail))) -> (forall (t : List.{u} α), motive t)
```

## List.nil rec rule

```
fun 
  (α : Type.{u}) 
  (motive : (List.{u} α) -> Sort.{u_1}) 
  (nilCase : motive (List.nil.{u} α)) 
  (consCase : forall (head : α) (tail : List.{u} α), (motive tail) -> (motive (List.cons.{u} α head tail))) => 
  nil
```

## List.cons rec rule

```
fun 
  (α : Type.{u}) 
  (motive : (List.{u} α) -> Sort.{u_1}) 
  (nilCase : motive (List.nil.{u} α)) 
  (consCase : forall (head : α) (tail : List.{u} α), (motive tail) -> (motive (List.cons.{u} α head tail))) 
  (head : α) 
  (tail : List.{u} α) => 
  consCase head tail (List.rec.{u_1, u} α motive nilCase consCase tail)
```


<!--
### k-like reduction
-->

### k-like 簡約

<!--
For some inductive types, known as "subsingleton eliminators", we can proceed with iota reduction even when the major premise's constructor is not directly exposed, as long as we know its type. This may be the case when, for example, the major premise appears as a free variable. This is known as k-like reduction, and is permitted because all elements of a subsingleton eliminator are identical. 
-->

「subsingleton eliminator」として知られているいくつかの帰納型では、major premise のコンストラクタが直接公開されていなくても、その型が分かっている限りι簡約を進めることができます。これは例えば major premise が自由変数として現れるような場合です。これは k-like 簡約と呼ばれ、subsingleton eliminator のすべての要素が同一であるため許容されます。

<!--
To be a subsingleton eliminator, an inductive declaration must be an inductive prop, must not be a mutual or nested inductive, must have exactly one constructor, and the sole constructor must take only the type's parameters as arguments (it cannot "hide" any information that isn't fully captured in its type signature).
-->

subsingleton eliminator であるためには、帰納的宣言は帰納的な命題でなければならず、また相互帰納的対象や入れ子の帰納的対象であってはならず、コンストラクタはちょうど1つでなければならず、その唯一のコンストラクタは型のパラメータだけを引数として取らなければなりません（型シグネチャで完全に捕捉されていない情報を「隠す」ことはできません）。

<!--
For example, the value of any element of the type `Eq Char 'x'` is fully determined just by its type, because all elements of this type are identical.
-->

例えば、`Eq Char 'x'` 型の要素の値はこの型のすべての要素は同一であるため、その型だけで完全に決定されます。

<!--
If iota reduction finds a major premise which is a subsingleton eliminator, it is permissible to substitute the major premise for an application of the type's constructor, because that is the only element the free variable could actually be. For example, a major premise which is a free variable of type `Eq Char 'a'` may be substituted for an explicitly constructed `Eq.refl Char 'a'`.
-->

ι簡約によって subsingleton eliminator である major premise が見つかった場合、その major premise を型コンストラクタの適用に置換することが許容されます。なぜならその自由変数としてあり得るのがこの型コンストラクタだけだからです。例えば、`Eq Char 'a'` 型の自由変数である major premise は明示的に構成された `Eq.refl Char 'a'` に置換することができます。

<!--
Getting to the nuts and bolts, if we neglected to look for and apply k-like reduction, free variables that are subsingleton eliminators would fail to identify the corresponding recursor rule, iota reduction would fail, and certain conversions expected to succeed would no longer succeed.
-->

根本的な話として、もし k-like 簡約を探索して適用することを怠れば、subsingleton eliminator である自由変数は対応する再帰規則を特定できず、ι簡約は失敗し、成功すると期待されたある種の変換は成功しなくなるでしょう。

<!--
### `Quot` reduction; `Quot.ind` and `Quot.lift`
-->

### `Quot` の簡約；`Quot.ind` と `Quot.lift`

<!--
`Quot` introduces two special cases which need to be handled by the kernel, one for `Quot.ind`, and one for `Quot.lift`.
-->

`Quot` はカーネルでハンドルするべき2つの特殊なケースを導入します。1つは `Quot.ind` で、もう一つは `Quot.lift` の場合です。

<!--
Both `Quot.ind` and `Quot.lift` deal with application of a function `f` to an argument `(a : α)`, where the `a` is a component of some `Quot r`, formed with `Quot.mk .. a`. 
-->

`Quot.ind` と `Quot.lift` はどちらも関数 `f` の引数 `(a : α)` への適用に対応しており、ここで `a` は `Quot r` の構成要素であり、`Quot.mk .. a` によって形成されます。

<!--
To execute the reduction, we need to pull out the argument that is the `f` element and the argument that is the `Quot` where we can find `(a : α)`, then apply the function `f` to `a`. Finally, we reapply any arguments that were part of some outer expression not related to the invocation of `Quot.ind` or `Quot.lift`.
-->

この簡約を実行するには、`f` の要素である引数と、`(a : α)` が見つかる `Quot` 要素である引数を取り出し、`a` に関数 `f` を適用する必要があります。最後に、`Quot.ind` や `Quot.lift` の呼び出しとは関係のない外部式の一部であった引数を再適用します。

<!--
Since this is only a reduction step, we rely on the type checking phases done elsewhere to provide assurances that the expression as a whole is well typed.
-->

これは簡約のステップにすぎないため、式全体が適切に型付けされていることを保証するために他の場所で行われる型チェックの段階に頼ります。

<!--
The type signatures for `Quot.ind` and `Quot.mk` are recreated below, mapping the elements of the telescope to what we should find as the arguments. The elements with a `*` are the ones we're interested in for reduction.
-->

以下に `Quot.ind` と `Quot.mk` の型シグネチャを再作成し、テレスコープの要素を引数として見つけるべきものにマッピングしています。`*` が付いている要素は簡約のために必要なものです。

```
Quotient primitive Quot.ind.{u} : ∀ {α : Sort u} {r : α → α → Prop} 
  {β : Quot r → Prop}, (∀ (a : α), β (Quot.mk r a)) → ∀ (q : Quot r), β q

  0  |-> {α : Sort u} 
  1  |-> {r : α → α → Prop} 
  2  |-> {β : Quot r → Prop}
  3* |-> (∀ (a : α), β (Quot.mk r a)) 
  4* |-> (q : Quot r)
  ...
```

```
Quotient primitive Quot.lift.{u, v} : {α : Sort u} →
  {r : α → α → Prop} → {β : Sort v} → (f : α → β) → 
  (∀ (a b : α), r a b → f a = f b) → Quot r → β

  0  |-> {α : Sort u}
  1  |-> {r : α → α → Prop} 
  2  |-> {β : Sort v} 
  3* |-> (f : α → β) 
  4  |-> (∀ (a b : α), r a b → f a = f b)
  5* |-> Quot r
  ...
```
