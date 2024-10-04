<!--
# Clarifying language
-->

# 言語の明確化

<!--
## Type, Sort, Prop
-->

## 型・ソート・命題

<!--
`Prop` refers to `Sort 0`
-->

`Prop` は `Sort 0` を指します。

<!--
`Type n` refers to `Sort (n+1)`
-->

`Type n` は `Sort (n+1)` を指します。

<!--
`Sort n` is how these are actually represented in the kernel, and can always be used.
-->

`Sort n` はカーネルでの実際の表現で、常に使用することができます。

<!--
The reason why `Type <N>` and `Prop` are sometimes used instead of always adhering to `Sort n` is that elements of `Type <N>` have certain important qualities and behaviors that are not observed by those of `Prop` and vice versa.
-->

時折 `Type <N>` と `Prop` が常に `Sort n` に従う代わりに使われることがあるのは、`Type <N>` の要素には `Prop` の要素では観察されない重要な性質や振る舞いがあり、その逆もまたあるからです。

<!--
Example: elements of `Prop` can be considered for definitional proof irrelevance, while elements of `Type _` can use large elimination without needing to satisfy other tests.
-->

例：`Prop` の要素は定義の証明の無関係性を考慮することができますが、`Type _` の要素は別でテストに合格することなく large elimination を使用することができます。

<!--
## Level/universe and Sort
-->

## レベル・宇宙とソート

<!--
The terms "level" and "universe" are basically synonymous; they refer to the same kind of kernel object.
-->

「レベル」と「宇宙」という用語は基本的には同義語です；これらは同じ種類のカーネルのオブジェクトを指します。

<!--
A small distinction that's sometimes made is that "universe parameter" may be implicitly restricted to levels that are variables/parameters. This is because "universe parameters" refers to the set of levels that parameterize a Lean declaration, which can only be identifiers, and are therefore restricted to identifiers. If this doesn't mean anything to you, don't worry about it for now. As an example, a Lean declaration may begin with `def Foo.{u} ..` meaning "a definition parameterized by the universe variable `u`", but it may not begin with `def Foo.{1} ..`, because `1` is an explicit level, and not a parameter.
-->

小さな違いとして、「宇宙パラメータ」は暗黙裡に変数・パラメータであるレベルに制限されることがあります。これは「宇宙パラメータ」が Lean の宣言をパラメータ化するレベルのあつまりを指しますが、これは識別子にしかなり得ないことから識別子に限定されます。もしこれが読者にとって意味が無くても今は気にしないでください。例として、ある Lean の宣言が `def Foo.{u} ..` ではじまると、これは「宇宙変数 `u` によってパラメータ化された定義」を意味しますが、これを `def Foo.{1} ..` で始めることはできません。なぜなら `1` は明示的なレベルであり、パラメータではないからです。

<!--
On the other hand, `Sort _` is an expression that represents a level.
-->

一方で、`Sort _` はレベルを表現した式のことです。

<!--
## Value vs. type
-->

## 値 vs 型

<!--
Expressions can be either values or types. Readers are probably familiar with the idea that `Nat.zero` is a value, while `Nat` is a type. An expression `e` is a value or "value level expression" if `infer e != Sort _`. An expression `e` is a type or "type level expression" if `infer(e) = Sort _`.
-->

式は値にも型にもなります。おそらく読者は `Nat.zero` が値である一方で `Nat` が型であることにはなじみがあるでしょう。もし `infer e != Sort _` ならば、式 `e` は「値レベルの式」です。もし `infer(e) = Sort _` ならば、式 `e` は型か「型レベルの式」です。

<!--
## Parameters vs. indices
-->

## パラメータ vs 添字

<!--
The distinction between a parameter and index comes into play when dealing with inductive types. Roughly speaking, elements of a telescope that come before the colon in a declaration are parameters, and elements that come after the colon are indices:
-->

パラメータと添字の区別は帰納型を扱う際に出てきます。大雑把に言えば、宣言のコロンの前にはめ込まれた要素はパラメータで、コロンの後に来る要素は添字です：

```
      parameter ----v         v---- index
inductive Nat.le (n : Nat) : Nat → Prop
```

<!--
The distinction is non-negligible within the kernel, because parameters are fixed within a declaration, while indices are not.
-->

この違いはカーネルに無視されません。というのもパラメータは宣言内で固定されますが、添字は固定されないからです。
