<!--
# Definitional equality
-->

# 定義上の同値

<!--
Definitional equality is implemented as a function that takes two expressions as input, and returns `true` if the two expressions are definitionally equal within Lean's theory, and `false` if they are not.
-->

定義上の同値は関数として実装されています。これは2つの式を受け取り、それらが Lean の理論の中で定義上等しい場合は `true` を、そうでない場合は `false` を返します。

<!--
Within the kernel, definitional equality is important simply because it's a necessary part of type checking. Definitional equality is still an important concept for Lean users who do not venture into the kernel, because definitional equalities are comparatively nice to work with in Lean's vernacular; for any `a` and `b` that are definitionally equal, Lean doesn't need any prompting or additional input from the user to determine whether two expressions are equal.
-->

カーネル内に置いて定義上の同値は重要なものですが、それは単に型チェックに必要だからです。またカーネル内部へ挑まない Lean ユーザにとっても定義上の同値は重要な概念です。というのも定義上の同値は Lean を書く上で比較的簡単に扱えるからです；定義上等しい `a` と `b` に対して、Lean は2つの式が等しいかどうかを判断するためにユーザからのプロンプトや追加の入力を必要としません。

<!--
There are two big-picture parts of implementing the definitional equality procedure. First, the individual tests that are used to check for different definitional equalities. For readers who are just interested in understanding definitional equality from the perspective of an end user, this is probably what you want to know.
-->

定義上の同値の処理を実装には大きく分けて2つの部分があります。まず、定義が等しいかどうかをチェックするための個別のテストです。エンドユーザの視点から定義上の同値を理解することに興味がある読者にとっては、おそらくこれが知りたいことでしょう。

<!--
Readers interested in writing a type checker should also understand how the individual checks are composed along with reduction and caching to make the problem tractable; naively running each check and reducing along the way is likely to yield unacceptable performance results.
-->

型チェッカを書くことに興味がある読者は、問題を扱いやすくするために個々のチェックが簡約やキャッシュに対してどのように組み合わされるかについても理解しておく必要があります；素朴に各チェックを実行し、それに従い簡約を行うことは受け入れがたいパフォーマンスとなってしまう可能性が高いです。

<!--
## Sort equality
-->

## ソートの同値

<!--
Two `Sort` expressions are definitionally equal if the levels they represent are equal by antisymmetry using the partial order on levels. 
-->

2つの `Sort` 式が定義上等しいのは、それらが表すレベルがレベルの半順序関係の反対称律によって等しい場合です。

```
defEq (Sort x) (Sort y):
  x ≤ y ∧ y ≤ x
```

<!--
## Const equality
-->

## 定数の同値

<!--
Two `Const` expressions are definitionally equal if their names are identical, and their levels are equal under antisymmetry.
-->

2つの `Const` 式が定義上等しいのは、名前が同一で、レベルが反対称律の下で等しい場合です。

<!--
```
defEq (Const n xs) (Const m ys):
  n == m ∧ forall (x, y) in (xs, ys), antisymmEq x y

  -- also assert that xs and ys have the same length if your `zip` doesn't do so.
```
-->
```
defEq (Const n xs) (Const m ys):
  n == m ∧ forall (x, y) in (xs, ys), antisymmEq x y

  -- もし利用する `zip` がよしなにしてくれない場合は、xs と ys が同じ長さであることも主張します
```

<!--
## Bound Variables
-->

## 束縛変数

