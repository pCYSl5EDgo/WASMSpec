<script async="async" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">MathJax.Hub.Config({"TeX": {"MAXBUFFER": 30720}})</script>

# 表記上のお約束

検証(バリデーション)は、`WebAssembly`モジュールが適格で有効であるか否かを検証します。
適格で有効なモジュールのみをインスタンス化することができます。

有効性はモジュールの抽象構文とその内容に対する型システムによって定義されます。
抽象構文の各部分には、それに適用される制約を指定する型付け規則があります。すべての規則は、2つの等価な形式で与えられます。

<ol>
    <li>散文的表記法では、人間にわかりやすい直感的な形で意味を記述します。</li>
    <li>形式的表記法では、数学的な形式で規則を記述します。[^1]</li>
</ol>

### 付記

散文的表記法と形式的表記法は等価です。
この仕様書を読むためには形式表記法の理解は必要ありません。
形式的表記法の方がプログラミング言語の意味論で広く使われている記法です。
より簡潔な記述であり、数学的な証明を容易に行うことができます。

---

どちらの場合も、規則は宣言的な方法で定式化されます。
つまり、制約を定式化するだけで、アルゴリズムを定義することはありません。
この仕様に従った命令シーケンスの型チェックのための健全で完全なアルゴリズムの概要が[付録](Appendix)において提供されています。

## コンテキスト

個々の定義の有効性は、周囲のモジュールとスコープ内の定義に関する関連情報を収集するコンテキストに対して相対的に指定されます:

- 型：現在のモジュールで定義されている型のリスト。
- 関数：現在のモジュールで宣言された関数のリスト。
- テーブル: 現在のモジュールで宣言されたテーブルのリスト。
- メモリ： 現在のモジュールで宣言されたメモリのリスト。
- グローバル ： 現在のモジュールで宣言されているグローバルのリスト。
- ローカル：現在の関数で宣言されたローカルのリスト（パラメータを含む）で、値の型で表されます。
- ラベル：現在の位置からアクセス可能なラベルのスタック。
- 戻り値: 現在の関数の戻り値の型で、オプションの結果型として表現されます。

言い換えればコンテキストは各インデックス空間に適した型のシーケンスを含み、その空間で定義された各エントリを記述します。
ローカル、ラベル、戻り値の型は、関数本体の命令を検証するためにのみ使用されそれ以外の場所では空のままにされます。
ラベルスタックはコンテキストの内唯一mutableな部分です。これは命令シーケンスの検証の進行と共に変更されます。

より具体的に言うならば、コンテキストは抽象構文を持つレコード`C`として定義されます。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef2623#1{{}}\mathdef2623{(context)} &amp; C &amp;::=&amp;
  \begin{array}[t]{l&#64;{~}ll}
  \{ &amp; {\mathsf{types}} &amp; {\mathit{functype}}^\ast, \\
     &amp; {\mathsf{funcs}} &amp; {\mathit{functype}}^\ast, \\
     &amp; {\mathsf{tables}} &amp; {\mathit{tabletype}}^\ast, \\
     &amp; {\mathsf{mems}} &amp; {\mathit{memtype}}^\ast, \\
     &amp; {\mathsf{globals}} &amp; {\mathit{globaltype}}^\ast, \\
     &amp; {\mathsf{locals}} &amp; {\mathit{valtype}}^\ast, \\
     &amp; {\mathsf{labels}} &amp; {\mathit{resulttype}}^\ast, \\
     &amp; {\mathsf{return}} &amp; {\mathit{resulttype}}^? ~\} \\
  \end{array}
\end{array}\end{split}\]</div>

コンテキストを操作する際には、`C.field`という形式のフィールドアクセスに加えて、以下のような表記を採用しています。

<ul>
    <li>コンテキストを綴る際には、空のフィールドは省略します。</li>
    <li><span>\(C,\mathsf{field}\,A^\ast\)</span>は、<span>\(C\)</span>と同じコンテキストを表しますが、フィールド構成要素のシーケンスの前に要素<span>\(A^\ast\)</span>が付加されています。</li>
</ul>

### 付記

<div><span>\(C.{\mathsf{labels}}[i]\)</span>のようなインデックス記法を使用して、コンテキスト内のそれぞれのインデックス空間のインデックスを検索します。
コンテキスト拡張記法<span>\(C,\mathsf{field}\,A\)</span>は、主にラベルインデックスのような相対インデックス空間を局所的に拡張するために使用されます。</div>

したがってこの記法は新しい相対インデックス0を導入し、既存のインデックスをシフトしながらそれぞれのシーケンスの先頭に追加するように定義されています。

## 散文的表記法

有効性の検証は、抽象構文の各関連部分のスタイル化された規則によって指定されます。
規則は、フレーズが有効なときに定義される制約を記述するだけでなく、型でそれを分類します。
これらの規則を記述する際には、以下の規約が採用されています。

- フレーズAは、それぞれの規則によって表現されているすべての制約が満たされている場合に限り、"型Tで有効"であると言われます。Tの形は、Aが何であるかに依存します。
- 規則は暗黙のうちに、与えられたコンテキストCを前提としています。
- いくつかの場所ではこのコンテキストは局所的にコンテキストC′に拡張され、追加の項目を持ちます。拡張されたコンテキストで具現化された仮定の下では、次の文が適用されなければならないことを表現するために、"Under context C′, ... statement .... "という定式化が採用されています。

### 付記

例えば、Aが関数であれば、Tは関数型であり、グローバルなAであれば、Tはグローバル型である、などです。

## 形式的表記法

この節では、規則を正式に指定するための表記法について簡単に説明します。
興味のある読者のために、より詳細な紹介はそれぞれの教科書[^2]に掲載されています。

フレーズAがそれぞれの型Tを持つという命題は、A:Tと書かれます。
しかし、一般的には、型付けは文脈Cに依存しています。
これを明示的に表現するために使用される完全な形式は、Cでエンコードされた仮定の下でA:Tが保持されるという判定C⊢A:Tです。

形式的な型付け規則は、型システムを指定するための標準的なアプローチを使用し、それを演繹規則に変換します。
すべての規則は以下の一般的な形式を持っています。

<div>\[\frac{
  \mathit{premise}_1 \qquad \mathit{premise}_2 \qquad \dots \qquad \mathit{premise}_n
}{
  \mathit{conclusion}
}\]</div>

このような規則は、すべての前提条件が成立すれば、結論が成立するという意味合いでおおまかに読まれます。
いくつかの規則には前提条件がないものがあり、それらは結論が無条件に成立する公理です。
結論は常に判断C⊢A:Tであり、抽象構文の関連する構成要素Aごとに1つのそれぞれの規則があります。

### 付記

例えば、`i32.add`命令の型付け規則は公理として与えることができます。

<div>\[\frac{
}{
  C \vdash {\mathsf{i32}}.{\mathsf{add}} : [{\mathsf{i32}}~{\mathsf{i32}}] {\rightarrow} [{\mathsf{i32}}]
}\]</div>

<div>この命令は常に<span>\([{\mathsf{i32}}~{\mathsf{i32}}] {\rightarrow} [{\mathsf{i32}}]\)</span>型で有効です(2つのi32値を消費して1つの値を生成することを意味します)。</div>

`local.get`のような命令は次のように型付けすることができます。

<div>\[\frac{
  C.{\mathsf{locals}}[x] = t
}{
  C \vdash {\mathsf{local.get}}~x : [] {\rightarrow} [t]
}\]</div>

<div>ここでは前提条件は即時ローカルインデックスxがコンテキスト内に存在することを強制します。
命令はそれぞれの型tの値を生成します（値を消費しません）。
<span>\(C.{\mathsf{locals}}[x]\)</span>が存在しない場合、前提条件は保持されず、命令は型が正しくありません。</div>

最後に、構造化命令は再帰的な規則を必要とし、その前提はそれ自体が型付けの判断となります。

