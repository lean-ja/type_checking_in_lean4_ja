<!--
# Future work and open issues
-->

# 課題と今後の展望

<!--
## File format
-->

## ファイルフォーマット

<!--
There is an open rfc [here](https://github.com/leanprover/lean4export/issues/3) about moving the export file format to json, which would be a major version change.
-->

エクスポートファイルのフォーマットをJSONに移行する件について、[こちら](https://github.com/leanprover/lean4export/issues/3) にオープンなRFCがあります。これは大きなバージョン変更になるでしょう。

<!--
## Ensuring Nat/String are defined properly.
-->

## 整数・文字列が適切に定義されていることの確認

<!--
The Lean community has not yet settled on a particular solution for determining whether the declarations in an export file for `Nat`, `String`, and the operations covered by the relevant kernel extensions match those expected by extensions in a way that does not pull the exporter into the trusted code. 
-->

`Nat`・`String`・およびそれらに関連するカーネル拡張がカバーする操作のためのエクスポートファイルの宣言が、信頼されたコードにエクスポータを引き込まない方法で拡張が期待するものと一致するかどうかを判断するための特定の解決策について Lean コミュニティはまだ定まっていません。

<!--
An approach similar to that taken for `Eq` and the `Quot` declarations (defining them by hand within the type checker, then asserting they're the same) is not feasible due to the complexity of the fully elaborated terms for the supported binary operations on `Nat`.
-->

`Eq` と `Quot` の宣言のようなアプローチ（型チェッカ内で手作業で定義し、それらが同じであることを保証する）は `Nat` でサポートしているバイナリ演算のための完全にエラボレートされた項が複雑であるため実行不可能です。

<!--
## Improving Pollack consistency
-->

## Pollack consistency の改善

<!--
Lean 4 offers very powerful facilities for defining custom syntax, macros, and pretty printer behaviors, and almost every aspect of Lean 4's internals is available to users. These elements of Lean's design were effective responses to real world feedback from the mathlib community during Lean 3's lifetime.
-->

Lean 4 はカスタム構文・マクロ・プリティプリンタの定義に非常に強力な機能を提供し、Lean 4 の内部のほとんどすべての側面がユーザによって利用可能です。Lean の設計のこれらの要素は Lean 3 の存続期間中に mathlib コミュニティから実際に寄せられたフィードバックに対する効果的な対応でした。

<!--
While these features were important factors in Lean's success as a tool for enabling large formalization efforts, they are also in tension with Lean4's Pollack consistency[^pollack], or lack thereof. Without replicating the macro and syntax extension capabilities in the pretty printer, type checkers cannot consistently read terms back to the user in a form that is recognizable. However, the idea of adding these features to a pretty printer is an unappealing expansion of the trusted code base. An alternative approach is to drop the pretty printer in favor of a trusted parser (ala metamath zero), but Lean's parser can be modified on the fly in userspace with custom syntax declarations.
-->

これらの機能は大規模な形式化作業を可能にするツールとして Lean が成功した重要な要因である一方、Lean 4 の Pollack consistency[^pollack] が危うく、もしくは損なわれる恐れがあります。プリティプリンタでマクロと構文拡張機能を複製しなければ、型チェッカは一貫して項を認識可能な形で読んでユーザに返すことができません。しかし、これらの機能をプリティプリンタに追加するアイデアは信頼されたコードベースを拡張する点においては魅力的なものではありません。別のアプローチとしてプリティプリンタをやめて、信頼できるパーサ（通称 metamath zero）を採用する方法もありますが、Lean のパーサはカスタム構文宣言を使ってユーザ空間でその場で変更することができます。

<!--
As Lean matures and adoption increases, there is likely to be a push for progress in the development of techniques and practices that allow users to take advantage of Lean's extensibility while sacrificing the least degree of Pollack consistency.
-->

Lean が成熟し、採用が進むにつれて Pollack consistency を犠牲にすることなく、ユーザが Lean の拡張性を活用できるような技術や手法の開発が進められるでしょう。

<!--
## Forward reasoning
-->

## 前方推論

<!--
Existing type checkers implement a form of backward reasoning; an alternate strategy for type checking is to accept and check forward reasoning chains worked out by an external program, potentially allowing for an even simpler type checker.
-->

既存の型チェッカは後方推論の形式を実装しています；型チェッカの戦略の別路線として、外部プログラムによって作成された前方推論の連鎖を受け入れ、チェックすることがあり、これは後方推論よりもシンプルな型チェッカとなる可能性があります。

[^pollack]: Freek Wiedijk. Pollack-inconsistency. Electronic Notes in Theoretical Computer Science, 285:85–100, 2012