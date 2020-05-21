<script async="async" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">MathJax.Hub.Config({"TeX": {"MAXBUFFER": 30720}})</script>

# 表記上のお約束

`WebAssembly`のコードは、モジュールをインスタンス化する時、または結果として得られたモジュールのインスタンス上でExportされた関数を呼び出すときに実行されます。

実行時の挙動は、プログラムの状態をモデル化する抽象マシンの観点から定義されます。
これには、オペランド値と制御構造を記録するスタックと、グローバル状態を含む抽象ストアが含まれます。

各命令に対して、その実行がプログラムの状態に与える影響を指定する規則があります。
さらに、モジュールのインスタンス化を記述する規則もあります。
[検証](Validation)と同様に、すべての規則は2つの等価な形式で与えられます。

- 散文的表記法では、人間にわかりやすい直感的な形で意味を記述します。
- 形式的表記法では、数学的な形式で規則を記述します。

## 散文的表記法

実行は抽象構文の各命令について様式化された段階的な規則によって指定されます。
これらの規則を記述する際には、以下の規則が採用されています。

- 実行規則は暗黙のうちに与えられたストア`S`を想定しています。
- 実行規則はまた、値、ラベル、フレームをpushまたはpopすることで変更される暗黙のスタックの存在も想定しています。
- 特定の規則では、スタックが少なくとも1つのフレームを含むことを要求します。最新のフレームはカレントフレームと呼ばれます。
- ストアとカレントフレームの両方は、それらのコンポーネントの一部を置き換えることで変更されます。このような置換はグローバルに適用されると仮定されます。
- 命令の実行がトラップされることがあります。この場合計算全体が中断され、それ以上のストアへの変更は行われません（他の計算はまだ開始できます）。(その後も他の計算を開始することができます。)
- 命令の実行は、次の命令を定義する指定されたターゲットへのジャンプで終了することもできます。
- 実行は、ブロックを形成する命令シーケンスに入ったり出たりすることができます。
- 命令シーケンスは、トラップやジャンプが発生しない限り、暗黙のうちに順番に実行されます。
- 規則の様々な場所で、プログラムの状態に関する重要な不変量を表現するアサーションが含まれています。

## 形式的表記法

正式な実行規則は運用セマンティクスを指定するための標準的なアプローチを使用してそれらを還元規則に還元します。
すべての規則は以下の一般的な形式を持っています。

<div>\[\mathit{configuration} \quad{\hookrightarrow}\quad \mathit{configuration}\]</div>

設定とはプログラムの状態を構文的に記述したものです。
各規則は実行の1つのステップを指定します。
与えられた設定に適用可能な還元規則が高々1つある限り、還元とそれに連なる実行は決定論的です。
`WebAssembly`には、これに対する例外がごくわずかしかありません。
それら例外はこの仕様で明示されています。

`WebAssembly`において、設定とは通常:

- 現在のストア`S`
- 現在の関数の呼び出しフレーム`F`
- 実行される命令のシーケンス

<div>を構成要素とするタプル<span>\((S; F; {\mathit{instr}}^\ast)\)</span>です。
</div>

不要な複雑生を避けるために、ストア`S`とフレーム`F`は、それらと関係ない還元規則では表記を省略されています。

スタックの個別の表現はありません。
その代わりに、構成の命令列の一部として便利に表現されています。
特に値は`const`命令と一致するように定義されており、`const`命令のシーケンスは、右に成長するオペランド「スタック」と解釈することができます。

ラベルとフレームは、似たようなものですので命令シーケンスの一部として定義されます。

還元の順序は、適切な評価コンテキストの定義によって決定されます。

還元はこれ以上の還元規則が適用できなくなると終了します。
`WebAssembly`型システムの健全性は元の命令シーケンスが、結果として得られるオペランドスタックの値として解釈できるconst命令のシーケンスに還元された場合、またはトラップが発生した場合にのみ、上記事実を保証しています。

### 付記

例として以下の命令シーケンスを考えることとします。

<div>\[({\mathsf{f64}}.{\mathsf{const}}~x_1)~({\mathsf{f64}}.{\mathsf{const}}~x_2)~{\mathsf{f64}}.{\mathsf{neg}}~({\mathsf{f64}}.{\mathsf{const}}~x_3)~{\mathsf{f64}}.{\mathsf{add}}~{\mathsf{f64}}.{\mathsf{mul}}\]</div>

これは3ステップの還元を経て最終形に落ち着きます。

<div>\[\begin{split}\begin{array}{ll}
& ({\mathsf{f64}}.{\mathsf{const}}~x_1)~({\mathsf{f64}}.{\mathsf{const}}~x_2)~{\mathsf{f64}}.{\mathsf{neg}}~({\mathsf{f64}}.{\mathsf{const}}~x_3)~{\mathsf{f64}}.{\mathsf{add}}~{\mathsf{f64}}.{\mathsf{mul}} \\
{\hookrightarrow} & ({\mathsf{f64}}.{\mathsf{const}}~x_1)~({\mathsf{f64}}.{\mathsf{const}}~x_4)~({\mathsf{f64}}.{\mathsf{const}}~x_3)~{\mathsf{f64}}.{\mathsf{add}}~{\mathsf{f64}}.{\mathsf{mul}} \\
{\hookrightarrow} & ({\mathsf{f64}}.{\mathsf{const}}~x_1)~({\mathsf{f64}}.{\mathsf{const}}~x_5)~{\mathsf{f64}}.{\mathsf{mul}} \\
{\hookrightarrow} & ({\mathsf{f64}}.{\mathsf{const}}~x_6) \\
\end{array}\end{split}\]</div>

<ul>
  <li><p><span>\(x_4 = -x_2\)</span></p></li>
  <li><p><span>\(x_5 = -x_2 + x_3\)</span></p></li>
  <li><p><span>\(x_6 = x_1 \cdot (-x_2 + x_3)\)</span></p></li>
</ul>

# ランタイムの構造

ストア、スタック、および値やモジュールインスタンスなどの`WebAssembly`抽象マシンを形成する他のランタイム構造は、追加の補助構文の観点から正確に作られています。

## 値

`WebAssembly`の計算は、4つの基本的な値の型を対照にしています。
それぞれ32ビット幅または64ビット幅の整数と浮動小数点データの値を操作します。

セマンティクスのほとんどの場所で、異なる型の値が発生する可能性があります。
曖昧さを避けるために、値はその型を明示的にする抽象構文で表現されます。
値を生成するconst命令と同じ表記法を再利用すると便利です。

<div>\[\begin{split}\begin{array}{llcl}
{\mathit{val}} &::=&
  {\mathsf{i32}}.{\mathsf{const}}~{\mathit{i32}} \\&&|&
  {\mathsf{i64}}.{\mathsf{const}}~{\mathit{i64}} \\&&|&
  {\mathsf{f32}}.{\mathsf{const}}~{\mathit{f32}} \\&&|&
  {\mathsf{f64}}.{\mathsf{const}}~{\mathit{f64}}
\end{array}\end{split}\]</div>

## 戻り値

戻り値は計算結果です。
複数乃至1つの値かトラップのどちらかです。

<div>\[\begin{split}\begin{array}{llcl}
{\mathit{result}} &::=&
  {\mathit{val}}^\ast \\&&|&
  {\mathsf{trap}}
\end{array}\end{split}\]</div>

## ストア

ストアは`WebAssembly`プログラムによって操作可能なすべてのグローバル状態を表します。
これは、抽象マシンのライフタイム中に割り当てられた関数、テーブル、メモリ、およびグローバルのすべてのインスタンスのランタイム表現で構成されています。

説明上では、ストアは各カテゴリの既存のインスタンスを列挙したレコードとして定義されます。

<div>\[\begin{split}\begin{array}{llll}
{\mathit{store}} &::=& \{~
  \begin{array}[t]{l@{~}ll}
  {\mathsf{funcs}} & {\mathit{funcinst}}^\ast, \\
  {\mathsf{tables}} & {\mathit{tableinst}}^\ast, \\
  {\mathsf{mems}} & {\mathit{meminst}}^\ast, \\
  {\mathsf{globals}} & {\mathit{globalinst}}^\ast ~\} \\
  \end{array}
\end{array}\end{split}\]</div>

実際にはストアから参照されなくなったオブジェクトを削除するために、ガベージコレクションのような技術を実装することができます。
しかしそのような技術は意味論的に観察できないためこの仕様の範囲外となります。

### 表記上のお約束

メタ変数`S`はコンテキストから明らかな場合にはストアの範囲になります。

## アドレス

ストア内の関数インスタンス、テーブルインスタンス、メモリインスタンス、およびグローバルインスタンスは、抽象アドレスで参照されます。
これらは、それぞれのストアコンポーネントへの単純なインデックスです。

<div>\[\begin{split}\begin{array}{llll}
{\mathit{addr}} &::=&
  0 ~|~ 1 ~|~ 2 ~|~ \dots \\
{\mathit{funcaddr}} &::=&
  {\mathit{addr}} \\
{\mathit{tableaddr}} &::=&
  {\mathit{addr}} \\
{\mathit{memaddr}} &::=&
  {\mathit{addr}} \\
{\mathit{globaladdr}} &::=&
  {\mathit{addr}} \\
\end{array}\end{split}\]</div>

エンベッダーは、そのアドレスに対応するExportされたストア オブジェクトに ID を割り当てることができます。
このIDが`WebAssembly`コード自体からは観測できない場合（関数インスタンスや不変グローバルなど）であっても、そのアドレスに対応するストア オブジェクトにIDを割り当てることができます。

### 付記

アドレスは、ランタイム・オブジェクトへの動的でグローバルに一意な参照であるのに対し、インデックスは元の定義への静的でモジュールローカルな参照です。
メモリアドレス`memaddr`は、ストア内のメモリインスタンスの抽象アドレスを示し、メモリインスタンス内のオフセットではありません。

ストアオブジェクトの割り当て数には特に制限がないため、論理アドレスは任意の大きな自然数になります。

## モジュールインスタンス

モジュールインスタンスは、モジュールのランタイム表現です。
モジュールのインスタンスを作成することで作成され、モジュールによってImport、定義、またはExportされたすべてのエンティティのランタイム表現を収集します。

<div>\[\begin{split}\begin{array}{llll}
{\mathit{moduleinst}} &::=& \{
  \begin{array}[t]{l@{~}ll}
  {\mathsf{types}} & {\mathit{functype}}^\ast, \\
  {\mathsf{funcaddrs}} & {\mathit{funcaddr}}^\ast, \\
  {\mathsf{tableaddrs}} & {\mathit{tableaddr}}^\ast, \\
  {\mathsf{memaddrs}} & {\mathit{memaddr}}^\ast, \\
  {\mathsf{globaladdrs}} & {\mathit{globaladdr}}^\ast, \\
  {\mathsf{exports}} & {\mathit{exportinst}}^\ast ~\} \\
  \end{array}
\end{array}\end{split}\]</div>

各コンポーネントは、Importされているか定義されているかに関わらず、元のモジュールからのそれぞれの宣言に対応するランタイムインスタンスを静的インデックスの順に参照します。
関数インスタンス、テーブルインスタンス、メモリインスタンス、グローバルインスタンスは、ストア内のそれぞれのアドレスを介して間接的に参照されます。

与えられたモジュールインスタンス内のすべてのExportインスタンスが異なる名前を持つことは、恒常的な前提条件です。

## 関数インスタンス

関数インスタンスは、関数のランタイム表現です。
これは、元のモジュールのランタイムモジュールインスタンスの上にある元の関数のクロージャです。
モジュールインスタンスは、関数の実行中に他の定義への参照を解決するために使用されます。

<div>\[\begin{split}\begin{array}{llll}
{\mathit{funcinst}} &::=&
  \{ {\mathsf{type}}~{\mathit{functype}}, {\mathsf{module}}~{\mathit{moduleinst}}, {\mathsf{code}}~{\mathit{func}} \} \\ &&|&
  \{ {\mathsf{type}}~{\mathit{functype}}, {\mathsf{hostcode}}~{\mathit{hostfunc}} \} \\
{\mathit{hostfunc}} &::=& \dots \\
\end{array}\end{split}\]</div>

ホスト関数とは、`WebAssembly`の外部で定義されますが、Importとしてモジュールに渡される関数のことです。
ホスト関数の定義と動作は、本仕様の範囲外です。
この仕様の目的のために、ホスト関数が呼び出されたとき、ランタイムの整合性を保証する特定の制約の範囲内で、非決定論的に振る舞うことを前提としています。

## 付記

関数インスタンスは不変であり、その同一性は WebAssembly コードでは観測できません。
しかし、エンベッダーは、それらのアドレスを区別するための暗黙的または明示的な手段を提供するかもしれません。

## テーブルインスタンス

テーブルインスタンスは、テーブルのランタイム表現です。
これは関数要素のベクトルとオプションで最大サイズを保持します（テーブルの定義サイトでテーブル型で指定された場合）。

各関数要素は、初期化されていないテーブル項目を表す空か、関数アドレスのいずれかです。
関数要素は、要素セグメントの実行によって、あるいはエンベッダーによって提供される外部の手段によって、変更することができます。

<div>\[\begin{split}\begin{array}{llll}
{\mathit{tableinst}} &::=&
  \{ {\mathsf{elem}}~{\mathit{vec}}({\mathit{funcelem}}), {\mathsf{max}}~{\mathit{u32}}^? \} \\
{\mathit{funcelem}} &::=&
  {\mathit{funcaddr}}^? \\
\end{array}\end{split}\]</div>

これは、要素ベクトルの長さが最大サイズを超えることがないというセマンティクスの不変性です（存在する場合）。

### 付記

`WebAssembly`の将来のバージョンでは、その他のテーブル要素型が追加されるかもしれません。

## メモリインスタンス

メモリインスタンスは、リニアメモリのランタイム表現です。
メモリの定義場所で最大サイズが指定されている場合は、オプションで最大バイト数のバイト列を保持します。

<div>\[\begin{split}\begin{array}{llll}
{\mathit{meminst}} &::=&
  \{ {\mathsf{data}}~{\mathit{vec}}({\mathit{byte}}), {\mathsf{max}}~{\mathit{u32}}^? \} \\
\end{array}\end{split}\]</div>

バイト列の長さは常にWebAssemblyのページサイズの倍数(定数65536)です。
メモリ型の場合と同様に、メモリインスタンス内の最大サイズはこのページサイズの単位で与えられます。

バイトは、メモリ命令、データセグメントの実行、またはエンベッダーによって提供される外部手段によって変更可能です。

ページサイズで割ったバイト列の長さが最大サイズを超えることはありません。

## グローバルインスタンス

グローバルインスタンスは、グローバル変数のランタイム表現です。
個々の値と、それが変更可能かどうかを示すフラグを保持します。

<div>\[\begin{split}\begin{array}{llll}
{\mathit{globalinst}} &::=&
  \{ {\mathsf{value}}~{\mathit{val}}, {\mathsf{mut}}~{\mathit{mut}} \} \\
\end{array}\end{split}\]</div>

変更可能なグローバルの値は、変数命令またはエンベッダーが提供する外部手段によって変更できます。

## Exportインスタンス

Exportインスタンスは、Exportのランタイム表現です。
Exportの名前と関連する外部値を定義します。

<div>\[\begin{split}\begin{array}{llll}
{\mathit{exportinst}} &::=&
  \{ {\mathsf{name}}~{\mathit{name}}, {\mathsf{value}}~{\mathit{externval}} \} \\
\end{array}\end{split}\]</div>

## 外部値

外部値は、ImportまたはExport可能なエンティティのランタイム表現です。
外部値は、共有ストア内の関数インスタンス、テーブルインスタンス、メモリインスタンス、グローバルインスタンスのいずれかを表すアドレスです。

<div>\[\begin{split}\begin{array}{llcl}
{\mathit{externval}} &::=& {\mathsf{func}}~{\mathit{funcaddr}} \\
&&|& {\mathsf{table}}~{\mathit{tableaddr}} \\
&&|& {\mathsf{mem}}~{\mathit{memaddr}} \\
&&|& {\mathsf{global}}~{\mathit{globaladdr}} \\
\end{array}\end{split}\]</div>

### 表記上のお約束

以下の補助記法は、外部値のシーケンスに対して定義されています。
これは、特定の種類のエントリを順序を保持してフィルタリングします。

<ul>
  <li><p><span>\({\mathrm{funcs}}({\mathit{externval}}^\ast) = [{\mathit{funcaddr}} ~|~ ({\mathsf{func}}~{\mathit{funcaddr}}) \in {\mathit{externval}}^\ast]\)</span></p></li>
  <li><p><span>\({\mathrm{tables}}({\mathit{externval}}^\ast) = [{\mathit{tableaddr}} ~|~ ({\mathsf{table}}~{\mathit{tableaddr}}) \in {\mathit{externval}}^\ast]\)</span></p></li>
  <li><p><span>\({\mathrm{mems}}({\mathit{externval}}^\ast) = [{\mathit{memaddr}} ~|~ ({\mathsf{mem}}~{\mathit{memaddr}}) \in {\mathit{externval}}^\ast]\)</span></p></li>
  <li><p><span>\({\mathrm{globals}}({\mathit{externval}}^\ast) = [{\mathit{globaladdr}} ~|~ ({\mathsf{global}}~{\mathit{globaladdr}}) \in {\mathit{externval}}^\ast]\)</span></p></li>
</ul>

## スタック

ストアの他にほとんどの命令は暗黙のスタックと対話します。
スタックには3種類のエントリが含まれています。

- 値: 命令のオペランド。
- ラベル: 分岐によって対象となるアクティブな構造化制御命令。
- アクティベーション: アクティブな関数呼び出しの呼び出しフレーム。

これらのエントリは、プログラムの実行中にスタック上で任意の順序で発生する可能性があります。
スタックエントリは、以下のように抽象構文で記述されます。

### 付記

オペランド、制御構造、および呼び出しのために別々のスタックを使用して`WebAssembly`のセマンティクスをモデル化することが可能です。
しかし、スタックは相互に依存しているので関連するスタックの高さについての追加のブックキーピングが必要になります。
この仕様の目的のためには、インターリーブされた表現の方がシンプルです。

### スタック上の値

そのまま表記されます。

### スタック上のラベル

ラベルは、引数のアリティ`n`とそれに関連する分岐ターゲットを持ち、構文的には命令シーケンスとして表現されます。

![](Pictures/Execution_0.png)

<div>直感的には、<span>\({\mathit{instr}}^\ast\)</span>は、元の制御構造体の代わりに、ブランチを取ったときに実行する継続部分です。
</div>

#### 付記

例えば、ループラベルは次のような形をしています。

<div>\[{\mathsf{label}}_n\{{\mathsf{loop}}~\dots~{\mathsf{end}}\}\]</div>

このラベルへの分岐を実行すると、ループが実行され、効果的に最初から再開されます。逆に、
単純なブロックラベルは次のような形式になります。

<div>\[{\mathsf{label}}_n\{\epsilon\}\]</div>

分岐する場合、空の継続は対象ブロックを終了させ、連続した命令で実行を進めることができます。

### アクティベーションフレーム

アクティベーションフレームは、各関数の戻り値のアリティ`n`を持ち、そのロケール（引数を含む）の値を静的なローカルインデックスに対応する順番で保持し、関数自身のモジュールインスタンスへの参照を保持します。

![](Pictures/Execution_1.png)

ローカルの値は、それぞれの変数命令によって変更されます。

### 表記上のお約束

<ul>
  <li><p>メタ変数<b>L</b>は、文脈から明らかなように、ラベルの範囲内にあります。</p></li>
  <li><p>メタ変数<b>F</b>は、コンテキストから明らかなフレームの範囲内にあります。</p></li>
  <li>以下の補助定義は、ブロック型を取り、現在のフレームでそれが示す関数型を検索します。
        <div>\[\begin{split}\begin{array}{lll}
{\mathrm{expand}}_F({\mathit{typeidx}}) &=& F.{\mathsf{module}}.{\mathsf{types}}[{\mathit{typeidx}}] \\
{\mathrm{expand}}_F([{\mathit{valtype}}^?]) &=& [] {\rightarrow} [{\mathit{valtype}}^?] \\
\end{array}\end{split}\]</div>
    </li>
</ul>

## 管理命令

本節は形式的記法にのみ関連します。

トラップ、呼び出し、制御命令の還元を表現するために、命令の構文を拡張して以下のような管理命令が含まれるようにしました。

![](Pictures/Execution_2.png)

`trap`命令は、トラップの発生を表します。
トラップは、入れ子になった命令シーケンスを通してバブルアップされ、最終的にはプログラム全体を単一のトラップ命令に減らし、突然の終了を示します。

`invoke`命令は、アドレスで識別される関数インスタンスの差し迫った呼び出しを表します。これにより、さまざまな形式の呼び出しの処理が統一されます。

`init_elem`命令と`init_data`命令は、モジュールのインスタンス化中に要素とデータセグメントの初期化を行います。

#### 付記

インスタンス化を個々の還元ステップに分割する理由は、スレッドのような将来の拡張と互換性のあるセマンティクスを提供するためです。

---

ラベル命令とフレーム命令は、ラベルとフレームを「スタック上」でモデル化します。
さらに、管理構文は、元の構造化制御命令または関数本体とそれらの命令シーケンスの入れ子構造をエンドマーカーで維持します。
そのようにして、内側の命令シーケンスの終わりは、外側のシーケンスの一部であるときに知られています。

#### 付記

例えば、ブロックの還元規則は以下のようになります。

<div>\[{\mathsf{label}}_0\{{\mathit{instr}}^\ast\}~{B}^l[{\mathsf{br}}~l]~{\mathsf{end}} \quad{\hookrightarrow}\quad {\mathit{instr}}^\ast\]</div>

これはブロックをラベル命令に置き換えますが、これはスタック上のラベルを「押す」と解釈できます。
`end`に到達したとき、つまり内部の命令列が空の列に還元されたとき、つまり結果として得られる値を表すn個の`const`命令の列に還元されたとき、ラベル命令はそれ自身の還元ルールによって除去されます。

<div>\[{\mathsf{label}}_m\{{\mathit{instr}}^\ast\}~{\mathit{val}}^n~{\mathsf{end}} \quad{\hookrightarrow}\quad {\mathit{val}}^n\]</div>

これは、スタックからラベルを取り除き、ローカルに蓄積されたオペランドの値だけを残すと解釈できます。

### ブロックコンテキスト

分岐の還元を指定するために、次の計算を行う場所を示す穴[_]を囲むラベルの数kをインデックスとしたブロックコンテキストの構文を以下のように定義しています。

![](Pictures/Execution_3.png)

この定義でにより、分岐命令や戻り命令の周囲にあるアクティブなラベルをインデックス化することができます。

#### 付記

例えば、単純な分岐の還元は次のように定義できます。

<div>\[{\mathsf{label}}_0\{{\mathit{instr}}^\ast\}~{B}^l[{\mathsf{br}}~l]~{\mathsf{end}} \quad{\hookrightarrow}\quad {\mathit{instr}}^\ast\]</div>

ここでは、コンテキストの穴[_]が分岐命令でインスタンス化されます。
分岐が発生すると、この規則は対象となるラベルと関連する命令シーケンスをラベルの続きに置き換えます。
選択されたラベルはラベルインデックス`l`で識別されます。
これは、ホップオーバーされなければならない周囲のラベル命令の数に対応します。

### 設定

コンフィギュレーションは、現在のストアと実行中のスレッドから構成されます。

スレッドとは、計算が実行されているモジュールインスタンスを参照するカレントフレーム(現在の関数がどこから来ているかを示す)に対して相対的に動作する命令の上での計算のことです。

<div>\[\begin{split}\begin{array}{llcl}
{\mathit{config}} &::=& {\mathit{store}}; {\mathit{thread}} \\
{\mathit{thread}} &::=& {\mathit{frame}}; {\mathit{instr}}^\ast \\
\end{array}\end{split}\]</div>

#### 付記

現在のバージョンの`WebAssembly`はシングルスレッドですが、将来的には複数のスレッドを使用した構成もサポートされるかもしれません。

### 評価コンテキスト

最後に、以下の評価コンテキストの定義とそれに関連した構造規則により、命令シーケンスや管理形式の中での還元やトラップの伝播が可能になります。

![](Pictures/Execution_4.png)

スレッドの命令シーケンスが結果、つまり値のシーケンスかトラップのいずれかに還元された場合、還元は終了します。

# 数値

数値プリミティブは一般的な方法で定義されており、ビット幅`N`のインデックスを持つ演算子によって定義されます。

いくつかの演算子は、いくつかの可能な結果（異なるNaN値など）のうちの1つを返すことができるため、非決定論的なものもあります。
技術的には、各演算子はこのようにして許容される値のセットを返します。
便宜上、決定論的な結果は単純な値として表現され、それぞれのシングルトン集合で識別されると仮定されます。

演算子の中には特定の入力に対して定義されていない部分的な演算子もあります。
技術的にはこれらの入力に対しては空の結果セットが返されます。

形式的な表記法では、各演算子は優先順位の低い順に適用される等式節によって定義されます。
つまり、与えられた引数に適用される最初の節が結果を定義します。

いくつかのケースでは、類似した節は±または∓という表記法を使用して1つに結合されます。
これらのプレースホルダが一つの節に複数存在する場合それらは一貫して解決されなければなりません。
つまりすべてのプレースホルダに対して上の符号が選択されるか、下の符号が選択されます。

### 付記

例えば、`fcopysign`演算子は次のように定義されています。

<div>\[\begin{split}\begin{array}{@{}lcll}
{\mathrm{fcopysign}}_N(\pm p_1, \pm p_2) &=& \pm p_1 \\
{\mathrm{fcopysign}}_N(\pm p_1, \mp p_2) &=& \mp p_1 \\
\end{array}\end{split}\]</div>

