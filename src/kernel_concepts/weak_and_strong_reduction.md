<!--
# Weak and strong reduction
-->

# 弱い簡約と強い簡約

<!--
The implementation details of Lean's reduction strategies is discussed in [another chapter](../type_checking/reduction.md); this section is specifically to clarify the difference between the general concepts of weak and strong reduction.
-->

Lean の簡約戦略の実装詳細については [別の章](../type_checking/reduction.md) で説明します；本節は、特に弱い簡約と強い簡約の一般的な概念の違いを明確にするためのものです。

<!--
## Weak reduction
-->

## 弱い簡約

<!--
Weak reduction refers to reduction that stops at binders which do not have an argument applied to them. By binders, we mean lambda, pi, and let expressions.
-->

弱い簡約とは、引数が適用されていない束縛子までで止まる簡約です。束縛子はラムダ・pi・let 式のことです。

<!--
For example, weak reduction can reduce `(fun (x y : Nat) => y + x) (0 : Nat)` to `(fun (y : Nat) => y + 0)`, but can do no further reduction.
-->

例えば、弱い簡約によって `(fun (x y : Nat) => y + x) (0 : Nat)` は `(fun (y : Nat) => y + 0)` に簡約できますが、これ以上の簡約はできません。

<!--
When we say or 'weak head normal form reduction', or just reduction without specifically identifying it as 'strong', we're talking about weak reduction. Strong reduction just happens as a byproduct of applying weak reduction after we've opened a binder somewhere else. 
-->

「弱頭正規形への簡約」や「強い」と特に特定せずに単に簡約と言ったりする場合は、弱い簡約のことを指します。強い簡約はどこかで束縛子を開いた後に弱い簡約を適用した副産物として発生します。

<!--
## Strong reduction
-->

## 強い簡約

<!--
Strong reduction refers to reduction under open binders; when we run across a binder without an accompanying argument (like a lambda expression with no `app` node applying an argument), we can traverse into the body and potentially do further reduction by creating and substituting in a free variable. Strong reduction is needed for type inference and definitional equality checking. For type inference, we also need the ability to "re-close" open terms, replacing free variables with the correct bound variables afer some reduction has been done in the body. This is not as simple as just replacing it with the same bound variable as before, because bound variables may have shifted, invalidating their old deBruijn index relative to the new rebuilt expression.
-->

強い簡約とは開いた束縛子のもとでの簡約を指します；対応する引数が無い束縛子（引数を適用する `app` ノードが無いラムダ式のようなもの）に遭遇した際、本体を走査して自由変数を作成して代入することでさらに簡約できる可能性があります。強い簡約は型推論と定義の定義上の同値のチェックに必要です。型推論にあたっては、本体で簡約が行われた後に自由変数を正しい束縛変数に置き換える、開いた項を「再度閉じる」機能も必要です。これは単に以前と同じ束縛変数に置き換えればよいというような単純なものではありません。なぜなら束縛変数がシフトし、新しく再構築された式に対する古い de Bruijn インデックスが無効になっている可能性があるからです。

<!--
As with weak reduction, strong reduction can stil reduce `(fun (x y : Nat) => y + x) (0 : Nat)` to `(fun (y : Nat) => y + 0)`, and instead of getting stuck, it can continue by substituting `y` for a free variable, reducing the expression further to `((fVar id, y, Nat) + 0)`, and `(fvar id, y, Nat)`. 
-->

弱い簡約と同様に、強い簡約でも `(fun (x y : Nat) => y + x) (0 : Nat)` は `(fun (y : Nat) => y + 0)` に簡約され、そこで詰まることなく `y` を自由変数に置き換えて簡約を続行し、式は `((fVar id, y, Nat) + 0)` と `(fvar id, y, Nat)` へと簡約できます。

<!--
As long as we keep the free variable information around _somewhere_, we can re-combine that information with the reduced `(fVar id, y, Nat)` to recreate `(fun (y : Nat) => bvar(0))`
-->

自由変数の情報を **どこか** に残しておく限り、その情報を簡約された `(fVar id, y, Nat)` と組み合わせることで `(fun (y : Nat) => bvar(0))` へと再作成することができます。
