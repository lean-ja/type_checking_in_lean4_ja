<!-- # Adversarial inputs -->

# 敵対的な入力

<!-- A topic that often accompanies the more general trust question is Lean's robustness against adversarial inputs. -->

より一般的な信頼の問題に対してしばしば付随するトピックとして、敵対的な入力に対する Lean の堅牢性が挙げられます。

<!-- A correct type checker will restrict the input it receives to the rules of Lean's type system under whatever axioms the operator allows. If the operator restricts the permitted axioms to the three "official" ones (`propext`, `Quot.sound`, `Classical.choice`), an input file should not be able to offer a proof of the prelude's `False` which is accepted by the type checker under any circumstances. -->

正しい型チェッカは利用者が許可する公理のもとにおいて、受け取る入力を Lean の型システムのルールで制限します。利用者が許容する公理を3つの「公式の」公理（`propext`・`Quot.sound`・`Classical.choice`）に制限した場合、入力ファイルはどのような状況においても型チェッカが受け入れる prelude の `False` である証明を提供することはできないはずです。

<!-- However, a minimal type checker will not actively protect against inputs which provide Lean declarations that are logically sound, but are designed to fool a human operator. For example, redefining deep dependencies an adversary knows will not be examined by a referee, or introducing unicode lookalikes to produce a pretty printer output that conceals modification of key definitions. -->

しかし、最小の型チェッカは論理的には正しいですが、利用者を欺くように設計された Lean の宣言を提供する入力から積極的に保護することはできません。例えば、敵が深い依存関係を理解した上で再定義してレフェリーが検査しないようにしたり、ユニコードに似たものでプリティプリンタの出力を生成するようなものを導入しつつその裏で重要な定義変更をしたりできてしまいます。

<!-- The idea that "a user might think a theorem has been formally proved, while in fact he or she
is misled about what it is that the system has actually done" is addressed by the idea of Pollack consistency and is explored in this publication[^pollack] by Freek Wiedijk.  -->

「ある定理が形式的に証明されたとユーザが考える一方で、実際にはシステムが実際に行ったことが何であるかについてユーザが誤解している」というアイデアは Pollack consistency という概念で扱われており、Freek Wiedijk によるこの書籍[^pollack] で議論されています。

<!-- Note that there is nothing in principle preventing developers from writing software or extending a type checker to provide protection against such attacks, it's just not captured by the minimal functionality required by the kernel. However, the extent to which Lean's users have embraced its powerful custom syntax and macro systems may pose some challenges for those interested in improving the story here. Readers should consider this somewhat of an [open issue for future work](./future_work.md#improving-pollack-consistency) -->

このようなカーネルの最小限の機能でカバーできない攻撃に対する保護を提供するために、開発者がソフトウェアを書いたり、型チェッカを拡張したりすることは原理的に妨げるものは何もありません。しかし、Lean のユーザがその強力なカスタム構文とマクロシステムをどの程度まで受け入れているか、ここまでの話を改善することに興味を持つ人々にとっていくつかの課題が生じる可能性があります。読者におかれましては、このことを多少なりとも [今後の課題](./future_work.md#improving-pollack-consistency) として考えるべきでしょう。

[^pollack]: Freek Wiedijk. Pollack-inconsistency. Electronic Notes in Theoretical Computer Science, 285:85–100, 2012