この定義は、各節を次のように2つに展開したものを略記したものと読むことにします。

<div>\[\begin{split}\begin{array}{@{}lcll}
{\mathrm{fcopysign}}_N(+ p_1, + p_2) &=& + p_1 \\
{\mathrm{fcopysign}}_N(- p_1, - p_2) &=& - p_1 \\
{\mathrm{fcopysign}}_N(+ p_1, - p_2) &=& - p_1 \\
{\mathrm{fcopysign}}_N(- p_1, + p_2) &=& + p_1 \\
\end{array}\end{split}\]</div>

### 表記上のお約束

<ul>
  <li><p>メタ変数 d はシングルビットの範囲を指定するために使用されます。</p></li>
  <li><p>メタ変数pは、浮動小数点値の倍数(nan および ∞を含む)の範囲を指定するために使用されます。</p></li>
  <li><p>メタ変数qは、nanまたは∞を除く有理数の倍数の範囲を表すために使用されます。</p></li>
  <li><p>表記法<span>\(f^{-1}\)</span>は、双射影関数fの逆数を表します。</p></li>
  <li>
    有理数の切り捨ては、通常の数学的定義を用いて、trunc(±q)と書きます。
    <div>\[\begin{split}\begin{array}{lll@{\qquad}l}
{\mathrm{trunc}}(\pm q) &=& \pm i & (\mathrel{\mbox{if}} i \in \mathbb{N} \wedge +q - 1 < i \leq +q) \\
\end{array}\end{split}\]</div>
  </li>
</ul>

## 表現

数値は、基本的にはビットの列として二進表現を持っています。

<div>\[\begin{split}\begin{array}{lll@{\qquad}l}
{\mathrm{bits}}_{\mathsf{i}N}(i) &=& {\mathrm{ibits}}_N(i) \\
{\mathrm{bits}}_{\mathsf{f}N}(z) &=& {\mathrm{fbits}}_N(z) \\
\end{array}\end{split}\]</div>

これらの関数はそれぞれ二項対立であり、それゆえに反転可能です。

### 整数

整数は2を基数とした符号なし数字として表現されます。

<div>\[\begin{split}\begin{array}{lll@{\qquad}l}
{\mathrm{ibits}}_N(i) &=& d_{N-1}~\dots~d_0 & (i = 2^{N-1}\cdot d_{N-1} + \dots + 2^0\cdot d_0) \\
\end{array}\end{split}\]</div>

∧、∨、または⊻のようなブール演算子は、それらを点単位で適用することで、同じ長さのビットシーケンスにリフトされます。

### 浮動小数点数

浮動小数点値は、[IEEE 754-2019](https://ieeexplore.ieee.org/document/8766229)で定義されたそれぞれのバイナリ形式で表現します。

<div>\[\begin{split}\begin{array}{lll@{\qquad}l}
{\mathrm{fbits}}_N(\pm (1+m\cdot 2^{-M})\cdot 2^e) &=& {\mathrm{fsign}}({\pm})~{\mathrm{ibits}}_E(e+{\mathrm{fbias}}_N)~{\mathrm{ibits}}_M(m) \\
{\mathrm{fbits}}_N(\pm (0+m\cdot 2^{-M})\cdot 2^e) &=& {\mathrm{fsign}}({\pm})~(0)^E~{\mathrm{ibits}}_M(m) \\
{\mathrm{fbits}}_N(\pm \infty) &=& {\mathrm{fsign}}({\pm})~(1)^E~(0)^M \\
{\mathrm{fbits}}_N(\pm {\mathsf{nan}}(n)) &=& {\mathrm{fsign}}({\pm})~(1)^E~{\mathrm{ibits}}_M(n) \\[1ex]
{\mathrm{fbias}}_N &=& 2^{E-1}-1 \\
{\mathrm{fsign}}({+}) &=& 0 \\
{\mathrm{fsign}}({-}) &=& 1 \\
\end{array}\end{split}\]</div>

<div><span>\(M = {\mathrm{signif}}(N)\)</span>であり、<span>\(E = {\mathrm{expon}}(N)\)</span>です。</div>

### ストレージ

数値はリトルエンディアンのバイト列としてメモリに格納されます。

<div>\[\begin{split}\begin{array}{lll@{\qquad}l}
{\mathrm{bytes}}_t(i) &=& {\mathrm{littleendian}}({\mathrm{bits}}_t(i)) \\[1ex]
{\mathrm{littleendian}}(\epsilon) &=& \epsilon \\
{\mathrm{littleendian}}(d^8~{d'}^\ast~) &=& {\mathrm{littleendian}}({d'}^\ast)~{\mathrm{ibits}}_8^{-1}(d^8) \\
\end{array}\end{split}\]</div>

ここでも、これらの関数は反転可能な二項演算です。

## 整数演算

### 符号化

整数演算子は iN 値に対して定義されています。

符号付き解釈を使用する演算子は、値が値の範囲の上半分にある場合（すなわち、その最上位ビットが1である場合）に2の補数を取る以下の定義を使用して値を変換します。

<div>\[\begin{split}\begin{array}{lll@{\qquad}l}
{\mathrm{signed}}_N(i) &=& i & (0 \leq i < 2^{N-1}) \\
{\mathrm{signed}}_N(i) &=& i - 2^N & (2^{N-1} \leq i < 2^N) \\
\end{array}\end{split}\]</div>

ここでも、これらの関数は反転可能な二項演算です。

### 真偽値化

述語、すなわちテストや関係演算子の整数結果は、条件に応じて値1または0を生成する以下の補助関数の助けを借りて定義されます。

<div>\[\begin{split}\begin{array}{lll@{\qquad}l}
{\mathrm{bool}}(C) &=& 1 & (\mathrel{\mbox{if}} C) \\
{\mathrm{bool}}(C) &=& 0 & (\mathrel{\mbox{otherwise}}) \\
\end{array}\end{split}\]</div>

---

<h3><span>\({\mathrm{iadd}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_1\)</span>と<span>\(i_2\)</span>で加算した結果をmodulo <span>\(2^N\)</span>したものを返します。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{iadd}}_N(i_1, i_2) &amp;=&amp; (i_1 + i_2) \mathbin{\mathrm{mod}} 2^N
\end{array}\]</div>

<h3><span>\({\mathrm{isub}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_1\)</span>と<span>\(i_2\)</span>で減算した結果をmodulo <span>\(2^N\)</span>したものを返します。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{isub}}_N(i_1, i_2) &amp;=&amp; (i_1 - i_2 + 2^N) \mathbin{\mathrm{mod}} 2^N
\end{array}\]</div>

<h3><span>\({\mathrm{imul}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_1\)</span>と<span>\(i_2\)</span>で乗算した結果をmodulo <span>\(2^N\)</span>したものを返します。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{imul}}_N(i_1, i_2) &amp;=&amp; (i_1 \cdot i_2) \mathbin{\mathrm{mod}} 2^N
\end{array}\]</div>

<h3><span>\({\mathrm{idiv\_u}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_2\)</span>が0の時結果は未定義です。</p></li>
  <li><p>そうでないならば、<span>\(i_1\)</span>を<span>\(i_2\)</span>で除算し、0に向けて切り捨てた結果をmodulo <span>\(2^N\)</span>したものを返します。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{idiv\_u}}_N(i_1, 0) &amp;=&amp; \{\} \\
{\mathrm{idiv\_u}}_N(i_1, i_2) &amp;=&amp; {\mathrm{trunc}}(i_1 / i_2) \\
\end{array}\end{split}\]</div>

### 付記

この関数はpartialです。

---

<h3><span>\({\mathrm{idiv\_s}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(j_1\)</span>が<span>\(i_1\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p><span>\(j_2\)</span>が<span>\(i_2\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p>もし<span>\(j_2\)</span>が0ならば、結果は未定義です。</p></li>
  <li><p>そうでないならば、<span>\(j_1\)</span>を<span>\(j_2\)</span>で除算した結果が<span>\(2^{N-1}\)</span>ならば、結果は未定義です。</p></li>
  <li><p>そうでないならば、<span>\(j_1\)</span>を<span>\(j_2\)</span>で除算した結果を0に向けて切り捨てたものが結果となります。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{idiv\_s}}_N(i_1, 0) &amp;=&amp; \{\} \\
{\mathrm{idiv\_s}}_N(i_1, i_2) &amp;=&amp; \{\} \qquad\qquad (\mathrel{\mbox{if}} {\mathrm{signed}}_N(i_1) / {\mathrm{signed}}_N(i_2) = 2^{N-1}) \\
{\mathrm{idiv\_s}}_N(i_1, i_2) &amp;=&amp; {\mathrm{signed}}_N^{-1}({\mathrm{trunc}}({\mathrm{signed}}_N(i_1) / {\mathrm{signed}}_N(i_2))) \\
\end{array}\end{split}\]</div>

### 付記

<div>\[(-2^{N-1})/(-1) = +2^{N-1}\]</div>

符号付きNビット2進整数では上記式の結果は表現可能な整数範囲外です。

この関数はpartialです。

---

<h3><span>\({\mathrm{irem\_u}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_2\)</span>が0ならば、結果は未定義です。</p></li>
  <li><p>そうでないならば、<span>\(i_1\)</span>を<span>\(i_2\)</span>で除算した剰余を結果とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{irem\_u}}_N(i_1, 0) &amp;=&amp; \{\} \\
{\mathrm{irem\_u}}_N(i_1, i_2) &amp;=&amp; i_1 - i_2 \cdot {\mathrm{trunc}}(i_1 / i_2) \\
\end{array}\end{split}\]</div>

### 付記

この関数はpartialです。

---

<h3><span>\({\mathrm{irem\_s}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(j_1\)</span>が<span>\(i_1\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p><span>\(j_2\)</span>が<span>\(i_2\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p>もし<span>\(j_2\)</span>が0ならば、結果は未定義です。</p></li>
  <li><p>そうでないならば、<span>\(j_1\)</span>を<span>\(j_2\)</span>で除算した剰余に<span>\(j_1\)</span>の符号を付加したものを結果とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{irem\_s}}_N(i_1, 0) &amp;=&amp; \{\} \\
{\mathrm{irem\_s}}_N(i_1, i_2) &amp;=&amp; {\mathrm{signed}}_N^{-1}(j_1 - j_2 \cdot {\mathrm{trunc}}(j_1 / j_2)) \\
  &amp;&amp; (\mathrel{\mbox{where}} j_1 = {\mathrm{signed}}_N(i_1) \wedge j_2 = {\mathrm{signed}}_N(i_2)) \\
\end{array}\end{split}\]</div>

### 付記

この関数はpartialです。

---

<h3><span>\({\mathrm{iand}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_1\)</span>と<span>\(i_2\)</span>の間でbit毎のand演算を行います。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{iand}}_N(i_1, i_2) &amp;=&amp; {\mathrm{ibits}}_N^{-1}({\mathrm{ibits}}_N(i_1) \wedge {\mathrm{ibits}}_N(i_2))
\end{array}\]</div>

<h3><span>\({\mathrm{ior}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_1\)</span>と<span>\(i_2\)</span>の間でbit毎のor演算を行います。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ior}}_N(i_1, i_2) &amp;=&amp; {\mathrm{ibits}}_N^{-1}({\mathrm{ibits}}_N(i_1) \vee {\mathrm{ibits}}_N(i_2))
\end{array}\]</div>

<h3><span>\({\mathrm{ixor}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_1\)</span>と<span>\(i_2\)</span>の間でbit毎のxor演算を行います。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ixor}}_N(i_1, i_2) &amp;=&amp; {\mathrm{ibits}}_N^{-1}({\mathrm{ibits}}_N(i_1) \veebar {\mathrm{ibits}}_N(i_2))
\end{array}\]</div>

<h3><span>\({\mathrm{ishl}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p>kが<span>\(i_2\)</span> modulo Nであるとします。</p></li>
  <li><p><span>\(i_1\)</span>に対してkビット左にシフトします。modulo <span>\(2^N\)</span></p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ishl}}_N(i_1, i_2) &amp;=&amp; {\mathrm{ibits}}_N^{-1}(d_2^{N-k}~0^k)
  &amp; (\mathrel{\mbox{if}} {\mathrm{ibits}}_N(i_1) = d_1^k~d_2^{N-k} \wedge k = i_2 \mathbin{\mathrm{mod}} N)
\end{array}\]</div>

<h3><span>\({\mathrm{ishr\_u}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p>kが<span>\(i_2\)</span> modulo Nであるとします。</p></li>
  <li><p><span>\(i_1\)</span>に対して右にkビット論理シフトします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ishr\_u}}_N(i_1, i_2) &amp;=&amp; {\mathrm{ibits}}_N^{-1}(0^k~d_1^{N-k})
  &amp; (\mathrel{\mbox{if}} {\mathrm{ibits}}_N(i_1) = d_1^{N-k}~d_2^k \wedge k = i_2 \mathbin{\mathrm{mod}} N)
\end{array}\]</div>

<h3><span>\({\mathrm{ishr\_s}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p>kが<span>\(i_2\)</span> modulo Nであるとします。</p></li>
  <li><p><span>\(i_1\)</span>に対して右にkビット算術シフトします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ishr\_s}}_N(i_1, i_2) &amp;=&amp; {\mathrm{ibits}}_N^{-1}(d_0^{k+1}~d_1^{N-k-1})
  &amp; (\mathrel{\mbox{if}} {\mathrm{ibits}}_N(i_1) = d_0~d_1^{N-k-1}~d_2^k \wedge k = i_2 \mathbin{\mathrm{mod}} N)
\end{array}\]</div>

<h3><span>\({\mathrm{irotl}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p>kが<span>\(i_2\)</span> modulo Nであるとします。</p></li>
  <li><p><span>\(i_1\)</span>を左にkビット回します。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{irotl}}_N(i_1, i_2) &amp;=&amp; {\mathrm{ibits}}_N^{-1}(d_2^{N-k}~d_1^k)
  &amp; (\mathrel{\mbox{if}} {\mathrm{ibits}}_N(i_1) = d_1^k~d_2^{N-k} \wedge k = i_2 \mathbin{\mathrm{mod}} N)
\end{array}\]</div>

<h3><span>\({\mathrm{irotr}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p>kが<span>\(i_2\)</span> modulo Nであるとします。</p></li>
  <li><p><span>\(i_1\)</span>を右にkビット回します。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{irotr}}_N(i_1, i_2) &amp;=&amp; {\mathrm{ibits}}_N^{-1}(d_2^k~d_1^{N-k})
  &amp; (\mathrel{\mbox{if}} {\mathrm{ibits}}_N(i_1) = d_1^{N-k}~d_2^k \wedge k = i_2 \mathbin{\mathrm{mod}} N)
\end{array}\]</div>

<h3><span>\({\mathrm{iclz}}_N(i)\)</span></h3>
<ul>
  <li><p>iの先行する0bitを数えます。iが0ならばすべて0であると見なします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{iclz}}_N(i) &amp;=&amp; k &amp; (\mathrel{\mbox{if}} {\mathrm{ibits}}_N(i) = 0^k~(1~d^\ast)^?)
\end{array}\]</div>

<h3><span>\({\mathrm{ictz}}_N(i)\)</span></h3>
<ul>
  <li><p>iの後継する0bitを数えます; iが0ならばすべて0であると見なします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ictz}}_N(i) &amp;=&amp; k &amp; (\mathrel{\mbox{if}} {\mathrm{ibits}}_N(i) = (d^\ast~1)^?~0^k)
\end{array}\]</div>

<h3><span>\({\mathrm{ipopcnt}}_N(i)\)</span></h3>
<ul>
  <li><p>iの0でないbitを数えます。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ipopcnt}}_N(i) &amp;=&amp; k &amp; (\mathrel{\mbox{if}} {\mathrm{ibits}}_N(i) = (0^\ast~1)^k~0^\ast)
\end{array}\]</div>

<h3><span>\({\mathrm{ieqz}}_N(i)\)</span></h3>

- iが0なら1を、そうでないならば0を戻り値とします。

<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ieqz}}_N(i) &amp;=&amp; {\mathrm{bool}}(i = 0)
\end{array}\]</div>

<h3><span>\({\mathrm{ieq}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_1\)</span>と<span>\(i_2\)</span>が等しいならば1を、そうでないならば0を戻り値とします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ieq}}_N(i_1, i_2) &amp;=&amp; {\mathrm{bool}}(i_1 = i_2)
\end{array}\]</div>

<h3><span>\({\mathrm{ine}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_1\)</span>と<span>\(i_2\)</span>が等しいならば0を、そうでないならば1を戻り値とします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ine}}_N(i_1, i_2) &amp;=&amp; {\mathrm{bool}}(i_1 \neq i_2)
\end{array}\]</div>

<h3><span>\({\mathrm{ilt\_u}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_1\)</span>が<span>\(i_2\)</span>より小さいならば1を、そうでないならば0を戻り値とします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ilt\_u}}_N(i_1, i_2) &amp;=&amp; {\mathrm{bool}}(i_1 &lt; i_2)
\end{array}\]</div>

<h3><span>\({\mathrm{ilt\_s}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(j_1\)</span>が<span>\(i_1\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p><span>\(j_2\)</span>が<span>\(i_2\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p><span>\(j_1\)</span>が<span>\(j_2\)</span>より小さいならば1を、そうでないならば0を戻り値とします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ilt\_s}}_N(i_1, i_2) &amp;=&amp; {\mathrm{bool}}({\mathrm{signed}}_N(i_1) &lt; {\mathrm{signed}}_N(i_2))
\end{array}\]</div>

<h3><span>\({\mathrm{igt\_u}}_N(i_1, i_2)\)</span></h3>
<ul>
    <li><p><span>\(i_1\)</span>が<span>\(i_2\)</span>より大きいならば1を、そうでないならば0を戻り値とします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{igt\_u}}_N(i_1, i_2) &amp;=&amp; {\mathrm{bool}}(i_1 &gt; i_2)
\end{array}\]</div>

<h3><span>\({\mathrm{igt\_s}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(j_1\)</span>が<span>\(i_1\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p><span>\(j_2\)</span>が<span>\(i_2\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p><span>\(j_1\)</span>が<span>\(j_2\)</span>より大きいならば1を、そうでないならば0を戻り値とします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{igt\_s}}_N(i_1, i_2) &amp;=&amp; {\mathrm{bool}}({\mathrm{signed}}_N(i_1) &gt; {\mathrm{signed}}_N(i_2))
\end{array}\]</div>

<h3><span>\({\mathrm{ile\_u}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_1\)</span>が<span>\(i_2\)</span>以下ならば1を、そうでないならば0を戻り値とします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ile\_u}}_N(i_1, i_2) &amp;=&amp; {\mathrm{bool}}(i_1 \leq i_2)
\end{array}\]</div>

<h3><span>\({\mathrm{ile\_s}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(j_1\)</span>が<span>\(i_1\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p><span>\(j_2\)</span>が<span>\(i_2\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p><span>\(j_1\)</span>が<span>\(j_2\)</span>以下ならば1を、そうでないならば0を戻り値とします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ile\_s}}_N(i_1, i_2) &amp;=&amp; {\mathrm{bool}}({\mathrm{signed}}_N(i_1) \leq {\mathrm{signed}}_N(i_2))
\end{array}\]</div>

<h3><span>\({\mathrm{ige\_u}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(i_1\)</span>が<span>\(i_2\)</span>以上ならば1を、そうでないならば0を戻り値とします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ige\_u}}_N(i_1, i_2) &amp;=&amp; {\mathrm{bool}}(i_1 \geq i_2)
\end{array}\]</div>

<h3><span>\({\mathrm{ige\_s}}_N(i_1, i_2)\)</span></h3>
<ul>
  <li><p><span>\(j_1\)</span>が<span>\(i_1\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p><span>\(j_2\)</span>が<span>\(i_2\)</span>について符号付き整数であると解釈した結果とします。</p></li>
  <li><p><span>\(j_1\)</span>が<span>\(j_2\)</span>以上ならば1を、そうでないならば0を戻り値とします。</p></li>
</ul>
<div>\[\begin{array}{&#64;{}lcll}
{\mathrm{ige\_s}}_N(i_1, i_2) &amp;=&amp; {\mathrm{bool}}({\mathrm{signed}}_N(i_1) \geq {\mathrm{signed}}_N(i_2))
\end{array}\]</div>

<h3><span>\({\mathrm{iextend}M\mathrm{\_s}}_N(i)\)</span></h3>
<ul>
  <li><p><a href="#extend_m_n">\({\mathrm{extend}^{\mathsf{s}}}_{M,N}(i)\)</a>を計算します</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{iextend}M\mathrm{\_s}}_{N}(i) &amp;=&amp; {\mathrm{extend}^{\mathsf{s}}}_{M,N}(i) \\
\end{array}\end{split}\]</div>

## 浮動小数点数演算

浮動小数点演算は[IEEE 754-2019](https://ieeexplore.ieee.org/document/8766229)規格に準拠しており、以下の条件を満たしています。

すべての演算子は、別段の指定がある場合を除き、ラウンド・トゥ・ニアレスト・タイ・トゥ・イーブンを使用します。デフォルト以外の有向丸め属性はサポートされていません。

オペレータがオペランドからNaNペイロードを伝播するという勧告に従うことは許可されるが、必須ではない。

すべての演算子は "ノンストップ "モードを使用し、浮動小数点の例外は他の方法では観測できない。特に、代替浮動小数点例外処理属性も、ステータスフラグ上の演算子もサポートされていません。クワイエットとシグナリングのNaNの間には、観測可能な違いはない。

### 付記

これらの制限の一部は、WebAssembly の将来のバージョンでは取り除かれる可能性があります。

### 丸め

丸めは、[IEEE 754-2019](https://ieeexplore.ieee.org/document/8766229)（セクション4.3.1）に対応して、常にround-to-nearest ties-to-evenです。

厳密浮動小数点数は、与えられたビット幅Ｎの浮動小数点数として正確に表現できる有理数です。

与えられた浮動小数点ビット幅Nの限界数は、その大きさが2の最小乗であり、幅Nの浮動小数点数として正確に表現できない正または負の数です（その大きさは、N=32の場合は2128、N=64の場合は21024です）。

候補数は、与えられたビット幅Nの正確な浮動小数点数、または正または負の限界数のいずれかです。

候補数のペアは候補数のペアz1,z2であり、その間には候補数は存在しません。

実数rは、次のようにしてビット幅 N の浮動小数点値に変換されます。

<ul>
  <li><p>rが0ならば0を戻り値とします。</p></li>
  <li><p>rが範囲内の場合、rを戻り値とします。</p></li>
  <li><p>rが上限値以上の場合、正のinfinityを戻り値とします。</p></li>
  <li><p>rが下限値以下の場合、負のinfinityを戻り値とします。</p></li>
  <li>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>は候補ペアとなります。<span>\(z_1 &lt; r &lt; z_2\)</span>について:
    <ul>
      <li><p>もし<span>\(|r - z_1| &lt; |r - z_2|\)</span>ならば、zを<span>\(z_1\)</span>であるとします。</p></li>
      <li><p>そうでないならば、<span>\(|r - z_1| &gt; |r - z_2|\)</span>ならば、zを<span>\(z_2\)</span>であるとします。</p></li>
      <li><p>そうでないならば、<span>\(|r - z_1| = |r - z_2|\)</span>と<span>\(z_1\)</span>の仮数がevenならば、zを<span>\(z_1\)</span>であるとします。</p></li>
      <li><p>そうでないならば、zを<span>\(z_2\)</span>.</p></li>
    </ul>
  </li>
  <li>もしzが0ならば、
    <ul>
      <li><p>もし<span>\(r &lt; 0\)</span>ならば、0を戻り値とします。</p></li>
      <li><p>そうでないならば、0を戻り値とします。</p></li>
    </ul>
    </li>
  <li>そうでないならば、zが境界上の数値であるとして、
    <ul>
      <li><p>もし<span>\(r &lt; 0\)</span>ならば、負のinfinityを戻り値とします。</p></li>
      <li><p>そうでないならば、正のinfinityを戻り値とします。</p></li>
    </ul>
  </li>
  <li><p>そうでないならば、zを戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{float}}_N(0) &amp;=&amp; 0 \\
{\mathrm{float}}_N(r) &amp;=&amp; r &amp; (\mathrel{\mbox{if}} r \in \mathrm{exact}_N) \\
{\mathrm{float}}_N(r) &amp;=&amp; +\infty &amp; (\mathrel{\mbox{if}} r \geq +\mathrm{limit}_N) \\
{\mathrm{float}}_N(r) &amp;=&amp; -\infty &amp; (\mathrel{\mbox{if}} r \leq -\mathrm{limit}_N) \\
{\mathrm{float}}_N(r) &amp;=&amp; \mathrm{closest}_N(r, z_1, z_2) &amp; (\mathrel{\mbox{if}} z_1 &lt; r &lt; z_2 \wedge (z_1,z_2) \in \mathrm{candidatepair}_N) \\[1ex]
\mathrm{closest}_N(r, z_1, z_2) &amp;=&amp; \mathrm{rectify}_N(r, z_1) &amp; (\mathrel{\mbox{if}} |r-z_1|&lt;|r-z_2|) \\
\mathrm{closest}_N(r, z_1, z_2) &amp;=&amp; \mathrm{rectify}_N(r, z_2) &amp; (\mathrel{\mbox{if}} |r-z_1|&gt;|r-z_2|) \\
\mathrm{closest}_N(r, z_1, z_2) &amp;=&amp; \mathrm{rectify}_N(r, z_1) &amp; (\mathrel{\mbox{if}} |r-z_1|=|r-z_2| \wedge \mathrm{even}_N(z_1)) \\
\mathrm{closest}_N(r, z_1, z_2) &amp;=&amp; \mathrm{rectify}_N(r, z_2) &amp; (\mathrel{\mbox{if}} |r-z_1|=|r-z_2| \wedge \mathrm{even}_N(z_2)) \\[1ex]
\mathrm{rectify}_N(r, \pm \mathrm{limit}_N) &amp;=&amp; \pm \infty \\
\mathrm{rectify}_N(r, 0) &amp;=&amp; 0 \qquad (r \geq 0) \\
\mathrm{rectify}_N(r, 0) &amp;=&amp; -0 \qquad (r &lt; 0) \\
\mathrm{rectify}_N(r, z) &amp;=&amp; z \\
\end{array}\end{split}\]</div>

