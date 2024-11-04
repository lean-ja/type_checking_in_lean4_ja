<!--
# What is the kernel?
-->

# カーネルとは何か？

<!--
The kernel is an implementation of Lean's logic in software; a computer program with the minimum amount of machinery required to construct elements of Lean's logical language and check those elements for correctness. The major components are:
-->

カーネル（kernel）とは Lean の論理をソフトウェアで実装したもののことです；Lean の論理言語の要素を構築し、それらの要素の正しさをチェックするための必要最小限の機構を備えたコンピュータプログラムです。主な構成要素は以下の通りです：

<!--
+ A sort of names used for addressing.
-->

+ アドレッシングのための名前。

<!--
+ A sort of universe levels.
-->

+ 宇宙レベル。

<!--
+ A sort of expressions (lambdas, variables, etc.)
-->

+ 式（ラムダ式、変数など）

<!--
+ A sort of declarations (axioms, definitions, theorems, inductive types, etc.)
-->

+ 宣言（公理・定義・定理・帰納型など）

<!--
+ Environments, which are maps of names to declarations.
-->

+ 環境、これは名前から宣言へのマップです。

<!--
+ Functionality for manipulating expressions. For example bound variable substitution and substitution of universe parameters.
-->

+ 式の操作のための機能。例えば、束縛変数の置換や宇宙パラメータの置換など。

<!--
+ Core operations used in type checking, including type inference, reduction, and definitional equality checking.
-->

+ 型推論・簡約・定義上の等しさ（definitional equality）など、型検査で用いられる中核的な操作。

<!--
+ Functionality for manipulating and checking inductive type declarations. For example, generating a type's recursors (elimination rules), and checking whether a type's constructors agree with the type's specification.
-->

+ 帰納型の宣言を操作し、チェックする機能。例えば、型の再帰子（recursor、除去則）を生成したり、型のコンストラクタが型の仕様に合致しているかどうかのチェックをしたりします。

<!--
+ Optional kernel extensions which permit the operations above to be performed on nat and string literals.
-->

+ オプションのカーネル拡張、自然数や文字列リテラルへ上記の操作を実行できます。

<!--
The purpose of isolating a small kernel and requiring Lean definitions to be translated to a minimal kernel language is to increase the trustworthiness of the proof system. Lean's design allows users to interact with a full-featured proof assistant which offers nice things like robust metaprogramming, rich editor support, and extensible syntax, while also permitting extraction of constructed proof terms into a form that can be verified without having to trust the correctness of the code that implements the higher level features that makes Lean (the proof assistant) productive and pleasant to use.
-->

カーネルを小さいパーツに分離し、Lean の定義を最小限のカーネル言語に翻訳することを求める目的は、証明システムの信頼性を高めることです。Lean の設計によって堅牢なメタプログラミング・豊富なエディタサポート・拡張可能な構文といった素晴らしい機能を網羅した証明支援系とユーザの対話が可能になる、一方で（証明支援系である）Lean を生産的で快適に使用できるようにする高水準の機能を実装するコードの正しさを信頼することなく、構築された証明項を検証可能な形に抽出することを可能にします。

<!--
In section 1.2.3 of the [_Certified Programming with Dependent Types_](http://adam.chlipala.net/cpdt/), Adam Chlipala defines what is sometimes referred to as the de Bruijn criterion, or de Bruijn principle.
-->

[_Certified Programming with Dependent Types_](http://adam.chlipala.net/cpdt/) の 1.2.3 節にて、Adam Chlipala は de Bruijn 基準、もしくは de Bruijn 原理と呼ばれるものを定義しています。

<!--
> Proof assistants satisfy the “de Bruijn criterion” when they produce proof terms in small kernel languages, even when they use complicated and extensible procedures to seek out proofs in the first place. These core languages have feature complexity on par with what you find in proposals for formal foundations for mathematics (e.g., ZF set theory). To believe a proof, we can ignore the possibility of bugs during search and just rely on a (relatively small) proof-checking kernel that we apply to the result of the search.
-->

> 証明支援系は、たとえ証明を探索するために複雑で拡張可能な手続きをはじめから使っている場合であっても、小さなカーネル言語で証明項を生成すれば「de Bruijn 基準」を満たす。これらのコア言語は、数学の形式的基礎（例えば ZF 集合論）の提案に見られるような複雑さを持っている。証明を信じるには、探索中のバグの可能性を無視し、探索結果に適用する（比較的小さな）証明チェック用のカーネルに頼ればよい。

<!--
Lean's kernel is small enough that developers can write their own implementation and independently check proofs in Lean by using an exporter[^1]. Lean's export format contains contains enough information about the exported declarations that users can optionally restrict their implementation to certain subsets of the full kernel. For example, users interested in the core functionality of inference, reduction, and definitional equality may opt out of implementing the functionality for checking inductive specifications.
-->

Lean のカーネルは十分に小さいため、開発者は独自の実装を書くことができ、エクスポータ [^1] を使うことで独自に Lean の証明をチェックすることができます。Lean のエクスポートフォーマットには、エクスポートされた宣言に関する十分な情報が含まれているため、ユーザはオプションで実装をカーネル全体の特定のサブセットに限定することができます。例えば、推論・簡約・定義上の等しさのコア機能に興味のあるユーザは帰納的な仕様のチェック機能の実装を省略することができます。

<!--
In addition to the list of items above, external type checkers will also need a parser for [Lean's export format](./export_format.md), and a pretty printer, for input and output respectively. The parser and pretty printer are not part of the kernel, but they are important if one wants to have interesting interactions with the kernel.
-->

上記のリストに加え、外部の型チェッカは [Lean のエクスポートフォーマット](./export_format.md) 用のパーサと、入出力用でそれぞれにプリティプリンタも必要になります。パーサとプリティプリンタはカーネルの一部ではありませんが、カーネルと興味深いやり取りをしたいのであれば重要です。

<!--
[^1]: Writing your own type checker is not an afternoon project, but it is well within the realm of what is achievable for citizen scientists.
-->

[^1]: 自分で型チェッカを書くことは容易なプロジェクトではありませんが、市民科学者にとっては十分達成可能な範囲です。
