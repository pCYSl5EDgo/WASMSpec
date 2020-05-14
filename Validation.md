<script async="async" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">MathJax.Hub.Config({"TeX": {"MAXBUFFER": 30720}})</script>

# 表記上のお約束

検証(バリデーション)は、`WebAssembly`モジュールが適格で有効であるか否かを検証します。
適格で有効なモジュールのみをインスタンス化することができます。

有効性はモジュールの抽象構文とその内容に対する型システムによって定義されます。
抽象構文の各部分には、それに適用される制約を指定する型付け規則があります。すべての規則は、2つの等価な形式で与えられます。

- 散文的表記法では、人間にわかりやすい直感的な形で意味を記述します。
- 形式的表記法では、数学的な形式[^1]で規則を記述します。

### 付記

散文的表記法と形式的表記法は等価です。
この仕様書を読むためには形式表記法の理解は必要ありません。
形式的表記法の方がプログラミング言語の意味論で広く使われている記法です。
より簡潔な記述であり、数学的な証明を容易に行うことができます。

---

どちらの場合も、規則は宣言的な方法で定式化されます。
つまり、制約を定式化するだけで、アルゴリズムを定義することはありません。
この仕様に従った命令シーケンスの型チェックのための健全で完全なアルゴリズムの概要が[付録](Appendix)において提供されています。

## 訳注

<div><a href="Structure#制御命令">\({\mathit{blocktype}}\)</a>はブロックの型を示し、関数型です。</div>

## コンテキスト

個々の定義の有効性は、周囲のモジュールとスコープ内の定義に関する関連情報を収集するコンテキストに対して相対的に指定されます:

- 型:現在のモジュールで定義されている型のリスト。
- 関数:現在のモジュールで宣言された関数のリスト。
- テーブル:現在のモジュールで宣言されたテーブルのリスト。
- メモリ:現在のモジュールで宣言されたメモリのリスト。
- グローバル:現在のモジュールで宣言されているグローバルのリスト。
- ローカル:現在の関数で宣言されたローカルのリスト（パラメータを含む）で、値の型で表されます。
- ラベル:現在の位置からアクセス可能なラベルのスタック。
- 戻り値:現在の関数の戻り値の型で、オプションの結果型として表現されます。

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

<div>
型付けは命令シーケンス<span>\({\mathit{instr}}^\ast\)</span>にまで及びます。
</div>

<div>
このような命令シーケンスを実行することでスタック上に元々あった<span>\(t_1^\ast\)</span>をpopし、実行後に<span>\(t_2^\ast\)</span>を新たにpushするならば関数型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>として表記できます。
</div>

幾つかの命令は命令の種類に応じて型を決定するということが出来ません。
このような命令は`ポリモーフィック`であると言えます。

`ポリモーフィズム`にも2種類あります。

<ul>
    <li><em>値のポリモーフィズム</em>:
命令のオペランドである値型<span>\(t\)</span>の型が不定です。
パラメトリック命令<span>\({\mathsf{drop}}\)</span>と<span>\({\mathsf{select}}\)</span>が当てはまります。</li>
    <li><em>スタックのポリモーフィズム</em>:
すべて（あるいは殆どの）関数型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>命令の型は不定です。
すべての制御命令のうち<em>無条件にジャンプするもの</em>、<span>\({\mathsf{unreachable}}\)</span>, <span>\({\mathsf{br}}\)</span>, <span>\({\mathsf{br\_table}}\)</span>, <span>\({\mathsf{return}}\)</span>などが当てはまります。</li>
</ul>

### 付記

<div>たとえば<span>\({\mathsf{select}}\)</span>命令はあらゆる値型<span>\(t\)</span>に対して<span>\([t~t~{\mathsf{i32}}] {\rightarrow} [t]\)</span>として有効な型です。</div>
よって、次の2つの命令について、
<div>\[({\mathsf{i32}}.{\mathsf{const}}~1)~~({\mathsf{i32}}.{\mathsf{const}}~2)~~({\mathsf{i32}}.{\mathsf{const}}~3)~~{\mathsf{select}}{}\]</div>
<div>\[({\mathsf{f64}}.{\mathsf{const}}~1.0)~~({\mathsf{f64}}.{\mathsf{const}}~2.0)~~({\mathsf{i32}}.{\mathsf{const}}~3)~~{\mathsf{select}}{}\]</div>
<div>双方とも有効であり、<span>\({\mathsf{select}}\)</span>により型付けされた<span>\(t\)</span>は<span>\({\mathsf{i32}}\)</span>か<span>\({\mathsf{f64}}\)</span>の一方になります。</div>

<span>\({\mathsf{unreachable}}\)</span>命令は<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>という型としてあらゆる<span>\(t_1^\ast\)</span>と<span>\(t_2^\ast\)</span>に対して有効です。

<div>よって、<span>\([] {\rightarrow} [{\mathsf{i32}}~{\mathsf{i32}}]\)</span>であると型が推測される<span>\({\mathsf{unreachable}}\)</span>命令に対して</div>
<div>\[{\mathsf{unreachable}}~~{\mathsf{i32}}.{\mathsf{add}}\]</div>
は有効です。