補足すると以下の式を満たしています。

<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
\mathrm{exact}_N &amp;=&amp; {\mathit{fN}} \cap \mathbb{Q} \\
\mathrm{limit}_N &amp;=&amp; 2^{2^{{\mathrm{expon}}(N)-1}} \\
\mathrm{candidate}_N &amp;=&amp; \mathrm{exact}_N \cup \{+\mathrm{limit}_N, -\mathrm{limit}_N\} \\
\mathrm{candidatepair}_N &amp;=&amp; \{ (z_1, z_2) \in \mathrm{candidate}_N^2 ~|~ z_1 &lt; z_2 \wedge \forall z \in \mathrm{candidate}_N, z \leq z_1 \vee z \geq z_2\} \\[1ex]
\mathrm{even}_N((d + m\cdot 2^{-M}) \cdot 2^e) &amp;\Leftrightarrow&amp; m \mathbin{\mathrm{mod}} 2 = 0 \\
\mathrm{even}_N(\pm \mathrm{limit}_N) &amp;\Leftrightarrow&amp; \mathrm{true} \\
\end{array}\end{split}\]</div>

### NaN伝播

`fneg`, `fabs`, `fcopysign`以外の浮動小数点演算子の結果がNaNである場合、その符号は非決定性であり、ペイロードは以下のように計算されます。

演算子へのすべてのNaN入力のペイロードが正準値であれば(NaN入力がない場合を含む)、出力のペイロードも正準値となります。

そうでない場合、ペイロードはすべての算術NaNの中から非決定論的に選択されます。
すなわち、その最上位ビットは1であり、他のすべては未指定です。

この非決定論的な結果は、以下の補助関数で表現され、入力のセットから許容される出力のセットを生成します。

<div>\[\begin{split}\begin{array}{lll@{\qquad}l}
{\mathrm{nans}}_N\{z^\ast\} &=& \{ + {\mathsf{nan}}(n), - {\mathsf{nan}}(n) ~|~ n = {\mathrm{canon}}_N \}
  & (\mathrel{\mbox{if}} \forall {\mathsf{nan}}(n) \in z^\ast,~ n = {\mathrm{canon}}_N) \\
{\mathrm{nans}}_N\{z^\ast\} &=& \{ + {\mathsf{nan}}(n), - {\mathsf{nan}}(n) ~|~ n \geq {\mathrm{canon}}_N \}
  & (\mathrel{\mbox{otherwise}}) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fadd}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>のうちどちらかがNaNならば、<span>\({\mathrm{nans}}_N\{z_1, z_2\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が異なる符号のinfinityならば、<span>\({\mathrm{nans}}_N\{\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が同じ符号のinfinityならば、そのinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>か<span>\(z_2\)</span>がinfinityならば、そのinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が符号の異なる0同士であるならば、正の0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が符号の同じ0同士であるならば、その0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>か<span>\(z_2\)</span>が0ならば、もう一方を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が同じ大きさの絶対値を持ち、符号が異なるならば、正の0を戻り値とします。</p></li>
  <li><p><span>\(z_1\)</span>と<span>\(z_2\)</span>を加算し、近傍の表現可能な値に丸めます。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fadd}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_2\} \\
{\mathrm{fadd}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_1\} \\
{\mathrm{fadd}}_N(\pm \infty, \mp \infty) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fadd}}_N(\pm \infty, \pm \infty) &amp;=&amp; \pm \infty \\
{\mathrm{fadd}}_N(z_1, \pm \infty) &amp;=&amp; \pm \infty \\
{\mathrm{fadd}}_N(\pm \infty, z_2) &amp;=&amp; \pm \infty \\
{\mathrm{fadd}}_N(\pm 0, \mp 0) &amp;=&amp; 0 \\
{\mathrm{fadd}}_N(\pm 0, \pm 0) &amp;=&amp; \pm 0 \\
{\mathrm{fadd}}_N(z_1, \pm 0) &amp;=&amp; z_1 \\
{\mathrm{fadd}}_N(\pm 0, z_2) &amp;=&amp; z_2 \\
{\mathrm{fadd}}_N(\pm q, \mp q) &amp;=&amp; 0 \\
{\mathrm{fadd}}_N(z_1, z_2) &amp;=&amp; {\mathrm{float}}_N(z_1 + z_2) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fsub}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>どちらか一方がNaNならば、<span>\({\mathrm{nans}}_N\{z_1, z_2\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が同じ符号のinfinityならば、<span>\({\mathrm{nans}}_N\{\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が異なる符号のinfinityならば、<span>\(z_1\)</span>を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>がinfinityならば、そのinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>がinfinityならば、そのinfinityの符号を反転したものを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が符号の同じ0同士であるならば、正の0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が符号の異なる0同士であるならば、<span>\(z_1\)</span>を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>が0ならば、<span>\(z_1\)</span>を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が0ならば、return <span>\(z_2\)</span> negated.</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が等しいならば、正の0を戻り値とします。</p></li>
  <li><p><span>\(z_2\)</span>を<span>\(z_1\)</span>で減算し、近傍の表現可能な値に丸めます。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fsub}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_2\} \\
{\mathrm{fsub}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_1\} \\
{\mathrm{fsub}}_N(\pm \infty, \pm \infty) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fsub}}_N(\pm \infty, \mp \infty) &amp;=&amp; \pm \infty \\
{\mathrm{fsub}}_N(z_1, \pm \infty) &amp;=&amp; \mp \infty \\
{\mathrm{fsub}}_N(\pm \infty, z_2) &amp;=&amp; \pm \infty \\
{\mathrm{fsub}}_N(\pm 0, \pm 0) &amp;=&amp; 0 \\
{\mathrm{fsub}}_N(\pm 0, \mp 0) &amp;=&amp; \pm 0 \\
{\mathrm{fsub}}_N(z_1, \pm 0) &amp;=&amp; z_1 \\
{\mathrm{fsub}}_N(\pm 0, \pm q_2) &amp;=&amp; \mp q_2 \\
{\mathrm{fsub}}_N(\pm q, \pm q) &amp;=&amp; 0 \\
{\mathrm{fsub}}_N(z_1, z_2) &amp;=&amp; {\mathrm{float}}_N(z_1 - z_2) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fmul}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>どちらか一方がNaNならば、<span>\({\mathrm{nans}}_N\{z_1, z_2\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、one of <span>\(z_1\)</span>と<span>\(z_2\)</span>が0とthe other infinityならば、<span>\({\mathrm{nans}}_N\{\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が同じ符号のinfinityならば、正のinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が異なる符号のinfinityならば、負のinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>か<span>\(z_2\)</span>がinfinityと同じ符号の値ならば、正のinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>か<span>\(z_2\)</span>がinfinityと異なる符号の値ならば、負のinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が符号の同じ0同士であるならば、正の0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が符号の異なる0同士であるならば、負の0を戻り値とします。</p></li>
  <li><p><span>\(z_1\)</span>と<span>\(z_2\)</span>を乗算し、近傍の表現可能な値に丸めます。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fmul}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_2\} \\
{\mathrm{fmul}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_1\} \\
{\mathrm{fmul}}_N(\pm \infty, \pm 0) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fmul}}_N(\pm \infty, \mp 0) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fmul}}_N(\pm 0, \pm \infty) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fmul}}_N(\pm 0, \mp \infty) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fmul}}_N(\pm \infty, \pm \infty) &amp;=&amp; +\infty \\
{\mathrm{fmul}}_N(\pm \infty, \mp \infty) &amp;=&amp; -\infty \\
{\mathrm{fmul}}_N(\pm q_1, \pm \infty) &amp;=&amp; +\infty \\
{\mathrm{fmul}}_N(\pm q_1, \mp \infty) &amp;=&amp; -\infty \\
{\mathrm{fmul}}_N(\pm \infty, \pm q_2) &amp;=&amp; +\infty \\
{\mathrm{fmul}}_N(\pm \infty, \mp q_2) &amp;=&amp; -\infty \\
{\mathrm{fmul}}_N(\pm 0, \pm 0) &amp;=&amp; + 0 \\
{\mathrm{fmul}}_N(\pm 0, \mp 0) &amp;=&amp; - 0 \\
{\mathrm{fmul}}_N(z_1, z_2) &amp;=&amp; {\mathrm{float}}_N(z_1 \cdot z_2) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fdiv}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>どちらか一方がNaNならば、<span>\({\mathrm{nans}}_N\{z_1, z_2\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が両方infinitiyならば、<span>\({\mathrm{nans}}_N\{\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が両方0ならば、<span>\({\mathrm{nans}}_N\{z_1, z_2\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>がinfinityで<span>\(z_2\)</span>が符号の同じ値ならば、正のinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>がinfinityで<span>\(z_2\)</span>が符号の異なる値ならば、負のinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>がinfinityで<span>\(z_1\)</span>が符号の同じ値ならば、正の0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>がinfinityで<span>\(z_1\)</span>が符号の異なる値ならば、負の0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が0で<span>\(z_2\)</span>が符号の同じ値ならば、正の0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が0で<span>\(z_2\)</span>が符号の異なる値ならば、負の0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>が0で<span>\(z_1\)</span>が符号の同じ値ならば、正のinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>が0で<span>\(z_1\)</span>が符号の異なる値ならば、負のinfinityを戻り値とします。</p></li>
  <li><p><span>\(z_1\)</span>を<span>\(z_2\)</span>で除算し、近傍の表現可能な値に丸めます。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fdiv}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_2\} \\
{\mathrm{fdiv}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_1\} \\
{\mathrm{fdiv}}_N(\pm \infty, \pm \infty) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fdiv}}_N(\pm \infty, \mp \infty) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fdiv}}_N(\pm 0, \pm 0) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fdiv}}_N(\pm 0, \mp 0) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fdiv}}_N(\pm \infty, \pm q_2) &amp;=&amp; +\infty \\
{\mathrm{fdiv}}_N(\pm \infty, \mp q_2) &amp;=&amp; -\infty \\
{\mathrm{fdiv}}_N(\pm q_1, \pm \infty) &amp;=&amp; 0 \\
{\mathrm{fdiv}}_N(\pm q_1, \mp \infty) &amp;=&amp; -0 \\
{\mathrm{fdiv}}_N(\pm 0, \pm q_2) &amp;=&amp; 0 \\
{\mathrm{fdiv}}_N(\pm 0, \mp q_2) &amp;=&amp; -0 \\
{\mathrm{fdiv}}_N(\pm q_1, \pm 0) &amp;=&amp; +\infty \\
{\mathrm{fdiv}}_N(\pm q_1, \mp 0) &amp;=&amp; -\infty \\
{\mathrm{fdiv}}_N(z_1, z_2) &amp;=&amp; {\mathrm{float}}_N(z_1 / z_2) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fmin}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>どちらか一方がNaNならば、<span>\({\mathrm{nans}}_N\{z_1, z_2\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>か<span>\(z_2\)</span>が負のinfinityならば、負のinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>か<span>\(z_2\)</span>が正のinfinityならば、もう一方を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が符号の異なる0同士であるならば、負の0を戻り値とします。</p></li>
  <li><p><span>\(z_1\)</span>と<span>\(z_2\)</span>のうち小さい方を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fmin}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_2\} \\
{\mathrm{fmin}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_1\} \\
{\mathrm{fmin}}_N(+ \infty, z_2) &amp;=&amp; z_2 \\
{\mathrm{fmin}}_N(- \infty, z_2) &amp;=&amp; - \infty \\
{\mathrm{fmin}}_N(z_1, + \infty) &amp;=&amp; z_1 \\
{\mathrm{fmin}}_N(z_1, - \infty) &amp;=&amp; - \infty \\
{\mathrm{fmin}}_N(\pm 0, \mp 0) &amp;=&amp; -0 \\
{\mathrm{fmin}}_N(z_1, z_2) &amp;=&amp; z_1 &amp; (\mathrel{\mbox{if}} z_1 \leq z_2) \\
{\mathrm{fmin}}_N(z_1, z_2) &amp;=&amp; z_2 &amp; (\mathrel{\mbox{if}} z_2 \leq z_1) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fmax}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>どちらか一方がNaNならば、<span>\({\mathrm{nans}}_N\{z_1, z_2\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>か<span>\(z_2\)</span>が正のinfinityならば、正のinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>か<span>\(z_2\)</span>が負のinfinityならば、もう一方を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が符号の異なる0同士であるならば、正の0を戻り値とします。</p></li>
  <li><p><span>\(z_1\)</span>と<span>\(z_2\)</span>のうち大きい方を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fmax}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_2\} \\
{\mathrm{fmax}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n), z_1\} \\
{\mathrm{fmax}}_N(+ \infty, z_2) &amp;=&amp; + \infty \\
{\mathrm{fmax}}_N(- \infty, z_2) &amp;=&amp; z_2 \\
{\mathrm{fmax}}_N(z_1, + \infty) &amp;=&amp; + \infty \\
{\mathrm{fmax}}_N(z_1, - \infty) &amp;=&amp; z_1 \\
{\mathrm{fmax}}_N(\pm 0, \mp 0) &amp;=&amp; 0 \\
{\mathrm{fmax}}_N(z_1, z_2) &amp;=&amp; z_1 &amp; (\mathrel{\mbox{if}} z_1 \geq z_2) \\
{\mathrm{fmax}}_N(z_1, z_2) &amp;=&amp; z_2 &amp; (\mathrel{\mbox{if}} z_2 \geq z_1) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fcopysign}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>が同じ符号ならば、<span>\(z_1\)</span>を戻り値とします。</p></li>
  <li><p><span>\(z_1\)</span>の符号を反転したものを戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fcopysign}}_N(\pm p_1, \pm p_2) &amp;=&amp; \pm p_1 \\
{\mathrm{fcopysign}}_N(\pm p_1, \mp p_2) &amp;=&amp; \mp p_1 \\
\end{array}\end{split}\]</div>
</div>

<h3><span>\({\mathrm{fabs}}_N(z)\)</span></h3>
<ul>
  <li><p>もしzがNaNならば、return z with 正のsign.</p></li>
  <li><p>そうでないならば、zがinfinityならば、正のinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、zが0ならば、正の0を戻り値とします。</p></li>
  <li><p>そうでないならば、zが正のvalueならば、zを戻り値とします。</p></li>
  <li><p>zの符号を反転したものを戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fabs}}_N(\pm {\mathsf{nan}}(n)) &amp;=&amp; +{\mathsf{nan}}(n) \\
{\mathrm{fabs}}_N(\pm \infty) &amp;=&amp; +\infty \\
{\mathrm{fabs}}_N(\pm 0) &amp;=&amp; 0 \\
{\mathrm{fabs}}_N(\pm q) &amp;=&amp; +q \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fneg}}_N(z)\)</span></h3>
<ul>
  <li><p>もしzがNaNならば、return z with negated sign.</p></li>
  <li><p>そうでないならば、zがinfinityならば、そのinfinityの符号を反転したものを戻り値とします。</p></li>
  <li><p>そうでないならば、zが0ならば、その0の符号を反転したものを戻り値とします。</p></li>
  <li><p>zの符号を反転したものを戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fneg}}_N(\pm {\mathsf{nan}}(n)) &amp;=&amp; \mp {\mathsf{nan}}(n) \\
{\mathrm{fneg}}_N(\pm \infty) &amp;=&amp; \mp \infty \\
{\mathrm{fneg}}_N(\pm 0) &amp;=&amp; \mp 0 \\
{\mathrm{fneg}}_N(\pm q) &amp;=&amp; \mp q \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fsqrt}}_N(z)\)</span></h3>
<ul>
  <li><p>もしzがNaNならば、<span>\({\mathrm{nans}}_N\{z\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、z has a 負のsignならば、<span>\({\mathrm{nans}}_N\{\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、zが正のinfinityならば、正のinfinityを戻り値とします。</p></li>
  <li><p>そうでないならば、zが0ならば、その0を戻り値とします。</p></li>
  <li><p>zの平方根を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fsqrt}}_N(\pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n)\} \\
{\mathrm{fsqrt}}_N(- \infty) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fsqrt}}_N(+ \infty) &amp;=&amp; + \infty \\
{\mathrm{fsqrt}}_N(\pm 0) &amp;=&amp; \pm 0 \\
{\mathrm{fsqrt}}_N(- q) &amp;=&amp; {\mathrm{nans}}_N\{\} \\
{\mathrm{fsqrt}}_N(+ q) &amp;=&amp; {\mathrm{float}}_N\left(\sqrt{q}\right) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fceil}}_N(z)\)</span></h3>
<ul>
  <li><p>もしzがNaNならば、<span>\({\mathrm{nans}}_N\{z\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、zがinfinityならば、zを戻り値とします。</p></li>
  <li><p>そうでないならば、zが0ならば、zを戻り値とします。</p></li>
  <li><p>そうでないならば、zが0未満で-1より大きいならば、負の0を戻り値とします。</p></li>
  <li><p>z以上の最小の整数を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fceil}}_N(\pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n)\} \\
{\mathrm{fceil}}_N(\pm \infty) &amp;=&amp; \pm \infty \\
{\mathrm{fceil}}_N(\pm 0) &amp;=&amp; \pm 0 \\
{\mathrm{fceil}}_N(- q) &amp;=&amp; -0 &amp; (\mathrel{\mbox{if}} -1 &lt; -q &lt; 0) \\
{\mathrm{fceil}}_N(\pm q) &amp;=&amp; {\mathrm{float}}_N(i) &amp; (\mathrel{\mbox{if}} \pm q \leq i &lt; \pm q + 1) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{ffloor}}_N(z)\)</span></h3>
<ul>
  <li><p>もしzがNaNならば、<span>\({\mathrm{nans}}_N\{z\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、zがinfinityならば、zを戻り値とします。</p></li>
  <li><p>そうでないならば、zが0ならば、zを戻り値とします。</p></li>
  <li><p>そうでないならば、zが0より大きく1未満ならば、正の0を戻り値とします。</p></li>
  <li><p>z以下の最大の整数を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{ffloor}}_N(\pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n)\} \\
{\mathrm{ffloor}}_N(\pm \infty) &amp;=&amp; \pm \infty \\
{\mathrm{ffloor}}_N(\pm 0) &amp;=&amp; \pm 0 \\
{\mathrm{ffloor}}_N(+ q) &amp;=&amp; 0 &amp; (\mathrel{\mbox{if}} 0 &lt; +q &lt; 1) \\
{\mathrm{ffloor}}_N(\pm q) &amp;=&amp; {\mathrm{float}}_N(i) &amp; (\mathrel{\mbox{if}} \pm q - 1 &lt; i \leq \pm q) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{ftrunc}}_N(z)\)</span></h3>
<ul>
  <li><p>もしzがNaNならば、<span>\({\mathrm{nans}}_N\{z\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、zがinfinityならば、zを戻り値とします。</p></li>
  <li><p>そうでないならば、zが0ならば、zを戻り値とします。</p></li>
  <li><p>そうでないならば、zが0より大きく1未満ならば、正の0を戻り値とします。</p></li>
  <li><p>そうでないならば、zが0未満で-1より大きいならば、負の0を戻り値とします。</p></li>
  <li><p>zの絶対値に最も近い最大の整数にzと同じ符号を付けたものを戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{ftrunc}}_N(\pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n)\} \\
{\mathrm{ftrunc}}_N(\pm \infty) &amp;=&amp; \pm \infty \\
{\mathrm{ftrunc}}_N(\pm 0) &amp;=&amp; \pm 0 \\
{\mathrm{ftrunc}}_N(+ q) &amp;=&amp; 0 &amp; (\mathrel{\mbox{if}} 0 &lt; +q &lt; 1) \\
{\mathrm{ftrunc}}_N(- q) &amp;=&amp; -0 &amp; (\mathrel{\mbox{if}} -1 &lt; -q &lt; 0) \\
{\mathrm{ftrunc}}_N(\pm q) &amp;=&amp; {\mathrm{float}}_N(\pm i) &amp; (\mathrel{\mbox{if}} +q - 1 &lt; i \leq +q) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fnearest}}_N(z)\)</span></h3>
<ul>
  <li><p>もしzがNaNならば、<span>\({\mathrm{nans}}_N\{z\}\)</span>の要素を戻り値とします。</p></li>
  <li><p>そうでないならば、zがinfinityならば、zを戻り値とします。</p></li>
  <li><p>そうでないならば、zが0ならば、zを戻り値とします。</p></li>
  <li><p>そうでないならば、zが0より大きく0.5以下ならば、正の0を戻り値とします。</p></li>
  <li><p>そうでないならば、zが0未満-0.5以上ならば、負の0を戻り値とします。</p></li>
  <li><p>zに近い整数に丸めます; 等距離ならば偶数を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fnearest}}_N(\pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\pm {\mathsf{nan}}(n)\} \\
{\mathrm{fnearest}}_N(\pm \infty) &amp;=&amp; \pm \infty \\
{\mathrm{fnearest}}_N(\pm 0) &amp;=&amp; \pm 0 \\
{\mathrm{fnearest}}_N(+ q) &amp;=&amp; 0 &amp; (\mathrel{\mbox{if}} 0 &lt; +q \leq 0.5) \\
{\mathrm{fnearest}}_N(- q) &amp;=&amp; -0 &amp; (\mathrel{\mbox{if}} -0.5 \leq -q &lt; 0) \\
{\mathrm{fnearest}}_N(\pm q) &amp;=&amp; {\mathrm{float}}_N(\pm i) &amp; (\mathrel{\mbox{if}} |i - q| &lt; 0.5) \\
{\mathrm{fnearest}}_N(\pm q) &amp;=&amp; {\mathrm{float}}_N(\pm i) &amp; (\mathrel{\mbox{if}} |i - q| = 0.5 \wedge i~\mbox{even}) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{feq}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>どちらか一方がNaNならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が0ならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が等しいならば、1を戻り値とします。</p></li>
  <li><p>0を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{feq}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; 0 \\
{\mathrm{feq}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; 0 \\
{\mathrm{feq}}_N(\pm 0, \mp 0) &amp;=&amp; 1 \\
{\mathrm{feq}}_N(z_1, z_2) &amp;=&amp; {\mathrm{bool}}(z_1 = z_2) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fne}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>どちらか一方がNaNならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が0ならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が等しいならば、0を戻り値とします。</p></li>
  <li><p>1を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fne}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; 1 \\
{\mathrm{fne}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; 1 \\
{\mathrm{fne}}_N(\pm 0, \mp 0) &amp;=&amp; 0 \\
{\mathrm{fne}}_N(z_1, z_2) &amp;=&amp; {\mathrm{bool}}(z_1 \neq z_2) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{flt}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>どちらか一方がNaNならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が等しいならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が正のinfinityならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が負のinfinityならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>が正のinfinityならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>が負のinfinityならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が0ならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が<span>\(z_2\)</span>未満ならば、1を戻り値とします。</p></li>
  <li><p>0を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{flt}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; 0 \\
{\mathrm{flt}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; 0 \\
{\mathrm{flt}}_N(z, z) &amp;=&amp; 0 \\
{\mathrm{flt}}_N(+ \infty, z_2) &amp;=&amp; 0 \\
{\mathrm{flt}}_N(- \infty, z_2) &amp;=&amp; 1 \\
{\mathrm{flt}}_N(z_1, + \infty) &amp;=&amp; 1 \\
{\mathrm{flt}}_N(z_1, - \infty) &amp;=&amp; 0 \\
{\mathrm{flt}}_N(\pm 0, \mp 0) &amp;=&amp; 0 \\
{\mathrm{flt}}_N(z_1, z_2) &amp;=&amp; {\mathrm{bool}}(z_1 &lt; z_2) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fgt}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>どちらか一方がNaNならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が等しいならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が正のinfinityならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が負のinfinityならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>が正のinfinityならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>が負のinfinityならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が0ならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が<span>\(z_2\)</span>より大きいならば、1を戻り値とします。</p></li>
  <li><p>0を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fgt}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; 0 \\
{\mathrm{fgt}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; 0 \\
{\mathrm{fgt}}_N(z, z) &amp;=&amp; 0 \\
{\mathrm{fgt}}_N(+ \infty, z_2) &amp;=&amp; 1 \\
{\mathrm{fgt}}_N(- \infty, z_2) &amp;=&amp; 0 \\
{\mathrm{fgt}}_N(z_1, + \infty) &amp;=&amp; 0 \\
{\mathrm{fgt}}_N(z_1, - \infty) &amp;=&amp; 1 \\
{\mathrm{fgt}}_N(\pm 0, \mp 0) &amp;=&amp; 0 \\
{\mathrm{fgt}}_N(z_1, z_2) &amp;=&amp; {\mathrm{bool}}(z_1 &gt; z_2) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fle}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>どちらか一方がNaNならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が等しいならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が正のinfinityならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が負のinfinityならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>が正のinfinityならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>が負のinfinityならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が0ならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が<span>\(z_2\)</span>以下ならば、1を戻り値とします。</p></li>
  <li><p>0を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fle}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; 0 \\
{\mathrm{fle}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; 0 \\
{\mathrm{fle}}_N(z, z) &amp;=&amp; 1 \\
{\mathrm{fle}}_N(+ \infty, z_2) &amp;=&amp; 0 \\
{\mathrm{fle}}_N(- \infty, z_2) &amp;=&amp; 1 \\
{\mathrm{fle}}_N(z_1, + \infty) &amp;=&amp; 1 \\
{\mathrm{fle}}_N(z_1, - \infty) &amp;=&amp; 0 \\
{\mathrm{fle}}_N(\pm 0, \mp 0) &amp;=&amp; 1 \\
{\mathrm{fle}}_N(z_1, z_2) &amp;=&amp; {\mathrm{bool}}(z_1 \leq z_2) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{fge}}_N(z_1, z_2)\)</span></h3>
<ul>
  <li><p>もし<span>\(z_1\)</span>と<span>\(z_2\)</span>どちらか一方がNaNならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が等しいならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が正のinfinityならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が負のinfinityならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>が正のinfinityならば、0を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_2\)</span>が負のinfinityならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>と<span>\(z_2\)</span>が0ならば、1を戻り値とします。</p></li>
  <li><p>そうでないならば、<span>\(z_1\)</span>が<span>\(z_2\)</span>以下ならば、1を戻り値とします。</p></li>
  <li><p>0を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{&#64;{}lcll}
{\mathrm{fge}}_N(\pm {\mathsf{nan}}(n), z_2) &amp;=&amp; 0 \\
{\mathrm{fge}}_N(z_1, \pm {\mathsf{nan}}(n)) &amp;=&amp; 0 \\
{\mathrm{fge}}_N(z, z) &amp;=&amp; 1 \\
{\mathrm{fge}}_N(+ \infty, z_2) &amp;=&amp; 1 \\
{\mathrm{fge}}_N(- \infty, z_2) &amp;=&amp; 0 \\
{\mathrm{fge}}_N(z_1, + \infty) &amp;=&amp; 0 \\
{\mathrm{fge}}_N(z_1, - \infty) &amp;=&amp; 1 \\
{\mathrm{fge}}_N(\pm 0, \mp 0) &amp;=&amp; 1 \\
{\mathrm{fge}}_N(z_1, z_2) &amp;=&amp; {\mathrm{bool}}(z_1 \geq z_2) \\
\end{array}\end{split}\]</div>

