<!-- # Unsafe declarations -->

# 安全でない宣言

<!-- Lean's vernacular allows users to write declarations marked as `unsafe`, which are permitted to do things that are normally forbidden. For example, Lean permits the following definition: -->

Lean の特異なマークとして、ユーザは `unsafe` とマークされた宣言を書くことができ、これによって通常であれば禁止されるようなものが許可されるようになります。例えば、Lean は次のような定義を許可しています：

```
  unsafe def y : Nat := y
```

<!-- Unsafe declarations are not exported[^note1], do not need to be trusted, and (for the record) are not permitted in proofs, even in the vernacular. Permitting unsafe declarations in the vernacular is still beneficial for Lean users, because it gives users more freedom when writing code that is used to produce proofs but doesn't have to be a proof in and of itself. -->

安全でない宣言はエクスポートされず [^note1]、信頼される必要もなく、（ちなみに）たとえこのマークがついたものの中であったとしても証明にて使うことは許可されません。それでもこのマークによる安全でない宣言の許可は Lean ユーザにとって有益です。なぜなら、証明を生成するために使用されるもののそれ自体に対しておよびそれ自体が証明である必要がないコードを書く際に、より多くの自由をユーザに与えるからです。

<!-- The aesop library provides us with an excellent real world example. [Aesop](https://github.com/leanprover-community/aesop) is an automation framework; it helps users generate proofs. At some point in development, the authors of aesop felt that the best way to express a certain part of their system was with a mutually defined inductive type, [seen here](https://github.com/leanprover-community/aesop/blob/69404390bdc1de946bf0a2e51b1a69f308e56d7a/Aesop/Tree/Data.lean#L375). It just so happens that this set of inductive type has an invalid occurrence of one of the types being declared within Lean's theory, and would not be permitted by Lean's kernel, so it needs to be marked `unsafe`. -->

このことについて aesop ライブラリは優れた実例を有しています。[Aesop](https://github.com/leanprover-community/aesop) は自動化のフレームワークです；これはユーザが証明を生成することを補助します。このライブラリのある時点で、aesop の作者たちはシステムのある部分を表現するにあたって、[ここで見られるように](https://github.com/leanprover-community/aesop/blob/69404390bdc1de946bf0a2e51b1a69f308e56d7a/Aesop/Tree/Data.lean#L375) 相互に定義された帰納型を使うことが最適だと考えました。偶然にも、この帰納型のセットは Lean の理論で宣言されている型の中に1つ無効なものがあり、Lean のカーネルでは許可されないため `unsafe` とマークする必要があります。

<!-- Permitting this definition as an `unsafe` declaration is still a win-win. The Aesop developers were able to use Lean to write their library the way they wanted, in Lean, without having to call out to (and learn) a separate metaprogramming DSL, they didn't have to jump through hoops to satisfy the kernel, and users of aesop can still export and verify the proofs produced *by* aesop without having to verify aesop itself. -->

それでもこの定義を `unsafe` の宣言として許可することは開発者とユーザのどちらにとっても有益です。aesop の開発者は、Lean を使ってこのライブラリを書くにあたり望むままにライブラリを書くことができますが、それにあたってメタプログラミングの DSL を呼び出すことなく（そして学ぶこともなく）、またカーネルを満足させるために抜け道を使う必要もありません。aesop のユーザも aesop **による** 証明を aesop 自体を検証する必要なくエクスポートし検証することができます。

<!-- [^note1]: There's technically nothing preventing an unsafe declaration from being put in an export file (especially since the exporter is not a trusted component), but checks run by the kernel will prevent unsafe declarations from being added to the environment if they are actually unsafe. A properly implemented type checker would throw an error if it received an export file declaring the aesop library code described above. -->

[^note1]: 技術的には、安全でない宣言をエクスポートファイルに入れることを妨げることはできません（特にエクスポートファイルは信頼できるコンポーネントではないため）が、カーネルが実行するチェックによって、安全でない宣言が実際に安全でない場合には環境に追加されることを防ぐことができます。適切に実装された型チェッカは上記の aesop ライブラリを宣言したエクスポートファイルを受け取った場合にエラーを投げるでしょう。