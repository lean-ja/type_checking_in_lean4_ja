<!--
# Instantiation and abstraction
-->

# インスタンス化と抽象化

<!--
Instantiation refers to substitution of bound variables for the appropriate arguments. Abstraction refers to replacement of free variables with the appropriate bound variable when replacing binders. Lean's kernel uses deBruijn indices for bound variables and unique identifiers for free variables.
-->

インスタンス化は束縛変数を適切な引数に置き換えることです。抽象化は束縛子を置換する際に、自由変数を適切な束縛変数に置き換えることです。Lean のカーネルは束縛変数には de Bruijn インデックスを使用し、自由変数には一意な識別子を使用します。

<!--
For our purposes, a free variable is a variable in an expression that refers to a binder which has been "opened", and is no longer immediately available to us, so we replace the corresponding bound variable with a free variable that has some information about the binder we're leaving behind.
-->

この目的において、自由変数はある「開いた」束縛子に指定されている式の中の変数のことで、これは即座に利用することができません。そのため対応する束縛変数を、残しておく束縛子に関する情報を持つ自由変数に置き換えます。

<!--
To illustrate, let's say we have some lambda expression `(fun (x : A) => <body>)` and we're doing type inference. Type inference has to traverse into the `<body>` part of the expression, which may contain a bound variable that refers to `x`. When we traverse into the body, we can either add `x` to some stateful context of binders and take the whole stateful context into the body with us, or we can temporarily replace all of the bound variables that refer to `x` with a free variable, allowing us to traverse into the body without having to carry any additional context.
-->

このことを示すために、あるラムダ式 `(fun (x : A) => <body>)` があり、これの型推論をしようとしているとしましょう。型推論は式の `<body>` 部分を走査する必要があり、その中には `x` を参照する束縛変数があるかもしれません。本体を走査する際、束縛子のステートフルなコンテキストに `x` を追加してステートフルなコンテキスト全体を本体に持ち込むか、一時的に `x` を参照するすべての束縛変数を自由変数に置き換えて追加のコンテキストを持たずに本体を走査するかのどちらかを行うことができます。

<!--
If we eventually come back to where we were before we opened the binder, abstraction allows us to replace all of the free variables that were once bound variables referring to `x` with new bound variables that again refer to `x`, with the correct deBruijn indices.
-->

最終的に束縛子を開く前にいた場所に戻れば、抽象化によってかつて `x` を参照する束縛変数だったすべての自由変数を、正しい de Bruijn インデックスによって再び `x` を参照する新しい束縛変数に置換することができます。

<!--
## Implementing free variable abstraction
-->

## 自由変数の抽象化の実装

<!--
For deBruijn levels, the free variables keep track of a number that says "I am a free variable representing the nth bound variable *from the top of the telescope*". 
-->

de Bruijn レベルにおいて、自由変数は「私は **telescope の頂点から** n 番目の束縛変数を表す自由変数である」ということを意味する数字を追跡します。

<!--
This is the opposite of a deBruijn index, which is a number indicating "the nth bound variable from the bottom of the telescope".
-->

これは「telescope の底から n 番目の束縛変数」を示す数値である de Bruijn インデックスの逆です。

<!--
Top and bottom here refer to visualizing the expression's telescope as a tree:
-->

ここでいう上と下は式の telescope を木に見立てています：

```
      fun
      /  \
    a    fun
        /   \
      b      ...
              \
              fun
             /   \
            e    bvar(0)
```

<!--
For example, with a lambda `fun (a b c d e) => bvar(0)`, the bound variable refers to `e`, by referencing "the 0th from the bottom".
-->

例えば、ラムダ式 `fun (a b c d e) => bvar(0)` の場合、束縛変数は「下から0番目」を参照して `e` を参照します。

<!--
In the lambda expression `fun (a b c d e) => fvar(4)`, the free variable is a deBruijn level representing `e` again, but this time as "the 4th from the top of the telescope".
-->

ラムダ式 `fun (a b c d e) => fvar(4)` では、自由変数はここでも再び `e` を表す de Bruijn レベルですが、今回は「telescope の上から4番目」です。

<!--
Why the distinction? When we create a free variable during strong reduction, we know a couple of things: we know that the free variable we're about to sub in might get moved around by further reduction, we know how many open binder are *ABOVE* us (because we had to visit them to get here), and we know we might need to quote/abstract this expression to replace the binders, meaning we need to re-bind the free variable. However, in that moment, we do NOT know how many binders remain below us, so we cannot say how many variables from the bottom that variable might be when it's eventually abstracted/quoted.
-->

なぜ区別するのでしょうか？強い簡約を行う際に自由変数を作成するとき、いくつかの事実が利用可能です：ここで割り当てようとしている自由変数がさらに簡約することで移動してしまうかもしれないこと、開いた束縛子がそこの **上** にいくつあるか（ここに到達するために束縛子を訪れないといけないため）、束縛子を置き換えるためにこの式を quote・抽象化する必要があるかもしれないこと、つまり自由変数を再束縛する必要があることなどです。しかし、その時点では束縛子が下にいくつ残っているかわからないため、その変数が最終的に抽象化・quote されたときに下から何番目の変数になるかはわかりません。

<!--
For implementations using unique identifiers to tag free variables, this problem is solved by having the actual telescope that's being reconstructed during abstraction. As long as you have the expression and a list of the uniquely-tagged free variables, you can abstract, because the position of the free variables within the list indicates their binder position.
-->

一意な識別子を使用して自由変数にタグをつける実装の場合、抽象化の際に再構築される実際の telescope があれば、この問題は解決します。式と一意にタグ付けされた自由変数のリストさえあれば、リスト内の自由変数の位置が束縛子の位置を示すため、抽象化することができます。