## 変換

<h3><span>\({\mathrm{extend}^{\mathsf{u}}}_{M,N}(i)\)</span></h3>
<ul>
  <li><p>iを戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{extend}^{\mathsf{u}}}_{M,N}(i) &amp;=&amp; i \\
\end{array}\end{split}\]</div>

### 付記

抽象構文では、符号なし拡張は同じ値を再解釈するだけです。

---

<h3><span>\({\mathrm{extend}^{\mathsf{s}}}_{M,N}(i)\)</span></h3>
<ul>
  <li><p>Let j be the signed interpretation of i of size M</p></li>
  <li><p>Return the two’s complement of j relative to size N</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{extend}^{\mathsf{s}}}_{M,N}(i) &amp;=&amp; {\mathrm{signed}}_N^{-1}({\mathrm{signed}}_M(i)) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{wrap}}_{M,N}(i)\)</span></h3>
<ul>
  <li><p>iを<span>\(2^N\)</span>で割った剰余を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{wrap}}_{M,N}(i) &amp;=&amp; i \mathbin{\mathrm{mod}} 2^N \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{trunc}^{\mathsf{u}}}_{M,N}(z)\)</span></h3>
<ul>
  <li><p>もしzがNaNであるならば、戻り値は未定義です。</p></li>
  <li><p>あるいはzがinfinityであるならば、戻り値は未定義です。</p></li>
  <li><p>あるいはzが数値であり、<span>\({\mathrm{trunc}}(z)\)</span>が対象の型の値の範囲内に存在するならばその値を戻り値とします。</p></li>
  <li><p>そうでないならば、戻り値は未定義です。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{trunc}^{\mathsf{u}}}_{M,N}(\pm {\mathsf{nan}}(n)) &amp;=&amp; \{\} \\
{\mathrm{trunc}^{\mathsf{u}}}_{M,N}(\pm \infty) &amp;=&amp; \{\} \\
{\mathrm{trunc}^{\mathsf{u}}}_{M,N}(\pm q) &amp;=&amp; {\mathrm{trunc}}(\pm q) &amp; (\mathrel{\mbox{if}} -1 &lt; {\mathrm{trunc}}(\pm q) &lt; 2^N) \\
{\mathrm{trunc}^{\mathsf{u}}}_{M,N}(\pm q) &amp;=&amp; \{\} &amp; (\mathrel{\mbox{otherwise}}) \\
\end{array}\end{split}\]</div>

### 付記

この命令はpartialです。
Nanに対して定義されていません。

---

<h3><span>\({\mathrm{trunc}^{\mathsf{s}}}_{M,N}(z)\)</span></h3>
<ul>
  <li><p>もしzがNaNであるならば、戻り値は未定義です。</p></li>
  <li><p>あるいはzがinfinityであるならば、戻り値は未定義です。</p></li>
  <li><p>あるいはzが数値であり、<span>\({\mathrm{trunc}}(z)\)</span>が対象の型の値の範囲内に存在するならばその値を戻り値とします。</p></li>
  <li><p>そうでないならば、戻り値は未定義です。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{trunc}^{\mathsf{s}}}_{M,N}(\pm {\mathsf{nan}}(n)) &amp;=&amp; \{\} \\
{\mathrm{trunc}^{\mathsf{s}}}_{M,N}(\pm \infty) &amp;=&amp; \{\} \\
{\mathrm{trunc}^{\mathsf{s}}}_{M,N}(\pm q) &amp;=&amp; {\mathrm{trunc}}(\pm q) &amp; (\mathrel{\mbox{if}} -2^{N-1} - 1 &lt; {\mathrm{trunc}}(\pm q) &lt; 2^{N-1}) \\
{\mathrm{trunc}^{\mathsf{s}}}_{M,N}(\pm q) &amp;=&amp; \{\} &amp; (\mathrel{\mbox{otherwise}}) \\
\end{array}\end{split}\]</div>

### 付記

この命令はpartialです。
Nanに対して定義されていません。

---

<h3><span>\({\mathrm{trunc\_sat\_u}}_{M,N}(z)\)</span></h3>
<ul>
  <li><p>もしzがNaNであるならば、0を戻り値とします。</p></li>
  <li><p>あるいはzが負のinfinityであるならば、0を戻り値とします。</p></li>
  <li><p>あるいはzが正のinfinityであるならば、<span>\(2^N - 1\)</span>を戻り値とします。</p></li>
  <li><p>あるいは<span>\({\mathrm{trunc}}(z)\)</span>が0未満ならば、0を戻り値とします。</p></li>
  <li><p>あるいは<span>\({\mathrm{trunc}}(z)\)</span>が<span>\(2^N - 1\)</span>より大きいならば、<span>\(2^N - 1\)</span>を戻り値とします。</p></li>
  <li><p><span>\({\mathrm{trunc}}(z)\)</span>を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{trunc\_sat\_u}}_{M,N}(\pm {\mathsf{nan}}(n)) &amp;=&amp; 0 \\
{\mathrm{trunc\_sat\_u}}_{M,N}(- \infty) &amp;=&amp; 0 \\
{\mathrm{trunc\_sat\_u}}_{M,N}(+ \infty) &amp;=&amp; 2^N - 1 \\
{\mathrm{trunc\_sat\_u}}_{M,N}(- q) &amp;=&amp; 0 &amp; (\mathrel{\mbox{if}} {\mathrm{trunc}}(- q) &lt; 0) \\
{\mathrm{trunc\_sat\_u}}_{M,N}(+ q) &amp;=&amp; 2^N - 1 &amp; (\mathrel{\mbox{if}} {\mathrm{trunc}}(+ q) &gt; 2^N - 1) \\
{\mathrm{trunc\_sat\_u}}_{M,N}(\pm q) &amp;=&amp; {\mathrm{trunc}}(\pm q) &amp; (otherwise) \\
\end{array}\end{split}\]</div>

### 付記

この命令はpartialです。
Nanに対して定義されていません。

---

<h3><span>\({\mathrm{trunc\_sat\_s}}_{M,N}(z)\)</span></h3>
<ul>
  <li><p>もしzがNaNであるならば、0を戻り値とします。</p></li>
  <li><p>あるいはzが負のinfinityであるならば、<span>\(-2^{N-1}\)</span>を戻り値とします。</p></li>
  <li><p>あるいはzが正のinfinityであるならば、<span>\(2^{N-1} - 1\)</span>を戻り値とします。</p></li>
  <li><p>あるいは<span>\({\mathrm{trunc}}(z)\)</span>が<span>\(-2^{N-1}\)</span>未満ならば、<span>\(-2^{N-1}\)</span>を戻り値とします。</p></li>
  <li><p>あるいは<span>\({\mathrm{trunc}}(z)\)</span>が<span>\(2^{N-1} - 1\)</span>より大きいならば、<span>\(2^{N-1} - 1\)</span>を戻り値とします。</p></li>
  <li><p><span>\({\mathrm{trunc}}(z)\)</span>を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{trunc\_sat\_s}}_{M,N}(\pm {\mathsf{nan}}(n)) &amp;=&amp; 0 \\
{\mathrm{trunc\_sat\_s}}_{M,N}(- \infty) &amp;=&amp; -2^{N-1} \\
{\mathrm{trunc\_sat\_s}}_{M,N}(+ \infty) &amp;=&amp; 2^{N-1}-1 \\
{\mathrm{trunc\_sat\_s}}_{M,N}(- q) &amp;=&amp; -2^{N-1} &amp; (\mathrel{\mbox{if}} {\mathrm{trunc}}(- q) &lt; -2^{N-1}) \\
{\mathrm{trunc\_sat\_s}}_{M,N}(+ q) &amp;=&amp; 2^{N-1} - 1 &amp; (\mathrel{\mbox{if}} {\mathrm{trunc}}(+ q) &gt; 2^{N-1} - 1) \\
{\mathrm{trunc\_sat\_s}}_{M,N}(\pm q) &amp;=&amp; {\mathrm{trunc}}(\pm q) &amp; (otherwise) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{promote}}_{M,N}(z)\)</span></h3>
<ul>
  <li><p>zがcanonical Nanであるならば、<span>\({\mathrm{nans}}_N\{\}\)</span> (例：サイズNのcanonical Nan)の要素を戻り値とします。</p></li>
  <li><p>あるいはzがNaNであるならば、<span>\({\mathrm{nans}}_N\{\pm {\mathsf{nan}}(1)\}\)</span> (例：サイズNのarithmetic Nan)の要素を戻り値とします。</p></li>
  <li><p>zを戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{promote}}_{M,N}(\pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\} &amp; (\mathrel{\mbox{if}} n = {\mathrm{canon}}_N) \\
{\mathrm{promote}}_{M,N}(\pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{+ {\mathsf{nan}}(1)\} &amp; (\mathrel{\mbox{otherwise}}) \\
{\mathrm{promote}}_{M,N}(z) &amp;=&amp; z \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{demote}}_{M,N}(z)\)</span></h3>
<ul>
  <li><p>zがcanonical Nanであるならば、<span>\({\mathrm{nans}}_N\{\}\)</span> (例：サイズNのcanonical Nan)の要素を戻り値とします。</p></li>
  <li><p>あるいはzがNaNであるならば、<span>\({\mathrm{nans}}_N\{\pm {\mathsf{nan}}(1)\}\)</span> (例：サイズNのNan)の要素を戻り値とします。</p></li>
  <li><p>あるいはzがinfinityであるならば、そのinfinityを戻り値とします。</p></li>
  <li><p>あるいはzが0であるならば、その0を戻り値とします。</p></li>
  <li><p><span>\({\mathrm{float}}_N(z)\)</span>を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{demote}}_{M,N}(\pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{\} &amp; (\mathrel{\mbox{if}} n = {\mathrm{canon}}_N) \\
{\mathrm{demote}}_{M,N}(\pm {\mathsf{nan}}(n)) &amp;=&amp; {\mathrm{nans}}_N\{+ {\mathsf{nan}}(1)\} &amp; (\mathrel{\mbox{otherwise}}) \\
{\mathrm{demote}}_{M,N}(\pm \infty) &amp;=&amp; \pm \infty \\
{\mathrm{demote}}_{M,N}(\pm 0) &amp;=&amp; \pm 0 \\
{\mathrm{demote}}_{M,N}(\pm q) &amp;=&amp; {\mathrm{float}}_N(\pm q) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{convert}^{\mathsf{u}}}_{M,N}(i)\)</span></h3>
<ul>
  <li><p><span>\({\mathrm{float}}_N(i)\)</span>を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{convert}^{\mathsf{u}}}_{M,N}(i) &amp;=&amp; {\mathrm{float}}_N(i) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{convert}^{\mathsf{s}}}_{M,N}(i)\)</span></h3>
<ul>
  <li><p>jiのsigned interpretationであるとします。</p></li>
  <li><p><span>\({\mathrm{float}}_N(j)\)</span>を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{convert}^{\mathsf{u}}}_{M,N}(i) &amp;=&amp; {\mathrm{float}}_N({\mathrm{signed}}_M(i)) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathrm{reinterpret}}_{t_1,t_2}(c)\)</span></h3>
