<!--
# Declarations
-->

# 宣言

<!--
Declarations are the big ticket items, and are the last domain elements we need to define.
-->

宣言は高額商品であり、最後の定義すべきな主要の要素です。

<!--
```
declarationInfo ::= Name, (universe params : List Level), (type : Expr)

declar ::= 
  Axiom declarationInfo
  | Definition declarationInfo (value : Expr) ReducibilityHint
  | Theorem declarationInfo (value : Expr) 
  | Opaque declarationInfo (value : Expr) 
  | Quot declarationInfo
  | InductiveType 
      declarationInfo
      is_recursive: Bool
      num_params: Nat
      num_indices: Nat
      -- The name of this type, and any others in a mutual block
      allIndNames: Name+
      -- The names of the constructors for *this* type only, 
      -- not including the constructors for mutuals that may 
      -- be in this block.
      constructorNames: Name*
      
  | Constructor 
      declarationInfo 
      (inductiveName : Name) 
      (numParams : Nat) 
      (numFields : Nat)

  | Recursor 
        declarationInfo 
        numParams : Nat
        numIndices : Nat
        numMotives : Nat
        numMinors : Nat
        RecRule+
        isK : Bool

RecRule ::= (constructor name : Name), (number of constructor args : Nat), (val : Expr)
```
-->
```
declarationInfo ::= Name, (universe params : List Level), (type : Expr)

declar ::= 
  Axiom declarationInfo
  | Definition declarationInfo (value : Expr) ReducibilityHint
  | Theorem declarationInfo (value : Expr) 
  | Opaque declarationInfo (value : Expr) 
  | Quot declarationInfo
  | InductiveType 
      declarationInfo
      is_recursive: Bool
      num_params: Nat
      num_indices: Nat
      -- この型の名前と相互ブロック内の他の型の名前
      allIndNames: Name+
      -- **この** 型のみについてのコンストラクタの名前
      -- このブロックに含まれる可能性のある相互のコンストラクタは含まない
      constructorNames: Name*
      
  | Constructor 
      declarationInfo 
      (inductiveName : Name) 
      (numParams : Nat) 
      (numFields : Nat)

  | Recursor 
        declarationInfo 
        numParams : Nat
        numIndices : Nat
        numMotives : Nat
        numMinors : Nat
        RecRule+
        isK : Bool

RecRule ::= (constructor name : Name), (number of constructor args : Nat), (val : Expr)
```


<!--
## Checking a declaration
-->

## 宣言のチェック


<!--
For all declarations, the following preliminary checks are performed before any additional procedures specific to certain kinds of declaration:
-->

すべての宣言に対して、特定の宣言に特化した追加の手続きが行われる前に、以下の予備的なチェックが行われます：

<!--
+ The universe parameters in the declaration's `declarationInfo` must not have duplicates. For example, a declaration `def Foo.{u, v, u} ...` would be prohibited.
-->

+ 宣言の `declarationInfo` 内の宇宙パラメータは重複してはなりません。例えば、`def Foo.{u, v, u} ...` という宣言は禁止されています。

<!--
+ The declaration's type must not have free variables; all variables in a "finished" declaration must correspond to a binder.
-->

+ 宣言の型は自由変数を持ってはなりません；「完成した」宣言の変数はすべて束縛子に対応しなければなりません。

<!--
+ The declaration's type must be a type (`infer declarationInfo.type` must produce a `Sort`). In Lean, a declaration `def Foo : Nat.succ := ..` is not permitted; `Nat.succ` is a value, not a type.
-->

+ 宣言の型は型でなければなりません（`infer declarationInfo.type` の結果は `Sort` でなければならない）。Lean では、`def Foo : Nat.succ := ..` という宣言は許可されていません；`Nat.succ` は値であり、型ではありません。

<!--
### Axiom
-->

### 公理

<!--
The only checks done against axioms are those done for all declarations which ensure the `declarationInfo` passes muster. If an axiom has a valid set of universe parameters and a valid type with no free variables, it is admitted to the environment.
-->

公理に対して行われる唯一のチェックは、`declarationInfo` が合格することを保証するすべての宣言に対して行われるチェックです。公理が有効な宇宙パラメータと有効な型を持ち、自由変数を持たない場合、その公理は環境に受け入れられます。

<!--
### Quot
-->

### 商

<!--
The `Quot` declarations are `Quot`, `Quot.mk`, `Quot.ind`, and `Quot.lift`. These declarations have prescribed types which are known to be sound within Lean's theory, so the environment's quotient declarations must match those types exactly. These types are hard-coded into kernel implementations since they are not prohibitively complex.
-->

`Quot` についての宣言は `Quot`・`Quot.mk`・`Quot.ind`・`Quot.lift` です。これらの宣言は Lean の理論で健全であるとされる型を持ち、環境の商についての宣言はこれらの型に正確に一致しなければなりません。これらの型は許されざるほど複雑というわけではないため、カーネルの実装にハードコードされています。

<!--
### Definition, theorem, opaque
-->

### 定義・定理・opaque

<!--
Definition, theorem, and opaque are interesting in that they both a type and a value. Checking these declarations involves inferring a type for the declaration's value, then asserting that the inferred type is definitionally equal to the ascribed type in the `declarationInfo`.
-->

定義・定理・opaque は型と値の両方を持つという点で興味深いものです。これらの宣言をチェックするには、宣言の値の型を推論し、その型が `declarationInfo` 内で定義された型と定義上等しいことを確認します。

<!--
In the case of a theorem, the `declarationInfo`'s type is what the user claims the type is, and therefore what the user is claiming to prove, while the value is what the user has offered as a proof of that type. Inferring the type of the received value amounts to checking what the proof is actually a proof of, and the definitional equality assertion ensures that the thing the value proves is actually what the user intended to prove.
-->

定理の場合、`declarationInfo` の型はユーザが主張する型であり、したがってユーザが証明することを主張しているもので、値はユーザによってその型の証明を提供されるものです。受け取った値の型を推論することは、その証明が実際に何を証明しているかをチェックすることに等しく、定義上の同値のチェックはその値が証明する事柄が実際にユーザが証明しようと意図したものであることを保証します。

<!--
#### Reducibility hints
-->

#### 簡約についてのヒント

<!--
Reducibility hints contain information about how a declaration should be unfolded. An `abbreviation` will generally always be unfolded, `opaque` will not be unfolded, and `regular N` might be unfolded depending on the value of `N`. The `regular` reducibility hints correspond to a definition's "height", which refers to the number of declarations that definition uses to define itself. A definition `x` with a value that refers to definition `y` will have a height value greater than `y`.
-->

簡約のヒントには、宣言がどのように展開されるべきかという情報が含まれています。`abbreviation` は一般的に常に展開され、`opaque` は展開されず、`regular N` は `N` の値によって展開されるかどうかが決定します。`regular` の簡約のヒントは定義の「高さ」に対応しています。この用語はその定義自体の定義に用いられている宣言の数を指します。定義 `y` を参照する値を持つ定義 `x` の高さは　`y` より大きくなります。