対照的に、同一のスタック上においては、
<div>\[{\mathsf{unreachable}}~~({\mathsf{i64}}.{\mathsf{const}}~0)~~{\mathsf{i32}}.{\mathsf{add}}\]</div>
は無効です。
<div>何故ならば<span>\({\mathsf{unreachable}}\)</span>命令に対して適切な型付けが出来ないからです。</div>

## 算術演算命令

<h3><span>\(t\mathsf{.}{\mathsf{const}}~c\)</span></h3>

<ul>
    <li>この命令は有効です。<span>\([] {\rightarrow} [t]\)</span>。</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t\mathsf{.}{\mathsf{const}}~c : [] {\rightarrow} [t]
}\]</div>

<h3><span>\(t\mathsf{.}{\mathit{unop}}\)</span></h3>

<ul>
    <li>この命令は有効です。<span>\([t] {\rightarrow} [t]\)</span>。</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t\mathsf{.}{\mathit{unop}} : [t] {\rightarrow} [t]
}\]</div>

<h3><span>\(t\mathsf{.}{\mathit{binop}}\)</span></h3>

<ul>
    <li>この命令は有効です。<span>\([t~t] {\rightarrow} [t]\)</span>。</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t\mathsf{.}{\mathit{binop}} : [t~t] {\rightarrow} [t]
}\]</div>

<h3><span>\(t\mathsf{.}{\mathit{testop}}\)</span></h3>

<ul>
    <li>この命令は有効です。<span>\([t] {\rightarrow} [{\mathsf{i32}}]\)</span>。</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t\mathsf{.}{\mathit{testop}} : [t] {\rightarrow} [{\mathsf{i32}}]
}\]</div>

<h3><span>\(t\mathsf{.}{\mathit{relop}}\)</span></h3>

<ul>
    <li>この命令は有効です。<span>\([t~t] {\rightarrow} [{\mathsf{i32}}]\)</span>。</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t\mathsf{.}{\mathit{relop}} : [t~t] {\rightarrow} [{\mathsf{i32}}]
}\]</div>

<h3><span>\(t_2\mathsf{.}{\mathit{cvtop}}\mathsf{\_}t_1\mathsf{\_}{\mathit{sx}}^?\)</span></h3>

<ul>
    <li>この命令は有効です。<span>\([t_1] {\rightarrow} [t_2]\)</span>。</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} t_2\mathsf{.}{\mathit{cvtop}}\mathsf{\_}t_1\mathsf{\_}{\mathit{sx}}^? : [t_1] {\rightarrow} [t_2]
}\]</div>

## パラメトリック命令

<h3><span>\({\mathsf{drop}}\)</span></h3>

<ul>
    <li>この命令は有効です。<span>\(t\)</span>を値型として、<span>\([t] {\rightarrow} []\)</span>。</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} {\mathsf{drop}} : [t] {\rightarrow} []
}\]</div>

<h3><span>\({\mathsf{select}}\)</span></h3>

<ul>
    <li>この命令は有効です。<span>\(t\)</span>を値型として、<span>\([t~t~{\mathsf{i32}}] {\rightarrow} [t]\)</span>。</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} {\mathsf{select}} : [t~t~{\mathsf{i32}}] {\rightarrow} [t]
}\]</div>

### 付記

`drop`と`select`の両方とも`値のポリモーフィズム`を満たす命令です。

## 変数命令

<h3><span>\({\mathsf{local.get}}~x\)</span></h3>

<ul>
    <li>ローカル変数<span>\(C.{\mathsf{locals}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li><span>\(t\)</span>は値型<span>\(C.{\mathsf{locals}}[x]\)</span>であるとします。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([] {\rightarrow} [t]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{locals}}[x] = t
}{
  C {\vdash} {\mathsf{local.get}}~x : [] {\rightarrow} [t]
}\]</div>

<h3><span>\({\mathsf{local.set}}~x\)</span></h3>

<ul>
    <li>ローカル変数<span>\(C.{\mathsf{locals}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li><span>\(t\)</span>は値型<span>\(C.{\mathsf{locals}}[x]\)</span>であるとします。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([t] {\rightarrow} []\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{locals}}[x] = t
}{
  C {\vdash} {\mathsf{local.set}}~x : [t] {\rightarrow} []
}\]</div>

<h3><span>\({\mathsf{local.tee}}~x\)</span></h3>

<ul>
    <li>ローカル変数<span>\(C.{\mathsf{locals}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li><span>\(t\)</span>は値型<span>\(C.{\mathsf{locals}}[x]\)</span>であるとします。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([t] {\rightarrow} [t]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{locals}}[x] = t
}{
  C {\vdash} {\mathsf{local.tee}}~x : [t] {\rightarrow} [t]
}\]</div>

<h3><span>\({\mathsf{global.get}}~x\)</span></h3>

<ul>
    <li>グローバル変数<span>\(C.{\mathsf{globals}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li><span>\({\mathit{mut}}~t\)</span>はグローバル型 <span>\(C.{\mathsf{globals}}[x]\)</span>であるとします。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([] {\rightarrow} [t]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{globals}}[x] = {\mathit{mut}}~t
}{
  C {\vdash} {\mathsf{global.get}}~x : [] {\rightarrow} [t]
}\]</div>

<h3><span>\({\mathsf{global.set}}~x\)</span></h3>