<div>\[\frac{
  C \vdash {\mathit{blocktype}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
  \qquad
  C,{\mathsf{label}}\,[t_2^\ast] \vdash {\mathit{instr}}^\ast : [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C \vdash {\mathsf{block}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

ブロック命令はそのボディ内の命令シーケンスが有効な場合にのみ有効となります。
さらに、戻り値型はブロックの注釈で記述された`blocktype`と一致しなければなりません。
そうであれば、ブロック命令はボディと同じ型を持ちます。
ボディの中には、対応する戻り値型に対応する追加ラベルが取得可能です。
これはコンテキスト`C`を前提とした追加ラベル情報で拡張することで表現されます。

# 型

ほとんどの型は普遍的に有効です。
しかし、リミット型には検証時にチェックする必要のある条件があります。
さらにその上、処理を容易にするために、ブロック型は素朴な関数型に変換されます。

## リミット

リミットは、与えられた範囲内で意味のある境界を持っていなければなりません。

<h3><span>\(\{ {\mathsf{min}}~n, {\mathsf{max}}~m^? \}\)</span></h3>

<ul class="simple">
    <li><span>\(n\)</span>は<span>\(k\)</span>より大きい必要があります。</li>
    <li><span>\(m^?\)</span>が空でないならば:
        <ul>
            <li><span>\(k\)</span>未満です。</li>
            <li><span>\(n\)</span>以上です。</li>
        </ul>
    </li>
    <li>以上の条件を満足する時、前提<span>\(k\)</span>の下でこのリミットは有効です。</li>
</ul>

<div>\[\frac{
  n \leq k
  \qquad
  (m \leq k)^?
  \qquad
  (n \leq m)^?
}{
  {\vdash} \{ {\mathsf{min}}~n, {\mathsf{max}}~m^? \} : k
}\]</div>

## ブロック型

ブロック型は2つの形式の内いずれか一方で表現できます。
いずれも次の規則に従うことでプレーンな関数型に変換されます。

<h3><span>\({\mathit{typeidx}}\)</span></h3>

- <div><span>\(C.{\mathsf{types}}[{\mathit{typeidx}}]\)</span>はコンテキストに定義されている必要があります。</div>
- <div>ブロック型は関数型<span>\(C.{\mathsf{types}}[{\mathit{typeidx}}]\)</span>として有効です。</div>

<div>\[\frac{
  C.{\mathsf{types}}[{\mathit{typeidx}}] = {\mathit{functype}}
}{
  C {\vdash} {\mathit{typeidx}} : {\mathit{functype}}
}\]</div>

<h3><span>\([{\mathit{valtype}}^?]\)</span></h3>

<ul>
    <li>ブロック型は関数型<span>\([] {\rightarrow} [{\mathit{valtype}}^?]\)</span>として有効です。</li>
</ul>

<div>\[\frac{
}{
  C {\vdash} [{\mathit{valtype}}^?] : [] {\rightarrow} [{\mathit{valtype}}^?]
}\]</div>

## 関数型

関数型は常に有効です。

<h3><span>\([t_1^n] {\rightarrow} [t_2^m]\)</span></h3>

<div>\[\frac{
}{
  {\vdash} [t_1^\ast] {\rightarrow} [t_2^\ast] \mathrel{\mbox{ok}}
}\]</div>

## テーブル型

<h3><span>\({\mathit{limits}}~{\mathit{elemtype}}\)</span></h3>

<ul>
    <li>リミット<span>\({\mathit{limits}}\)</span>は<span>\(2^{32}\)</span>を境界条件として与えられます。その上でリミットは有効でなくてはなりません。</li>
    <li>以上の条件を満足する時、これは有効です。</li>
</ul>

<div>\[\frac{
  {\vdash} {\mathit{limits}} : 2^{32}
}{
  {\vdash} {\mathit{limits}}~{\mathit{elemtype}} \mathrel{\mbox{ok}}
}\]</div>

## メモリ型

<h3><span>\({\mathit{limits}}\)</span></h3>

<ul>
    <li>リミット<span>\({\mathit{limits}}\)</span>は<span>\(2^{16}\)</span>を境界条件として与えられます。その上でリミットは有効でなくてはなりません。</li>
    <li>以上の条件を満足する時、これは有効です。</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{limits}} : 2^{16}
}{
  {\vdash} {\mathit{limits}} \mathrel{\mbox{ok}}
}\]</div>

<div>訳注:Threadでは<span>\({\mathit{share}}\)</span>が追加されています。</div>

## グローバル型

グローバル型は常に有効です。

<h3><span>\({\mathit{mut}}~{\mathit{valtype}}\)</span></h3>

<div>\[\frac{
}{
  {\vdash} {\mathit{mut}}~{\mathit{valtype}} \mathrel{\mbox{ok}}
}\]</div>


## 外部型

<h3><span>\({\mathsf{func}}~{\mathit{functype}}\)</span></h3>

<ul>
    <li>関数型<span>\({\mathit{functype}}\)</span>は有効でなくてはなりません。</li>
    <li>以上の条件を満足する時、これは有効です。</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{functype}} \mathrel{\mbox{ok}}
}{
  {\vdash} {\mathsf{func}}~{\mathit{functype}} \mathrel{\mbox{ok}}
}\]</div>

<h3><span>\({\mathsf{table}}~{\mathit{tabletype}}\)</span></h3>

<ul>
    <li>テーブル型<span>\({\mathit{tabletype}}\)</span>は有効でなくてはなりません。</li>
    <li>以上の条件を満足する時、これは有効です。</li>
</ul>

<div>\[\frac{
  {\vdash} {\mathit{tabletype}} \mathrel{\mbox{ok}}
}{
  {\vdash} {\mathsf{table}}~{\mathit{tabletype}} \mathrel{\mbox{ok}}
}\]</div>
</div>

<h3><span>\({\mathsf{mem}}~{\mathit{memtype}}\)</span></h3>

<ul>
    <li>メモリ型<span>\({\mathit{memtype}}\)</span>は有効でなくてはなりません。</li>
    <li>以上の条件を満足する時、これは有効です。</li>
</ul>

<div>\[\frac{
  {\vdash} {\mathit{memtype}} \mathrel{\mbox{ok}}
}{
  {\vdash} {\mathsf{mem}}~{\mathit{memtype}} \mathrel{\mbox{ok}}
}\]</div>

<h3><span>\({\mathsf{global}}~{\mathit{globaltype}}\)</span></h3>

<ul>
    <li>グローバル型<span>\({\mathit{globaltype}}\)</span>は有効でなくてはなりません。</li>
    <li>以上の条件を満足する時、これは有効です。</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{globaltype}} \mathrel{\mbox{ok}}
}{
  {\vdash} {\mathsf{global}}~{\mathit{globaltype}} \mathrel{\mbox{ok}}
}\]</div>

# 命令

<div>命令は、オペランドスタックをどのように操作するかを表す関数型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>で分類されます。
これらの型は、命令がポップアップオフする<span>\(t_1^\ast\)</span>型の引数値を持つ必要な入力スタックと、命令がプッシュバックする<span>\(t_2^\ast\)</span>型の結果値を持つ必要な出力スタックを記述しています。</div>

### 付記

<div>例えば<span>\(\mathsf{i32}.\mathsf{add}\)</span>は<span>\([{\mathsf{i32}}~{\mathsf{i32}}] {\rightarrow} [{\mathsf{i32}}]\)</span>型です。
2つの<span>\(\mathsf{i32}\)</span>をpopして1つpushします。</div>

---

型付けは命令シーケンス<span>\({\mathit{instr}}^\ast\)</span>にまで及びます。
このような命令シーケンスは関数型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> if the accumulative effect of executing the instructions is consuming values of types <span>\(t_1^\ast\)</span> off the operand stack and pushing new values of types <span>\(t_2^\ast\)</span>.
<p id="polymorphism">For some instructions, the typing rules do not fully constrain the type,
and therefore allow for multiple types.
Such instructions are called <em>polymorphic</em>.
Two degrees of polymorphism can be distinguished:
<ul>
    <li><em>value-polymorphic</em>:
the 値型<span>\(t\)</span> of one or several individual operands is unconstrained.
That is the case for all パラメトリック命令 like <span>\({\mathsf{drop}}\)</span> and <span>\({\mathsf{select}}\)</span>.</li>
    <li><em>stack-polymorphic</em>:
