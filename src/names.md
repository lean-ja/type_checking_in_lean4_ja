<!--
# Names
-->

# 名前

<!--
The first of the kernel's primitive types is `Name`, which is sort of what it sounds like; it provides kernel items with a way of addressing things.
-->

カーネルのプリミティブ型の中でも最初のものは `Name` で、これはその名称の通りのもののあつまりを指します；これはアドレスのための方法を備えたカーネルの要素を提供します。

```
Name ::= anonymous | str Name String | num Name Nat
```

<!--
Elements of the `Name` type are displayed as dot-separated names, which users of Lean are probably familiar with. For example, `num (str (anonymous) "foo") 7` is displayed as `foo.7`. 
-->

`Name` 型の要素はドット区切りの名前で表示されます。これはおそらく Lean ユーザにはなじみ深いでしょう。例えば、`num (str (anonymous) "foo") 7` は `foo.7` と表示されます。

<!--
# Implementation notes
-->

# 実装についての注記

<!--
The implementation of names assumes UTF-8 strings, with characters as unicode scalars (these assumptions about the implementing language's string type are also important for the string literal kernel extension). 
-->

名前の実装では、文字が Unicode scalar である UTF-8 文字列を想定しています（言語の文字列型のこの実装に関するこれらの想定は、文字列リテラルのカーネル拡張にとっても重要です）。

<!--
Some information on the lexical structure of names can be found [here](https://github.com/leanprover/lean4/blob/504b6dc93f46785ccddb8c5ff4a8df5be513d887/doc/lexical_structure.md?plain=1#L40)
-->

名前の字句構造に関するいくつかの情報については [こちら](https://github.com/leanprover/lean4/blob/504b6dc93f46785ccddb8c5ff4a8df5be513d887/doc/lexical_structure.md?plain=1#L40) を参照してください。

<!--
The exporter does not explicitly output the anonymous name, and expects it to be the 0th element of the imported names.
-->


エクスポータは匿名の名前を明示的に出力せず、インポートされた名前の0番目の要素であることを期待します。
