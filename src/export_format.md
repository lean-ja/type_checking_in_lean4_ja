<!-- # Export format -->

# エクスポートファイル

<!-- An exporter is a program which emits Lean declarations using the kernel language, for consumption by external type checkers. Producing an export file is a complete exit from the Lean ecosystem; the data in the file can be checked with entirely external software, and the exporter itself is not a trusted component. Rather than inspecting the export file itself to see whether the declarations were exported as the developer intended, the exported declarations are checked by the external checker, and are displayed back to the user by a pretty printer, which produces output far more readable than the export file. Readers can (and are encouraged to) write their own external checkers for Lean export files. -->

エクスポータは外部の型チェッカが利用できるように、カーネル言語を使って Lean の宣言の出力を行うプログラムのことです。出力されるエクスポートファイルは Lean のエコシステムから完全に分離されています；ファイル内のデータは完全に外部のソフトウェアでチェックすることができ、エクスポータ自体は信頼できるコンポーネントではありません。これらの宣言が開発者の意図通りにエクスポートされたかどうかを確認するためにエクスポートファイル自体を検査するのではなく、エクスポートされた宣言は外部のチェッカによってチェックされ、エクスポートファイルよりもはるかに読みやすい出力を生成するプリティプリンタによる表示をユーザは確認します。読者は Lean のエクスポートファイル用に独自の外部チェッカを書くことができます（また、推奨されます）。

<!-- The official exporter is [lean4export](https://github.com/leanprover/lean4export). -->

公式のエクスポータは [lean4export](https://github.com/leanprover/lean4export) です。

<!-- A description of the current export file format can be found below. There are also [ongoing discussions](https://github.com/leanprover/lean4export/issues/3) about how best to evolve the export format. -->

現在のエクスポートファイルのフォーマットは下記に詳細を挙げます。また、エクスポートフォーマットをどのように進化させるのがベストなのかについて [現在進行形の議論](https://github.com/leanprover/lean4export/issues/3) もあります。

## (ver 0.1.2)

<!-- For clarity, some of the compound items are decorated here with a name, for example `(name : T)`, but they appear in the export file as just an element of `T`. -->

分かりやすくするために、複合的な要素のいくつかは、例えば `(name : T)` のようにここでは名前で装飾されていますが、エクスポートファイルでは単なる `T` の要素として表示されます。

The export scheme for mutual and nested inductives is as follows: 
+ `Inductive.inductiveNames` contains the names of all types in the `mutual .. end` block. The names of any other inductive types used in a nested (but not mutual) construction will not be included.
+ `Inductive.constructorNames` contains the names of all constructors for THAT inductive type, and no others (no constructors of the other types in a mutual block, and no constructors from any nested construction).

相互の帰納とネストした帰納についてのエクスポートスキーマは次の通りです：
+ `Inductive.inductiveNames` は `mutual .. end` ブロック内のすべての型の名前を含みます。入れ子になった（しかし相互ではない）構成で使用される他の帰納型の名前は含まれません。
+ `Inductive.constructorNames` にはその帰納型のすべてのコンストラクタの名前が含まれ、他には何も含まれません（相互ブロック内の他の型のコンストラクタや、入れ子構造のコンストラクタは含まれません）。

<!-- **NOTE:** readers writing their own parsers and/or checkers should initialize names[0] as the anonymous name, and levels[0] as universe zero, as they are not emitted by the exporter, but are expected to occupy the name and level indices for 0. -->

**注意：** 独自のパーサやチェッカを書いている読者は name[0] を匿名の名前として初期化し、level[0] を宇宙レベル 0 として初期化すべきです。こうすることでこれらはエクスポータに出力されず、しかしこれらの名前とレベルの添字が 0 を占めることが期待されます。

```
File ::= ExportFormatVersion Item*

ExportFormatVersion ::= nat '.' nat '.' nat

Item ::= Name | Universe | Expr | RecRule | Declaration

Declaration ::= 
    | Axiom 
    | Quotient 
    | Definition 
    | Theorem 
    | Inductive 
    | Constructor 
    | Recursor

nidx, uidx, eidx, ridx ::= nat

Name ::=
  | nidx "#NS" nidx string
  | nidx "#NI" nidx nat

Universe ::=
  | uidx "#US"  uidx
  | uidx "#UM"  uidx uidx
  | uidx "#UIM" uidx uidx
  | uidx "#UP"  nidx

Expr ::=
  | eidx "#EV"  nat
  | eidx "#ES"  uidx
  | eidx "#EC"  nidx uidx*
  | eidx "#EA"  eidx eidx
  | eidx "#EL"  Info nidx eidx
  | eidx "#EP"  Info nidx eidx eidx
  | eidx "#EZ"  Info nidx eidx eidx eidx
  | eidx "#EJ"  nidx nat eidx
  | eidx "#ELN" nat
  | eidx "#ELS" (hexhex)*
  -- metadata node w/o extensions
  | eidx "#EM" mptr eidx

Info ::= "#BD" | "#BI" | "#BS" | "#BC"

Hint ::= "O" | "A" | "R" nat

RecRule ::= ridx "#RR" (ctorName : nidx) (nFields : nat) (val : eidx)

Axiom ::= "#AX" (name : nidx) (type : eidx) (uparams : uidx*)

Def ::= "#DEF" (name : nidx) (type : eidx) (value : eidx) (hint : Hint) (uparams : uidx*)
  
Theorem ::= "#THM" (name : nidx) (type : eidx) (value : eidx) (uparams: uidx*)

Quotient ::= "#QUOT" (name : nidx) (type : eidx) (uparams : uidx*)

Inductive ::= 
  "#IND"
  (name : nidx) 
  (type : eidx) 
  (isRecursive: 0 | 1)
  (isNested : 0 | 1)
  (numParams: nat) 
  (numIndices: nat)
  (numInductives: nat)
  (inductiveNames: nidx {numInductives})
  (numConstructors : nat) 
  (constructorNames : nidx {numConstructors}) 
  (uparams: uidx*)

Constructor ::= 
  "#CTOR"
  (name : nidx) 
  (type : eidx) 
  (parentInductive : nidx) 
  (constructorIndex : nat)
  (numParams : nat)
  (numFields : nat)
  (uparams: uidx*)

Recursor ::= 
  "#REC"
  (name : nidx)
  (type : eidx)
  (numInductives : nat)
  (inductiveNames: nidx {numInductives})
  (numParams : nat)
  (numIndices : nat)
  (numMotives : nat)
  (numMinors : nat)
  (numRules : nat)
  (recRules : ridx {numRules})
  (k : 1 | 0)
  (uparams : uidx*)
```