the entire (or most of the) 関数型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> of the instruction is unconstrained.
That is the case for all 制御命令 that perform an <em>unconditional control transfer</em>, such as <span>\({\mathsf{unreachable}}\)</span>, <span>\({\mathsf{br}}\)</span>, <span>\({\mathsf{br\_table}}\)</span>, and <span>\({\mathsf{return}}\)</span>.</li>
</ul>


### 付記

For example, the <span>\({\mathsf{select}}\)</span> instruction is valid with type <span>\([t~t~{\mathsf{i32}}] {\rightarrow} [t]\)</span>, for any possible 値型<span>\(t\)</span>.   Consequently, both instruction sequences
<div>\[({\mathsf{i32}}.{\mathsf{const}}~1)~~({\mathsf{i32}}.{\mathsf{const}}~2)~~({\mathsf{i32}}.{\mathsf{const}}~3)~~{\mathsf{select}}{}\]</div>
and
<div>\[({\mathsf{f64}}.{\mathsf{const}}~1.0)~~({\mathsf{f64}}.{\mathsf{const}}~2.0)~~({\mathsf{i32}}.{\mathsf{const}}~3)~~{\mathsf{select}}{}\]</div>
are valid, with <span>\(t\)</span> in the typing of <span>\({\mathsf{select}}\)</span> being instantiated to <span>\({\mathsf{i32}}\)</span> or <span>\({\mathsf{f64}}\)</span>, respectively.
The <span>\({\mathsf{unreachable}}\)</span> instruction is valid with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> for any possible sequences of value types <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>.
Consequently,
<div>\[{\mathsf{unreachable}}~~{\mathsf{i32}}.{\mathsf{add}}\]</div>
is valid by assuming type <span>\([] {\rightarrow} [{\mathsf{i32}}~{\mathsf{i32}}]\)</span> for the <span>\({\mathsf{unreachable}}\)</span> instruction.
In contrast,
<div>\[{\mathsf{unreachable}}~~({\mathsf{i64}}.{\mathsf{const}}~0)~~{\mathsf{i32}}.{\mathsf{add}}\]</div>
is invalid, because there is no possible type to pick for the <span>\({\mathsf{unreachable}}\)</span> instruction that would make the sequence well-typed.


## 算術演算命令

<h3><span>\(t\mathsf{.}{\mathsf{const}}~c\)</span></h3>

<ul>
    <li>The instruction is valid with type <span>\([] {\rightarrow} [t]\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t\mathsf{.}{\mathsf{const}}~c : [] {\rightarrow} [t]
}\]</div>



<h3><span>\(t\mathsf{.}{\mathit{unop}}\)</span></h3>

<ul>
    <li>The instruction is valid with type <span>\([t] {\rightarrow} [t]\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t\mathsf{.}{\mathit{unop}} : [t] {\rightarrow} [t]
}\]</div>




<h3><span>\(t\mathsf{.}{\mathit{binop}}\)</span></h3>

<ul>
    <li>The instruction is valid with type <span>\([t~t] {\rightarrow} [t]\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t\mathsf{.}{\mathit{binop}} : [t~t] {\rightarrow} [t]
}\]</div>


<h3><span>\(t\mathsf{.}{\mathit{testop}}\)</span></h3>

<ul>
    <li>The instruction is valid with type <span>\([t] {\rightarrow} [{\mathsf{i32}}]\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t\mathsf{.}{\mathit{testop}} : [t] {\rightarrow} [{\mathsf{i32}}]
}\]</div>


<h3><span>\(t\mathsf{.}{\mathit{relop}}\)</span></h3>

<ul>
    <li>The instruction is valid with type <span>\([t~t] {\rightarrow} [{\mathsf{i32}}]\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t\mathsf{.}{\mathit{relop}} : [t~t] {\rightarrow} [{\mathsf{i32}}]
}\]</div>

<h3><span>\(t_2\mathsf{.}{\mathit{cvtop}}\mathsf{\_}t_1\mathsf{\_}{\mathit{sx}}^?\)</span></h3>

<ul>
    <li>The instruction is valid with type <span>\([t_1] {\rightarrow} [t_2]\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t_2\mathsf{.}{\mathit{cvtop}}\mathsf{\_}t_1\mathsf{\_}{\mathit{sx}}^? : [t_1] {\rightarrow} [t_2]
}\]</div>


## パラメトリック命令


<h3><span>\({\mathsf{drop}}\)</span></h3>

<ul>
    <li>The instruction is valid with type <span>\([t] {\rightarrow} []\)</span>, for any 値型<span>\(t\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} {\mathsf{drop}} : [t] {\rightarrow} []
}\]</div>

<h3><span>\({\mathsf{select}}\)</span></h3>

<ul>
    <li>The instruction is valid with type <span>\([t~t~{\mathsf{i32}}] {\rightarrow} [t]\)</span>, for any 値型<span>\(t\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} {\mathsf{select}} : [t~t~{\mathsf{i32}}] {\rightarrow} [t]
}\]</div>

### 付記

Both <span>\({\mathsf{drop}}\)</span> and <span>\({\mathsf{select}}\)</span> are <a class="reference internal" href="#polymorphism"><span class="std std-ref">value-polymorphic</span></a> instructions.


## 変数命令

<h3><span>\({\mathsf{local.get}}~x\)</span></h3>

<ul>
    <li>The local <span>\(C.{\mathsf{locals}}[x]\)</span> must be defined in the context.</li>
    <li>Let <span>\(t\)</span> be the 値型<span>\(C.{\mathsf{locals}}[x]\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([] {\rightarrow} [t]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{locals}}[x] = t
}{
  C {\vdash} {\mathsf{local.get}}~x : [] {\rightarrow} [t]
}\]</div>


<h3><span>\({\mathsf{local.set}}~x\)</span></h3>

<ul>
    <li>The local <span>\(C.{\mathsf{locals}}[x]\)</span> must be defined in the context.</li>
    <li>Let <span>\(t\)</span> be the 値型<span>\(C.{\mathsf{locals}}[x]\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t] {\rightarrow} []\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{locals}}[x] = t
}{
  C {\vdash} {\mathsf{local.set}}~x : [t] {\rightarrow} []
}\]</div>



<h3><span>\({\mathsf{local.tee}}~x\)</span></h3>

<ul>
    <li>The local <span>\(C.{\mathsf{locals}}[x]\)</span> must be defined in the context.</li>
    <li>Let <span>\(t\)</span> be the 値型<span>\(C.{\mathsf{locals}}[x]\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t] {\rightarrow} [t]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{locals}}[x] = t
}{
  C {\vdash} {\mathsf{local.tee}}~x : [t] {\rightarrow} [t]
}\]</div>


<h3><span>\({\mathsf{global.get}}~x\)</span></h3>

<ul>
    <li>The global <span>\(C.{\mathsf{globals}}[x]\)</span> must be defined in the context.</li>
    <li>Let <span>\({\mathit{mut}}~t\)</span> be the グローバル型 <span>\(C.{\mathsf{globals}}[x]\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([] {\rightarrow} [t]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{globals}}[x] = {\mathit{mut}}~t
}{
  C {\vdash} {\mathsf{global.get}}~x : [] {\rightarrow} [t]
}\]</div>


<h3><span>\({\mathsf{global.set}}~x\)</span></h3>

<ul>
    <li>The global <span>\(C.{\mathsf{globals}}[x]\)</span> must be defined in the context.</li>
    <li>Let <span>\({\mathit{mut}}~t\)</span> be the グローバル型 <span>\(C.{\mathsf{globals}}[x]\)</span>.</li>
    <li>The mutability <span>\({\mathit{mut}}\)</span> must be <span>\({\mathsf{var}}\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t] {\rightarrow} []\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{globals}}[x] = {\mathsf{var}}~t
}{
  C {\vdash} {\mathsf{global.set}}~x : [t] {\rightarrow} []
}\]</div>


## メモリ命令


<h3><span>\(t\mathsf{.}{\mathsf{load}}~{\mathit{memarg}}\)</span></h3>

