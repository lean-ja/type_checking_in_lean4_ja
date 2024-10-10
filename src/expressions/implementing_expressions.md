<!--
# Implementing Expressions
-->

# 式の実装

<!--
A few noteworthy points about the expression type for readers interested in writing their own kernel in whole or in part...
-->

カーネルを一部もしくは全部書いてみたい読者のために、式の型について特筆すべき点をいくつか挙げておくと……

<!--
## Stored data
-->

## データの格納

<!--
Expressions need to store some data inline or cache it somewhere to prevent prohibitively expensive recomputation. For example, creating an expression `app x y` needs to calculate and then store the hash digest of the resulting app expression, and it needs to do so by getting the cached hash digests of `x` and `y` instead of traversing the entire tree of `x` to recursively calculate the digest of `x`, then doing the same for `y`.
-->

式は法外に高価な再計算を防ぐために、ある程度のデータをインラインで保持するか、どこかにキャッシュする必要があります。例えば、`app x y` という式を作成する場合、結果の app 式のハッシュダイジェストを計算し保存する必要があります。その際には `x` と `y` の木全体を走査して `x` と `y` のダイジェストをそれぞれ再帰的に計算するのではなく、キャッシュされた `x` と `y` のハッシュダイジェストを取得する必要があります。

<!--
The data you will probably want to store inline are the hash digest, the number of loose bound variables in an expression, and whether or not the expression has free variables. The latter two are useful for optimizing instantiation and abstraction respectively.
-->

おそらくインラインで保存したくなるデータはハッシュダイジェスト・式中の loose な束縛変数の数・式に自由変数があるかどうかです。後者の2つはそれぞれインスタンス化と抽象化を最適化するにあたって便利です。

<!--
An example "smart constructor" for `app` expressions would be:
-->

`app` 式の「スマートなコンストラクタ」は次の例のようになるでしょう：

```
def mkApp x y:
  let hash := hash x.storedHash y.storedHash
  let numLooseBvars := max x.numLooseBvars y.numLooseBvars
  let hasFvars := x.hasFvars || y.hasFvars
  .app x y (cachedData := hash numLooseBVars hasFVars)
```

<!--
## No deep copies
-->

## ディープコピーをしない

<!--
Expressions should be implemented such that child expressions used to construct some parent expression are not deep copied. Put another way, creating an expression `app x y` should not recursively copy the elements of `x` and `y`, rather it should take some kind of reference, whether it's a pointer, integer index, reference to a garbage collected object, reference counted object, or otherwise (any of these strategies should deliver acceptable performance). If your default strategy for constructing expressions involves deep copies, you will not be able to construct any nontrivial environments without consuming massive amounts of memory.
-->

式は、親の式を構築するために使用する子の式をディープコピーしないように実装されるべきです。別の言い方をすれば、`app x y` という式を作成する場合、`x` と `y` の要素を再帰的にコピーするべきではありません。むしろポインタ・整数のインデックス・ガベージコレクションされるオブジェクトへの参照・参照カウントの対象のオブジェクトなど、なんらかの参照を取るべきです（これらの戦略のいずれでも許容可能なパフォーマンスを提供する必要があります）。式を構築するためのデフォルトの戦略にディープコピーが含まれてしまうと、自明でない環境を構築するために大量のメモリを消費しなければならなくなります。

<!--
## Example implementation for number of loose bound variables
-->

## loose な束縛変数の数の実装例

```
numLooseBVars e:
    match e with
    | Sort | Const | FVar | StringLit | NatLit => 0
    | Var dbjIdx => dbjIdx + 1,
    | App fun arg => max fun.numLooseBvars arg.numLooseBvars
    | Pi binder body | Lambda binder body => 
    |   max binder.numLooseBVars (body.numLooseBVars - 1)
    | Let binder val body =>
    |   max (max binder.numLooseBvars val.numLooseBvars) (body.numLooseBvars - 1)
    | Proj _ _ structure => structure.numLooseBvars
```

<!--
For `Var` expressions, the number of loose bound variables is the deBruijn index plus one, because we're counting the number of binders that would need to be applied for that variable to no longer be loose (the `+1` is because deBruijn indices are 0-based). For the expression `Var(0)`, one binder needs to be placed above the bound variable in order for the variable to no longer be loose. For `Var(3)`, we need four:
-->

`Var` 式において、loose な束縛変数の数は de Bruijn インデックスに1を足した値になります。これは、その変数が loose でなくなるために必要な束縛子の数を数えているからです（`+1` は de Bruijn インデックスが0始まりであるからです）。式 `Var(0)` の場合、その変数が loose でなくなるためには、束縛子を束縛変数の上に1つ置く必要があります。`Var(3)` の場合は4つ必要です：

```
--  3 2 1 0
fun a b c d => Var(3)
```

<!--
When we create a new binder (lambda, pi, or let), we can subtract 1 (using saturating/natural number subtraction) from the number of loose bvars in the body, because the body is now under one additional binder.
-->

新しい束縛子（ラムダ式・pi・let式）を作るとき、本体の loose な bvar の数から1を引きます（自然数の飽和減算を使います）。なぜなら本体は1つの追加の束縛子の下に置かれるからです。
