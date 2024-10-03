<!--
# The big picture
-->

# 全体像

<!--
To give the reader a road map, the entire procedure of checking an export file consists of these steps:
-->

読者に道筋を示すために、エクスポートファイルをチェックする全手順を構成するステップを以下に挙げます：

<!--
+ Parse an export file, yielding a collection of components for each primitive sort: names, levels, and expressions, as well as a collection of declarations.
-->

+ エクスポートファイルを解析し、各プリミティブのあつまりのコンポーネントのコレクションを生成します：ここでプリミティブは名前・レベル・式と宣言のコレクションです。

<!--
+ The collection of parsed declarations represents an environment, which is a mapping from each declaration's name to the declaration itself; these are the actual targets of the type checking process.
-->

+ パースされた宣言のコレクションは環境を表します。環境とは各宣言の名前から宣言そのものへのマッピングです；これらの宣言こそ型チェックプロセスのターゲットです。

<!--
+ For each declaration in the environment, the kernel requires that the declaration is not already declared in the environment, has no duplicate universe parameters, that the declaration's type is actually a type and not a value (that `infer declar.ty` returns an expression `Sort <n>`), and that the declaration's type has no free variables.
-->

+ 環境内の各宣言に対してカーネルは、その宣言が環境内に既に宣言されていないこと、重複した宇宙パラメータを持たないこと、宣言の型が値ではなく実際に型であること（`infer declar.ty` が式 `Sort <n>` を返すこと）、宣言の型に自由変数がないことを要求します。

<!--
+ For definitions, theorems, and opaque declarations, assert that inferring the type of the definition's value yields an expression which is definitionally equal to the type the user assigned to the declaration. This is where the rubber meets the road in terms of asserting that proofs are correct, and for theorems, this is the step that corresponds to "the user says this is a proof of `P`, does the value actually constitute a valid proof of `P`".
-->

+ 定義・定理・opaque な宣言については、定義の値の型を推論することでユーザが宣言に割り当てた型と定義上等しい式が得られることを保証します。これは証明が正しいことを保証する点において肝心なところであり、定理については「ユーザはこれが `P` の証明であると言っているが、その値は実際に `P` の有効な証明を構成しているか？」ということに対応するステップです。

<!--
+ For inductive declarations, their constructors, and recursors, check that they are properly formed and comply with the rules of Lean's type theory (more on this later). 
-->

+ 帰納的な宣言・それのコンストラクタ・再帰子については、それらが適切に形成され、Lean の型理論の規則に準拠していることをチェックします（これについては後で詳しく説明します）。

<!--
+ If the export file includes the primitive declarations for quotient types, ensure those declarations have the correct types, and that the `Eq` type exists, and is defined properly (since quotients rely on equality).
-->

+ エクスポートファイルに商の型のプリミティブ宣言が含まれている場合は、それらの宣言が正しい型を持っていること、そして `Eq` 型が存在し、適切に定義されていることを確認します（商は同値に依存するため）。

<!--
+ Finally, pretty print any declarations requested by the user, so they can check that the declarations checked match the declarations they exported.
-->

+ 最後に、ユーザから要求された宣言をプリティプリントし、チェックした宣言がエクスポートした宣言と一致しているかどうかをユーザがチェックできるようにします。