<ul>
    <li>The memory <span>\(C.{\mathsf{mems}}[0]\)</span> must be defined in the context.</li>
    <li>The alignment <span>\(2^{{\mathit{memarg}}.{\mathsf{align}}}\)</span> must not be larger than the bit幅 of <span>\(t\)</span> divided by <span>\(8\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([{\mathsf{i32}}] {\rightarrow} [t]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[0] = {\mathit{memtype}}
  \qquad
  2^{{\mathit{memarg}}.{\mathsf{align}}} \leq |t|/8
}{
  C {\vdash} t\mathsf{.load}~{\mathit{memarg}} : [{\mathsf{i32}}] {\rightarrow} [t]
}\]</div>


<h3><span>\(t\mathsf{.}{\mathsf{load}}{N}\mathsf{\_}{\mathit{sx}}~{\mathit{memarg}}\)</span></h3>

<ul>
    <li>The memory <span>\(C.{\mathsf{mems}}[0]\)</span> must be defined in the context.</li>
    <li>The alignment <span>\(2^{{\mathit{memarg}}.{\mathsf{align}}}\)</span> must not be larger than <span>\(N/8\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([{\mathsf{i32}}] {\rightarrow} [t]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[0] = {\mathit{memtype}}
  \qquad
  2^{{\mathit{memarg}}.{\mathsf{align}}} \leq N/8
}{
  C {\vdash} t\mathsf{.load}N\mathsf{\_}{\mathit{sx}}~{\mathit{memarg}} : [{\mathsf{i32}}] {\rightarrow} [t]
}\]</div>


<h3><span>\(t\mathsf{.}{\mathsf{store}}~{\mathit{memarg}}\)</span></h3>

<ul>
    <li>The memory <span>\(C.{\mathsf{mems}}[0]\)</span> must be defined in the context.</li>
    <li>The alignment <span>\(2^{{\mathit{memarg}}.{\mathsf{align}}}\)</span> must not be larger than the bit幅 of <span>\(t\)</span> divided by <span>\(8\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([{\mathsf{i32}}~t] {\rightarrow} []\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[0] = {\mathit{memtype}}
  \qquad
  2^{{\mathit{memarg}}.{\mathsf{align}}} \leq |t|/8
}{
  C {\vdash} t\mathsf{.store}~{\mathit{memarg}} : [{\mathsf{i32}}~t] {\rightarrow} []
}\]</div>



<h3><span>\(t\mathsf{.}{\mathsf{store}}{N}~{\mathit{memarg}}\)</span></h3>

<ul>
    <li>The memory <span>\(C.{\mathsf{mems}}[0]\)</span> must be defined in the context.</li>
    <li>The alignment <span>\(2^{{\mathit{memarg}}.{\mathsf{align}}}\)</span> must not be larger than <span>\(N/8\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([{\mathsf{i32}}~t] {\rightarrow} []\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[0] = {\mathit{memtype}}
  \qquad
  2^{{\mathit{memarg}}.{\mathsf{align}}} \leq N/8
}{
  C {\vdash} t\mathsf{.store}N~{\mathit{memarg}} : [{\mathsf{i32}}~t] {\rightarrow} []
}\]</div>

<h3><span>\({\mathsf{memory.size}}\)</span></h3>

<ul>
    <li>The memory <span>\(C.{\mathsf{mems}}[0]\)</span> must be defined in the context.</li>
    <li>Then the instruction is valid with type <span>\([] {\rightarrow} [{\mathsf{i32}}]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[0] = {\mathit{memtype}}
}{
  C {\vdash} {\mathsf{memory.size}} : [] {\rightarrow} [{\mathsf{i32}}]
}\]</div>


<h3><span>\({\mathsf{memory.grow}}\)</span></h3>

<ul>
    <li>The memory <span>\(C.{\mathsf{mems}}[0]\)</span> must be defined in the context.</li>
    <li>Then the instruction is valid with type <span>\([{\mathsf{i32}}] {\rightarrow} [{\mathsf{i32}}]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[0] = {\mathit{memtype}}
}{
  C {\vdash} {\mathsf{memory.grow}} : [{\mathsf{i32}}] {\rightarrow} [{\mathsf{i32}}]
}\]</div>


## 制御命令


<h3><span>\({\mathsf{nop}}\)</span></h3>

<ul>
    <li>The instruction is valid with type <span>\([] {\rightarrow} []\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} {\mathsf{nop}} : [] {\rightarrow} []
}\]</div>
</div>


<h3><span>\({\mathsf{unreachable}}\)</span></h3>

<ul>
    <li>The instruction is valid with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>, for any sequences of 値型 <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} {\mathsf{unreachable}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

### 付記

<div><span>\({\mathsf{unreachable}}\)</span></div>

---

<h3><span>\({\mathsf{block}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}}\)</span></h3>

