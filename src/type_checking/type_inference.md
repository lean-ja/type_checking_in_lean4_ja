<!--
# Type Inference
-->

# 型推論

<!--
Type inference is a procedure for determining the type of a given expression, and is one of the core functionalities of Lean's kernel. Type inference is how we determine that `Nat.zero` is an element of the type `Nat`, or that `(fun (x : Char) => var(0))` is an element of the type `Char -> Char`.
-->

型推論は与えられた式の型を決定するための手続きであり、Leanのカーネルのコア機能の1つです。型推論は `Nat.zero` が `Nat` 型の要素であることや、`(fun (x : Char) => var(0))` が `Char -> Char` 型の要素であることを決定する方法です。

<!--
This section begins by examining the simplest complete procedure for type inference, then the more performant but slightly more complex version of each procedure.
-->

本節ではまず型推論のためのもっとも単純で完全な手続きを確かめ、次いでそれぞれの手続きより性能の高い、しかし少し複雑なバージョンについて確認します。

<!--
We will also look at a number of additional correctness assertions that Lean's kernel makes during type inference.
-->

また、Lean のカーネルが型推論中に行ういくつかの付加的な正しさの主張についても見ていきます。

<!--
## Bound variables
-->

## 束縛変数

<!--
If you're following Lean's implementation and using the locally nameless approach, you should not run into bound variables during type inference, because all open binders will be instantiated with the appropriate free variables.
-->

Lean の実装に従って locally nameless なアプローチを使う場合、型推論中で束縛変数に遭遇することはないでしょう。というのもすべての開いた束縛子は適切な自由変数でインスタンス化されているからです。

<!--
When we come across a binder, we need to traverse into the body to determine the body's type. There are two main approaches one can take to preserve the information about the binder type; the one used by Lean proper is to create a free variable that retains the binder's type information, replace the corresponding bound variables with the free variable using instantiation, and then enter the body. This is nice, because we don't have to keep track of a separate piece of state for a typing context.
-->

束縛子に遭遇した場合、本体の型を決定するために本体を走査する必要があります。束縛子の型に関する情報を保持するには主に2つのアプローチがあります；Lean が正式に採用しているものは、束縛子の型情報を保持する自由変数を作成し、インスタンス化を対応する束縛変数を自由変数に置き換えてから本体に入るというものです。これは型付けコンテキストのために個別の状態を追跡する必要がないため良い方法です。

<!--
For closure-based implementations, you will generally have a separate typing context that keeps track of the open binders; running into a bound variable then means that you will index into your typing context to get the type of that variable.
-->

クロージャに基づく実装では、一般的に開いた束縛子を追跡するための別の型付けコンテキストを持つことになります；束縛変数に遭遇すると、その変数の型を取得するために、型付けコンテキストにインデックスを作ることになります。

<!--
## Free variables
-->

## 自由変数

<!--
When a free variable is created, it's given the type information from the binder it represents, so we can just take that type information as the result of inference.
-->

自由変数が作られる際には、それが表す束縛子から型情報が与えられるため、型推論の結果としてその型情報を受け取ればよいです。

```
infer FVar id binder:
  binder.type
```

<!--
## Function application
-->

## 関数適用

```
infer App(f, arg):
  match (whnf $ infer f) with
  | Pi binder body => 
    assert! defEq(binder.type, infer arg)
    instantiate(body, arg)
  | _ => error
```

<!--
The additional assertion needed here is that the type of `arg` matches the type of `binder`. For example, in the expression
-->

ここで追加の主張として必要なものは、`arg` の型が `binder` の型と一致することです。例えば、次の式

<!--
`(fun (n : Nat) => 2 * n) 10`, we would need to assert that `defEq(Nat, infer(10))`.
-->

`(fun (n : Nat) => 2 * n) 10` について、`defEq(Nat, infer(10))` を主張しなければなりません。

<!--
While existing implementations prefer to perform this check inline, one could potentially store this equality assertion for processing elsewhere.
-->

既存の実装ではこのチェックをインラインで実行することを好んでいますが、他の場所で処理するためにこの同値の主張を保存することもできます。

<!--
## Lambda
-->

## ラムダ式

```
infer Lambda(binder, body):
  assert! infersAsSort(binder.type)
  let binderFvar := fvar(binder)
  let bodyType := infer $ instantiate(body, binderFVar)
  Pi binder (abstract bodyType binderFVar)
```

# Pi

```
infer Pi binder body:
  let l := inferSortOf binder
  let r := inferSortOf $ instantiate body (fvar(binder))
  imax(l, r)

inferSortOf e:
  match (whnf (infer e)) with
  | sort level => level
  | _ => error
```

<!--
## Sort
-->

