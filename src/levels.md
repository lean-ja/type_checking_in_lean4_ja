<!--
# Universe levels
-->

# 宇宙レベル

<!--
This section will describe universe levels from an implementation perspective, and will cover what readers need to know when it comes to type checking Lean declarations. More in-depth treatment of their role in Lean's type theory can be found in [TPIL4](https://lean-lang.org/theorem_proving_in_lean4/dependent_type_theory.html#types-as-objects), or section 2 of [Mario Carneiro's thesis](https://github.com/digama0/lean-type-theory)
-->

本節では、実装の観点から宇宙レベルについて説明し、読者が Lean の宣言を型チェックする際に知っておくべきことを取り上げます。Lean の型理論における宇宙レベルの役割についてのより深い説明は [TPIL4](https://lean-lang.org/theorem_proving_in_lean4/dependent_type_theory.html#types-as-objects) [^fn1] や [Mario Carneiro の論文](https://github.com/digama0/lean-type-theory) の第二節を参照してください。

<!--
The syntax for universe levels is as follows:
-->

宇宙レベルの構文は以下のようになっています：

```
Level ::= Zero | Succ Level | Max Level Level | IMax Level Level | Param Name
```

<!--
Properties of the `Level` type that readers should take note of are the existence of a partial order on universe levels, the presence of variables (the `Param` constructor), and the distinction between `Max` and `IMax`. 
-->

読者が注意すべき `Level` 型の特徴は、宇宙レベルに半順序が存在すること、変数の存在（`Param` コンストラクタ）、そして `Max` と `IMax` の区別です。

<!--
`Max` simply constructs a universe level that represents the larger of the left and right arguments. For example, `Max(1, 2)` simplifies to `2`, and `Max(u, u+1)` simplifies to `u+1`. The `IMax` constructor represents the larger of the left and right arguments, *unless* the right argument simplifies to `Zero`, in which case the entire `IMax` resolves to `0`.
-->

`Max` は単純に左右の引数のうち大きい方を表す宇宙レベルを構築します。例えば、`Max(1, 2)` は `2` に単純化され、`Max(u, u+1)` は `u+1` に単純化されます。`IMax` コンストラクタは左と右の引数のうち大きい方を表しますが、これは右の引数が `Zero` に単純 **されない場合に限り** 、またこの場合は `IMax` は `0` へと解決されます。

<!--
The important part about `IMax` is its interaction with the type inference procedure to ensure that, for example, `forall (x y : Sort 3), Nat` is inferred as `Sort 4`, but `forall (x y : Sort 3), True` is inferred as `Prop`.
-->

`IMax` の重要な点は、例えば `forall (x y : Sort 3), Nat` は `Sort 4` と推論されるが、`forall (x y : Sort 3), True` は `Prop` と推論されるように型推論の処理と相互作用することにあります。

<!--
## Partial order on levels
-->

## レベル上の半順序

<!--
Lean's `Level` type is equipped with a partial order, meaning there's a "less than or equals" test we can perform on pairs of levels. The rather nice implementation below comes from Gabriel Ebner's Lean 3 checker [trepplein](https://github.com/gebner/trepplein/tree/master). While there are quite a few cases that need to be covered, the only complex matches are those relying on `cases`, which checks whether `x ≤ y` by examining whether `x ≤ y` holds when a parameter `p` is substituted for `Zero`, and when `p` is substituted for `Succ p`.
-->

Lean の `Level` 型は半順序を備えています。つまりレベルのペアに対して実行可能な「以下」の検査が存在します。以下のなかなか素晴らしい実装は Gabriel Ebner の Lean 3 チェッカである [trepplein](https://github.com/gebner/trepplein/tree/master) から持ってきています。カバーしなければならないケースはかなり多いですが、複雑なマッチは `cases` に依存しているものだけです。これは `x ≤ y` であるかどうかのチェックについて、パラメータ `p` に `Zero` が代入された時、および `p` に `Succ p` が代入されたときに `x ≤ y` が成り立つかどうか調べることで行っています。

```
  leq (x y : Level) (balance : Integer): bool :=
    Zero, _ if balance >= 0 => true
    _, Zero if balance < 0 => false
    Param(i), Param(j) => i == j && balance >= 0
    Param(_), Zero => false
    Zero, Param(_) => balance >= 0
    Succ(l1_), _ => leq l1_ l2 (balance - 1)
    _, Succ(l2_) => leq l1 l2_ (balance + 1)

    -- descend left
    Max(a, b), _ => (leq a l2 balance) && (leq b l2 balance)

    -- descend right
    (Param(_) | Zero), Max(a, b) => (leq l1 a balance) || (leq l1 b balance)

    -- imax
    IMax(a1, b1), IMax(a2, b2) if a1 == a2 && b1 == b2 => true
    IMax(_, p @ Param(_)), _ => cases(p)
    _, IMax(_, p @ Param(_)) => cases(p)
    IMax(a, IMax(b, c)), _ => leq Max(IMax(a, c), IMax(b, c)) l2 balance
    IMax(a, Max(b, c)), _ => leq (simplify Max(IMax(a, b), IMax(a, c))) l2 balance
    _, IMax(a, IMax(b, c)) => leq l1 Max(IMax(a, c), IMax(b, c)) balance
    _, IMax(a, Max(b, c)) => leq l1 (simplify Max(IMax(a, b), IMax(a, c))) balance


  cases l1 l2 p: bool :=
    leq (simplify $ subst l1 p zero) (simplify $ subst l2 p zero)
    ∧
    leq (simplify $ subst l1 p (Succ p)) (simplify $ subst l2 p (Succ p))
```

<!--
## Equality for levels
-->

## レベルの同値

<!--
The `Level` type recognizes equality by antisymmetry, meaning two levels `l1` and `l2` are equal if `l1 ≤ l2` and `l2 ≤ l1`.
-->

`Level` 型では反対称律による同値が認められています。つまり、2つのレベル `l1` と `l2` は `l1 ≤ l2` かつ `l2 ≤ l1` ならば等しいです。

<!--
# Implementation notes
-->

# 実装上の注記

<!--
Be aware that the exporter does not export `Zero`, but it is assumed to be the 0th element of `Level`.
-->

エクスポータは `Zero` をエクスポートしないことに注意してください。しかし、これは `Level` の0番目の要素として仮定されます。

<!--
For what it's worth, the implementation of `Level` does not have a large impact on performance, so don't feel the need to aggressively optimize here.
-->

ただこう言ってはなんですが、`Level` の実装はパフォーマンスに大きな影響を与えないため、ここで積極的に最適化する必要はありません。

[^fn1]: 訳注：日本語訳は <https://aconite-ac.github.io/theorem_proving_in_lean4_ja/dependent_type_theory.html#types-as-objects-%E9%A0%85%E3%81%A8%E3%81%97%E3%81%A6%E3%81%AE%E5%9E%8B>