<ul>
    <li>The ブロック型 must be 有効 as some 関数型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
    <li>Let <span>\(C'\)</span> be the same コンテキスト as <span>\(C\)</span>, but with the 戻り値型 <span>\([t_2^\ast]\)</span> prepended to the <span>\({\mathsf{labels}}\)</span> vector.</li>
    <li>Under context <span>\(C'\)</span>,
the instruction sequence <span>\({\mathit{instr}}^\ast\)</span> must be 有効 with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
    <li>Then the compound instruction is valid with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{blocktype}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
  \qquad
  C,{\mathsf{labels}}\,[t_2^\ast] {\vdash} {\mathit{instr}}^\ast : [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C {\vdash} {\mathsf{block}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

### 付記

The <a class="reference internal" href="conventions.html#notation-extend"><span class="std std-ref">notation</span></a> <span>\(C,{\mathsf{labels}}\,[t^\ast]\)</span> inserts the new label type at index <span>\(0\)</span>, shifting all others.

---

<h3><span>\({\mathsf{loop}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}}\)</span></h3>

<ul>
    <li>The ブロック型 must be 有効 as some 関数型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
    <li>Let <span>\(C'\)</span> be the same コンテキスト as <span>\(C\)</span>, but with the 戻り値型 <span>\([t_1^\ast]\)</span> prepended to the <span>\({\mathsf{labels}}\)</span> vector.</li>
    <li>Under context <span>\(C'\)</span>,
the instruction sequence <span>\({\mathit{instr}}^\ast\)</span> must be 有効 with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
    <li>Then the compound instruction is valid with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{blocktype}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
  \qquad
  C,{\mathsf{labels}}\,[t_1^\ast] {\vdash} {\mathit{instr}}^\ast : [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C {\vdash} {\mathsf{loop}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>


### 付記

The <a class="reference internal" href="conventions.html#notation-extend"><span class="std std-ref">notation</span></a> <span>\(C,{\mathsf{labels}}\,[t^\ast]\)</span> inserts the new label type at index <span>\(0\)</span>, shifting all others.

---

<h3><span>\({\mathsf{if}}~{\mathit{blocktype}}~{\mathit{instr}}_1^\ast~{\mathsf{else}}~{\mathit{instr}}_2^\ast~{\mathsf{end}}\)</span></h3>

<ul>
    <li>The ブロック型 must be 有効 as some 関数型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
    <li>Let <span>\(C'\)</span> be the same コンテキスト as <span>\(C\)</span>, but with the 戻り値型 <span>\([t_2^\ast]\)</span> prepended to the <span>\({\mathsf{labels}}\)</span> vector.</li>
    <li>Under context <span>\(C'\)</span>,
the instruction sequence <span>\({\mathit{instr}}_1^\ast\)</span> must be 有効 with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
    <li>Under context <span>\(C'\)</span>,
the instruction sequence <span>\({\mathit{instr}}_2^\ast\)</span> must be 有効 with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
    <li>Then the compound instruction is valid with type <span>\([t_1^\ast~{\mathsf{i32}}] {\rightarrow} [t_2^\ast]\)</span>.</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{blocktype}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
  \qquad
  C,{\mathsf{labels}}\,[t_2^\ast] {\vdash} {\mathit{instr}}_1^\ast : [t_1^\ast] {\rightarrow} [t_2^\ast]
  \qquad
  C,{\mathsf{labels}}\,[t_2^\ast] {\vdash} {\mathit{instr}}_2^\ast : [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C {\vdash} {\mathsf{if}}~{\mathit{blocktype}}~{\mathit{instr}}_1^\ast~{\mathsf{else}}~{\mathit{instr}}_2^\ast~{\mathsf{end}} : [t_1^\ast~{\mathsf{i32}}] {\rightarrow} [t_2^\ast]
}\]</div>

### 付記

The <a class="reference internal" href="conventions.html#notation-extend"><span class="std std-ref">notation</span></a> <span>\(C,{\mathsf{labels}}\,[t^\ast]\)</span> inserts the new label type at index <span>\(0\)</span>, shifting all others.

---

<h3><span>\({\mathsf{br}}~l\)</span></h3>

<ul>
    <li>The label <span>\(C.{\mathsf{labels}}[l]\)</span> must be defined in the context.</li>
    <li>Let <span>\([t^\ast]\)</span> be the 戻り値型 <span>\(C.{\mathsf{labels}}[l]\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]\)</span>, for any sequences of 値型 <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{labels}}[l] = [t^\ast]
}{
  C {\vdash} {\mathsf{br}}~l : [t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

### 付記

The ラベルインデックス space in the コンテキスト <span>\(C\)</span> contains the most recent label first, so that <span>\(C.{\mathsf{labels}}[l]\)</span> performs a relative lookup as expected.
The <span>\({\mathsf{br}}\)</span> instruction is <a class="reference internal" href="#polymorphism"><span class="std std-ref">stack-polymorphic</span></a>.

---

<h3><span>\({\mathsf{br\_if}}~l\)</span></h3>

<ul>
    <li>The label <span>\(C.{\mathsf{labels}}[l]\)</span> must be defined in the context.</li>
    <li>Let <span>\([t^\ast]\)</span> be the 戻り値型 <span>\(C.{\mathsf{labels}}[l]\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t^\ast~{\mathsf{i32}}] {\rightarrow} [t^\ast]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{labels}}[l] = [t^\ast]
}{
  C {\vdash} {\mathsf{br\_if}}~l : [t^\ast~{\mathsf{i32}}] {\rightarrow} [t^\ast]
}\]</div>

### 付記

The ラベルインデックス space in the コンテキスト <span>\(C\)</span> contains the most recent label first, so that <span>\(C.{\mathsf{labels}}[l]\)</span> performs a relative lookup as expected.

---

<h3><span>\({\mathsf{br\_table}}~l^\ast~l_N\)</span></h3>

<ul>
    <li>The label <span>\(C.{\mathsf{labels}}[l_N]\)</span> must be defined in the context.</li>
    <li>Let <span>\([t^\ast]\)</span> be the 戻り値型 <span>\(C.{\mathsf{labels}}[l_N]\)</span>.</li>
    <li>For all <span>\(l_i\)</span> in <span>\(l^\ast\)</span>,
the label <span>\(C.{\mathsf{labels}}[l_i]\)</span> must be defined in the context.</li>
    <li>For all <span>\(l_i\)</span> in <span>\(l^\ast\)</span>,
<span>\(C.{\mathsf{labels}}[l_i]\)</span> must be <span>\([t^\ast]\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t_1^\ast~t^\ast~{\mathsf{i32}}] {\rightarrow} [t_2^\ast]\)</span>, for any sequences of 値型 <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>.</li>
</ul>
<div>\[\frac{
  (C.{\mathsf{labels}}[l] = [t^\ast])^\ast
  \qquad
  C.{\mathsf{labels}}[l_N] = [t^\ast]
}{
  C {\vdash} {\mathsf{br\_table}}~l^\ast~l_N : [t_1^\ast~t^\ast~{\mathsf{i32}}] {\rightarrow} [t_2^\ast]
}\]</div>

### 付記

The ラベルインデックス space in the コンテキスト <span>\(C\)</span> contains the most recent label first, so that <span>\(C.{\mathsf{labels}}[l_i]\)</span> performs a relative lookup as expected.
The <span>\({\mathsf{br\_table}}\)</span> instruction is <a class="reference internal" href="#polymorphism"><span class="std std-ref">stack-polymorphic</span></a>.

---

<h3><span>\({\mathsf{return}}\)</span></h3>

<ul>
    <li>The return type <span>\(C.{\mathsf{return}}\)</span> must not be absent in the context.</li>
    <li>Let <span>\([t^\ast]\)</span> be the 戻り値型 of <span>\(C.{\mathsf{return}}\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]\)</span>, for any sequences of 値型 <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{return}} = [t^\ast]
}{
  C {\vdash} {\mathsf{return}} : [t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

### 付記

The <span>\({\mathsf{return}}\)</span> instruction is <a class="reference internal" href="#polymorphism"><span class="std std-ref">stack-polymorphic</span></a>.
<span>\(C.{\mathsf{return}}\)</span> is absent (set to <span>\(\epsilon\)</span>) when validating an 式 that is not a function body.
This differs from it being set to the empty result type (<span>\([\epsilon]\)</span>),
which is the case for functions not returning anything.

---

<h3><span>\({\mathsf{call}}~x\)</span></h3>

<ul>
    <li>The function <span>\(C.{\mathsf{funcs}}[x]\)</span> must be defined in the context.</li>
    <li>Then the instruction is valid with type <span>\(C.{\mathsf{funcs}}[x]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{funcs}}[x] = [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C {\vdash} {\mathsf{call}}~x : [t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

<h3><span>\({\mathsf{call\_indirect}}~x\)</span></h3>

<ul>
    <li>The table <span>\(C.{\mathsf{tables}}[0]\)</span> must be defined in the context.</li>
    <li>Let <span>\({\mathit{limits}}~{\mathit{elemtype}}\)</span> be the テーブル型 <span>\(C.{\mathsf{tables}}[0]\)</span>.</li>
    <li>The 要素型 <span>\({\mathit{elemtype}}\)</span> must be <span>\({\mathsf{funcref}}\)</span>.</li>
    <li>The type <span>\(C.{\mathsf{types}}[x]\)</span> must be defined in the context.</li>
    <li>Let <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> be the 関数型<span>\(C.{\mathsf{types}}[x]\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t_1^\ast~{\mathsf{i32}}] {\rightarrow} [t_2^\ast]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{tables}}[0] = {\mathit{limits}}~{\mathsf{funcref}}
  \qquad
  C.{\mathsf{types}}[x] = [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C {\vdash} {\mathsf{call\_indirect}}~x : [t_1^\ast~{\mathsf{i32}}] {\rightarrow} [t_2^\ast]
}\]</div>

## 命令シーケンス

<h3>空の命令シーケンス: <span>\(\epsilon\)</span></h3>

<ul>
    <li>The empty instruction sequence is valid with type <span>\([t^\ast] {\rightarrow} [t^\ast]\)</span>, for any sequence of 値型 <span>\(t^\ast\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} \epsilon : [t^\ast] {\rightarrow} [t^\ast]
}\]</div>

<h3>空でない命令シーケンス: <span>\({\mathit{instr}}^\ast~{\mathit{instr}}_N\)</span></h3>

<ul>
    <li>The instruction sequence <span>\({\mathit{instr}}^\ast\)</span> must be valid with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>, for some sequences of 値型 <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>.</li>
    <li>The instruction <span>\({\mathit{instr}}_N\)</span> must be valid with type <span>\([t^\ast] {\rightarrow} [t_3^\ast]\)</span>, for some sequences of 値型 <span>\(t^\ast\)</span> and <span>\(t_3^\ast\)</span>.</li>
    <li>There must be a sequence of 値型 <span>\(t_0^\ast\)</span>, such that <span>\(t_2^\ast = t_0^\ast~t^\ast\)</span>.</li>
    <li>Then the combined instruction sequence is valid with type <span>\([t_1^\ast] {\rightarrow} [t_0^\ast~t_3^\ast]\)</span>.</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{instr}}^\ast : [t_1^\ast] {\rightarrow} [t_0^\ast~t^\ast]
  \qquad
  C {\vdash} {\mathit{instr}}_N : [t^\ast] {\rightarrow} [t_3^\ast]
}{
  C {\vdash} {\mathit{instr}}^\ast~{\mathit{instr}}_N : [t_1^\ast] {\rightarrow} [t_0^\ast~t_3^\ast]
}\]</div>