## ソート

<!--
The type of any `Sort n` is just `Sort (n+1)`.
-->

任意の `Sort n` の型は `Sort (n+1)` です。

```
infer Sort level:
  Sort (succ level)
```

<!--
## Const
-->

## 定数

<!--
`const` expressions are used to refer to other declarations by name, and any other declaration referred to must have been previously declared and had its type checked. Since we therefore already know what the type of the referred to declaration is, we can just look it up in the environment. We do have to substitute in the current declaration's universe levels for the indexed definition's universe parameters however.
-->

`const` 式は他の宣言を名前で参照するために使われ、参照される任意の他の宣言はこれより以前に宣言され、型がチェックされていなければなりません。したがって、参照される宣言の型が何であるかはすでに分かっているため、それを環境から探してくるだけで良いのです。ただし、現在の宣言の宇宙レベルを添字付けられた定義の宇宙パラメータに代入する必要があります。

```
infer Const name levels:
  let knownType := environment[name].type
  substituteLevels (e := knownType) (ks := knownType.uparams) (vs := levels)
```

## Let

```
infer Let binder val body:
  assert! inferSortOf binder
  assert! defEq(infer(val), binder.type)
  infer (instantiate body val)
```

<!--
## Proj
-->

## 射影

<!--
We're trying to infer the type of something like `Proj (projIdx := 0) (structure := Prod.mk A B (a : A) (b : B))`.
-->

ここで `Proj (projIdx := 0) (structure := Prod.mk A B (a : A) (b : B))` のようなものの型を推論しようとしているとしましょう。

<!--
Start by inferring the type of the structure offered; from that we can get the structure name and look up the structure and constructor type in the environment.
-->

まず渡される構造体の型を推測することから始めます；そこから構造体の名前を取得し、構造体とコンストラクタの型を環境で調べることができます。

<!--
Traverse the constructor type's telescope, substituting the parameters of `Prod.mk` into the telescope for the constructor type. If we looked up the constructor type `A -> B -> (a : A) -> (b : B) -> Prod A B`, substitute A and B, leaving the telescope `(a : A) -> (b : B) -> Prod A B`.
-->

コンストラクタの型のテレスコープを走査し、`Prod.mk` のパラメータをコンストラクタ型のテレスコープに代入します。`A -> B -> (a : A) -> (b : B) -> Prod A B` というコンストラクタ型に着目すると、A と B を代入することで、`(a : A) -> (b : B) -> Prod A B` というテレスコープが残ります。

<!--
The remaining parts of the constructor's telescope represent the structure's fields and have the type information in the binder, so we can just examine `telescope[projIdx]` and take the binder type. We do have to take care of one more thing; because later structure fields can depend on earlier structure fields, we need to instantiate the rest of the telescope (the body at each stage) with `proj thisFieldIdx s` where `s` is the original structure in the proj expression we're trying to infer.
-->

コンストラクタのテレスコープの残りの部分は構造体のフィールドを表しており、束縛子に型情報を持っているため、`telescope[projIdx]` を調べて束縛子の型を取ればよいです。やることはあと1つ残っています；後続の構造体のフィールドはその前の構造体のフィールドに依存することがあるため、残りのテレスコープ（各ステージの本体）を `proj thisFieldIdx s` でインスタンス化する必要があります。ここで `s` は推測する対象の射影の式中のオリジナルの構造体です。

```
infer Projection(projIdx, structure):
  let structType := whnf (infer structure)
  let (const structTyName levels) tyArgs := structType.unfoldApps
  let InductiveInfo := env[structTyName]
  -- This inductive should only have the one constructor since it's claiming to be a structure.
  let ConstructorInfo := env[InductiveInfo.constructorNames[0]]

  let mut constructorType := substLevels ConstructorInfo.type (newLevels := levels)

  for tyArg in tyArgs.take constructorType.numParams
    match (whnf constructorType) with
      | pi _ body => inst body tyArg
      | _ => error

  for i in [0:projIdx]
    match (whnf constructorType) with
      | pi _ body => inst body (proj i structure)
      | _ => error

  match (whnf constructorType) with
    | pi binder _=> binder.type
    | _ => error 
```

<!--
## Nat literals
-->

## 数値リテラル

<!--
Nat literals infer as the constant referring to the declaration `Nat`.
-->

数値リテラルは `Nat` 宣言を参照する定数として推測されます。

```
infer NatLiteral _:
  Const(Nat, [])
```

<!--
## String literals
-->

## 文字列リテラル

<!--
String literals infer as the constant referring to the declaration `String`.
-->

文字列リテラルは `String` 宣言を参照する定数として推測されます。

```
infer StringLiteral _:
  Const(String, [])
```