<!--
For implementations using a substitution-based strategy like locally nameless (if you're following the C++ or lean4lean implementations, this is you), encountering a bound variable is an error; bound variables should have been replaced during weak reduction if they referred to an argument, or they should have been replaced with a free variable via strong reduction as part of a definitional equality check for a pi or lambda expression.
-->

locally nameless のような置換ベースの戦略を使用している場合（もし C++ や lean4lean の実装に従っているならあなたのことです）、束縛変数に遭遇するとエラーになります；というのも束縛変数はそれが引数を参照している場合、弱い簡約を行う中で置換されるか、pi またはラムダ式の定義上の同値チェックの一部として強い簡約を介して自由変数に置換されるべきです。

<!--
For closure-based implementations, look up the elements corresponding to the bound variables and assert that they are definitionally equal.
-->

クロージャベースの実装では、束縛変数に対応する要素を調べ、それらが定義上等しいことを保証します。

<!--
## Free Variables
-->

## 自由変数

<!--
Two free variables are definitionally equal if they have the same identifier (unique ID or deBruijn level). Assertions about the equality of the binder types should have been performed wherever the free variables were constructed (like the definitional equality check for pi/lambda expressions), so it is not necessary to re-check that now.
-->

2つの自由変数が定義上等しいのは、同じ識別子（一意な ID か de Bruijn レベル）を持っている場合です。束縛子の型が等しいかどうかの検証は、（pi ・ラムダ式の定義上の同値チェックと同じように）自由変数が構築された場所で必ず行われたはずなので、今更再チェックする必要はありません。

```
defEqFVar (id1, _) (id2, _):
  id1 == id2
```

<!--
## App
-->

## 関数適用

<!--
Two application expressions are definitionally equal if their function component and argument components are definitionally equal.
-->

2つの関数適用の式が定義上等しいのは、関数のコンポーネントと引数のコンポーネントが定義上等しい場合です。

```
defEqApp (App f a) (App g b):
  defEq f g && defEq a b
```


## Pi

<!--
Two Pi expressions are definitionally equal if their binder types are definitionally equal, and their body types, after substituting in the appropriate free variable, are definitionally equal.
-->

2つの Pi の式が定義上等しいのは、束縛子の型が定義上等しく、適切な自由変数を代入した後の本体の型が定義上等しい場合です。

```
defEq (Pi s a) (Pi t b)
  if defEq s.type t.type
  then
    let thisFvar := fvar s
    defEq (inst a thisFvar) (inst b thisFvar)
  else
    false
```

<!--
## Lambda
-->

## ラムダ式

<!--
Lambda uses the same test as Pi:
-->

ラムダ式では Pi と同じチェックを用います：

```
defEq (Lambda s a) (Lambda t b)
  if defEq s.type t.type
  then
    let thisFvar := fvar s
    defEq (inst a thisFvar) (inst b thisFvar)
  else
    false
```

<!--
## Structural eta
-->

## 構造的なη

<!--
Lean recognizes definitional equality of two elements `x` and `y` if they're both instances of some structure type, and the fields are definitionally equal using the following procedure comparing the constructor arguments of one and the projected fields of the other:
-->

Lean は2つの要素 `x` と `y` が両方ともある構造体型のインスタンスであり、一方のコンストラクタ引数と他方の射影されたフィールドを比較する以下の手順を使用してフィールドが定義上等しい場合、定義上の同値を認めます：

<!--
```
defEqEtaStruct x y:
  let (yF, yArgs) := unfoldApps y
  if 
    yF is a constructor for an inductive type `T` 
    && `T` can be a struct
    && yArgs.len == T.numParams + T.numFields
    && defEq (infer x) (infer y)
  then
    forall i in 0..t.numFields, defEq Proj(i+T.numParams, x) yArgs[i+T.numParams]

    -- we add `T.numParams` to the index because we only want 
    -- to test the non-param arguments. we already know the 
    -- parameters are defEq because the inferred types are 
    -- definitionally equal.
```
-->
```
defEqEtaStruct x y:
  let (yF, yArgs) := unfoldApps y
  if 
    yF is a constructor for an inductive type `T` 
    && `T` can be a struct
    && yArgs.len == T.numParams + T.numFields
    && defEq (infer x) (infer y)
  then
    forall i in 0..t.numFields, defEq Proj(i+T.numParams, x) yArgs[i+T.numParams]

    -- `T.numParams` をインデックスに追加しているのは、
    -- パラメータ以外の引き数のみをテストしたいからです。
    -- 推論される型が定義上等しいため、パラメータが defEq であることはすでに分かっています。
```

<!--
The more pedestrian case of congruence `T.mk a .. N` = `T.mk x .. M` if `[a, .., N] = [x, .., M]`, is simply handled by the `App` test.
-->

より一般的な例である `[a, .., N] = [x, .., M]` の時の合同 `T.mk a .. N` = `T.mk x .. M` は、単純に `App` のチェックで処理できます。

<!--
## Unit-like equality
-->

## ユニットのようなものの同値

<!--
Lean recognizes definitional equality of two elements `x: S p_0 .. p_N` and `y: T p_0 .. p_M` under the following conditions:
-->

Lean では以下の条件の下にある2つの要素 `x: S p_0 .. p_N` と `y: T p_0 .. p_M` が定義上等しいとしています：

<!--
+ `S` is an inductive type
+ `S` has no indices
+ `S` has only one constructor which takes no arguments other than the parameters of `S`, `p_0 .. p_N`
+ The types `S p_0 .. p_N` and `T p0 .. p_M` are definitionally equal
-->

+ `S` は帰納型である
+ `S` は添字を持たない
+ `S` はただ一つのコンストラクタを持ち、それが `S` のパラメータ `p_0 .. p_N` 以外に引数を取らないこと
+ 型 `S p_0 .. p_N` と `T p0 .. p_M` が定義上等しいこと

<!--
Intuitively this definitional equality is fine, because all of the information that elements of these types can convey is captured by their types, and we're requiring those types to be definitionally equal.
-->

直観的にこの定義上の同値は良いでしょう。なぜなら、これらの型の要素が伝えることのできる情報はすべて型に捕捉されており、私たちはこれらの型が定義上等しいことを要求しているからです。

<!--
## Eta expansion
-->

## η 展開

```
defEqEtaExpansion x y : bool :=
  match x, (whnf $ infer y) with
  | Lambda .., Pi binder _ => defEq x (App (Lambda binder (Var 0)) y)
  | _, _ => false
```

<!--
The lambda created on the right, `(fun _ => $0) y` trivially reduces to `y`, but the addition of the lambda binder gives the `x` and `y'` a chance to match with the rest of the definitional equality procedure.
-->

右辺でラムダ式が作られており、この `(fun _ => $0) y` は明らかに `y` に簡約されますが、ラムダ式の束縛子を追加することで、 `x` と `y` が残りの定義上の同値の手続きによる一致の機会を与えます。

<!--
## Proof irrelevant equality
-->

# proof irrelevance な同値

<!--
Lean treats proof irrelevant equality as definitional. For example, Lean's definitional equality procedure treats any two proofs of `2 + 2 = 4` as definitionally equal expressions.
-->

Lean は proof irrelevance な同値を定義上のものとして扱います。例えば、Lean の定義上の同値の手続きは `2 + 2 = 4` についての任意の2つの証明を定義上同値として扱います。

If a type `T` infers as `Sort 0`, we know it's a proof, because it is an element of `Prop` (remember that `Prop` is `Sort 0`).

```
defEqByProofIrrelevance p q :
  infer(p) == S ∧ 
  infer(q) == T ∧
  infer(S) == Sort(0) ∧
  infer(T) == Sort(0) ∧
  defEq(S, T)
```

<!--
If `p` is a proof of type `A` and `q` is a proof of type `B`, then if `A` is definitionally equal to `B`, `p` and `q` are definitionally equal by proof irrelevance.
-->

`p` が `A` 型の証明で、`q` が `B` 型の証明である場合、`A` が `B` と定義上等しいなら、`p` と `q` は proof irrelevance によって定義上等しいです。

<!--
## Natural numbers (nat literals)
-->

## 整数（整数リテラル）

<!--
Two nat literals are definitionally equal if they can be reduced to `Nat.zero`, or they can be reduced as (`Nat.succ x`, `Nat.succ y`), where `x` and `y` are definitionally equal.
-->

2つの整数リテラルが定義上等しいのは、それらが `Nat.zero` に簡約できる場合か、(`Nat.succ x`, `Nat.succ y`) へと簡約できる場合で、ここで `x` と `y` は定義上等しいです。

```
match X, Y with
| Nat.zero, Nat.zero => true
| NatLit 0, NatLit 0 => true
| Nat.succ x, NatLit (y+1) => defEq x (NatLit y)
| NatLit (x+1), Nat.succ y => defEq (NatLit x) y
| NatLit (x+1), NatLit (y+1) => x == y
| _, _ => false
```

<!--
## String literal
-->

## 文字列リテラル

`StringLit(s), App(String.mk, a)`

<!--
The string literal `s` is converted to an application of `Const(String.mk, [])` to a `List Char`. Because Lean's `Char` type is used to represent unicode scalar values, their integer representation is a 32-bit unsigned integer.
-->

文字列リテラル `s` は `Const(String.mk, [])` の適用によって `List Char` に変換されます。Lean の `Char` 型は Unicode スカラー値を表現するために使用されるため、その整数表現は32ビットの符号なし整数です。

<!--
To illustrate, the string literal "ok", which uses two characters corresponding to the 32 bit unsigned integers `111` and `107` is converted to:
-->

例として、32ビットの符号なし整数 `111` と `107` に対応する2つの文字を使った文字列リテラル「ok」は次のように変換されます。

>(String.mk (((List.cons Char) (Char.ofNat.[] NatLit(111))) (((List.cons Char) (Char.ofNat NatLit(107))) (List.nil Char))))

<!--
## Lazy delta reduction and congruence
-->

## Lazy delta 簡約

<!--
The available kernel implementations implement a "lazy delta reduction" procedure as part of the definitional equality check, which unfolds definitions lazily using [reducibility hints](./declarations.md#reducibility-hints) and checks for congruence when things look promising. This is a much more efficient strategy than eagerly reducing both expressions completely before checking for definitional equality.
-->

利用可能なカーネル実装は [簡約のヒント](./declarations.md#reducibility-hints) を使用して遅延的に定義を展開し、物事が有望に見えるときに一致をチェックする定義上の同値チェックの一部として「Lazy delta 簡約」手順を実装しています。これは定義が一致するかどうかをチェックする前に両方の式を完全に簡約するよりもはるかに効率的な戦略です。

<!--
If we have two expressions `a` and `b`, where `a` is an application of a definition with height 10, and `b` is an application of a definition with height 12, the lazy delta procedure takes the more efficient route of unfolding `b` to try and get closer to `a`, as opposed to unfolding both of them completely, or blindly choosing one side to unfold.
-->

`a` と `b` という2つの式があり、`a` が高さ10の定義の適用で、`b` が高さ12の定義の適用である場合、両者を完全に展開したりやみくもにどちらか片方を展開するのではなく、Lazy delta 手順は `b` を展開して `a` に近づけようとするより効率的なルートを取ります。

<!--
If the lazy delta procedure finds two expressions which are an application of a `const` expression to arguments, and the `const` expressions refer to the same declaration, the expressions are checked for congruence (whether they're the same consts applied to definitionally equal arguments). Congruence failures are cached, and for readers writing their own kernel, caching these failures turns out to be a performance critical optimization, since the congruence check involves a potentially expensive call to `def_eq_args`.
-->

Lazy delta 処理が引数への `const` 式の適用である2つの式を見つけ、その `const` 式が同じ宣言を参照している場合、その式の合同（定義上等しい引数に適用される同じ定数であるかどうか）がチェックされます。合同チェックは `def_eq_args` の呼び出しが効果になる可能性があるため、独自のカーネルを書く読者にとって、これは失敗をキャッシュすることはパフォーマンス上重要な最適化であることがわかります。

<!--
## Syntactic equality (also structural or pointer equality)
-->

## 文法上の同値（または構造体・ポインタ上の同値）

<!--
Two expressions are definitionally equal if they refer to exactly the same implementing object, as long as the type checker ensures that two objects are equal if and only if they are constructed from the same components (where the relevant constructors are those for Name, Level, and Expr).
-->

型チェッカが2つのオブジェクトが同じコンポーネント（Name・Level・Expr のコンストラクタが該当します）から構築される場合に限り等しいことを保証する限り、2つの式が全く同じ実装されたオブジェクトを参照する場合、これらは定義上等しくなります。