## 式

<div><span>\([t^\ast]\)</span></div>

<h3><span>\({\mathit{instr}}^\ast~{\mathsf{end}}\)</span></h3>

<ul>
    <li>The instruction sequence <span>\({\mathit{instr}}^\ast\)</span> must be 有効 with type <span>\([] {\rightarrow} [t^\ast]\)</span>, for some 戻り値型 <span>\([t^\ast]\)</span>.</li>
    <li>Then the expression is valid with 戻り値型 <span>\([t^\ast]\)</span>.</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{instr}}^\ast : [] {\rightarrow} [t^\ast]
}{
  C {\vdash} {\mathit{instr}}^\ast~{\mathsf{end}} : [t^\ast]
}\]</div>

### 定数式

<ul>
    <li>In a <em>constant</em> expression <span>\({\mathit{instr}}^\ast~{\mathsf{end}}\)</span> all instructions in <span>\({\mathit{instr}}^\ast\)</span> must be constant.</li>
    <li>A constant instruction <span>\({\mathit{instr}}\)</span> must be:
<ul>
    <li>either of the form <span>\(t.{\mathsf{const}}~c\)</span>,</li>
    <li>or of the form <span>\({\mathsf{global.get}}~x\)</span>, in which case <span>\(C.{\mathsf{globals}}[x]\)</span> must be a グローバル型 of the form <span>\({\mathsf{const}}~t\)</span>.</li>
</ul>
</li>
</ul>
<div>\[\frac{
  (C {\vdash} {\mathit{instr}} {\mathrel{\mbox{const}}})^\ast
}{
  C {\vdash} {\mathit{instr}}^\ast~{\mathsf{end}} {\mathrel{\mbox{const}}}
}\]</div>
<div>\[\frac{
}{
  C {\vdash} t.{\mathsf{const}}~c {\mathrel{\mbox{const}}}
}
\qquad
\frac{
  C.{\mathsf{globals}}[x] = {\mathsf{const}}~t
}{
  C {\vdash} {\mathsf{global.get}}~x {\mathrel{\mbox{const}}}
}\]</div>

### 付記


# モジュール

## 関数

<h3><span>\(\{ {\mathsf{type}}~x, {\mathsf{locals}}~t^\ast, {\mathsf{body}}~{\mathit{expr}} \}\)</span></h3>
<ul>
    <li>The type <span>\(C.{\mathsf{types}}[x]\)</span> must be defined in the context.</li>
    <li>Let <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> be the 関数型<span>\(C.{\mathsf{types}}[x]\)</span>.</li>
    <li>Let <span>\(C'\)</span> be the same コンテキスト as <span>\(C\)</span>,
but with:
<ul>
    <li><span>\({\mathsf{locals}}\)</span> set to the sequence of 値型 <span>\(t_1^\ast~t^\ast\)</span>, concatenating parameters and locals,</li>
    <li><span>\({\mathsf{labels}}\)</span> set to the singular sequence containing only 戻り値型 <span>\([t_2^\ast]\)</span>.</li>
    <li><span>\({\mathsf{return}}\)</span> set to the 戻り値型 <span>\([t_2^\ast]\)</span>.</li>
</ul>
</li>
    <li>Under the context <span>\(C'\)</span>,
the expression <span>\({\mathit{expr}}\)</span> must be valid with type <span>\([t_2^\ast]\)</span>.</li>
    <li>Then the function definition is valid with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{types}}[x] = [t_1^\ast] {\rightarrow} [t_2^\ast]
  \qquad
  C,{\mathsf{locals}}\,t_1^\ast~t^\ast,{\mathsf{labels}}~[t_2^\ast],{\mathsf{return}}~[t_2^\ast] {\vdash} {\mathit{expr}} : [t_2^\ast]
}{
  C {\vdash} \{ {\mathsf{type}}~x, {\mathsf{locals}}~t^\ast, {\mathsf{body}}~{\mathit{expr}} \} : [t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

## テーブル

<h3><span>\(\{ {\mathsf{type}}~{\mathit{tabletype}} \}\)</span></h3>
<ul>
    <li>The テーブル型 <span>\({\mathit{tabletype}}\)</span> must be 有効.</li>
    <li>Then the table definition is valid with type <span>\({\mathit{tabletype}}\)</span>.</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{tabletype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} \{ {\mathsf{type}}~{\mathit{tabletype}} \} : {\mathit{tabletype}}
}\]</div>

## メモリ

<h3><span>\(\{ {\mathsf{type}}~{\mathit{memtype}} \}\)</span></h3>
<ul>
    <li>The メモリ型 <span>\({\mathit{memtype}}\)</span> must be 有効.</li>
    <li>Then the memory definition is valid with type <span>\({\mathit{memtype}}\)</span>.</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{memtype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} \{ {\mathsf{type}}~{\mathit{memtype}} \} : {\mathit{memtype}}
}\]</div>

## グローバル

<h3><span>\(\{ {\mathsf{type}}~{\mathit{mut}}~t, {\mathsf{init}}~{\mathit{expr}} \}\)</span></h3>
<ul>
    <li>The グローバル型 <span>\({\mathit{mut}}~t\)</span> must be 有効.</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 有効 with 戻り値型 <span>\([t]\)</span>.</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 定数.</li>
    <li>Then the global definition is valid with type <span>\({\mathit{mut}}~t\)</span>.</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{mut}}~t \mathrel{\mbox{ok}}
  \qquad
  C {\vdash} {\mathit{expr}} : [t]
  \qquad
  C {\vdash} {\mathit{expr}} {\mathrel{\mbox{const}}}
}{
  C {\vdash} \{ {\mathsf{type}}~{\mathit{mut}}~t, {\mathsf{init}}~{\mathit{expr}} \} : {\mathit{mut}}~t
}\]</div>

## Elementセグメント

<h3><span>\(\{ {\mathsf{table}}~x, {\mathsf{offset}}~{\mathit{expr}}, {\mathsf{init}}~y^\ast \}\)</span></h3>
<ul>
    <li>The table <span>\(C.{\mathsf{tables}}[x]\)</span> must be defined in the context.</li>
    <li>Let <span>\({\mathit{limits}}~{\mathit{elemtype}}\)</span> be the テーブル型 <span>\(C.{\mathsf{tables}}[x]\)</span>.</li>
    <li>The 要素型 <span>\({\mathit{elemtype}}\)</span> must be <span>\({\mathsf{funcref}}\)</span>.</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 有効 with 戻り値型 <span>\([{\mathsf{i32}}]\)</span>.</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 定数.</li>
    <li>For each <span>\(y_i\)</span> in <span>\(y^\ast\)</span>,
the function <span>\(C.{\mathsf{funcs}}[y]\)</span> must be defined in the context.</li>
    <li>Then the element segment is valid.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{tables}}[x] = {\mathit{limits}}~{\mathsf{funcref}}
  \qquad
  C {\vdash} {\mathit{expr}} : [{\mathsf{i32}}]
  \qquad
  C {\vdash} {\mathit{expr}} {\mathrel{\mbox{const}}}
  \qquad
  (C.{\mathsf{funcs}}[y] = {\mathit{functype}})^\ast
}{
  C {\vdash} \{ {\mathsf{table}}~x, {\mathsf{offset}}~{\mathit{expr}}, {\mathsf{init}}~y^\ast \} \mathrel{\mbox{ok}}
}\]</div>

## Dataセグメント

<h3><span>\(\{ {\mathsf{data}}~x, {\mathsf{offset}}~{\mathit{expr}}, {\mathsf{init}}~b^\ast \}\)</span></h3>
<ul>
    <li>The memory <span>\(C.{\mathsf{mems}}[x]\)</span> must be defined in the context.</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 有効 with 戻り値型 <span>\([{\mathsf{i32}}]\)</span>.</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 定数.</li>
    <li>Then the data segment is valid.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[x] = {\mathit{limits}}
  \qquad
  C {\vdash} {\mathit{expr}} : [{\mathsf{i32}}]
  \qquad
  C {\vdash} {\mathit{expr}} {\mathrel{\mbox{const}}}
}{
  C {\vdash} \{ {\mathsf{data}}~x, {\mathsf{offset}}~{\mathit{expr}}, {\mathsf{init}}~b^\ast \} \mathrel{\mbox{ok}}
}\]</div>

