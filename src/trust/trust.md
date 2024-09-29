
<!--
# Trust
-->

# 信頼

<!--
A big part of Lean's value proposition is the ability to construct mathematical proofs, including proofs about program correctness. A common question from users is how much trust, and in what exactly, is involved in trusting Lean.
-->

Lean が提供する価値の大部分は、プログラムの正しさに関する証明を含む数学的証明を構築する能力です。これに対するユーザの一般的な疑念は、Lean を信頼するには、具体的に何をどの程度信頼すればよいのかということです。

<!--
An answer to this question has two parts: what users need to trust in order to trust proofs in Lean, and what users need to trust in order to trust executable programs obtained by compiling a Lean program. 
-->

この質問に対する回答は2つの部分からなります：すなわち、Lean における証明を信頼するためにユーザがまず信頼すべきものは何かということ、そして Lean プログラムをコンパイルして得られる実行可能プログラムを信頼するためにユーザが信頼すべきものは何かということです。

<!--
Concretely, the distinction is that proofs (which includes statements about programs) and uncompiled programs can be expressed directly in Lean's kernel language and checked by an implementation of the kernel. They do not need to be compiled to an executable, therefore the trust is limited to whatever implementation of the kernel they're being checked with, and the Lean compiler does not become part of the trusted code base.
-->

この違いは具体的には、証明（プログラムに関する文を含む）とコンパイルされていないプログラムが Lean のカーネル言語で直接表現でき、カーネルの実装によってチェックされるというものです。これらは実行ファイルにコンパイルする必要はなく、そのため、この信頼はこれらをチェックするカーネルの実装に限定され、Lean のコンパイラは信頼できるコードベースの一部にはなりません。

<!--
Trusting the correctness of compiled Lean programs requires trust in Lean's compiler, which is separate from the kernel and is not part of Lean's core logic. There is a distinction between trusting _statements about programs_ in Lean, and trusting _programs produced by the Lean compiler_. Statements about Lean programs are proofs, and fall into the category that only requires trust in the kernel. Trusting that proofs about a program _extend to the behavior of a compiled program_ brings the compiler into the trusted code base.
-->

コンパイルされた Lean プログラムの正しさを信頼するには、Lean のコンパイラを信頼する必要がありますが、コンパイラはカーネルとは別物であり、Lean のコアロジックの一部ではありません。ここで Lean の **文とプログラム** を信頼することと、**Lean のコンパイラが生成したプログラム** を信頼することは区別されます。Lean のプログラムに関する文は証明であり、カーネルだけを信頼すればよいカテゴリに入ります。プログラムに関する証明が **コンパイルされたプログラムの動作にまで及ぶこと** を信頼することで、コンパイラは信頼されるコードベースに入ります。

<!--
**NOTE**: Tactics and other metaprograms, even tactics that are compiled, do *not* need to be trusted _at all_; they are untrusted code which is used to produce kernel terms for use by something else. A proposition `P` can be proved in Lean using an arbitrarily complex compiled metaprogram without expanding the trusted code base beyond the kernel, because the metaprogram is required to produce a proof expressed in Lean's kernel language.
-->

**注意**：タクティクや他のメタプログラムは、たとえコンパイルされたタクティクであっても信頼される必要は **全くありません**；これらは他で使用されるカーネルの項を生成するために使用される信頼されていないコードです。ある命題 `P` は信頼されたコードベースをカーネルを超えて拡張することなく、任意に複雑なコンパイル済みのメタプログラムを使って証明することができます。なぜならこのメタプログラムは Lean のカーネル言語で表現された証明を生成する必要があるからです。

<!--
+ These statements hold for proofs that are [exported](../export_format.md). To satisfy more ~~pedantic~~ vigilant readers, this does necessarily entail some degree of trust in, for example, the operating system on the computer used to run the exporter and verifier, the hardware, etc.
-->

+ これらの文は [エクスポートされた](../export_format.md) 証明に対して有効です。より ~~くどい~~ 警戒心の強い読者を満足させるために、これは例えば、エクスポータと検証器を実行するために使用されるコンピュータの OS・ハードウェアなどに対するある程度の信頼を必然的に伴います。

<!--
+ For proofs that are not exported, users are additionally trusting the elements of Lean outside the kernel (the elaborator, parser, etc.).
-->

+ エクスポートされていない証明については、ユーザはさらにカーネル外の Lean の要素（エラボレータ、パーサなど）を信頼することになります。

<!--
## An more itemized list
-->

## より詳細なリスト

<!--
A more itemized description of the trust involved in Lean 4 comes from a post by Mario Carneiro on the Lean Zulip. 
-->

Lean 4 に関連する信頼について、Mario Carneiro による Lean の Zulip への投稿からより詳細に説明しましょう。

