# Type Checking in Lean 4

<!--
This document exists to help readers better understand Lean's kernel, clarify the trust assumptions involved in using Lean, and serve as a resource for those who wish to write their own external type checkers for Lean's kernel language.
-->

このドキュメントは、読者が Lean のカーネルをよりよく理解し、Lean を使用する際の信頼の前提を明確にし、Lean のカーネル言語用に独自の外部型チェッカを書きたい人のためのリソースとなることを目的としています。

## この翻訳について

この翻訳は有志による **非公式** 翻訳です。翻訳に際して分かりやすさのために表現を大きく変えた箇所があります。また、用語の訳が一般的でない・誤りを含む可能性があります。必要に応じて原文 [Type Checking in Lean 4](https://ammkrn.github.io/type_checking_in_lean4/)([GitHub](https://github.com/ammkrn/type_checking_in_lean4))をご覧ください。

原文のライセンスは[Apache License 2.0](https://github.com/lean-ja/type_checking_in_lean4_ja/blob/main/LICENSE)であり、それに基づいて原文を翻訳・公開しています。

誤字脱字・内容の誤りの指摘・フォークからのPull Request・フォークによる翻訳の改変等歓迎いたします。ご指摘は [当該リポジトリ](https://github.com/lean-ja/type_checking_in_lean4_ja) にてIssue・Pull Requestで受け付けております。

翻訳に際して、機械翻訳サービス [DeepL翻訳](https://www.deepl.com/ja/translator) を参考にしました。

## バージョン情報

この翻訳は原文のcommit [fb99301873182d6a6712523c78502e68d965de05](https://github.com/lean-ja/type_checking_in_lean4_ja/commit/fb99301873182d6a6712523c78502e68d965de05) に基づいています。
