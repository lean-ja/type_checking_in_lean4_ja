
<!--
# The Secret Life of Inductive Types
-->

# 帰納型の知られざる一生

<!--
### Inductive
-->

### 帰納的対象

<!--
For clarity, the whole shebang of "an inductive declaration" is a type, a list of constructors, and a list of recursors. The declaration's type and constructors are specified by the user, and the recursor is derived from those elements. Each recursor also gets a list of "recursor rules", also known as computation rules, which are value level expressions used in iota reduction (a fancy word for pattern matching). Going forward, we will do our best do distinguish between "an inductive type" and "an inductive declaration".
-->

より精確には、「帰納的宣言」の全体像は型・コンストラクタのリスト・再帰子のリストからなります。宣言の型とコンストラクタはユーザが指定し、再帰子はそれらの要素から派生します。各再帰子は計算規則とも呼ばれる「再帰子規則」のリストも取得します。これはι簡約（パターンマッチの大げさな言い方）で使用される値レベルの式です。これ以降においては、「帰納型」と「帰納的宣言」はなるべく区別するようにします。

<!--
Lean's kernel natively supports mutual inductive declarations, in which case there is a list of (type, list constructor) pairs. The kernel supports nested inductive declarations by temporarily transforming them to mutual inductives (more on this below).
-->

Lean のカーネルでは相互帰納的宣言をネイティブにサポートしています。この場合、（型とコンストラクタのリストの）ペアのリストが存在します。カーネルは入れ子になった帰納的宣言について、一時的に相互帰納的宣言に変換することでサポートします（これについては後述します）。

<!--
### Inductive types
-->

### 帰納型

<!--
The kernel requires the "inductive type" part of an inductive declaration to actually be a type, and not a value (`infer ty` must produce some `sort <n>`). For mutual inductives, the types being declared must all be in the same universe and have the same parameters.
-->

カーネルは帰納的宣言のうち「帰納型」について、値ではなく実際に型であることを要求します（`infer ty` は何らかの `sort <n>` を生成しなければなりません）。相互帰納型では、宣言される型がすべて同じ宇宙にあり、同じパラメータを持たなければなりません。

<!--
### Constructor
-->

### コンストラクタ

<!--
For any constructor of an inductive type, the following checks are enforced by the kernel:
-->

帰納型のどのコンストラクタにおいても、カーネルによって以下のチェックが課されます：

<!--
+ The constructor's type/telescope has to share the same parameters as the type of the inductive being declared.
-->

+ コンストラクタの型・テレスコープは宣言されている帰納型と同じパラメータを共有する必要があります。

<!--
+ For the non-parameter elements of the constructor type's telescope, the binder type must actually be a type (must infer as `Sort _`).
-->

+ コンストラクタ型のテレスコープのパラメータではない要素では、束縛子の型は実際に型でなければなりません（`Sort _` として推論されなければなりません）。

<!--
+ For any non-parameter element of the constructor type's telescope, the element's inferred sort must be less than or equal to the inductive type's sort, or the inductive type being declared has to be a prop.
-->

+ コンストラクタ型のテレスコープのパラメータではない要素では、その要素から推論されたソートが帰納型のソート以下であるか、もしくは帰納型が命題として宣言されていなければなりません。