<!--
> In general:
> 
> 1. You trust that the lean logic is sound (author's note: this would include any kernel extensions, like those for Nat and String)
> 
> 2. If you didn't prove the program correct, you trust that the elaborator has converted your input into the lean expression denoting the program you expect. 
> 
> 3. If you did prove the program correct, you trust that the proofs about the program have been checked (use external checkers to eliminate this)
> 
> 4. You trust that the hardware / firmware / OS software running all of these things didn't break or lie to you
> 
> 5. (When running the program) You trust that the hardware / firmware / OS software faithfully executes the program according to spec and there are no debuggers or magnets on the hard drive or cosmic rays messing with your output
>
> For compiled executables:
>
> 6. You trust that any compiler overrides (extern / implemented_by) do not violate the lean logic (i.e. the model matches the implementation)
>
> 7. You trust the lean compiler (which lowered the lean code to C) to preserve the semantics of the program
>
> 8. You trust clang / LLVM to convert the C program into an executable with the same semantics
-->

> 一般的に：
>
> 1. Lean のロジックが健全であること（原著者注：これには Nat や String のようなカーネルの拡張も含まれる）を信頼する。
>
> 2. プログラムの正しさを証明しなかった場合、エラボレータがあなたの入力から期待するプログラムを示す Lean の式に変換したことを信頼する。
>
> 3. プログラムが正しいことを証明したのであれば、そのプログラムに関する証明がチェックされていることを信頼する（これを排除するために外部のチェッカを使う）。
>
> 4. これらすべてを動かしているハードウェア・ファームウェア・OS ソフトウェアが壊れたり、嘘をついたりしていないことを信頼する。
>
> 5. （プログラムを実行する時）ハードウェア・ファームウェア・OS ソフトウェアが仕様通りに忠実にプログラムを実行し、デバッガやハードディスク上の磁石や宇宙線がプログラムの出力を乱すことがないことを信頼する。
>
> 複雑な実行ファイルについて：
>
> 6. コンパイラのオーバーライド（extern・implemented_by）がLean のロジックに違反しないことを信頼する（すなわち、モデルが実装と一致する）。
>
> 7. Lean のコンパイラ（Lean コードを C に変換）がプログラムの意味を維持することを信頼する。
>
> 8. C プログラムを同じ意味を持つ実行ファイルに変換する clang・LLVM を信頼する。

<!--
The first set of points applies to both proofs and compiled executables, while the second set applies specifically to compiled executable programs.
-->

前半のポイントは証明とコンパイルされた実行ファイルの両方に適用され、後半のポイントは特にコンパイルされた実行ファイルに適用されます。

<!--
## Trust for external checkers
-->

## 外部チェッカへの信頼

<!--
1. You're still trusting Lean's logic is sound.
-->

1. Lean のロジックが健全であることをとにかく信頼する。

<!--
2. You're trusting that the developers of the external checker properly implemented the program.
-->

2. 外部チェッカの開発者が適切にプログラムを実装していることを信頼する。

<!--
3. You're trusting the implementing language's compiler or interpreter. If you run multiple external checkers, you can think of them as circles in a venn diagram; you're trusting that the part where the circles intersect is free of soundness issues.
-->

3. 実装言語のコンパイラやインタプリタを信頼する。複数の外部チェッカを実行する場合、それらをベン図の円のように考えることができます；円が交差する部分に健全性の問題がないことを信頼します。

<!--
4. For the Nat and String kernel extensions, you're probably trusting a bignum library and the UTF-8 string type of the implementing language.
-->

4. Nat と String のカーネル拡張について、おそらく実装言語の bignum ライブラリと UTF-8 文字列型を信頼する。

<!--
The advantages of using external checkers are:
-->

外部チェッカを使用する利点は以下の通りです：

<!--
+ Users can check their results with something that is completely disjoint from the Lean ecosystem, and is not dependent on any parts of Lean's code base.
-->

+ ユーザは Lean のエコシステムから切り離され、Lean のコードベースのどの部分にも依存しないものを使って結果をチェックすることができます。

<!--
+ External checkers can be written to take advantage of mature compilers or interpreters.
-->

+ 外部チェッカは成熟したコンパイラやインタプリタを利用するように書くことができます。

<!--
+ For kernel extensions, users can cross-check the results of multiple bignum/string implementations.
-->

+ カーネル拡張に対しては、ユーザは複数の bignum・string 実装の結果をクロスチェックできます。

<!--
+ Using the export feature is the only way to get out of trusting the parts of Lean outside the kernel, so there's a benefit to doing this even if the export file is checked by something like [lean4lean](https://github.com/digama0/lean4lean/tree/master). Users worried about fallout from misuse of Lean's metaprogramming features are therefore encouraged to use the export feature.
-->

+ エクスポート機能を使うことは、Lean のカーネル外の部分を信頼することから抜け出す唯一の方法です。したがって、エクスポートされたファイルが [lean4lean](https://github.com/digama0/lean4lean/tree/master) のようなものによってチェックされたとしてもこれを行うことには利点があります。それゆえ Lean のメタプログラミング機能の誤用による影響を心配するユーザはエクスポート機能を使うことが推奨されます。