<ul>
    <li>グローバル変数<span>\(C.{\mathsf{globals}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li><span>\({\mathit{mut}}~t\)</span>はグローバル型 <span>\(C.{\mathsf{globals}}[x]\)</span>であるとします。</li>
    <li><span>\({\mathit{mut}}\)</span>が<span>\({\mathsf{var}}\)</span>であるとします。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([t] {\rightarrow} []\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{globals}}[x] = {\mathsf{var}}~t
}{
  C {\vdash} {\mathsf{global.set}}~x : [t] {\rightarrow} []
}\]</div>

## メモリ命令

<h3><span>\(t\mathsf{.}{\mathsf{load}}~{\mathit{memarg}}\)</span></h3>

<ul>
    <li>メモリ<span>\(C.{\mathsf{mems}}[0]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>アラインメント<span>\(2^{{\mathit{memarg}}.{\mathsf{align}}\)</span>は<span>\(t\)</span>のbit幅を<span>\(8\)</span>で除算したもの以下です。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([{\mathsf{i32}}] {\rightarrow} [t]\)</span>。</li>
</ul>

訳注:何故かGoogle Chromeではmathjaxが適切に数式を解釈してくれないのでpngファイルを設置します。大きさが異なり申し訳ないです。

![](Validation_0.png)

<h3><span>\(t\mathsf{.}{\mathsf{load}}{N}\mathsf{\_}{\mathit{sx}}~{\mathit{memarg}}\)</span></h3>

<ul>
    <li>メモリ<span>\(C.{\mathsf{mems}}[0]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>アラインメント<span>\(2^{{\mathit{memarg}}.{\mathsf{align}}\)</span>は<span>\(N/8\)</span>以下です。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([{\mathsf{i32}}] {\rightarrow} [t]\)</span>。</li>
</ul>

![](Validation_1.png)

<h3><span>\(t\mathsf{.}{\mathsf{store}}~{\mathit{memarg}}\)</span></h3>

<ul>
    <li>メモリ<span>\(C.{\mathsf{mems}}[0]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>アラインメント<span>\(2^{{\mathit{memarg}}.{\mathsf{align}}\)</span>は<span>\(t\)</span>のbit幅を<span>\(8\)</span>で除算したもの以下です。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([{\mathsf{i32}}~t] {\rightarrow} []\)</span>。</li>
</ul>

![](Validation_2.png)

<h3><span>\(t\mathsf{.}{\mathsf{store}}{N}~{\mathit{memarg}}\)</span></h3>

<ul>
    <li>メモリ<span>\(C.{\mathsf{mems}}[0]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>アラインメント<span>\(2^{{\mathit{memarg}}.{\mathsf{align}}\)</span>は<span>\(N/8\)</span>以下です。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([{\mathsf{i32}}~t] {\rightarrow} []\)</span>。</li>
</ul>

![](Validation_3.png)

<h3><span>\({\mathsf{memory.size}}\)</span></h3>

<ul>
    <li>メモリ<span>\(C.{\mathsf{mems}}[0]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([] {\rightarrow} [{\mathsf{i32}}]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[0] = {\mathit{memtype}}
}{
  C {\vdash} {\mathsf{memory.size}} : [] {\rightarrow} [{\mathsf{i32}}]
}\]</div>

<h3><span>\({\mathsf{memory.grow}}\)</span></h3>

<ul>
    <li>メモリ<span>\(C.{\mathsf{mems}}[0]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([{\mathsf{i32}}] {\rightarrow} [{\mathsf{i32}}]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[0] = {\mathit{memtype}}
}{
  C {\vdash} {\mathsf{memory.grow}} : [{\mathsf{i32}}] {\rightarrow} [{\mathsf{i32}}]
}\]</div>

## 制御命令

<h3><span>\({\mathsf{nop}}\)</span></h3>

<ul>
    <li>この命令は有効です。<span>\([] {\rightarrow} []\)</span>。</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} {\mathsf{nop}} : [] {\rightarrow} []
}\]</div>

<h3><span>\({\mathsf{unreachable}}\)</span></h3>

<ul>
    <li>この命令は有効です。あらゆる値型のシーケンス<span>\(t_1^\ast\)</span>と<span>\(t_2^\ast\)</span>に対して<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>。</li>
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
    <li>ブロック型は有効関数型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>として有効でなくてはなりません。</li>
    <li>コンテキスト<span>\(C'\)</span>をコンテキスト<span>\(C\)</span>と同一であるとしますが、しかし戻り値型<span>\([t_2^\ast]\)</span>がラベルベクトル<span>\({\mathsf{labels}}\)</span>の前に付加されているとします。</li>
    <li>コンテキスト<span>\(C'\)</span>を前提として、命令シーケンス<span>\({\mathit{instr}}^\ast\)</span>は型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>として有効であるとします。</li>
    <li>以上の条件を満足する時、複合命令は有効です。<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>。</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{blocktype}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
  \qquad
  C,{\mathsf{labels}}\,[t_2^\ast] {\vdash} {\mathit{instr}}^\ast : [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C {\vdash} {\mathsf{block}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

### 付記

<div>コンテキスト<span>\(C,{\mathsf{labels}}\,[t^\ast]\)</span>はラベル型をインデックス<span>\(0\)</span>に挿入し、他をずらします。</div>

---

<h3><span>\({\mathsf{loop}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}}\)</span></h3>

<ul>
    <li>ブロック型は有効関数型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>として有効でなくてはなりません。</li>
    <li>コンテキスト<span>\(C'\)</span>をコンテキスト<span>\(C\)</span>と同一であるとしますが、しかし戻り値型<span>\([t_2^\ast]\)</span>がラベルベクトル<span>\({\mathsf{labels}}\)</span>の前に付加されているとします。</li>
    <li>コンテキスト<span>\(C'\)</span>を前提として、命令シーケンス<span>\({\mathit{instr}}^\ast\)</span>は型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>として有効であるとします。</li>
    <li>以上の条件を満足する時、複合命令は有効です。<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>。</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{blocktype}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
  \qquad
  C,{\mathsf{labels}}\,[t_1^\ast] {\vdash} {\mathit{instr}}^\ast : [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C {\vdash} {\mathsf{loop}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

### 付記

<div>コンテキスト<span>\(C,{\mathsf{labels}}\,[t^\ast]\)</span>はラベル型をインデックス<span>\(0\)</span>に挿入し、他をずらします。</div>

---

<h3><span>\({\mathsf{if}}~{\mathit{blocktype}}~{\mathit{instr}}_1^\ast~{\mathsf{else}}~{\mathit{instr}}_2^\ast~{\mathsf{end}}\)</span></h3>

<ul>
    <li>ブロック型は有効関数型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>として有効でなくてはなりません。</li>
    <li>コンテキスト<span>\(C'\)</span>をコンテキスト<span>\(C\)</span>と同一であるとしますが、しかし戻り値型<span>\([t_2^\ast]\)</span>がラベルベクトル<span>\({\mathsf{labels}}\)</span>の前に付加されているとします。</li>
    <li>コンテキスト<span>\(C'\)</span>を前提として、命令シーケンス<span>\({\mathit{instr}}_1^\ast\)</span>は型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>として有効であるとします。</li>
    <li>コンテキスト<span>\(C'\)</span>を前提として、命令シーケンス<span>\({\mathit{instr}}_2^\ast\)</span>は型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>として有効であるとします。</li>
    <li>以上の条件を満足する時、複合命令は有効です。<span>\([t_1^\ast~{\mathsf{i32}}] {\rightarrow} [t_2^\ast]\)</span>。</li>
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

<div>コンテキスト<span>\(C,{\mathsf{labels}}\,[t^\ast]\)</span>はラベル型をインデックス<span>\(0\)</span>に挿入し、他をずらします。</div>

---

<h3><span>\({\mathsf{br}}~l\)</span></h3>

<ul>
    <li>ラベル<span>\(C.{\mathsf{labels}}[l]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li><span>\([t^\ast]\)</span>は戻り値型<span>\(C.{\mathsf{labels}}[l]\)</span>であるとします。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]\)</span>, for any sequences of 値型 <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{labels}}[l] = [t^\ast]
}{
  C {\vdash} {\mathsf{br}}~l : [t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

<h3><span>\({\mathsf{br\_if}}~l\)</span></h3>

<ul>
    <li>ラベル<span>\(C.{\mathsf{labels}}[l]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li><span>\([t^\ast]\)</span>は戻り値型<span>\(C.{\mathsf{labels}}[l]\)</span>であるとします。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([t^\ast~{\mathsf{i32}}] {\rightarrow} [t^\ast]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{labels}}[l] = [t^\ast]
}{
  C {\vdash} {\mathsf{br\_if}}~l : [t^\ast~{\mathsf{i32}}] {\rightarrow} [t^\ast]
}\]</div>

<h3><span>\({\mathsf{br\_table}}~l^\ast~l_N\)</span></h3>

<ul>
    <li>ラベル<span>\(C.{\mathsf{labels}}[l_N]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li><span>\([t^\ast]\)</span>は戻り値型 <span>\(C.{\mathsf{labels}}[l_N]\)</span>であるとします。</li>
    <li>全ての<span>\(l^\ast\)</span>中の<span>\(l_i\)</span>について、ラベル<span>\(C.{\mathsf{labels}}[l_i]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>全ての<span>\(l^\ast\)</span>中の<span>\(l_i\)</span>について、ラベル<span>\(C.{\mathsf{labels}}[l_i]\)</span>は<span>\([t^\ast]\)</span>でなければなりません。</li>
    <li>以上の条件を満足する時、この命令は有効です。あらゆる値型のシーケンス<span>\(t_1^\ast\)</span>と<span>\(t_2^\ast\)</span>に対して<span>\([t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]\)</span>。</li>
</ul>
<div>\[\frac{
  (C.{\mathsf{labels}}[l] = [t^\ast])^\ast
  \qquad
  C.{\mathsf{labels}}[l_N] = [t^\ast]
}{
  C {\vdash} {\mathsf{br\_table}}~l^\ast~l_N : [t_1^\ast~t^\ast~{\mathsf{i32}}] {\rightarrow} [t_2^\ast]
}\]</div>

<h3><span>\({\mathsf{return}}\)</span></h3>

<ul>
    <li>戻り値型<span>\(C.{\mathsf{return}}\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li><span>\([t^\ast]\)</span>が<span>\(C.{\mathsf{return}}\)</span>の戻り値型であるとします。</li>
    <li>以上の条件を満足する時、この命令は有効です。あらゆる値型のシーケンス<span>\(t_1^\ast\)</span>と<span>\(t_2^\ast\)</span>に対して<span>\([t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{return}} = [t^\ast]
}{
  C {\vdash} {\mathsf{return}} : [t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

<h3><span>\({\mathsf{call}}~x\)</span></h3>

<ul>
    <li>関数<span>\(C.{\mathsf{funcs}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\(C.{\mathsf{funcs}}[x]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{funcs}}[x] = [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C {\vdash} {\mathsf{call}}~x : [t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

<h3><span>\({\mathsf{call\_indirect}}~x\)</span></h3>

<ul>
    <li>テーブル<span>\(C.{\mathsf{tables}}[0]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li><span>\({\mathit{limits}}~{\mathit{elemtype}}\)</span>がテーブル型<span>\(C.{\mathsf{tables}}[0]\)</span>であるとします。</li>
    <li>要素型<span>\({\mathit{elemtype}}\)</span>は<span>\({\mathsf{funcref}}\)</span>でなければなりません。</li>
    <li>型<span>\(C.{\mathsf{types}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li><span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>が関数型<span>\(C.{\mathsf{types}}[x]\)</span>であるとします。</li>
    <li>以上の条件を満足する時、この命令は有効です。<span>\([t_1^\ast~{\mathsf{i32}}] {\rightarrow} [t_2^\ast]\)</span>。</li>
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
    <li>空の命令シーケンスはあらゆる値型のシーケンス<span>\(t^\ast\)</span>に対して型<span>\([t^\ast] {\rightarrow} [t^\ast]\)</span>として有効です。</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} \epsilon : [t^\ast] {\rightarrow} [t^\ast]
}\]</div>

<h3>空でない命令シーケンス: <span>\({\mathit{instr}}^\ast~{\mathit{instr}}_N\)</span></h3>

<ul>
    <li>命令シーケンス<span>\({\mathit{instr}}^\ast\)</span>はある種の値型のシーケンス<span>\(t_1^\ast\)</span>と<span>\(t_2^\ast\)</span>に対して型<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>として有効です。</li>
    <li>命令<span>\({\mathit{instr}}_N\)</span>はある種の値型のシーケンス<span>\(t^\ast\)</span>と<span>\(t_3^\ast\)</span>に対して型<span>\([t^\ast] {\rightarrow} [t_3^\ast]\)</span>として有効です。</li>
    <li><span>\(t_2^\ast = t_0^\ast~t^\ast\)</span>のような値型のシーケンス<span>\(t_0^\ast\)</span>が存在せねばなりません。</li>
    <li>以上の条件を満足する時、複合命令シーケンスは型<span>\([t_1^\ast] {\rightarrow} [t_0^\ast~t_3^\ast]\)</span>として有効です。</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{instr}}^\ast : [t_1^\ast] {\rightarrow} [t_0^\ast~t^\ast]
  \qquad
  C {\vdash} {\mathit{instr}}_N : [t^\ast] {\rightarrow} [t_3^\ast]
}{
  C {\vdash} {\mathit{instr}}^\ast~{\mathit{instr}}_N : [t_1^\ast] {\rightarrow} [t_0^\ast~t_3^\ast]
}\]</div>

## 式

<div>式は戻り値型<span>\([t^\ast]\)</span>として有効です。</div>

<h3><span>\({\mathit{instr}}^\ast~{\mathsf{end}}\)</span></h3>

<ul>
    <li>命令シーケンス<span>\({\mathit{instr}}^\ast\)</span>はいくつかの戻り値型<span>\([t^\ast]\)</span>に対して型<span>\([] {\rightarrow} [t^\ast]\)</span>として有効です。</li>
    <li>以上の条件を満足する時、式は戻り値型<span>\([t^\ast]\)</span>として有効です。</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{instr}}^\ast : [] {\rightarrow} [t^\ast]
}{
  C {\vdash} {\mathit{instr}}^\ast~{\mathsf{end}} : [t^\ast]
}\]</div>

### 定数式

<ul>
    <li>定数式<span>\({\mathit{instr}}^\ast~{\mathsf{end}}\)</span>中の全ての命令<span>\({\mathit{instr}}^\ast\)</span>は定数でなければなりません。</li>
    <li>定数命令<span>\({\mathit{instr}}\)</span>は必ず:
        <ul>
            <li>いずれかの形式を取る命令<span>\(t.{\mathsf{const}}~c\)</span>であるか、</li>
            <li>もしくは、形式<span>\({\mathsf{global.get}}~x\)</span>を取ります。この場合<span>\(C.{\mathsf{globals}}[x]\)</span>は必ずグローバル型<span>\({\mathsf{const}}~t\)</span>でなければなりません。</li>
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

現在、グローバルの初期化子として発生する定数式は含まれる`global.get`命令がインポートされたグローバルを参照することしかできないという制約があります。
これはコンテキスト`C`を制約することで、モジュールの検証ルールで強制されます。

定数式の定義は、WebAssembly の将来のバージョンで拡張される可能性があります。

# モジュール

## 関数

<h3><span>\(\{ {\mathsf{type}}~x, {\mathsf{locals}}~t^\ast, {\mathsf{body}}~{\mathit{expr}} \}\)</span></h3>
<ul>
    <li>The type <span>\(C.{\mathsf{types}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>Let <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> be the 関数型<span>\(C.{\mathsf{types}}[x]\)</span>。</li>
    <li>Let <span>\(C'\)</span> be the same コンテキスト as <span>\(C\)</span>,
but with:
<ul>
    <li><span>\({\mathsf{locals}}\)</span> set to the sequence of 値型 <span>\(t_1^\ast~t^\ast\)</span>, concatenating parameters and locals,</li>
    <li><span>\({\mathsf{labels}}\)</span> set to the singular sequence containing only 戻り値型 <span>\([t_2^\ast]\)</span>。</li>
    <li><span>\({\mathsf{return}}\)</span> set to the 戻り値型 <span>\([t_2^\ast]\)</span>。</li>
</ul>
</li>
    <li>Under the context <span>\(C'\)</span>,
the expression <span>\({\mathit{expr}}\)</span> must be valid with type <span>\([t_2^\ast]\)</span>。</li>
    <li>以上の条件を満足する時、関数定義は有効です。<span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>。</li>
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
    <li>The テーブル型 <span>\({\mathit{tabletype}}\)</span>は有効でなくてはなりません。</li>
    <li>以上の条件を満足する時、テーブル定義は有効です。<span>\({\mathit{tabletype}}\)</span>。</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{tabletype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} \{ {\mathsf{type}}~{\mathit{tabletype}} \} : {\mathit{tabletype}}
}\]</div>

## メモリ

<h3><span>\(\{ {\mathsf{type}}~{\mathit{memtype}} \}\)</span></h3>
<ul>
    <li>The メモリ型 <span>\({\mathit{memtype}}\)</span>は有効でなくてはなりません。</li>
    <li>以上の条件を満足する時、メモリ定義は有効です。<span>\({\mathit{memtype}}\)</span>。</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{memtype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} \{ {\mathsf{type}}~{\mathit{memtype}} \} : {\mathit{memtype}}
}\]</div>

## グローバル

<h3><span>\(\{ {\mathsf{type}}~{\mathit{mut}}~t, {\mathsf{init}}~{\mathit{expr}} \}\)</span></h3>
<ul>
    <li>The グローバル型 <span>\({\mathit{mut}}~t\)</span>は有効でなくてはなりません。</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 有効 with 戻り値型 <span>\([t]\)</span>。</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 定数。</li>
    <li>以上の条件を満足する時、グローバル定義は有効です。<span>\({\mathit{mut}}~t\)</span>。</li>
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
    <li>テーブル<span>\(C.{\mathsf{tables}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>Let <span>\({\mathit{limits}}~{\mathit{elemtype}}\)</span> be the テーブル型 <span>\(C.{\mathsf{tables}}[x]\)</span>。</li>
    <li>要素型<span>\({\mathit{elemtype}}\)</span> must be <span>\({\mathsf{funcref}}\)</span>。</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 有効 with 戻り値型 <span>\([{\mathsf{i32}}]\)</span>。</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 定数。</li>
    <li>For each <span>\(y_i\)</span> in <span>\(y^\ast\)</span>,
関数<span>\(C.{\mathsf{funcs}}[y]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>以上の条件を満足する時、the element segment is valid。</li>
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
    <li>メモリ<span>\(C.{\mathsf{mems}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 有効 with 戻り値型 <span>\([{\mathsf{i32}}]\)</span>。</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be 定数。</li>
    <li>以上の条件を満足する時、the data segment is valid。</li>
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
    <li>関数<span>\(C.{\mathsf{funcs}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>The type of <span>\(C.{\mathsf{funcs}}[x]\)</span> must be <span>\([] {\rightarrow} []\)</span>。</li>
    <li>以上の条件を満足する時、the start function is valid。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{funcs}}[x] = [] {\rightarrow} []
}{
  C {\vdash} \{ {\mathsf{func}}~x \} \mathrel{\mbox{ok}}
}\]</div>

## Export

<h3><span>\(\{ {\mathsf{name}}~{\mathit{name}}, {\mathsf{desc}}~{\mathit{exportdesc}} \}\)</span></h3>
<ul>
    <li>The export description <span>\({\mathit{exportdesc}}\)</span> must be valid with 外部型 <span>\({\mathit{externtype}}\)</span>。</li>
    <li>以上の条件を満足する時、the export is valid with 外部型 <span>\({\mathit{externtype}}\)</span>。</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{exportdesc}} : {\mathit{externtype}}
}{
  C {\vdash} \{ {\mathsf{name}}~{\mathit{name}}, {\mathsf{desc}}~{\mathit{exportdesc}} \} : {\mathit{externtype}}
}\]</div>

<h3><span>\({\mathsf{func}}~x\)</span></h3>
<ul>
    <li>関数<span>\(C.{\mathsf{funcs}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>以上の条件を満足する時、the export description is valid with 外部型 <span>\({\mathsf{func}}~C.{\mathsf{funcs}}[x]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{funcs}}[x] = {\mathit{functype}}
}{
  C {\vdash} {\mathsf{func}}~x : {\mathsf{func}}~{\mathit{functype}}
}\]</div>

<h3><span>\({\mathsf{table}}~x\)</span></h3>
<ul>
    <li>テーブル<span>\(C.{\mathsf{tables}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>以上の条件を満足する時、the export description is valid with 外部型 <span>\({\mathsf{table}}~C.{\mathsf{tables}}[x]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{tables}}[x] = {\mathit{tabletype}}
}{
  C {\vdash} {\mathsf{table}}~x : {\mathsf{table}}~{\mathit{tabletype}}
}\]</div>

<h3><span>\({\mathsf{mem}}~x\)</span></h3>
<ul>
    <li>メモリ<span>\(C.{\mathsf{mems}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>以上の条件を満足する時、the export description is valid with 外部型 <span>\({\mathsf{mem}}~C.{\mathsf{mems}}[x]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[x] = {\mathit{memtype}}
}{
  C {\vdash} {\mathsf{mem}}~x : {\mathsf{mem}}~{\mathit{memtype}}
}\]</div>

<h3><span>\({\mathsf{global}}~x\)</span></h3>
<ul>
    <li>The global <span>\(C.{\mathsf{globals}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>以上の条件を満足する時、the export description is valid with 外部型 <span>\({\mathsf{global}}~C.{\mathsf{globals}}[x]\)</span>。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{globals}}[x] = {\mathit{globaltype}}
}{
  C {\vdash} {\mathsf{global}}~x : {\mathsf{global}}~{\mathit{globaltype}}
}\]</div>

## Import

<h3><span>\(\{ {\mathsf{module}}~{\mathit{name}}_1, {\mathsf{name}}~{\mathit{name}}_2, {\mathsf{desc}}~{\mathit{importdesc}} \}\)</span></h3>
<ul>
    <li>The import description <span>\({\mathit{importdesc}}\)</span> must be valid with type <span>\({\mathit{externtype}}\)</span>。</li>
    <li>以上の条件を満足する時、Importは有効であり、型は<span>\({\mathit{externtype}}\)</span>となります。</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{importdesc}} : {\mathit{externtype}}
}{
  C {\vdash} \{ {\mathsf{module}}~{\mathit{name}}_1, {\mathsf{name}}~{\mathit{name}}_2, {\mathsf{desc}}~{\mathit{importdesc}} \} : {\mathit{externtype}}
}\]</div>

<h3><span>\({\mathsf{func}}~x\)</span></h3>
<ul>
    <li>関数<span>\(C.{\mathsf{types}}[x]\)</span>はコンテキスト中に定義されていなければなりません。</li>
    <li>Let <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> be the 関数型<span>\(C.{\mathsf{types}}[x]\)</span>。</li>
    <li>以上の条件を満足する時、Import詳細は有効であり、型は<span>\({\mathsf{func}}~[t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>となります。</li>
</ul>
<div>\[\frac{
  C.{\mathsf{types}}[x] = [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C {\vdash} {\mathsf{func}}~x : {\mathsf{func}}~[t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

<h3><span>\({\mathsf{table}}~{\mathit{tabletype}}\)</span></h3>
<ul>
    <li>テーブル型<span>\({\mathit{tabletype}}\)</span>は有効でなくてはなりません。</li>
    <li>以上の条件を満足する時、Import詳細は有効であり、型は<span>\({\mathsf{table}}~{\mathit{tabletype}}\)</span>となります。</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{tabletype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} {\mathsf{table}}~{\mathit{tabletype}} : {\mathsf{table}}~{\mathit{tabletype}}
}\]</div>

<h3><span>\({\mathsf{mem}}~{\mathit{memtype}}\)</span></h3>
<ul>
    <li>メモリ型<span>\({\mathit{memtype}}\)</span>は有効でなくてはなりません。</li>
    <li>以上の条件を満足する時、Import詳細は有効であり、型は<span>\({\mathsf{mem}}~{\mathit{memtype}}\)</span>となります。</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{memtype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} {\mathsf{mem}}~{\mathit{memtype}} : {\mathsf{mem}}~{\mathit{memtype}}
}\]</div>

<h3><span>\({\mathsf{global}}~{\mathit{globaltype}}\)</span></h3>
<ul>
    <li>グローバル型<span>\({\mathit{globaltype}}\)</span>は有効でなくてはなりません。</li>
    <li>以上の条件を満足する時、Import詳細は有効であり、型は<span>\({\mathsf{global}}~{\mathit{globaltype}}\)</span>となります。</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{globaltype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} {\mathsf{global}}~{\mathit{globaltype}} : {\mathsf{global}}~{\mathit{globaltype}}
}\]</div>

## モジュール群

<ul>
    <li>Let <span>\({\mathit{module}}\)</span> be the module to validate。</li>
    <li>Let <span>\(C\)</span> be a コンテキスト where:
        <ul>
            <li><span>\(C.{\mathsf{types}}\)</span> is <span>\({\mathit{module}}.{\mathsf{types}}\)</span>,</li>
            <li><span>\(C.{\mathsf{funcs}}\)</span> is <span>\({\mathrm{funcs}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{ft}^\ast\)</span>, with the import’s 外部型 <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function types</span></a> <span>\(\mathit{ft}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{tables}}\)</span> is <span>\({\mathrm{tables}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{tt}^\ast\)</span>, with the import’s 外部型 <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-tabletype"><span class="std std-ref">table types</span></a> <span>\(\mathit{tt}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{mems}}\)</span> is <span>\({\mathrm{mems}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{mt}^\ast\)</span>, with the import’s 外部型 <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-memtype"><span class="std std-ref">memory types</span></a> <span>\(\mathit{mt}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{globals}}\)</span> is <span>\({\mathrm{globals}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{gt}^\ast\)</span>, with the import’s 外部型 <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-globaltype"><span class="std std-ref">global types</span></a> <span>\(\mathit{gt}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{locals}}\)</span> is empty,</li>
            <li><span>\(C.{\mathsf{labels}}\)</span> is empty,</li>
            <li><span>\(C.{\mathsf{return}}\)</span> is empty。</li>
        </ul>
    </li>
    <li>Let <span>\(C'\)</span> be the コンテキスト where <span>\(C'.{\mathsf{globals}}\)</span> is the sequence <span>\({\mathrm{globals}}(\mathit{it}^\ast)\)</span> and all other fields are empty。</li>
    <li>Under the context <span>\(C\)</span>:
        <ul>
            <li>For each <span>\({\mathit{functype}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{types}}\)</span>, the 関数型<span>\({\mathit{functype}}_i\)</span>は有効でなくてはなりません。</li>
            <li>For each <span>\({\mathit{func}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{funcs}}\)</span>, the definition <span>\({\mathit{func}}_i\)</span> must be 有効 with a 関数型<span>\(\mathit{ft}_i\)</span>。</li>
            <li>For each <span>\({\mathit{table}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{tables}}\)</span>, the definition <span>\({\mathit{table}}_i\)</span> must be 有効 with a テーブル型 <span>\(\mathit{tt}_i\)</span>。</li>
            <li>For each <span>\({\mathit{mem}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{mems}}\)</span>, the definition <span>\({\mathit{mem}}_i\)</span> must be 有効 with a メモリ型 <span>\(\mathit{mt}_i\)</span>。</li>
            <li>For each <span>\({\mathit{global}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{globals}}\)</span>:
            <ul>
                <li>Under the context <span>\(C'\)</span>, the definition <span>\({\mathit{global}}_i\)</span> must be 有効 with a グローバル型 <span>\(\mathit{gt}_i\)</span>。</li>
            </ul>
            </li>
            <li>For each <span>\({\mathit{elem}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{elem}}\)</span>, the segment <span>\({\mathit{elem}}_i\)</span>は有効でなくてはなりません。</li>
            <li>For each <span>\({\mathit{data}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{data}}\)</span>, the segment <span>\({\mathit{data}}_i\)</span>は有効でなくてはなりません。</li>
            <li>If <span>\({\mathit{module}}.{\mathsf{start}}\)</span> is non-empty, then <span>\({\mathit{module}}.{\mathsf{start}}\)</span>は有効でなくてはなりません。</li>
            <li>For each <span>\({\mathit{import}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{imports}}\)</span>, the segment <span>\({\mathit{import}}_i\)</span> must be 有効 with an 外部型 <span>\(\mathit{it}_i\)</span>。</li>
            <li>For each <span>\({\mathit{export}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{exports}}\)</span>, the segment <span>\({\mathit{export}}_i\)</span> must be 有効 with 外部型 <span>\(\mathit{et}_i\)</span>。</li>
        </ul>
    </li>
    <li>The length of <span>\(C.{\mathsf{tables}}\)</span> must not be larger than <span>\(1\)</span>。</li>
    <li>The length of <span>\(C.{\mathsf{mems}}\)</span> must not be larger than <span>\(1\)</span>。</li>
    <li>All export names <span>\({\mathit{export}}_i.{\mathsf{name}}\)</span> must be different。</li>
    <li>Let <span>\(\mathit{ft}^\ast\)</span> be the concatenation of the internal 関数型 <span>\(\mathit{ft}_i\)</span>, in index order。</li>
    <li>Let <span>\(\mathit{tt}^\ast\)</span> be the concatenation of the internal テーブル型 <span>\(\mathit{tt}_i\)</span>, in index order。</li>
    <li>Let <span>\(\mathit{mt}^\ast\)</span> be the concatenation of the internal メモリ型 <span>\(\mathit{mt}_i\)</span>, in index order。</li>
    <li>Let <span>\(\mathit{gt}^\ast\)</span> be the concatenation of the internal グローバル型 <span>\(\mathit{gt}_i\)</span>, in index order。</li>
    <li>Let <span>\(\mathit{it}^\ast\)</span> be the concatenation of 外部型 <span>\(\mathit{it}_i\)</span> of the imports, in index order。</li>
    <li>Let <span>\(\mathit{et}^\ast\)</span> be the concatenation of 外部型 <span>\(\mathit{et}_i\)</span> of the exports, in index order。</li>
    <li>以上の条件を満足する時、the module is valid with 外部型 <span>\(\mathit{it}^\ast {\rightarrow} \mathit{et}^\ast\)</span>。</li>
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