<!--
+ No argument to the constructor may contain a non-positive occurrence of the type being declared (readers can explore this issue in depth [here](https://counterexamples.org/strict-positivity.html?highlight=posi#positivity-strict-and-otherwise)).
-->

+ コンストラクタの引数は宣言されている型の positive ではない出現を含んではなりません（この問題についてはより知りたい読者は [こちら](https://counterexamples.org/strict-positivity.html?highlight=posi#positivity-strict-and-otherwise) を参照してください）。

<!--
+ The end of the constructor's telescope must be a valid application of arguments to the type being declared. For example, we require the `List.cons ..` constructor to end with `.. -> List A`, and it would be an error for `List.cons` to end with `.. -> Nat`
-->

+ コンストラクタのテレスコープの終わりは、宣言されている型への引数の有効な適用でなければなりません。例えば、`List.cons ..` コンストラクタは `.. -> List A` で終わる必要があり、`.. -> Nat` で終わる `List.cons ..` はエラーになります。

<!--
#### Nested inductives 
-->

#### 入れ子になった帰納的対象

<!--
Checking nested inductives is a more laborious procedure that involves temporarily specializing the nested parts of the inductive types in a mutual block so that we just have a "normal" (non-nested) set of mutual inductives, checking the specialized types, then unspecializing everything and admitting those types.
-->

入れ子になった帰納的対象をチェックするのはとても手間のかかる手順です。まず相互ブロックの中の帰納型の入れ子部分を一時的に特殊化することで相互帰納型の「通常の」（入れ子になっていない）あつまりを持つようにし、特殊化された型をチェックし、それからすべての特殊化を解除してそれらの型を認可します。

<!--
Consider this definition of S-expressions, with the nested construction `Array Sexpr`:
-->

次の入れ子構造 `Array Sexpr` を伴った S 式の定義を考えてみましょう：

```
inductive Sexpr
| atom (c : Char) : Sexpr
| ofArray : Array Sexpr -> Sexpr
```

<!--
Zooming out, the process of checking a nested inductive declaration has three steps:
-->

俯瞰的に見ると、入れ子の帰納的宣言をチェックするプロセスには3つのステップがあります：

<!--
1. Convert the nested inductive declaration to a mutual inductive declaration by specializing the "container types" in which the current type is being nested. If the container type is itself defined in terms of other types, we'll need to reach those components for specialization as well. In the example above, we use `Array` as a container type, and `Array` is defined in terms of `List`, so we need to treat both `Array` and `List` as container types.
-->

1. 現在の型がネストされている「コンテナ型」を特殊化することによって、入れ子になった帰納的宣言を相互帰納的宣言に変換する。コンテナ型自体が他の型で定義されている場合は、それらの要素も特殊化する必要があります。上の例では、コンテナ型として `Array` を使用していますが、`Array` は `List` の用語によって定義されているため、`Array` と `List` の両方をコンテナ型として扱う必要があります。

<!--
2. Do the normal checks and construction steps for a mutual inductive type.
-->

2. 相互帰納型の通常のチェックと構築の手順を行う。

<!--
3. Convert the specialized nested types back to the original form (un-specializing), adding the recovered/unspecialized declarations to the environment.
-->

3. 特殊化された入れ子の型をもとの形に戻し（非特殊化）、復帰・非特殊化された宣言を環境に追加する。

<!--
An example of this specialization would be the conversion of the `Sexpr` nested inductive above as:
-->

この特殊化の例として、上記の入れ子になった帰納的対象 `Sexpr` は次のように変換されます：

```
mutual
  inductive Sexpr
    | atom : Char -> Sexpr
    | ofList : ListSexpr -> Sexpr

  inductive ListSexpr 
    | nil : ListSexpr
    | cons : Sexpr -> ListSexpr -> ListSexpr 

  inductive ArraySexpr
    | mk : ListSexpr -> ArraySexpr
end
```

<!--
Then recovering the original inductive declaration in the process of checking these types. To clarify, when we say "specialize", the new `ListSexpr` and `ArraySexpr` types above are specialized in the sense that they're defined only as lists and arrays of `Sexpr`, as opposed to being generic over some arbitrary type as with the regular `List` type.
-->

そして、これらの型をチェックする過程で元の帰納的宣言を回復します。より明確にすると、「特殊化」といった場合、上記の新しい `ListSexpr` 型と `ArraySexpr` 型は通常の `List` 型のように任意の型に対して汎用的であることは対照的に、`Sexpr` のリストと配列としてのみ定義されているという意味で特殊化されています。

<!--
### Recursors
-->

### 再帰子

TBD