<ul>
  <li><p><span>\(d^\ast\)</span>がビット列<span>\({\mathrm{bits}}_{t_1}(c)\)</span>であるとします。</p></li>
  <li><p><span>\({\mathrm{bits}}_{t_2}(c') = d^\ast\)</span>となる定数<span>\(c'\)</span>を戻り値とします。</p></li>
</ul>
<div>\[\begin{split}\begin{array}{lll&#64;{\qquad}l}
{\mathrm{reinterpret}}_{t_1,t_2}(c) &amp;=&amp; {\mathrm{bits}}_{t_2}^{-1}({\mathrm{bits}}_{t_1}(c)) \\
\end{array}\end{split}\]</div>

# 命令

`WebAssembly`の計算は個々の命令を実行することで行われます。

## 算術演算命令

数値命令は一般的な数値演算子の観点から定義されます。
数値命令とその基礎となる演算子との対応付けは以下の定義で表されます。

<div>\[\begin{split}\begin{array}{lll@{\qquad}l}
\mathit{op}_{\mathsf{i}N}(n_1,\dots,n_k) &=& \mathrm{i}\mathit{op}_N(n_1,\dots,n_k) \\
\mathit{op}_{\mathsf{f}N}(z_1,\dots,z_k) &=& \mathrm{f}\mathit{op}_N(z_1,\dots,z_k) \\
\end{array}\end{split}\]</div>

また、変換演算子については

<div>\[\begin{split}\begin{array}{lll@{\qquad}l}
\mathit{cvtop}^{{\mathit{sx}}^?}_{t_1,t_2}(c) &=& \mathit{cvtop}^{{\mathit{sx}}^?}_{|t_1|,|t_2|}(c) \\
\end{array}\end{split}\]</div>

基礎となる演算子が部分的である場合、対応する命令は結果が定義されていない場合にトラップします。

基礎となる演算子が非決定論的である場合、複数の可能なNaN値のうちの1つを返す可能性があるため、対応する命令も同様です。

### 付記

<div>例えば、オペランド<span>\(i_1, i_2\)</span>に適用された命令<span>\({\mathsf{i32}}.{\mathsf{add}}\)</span>の結果は<span>\({\mathsf{add}}_{{\mathsf{i32}}}(i_1, i_2)\)</span>を呼び出します。
これは上記の定義を介してジェネリックな<span>\({\mathrm{iadd}}_{32}(i_1, i_2)\)</span>にマップされます。
同様に、zに適用された<span>\({\mathsf{i64}}.{\mathsf{trunc}}\mathsf{\_}{\mathsf{f32}}\mathsf{\_s}\)</span>は、<span>\({\mathsf{trunc}}^{\mathsf{s}}_{{\mathsf{f32}},{\mathsf{i64}}}(z)\)</span>を呼び出します。
これはジェネリックな<span>\({\mathrm{trunc}^{\mathsf{s}}}_{32,64}(z)\)</span>にマップされます。
</div>

---

<h3><span>\(t\mathsf{.}{\mathsf{const}}~c\)</span></h3>
<ol>
  <li><p><span>\(t.{\mathsf{const}}~c\)</span>をスタックにpushします。</p></li>
</ol>

### 付記

`const`命令は値と一致するので、この命令には正式な還元規則は必要ありません。

---

<h3><span>\(t\mathsf{.}{\mathit{unop}}\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型tの値がスタックの一番上に存在するはずです。</p></li>
  <li><p>スタックから<span>\(t.{\mathsf{const}}~c_1\)</span>をpopします。</p></li>
  <li>もし<span>\({\mathit{unop}}_t(c_1)\)</span>が定義されているならば:
    <ol>
      <li><p>cが<span>\({\mathit{unop}}_t(c_1)\)</span>の計算結果として算出可能であるとします。</p></li>
      <li><p><span>\(t.{\mathsf{const}}~c\)</span>をスタックにpushします。</p></li>
    </ol>
  </li>
  <li>そうでないならば:
    <ol>
      <li><p>トラップします。</p></li>
    </ol>
  </li>
</ol>
<div>\[\begin{split}\begin{array}{lcl&#64;{\qquad}l}
(t\mathsf{.}{\mathsf{const}}~c_1)~t\mathsf{.}{\mathit{unop}} &amp;{\hookrightarrow}&amp; (t\mathsf{.}{\mathsf{const}}~c)
  &amp; (\mathrel{\mbox{if}} c \in {\mathit{unop}}_t(c_1)) \\
(t\mathsf{.}{\mathsf{const}}~c_1)~t\mathsf{.}{\mathit{unop}} &amp;{\hookrightarrow}&amp; {\mathsf{trap}}
  &amp; (\mathrel{\mbox{if}} {\mathit{unop}}_{t}(c_1) = \{\})
\end{array}\end{split}\]</div>

<h3><span>\(t\mathsf{.}{\mathit{binop}}\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型tの値がスタックの上に2つ連続して存在するはずです。</p></li>
  <li><p>スタックから<span>\(t.{\mathsf{const}}~c_2\)</span>をpopします。</p></li>
  <li><p>スタックから<span>\(t.{\mathsf{const}}~c_1\)</span>をpopします。</p></li>
  <li>もし<span>\({\mathit{binop}}_t(c_1, c_2)\)</span>が定義されているならば:
    <ol>
      <li><p>cが<span>\({\mathit{binop}}_t(c_1, c_2)\)</span>の計算結果として算出可能であるとします。</p></li>
      <li><p><span>\(t.{\mathsf{const}}~c\)</span>をスタックにpushします。</p></li>
    </ol>
  </li>
  <li>そうでないならば:
    <ol>
      <li><p>トラップします。</p></li>
    </ol>
  </li>
</ol>
<div>\[\begin{split}\begin{array}{lcl&#64;{\qquad}l}
(t\mathsf{.}{\mathsf{const}}~c_1)~(t\mathsf{.}{\mathsf{const}}~c_2)~t\mathsf{.}{\mathit{binop}} &amp;{\hookrightarrow}&amp; (t\mathsf{.}{\mathsf{const}}~c)
  &amp; (\mathrel{\mbox{if}} c \in {\mathit{binop}}_t(c_1,c_2)) \\
(t\mathsf{.}{\mathsf{const}}~c_1)~(t\mathsf{.}{\mathsf{const}}~c_2)~t\mathsf{.}{\mathit{binop}} &amp;{\hookrightarrow}&amp; {\mathsf{trap}}
  &amp; (\mathrel{\mbox{if}} {\mathit{binop}}_{t}(c_1,c2) = \{\})
\end{array}\end{split}\]</div>

<h3><span>\(t\mathsf{.}{\mathit{testop}}\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型tの値がスタックの一番上に存在するはずです。</p></li>
  <li><p>スタックから<span>\(t.{\mathsf{const}}~c_1\)</span>をpopします。</p></li>
  <li><p>cが<span>\({\mathit{testop}}_t(c_1)\)</span>の計算結果であるとします。</p></li>
  <li><p><span>\({\mathsf{i32}}.{\mathsf{const}}~c\)</span>をスタックにpushします。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{lcl&#64;{\qquad}l}
(t\mathsf{.}{\mathsf{const}}~c_1)~t\mathsf{.}{\mathit{testop}} &amp;{\hookrightarrow}&amp; ({\mathsf{i32}}\mathsf{.}{\mathsf{const}}~c)
  &amp; (\mathrel{\mbox{if}} c = {\mathit{testop}}_t(c_1)) \\
\end{array}\end{split}\]</div>

<h3><span>\(t\mathsf{.}{\mathit{relop}}\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型tの値がスタックの上に2つ連続して存在するはずです。</p></li>
  <li><p>スタックから<span>\(t.{\mathsf{const}}~c_2\)</span>をpopします。</p></li>
  <li><p>スタックから<span>\(t.{\mathsf{const}}~c_1\)</span>をpopします。</p></li>
  <li><p>cが<span>\({\mathit{relop}}_t(c_1, c_2)\)</span>の計算結果であるとします。</p></li>
  <li><p><span>\({\mathsf{i32}}.{\mathsf{const}}~c\)</span>をスタックにpushします。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{lcl&#64;{\qquad}l}
(t\mathsf{.}{\mathsf{const}}~c_1)~(t\mathsf{.}{\mathsf{const}}~c_2)~t\mathsf{.}{\mathit{relop}} &amp;{\hookrightarrow}&amp; ({\mathsf{i32}}\mathsf{.}{\mathsf{const}}~c)
  &amp; (\mathrel{\mbox{if}} c = {\mathit{relop}}_t(c_1,c_2)) \\
\end{array}\end{split}\]</div>

<h3><span>\(t_2\mathsf{.}{\mathit{cvtop}}\mathsf{\_}t_1\mathsf{\_}{\mathit{sx}}^?\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型<span>\(t_1\)</span>の値がスタックの上に2つ連続して存在するはずです。</p></li>
  <li><p>スタックから<span>\(t_1.{\mathsf{const}}~c_1\)</span>をpopします。</p></li>
  <li>もし<span>\({\mathit{cvtop}}^{{\mathit{sx}}^?}_{t_1,t_2}(c_1)\)</span>が定義されているならば:
    <ol>
      <li><p><span>\(c_2\)</span>が<span>\({\mathit{cvtop}}^{{\mathit{sx}}^?}_{t_1,t_2}(c_1)\)</span>の計算結果であるとします。</p></li>
      <li><p><span>\(t_2.{\mathsf{const}}~c_2\)</span>をスタックにpushします。</p></li>
    </ol>
  </li>
  <li>そうでないならば:
    <ol>
      <li><p>トラップします。</p></li>
    </ol>
  </li>
</ol>
<div>\[\begin{split}\begin{array}{lcl&#64;{\qquad}l}
(t_1\mathsf{.}{\mathsf{const}}~c_1)~t_2\mathsf{.}{\mathit{cvtop}}\mathsf{\_}t_1\mathsf{\_}{\mathit{sx}}^? &amp;{\hookrightarrow}&amp; (t_2\mathsf{.}{\mathsf{const}}~c_2)
  &amp; (\mathrel{\mbox{if}} c_2 \in {\mathit{cvtop}}^{{\mathit{sx}}^?}_{t_1,t_2}(c_1)) \\
(t_1\mathsf{.}{\mathsf{const}}~c_1)~t_2\mathsf{.}{\mathit{cvtop}}\mathsf{\_}t_1\mathsf{\_}{\mathit{sx}}^? &amp;{\hookrightarrow}&amp; {\mathsf{trap}}
  &amp; (\mathrel{\mbox{if}} {\mathit{cvtop}}^{{\mathit{sx}}^?}_{t_1,t_2}(c_1) = \{\})
\end{array}\end{split}\]</div>

## パラメトリック命令

<h3><span>\({\mathsf{drop}}\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックは空ではありません。</p></li>
  <li><p>スタックから<span>\({\mathit{val}}\)</span>をpopします。</p></li>
</ol>
<div>\[\begin{array}{lcl&#64;{\qquad}l}
{\mathit{val}}~~{\mathsf{drop}} &amp;{\hookrightarrow}&amp; \epsilon
\end{array}\]</div>

<h3><span>\({\mathsf{select}}\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型<span>\({\mathsf{i32}}\)</span>の値がスタックの上に1つ存在するはずです。</p></li>
  <li><p>スタックから<span>\({\mathsf{i32}}.{\mathsf{const}}~c\)</span>をpopします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、同じ型の値がスタックの上に2つ以上存在するはずです。</p></li>
  <li><p>スタックから<span>\({\mathit{val}}_2\)</span>をpopします。</p></li>
  <li><p>スタックから<span>\({\mathit{val}}_1\)</span>をpopします。</p></li>
  <li>cが0でないならば:
    <ol>
      <li><p><span>\({\mathit{val}}_1\)</span>をスタックにpushしなおします。</p></li>
    </ol>
  </li>
  <li>そうでないならば、:
    <ol>
      <li><p><span>\({\mathit{val}}_2\)</span>をスタックにpushしなおします。</p></li>
    </ol>
  </li>
</ol>
<div>\[\begin{split}\begin{array}{lcl&#64;{\qquad}l}
{\mathit{val}}_1~{\mathit{val}}_2~({\mathsf{i32}}\mathsf{.}{\mathsf{const}}~c)~{\mathsf{select}} &amp;{\hookrightarrow}&amp; {\mathit{val}}_1
  &amp; (\mathrel{\mbox{if}} c \neq 0) \\
{\mathit{val}}_1~{\mathit{val}}_2~({\mathsf{i32}}\mathsf{.}{\mathsf{const}}~c)~{\mathsf{select}} &amp;{\hookrightarrow}&amp; {\mathit{val}}_2
  &amp; (\mathrel{\mbox{if}} c = 0) \\
\end{array}\end{split}\]</div>

## 変数命令

<h3><span>\({\mathsf{local.get}}~x\)</span></h3>
<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(F.{\mathsf{locals}}[x]\)</span>は存在します。</p></li>
  <li><p><span>\({\mathit{val}}\)</span>が値<span>\(F.{\mathsf{locals}}[x]\)</span>であるとします。</p></li>
  <li><p><span>\({\mathit{val}}\)</span>をスタックにpushします。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{lcl&#64;{\qquad}l}
F; ({\mathsf{local.get}}~x) &amp;{\hookrightarrow}&amp; F; {\mathit{val}}
  &amp; (\mathrel{\mbox{if}} F.{\mathsf{locals}}[x] = {\mathit{val}}) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{local.set}}~x\)</span></h3>
<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(F.{\mathsf{locals}}[x]\)</span>は存在します。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックは空ではありません。</p></li>
  <li><p><span>\({\mathit{val}}\)</span>をスタックからpopします。</p></li>
  <li><p><span>\(F.{\mathsf{locals}}[x]\)</span>を<span>\({\mathit{val}}\)</span>で置換します。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{lcl&#64;{\qquad}l}
F; {\mathit{val}}~({\mathsf{local.set}}~x) &amp;{\hookrightarrow}&amp; F'; \epsilon
  &amp; (\mathrel{\mbox{if}} F' = F {\mathrel{\mbox{with}}} {\mathsf{locals}}[x] = {\mathit{val}}) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{local.tee}}~x\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックは空ではありません。</p></li>
  <li><p><span>\({\mathit{val}}\)</span>をスタックからpopします。</p></li>
  <li><p><span>\({\mathit{val}}\)</span>をスタックにpushします。</p></li>
  <li><p><span>\({\mathit{val}}\)</span>をスタックにpushします。</p></li>
  <li><p>命令<span>\(({\mathsf{local.set}}~x)\)</span>を実行します。</p></li>
</ol>
<div>\[\begin{array}{lcl&#64;{\qquad}l}
{\mathit{val}}~({\mathsf{local.tee}}~x) &amp;{\hookrightarrow}&amp; {\mathit{val}}~{\mathit{val}}~({\mathsf{local.set}}~x)
\end{array}\]</div>

<h3><span>\({\mathsf{global.get}}~x\)</span></h3>
<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(F.{\mathsf{module}}.{\mathsf{globaladdrs}}[x]\)</span>は存在します。</p></li>
  <li><p>aがグローバルアドレス<span>\(F.{\mathsf{module}}.{\mathsf{globaladdrs}}[x]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(S.{\mathsf{globals}}[a]\)</span>は存在します。</p></li>
  <li><p><span>\(\mathit{glob}\)</span>がグローバルインスタンス<span>\(S.{\mathsf{globals}}[a]\)</span>であるとします。</p></li>
  <li><p><span>\({\mathit{val}}\)</span>が値<span>\(\mathit{glob}.{\mathsf{value}}\)</span>であるとします。</p></li>
  <li><p><span>\({\mathit{val}}\)</span>をスタックにpushします。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{l}
\begin{array}{lcl&#64;{\qquad}l}
S; F; ({\mathsf{global.get}}~x) &amp;{\hookrightarrow}&amp; S; F; {\mathit{val}}
\end{array}
\\ \qquad
  (\mathrel{\mbox{if}} S.{\mathsf{globals}}[F.{\mathsf{module}}.{\mathsf{globaladdrs}}[x]].{\mathsf{value}} = {\mathit{val}}) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{global.set}}~x\)</span></h3>
<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(F.{\mathsf{module}}.{\mathsf{globaladdrs}}[x]\)</span>は存在します。</p></li>
  <li><p>aがグローバルアドレス<span>\(F.{\mathsf{module}}.{\mathsf{globaladdrs}}[x]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(S.{\mathsf{globals}}[a]\)</span>は存在します。</p></li>
  <li><p><span>\(\mathit{glob}\)</span>がグローバルインスタンス<span>\(S.{\mathsf{globals}}[a]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックは空ではありません。</p></li>
  <li><p><span>\({\mathit{val}}\)</span>をスタックからpopします。</p></li>
  <li><p><span>\(\mathit{glob}.{\mathsf{value}}\)</span>を<span>\({\mathit{val}}\)</span>で置換します。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{l}
\begin{array}{lcl&#64;{\qquad}l}
S; F; {\mathit{val}}~({\mathsf{global.set}}~x) &amp;{\hookrightarrow}&amp; S'; F; \epsilon
\end{array}
\\ \qquad
(\mathrel{\mbox{if}} S' = S {\mathrel{\mbox{with}}} {\mathsf{globals}}[F.{\mathsf{module}}.{\mathsf{globaladdrs}}[x]].{\mathsf{value}} = {\mathit{val}}) \\
\end{array}\end{split}\]</div>

### 付記

バリデーション/検証によりグローバルがmutableであると保証されます。

## メモリ命令

### 付記

<div>ロード命令とストア命令でのアラインメント<span>\({\mathit{memarg}}.{\mathsf{align}}\)</span>はセマンティクスに影響を与えません。

これは、メモリがアクセスされるオフセット<span>\(\mathit{ea}\)</span>がプロパティ<span>\(\mathit{ea} \mathbin{\mathrm{mod}} 2^{{\mathit{memarg}}.{\mathsf{align}}} = 0\)</span>を満たすように意図されていることを示しています。

`WebAssembly`の実装は、このヒントを使用して意図された使用のために最適化することができます。

このプロパティに違反するアラインメントされていないアクセスは、アノテーションに関係なく許可されており、成功しなければなりません。
しかし、ハードウェアによってはかなり遅くなるかもしれません。
</div>

<h3><span>\(t\mathsf{.}{\mathsf{load}}~{\mathit{memarg}}\)</span> and <span>\(t\mathsf{.}{\mathsf{load}}{N}\mathsf{\_}{\mathit{sx}}~{\mathit{memarg}}\)</span></h3>
<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]\)</span>は存在します。</p></li>
  <li><p>aがメモリアドレス<span>\(F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(S.{\mathsf{mems}}[a]\)</span>は存在します。</p></li>
  <li><p><span>\(\mathit{mem}\)</span>がメモリインスタンス<span>\(S.{\mathsf{mems}}[a]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型<span>\({\mathsf{i32}}\)</span>の値はスタックの一番上に存在します。</p></li>
  <li><p>スタックから値<span>\({\mathsf{i32}}.{\mathsf{const}}~i\)</span>をpopします</p></li>
  <li><p><span>\(\mathit{ea}\)</span>がthe integer <span>\(i + {\mathit{memarg}}.{\mathsf{offset}}\)</span>であるとします。</p></li>
  <li>もしNが命令の部分でないならば:
    <ol>
      <li><p>Nが値型tのビット幅<span>\(|t|\)</span>であるとします。</p></li>
    </ol>
  </li>
  <li>もし<span>\(\mathit{ea} + N/8\)</span>が<span>\(\mathit{mem}.{\mathsf{data}}\)</span>の長さより大きいならば:
    <ol>
      <li><p>トラップします。</p></li>
    </ol>
  </li>
  <li><p><span>\(b^\ast\)</span>がbyteシーケンス<span>\(\mathit{mem}.{\mathsf{data}}[\mathit{ea} {\mathrel{\mathbf{:}}} N/8]\)</span>であるとします。</p></li>
  <li>もしNと<span>\({\mathit{sx}}\)</span>が命令の部分であるならば:
    <ol>
      <li><p>nが数値であり、条件<span>\({\mathrm{bytes}}_{{\mathit{iN}}}(n) = b^\ast\)</span>を満足するものであるとします。</p></li>
      <li><p>cが<span>\({\mathrm{extend}}\mathrm{\_}{\mathit{sx}}_{N,|t|}(n)\)</span>の計算結果であるとします。</p></li>
    </ol>
  </li>
  <li>そうでないならば:
    <ol>
      <li><p>cが定数であり、条件<span>\({\mathrm{bytes}}_t(c) = b^\ast\)</span>を満足するものであるとします。</p></li>
    </ol>
  </li>
  <li><p>スタックに値<span>\(t.{\mathsf{const}}~c\)</span>をpushします。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{l}
\begin{array}{lcl&#64;{\qquad}l}
S; F; ({\mathsf{i32}}.{\mathsf{const}}~i)~(t.{\mathsf{load}}~{\mathit{memarg}}) &amp;{\hookrightarrow}&amp; S; F; (t.{\mathsf{const}}~c)
\end{array}
\\ \qquad
  \begin{array}[t]{&#64;{}r&#64;{~}l&#64;{}}
  (\mathrel{\mbox{if}} &amp; \mathit{ea} = i + {\mathit{memarg}}.{\mathsf{offset}} \\
  \wedge &amp; \mathit{ea} + |t|/8 \leq |S.{\mathsf{mems}}[F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]].{\mathsf{data}}| \\
  \wedge &amp; {\mathrm{bytes}}_t(c) = S.{\mathsf{mems}}[F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]].{\mathsf{data}}[\mathit{ea} {\mathrel{\mathbf{:}}} |t|/8])
  \end{array}
\\[1ex]
\begin{array}{lcl&#64;{\qquad}l}
S; F; ({\mathsf{i32}}.{\mathsf{const}}~i)~(t.{\mathsf{load}}{N}\mathsf{\_}{\mathit{sx}}~{\mathit{memarg}}) &amp;{\hookrightarrow}&amp;
  S; F; (t.{\mathsf{const}}~{\mathrm{extend}}\mathrm{\_}{\mathit{sx}}_{N,|t|}(n))
\end{array}
\\ \qquad
  \begin{array}[t]{&#64;{}r&#64;{~}l&#64;{}}
  (\mathrel{\mbox{if}} &amp; \mathit{ea} = i + {\mathit{memarg}}.{\mathsf{offset}} \\
  \wedge &amp; \mathit{ea} + N/8 \leq |S.{\mathsf{mems}}[F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]].{\mathsf{data}}| \\
  \wedge &amp; {\mathrm{bytes}}_{{\mathit{iN}}}(n) = S.{\mathsf{mems}}[F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]].{\mathsf{data}}[\mathit{ea} {\mathrel{\mathbf{:}}} N/8])
  \end{array}
\\[1ex]
\begin{array}{lcl&#64;{\qquad}l}
S; F; ({\mathsf{i32}}.{\mathsf{const}}~k)~(t.{\mathsf{load}}({N}\mathsf{\_}{\mathit{sx}})^?~{\mathit{memarg}}) &amp;{\hookrightarrow}&amp; S; F; {\mathsf{trap}}
\end{array}
\\ \qquad
  (\mathrel{\mbox{otherwise}}) \\
\end{array}\end{split}\]</div>

<h3><span>\(t\mathsf{.}{\mathsf{store}}~{\mathit{memarg}}\)</span> and <span>\(t\mathsf{.}{\mathsf{store}}{N}~{\mathit{memarg}}\)</span></h3>
<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]\)</span>は存在します。</p></li>
  <li><p>aがメモリアドレス<span>\(F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(S.{\mathsf{mems}}[a]\)</span>は存在します。</p></li>
  <li><p><span>\(\mathit{mem}\)</span>がメモリインスタンス<span>\(S.{\mathsf{mems}}[a]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型tの値はスタックの一番上に存在します。</p></li>
  <li><p>スタックから値<span>\(t.{\mathsf{const}}~c\)</span>をpopします</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型<span>\({\mathsf{i32}}\)</span>の値はスタックの一番上に存在します。</p></li>
  <li><p>スタックから値<span>\({\mathsf{i32}}.{\mathsf{const}}~i\)</span>をpopします</p></li>
  <li><p><span>\(\mathit{ea}\)</span>がthe integer <span>\(i + {\mathit{memarg}}.{\mathsf{offset}}\)</span>であるとします。</p></li>
  <li>もしNが命令の部分でないならば:
    <ol>
      <li><p>Nが値型tのビット幅<span>\(|t|\)</span>であるとします。</p></li>
    </ol>
  </li>
  <li>もし<span>\(\mathit{ea} + N/8\)</span>が<span>\(\mathit{mem}.{\mathsf{data}}\)</span>の長さより大きいならば:
    <ol>
      <li><p>トラップします。</p></li>
    </ol>
  </li>
  <li>もしNが命令の部分であるならば:
    <ol>
      <li><p>nが<span>\({\mathrm{wrap}}_{|t|,N}(c)\)</span>の計算結果であるとします。</p></li>
      <li><p><span>\(b^\ast\)</span>がbyteシーケンス<span>\({\mathrm{bytes}}_{{\mathit{iN}}}(n)\)</span>であるとします。</p></li>
    </ol>
  </li>
  <li>そうでないならば:
    <ol>
      <li><p><span>\(b^\ast\)</span>がbyteシーケンス<span>\({\mathrm{bytes}}_t(c)\)</span>であるとします。</p></li>
    </ol>
  </li>
  <li><p>byte列<span>\(\mathit{mem}.{\mathsf{data}}[\mathit{ea} {\mathrel{\mathbf{:}}} N/8]\)</span>を<span>\(b^\ast\)</span>で置換します。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{l}
\begin{array}{lcl&#64;{\qquad}l}
S; F; ({\mathsf{i32}}.{\mathsf{const}}~i)~(t.{\mathsf{const}}~c)~(t.{\mathsf{store}}~{\mathit{memarg}}) &amp;{\hookrightarrow}&amp; S'; F; \epsilon
\end{array}
\\ \qquad
  \begin{array}[t]{&#64;{}r&#64;{~}l&#64;{}}
  (\mathrel{\mbox{if}} &amp; \mathit{ea} = i + {\mathit{memarg}}.{\mathsf{offset}} \\
  \wedge &amp; \mathit{ea} + |t|/8 \leq |S.{\mathsf{mems}}[F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]].{\mathsf{data}}| \\
  \wedge &amp; S' = S {\mathrel{\mbox{with}}} {\mathsf{mems}}[F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]].{\mathsf{data}}[\mathit{ea} {\mathrel{\mathbf{:}}} |t|/8] = {\mathrm{bytes}}_t(c)
  \end{array}
\\[1ex]
\begin{array}{lcl&#64;{\qquad}l}
S; F; ({\mathsf{i32}}.{\mathsf{const}}~i)~(t.{\mathsf{const}}~c)~(t.{\mathsf{store}}{N}~{\mathit{memarg}}) &amp;{\hookrightarrow}&amp; S'; F; \epsilon
\end{array}
\\ \qquad
  \begin{array}[t]{&#64;{}r&#64;{~}l&#64;{}}
  (\mathrel{\mbox{if}} &amp; \mathit{ea} = i + {\mathit{memarg}}.{\mathsf{offset}} \\
  \wedge &amp; \mathit{ea} + N/8 \leq |S.{\mathsf{mems}}[F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]].{\mathsf{data}}| \\
  \wedge &amp; S' = S {\mathrel{\mbox{with}}} {\mathsf{mems}}[F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]].{\mathsf{data}}[\mathit{ea} {\mathrel{\mathbf{:}}} N/8] = {\mathrm{bytes}}_{{\mathit{iN}}}({\mathrm{wrap}}_{|t|,N}(c))
  \end{array}
\\[1ex]
\begin{array}{lcl&#64;{\qquad}l}
S; F; ({\mathsf{i32}}.{\mathsf{const}}~k)~(t.{\mathsf{const}}~c)~(t.{\mathsf{store}}{N}^?~{\mathit{memarg}}) &amp;{\hookrightarrow}&amp; S; F; {\mathsf{trap}}
\end{array}
\\ \qquad
  (\mathrel{\mbox{otherwise}}) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{memory.size}}\)</span></h3>
<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]\)</span>は存在します。</p></li>
  <li><p>aがメモリアドレス<span>\(F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(S.{\mathsf{mems}}[a]\)</span>は存在します。</p></li>
  <li><p><span>\(\mathit{mem}\)</span>がメモリインスタンス<span>\(S.{\mathsf{mems}}[a]\)</span>であるとします。</p></li>
  <li><p><span>\(\mathit{sz}\)</span>が<span>\(\mathit{mem}.{\mathsf{data}}\)</span>の長さをページサイズで除算したものであるとします。</p></li>
  <li><p>スタックに値<span>\({\mathsf{i32}}.{\mathsf{const}}~\mathit{sz}\)</span>をpushします。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{l}
\begin{array}{lcl&#64;{\qquad}l}
S; F; {\mathsf{memory.size}} &amp;{\hookrightarrow}&amp; S; F; ({\mathsf{i32}}.{\mathsf{const}}~\mathit{sz})
\end{array}
\\ \qquad
  (\mathrel{\mbox{if}} |S.{\mathsf{mems}}[F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]].{\mathsf{data}}| = \mathit{sz}\cdot64\,\mathrm{Ki}) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{memory.grow}}\)</span></h3>
<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]\)</span>は存在します。</p></li>
  <li><p>aがメモリアドレス<span>\(F.{\mathsf{module}}.{\mathsf{memaddrs}}[0]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(S.{\mathsf{mems}}[a]\)</span>は存在します。</p></li>
  <li><p><span>\(\mathit{mem}\)</span>がメモリインスタンス<span>\(S.{\mathsf{mems}}[a]\)</span>であるとします。</p></li>
  <li><p><span>\(\mathit{sz}\)</span>が<span>\(S.{\mathsf{mems}}[a]\)</span>の長さをページサイズで除算したものであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型<span>\({\mathsf{i32}}\)</span>の値はスタックの一番上に存在します。</p></li>
  <li><p>スタックから値<span>\({\mathsf{i32}}.{\mathsf{const}}~n\)</span>をpopします</p></li>
  <li><p><span>\(\mathit{err}\)</span>が<span>\({\mathit{i32}}\)</span>型の<span>\(2^{32}-1\)</span>であり、<span>\({\mathrm{signed}}_{32}(\mathit{err})\)</span>が<span>\(-1\)</span>であるとします。</p></li>
  <li><p>あるいは<span>\(\mathit{mem}\)</span>をnページに伸長するよう試みます:</p></li>
</ol>
<blockquote>
  <div>
    <ol>
      <li><p>もし成功したならば、スタックに値<span>\({\mathsf{i32}}.{\mathsf{const}}~\mathit{sz}\)</span>をpushします。</p></li>
      <li><p>そうでないならば、スタックに値<span>\({\mathsf{i32}}.{\mathsf{const}}~\mathit{err}\)</span>をpushします。</p></li>
    </ol>
  </div>
</blockquote>
<ol>
  <li><p>あるいは、スタックに値<span>\({\mathsf{i32}}.{\mathsf{const}}~\mathit{err}\)</span>をpushします。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{l}
\begin{array}{lcl&#64;{\qquad}l}
S; F; ({\mathsf{i32}}.{\mathsf{const}}~n)~{\mathsf{memory.grow}} &amp;{\hookrightarrow}&amp; S'; F; ({\mathsf{i32}}.{\mathsf{const}}~\mathit{sz})
\end{array}
\\ \qquad
  \begin{array}[t]{&#64;{}r&#64;{~}l&#64;{}}
  (\mathrel{\mbox{if}} &amp; F.{\mathsf{module}}.{\mathsf{memaddrs}}[0] = a \\
  \wedge &amp; \mathit{sz} = |S.{\mathsf{mems}}[a].{\mathsf{data}}|/64\,\mathrm{Ki} \\
  \wedge &amp; S' = S {\mathrel{\mbox{with}}} {\mathsf{mems}}[a] = {\mathrm{growmem}}(S.{\mathsf{mems}}[a], n)) \\
  \end{array}
\\[1ex]
\begin{array}{lcl&#64;{\qquad}l}
S; F; ({\mathsf{i32}}.{\mathsf{const}}~n)~{\mathsf{memory.grow}} &amp;{\hookrightarrow}&amp; S; F; ({\mathsf{i32}}.{\mathsf{const}}~{\mathrm{signed}}_{32}^{-1}(-1))
\end{array}
\end{array}\end{split}\]</div>

### 付記

`memory.grow`命令は非決定論的です。
成功して古いメモリサイズszを返すか、失敗して-1を返すかのどちらかです。

失敗は、参照されているメモリインスタンスの最大サイズが定義されていて、それを超えてしまう場合に発生しなければなりません。
しかし、失敗は他の場合にも起こり得ます。
実際には、どちらを選択するかはエンベッダーが利用できるリソースに依存します。

## 制御命令

<h3><span>\({\mathsf{nop}}\)</span></h3>
<ol>
  <li><p>何もしません。</p></li>
</ol>
<div>\[\begin{array}{lcl&#64;{\qquad}l}
{\mathsf{nop}} &amp;{\hookrightarrow}&amp; \epsilon
\end{array}\]</div>

<h3><span>\({\mathsf{unreachable}}\)</span></h3>
<ol>
  <li><p>トラップします。</p></li>
</ol>
<div>\[\begin{array}{lcl&#64;{\qquad}l}
{\mathsf{unreachable}} &amp;{\hookrightarrow}&amp; {\mathsf{trap}}
\end{array}\]</div>

<h3><span>\({\mathsf{block}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}}\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\({\mathrm{expand}}_F({\mathit{blocktype}})\)</span>は定義されています。</p></li>
  <li><p><span>\([t_1^m] {\rightarrow} [t_2^n]\)</span>が関数型<span>\({\mathrm{expand}}_F({\mathit{blocktype}})\)</span>であるとします。</p></li>
  <li><p>Lがアリティnかつ継続先がブロック終端であるラベルであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックには少なくともm個値が存在します。</p></li>
  <li><p>スタックから<span>\({\mathit{val}}^m\)</span>をpopします。</p></li>
  <li><p>ブロック<span>\({\mathit{val}}^m~{\mathit{instr}}^\ast\)</span>にラベルLとして突入します。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{lcl&#64;{\qquad}l}
F; {\mathit{val}}^m~{\mathsf{block}}~\mathit{bt}~{\mathit{instr}}^\ast~{\mathsf{end}} &amp;{\hookrightarrow}&amp;
  F; {\mathsf{label}}_n\{\epsilon\}~{\mathit{val}}^m~{\mathit{instr}}^\ast~{\mathsf{end}}
  &amp; (\mathrel{\mbox{if}} {\mathrm{expand}}_F(\mathit{bt}) = [t_1^m] {\rightarrow} [t_2^n])
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{loop}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}}\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\({\mathrm{expand}}_F({\mathit{blocktype}})\)</span>は定義されています。</p></li>
  <li><p><span>\([t_1^m] {\rightarrow} [t_2^n]\)</span>が関数型<span>\({\mathrm{expand}}_F({\mathit{blocktype}})\)</span>であるとします。</p></li>
  <li><p>Lがアリティmかつ継続先がループ先頭であるラベルであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックには少なくともm個値が存在します。</p></li>
  <li><p>スタックから<span>\({\mathit{val}}^m\)</span>をpopします。</p></li>
  <li><p>ブロック<span>\({\mathit{val}}^m~{\mathit{instr}}^\ast\)</span>にラベルLとして突入します。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{lcl&#64;{\qquad}l}
F; {\mathit{val}}^m~{\mathsf{loop}}~\mathit{bt}~{\mathit{instr}}^\ast~{\mathsf{end}} &amp;{\hookrightarrow}&amp;
  F; {\mathsf{label}}_m\{{\mathsf{loop}}~\mathit{bt}~{\mathit{instr}}^\ast~{\mathsf{end}}\}~{\mathit{val}}^m~{\mathit{instr}}^\ast~{\mathsf{end}}
  &amp; (\mathrel{\mbox{if}} {\mathrm{expand}}_F(\mathit{bt}) = [t_1^m] {\rightarrow} [t_2^n])
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{if}}~{\mathit{blocktype}}~{\mathit{instr}}_1^\ast~{\mathsf{else}}~{\mathit{instr}}_2^\ast~{\mathsf{end}}\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\({\mathrm{expand}}_F({\mathit{blocktype}})\)</span>は定義されています。</p></li>
  <li><p><span>\([t_1^m] {\rightarrow} [t_2^n]\)</span>が関数型<span>\({\mathrm{expand}}_F({\mathit{blocktype}})\)</span>であるとします。</p></li>
  <li><p>Lがアリティnかつ継続先が<span>\({\mathsf{if}}\)</span>命令終端であるラベルであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型<span>\({\mathsf{i32}}\)</span>の値はスタックの一番上に存在します。</p></li>
  <li><p>スタックから値<span>\({\mathsf{i32}}.{\mathsf{const}}~c\)</span>をpopします</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックには少なくともm個値が存在します。</p></li>
  <li><p>スタックから<span>\({\mathit{val}}^m\)</span>をpopします。</p></li>
  <li>cが0でないならば、
    <ol>
      <li><p>ブロック<span>\({\mathit{val}}^m~{\mathit{instr}}_1^\ast\)</span>にラベルLとして突入します。</p></li>
    </ol>
  </li>
  <li>そうでないならば:
    <ol>
      <li><p>ブロック<span>\({\mathit{val}}^m~{\mathit{instr}}_2^\ast\)</span>にラベルLとして突入します。</p></li>
    </ol>
  </li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{lcl&#64;{\qquad}l}
F; {\mathit{val}}^m~({\mathsf{i32}}.{\mathsf{const}}~c)~{\mathsf{if}}~\mathit{bt}~{\mathit{instr}}_1^\ast~{\mathsf{else}}~{\mathit{instr}}_2^\ast~{\mathsf{end}} &amp;{\hookrightarrow}&amp;
  F; {\mathsf{label}}_n\{\epsilon\}~{\mathit{val}}^m~{\mathit{instr}}_1^\ast~{\mathsf{end}}
  &amp; (\mathrel{\mbox{if}} c \neq 0 \wedge {\mathrm{expand}}_F(\mathit{bt}) = [t_1^m] {\rightarrow} [t_2^n]) \\
F; {\mathit{val}}^m~({\mathsf{i32}}.{\mathsf{const}}~c)~{\mathsf{if}}~\mathit{bt}~{\mathit{instr}}_1^\ast~{\mathsf{else}}~{\mathit{instr}}_2^\ast~{\mathsf{end}} &amp;{\hookrightarrow}&amp;
  F; {\mathsf{label}}_n\{\epsilon\}~{\mathit{val}}^m~{\mathit{instr}}_2^\ast~{\mathsf{end}}
  &amp; (\mathrel{\mbox{if}} c = 0 \wedge {\mathrm{expand}}_F(\mathit{bt}) = [t_1^m] {\rightarrow} [t_2^n]) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{br}}~l\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックには少なくとも<span>\(l+1\)</span>個ラベルが存在します。</p></li>
  <li><p>Lが0-index startでl番目のスタックに現れるラベルであるとします。</p></li>
  <li><p>nがLのアリティであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックには少なくともn個値が存在します。</p></li>
  <li><p>スタックから<span>\({\mathit{val}}^n\)</span>をpopします。</p></li>
  <li><span>\(l+1\)</span>回繰り返します:
    <ol>
      <li>スタックの一番上に存在するものが値である間:
        <ol>
          <li><p>スタックから値をpopします。</p></li>
        </ol>
      </li>
      <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックの一番上にはラベルが存在します。</p></li>
      <li><p>スタックからラベルをpopします。</p></li>
    </ol>
  </li>
  <li><p>スタックに値<span>\({\mathit{val}}^n\)</span>をpushします。</p></li>
  <li><p>Lの続きにジャンプします。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{lcl&#64;{\qquad}l}
{\mathsf{label}}_n\{{\mathit{instr}}^\ast\}~{B}^l[{\mathit{val}}^n~({\mathsf{br}}~l)]~{\mathsf{end}} &amp;{\hookrightarrow}&amp; {\mathit{val}}^n~{\mathit{instr}}^\ast
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{br\_if}}~l\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型<span>\({\mathsf{i32}}\)</span>の値はスタックの一番上に存在します。</p></li>
  <li><p>スタックから値<span>\({\mathsf{i32}}.{\mathsf{const}}~c\)</span>をpopします</p></li>
  <li>cが0でないならば、
    <ol>
      <li><p>命令<span>\(({\mathsf{br}}~l)\)</span>を実行します。</p></li>
    </ol>
  </li>
  <li>そうでないならば:
    <ol>
      <li><p>何もしません。</p></li>
    </ol>
  </li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{lcl&#64;{\qquad}l}
({\mathsf{i32}}.{\mathsf{const}}~c)~({\mathsf{br\_if}}~l) &amp;{\hookrightarrow}&amp; ({\mathsf{br}}~l)
  &amp; (\mathrel{\mbox{if}} c \neq 0) \\
({\mathsf{i32}}.{\mathsf{const}}~c)~({\mathsf{br\_if}}~l) &amp;{\hookrightarrow}&amp; \epsilon
  &amp; (\mathrel{\mbox{if}} c = 0) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{br\_table}}~l^\ast~l_N\)</span></h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型<span>\({\mathsf{i32}}\)</span>の値はスタックの一番上に存在します。</p></li>
  <li><p>スタックから値<span>\({\mathsf{i32}}.{\mathsf{const}}~i\)</span>をpopします</p></li>
  <li>もしiが<span>\(l^\ast\)</span>の長さ未満ならば:
    <ol>
      <li><p><span>\(l_i\)</span>がthe label <span>\(l^\ast[i]\)</span>であるとします。</p></li>
      <li><p>命令<span>\(({\mathsf{br}}~l_i)\)</span>を実行します。</p></li>
    </ol>
  </li>
  <li>そうでないならば:
    <ol>
      <li><p>命令<span>\(({\mathsf{br}}~l_N)\)</span>を実行します。</p></li>
    </ol>
  </li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{lcl&#64;{\qquad}l}
({\mathsf{i32}}.{\mathsf{const}}~i)~({\mathsf{br\_table}}~l^\ast~l_N) &amp;{\hookrightarrow}&amp; ({\mathsf{br}}~l_i)
  &amp; (\mathrel{\mbox{if}} l^\ast[i] = l_i) \\
({\mathsf{i32}}.{\mathsf{const}}~i)~({\mathsf{br\_table}}~l^\ast~l_N) &amp;{\hookrightarrow}&amp; ({\mathsf{br}}~l_N)
  &amp; (\mathrel{\mbox{if}} |l^\ast| \leq i) \\
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{return}}\)</span></h3>
<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>nがFのアリティであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックには少なくともn個値が存在します。</p></li>
  <li><p><span>\({\mathit{val}}^n\)</span>の戻り値をスタックからpopします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックは少なくとも1つフレームを含みます。</p></li>
  <li>スタックの一番上の要素がフレームでない間:
    <ol>
      <li><p>popします。</p></li>
    </ol>
  </li>
  <li><p>前提条件：スタックの一番上にフレームFが存在します。</p></li>
  <li><p>スタックからフレームをpopします。</p></li>
  <li><p>スタックに<span>\({\mathit{val}}^n\)</span>をpushします。</p></li>
  <li><p>フレームをpushした元々の呼び出しに続く命令にジャンプします。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{lcl&#64;{\qquad}l}
{\mathsf{frame}}_n\{F\}~{B}^k[{\mathit{val}}^n~{\mathsf{return}}]~{\mathsf{end}} &amp;{\hookrightarrow}&amp; {\mathit{val}}^n
\end{array}\end{split}\]</div>

<h3><span>\({\mathsf{call}}~x\)</span></h3>
<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(F.{\mathsf{module}}.{\mathsf{funcaddrs}}[x]\)</span>は存在します。</p></li>
  <li><p>aが関数アドレス<span>\(F.{\mathsf{module}}.{\mathsf{funcaddrs}}[x]\)</span>であるとします。</p></li>
  <li><p>アドレスaにある関数インスタンスを実行します。</p></li>
</ol>
<div>\[\begin{array}{lcl&#64;{\qquad}l}
F; ({\mathsf{call}}~x) &amp;{\hookrightarrow}&amp; F; ({\mathsf{invoke}}~a)
  &amp; (\mathrel{\mbox{if}} F.{\mathsf{module}}.{\mathsf{funcaddrs}}[x] = a)
\end{array}\]</div>

<h3><span>\({\mathsf{call\_indirect}}~x\)</span></h3>
<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(F.{\mathsf{module}}.{\mathsf{tableaddrs}}[0]\)</span>は存在します。</p></li>
  <li><p><span>\(\mathit{ta}\)</span>がテーブルアドレス<span>\(F.{\mathsf{module}}.{\mathsf{tableaddrs}}[0]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(S.{\mathsf{tables}}[\mathit{ta}]\)</span>は存在します。</p></li>
  <li><p><span>\(\mathit{tab}\)</span>がテーブルインスタンス<span>\(S.{\mathsf{tables}}[\mathit{ta}]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(F.{\mathsf{module}}.{\mathsf{types}}[x]\)</span>は存在します。</p></li>
  <li><p><span>\(\mathit{ft}_{\mathrm{expect}}\)</span>が関数型<span>\(F.{\mathsf{module}}.{\mathsf{types}}[x]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、値型<span>\({\mathsf{i32}}\)</span>の値はスタックの一番上に存在します。</p></li>
  <li><p>スタックから値<span>\({\mathsf{i32}}.{\mathsf{const}}~i\)</span>をpopします</p></li>
  <li>もしiが<span>\(\mathit{tab}.{\mathsf{elem}}\)</span>の長さ未満ならば:
    <ol>
      <li><p>トラップします。</p></li>
    </ol>
  </li>
  <li>もし<span>\(\mathit{tab}.{\mathsf{elem}}[i]\)</span>が初期化されていないならば:
    <ol>
      <li><p>トラップします。</p></li>
    </ol>
  </li>
  <li><p>aが関数アドレス<span>\(\mathit{tab}.{\mathsf{elem}}[i]\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(S.{\mathsf{funcs}}[a]\)</span>は存在します。</p></li>
  <li><p><span>\(\mathit{f}\)</span>が関数インスタンス<span>\(S.{\mathsf{funcs}}[a]\)</span>であるとします。</p></li>
  <li><p><span>\(\mathit{ft}_{\mathrm{actual}}\)</span>が関数型<span>\(\mathit{f}.{\mathsf{type}}\)</span>であるとします。</p></li>
  <li>もし<span>\(\mathit{ft}_{\mathrm{actual}}\)</span>と<span>\(\mathit{ft}_{\mathrm{expect}}\)</span>が互いに異なるならば:
    <ol>
      <li><p>トラップします。</p></li>
    </ol>
  </li>
  <li><p>アドレスaにある関数インスタンスを実行します。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{l}
\begin{array}{lcl&#64;{\qquad}l}
S; F; ({\mathsf{i32}}.{\mathsf{const}}~i)~({\mathsf{call\_indirect}}~x) &amp;{\hookrightarrow}&amp; S; F; ({\mathsf{invoke}}~a)
\end{array}
\\ \qquad
  \begin{array}[t]{&#64;{}r&#64;{~}l&#64;{}}
  (\mathrel{\mbox{if}} &amp; S.{\mathsf{tables}}[F.{\mathsf{module}}.{\mathsf{tableaddrs}}[0]].{\mathsf{elem}}[i] = a \\
  \wedge &amp; S.{\mathsf{funcs}}[a] = f \\
  \wedge &amp; F.{\mathsf{module}}.{\mathsf{types}}[x] = f.{\mathsf{type}})
  \end{array}
\\[1ex]
\begin{array}{lcl&#64;{\qquad}l}
S; F; ({\mathsf{i32}}.{\mathsf{const}}~i)~({\mathsf{call\_indirect}}~x) &amp;{\hookrightarrow}&amp; S; F; {\mathsf{trap}}
\end{array}
\\ \qquad
  (\mathrel{\mbox{otherwise}})
\end{array}\end{split}\]</div>

## ブロック

以下の補助規則は、ブロックを構成する命令列を実行する際のセマンティクスを定義しています。

<h3>ラベルLと共に<span>\({\mathit{instr}}^\ast\)</span>に突入する</h3>
<ol>
  <li><p>スタックにLをpushします。</p></li>
  <li><p>命令シーケンス<span>\({\mathit{instr}}^\ast\)</span>の最初にジャンプします。</p></li>
</ol>

### 付記

構造化制御命令が直接還元する管理命令にはラベルLが埋め込まれているため、命令列に入る際の形式的な還元ルールは必要ありません。

---

<h3>ラベルLと共に<span>\({\mathit{instr}}^\ast\)</span>から脱する</h3>

ブロック終端にジャンプやトラップせず到達した時、以下のステップを実行します。

<ol>
  <li><p>mがthe number of values on the top of the stackであるとします。</p></li>
  <li><p>スタックから<span>\({\mathit{val}}^m\)</span>をpopします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、ラベルLはスタックの一番上に存在します。</p></li>
  <li><p>スタックからラベルをpopします。</p></li>
  <li><p>スタックに<span>\({\mathit{val}}^m\)</span>をpushしなおします。</p></li>
  <li><p>ラベルLに紐付けられた構造化制御命令の<span>\({\mathsf{end}}\)</span>の後の位置にジャンプします。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{lcl&#64;{\qquad}l}
{\mathsf{label}}_n\{{\mathit{instr}}^\ast\}~{\mathit{val}}^m~{\mathsf{end}} &amp;{\hookrightarrow}&amp; {\mathit{val}}^m
\end{array}\end{split}\]</div>

### 付記

このセマンティクスはループ命令に含まれる命令列にも適用されます。
したがって、明示的に後方への分岐が実行されない限りループの実行は終了から外れます。

## 関数呼び出し

<h3>Invocation of function address a</h3>
<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(S.{\mathsf{funcs}}[a]\)</span>は存在します。</p></li>
  <li><p>fが関数インスタンス<span>\(S.{\mathsf{funcs}}[a]\)</span>であるとします。</p></li>
  <li><p><span>\([t_1^n] {\rightarrow} [t_2^m]\)</span>が関数型<span>\(f.{\mathsf{type}}\)</span>であるとします。</p></li>
  <li><p><span>\(t^\ast\)</span>が値型<span>\(f.{\mathsf{code}}.{\mathsf{locals}}\)</span>のリストであるとします。</p></li>
  <li><p><span>\({\mathit{instr}}^\ast~{\mathsf{end}}\)</span>が式<span>\(f.{\mathsf{code}}.{\mathsf{body}}\)</span>であるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタック上に少なくともn個の値が存在します。</p></li>
  <li><p>スタックから<span>\({\mathit{val}}^n\)</span>をpopします。</p></li>
  <li><p><span>\({\mathit{val}}_0^\ast\)</span>が型<span>\(t^\ast\)</span>の0に相当する値のリストであるとします。</p></li>
  <li><p>Fがフレーム<span>\(\{ {\mathsf{module}}~f.{\mathsf{module}}, {\mathsf{locals}}~{\mathit{val}}^n~{\mathit{val}}_0^\ast \}\)</span>であるとします。</p></li>
  <li><p>スタックにアリティmのフレームFをpushします。</p></li>
  <li><p>Lがアリティmかつ継続先が関数終端であるラベルであるとします。</p></li>
  <li><p>Enter 命令シーケンス<span>\({\mathit{instr}}^\ast\)</span> with label L。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{l}
\begin{array}{lcl&#64;{\qquad}l}
S; {\mathit{val}}^n~({\mathsf{invoke}}~a) &amp;{\hookrightarrow}&amp; S; {\mathsf{frame}}_m\{F\}~{\mathsf{label}}_m\{\}~{\mathit{instr}}^\ast~{\mathsf{end}}~{\mathsf{end}}
\end{array}
\\ \qquad
  \begin{array}[t]{&#64;{}r&#64;{~}l&#64;{}}
  (\mathrel{\mbox{if}} &amp; S.{\mathsf{funcs}}[a] = f \\
  \wedge &amp; f.{\mathsf{type}} = [t_1^n] {\rightarrow} [t_2^m] \\
  \wedge &amp; f.{\mathsf{code}} = \{ {\mathsf{type}}~x, {\mathsf{locals}}~t^k, {\mathsf{body}}~{\mathit{instr}}^\ast~{\mathsf{end}} \} \\
  \wedge &amp; F = \{ {\mathsf{module}}~f.{\mathsf{module}}, ~{\mathsf{locals}}~{\mathit{val}}^n~(t.{\mathsf{const}}~0)^k \})
  \end{array} \\
\end{array}\end{split}\]</div>

<h3>関数から戻る</h3>

関数終端にジャンプまたはトラップせずに到達した時、以下のステップを実行します。

<ol>
  <li><p>Fがカレントフレームであるとします。</p></li>
  <li><p>nがthe arity of the activation of Fであるとします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックにn個値が存在します。</p></li>
  <li><p>スタックからthe results <span>\({\mathit{val}}^n\)</span>をpopします。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックの一番上にフレームFが存在します。</p></li>
  <li><p>スタックからフレームをpopします。</p></li>
  <li><p>スタックに<span>\({\mathit{val}}^n\)</span>をpushしなおします。</p></li>
  <li><p>元々の呼び出しの後にジャンプします。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{lcl&#64;{\qquad}l}
{\mathsf{frame}}_n\{F\}~{\mathit{val}}^n~{\mathsf{end}} &amp;{\hookrightarrow}&amp; {\mathit{val}}^n
\end{array}\end{split}\]</div>

<h3>Host Functions</h3>
ホスト関数の呼び出しは、非決定論的な振る舞いをします。
それは、トラップで終了するか、定期的に返すかのどちらかです。
しかし、後者の場合、関数の型に応じて、スタック上に適切な数と型の`WebAssembly`値を消費して生成しなければなりません。

ホスト関数は、ストアを修正することもできます。
しかし、すべてのストアの変更は、元のストアの拡張に帰結しなければなりません。

つまり、変更可能なコンテンツのみを変更しインスタンスを削除してはなりません。
さらに、結果として得られるストアは有効である必要があります。
すなわち、すべてのデータとコードが正しく型付けされていなければなりません。

<div>\[\begin{split}~\\[-1ex]
\begin{array}{l}
\begin{array}{lcl&#64;{\qquad}l}
S; {\mathit{val}}^n~({\mathsf{invoke}}~a) &amp;{\hookrightarrow}&amp; S'; {\mathit{result}}
\end{array}
\\ \qquad
  \begin{array}[t]{&#64;{}r&#64;{~}l&#64;{}}
  (\mathrel{\mbox{if}} &amp; S.{\mathsf{funcs}}[a] = \{ {\mathsf{type}}~[t_1^n] {\rightarrow} [t_2^m], {\mathsf{hostcode}}~\mathit{hf} \} \\
  \wedge &amp; (S'; {\mathit{result}}) \in \mathit{hf}(S; {\mathit{val}}^n)) \\
  \end{array} \\
\begin{array}{lcl&#64;{\qquad}l}
S; {\mathit{val}}^n~({\mathsf{invoke}}~a) &amp;{\hookrightarrow}&amp; S; {\mathit{val}}^n~({\mathsf{invoke}}~a)
\end{array}
\\ \qquad
  \begin{array}[t]{&#64;{}r&#64;{~}l&#64;{}}
  (\mathrel{\mbox{if}} &amp; S.{\mathsf{funcs}}[a] = \{ {\mathsf{type}}~[t_1^n] {\rightarrow} [t_2^m], {\mathsf{hostcode}}~\mathit{hf} \} \\
  \wedge &amp; \bot \in \mathit{hf}(S; {\mathit{val}}^n)) \\
  \end{array} \\
\end{array}\end{split}\]</div>

ここで、<span>\(\mathit{hf}(S; {\mathit{val}}^n)\)</span>は、引数<span>\({\mathit{val}}^n\)</span>を持つ現在のストアSにおけるホスト関数<span>\(\mathit{hf}\)</span>の実装定義実行を示します。
それは可能な結果の集合を返し、各要素は修正されたストア<span>\(S'\)</span>と結果のペアか発散を示す特別な値⊥のいずれかです。
ホスト関数は、結果の集合が単数ではない少なくとも1つの引数がある場合、非決定論的です。

ホスト関数の存在下でWebAssemblyの実装が健全であるためには、すべてのホスト関数のインスタンスは有効でなければなりません。

つまり、適切な前後条件に従うことを意味します：
有効なストアSの下で、与えられた引数valnがパラメータ型tn1にマッチし、ホスト関数を実行すると、それぞれが発散であるか、あるいはSの拡張である有効なストアS′と、与えられた戻り値型tm2にマッチする結果からなる、空でない可能性のある結果の集合が得られなければなりません。

これらの概念はすべて付録で正確に説明されています。

### 付記

ホスト関数は、モジュールからエクスポートされた関数を呼び出すことで WebAssembly に呼び戻すことができます。
しかし、そのような呼び出しの効果は、ホスト関数に許可されている非決定論的な振る舞いに支配されます。

---

## 式

式は、そのモジュールインスタンスを指す現在のフレームから相対的に評価されます。

<ol>
  <li><p>式の命令シーケンス<span>\({\mathit{instr}}^\ast\)</span>の最初にジャンプします。</p></li>
  <li><p>命令シーケンスを実行します。</p></li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックの一番上には値が存在します。</p></li>
  <li><p>スタックから値<span>\({\mathit{val}}\)</span>をpopします</p></li>
</ol>
値<span>\({\mathit{val}}\)</span>は評価の結果です。
<div>\[S; F; {\mathit{instr}}^\ast {\hookrightarrow} S'; F'; {\mathit{instr}}'^\ast
\qquad (\mathrel{\mbox{if}} S; F; {\mathit{instr}}^\ast~{\mathsf{end}} {\hookrightarrow} S'; F'; {\mathit{instr}}'^\ast~{\mathsf{end}})\]</div>

### 付記

式の評価はこの還元規則を値に到達するまで繰り返し実行します。
関数本体を構成する式は、関数呼び出し時に実行されます。

# モジュール

モジュールの場合、実行セマンティクスは主にインスタンス化を定義し、モジュールとそれに含まれる定義のインスタンスを割り当て、含まれる要素とDataセグメントからテーブルとメモリを初期化し、存在する場合は開始関数を呼び出します。
またエクスポートされた関数の呼び出しも含まれます。

インスタンス化は、Importの型チェックやインスタンスの割り当てのための多くの補助的な概念に依存します。

## 外部型

Importに対する外部値をチェックする目的の下、そのような値は外部型を付与されます。
以下の補助型付け規則は、参照されたインスタンスが存在するストアSに対する相対的な型付け関係を指定します。

<h3><span>\({\mathsf{func}}~a\)</span></h3>
<ul>
  <li><p>ストアのエントリー<span>\(S.{\mathsf{funcs}}[a]\)</span>は必ず関数インスタンス<span>\({\mathsf{type}}~{\mathit{functype}}, \dots\)</span>である必要があります。</p></li>
  <li><p>この時、外部型<span>\({\mathsf{func}}~{\mathit{functype}}\)</span>について<span>\({\mathsf{func}}~a\)</span>は有効です。</p></li>
</ul>
<div>\[\frac{
  S.{\mathsf{funcs}}[a] = \{{\mathsf{type}}~{\mathit{functype}}, \dots\}
}{
  S {\vdash} {\mathsf{func}}~a : {\mathsf{func}}~{\mathit{functype}}
}\]</div>

<h3><span>\({\mathsf{table}}~a\)</span></h3>
<ul>
  <li><p>ストアのエントリー<span>\(S.{\mathsf{tables}}[a]\)</span>は必ずテーブルインスタンス<span>\(\{{\mathsf{elem}}~(\mathit{fa}^?)^n, {\mathsf{max}}~m^?\}\)</span>である必要があります。</p></li>
  <li><p>この時、外部型<span>\({\mathsf{table}}~(\{{\mathsf{min}}~n, {\mathsf{max}}~m^?\}~{\mathsf{funcref}})\)</span>について<span>\({\mathsf{table}}~a\)</span>は有効です。</p></li>
</ul>
<div>\[\frac{
  S.{\mathsf{tables}}[a] = \{ {\mathsf{elem}}~(\mathit{fa}^?)^n, {\mathsf{max}}~m^? \}
}{
  S {\vdash} {\mathsf{table}}~a : {\mathsf{table}}~(\{{\mathsf{min}}~n, {\mathsf{max}}~m^?\}~{\mathsf{funcref}})
}\]</div>

<h3><span>\({\mathsf{mem}}~a\)</span></h3>
<ul>
  <li><p>ストアのエントリー<span>\(S.{\mathsf{mems}}[a]\)</span>は必ずnについてメモリインスタンス<span>\(\{{\mathsf{data}}~b^{n\cdot64\,\mathrm{Ki}}, {\mathsf{max}}~m^?\}\)</span>である必要があります。</p></li>
  <li><p>この時、外部型<span>\({\mathsf{mem}}~(\{{\mathsf{min}}~n, {\mathsf{max}}~m^?\})\)</span>について<span>\({\mathsf{mem}}~a\)</span>は有効です。</p></li>
</ul>
<div>\[\frac{
  S.{\mathsf{mems}}[a] = \{ {\mathsf{data}}~b^{n\cdot64\,\mathrm{Ki}}, {\mathsf{max}}~m^? \}
}{
  S {\vdash} {\mathsf{mem}}~a : {\mathsf{mem}}~\{{\mathsf{min}}~n, {\mathsf{max}}~m^?\}
}\]</div>

<h3><span>\({\mathsf{global}}~a\)</span></h3>
<ul>
  <li><p>ストアのエントリー<span>\(S.{\mathsf{globals}}[a]\)</span>は必ずグローバルインスタンス<span>\(\{{\mathsf{value}}~(t.{\mathsf{const}}~c), {\mathsf{mut}}~{\mathit{mut}}\}\)</span>である必要があります。</p></li>
  <li><p>この時、外部型<span>\({\mathsf{global}}~({\mathit{mut}}~t)\)</span>について<span>\({\mathsf{global}}~a\)</span>は有効です。</p></li>
</ul>
<div>\[\frac{
  S.{\mathsf{globals}}[a] = \{ {\mathsf{value}}~(t.{\mathsf{const}}~c), {\mathsf{mut}}~{\mathit{mut}} \}
}{
  S {\vdash} {\mathsf{global}}~a : {\mathsf{global}}~({\mathit{mut}}~t)
}\]</div>

## Importマッチング

モジュールをインスタンス化する際には、各Importを分類しているそれぞれの外部型と一致する型の外部値を提供しなければなりません。
場合によっては、以下に定義されているように、これによってシンプルな形でのサブタイプ化が可能になります。

<h3>Limits</h3>

リミット<span>\(\{ {\mathsf{min}}~n_1, {\mathsf{max}}~m_1^? \}\)</span>は以下の条件を満たした場合のみリミット<span>\(\{ {\mathsf{min}}~n_2, {\mathsf{max}}~m_2^? \}\)</span>に合致します:

<ul>
  <li><p><span>\(n_1\)</span>は<span>\(n_2\)</span>以上です</p></li>
  <li>どちらか一方に該当します:
    <ul>
      <li><p><span>\(m_2^?\)</span>は空です。</p></li>
    </ul>
  </li>
  <li>あるいは:
    <ul>
      <li><p><span>\(m_1^?\)</span>と<span>\(m_2^?\)</span>は空ではありません。</p></li>
      <li><p><span>\(m_1\)</span>は<span>\(m_2\)</span>以下です。</p></li>
    </ul>
  </li>
</ul>
<div>\[\begin{split}~\\[-1ex]
\frac{
  n_1 \geq n_2
}{
  {\vdash} \{ {\mathsf{min}}~n_1, {\mathsf{max}}~m_1^? \} {\leq} \{ {\mathsf{min}}~n_2, {\mathsf{max}}~\epsilon \}
}
\quad
\frac{
  n_1 \geq n_2
  \qquad
  m_1 \leq m_2
}{
  {\vdash} \{ {\mathsf{min}}~n_1, {\mathsf{max}}~m_1 \} {\leq} \{ {\mathsf{min}}~n_2, {\mathsf{max}}~m_2 \}
}\end{split}\]</div>

<h3>Functions</h3>

<div>外部型<span>\({\mathsf{func}}~{\mathit{functype}}_1\)</span>は<span>\({\mathsf{func}}~{\mathit{functype}}_2\)</span>に次の条件を満たした場合のみ合致します:</div>

<ul>
  <li><p><span>\({\mathit{functype}}_1\)</span>と<span>\({\mathit{functype}}_2\)</span>が等しい。</p></li>
</ul>
<div>\[\begin{split}~\\[-1ex]
\frac{
}{
  {\vdash} {\mathsf{func}}~{\mathit{functype}} {\leq} {\mathsf{func}}~{\mathit{functype}}
}\end{split}\]</div>

<h3>Tables</h3>

<div>外部型<span>\({\mathsf{table}}~({\mathit{limits}}_1~{\mathit{elemtype}}_1)\)</span>は<span>\({\mathsf{table}}~({\mathit{limits}}_2~{\mathit{elemtype}}_2)\)</span>に次の条件を満たした場合のみ合致します:</div>

<ul>
  <li><p>リミット<span>\({\mathit{limits}}_1\)</span>は<span>\({\mathsf{table}}~{\mathit{limits}}_2\)</span>に合致します。</p></li>
  <li><p><span>\({\mathit{elemtype}}_1\)</span>と<span>\({\mathit{elemtype}}_2\)</span>が等しい。</p></li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{limits}}_1 {\leq} {\mathit{limits}}_2
}{
  {\vdash} {\mathsf{table}}~({\mathit{limits}}_1~{\mathit{elemtype}}) {\leq} {\mathsf{table}}~({\mathit{limits}}_2~{\mathit{elemtype}})
}\]</div>

<h3>Memories</h3>

外部型<span>\({\mathsf{mem}}~{\mathit{limits}}_1\)</span>は<span>\({\mathsf{mem}}~{\mathit{limits}}_2\)</span>に次の条件を満たした場合のみ合致します:

<ul>
  <li><p>リミット<span>\({\mathit{limits}}_1\)</span>は<span>\({\mathsf{mem}}~{\mathit{limits}}_2\)</span>に合致する。</p></li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{limits}}_1 {\leq} {\mathit{limits}}_2
}{
  {\vdash} {\mathsf{mem}}~{\mathit{limits}}_1 {\leq} {\mathsf{mem}}~{\mathit{limits}}_2
}\]</div>

<h3>Globals</h3>

外部型<span>\({\mathsf{global}}~{\mathit{globaltype}}_1\)</span>は<span>\({\mathsf{global}}~{\mathit{globaltype}}_2\)</span>に次の条件を満たした場合のみ合致します:

<ul>
  <li><p><span>\({\mathit{globaltype}}_1\)</span>と<span>\({\mathit{globaltype}}_2\)</span>が等しい。</p></li>
</ul>
<div>\[\begin{split}~\\[-1ex]
\frac{
}{
  {\vdash} {\mathsf{global}}~{\mathit{globaltype}} {\leq} {\mathsf{global}}~{\mathit{globaltype}}
}\end{split}\]</div>

## アロケーション

関数、テーブル、メモリ、グローバルの新規インスタンスは、以下の補助関数で定義されたストア S に割り当てられます。

<h3>Functions</h3>
<ol>
  <li><p><span>\({\mathit{func}}\)</span>はthe function to allocate and <span>\({\mathit{moduleinst}}\)</span> its module instanceであるとします。</p></li>
  <li><p>aはS中の最初の自由関数アドレスであるとします。</p></li>
  <li><p><span>\({\mathit{functype}}\)</span>は関数型<span>\({\mathit{moduleinst}}.{\mathsf{types}}[{\mathit{func}}.{\mathsf{type}}]\)</span>であるとします。</p></li>
  <li><p><span>\({\mathit{funcinst}}\)</span>は関数インスタンス<span>\(\{ {\mathsf{type}}~{\mathit{functype}}, {\mathsf{module}}~{\mathit{moduleinst}}, {\mathsf{code}}~{\mathit{func}} \}\)</span>であるとします。</p></li>
  <li><p>Sの<span>\({\mathsf{funcs}}\)</span>に<span>\({\mathit{funcinst}}\)</span>を追加します。</p></li>
  <li><p>aを戻り値とします。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{rlll}
{\mathrm{allocfunc}}(S, {\mathit{func}}, {\mathit{moduleinst}}) &amp;=&amp; S', {\mathit{funcaddr}} \\[1ex]
\mbox{where:} \hfill \\
{\mathit{funcaddr}} &amp;=&amp; |S.{\mathsf{funcs}}| \\
{\mathit{functype}} &amp;=&amp; {\mathit{moduleinst}}.{\mathsf{types}}[{\mathit{func}}.{\mathsf{type}}] \\
{\mathit{funcinst}} &amp;=&amp; \{ {\mathsf{type}}~{\mathit{functype}}, {\mathsf{module}}~{\mathit{moduleinst}}, {\mathsf{code}}~{\mathit{func}} \} \\
S' &amp;=&amp; S {\oplus} \{{\mathsf{funcs}}~{\mathit{funcinst}}\} \\
\end{array}\end{split}\]</div>

<h3>Host Functions</h3>
<ol>
  <li><p><span>\({\mathit{hostfunc}}\)</span>はthe host function to allocate and <span>\({\mathit{functype}}\)</span> its function typeであるとします。</p></li>
  <li><p>aはS中の最初の自由関数アドレスであるとします。</p></li>
  <li><p><span>\({\mathit{funcinst}}\)</span>は関数インスタンス<span>\(\{ {\mathsf{type}}~{\mathit{functype}}, {\mathsf{hostcode}}~{\mathit{hostfunc}} \}\)</span>であるとします。</p></li>
  <li><p>Sの<span>\({\mathsf{funcs}}\)</span>に<span>\({\mathit{funcinst}}\)</span>を追加します。</p></li>
  <li><p>aを戻り値とします。</p></li>
</ol>
<div>\[\begin{split}~\\[-1ex]
\begin{array}{rlll}
{\mathrm{allochostfunc}}(S, {\mathit{functype}}, {\mathit{hostfunc}}) &amp;=&amp; S', {\mathit{funcaddr}} \\[1ex]
\mbox{where:} \hfill \\
{\mathit{funcaddr}} &amp;=&amp; |S.{\mathsf{funcs}}| \\
{\mathit{funcinst}} &amp;=&amp; \{ {\mathsf{type}}~{\mathit{functype}}, {\mathsf{hostcode}}~{\mathit{hostfunc}} \} \\
S' &amp;=&amp; S {\oplus} \{{\mathsf{funcs}}~{\mathit{funcinst}}\} \\
\end{array}\end{split}\]</div>

<h3>Tables</h3>
<ol>
  <li><p><span>\({\mathit{tabletype}}\)</span>はthe table type to allocateであるとします。</p></li>
  <li><p><span>\((\{{\mathsf{min}}~n, {\mathsf{max}}~m^?\}~{\mathit{elemtype}})\)</span>はthe structure of table type <span>\({\mathit{tabletype}}\)</span>であるとします。</p></li>
  <li><p>aはthe first free table address in Sであるとします。</p></li>
  <li><p><span>\({\mathit{tableinst}}\)</span>はテーブルインスタンス<span>\(\{ {\mathsf{elem}}~(\epsilon)^n, {\mathsf{max}}~m^? \}\)</span> with n empty elementsであるとします。</p></li>
  <li><p>Sの<span>\({\mathsf{tables}}\)</span>に<span>\({\mathit{tableinst}}\)</span>を追加します。</p></li>
  <li><p>aを戻り値とします。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{rlll}
{\mathrm{alloctable}}(S, {\mathit{tabletype}}) &amp;=&amp; S', {\mathit{tableaddr}} \\[1ex]
\mbox{where:} \hfill \\
{\mathit{tabletype}} &amp;=&amp; \{{\mathsf{min}}~n, {\mathsf{max}}~m^?\}~{\mathit{elemtype}} \\
{\mathit{tableaddr}} &amp;=&amp; |S.{\mathsf{tables}}| \\
{\mathit{tableinst}} &amp;=&amp; \{ {\mathsf{elem}}~(\epsilon)^n, {\mathsf{max}}~m^? \} \\
S' &amp;=&amp; S {\oplus} \{{\mathsf{tables}}~{\mathit{tableinst}}\} \\
\end{array}\end{split}\]</div>

<h3>Memories</h3>
<ol>
  <li><p><span>\({\mathit{memtype}}\)</span>はthe memory type to allocateであるとします。</p></li>
  <li><p><span>\(\{{\mathsf{min}}~n, {\mathsf{max}}~m^?\}\)</span>はthe structure of memory type <span>\({\mathit{memtype}}\)</span>であるとします。</p></li>
  <li><p>aはS中の最初の自由メモリアドレスであるとします。</p></li>
  <li><p><span>\({\mathit{meminst}}\)</span>はメモリインスタンス<span>\(\{ {\mathsf{data}}~(\def\mathdef1219#1{\mathtt{0x#1}}\mathdef1219{00})^{n \cdot 64\,\mathrm{Ki}}, {\mathsf{max}}~m^? \}\)</span> that contains n pages of zeroed bytesであるとします。</p></li>
  <li><p>Sの<span>\({\mathsf{mems}}\)</span>に<span>\({\mathit{meminst}}\)</span>を追加します。</p></li>
  <li><p>aを戻り値とします。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{rlll}
{\mathrm{allocmem}}(S, {\mathit{memtype}}) &amp;=&amp; S', {\mathit{memaddr}} \\[1ex]
\mbox{where:} \hfill \\
{\mathit{memtype}} &amp;=&amp; \{{\mathsf{min}}~n, {\mathsf{max}}~m^?\} \\
{\mathit{memaddr}} &amp;=&amp; |S.{\mathsf{mems}}| \\
{\mathit{meminst}} &amp;=&amp; \{ {\mathsf{data}}~(\def\mathdef1220#1{\mathtt{0x#1}}\mathdef1220{00})^{n \cdot 64\,\mathrm{Ki}}, {\mathsf{max}}~m^? \} \\
S' &amp;=&amp; S {\oplus} \{{\mathsf{mems}}~{\mathit{meminst}}\} \\
\end{array}\end{split}\]</div>

<h3>Globals</h3>
<ol>
  <li><p><span>\({\mathit{globaltype}}\)</span>はアロケート予定のグローバル型であり、<span>\({\mathit{val}}\)</span>はグローバル初期化に使用する値であるとします。</p></li>
  <li><p><span>\({\mathit{mut}}~t\)</span>はグローバル型<span>\({\mathit{globaltype}}\)</span>の構造であるとします。</p></li>
  <li><p>aはS中の最初の自由グローバルアドレスであるとします。</p></li>
  <li><p><span>\({\mathit{globalinst}}\)</span>はグローバルインスタンス<span>\(\{ {\mathsf{value}}~{\mathit{val}}, {\mathsf{mut}}~{\mathit{mut}} \}\)</span>であるとします。</p></li>
  <li><p>Sの<span>\({\mathsf{globals}}\)</span>に<span>\({\mathit{globalinst}}\)</span>を追加します。</p></li>
  <li><p>aを戻り値とします。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{rlll}
{\mathrm{allocglobal}}(S, {\mathit{globaltype}}, {\mathit{val}}) &amp;=&amp; S', {\mathit{globaladdr}} \\[1ex]
\mbox{where:} \hfill \\
{\mathit{globaltype}} &amp;=&amp; {\mathit{mut}}~t \\
{\mathit{globaladdr}} &amp;=&amp; |S.{\mathsf{globals}}| \\
{\mathit{globalinst}} &amp;=&amp; \{ {\mathsf{value}}~{\mathit{val}}, {\mathsf{mut}}~{\mathit{mut}} \} \\
S' &amp;=&amp; S {\oplus} \{{\mathsf{globals}}~{\mathit{globalinst}}\} \\
\end{array}\end{split}\]</div>

<h3>Growing tables</h3>
<ol>
  <li><p><span>\({\mathit{tableinst}}\)</span>はテーブルインスタンスto grow and n the number of elements by which to grow itであるとします。</p></li>
  <li><p><span>\(\mathit{len}\)</span>はn added to the length of <span>\({\mathit{tableinst}}.{\mathsf{elem}}\)</span>であるとします。</p></li>
  <li><p>もし<span>\(\mathit{len}\)</span> is larger than or equal to <span>\(2^{32}\)</span>, then fail。</p></li>
  <li><p>もし<span>\({\mathit{tableinst}}.{\mathsf{max}}\)</span> is not empty and its value is smaller than <span>\(\mathit{len}\)</span>, then fail。</p></li>
  <li><p><span>\({\mathit{tableinst}}.{\mathsf{elem}}\)</span>にn empty elementsを追加します。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{rllll}
{\mathrm{growtable}}({\mathit{tableinst}}, n) &amp;=&amp; {\mathit{tableinst}} {\mathrel{\mbox{with}}} {\mathsf{elem}} = {\mathit{tableinst}}.{\mathsf{elem}}~(\epsilon)^n \\
  &amp;&amp; (
    \begin{array}[t]{&#64;{}r&#64;{~}l&#64;{}}
    \mathrel{\mbox{if}} &amp; \mathit{len} = n + |{\mathit{tableinst}}.{\mathsf{elem}}| \\
    \wedge &amp; \mathit{len} &lt; 2^{32} \\
    \wedge &amp; ({\mathit{tableinst}}.{\mathsf{max}} = \epsilon \vee \mathit{len} \leq {\mathit{tableinst}}.{\mathsf{max}})) \\
    \end{array} \\
\end{array}\end{split}\]</div>

<h3>Growing memories</h3>
<ol>
  <li><p><span>\({\mathit{meminst}}\)</span>は伸長するメモリインスタンスであり、nは伸長するページ数であるとします。</p></li>
  <li><p>前提条件：<span>\({\mathit{meminst}}.{\mathsf{data}}\)</span>の長さはページサイズ<span>\(64\,\mathrm{Ki}\)</span>により割り切れます。</p></li>
  <li><p><span>\(\mathit{len}\)</span>はnに<span>\({\mathit{meminst}}.{\mathsf{data}}\)</span>の長さをページサイズ<span>\(64\,\mathrm{Ki}\)</span>で除算したものを加算したものであるとします。</p></li>
  <li><p>もし<span>\(\mathit{len}\)</span>が<span>\(2^{16}\)</span>より大きいならば失敗します。</p></li>
  <li><p>もし<span>\({\mathit{meminst}}.{\mathsf{max}}\)</span>が空ではなく、かつその値が<span>\(\mathit{len}\)</span>未満ならば失敗します。</p></li>
  <li><p><span>\({\mathit{meminst}}.{\mathsf{data}}\)</span>にn×<span>\(64\,\mathrm{Ki}\)</span> bytesな値<span>\(\def\mathdef1221#1{\mathtt{0x#1}}\mathdef1221{00}\)</span>を追加します。</p></li>
</ol>
<div>\[\begin{split}\begin{array}{rllll}
{\mathrm{growmem}}({\mathit{meminst}}, n) &amp;=&amp; {\mathit{meminst}} {\mathrel{\mbox{with}}} {\mathsf{data}} = {\mathit{meminst}}.{\mathsf{data}}~(\def\mathdef1222#1{\mathtt{0x#1}}\mathdef1222{00})^{n \cdot 64\,\mathrm{Ki}} \\
  &amp;&amp; (
    \begin{array}[t]{&#64;{}r&#64;{~}l&#64;{}}
    \mathrel{\mbox{if}} &amp; \mathit{len} = n + |{\mathit{meminst}}.{\mathsf{data}}| / 64\,\mathrm{Ki} \\
    \wedge &amp; \mathit{len} \leq 2^{16} \\
    \wedge &amp; ({\mathit{meminst}}.{\mathsf{max}} = \epsilon \vee \mathit{len} \leq {\mathit{meminst}}.{\mathsf{max}})) \\
    \end{array} \\
\end{array}\end{split}\]</div>

<h3>Modules</h3>

モジュールをアロケートする関数は適切なモジュールのImportベクトルに合致するとされる外部値のリストとモジュールのグローバルを初期化する値のリストであるとします。

<ol>
  <li><p><span>\({\mathit{module}}\)</span>はアロケートするモジュールと<span>\({\mathit{externval}}_{\mathrm{im}}^\ast\)</span>モジュールのImportに使用される値のリストと<span>\({\mathit{val}}^\ast\)</span>がモジュールのグローバルを初期化する値であるとします。</p></li>
  <li><span>\({\mathit{module}}.{\mathsf{funcs}}\)</span>中の各関数<span>\({\mathit{func}}_i\)</span>について:
    <ol>
      <li><p><span>\({\mathit{funcaddr}}_i\)</span>は以下に定義されるモジュールインスタンス<span>\({\mathit{moduleinst}}\)</span>によりアロケートされる関数<span>\({\mathit{func}}_i\)</span>であるとします。</p></li>
    </ol>
  </li>
  <li><span>\({\mathit{module}}.{\mathsf{tables}}\)</span>中の各テーブル<span>\({\mathit{table}}_i\)</span>について:
    <ol>
      <li><p><span>\({\mathit{tableaddr}}_i\)</span>は<span>\({\mathit{table}}_i.{\mathsf{type}}\)</span>によりアロケートされるテーブルであるとします。</p></li>
    </ol>
  </li>
  <li><span>\({\mathit{module}}.{\mathsf{mems}}\)</span>中の各メモリ<span>\({\mathit{mem}}_i\)</span>について:
    <ol>
      <li><p><span>\({\mathit{memaddr}}_i\)</span>は<span>\({\mathit{mem}}_i.{\mathsf{type}}\)</span>によりアロケートされるメモリアドレスであるとします。</p></li>
    </ol>
  </li>
  <li><span>\({\mathit{module}}.{\mathsf{globals}}\)</span>中の各グローバル<span>\({\mathit{global}}_i\)</span>について:
    <ol>
      <li><p><span>\({\mathit{globaladdr}}_i\)</span>は値<span>\({\mathit{val}}^\ast[i]\)</span>を以て初期化される<span>\({\mathit{global}}_i.{\mathsf{type}}\)</span>によりアロケートされるグローバルアドレスであるとします。</p></li>
    </ol>
  </li>
  <li><p><span>\({\mathit{funcaddr}}^\ast\)</span>はインデックスで整列された関数アドレス<span>\({\mathit{funcaddr}}_i\)</span>の連続したものであるとします。</p></li>
  <li><p><span>\({\mathit{tableaddr}}^\ast\)</span>はインデックスで整列されたテーブルアドレス<span>\({\mathit{tableaddr}}_i\)</span>の連続したものであるとします。</p></li>
  <li><p><span>\({\mathit{memaddr}}^\ast\)</span>はインデックスで整列されたメモリアドレス<span>\({\mathit{memaddr}}_i\)</span>の連続したものであるとします。</p></li>
  <li><p><span>\({\mathit{globaladdr}}^\ast\)</span>はインデックスで整列されたグローバルアドレス<span>\({\mathit{globaladdr}}_i\)</span>の連続したものであるとします。</p></li>
  <li><p><span>\({\mathit{funcaddr}}_{\mathrm{mod}}^\ast\)</span>は<span>\({\mathit{funcaddr}}^\ast\)</span>に連続する<span>\({\mathit{externval}}_{\mathrm{im}}^\ast\)</span>から抽出された関数アドレスのリストであるとします。</p></li>
  <li><p><span>\({\mathit{tableaddr}}_{\mathrm{mod}}^\ast\)</span>は<span>\({\mathit{tableaddr}}^\ast\)</span>に連続する<span>\({\mathit{externval}}_{\mathrm{im}}^\ast\)</span>から抽出されたテーブルアドレスのリストであるとします。</p></li>
  <li><p><span>\({\mathit{memaddr}}_{\mathrm{mod}}^\ast\)</span>は<span>\({\mathit{memaddr}}^\ast\)</span>に連続する<span>\({\mathit{externval}}_{\mathrm{im}}^\ast\)</span>から抽出されたメモリアドレスのリストであるとします。</p></li>
  <li><p><span>\({\mathit{globaladdr}}_{\mathrm{mod}}^\ast\)</span>は<span>\({\mathit{globaladdr}}^\ast\)</span>に連続する<span>\({\mathit{externval}}_{\mathrm{im}}^\ast\)</span>から抽出されたグローバルアドレスのリストであるとします。</p></li>
  <li><span>\({\mathit{module}}.{\mathsf{exports}}\)</span>中の各Export<span>\({\mathit{export}}_i\)</span>について:
    <ol>
      <li><p>もし<span>\({\mathit{export}}_i\)</span>がExport関数インデックスxに対応する関数であるならば、<span>\({\mathit{externval}}_i\)</span>は外部値<span>\({\mathsf{func}}~({\mathit{funcaddr}}_{\mathrm{mod}}^\ast[x])\)</span>であるとします。</p></li>
      <li><p>そうでないならば、<span>\({\mathit{export}}_i\)</span>がExportテーブルインデックスxに対応するテーブルであるならば、<span>\({\mathit{externval}}_i\)</span>は外部値<span>\({\mathsf{table}}~({\mathit{tableaddr}}_{\mathrm{mod}}^\ast[x])\)</span>であるとします。</p></li>
      <li><p>そうでないならば、<span>\({\mathit{export}}_i\)</span>がExportメモリインデックスxに対応するメモリであるならば、<span>\({\mathit{externval}}_i\)</span>は外部値<span>\({\mathsf{mem}}~({\mathit{memaddr}}_{\mathrm{mod}}^\ast[x])\)</span>であるとします。</p></li>
      <li><p>そうでないならば、<span>\({\mathit{export}}_i\)</span>がExportグローバルインデックスxに対応するグローバルであるならば、<span>\({\mathit{externval}}_i\)</span>は外部値<span>\({\mathsf{global}}~({\mathit{globaladdr}}_{\mathrm{mod}}^\ast[x])\)</span>であるとします。</p></li>
      <li><p><span>\({\mathit{exportinst}}_i\)</span>はExportインスタンス<span>\(\{{\mathsf{name}}~({\mathit{export}}_i.{\mathsf{name}}), {\mathsf{value}}~{\mathit{externval}}_i\}\)</span>であるとします。</p></li>
    </ol>
  </li>
  <li><p><span>\({\mathit{exportinst}}^\ast\)</span>はthe the concatenation of the export instances <span>\({\mathit{exportinst}}_i\)</span> in index orderであるとします。</p></li>
  <li><p><span>\({\mathit{moduleinst}}\)</span>はthe module instance <span>\(\{{\mathsf{types}}~({\mathit{module}}.{\mathsf{types}}),\)</span> <span>\({\mathsf{funcaddrs}}~{\mathit{funcaddr}}_{\mathrm{mod}}^\ast,\)</span> <span>\({\mathsf{tableaddrs}}~{\mathit{tableaddr}}_{\mathrm{mod}}^\ast,\)</span> <span>\({\mathsf{memaddrs}}~{\mathit{memaddr}}_{\mathrm{mod}}^\ast,\)</span> <span>\({\mathsf{globaladdrs}}~{\mathit{globaladdr}}_{\mathrm{mod}}^\ast,\)</span> <span>\({\mathsf{exports}}~{\mathit{exportinst}}^\ast\}\)</span>であるとします。</p></li>
  <li><p><span>\({\mathit{moduleinst}}\)</span>を戻り値とします。</p></li>
</ol>
<div>\[\begin{split}~\\
\begin{array}{rlll}
{\mathrm{allocmodule}}(S, {\mathit{module}}, {\mathit{externval}}_{\mathrm{im}}^\ast, {\mathit{val}}^\ast) &amp;=&amp; S', {\mathit{moduleinst}}    \end{array}\end{split}\]</div>
where:
<div>\[\begin{split}\begin{array}{rlll}
{\mathit{moduleinst}} &amp;=&amp; \{~
  \begin{array}[t]{&#64;{}l&#64;{}}
  {\mathsf{types}}~{\mathit{module}}.{\mathsf{types}}, \\
  {\mathsf{funcaddrs}}~{\mathrm{funcs}}({\mathit{externval}}_{\mathrm{im}}^\ast)~{\mathit{funcaddr}}^\ast, \\
  {\mathsf{tableaddrs}}~{\mathrm{tables}}({\mathit{externval}}_{\mathrm{im}}^\ast)~{\mathit{tableaddr}}^\ast, \\
  {\mathsf{memaddrs}}~{\mathrm{mems}}({\mathit{externval}}_{\mathrm{im}}^\ast)~{\mathit{memaddr}}^\ast, \\
  {\mathsf{globaladdrs}}~{\mathrm{globals}}({\mathit{externval}}_{\mathrm{im}}^\ast)~{\mathit{globaladdr}}^\ast, \\
  {\mathsf{exports}}~{\mathit{exportinst}}^\ast ~\}
  \end{array} \\[1ex]
S_1, {\mathit{funcaddr}}^\ast &amp;=&amp; {\mathrm{allocfunc}}^\ast(S, {\mathit{module}}.{\mathsf{funcs}}, {\mathit{moduleinst}}) \\
S_2, {\mathit{tableaddr}}^\ast &amp;=&amp; {\mathrm{alloctable}}^\ast(S_1, ({\mathit{table}}.{\mathsf{type}})^\ast)
  \qquad\qquad\qquad~ (\mathrel{\mbox{where}} {\mathit{table}}^\ast = {\mathit{module}}.{\mathsf{tables}}) \\
S_3, {\mathit{memaddr}}^\ast &amp;=&amp; {\mathrm{allocmem}}^\ast(S_2, ({\mathit{mem}}.{\mathsf{type}})^\ast)
  \qquad\qquad\qquad~ (\mathrel{\mbox{where}} {\mathit{mem}}^\ast = {\mathit{module}}.{\mathsf{mems}}) \\
S', {\mathit{globaladdr}}^\ast &amp;=&amp; {\mathrm{allocglobal}}^\ast(S_3, ({\mathit{global}}.{\mathsf{type}})^\ast, {\mathit{val}}^\ast)
  \qquad\quad~ (\mathrel{\mbox{where}} {\mathit{global}}^\ast = {\mathit{module}}.{\mathsf{globals}}) \\
{\mathit{exportinst}}^\ast &amp;=&amp; \{ {\mathsf{name}}~({\mathit{export}}.{\mathsf{name}}), {\mathsf{value}}~{\mathit{externval}}_{\mathrm{ex}} \}^\ast
  \quad (\mathrel{\mbox{where}} {\mathit{export}}^\ast = {\mathit{module}}.{\mathsf{exports}}) \\[1ex]
{\mathrm{funcs}}({\mathit{externval}}_{\mathrm{ex}}^\ast) &amp;=&amp; ({\mathit{moduleinst}}.{\mathsf{funcaddrs}}[x])^\ast
  \qquad~ (\mathrel{\mbox{where}} x^\ast = {\mathrm{funcs}}({\mathit{module}}.{\mathsf{exports}})) \\
{\mathrm{tables}}({\mathit{externval}}_{\mathrm{ex}}^\ast) &amp;=&amp; ({\mathit{moduleinst}}.{\mathsf{tableaddrs}}[x])^\ast
  \qquad (\mathrel{\mbox{where}} x^\ast = {\mathrm{tables}}({\mathit{module}}.{\mathsf{exports}})) \\
{\mathrm{mems}}({\mathit{externval}}_{\mathrm{ex}}^\ast) &amp;=&amp; ({\mathit{moduleinst}}.{\mathsf{memaddrs}}[x])^\ast
  \qquad (\mathrel{\mbox{where}} x^\ast = {\mathrm{mems}}({\mathit{module}}.{\mathsf{exports}})) \\
{\mathrm{globals}}({\mathit{externval}}_{\mathrm{ex}}^\ast) &amp;=&amp; ({\mathit{moduleinst}}.{\mathsf{globaladdrs}}[x])^\ast
  \qquad\!\!\! (\mathrel{\mbox{where}} x^\ast = {\mathrm{globals}}({\mathit{module}}.{\mathsf{exports}})) \\
\end{array}\end{split}\]</div>
Here, the notation <span>\(\mathrm{allocx}^\ast\)</span> is shorthand for multiple allocations of object kind X, defined as follows:
<div>\[\begin{split}\begin{array}{rlll}
\mathrm{allocx}^\ast(S_0, X^n, \dots) &amp;=&amp; S_n, a^n \\[1ex]
\mbox{where for all $i &lt; n$:} \hfill \\
S_{i+1}, a^n[i] &amp;=&amp; \mathrm{allocx}(S_i, X^n[i], \dots)
\end{array}\end{split}\]</div>

更にその上、もし積<span>\(\dots\)</span>がシーケンス<span>\(A^n\)</span>ならば、このシーケンスの要素はpassed to the allocation function pointwise。

### 付記

モジュールの割り当ての定義は、必要なクロージャを形成するために結果として得られるモジュールインスタンスmoduleinstが引数として関数アロケータに渡されます。
このため、関連する関数の割り当てと相互に再帰的です。

実装では、この再帰は二次ステップで一方または他方を変更することで簡単に解くことができます。

## インスタンス化

<div>
ストアSが与えられると、モジュールモジュールは、以下のように必要なImportを供給する外部値のリスト<span>\({\mathit{externval}}^n\)</span>でインスタンス化されます。

インスタンス化は、モジュールが有効であり、提供されたImportが宣言された型と一致しているかどうかをチェックし、そうでない場合はエラーで失敗することがあります。
また、インスタンシエーションは、スタート関数を実行することでトラップが発生することもあります。

そのような状態がどのように報告されるかは、エンベッダーに任されています。
</div>

<ol>
  <li>もし<span>\({\mathit{module}}\)</span>が無効ならば:
    <ol>
      <li><p>失敗します。</p></li>
    </ol>
  </li>
  <li><p>前提条件：<span>\({\mathit{module}}\)</span>はImportされた外部型<span>\({\mathit{externtype}}_{\mathrm{im}}^m\)</span>として有効です。</p></li>
  <li>もしImportされた数値mが外部値のnと等しくないならば:
    <ol>
      <li><p>失敗します。</p></li>
    </ol>
  </li>
  <li><span>\({\mathit{externtype}}_{\mathrm{im}}^n\)</span>中の<span>\({\mathit{externval}}^n\)</span>中の各外部値<span>\({\mathit{externval}}_i\)</span>と外部型<span>\({\mathit{externtype}}'_i\)</span>について:
    <ol>
      <li>もし<span>\({\mathit{externval}}_i\)</span>がストアS中に存在する外部型<span>\({\mathit{externtype}}_i\)</span>で無効ならば:
        <ol>
          <li><p>失敗します。</p></li>
        </ol>
      </li>
      <li>もし<span>\({\mathit{externtype}}_i\)</span>が<span>\({\mathit{externtype}}'_i\)</span>に合致しないならば:
        <ol>
          <li><p>失敗します。</p></li>
        </ol>
      </li>
    </ol>
  </li>
</ol>
<ol>
  <li><span>\({\mathit{val}}^\ast\)</span>は<span>\({\mathit{module}}\)</span>と<span>\({\mathit{externval}}^n\)</span>により決定されるグローバル初期化に用いる値のベクトルであるとします。これらは以下のように計算されるでしょう。
    <ol>
      <li><p><span>\({\mathit{moduleinst}}_{\mathrm{im}}\)</span>はImportされたグローバルインスタンスのみによって構成される補助モジュールインスタンス<span>\(\{{\mathsf{globaladdrs}}~{\mathrm{globals}}({\mathit{externval}}^n)\}\)</span>であるとします。</p></li>
      <li><p><span>\(F_{\mathrm{im}}\)</span>は補助フレーム<span>\(\{ {\mathsf{module}}~{\mathit{moduleinst}}_{\mathrm{im}}, {\mathsf{locals}}~\epsilon \}\)</span>であるとします。</p></li>
      <li><p>スタックにフレーム<span>\(F_{\mathrm{im}}\)</span>をpushします。</p></li>
      <li><span>\({\mathit{module}}.{\mathsf{globals}}\)</span>中の各グローバル<span>\({\mathit{global}}_i\)</span>について:
        <ol>
          <li><p><span>\({\mathit{val}}_i\)</span>は初期化式<span>\({\mathit{global}}_i.{\mathsf{init}}\)</span>の評価結果であるとします。</p></li>
        </ol>
      </li>
      <li><p>前提条件：バリデーション/検証を経て保証されることですが、フレーム<span>\(F_{\mathrm{im}}\)</span>はスタックの一番上に存在します</p></li>
      <li><p>スタックからフレーム<span>\(F_{\mathrm{im}}\)</span>をpopします。</p></li>
    </ol>
  </li>
  <li><p><span>\({\mathit{moduleinst}}\)</span>がストアS中の<span>\({\mathit{module}}\)</span>においてアロケートされた新しいモジュールのインスタンスであり、Importは<span>\({\mathit{externval}}^n\)</span>で、グローバル初期化の値が<span>\({\mathit{val}}^\ast\)</span>であり、<span>\(S'\)</span>がモジュールのアロケーションにより拡張されたストアであるとします。</p></li>
  <li><p>Fはフレーム<span>\(\{ {\mathsf{module}}~{\mathit{moduleinst}}, {\mathsf{locals}}~\epsilon \}\)</span>であるとします。</p></li>
  <li><p>スタックにフレームFをpushします。</p></li>
  <li><span>\({\mathit{module}}.{\mathsf{elem}}\)</span>中の各Elementセグメント<span>\({\mathit{elem}}_i\)</span>について:
    <blockquote>
      <div>
        <ol>
          <li><p><span>\(\mathit{eoval}_i\)</span>は式<span>\({\mathit{elem}}_i.{\mathsf{offset}}\)</span>の評価結果であるとします。</p></li>
          <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(\mathit{eoval}_i\)</span>が<span>\({\mathsf{i32}}.{\mathsf{const}}~\mathit{eo}_i\)</span>の形式であるとします。</p></li>
          <li><p><span>\({\mathit{tableidx}}_i\)</span>はテーブルインデックス<span>\({\mathit{elem}}_i.{\mathsf{table}}\)</span>であるとします。</p></li>
          <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\({\mathit{moduleinst}}.{\mathsf{tableaddrs}}[{\mathit{tableidx}}_i]\)</span>は存在します。</p></li>
          <li><p><span>\({\mathit{tableaddr}}_i\)</span>はテーブルアドレス<span>\({\mathit{moduleinst}}.{\mathsf{tableaddrs}}[{\mathit{tableidx}}_i]\)</span>であるとします。</p></li>
          <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(S'.{\mathsf{tables}}[{\mathit{tableaddr}}_i]\)</span>は存在します。</p></li>
          <li><p><span>\({\mathit{tableinst}}_i\)</span>はテーブルインスタンス<span>\(S'.{\mathsf{tables}}[{\mathit{tableaddr}}_i]\)</span>であるとします。</p></li>
          <li><p><span>\(\mathit{eend}_i\)</span>は<span>\(\mathit{eo}_i\)</span> plus the length of <span>\({\mathit{elem}}_i.{\mathsf{init}}\)</span>であるとします。</p></li>
          <li>もし<span>\(\mathit{eend}_i\)</span>が<span>\({\mathit{tableinst}}_i.{\mathsf{elem}}\)</span>の長さより大きいならば:
            <ol>
              <li><p>失敗します。</p></li>
            </ol>
          </li>
        </ol>
      </div>
    </blockquote>
  </li>
  <li><span>\({\mathit{module}}.{\mathsf{data}}\)</span>中の各Dataセグメント<span>\({\mathit{data}}_i\)</span>について:
    <ol>
      <li><p><span>\(\mathit{doval}_i\)</span>は式<span>\({\mathit{data}}_i.{\mathsf{offset}}\)</span>の評価結果であるとします。</p></li>
      <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(\mathit{doval}_i\)</span> is of the form <span>\({\mathsf{i32}}.{\mathsf{const}}~\mathit{do}_i\)</span>。</p></li>
      <li><p><span>\({\mathit{memidx}}_i\)</span>はメモリインデックス<span>\({\mathit{data}}_i.{\mathsf{data}}\)</span>であるとします。</p></li>
      <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\({\mathit{moduleinst}}.{\mathsf{memaddrs}}[{\mathit{memidx}}_i]\)</span>は存在します。</p></li>
      <li><p><span>\({\mathit{memaddr}}_i\)</span>はメモリアドレス<span>\({\mathit{moduleinst}}.{\mathsf{memaddrs}}[{\mathit{memidx}}_i]\)</span>であるとします。</p></li>
      <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\(S'.{\mathsf{mems}}[{\mathit{memaddr}}_i]\)</span>は存在します。</p></li>
      <li><p><span>\({\mathit{meminst}}_i\)</span>はメモリインスタンス<span>\(S'.{\mathsf{mems}}[{\mathit{memaddr}}_i]\)</span>であるとします。</p></li>
      <li><p><span>\(\mathit{dend}_i\)</span>は<span>\(\mathit{do}_i\)</span> plus the length of <span>\({\mathit{data}}_i.{\mathsf{init}}\)</span>であるとします。</p></li>
      <li>もし<span>\(\mathit{dend}_i\)</span>が<span>\({\mathit{meminst}}_i.{\mathsf{data}}\)</span>の長さより大きいならば:
        <ol>
          <li><p>失敗します。</p></li>
        </ol>
      </li>
    </ol>
  </li>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、フレームFはスタックの一番上に存在します</p></li>
  <li><p>スタックからフレームをpopします。</p></li>
  <li><span>\({\mathit{module}}.{\mathsf{elem}}\)</span>中の各Elementセグメント<span>\({\mathit{elem}}_i\)</span>について:
    <ol>
      <li><span>\({\mathit{elem}}_i.{\mathsf{init}}\)</span> (<span>\(j = 0\)</span>から開始します)中の各function index <span>\({\mathit{funcidx}}_{ij}\)</span>について:
        <ol>
          <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\({\mathit{moduleinst}}.{\mathsf{funcaddrs}}[{\mathit{funcidx}}_{ij}]\)</span>は存在します。</p></li>
          <li><p><span>\({\mathit{funcaddr}}_{ij}\)</span>は関数アドレス<span>\({\mathit{moduleinst}}.{\mathsf{funcaddrs}}[{\mathit{funcidx}}_{ij}]\)</span>であるとします。</p></li>
          <li><p><span>\({\mathit{tableinst}}_i.{\mathsf{elem}}[\mathit{eo}_i + j]\)</span>を<span>\({\mathit{funcaddr}}_{ij}\)</span>で置換します。</p></li>
        </ol>
      </li>
    </ol>
  </li>
  <li><span>\({\mathit{module}}.{\mathsf{data}}\)</span>中の各Dataセグメント<span>\({\mathit{data}}_i\)</span>について:
    <ol>
      <li><span>\({\mathit{data}}_i.{\mathsf{init}}\)</span> (<span>\(j = 0\)</span>から開始します)中の各byte <span>\(b_{ij}\)</span>について:
        <ol>
          <li><p><span>\({\mathit{meminst}}_i.{\mathsf{data}}[\mathit{do}_i + j]\)</span>を<span>\(b_{ij}\)</span>で置換します。</p></li>
        </ol>
      </li>
    </ol>
  </li>
  <li>もしthe start function <span>\({\mathit{module}}.{\mathsf{start}}\)</span> is not empty, then:
    <ol>
      <li><p>前提条件：バリデーション/検証を経て保証されることですが、<span>\({\mathit{moduleinst}}.{\mathsf{funcaddrs}}[{\mathit{module}}.{\mathsf{start}}.{\mathsf{func}}]\)</span>は存在します。</p></li>
      <li><p><span>\({\mathit{funcaddr}}\)</span>は関数アドレス<span>\({\mathit{moduleinst}}.{\mathsf{funcaddrs}}[{\mathit{module}}.{\mathsf{start}}.{\mathsf{func}}]\)</span>であるとします。</p></li>
      <li><p>関数アドレス<span>\({\mathit{funcaddr}}\)</span>にある関数インスタンスを呼び出します。</p></li>
    </ol>
  </li>
</ol>
<div>\[\begin{split}~\\
\begin{array}{&#64;{}rcll}
{\mathrm{instantiate}}(S, {\mathit{module}}, {\mathit{externval}}^n) &amp;=&amp; S'; F;
  \begin{array}[t]{&#64;{}l&#64;{}}
  ({\mathsf{init\_elem}}~{\mathit{tableaddr}}~\mathit{eo}~{\mathit{elem}}.{\mathsf{init}})^\ast \\
  ({\mathsf{init\_data}}~{\mathit{memaddr}}~\mathit{do}~{\mathit{data}}.{\mathsf{init}})^\ast \\
  ({\mathsf{invoke}}~{\mathit{funcaddr}})^? \\
  \end{array} \\
&amp;(\mathrel{\mbox{if}}
  &amp; {\vdash} {\mathit{module}} : {\mathit{externtype}}_{\mathrm{im}}^n {\rightarrow} {\mathit{externtype}}_{\mathrm{ex}}^\ast \\
  &amp;\wedge&amp; (S {\vdash} {\mathit{externval}} : {\mathit{externtype}})^n \\
  &amp;\wedge&amp; ({\vdash} {\mathit{externtype}} {\leq} {\mathit{externtype}}_{\mathrm{im}})^n \\[1ex]
  &amp;\wedge&amp; {\mathit{module}}.{\mathsf{globals}} = {\mathit{global}}^\ast \\
  &amp;\wedge&amp; {\mathit{module}}.{\mathsf{elem}} = {\mathit{elem}}^\ast \\
  &amp;\wedge&amp; {\mathit{module}}.{\mathsf{data}} = {\mathit{data}}^\ast \\
  &amp;\wedge&amp; {\mathit{module}}.{\mathsf{start}} = {\mathit{start}}^? \\[1ex]
  &amp;\wedge&amp; S', {\mathit{moduleinst}} = {\mathrm{allocmodule}}(S, {\mathit{module}}, {\mathit{externval}}^n, {\mathit{val}}^\ast) \\
  &amp;\wedge&amp; F = \{ {\mathsf{module}}~{\mathit{moduleinst}}, {\mathsf{locals}}~\epsilon \} \\[1ex]
  &amp;\wedge&amp; (S'; F; {\mathit{global}}.{\mathsf{init}} {\hookrightarrow}^\ast S'; F; {\mathit{val}}~{\mathsf{end}})^\ast \\
  &amp;\wedge&amp; (S'; F; {\mathit{elem}}.{\mathsf{offset}} {\hookrightarrow}^\ast S'; F; {\mathsf{i32}}.{\mathsf{const}}~\mathit{eo}~{\mathsf{end}})^\ast \\
  &amp;\wedge&amp; (S'; F; {\mathit{data}}.{\mathsf{offset}} {\hookrightarrow}^\ast S'; F; {\mathsf{i32}}.{\mathsf{const}}~\mathit{do}~{\mathsf{end}})^\ast \\[1ex]
  &amp;\wedge&amp; (\mathit{eo} + |{\mathit{elem}}.{\mathsf{init}}| \leq |S'.{\mathsf{tables}}[{\mathit{tableaddr}}].{\mathsf{elem}}|)^\ast \\
  &amp;\wedge&amp; (\mathit{do} + |{\mathit{data}}.{\mathsf{init}}| \leq |S'.{\mathsf{mems}}[{\mathit{memaddr}}].{\mathsf{data}}|)^\ast
\\[1ex]
  &amp;\wedge&amp; ({\mathit{tableaddr}} = {\mathit{moduleinst}}.{\mathsf{tableaddrs}}[{\mathit{elem}}.{\mathsf{table}}])^\ast \\
  &amp;\wedge&amp; ({\mathit{memaddr}} = {\mathit{moduleinst}}.{\mathsf{memaddrs}}[{\mathit{data}}.{\mathsf{data}}])^\ast \\
  &amp;\wedge&amp; ({\mathit{funcaddr}} = {\mathit{moduleinst}}.{\mathsf{funcaddrs}}[{\mathit{start}}.{\mathsf{func}}])^?)
\\[2ex]
S; F; {\mathsf{init\_elem}}~a~i~\epsilon &amp;{\hookrightarrow}&amp;
  S; F; \epsilon \\
S; F; {\mathsf{init\_elem}}~a~i~(x_0~x^\ast) &amp;{\hookrightarrow}&amp;
  S'; F; {\mathsf{init\_elem}}~a~(i+1)~x^\ast \\ &amp;&amp;
  (\mathrel{\mbox{if}} S' = S {\mathrel{\mbox{with}}} {\mathsf{tables}}[a].{\mathsf{elem}}[i] = F.{\mathsf{module}}.{\mathsf{funcaddrs}}[x_0])
\\[1ex]
S; F; {\mathsf{init\_data}}~a~i~\epsilon &amp;{\hookrightarrow}&amp;
  S; F; \epsilon \\
S; F; {\mathsf{init\_data}}~a~i~(b_0~b^\ast) &amp;{\hookrightarrow}&amp;
  S'; F; {\mathsf{init\_data}}~a~(i+1)~b^\ast \\ &amp;&amp;
  (\mathrel{\mbox{if}} S' = S {\mathrel{\mbox{with}}} {\mathsf{mems}}[a].{\mathsf{data}}[i] = b_0)
\end{array}\end{split}\]</div>

### 付記

<div>グローバル初期化値<span>\({\mathit{val}}^\ast\)</span>はモジュールアロケータに渡されますが、アロケーションによって返されるストアS′とモジュールインスタンスmoduleinstに依存するため、モジュールのアロケーションとグローバル初期化子の評価は相互に再帰的です。しかし、この再帰はあくまでも仕様の工夫です。検証のため、初期化値は、初期ストアのグローバルイニシャライザを評価する単純なプリパスから容易に決定することができます。

ストアの観察可能な突然変異が起こる前に、すべての失敗条件がチェックされます。ストアの突然変異はアトミックではなく、他のスレッドとインタリーブされる可能性のある個々のステップで発生します。

定数式の評価はストアには影響しません。
</div>

## 呼び出し

<div>モジュールがインスタンス化されると、エクスポートされた関数は、ストアSの関数アドレスfuncaddrと引数値の適切なリスト<span>\({\mathit{val}}^\ast\)</span>を介して外部から呼び出すことができます。

引数が関数型に合わない場合、呼び出しはエラーで失敗することがあります。また、呼び出しがトラップになることもあります。そのような状態をどのように報告するかは、エンベッダーが定義します。
</div>

### 付記

呼び出しを実行する前に、エンベッダーAPIが静的または動的に型チェックを行った場合、トラップ以外の障害は発生しません。

---

以下の手順を実行します:

<ol>
  <li><p>前提条件：<span>\(S.{\mathsf{funcs}}[{\mathit{funcaddr}}]\)</span>は存在します。</p></li>
  <li><p><span>\({\mathit{funcinst}}\)</span>は関数インスタンス<span>\(S.{\mathsf{funcs}}[{\mathit{funcaddr}}]\)</span>であるとします。</p></li>
  <li><p><span>\([t_1^n] {\rightarrow} [t_2^m]\)</span>は関数型<span>\({\mathit{funcinst}}.{\mathsf{type}}\)</span>であるとします。</p></li>
  <li>もし与えられた引数の長さ<span>\(|{\mathit{val}}^\ast|\)</span>が期待される引数の長さの数値nと異なるならば:
    <ol>
      <li><p>失敗します。</p></li>
    </ol>
  </li>
  <li><span>\({\mathit{val}}^\ast\)</span>中の各<span>\(t_1^n\)</span>の値型<span>\(t_i\)</span>と対応する値<span>\(val_i\)</span>について:
    <ol>
      <li>もし<span>\({\mathit{val}}_i\)</span>が<span>\(c_i\)</span>について<span>\(t_i.{\mathsf{const}}~c_i\)</span>でないならば:
        <ol>
          <li><p>失敗します。</p></li>
        </ol>
      </li>
    </ol>
  </li>
  <li><p>Fはダミーフレーム<span>\(\{ {\mathsf{module}}~\{\}, {\mathsf{locals}}~\epsilon \}\)</span>であるとします。</p></li>
  <li><p>スタックにフレームFをpushします。</p></li>
  <li><p>スタックに値<span>\({\mathit{val}}^\ast\)</span>をpushします。</p></li>
  <li><p>アドレス<span>\({\mathit{funcaddr}}\)</span>にある関数インスタンスを呼び出します。</p></li>
</ol>

関数がreturnした場合、次のステップを実行します:

<ol>
  <li><p>前提条件：バリデーション/検証を経て保証されることですが、スタックの一番上にm valuesが存在します。</p></li>
  <li><p>スタックから<span>\({\mathit{val}}_{\mathrm{res}}^m\)</span>をpopします。</p></li>
</ol>

<div><span>\({\mathit{val}}_{\mathrm{res}}^m\)</span>の値が呼び出しの結果として戻り値になります。</div>

<div>\[\begin{split}~\\[-1ex]
\begin{array}{&#64;{}lcl}
{\mathrm{invoke}}(S, {\mathit{funcaddr}}, {\mathit{val}}^n) &amp;=&amp; S; F; {\mathit{val}}^n~({\mathsf{invoke}}~{\mathit{funcaddr}}) \\
  &amp;(\mathrel{\mbox{if}} &amp; S.{\mathsf{funcs}}[{\mathit{funcaddr}}].{\mathsf{type}} = [t_1^n] {\rightarrow} [t_2^m] \\
  &amp;\wedge&amp; {\mathit{val}}^n = (t_1.{\mathsf{const}}~c)^n \\
  &amp;\wedge&amp; F = \{ {\mathsf{module}}~\{\}, {\mathsf{locals}}~\epsilon \}) \\
\end{array}\end{split}\]</div>

# LINK

<footer>
    <nav>
        <ul>
          <li><p><a href="Validation" rel="prev">Prev: 検証</a></p></li>
          <li><p><a href="./">Top: Index</a></p></li>
          <li><p><a href="BinaryFormat" rel="next">Next: Binary Format</a></p></li>
        </ul>
        <a href="LICENSE" rel="license">LICENSE</a>
    </nav>
</footer>