## 開始関数

<h3><span>\(\{ {\mathsf{func}}~x \}\)</span></h3>
<ul>
    <li>The function <span>\(C.{\mathsf{funcs}}[x]\)</span> must be defined in the context.</li>
    <li>The type of <span>\(C.{\mathsf{funcs}}[x]\)</span> must be <span>\([] {\rightarrow} []\)</span>.</li>
    <li>Then the start function is valid.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{funcs}}[x] = [] {\rightarrow} []
}{
  C {\vdash} \{ {\mathsf{func}}~x \} \mathrel{\mbox{ok}}
}\]</div>

## Export

<h3><span>\(\{ {\mathsf{name}}~{\mathit{name}}, {\mathsf{desc}}~{\mathit{exportdesc}} \}\)</span></h3>
<ul>
    <li>The export description <span>\({\mathit{exportdesc}}\)</span> must be valid with 外部型 <span>\({\mathit{externtype}}\)</span>.</li>
    <li>Then the export is valid with 外部型 <span>\({\mathit{externtype}}\)</span>.</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{exportdesc}} : {\mathit{externtype}}
}{
  C {\vdash} \{ {\mathsf{name}}~{\mathit{name}}, {\mathsf{desc}}~{\mathit{exportdesc}} \} : {\mathit{externtype}}
}\]</div>

<h3><span>\({\mathsf{func}}~x\)</span></h3>
<ul>
    <li>The function <span>\(C.{\mathsf{funcs}}[x]\)</span> must be defined in the context.</li>
    <li>Then the export description is valid with 外部型 <span>\({\mathsf{func}}~C.{\mathsf{funcs}}[x]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{funcs}}[x] = {\mathit{functype}}
}{
  C {\vdash} {\mathsf{func}}~x : {\mathsf{func}}~{\mathit{functype}}
}\]</div>

<h3><span>\({\mathsf{table}}~x\)</span></h3>
<ul>
    <li>The table <span>\(C.{\mathsf{tables}}[x]\)</span> must be defined in the context.</li>
    <li>Then the export description is valid with 外部型 <span>\({\mathsf{table}}~C.{\mathsf{tables}}[x]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{tables}}[x] = {\mathit{tabletype}}
}{
  C {\vdash} {\mathsf{table}}~x : {\mathsf{table}}~{\mathit{tabletype}}
}\]</div>

<h3><span>\({\mathsf{mem}}~x\)</span></h3>
<ul>
    <li>The memory <span>\(C.{\mathsf{mems}}[x]\)</span> must be defined in the context.</li>
    <li>Then the export description is valid with 外部型 <span>\({\mathsf{mem}}~C.{\mathsf{mems}}[x]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[x] = {\mathit{memtype}}
}{
  C {\vdash} {\mathsf{mem}}~x : {\mathsf{mem}}~{\mathit{memtype}}
}\]</div>

<h3><span>\({\mathsf{global}}~x\)</span></h3>
<ul>
    <li>The global <span>\(C.{\mathsf{globals}}[x]\)</span> must be defined in the context.</li>
    <li>Then the export description is valid with 外部型 <span>\({\mathsf{global}}~C.{\mathsf{globals}}[x]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{globals}}[x] = {\mathit{globaltype}}
}{
  C {\vdash} {\mathsf{global}}~x : {\mathsf{global}}~{\mathit{globaltype}}
}\]</div>

## Import

<h3><span>\(\{ {\mathsf{module}}~{\mathit{name}}_1, {\mathsf{name}}~{\mathit{name}}_2, {\mathsf{desc}}~{\mathit{importdesc}} \}\)</span></h3>
<ul>
    <li>The import description <span>\({\mathit{importdesc}}\)</span> must be valid with type <span>\({\mathit{externtype}}\)</span>.</li>
    <li>Then the import is valid with type <span>\({\mathit{externtype}}\)</span>.</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{importdesc}} : {\mathit{externtype}}
}{
  C {\vdash} \{ {\mathsf{module}}~{\mathit{name}}_1, {\mathsf{name}}~{\mathit{name}}_2, {\mathsf{desc}}~{\mathit{importdesc}} \} : {\mathit{externtype}}
}\]</div>

<h3><span>\({\mathsf{func}}~x\)</span></h3>
<ul>
    <li>The function <span>\(C.{\mathsf{types}}[x]\)</span> must be defined in the context.</li>
    <li>Let <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> be the 関数型<span>\(C.{\mathsf{types}}[x]\)</span>.</li>
    <li>Then the import description is valid with type <span>\({\mathsf{func}}~[t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{types}}[x] = [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C {\vdash} {\mathsf{func}}~x : {\mathsf{func}}~[t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

<h3><span>\({\mathsf{table}}~{\mathit{tabletype}}\)</span></h3>
<ul>
    <li>The table type <span>\({\mathit{tabletype}}\)</span> must be 有効.</li>
    <li>Then the import description is valid with type <span>\({\mathsf{table}}~{\mathit{tabletype}}\)</span>.</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{tabletype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} {\mathsf{table}}~{\mathit{tabletype}} : {\mathsf{table}}~{\mathit{tabletype}}
}\]</div>

<h3><span>\({\mathsf{mem}}~{\mathit{memtype}}\)</span></h3>
<ul>
    <li>The memory type <span>\({\mathit{memtype}}\)</span> must be 有効.</li>
    <li>Then the import description is valid with type <span>\({\mathsf{mem}}~{\mathit{memtype}}\)</span>.</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{memtype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} {\mathsf{mem}}~{\mathit{memtype}} : {\mathsf{mem}}~{\mathit{memtype}}
}\]</div>

<h3><span>\({\mathsf{global}}~{\mathit{globaltype}}\)</span></h3>
<ul>
    <li>The global type <span>\({\mathit{globaltype}}\)</span> must be 有効.</li>
    <li>Then the import description is valid with type <span>\({\mathsf{global}}~{\mathit{globaltype}}\)</span>.</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{globaltype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} {\mathsf{global}}~{\mathit{globaltype}} : {\mathsf{global}}~{\mathit{globaltype}}
}\]</div>

## モジュール群

<ul>
    <li>Let <span>\({\mathit{module}}\)</span> be the module to validate.</li>
    <li>Let <span>\(C\)</span> be a コンテキスト where:
        <ul>
            <li><span>\(C.{\mathsf{types}}\)</span> is <span>\({\mathit{module}}.{\mathsf{types}}\)</span>,</li>
            <li><span>\(C.{\mathsf{funcs}}\)</span> is <span>\({\mathrm{funcs}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{ft}^\ast\)</span>, with the import’s 外部型 <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function types</span></a> <span>\(\mathit{ft}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{tables}}\)</span> is <span>\({\mathrm{tables}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{tt}^\ast\)</span>, with the import’s 外部型 <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-tabletype"><span class="std std-ref">table types</span></a> <span>\(\mathit{tt}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{mems}}\)</span> is <span>\({\mathrm{mems}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{mt}^\ast\)</span>, with the import’s 外部型 <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-memtype"><span class="std std-ref">memory types</span></a> <span>\(\mathit{mt}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{globals}}\)</span> is <span>\({\mathrm{globals}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{gt}^\ast\)</span>, with the import’s 外部型 <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-globaltype"><span class="std std-ref">global types</span></a> <span>\(\mathit{gt}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{locals}}\)</span> is empty,</li>
            <li><span>\(C.{\mathsf{labels}}\)</span> is empty,</li>
            <li><span>\(C.{\mathsf{return}}\)</span> is empty.</li>
        </ul>
    </li>
    <li>Let <span>\(C'\)</span> be the コンテキスト where <span>\(C'.{\mathsf{globals}}\)</span> is the sequence <span>\({\mathrm{globals}}(\mathit{it}^\ast)\)</span> and all other fields are empty.</li>
    <li>Under the context <span>\(C\)</span>:
        <ul>
            <li>For each <span>\({\mathit{functype}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{types}}\)</span>,
        the 関数型<span>\({\mathit{functype}}_i\)</span> must be 有効.</li>
            <li>For each <span>\({\mathit{func}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{funcs}}\)</span>,
        the definition <span>\({\mathit{func}}_i\)</span> must be 有効 with a 関数型<span>\(\mathit{ft}_i\)</span>.</li>
            <li>For each <span>\({\mathit{table}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{tables}}\)</span>,
        the definition <span>\({\mathit{table}}_i\)</span> must be 有効 with a テーブル型 <span>\(\mathit{tt}_i\)</span>.</li>
            <li>For each <span>\({\mathit{mem}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{mems}}\)</span>,
        the definition <span>\({\mathit{mem}}_i\)</span> must be 有効 with a メモリ型 <span>\(\mathit{mt}_i\)</span>.</li>
            <li>For each <span>\({\mathit{global}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{globals}}\)</span>:
            <ul>
                <li>Under the context <span>\(C'\)</span>, the definition <span>\({\mathit{global}}_i\)</span> must be 有効 with a グローバル型 <span>\(\mathit{gt}_i\)</span>.</li>
            </ul>
            </li>
            <li>For each <span>\({\mathit{elem}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{elem}}\)</span>,
        the segment <span>\({\mathit{elem}}_i\)</span> must be 有効.</li>
            <li>For each <span>\({\mathit{data}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{data}}\)</span>, the segment <span>\({\mathit{data}}_i\)</span> must be 有効.</li>
            <li>If <span>\({\mathit{module}}.{\mathsf{start}}\)</span> is non-empty, then <span>\({\mathit{module}}.{\mathsf{start}}\)</span> must be 有効.</li>
            <li>For each <span>\({\mathit{import}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{imports}}\)</span>, the segment <span>\({\mathit{import}}_i\)</span> must be 有効 with an 外部型 <span>\(\mathit{it}_i\)</span>.</li>
            <li>For each <span>\({\mathit{export}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{exports}}\)</span>, the segment <span>\({\mathit{export}}_i\)</span> must be 有効 with 外部型 <span>\(\mathit{et}_i\)</span>.</li>
        </ul>
    </li>
    <li>The length of <span>\(C.{\mathsf{tables}}\)</span> must not be larger than <span>\(1\)</span>.</li>
    <li>The length of <span>\(C.{\mathsf{mems}}\)</span> must not be larger than <span>\(1\)</span>.</li>
    <li>All export names <span>\({\mathit{export}}_i.{\mathsf{name}}\)</span> must be different.</li>
    <li>Let <span>\(\mathit{ft}^\ast\)</span> be the concatenation of the internal 関数型 <span>\(\mathit{ft}_i\)</span>, in index order.</li>
    <li>Let <span>\(\mathit{tt}^\ast\)</span> be the concatenation of the internal テーブル型 <span>\(\mathit{tt}_i\)</span>, in index order.</li>
    <li>Let <span>\(\mathit{mt}^\ast\)</span> be the concatenation of the internal メモリ型 <span>\(\mathit{mt}_i\)</span>, in index order.</li>
    <li>Let <span>\(\mathit{gt}^\ast\)</span> be the concatenation of the internal グローバル型 <span>\(\mathit{gt}_i\)</span>, in index order.</li>
    <li>Let <span>\(\mathit{it}^\ast\)</span> be the concatenation of 外部型 <span>\(\mathit{it}_i\)</span> of the imports, in index order.</li>
    <li>Let <span>\(\mathit{et}^\ast\)</span> be the concatenation of 外部型 <span>\(\mathit{et}_i\)</span> of the exports, in index order.</li>
    <li>Then the module is valid with 外部型 <span>\(\mathit{it}^\ast {\rightarrow} \mathit{et}^\ast\)</span>.</li>
</ul>
<div>\[\begin{split}\frac{
  \begin{array}{&#64;{}c&#64;{}}
  ({\vdash} {\mathit{functype}} \mathrel{\mbox{ok}})^\ast
  \quad
  (C {\vdash} {\mathit{func}} : \mathit{ft})^\ast
  \quad
  (C {\vdash} {\mathit{table}} : \mathit{tt})^\ast
  \quad
  (C {\vdash} {\mathit{mem}} : \mathit{mt})^\ast
  \quad
  (C' {\vdash} {\mathit{global}} : \mathit{gt})^\ast
  \\
  (C {\vdash} {\mathit{elem}} \mathrel{\mbox{ok}})^\ast
  \quad
  (C {\vdash} {\mathit{data}} \mathrel{\mbox{ok}})^\ast
  \quad
  (C {\vdash} {\mathit{start}} \mathrel{\mbox{ok}})^?
  \quad
  (C {\vdash} {\mathit{import}} : \mathit{it})^\ast
  \quad
  (C {\vdash} {\mathit{export}} : \mathit{et})^\ast
  \\
  \mathit{ift}^\ast = {\mathrm{funcs}}(\mathit{it}^\ast)
  \qquad
  \mathit{itt}^\ast = {\mathrm{tables}}(\mathit{it}^\ast)
  \qquad
  \mathit{imt}^\ast = {\mathrm{mems}}(\mathit{it}^\ast)
  \qquad
  \mathit{igt}^\ast = {\mathrm{globals}}(\mathit{it}^\ast)
  \\
  C = \{ {\mathsf{types}}~{\mathit{functype}}^\ast, {\mathsf{funcs}}~\mathit{ift}^\ast~\mathit{ft}^\ast, {\mathsf{tables}}~\mathit{itt}^\ast~\mathit{tt}^\ast, {\mathsf{mems}}~\mathit{imt}^\ast~\mathit{mt}^\ast, {\mathsf{globals}}~\mathit{igt}^\ast~\mathit{gt}^\ast \}
  \\
  C' = \{ {\mathsf{globals}}~\mathit{igt}^\ast \}
  \qquad
  |C.{\mathsf{tables}}| \leq 1
  \qquad
  |C.{\mathsf{mems}}| \leq 1
  \qquad
  ({\mathit{export}}.{\mathsf{name}})^\ast ~\mathrm{disjoint}
  \end{array}
}{
  {\vdash} \{
    \begin{array}[t]{&#64;{}l&#64;{}}
      {\mathsf{types}}~{\mathit{functype}}^\ast,
      {\mathsf{funcs}}~{\mathit{func}}^\ast,
      {\mathsf{tables}}~{\mathit{table}}^\ast,
      {\mathsf{mems}}~{\mathit{mem}}^\ast,
      {\mathsf{globals}}~{\mathit{global}}^\ast, \\
      {\mathsf{elem}}~{\mathit{elem}}^\ast,
      {\mathsf{data}}~{\mathit{data}}^\ast,
      {\mathsf{start}}~{\mathit{start}}^?,
      {\mathsf{imports}}~{\mathit{import}}^\ast,
      {\mathsf{exports}}~{\mathit{export}}^\ast \} : \mathit{it}^\ast {\rightarrow} \mathit{et}^\ast \\
    \end{array}
}\end{split}\]</div>

### 付記

# LINK

<footer>
    <nav>
        <ul>
            <li><a href="Structure" rel="prev">Prev: 構造</a></li>
            <li><a href="./">Top: Index</a></li>
            <li><a href="Execution" rel="next">Next: 実行</a></li>
        </ul>
        <a href="LICENSE" rel="license">LICENSE</a>
    </nav>
</footer>

[^1]: セマンティクスは以下の論文に由来しています。Andreas%20Haas、Andreas%20Rossberg、Derek%20Schuff、Ben%20Titzer、Dan%20Gohman、Luke%20Wagner、Alon%20Zakai、JF%20Bastien、Michael%20Holman。WebAssemblyでWebをスピードアップさせる.第38回ACM%20SIGPLAN%20Conference%20on%20Programming%20Language%20Design%20and%20Implementation%20(PLDI%202017).%20ACM%202017.
[^2]: 例えば[ベンジャミン・ピアース%20型とプログラミング言語。MITプレス2002](https://www.cis.upenn.edu/~bcpierce/tapl/)