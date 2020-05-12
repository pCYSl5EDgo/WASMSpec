<script async="async" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">MathJax.Hub.Config({"TeX": {"MAXBUFFER": 30720}})</script>

# 表記上のお約束

`WebAssembly`は複数の具体的な表現（そのバイナリ形式とテキスト形式）を持つプログラミング言語です。
両方とも共通の構造に対応しています。
簡潔さのためにこの構造は抽象構文の形で記述されています。
この仕様のすべての部分はこの抽象構文の観点から定義されています。

## 文法表記法

抽象構文の文法規則を定義する際には以下の規則を採用しています。

<ul>
    <li>終端記号（アトム）はサンセリフフォントで表記します: <span>\(\mathsf{i32}, \mathsf{end}\)</span>。</li>
    <li>非末端記号はイタリック体で表記します: <span>\(\mathit{valtype}, \mathit{instr}\)</span>。</li>
    <li><span>\(A^n\)</span>は<span>\(A\)</span>の<span>\(n\geq 0\)</span>回の繰り返しです。</li>
    <li><span>\(A^\ast\)</span>は<span>\(A\)の繰り返しの空のシーケンスの可能性があります（これは n が関係ない場合に使用される<span>\(A^n\)</span>の略記法です）。</span></li>
    <li><span>\(A^+\)</span>は<span>\(A\)</span> の繰り返しの空ではないシーケンスです（これは n≧1 の場合の<span>\(A^n\)</span>の速記法です）。</li>
    <li><span>\(A^?\)</span>は<span>\(A\)</span>が1つ以下存在するかもしれないことを示します。</li>
    <li>積和は<span>\(\mathit{sym} ::= A_1 ~|~ \dots ~|~ A_n\)</span>と表記します。</li>
    <li>大きな積和は複数の定義に分割されることがあり、連続体<span>\(\mathit{sym} ::= \dots ~|~ A_2\)</span>を省略記号で開始し、<span>\(\mathit{sym} ::= A_1 ~|~ \dots\)</span>で明示的に終わらせることで示されます。</li>
    <li>いくつかの積和は括弧内に付記された条件“<span>\((\mathrel{\mbox{if}} \mathit{condition})\)</span>”で補強されていますが、これは積和を多くの別々のケースに組み合わせて拡張するための略記法です。</li>
</ul>

## 補助記法

構文を扱う際には、以下の表記法も併用します。

### シーケンス

<ul>
    <li><span>\(\epsilon\)</span>は空のシーケンスを表します。</li>
    <li><span>\(|s|\)</span>はシーケンス<span>\(s\)</span>の長さを表します。</li>
    <li><span>\(s[i]\)</span>はシーケンス<span>\(s\)</span>の0 index startで<span>\(i\)</span>番目の要素を表します。</li>
    <li><span>\(s[i{\mathrel{\mathbf{:}}} n]\)</span>はシーケンス<span>\(s\)</span>の<span>\(s[i]~\dots~s[i+n-1]\)</span>番目の要素を含む部分シーケンスを表します。</li>
    <li><span>\(s {\mathrel{\mbox{with}}} [i] = A\)</span>はシーケンス<span>\(s\)</span>の<span>\(i\)</span>番目の要素が<span>\(A\)</span>に置換されたものを表します。</li>
    <li><span>\(s {\mathrel{\mbox{with}}} [i{\mathrel{\mathbf{:}}} n] = A^n\)</span>は<span>\(s\)</span>の部分シーケンス<span>\(s[i{\mathrel{\mathbf{:}}} n]\)</span>が<span>\(A^n\)</span>で置換したものを表します。</li>
    <li><span>\({\mathrm{concat}}(s^\ast)\)</span>は<span>\(s^\ast\)</span>に含まれるすべてのシーケンス<span>\(s_i\)</span>を連結したものを表します。</li>
</ul>

### メタ構文

<ul>
    <li><span>\(x\)</span>が非終端記号である<span>\(x^n\)</span>は<span>\(x\)</span>のシーケンスを表します。</li>
    <li><span>\(x^n\)</span>が与えられた場合、<span>\((A_1~x~A_2)^n\)</span>と書かれた中に出現した<span>\(x\)</span>は<span>\(x^n\)</span>に対応した長さで存在するものとします。シーケンス中の構文のマッピングを暗示しています。</li>
</ul>

### 積和

<div>以下の形式の積和は、「フィールド」<span>\(\mathsf{field}_i\)</span>の固定集合をそれぞれ「値」<span>\(A_i\)</span>にマッピングするレコードとして解釈されます。</div>
<div>\[\mathit{r} ~::=~ \{ \mathsf{field}_1~A_1, \mathsf{field}_2~A_2, \dots \}\]</div>

このようなレコードを操作する際には以下のような表記を採用しています:

<ul>
    <li><span>\(r.\mathsf{field}\)</span>は<span>\(r\)</span>のフィールドである<span>\(\mathsf{field}\)</span>を表します。</li>
    <li><span>\(r {\mathrel{\mbox{with}}} \mathsf{field} = A\)</span>は<span>\(r\)</span>のフィールド<span>\(\mathsf{field}\)</span>の内容を<span>\(A\)</span>で置換したものを表します。</li>
    <li><span>\(r_1 {\oplus} r_2\)</span> は、2つの同じフィールドを持つフィールドのシーケンスに対してそれぞれのフィールドを合成したシーケンスを表します。:<div>\[\{ \mathsf{field}_1\,A_1^\ast, \mathsf{field}_2\,A_2^\ast, \dots \} {\oplus} \{ \mathsf{field}_1\,B_1^\ast, \mathsf{field}_2\,B_2^\ast, \dots \} = \{ \mathsf{field}_1\,A_1^\ast~B_1^\ast, \mathsf{field}_2\,A_2^\ast~B_2^\ast, \dots \}\]</div></li>
    <li><span>\({\bigoplus} r^\ast\)</span> はそれぞれレコードのシーケンスの合成を表し、シーケンスが空の場合は、結果として得られるレコードのすべてのフィールドが空になります。</li>
</ul>

シーケンスとレコードの更新記法は、"パス "によってアクセスされるネストされた構成要素に再帰的に一般化されます：
<div><span>\(\mathit{pth} ::= ([\dots] \;| \;.\mathsf{field})^+\)</span></div>

<ul class="simple">
    <li><span>\(s {\mathrel{\mbox{with}}} [i]\,\mathit{pth} = A\)</span>は<span>\(s {\mathrel{\mbox{with}}} [i] = (s[i] {\mathrel{\mbox{with}}} \mathit{pth} = A)\)</span>の短縮形です。</li>
    <li><span>\(r {\mathrel{\mbox{with}}} \mathsf{field}\,\mathit{pth} = A\)</span>は<span>\(r {\mathrel{\mbox{with}}} \mathsf{field} = (r.\mathsf{field} {\mathrel{\mbox{with}}} \mathit{pth} = A)\)</span>の短縮形です。</li>
</ul>

<div>ここで<span>\(r {\mathrel{\mbox{with}}}~.\mathsf{field} = A\)</span>は<span>\(r {\mathrel{\mbox{with}}} \mathsf{field} = A\)</span>の短縮形です。</div>

### ベクトル

<div><b>ベクトル</b>は<span>\(A^n\)</span>または<span>\(A^\ast\)</span>の束縛された表現です。(<span>\(A\)</span>は値や複合型です。)</div>
<div><span>\(2^{32}-1\)</span>要素までベクトルは持てます。</div>

<div>
\[\begin{split}\begin{array}{lllll}
\def\mathdef1445#1{{}}\mathdef1445{vector} &amp; {\mathit{vec}}(A) &amp;::=&amp;
  A^n
  &amp; (\mathrel{\mbox{if}} n &lt; 2^{32})\\
\end{array}\end{split}\]</div>

# 値

## Byte

値の最も単純な形式は、解釈されない生のbyteです。抽象構文では、これらは16進リテラルとして表現されます。

### 表記上のお約束

 - メタ変数`b`はbyteを表します。
 - byteはしばしばn &lt; 256を満たす自然数として表されます。

## 整数

異なる値範囲を持つ整数の異なるクラスは、そのビット幅`N`と符号なしか符号付きかによって区別されます。

<div>
\[\begin{split}\begin{array}{llll}
\def\mathdef1632#1{{}}\mathdef1632{unsigned integer} &amp; {\mathit{u}N} &amp;::=&amp;
  0 ~|~ 1 ~|~ \dots ~|~ 2^N{-}1 \\
\def\mathdef1632#1{{}}\mathdef1632{signed integer} &amp; {\mathit{s}N} &amp;::=&amp;
  -2^{N-1} ~|~ \dots ~|~ {-}1 ~|~ 0 ~|~ 1 ~|~ \dots ~|~ 2^{N-1}{-}1 \\
\def\mathdef1632#1{{}}\mathdef1632{uninterpreted integer} &amp; {\mathit{i}N} &amp;::=&amp;
  {\mathit{u}N} \\
\end{array}\end{split}\]</div>

後者のクラスは解釈されない整数を定義し、その符号化の解釈はコンテキストに応じて変化します。
抽象構文では、これらは符号なしの値として表現されます。
しかし、一部の操作では2の補数解釈に基づいて符号付きに変換されます。

### 付記

この仕様で使用される主な整数型は、u32、u64、s32、s64、i8、i16、i32、i64です。
しかし他の整数型もまた浮動小数点数の定義などにおいて補助的な構文として使用されます。

### 表記上のお約束

<ul>
    <li>メタ変数<span>\(m, n, i\)</span>は整数型を表します。</li>
    <li>数値は上の文法のように単純な算術で表すことができます。整数型の演算<span>\(2^N\)</span>をシーケンス<span>\((1)^N\)</span>などから区別するため後者のシーケンス表現は括弧で括られることとします。</li>
</ul>

## 浮動小数点数

浮動小数点データは、[IEEE 754-2019](https://ieeexplore.ieee.org/document/8766229)のそれぞれのバイナリフォーマットに対応する32ビットまたは64ビットの値を表します。
<div>
各値は符号と大きさを持ちます。
大きさは<span>\(m_0.m_1m_2\dots m_M \cdot2^e\)</span>の正規数（e は指数、m は最大符号ビット m0 が 1 の符号）、または指数を最小値に固定して m0 を 0 とした非正規化数で表すことができます。
符号は2進数なので正規数は<span>\((1 + m\cdot 2^{-M}) \cdot 2^e\)</span>という形で表され、Mはmのビット幅で、非正規化数も同様です。
</div>

大きさは、無限大とNaNという特殊な値も考えられます。
NaN値は基礎となる2値表現の中の仮数ビットを記述するペイロードを持ちます。
Signaling NANとQuiet NaNの間には区別はありません。[^1]

<div>
\[\begin{split}\begin{array}{llcll}
\def\mathdef1632#1{{}}\mathdef1632{floating-point value} &amp; {\mathit{f}N} &amp;::=&amp;
  {+} {\mathit{f}\mathit{Nmag}} ~|~ {-} {\mathit{f}\mathit{Nmag}} \\
\def\mathdef1632#1{{}}\mathdef1632{floating-point magnitude} &amp; {\mathit{f}\mathit{Nmag}} &amp;::=&amp;
  (1 + {\mathit{u}M}\cdot 2^{-M}) \cdot 2^e &amp; (\mathrel{\mbox{if}} -2^{E-1}+2 \leq e \leq 2^{E-1}-1) \\ &amp;&amp;|&amp;
  (0 + {\mathit{u}M}\cdot 2^{-M}) \cdot 2^e &amp; (\mathrel{\mbox{if}} e = -2^{E-1}+2) \\ &amp;&amp;|&amp;
  \infty \\ &amp;&amp;|&amp;
  {\mathsf{nan}}(n) &amp; (\mathrel{\mbox{if}} 1 \leq n &lt; 2^M) \\
\end{array}\end{split}\]</div>

<div><span>\(M = {\mathrm{signif}}(N)\)</span>と<span>\(E = {\mathrm{expon}}(N)\)</span>という仮定を置きます。なおsignifとexponenは以下の引数と戻り値の組み合わせを取ります。</div>

<div id="aux-exponent">
\[\begin{split}\begin{array}{lclllllcl}
{\mathrm{signif}}(32) &amp;=&amp; 23 &amp;&amp;&amp;&amp;
{\mathrm{expon}}(32) &amp;=&amp; 8 \\
{\mathrm{signif}}(64) &amp;=&amp; 52 &amp;&amp;&amp;&amp;
{\mathrm{expon}}(64) &amp;=&amp; 11 \\
\end{array}\end{split}\]
</div>

<div>canonical NaNは浮動小数点値±nan(<span>\(\pm{\mathsf{nan}}({\mathrm{canon}}_N)\)</span>)であり、<span>\(\pm{\mathsf{nan}}({\mathrm{canon}}_N)\)</span>は最上位ビットが1であり、他のビットはすべて0であるペイロードです。
<div>\[{\mathrm{canon}}_N = 2^{{\mathrm{signif}}(N)-1}\]</div>

算術NaNとは、n≧<span>\(\pm{\mathsf{nan}}({\mathrm{canon}}_N)\)</span>の浮動小数点値±nan(n)で、最上位ビットが1で、他のビットは任意の値を取るものです。
</div>

### 付記

抽象構文では、非正規化数は記号の先頭の0によって区別されます。
非正規化数の指数は、正規数の最小の指数と同じ値を持ちます。
2進表現でのみ、非正規化数の指数は、任意の正規数の指数とは異なる符号化が行われます。

### 表記上のお約束

<ul>
    <li>メタ変数<span>\(z\)</span>は浮動小数点数型を表します。</li>
</ul>

## 名前

名前は、[Unicode](http://www.unicode.org/versions/latest/)（2.4節）で定義されているスカラー値の文字列です。

<div>
\[\begin{split}\begin{array}{llclll}
\def\mathdef1632#1{{}}\mathdef1632{name} &amp; {\mathit{name}} &amp;::=&amp;
  {\mathit{char}}^\ast \qquad\qquad (\mathrel{\mbox{if}} |{\mathrm{utf8}}({\mathit{char}}^\ast)| &lt; 2^{32}) \\
\def\mathdef1632#1{{}}\mathdef1632{character} &amp; {\mathit{char}} &amp;::=&amp;
  \def\mathdef1671#1{\mathrm{U{+}#1}}\mathdef1671{00} ~|~ \dots ~|~ \def\mathdef1672#1{\mathrm{U{+}#1}}\mathdef1672{D7FF} ~|~
  \def\mathdef1673#1{\mathrm{U{+}#1}}\mathdef1673{E000} ~|~ \dots ~|~ \def\mathdef1674#1{\mathrm{U{+}#1}}\mathdef1674{10FFFF} \\
\end{array}\end{split}\]</div>

Binary Format仕様の制限により、名前の長さはUTF-8エンコーディングの長さに制限されます。

### 表記上のお約束

文字（Unicodeスカラ値）は、自然数`n`<1114112と暗黙的に同一であると見做し得ます。

# 型

`WebAssembly`の様々なエンティティは型によって分類されます。型は、検証、インスタンス化、および場合によっては実行時にチェックされます。

## 値の取り得る型

`WebAssembly`コードが計算できる個々の値と、変数が受け入れる値を分類します。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1595#1{{}}\mathdef1595{value type} &amp; {\mathit{valtype}} &amp;::=&amp;
  {\mathsf{i32}} ~|~ {\mathsf{i64}} ~|~ {\mathsf{f32}} ~|~ {\mathsf{f64}} \\
\end{array}\end{split}\]</div>

`i32`と`i64`は、それぞれ32ビットと64ビットの整数を分類します。
整数表現は符号の有無とは無縁であり、その解釈は個々の演算によって決定されます。

`f32`と`f64`はそれぞれ32ビットと64ビットの浮動小数点データを分類します。
これらは、[IEEE 754-2019](https://ieeexplore.ieee.org/document/8766229)で定義されている単精度および倍精度としても知られる浮動小数点表現に対応しています。

### 表記上のお約束

メタ変数`t`は、文脈から明らかな値の型の範囲を表します。

表記法｜t｜は値の型のビット幅を表します。つまり、｜i32｜=｜f32｜=32、｜i64｜=｜f64｜=64です。

## 戻り値の取り得る型

`戻り値型`は命令や関数を実行した結果の返り値の型です。
角括弧\[\]で囲われた型の[ベクトル](#ベクトル)です。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1595#1{{}}\mathdef1595{result type} &amp; {\mathit{resulttype}} &amp;::=&amp;
  [{\mathit{vec}}({\mathit{valtype}})] \\
\end{array}\end{split}\]</div>

## 関数の取り得る型

関数の型は関数のシグネチャです。
引数の型の[ベクトル](#ベクトル)を戻り値の型の[ベクトル](#ベクトル)に射影します。
また、命令の入力と出力を分類するためにも使用されます。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1595#1{{}}\mathdef1595{function type} &amp; {\mathit{functype}} &amp;::=&amp;
  {\mathit{resulttype}} {\rightarrow} {\mathit{resulttype}} \\
\end{array}\end{split}\]</div>

## リミット

リミットは、[メモリ型](#メモリ型)と[テーブル型](#テーブル型)に関連付けられたサイズ変更可能なストレージのサイズ範囲です。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1595#1{{}}\mathdef1595{limits} &amp; {\mathit{limits}} &amp;::=&amp;
  \{ {\mathsf{min}}~{\mathit{u32}}, {\mathsf{max}}~{\mathit{u32}}^? \} \\
\end{array}\end{split}\]</div>

上限を指定しない場合、環境が許す範囲において任意の大きさを取り得ます。

## メモリ型

メモリ型は、リニアメモリとそのサイズ範囲です。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1595#1{{}}\mathdef1595{memory type} &amp; {\mathit{memtype}} &amp;::=&amp;
  {\mathit{limits}} \\
\end{array}\end{split}\]</div>

リミットは、メモリの最小サイズと最大サイズを指定します。
リミットの数値の単位は[ページサイズ](Execution#ページサイズ)です。

<details><summary>訳者注:Thread</summary><div><div>\[\begin{split}\begin{array}{llll}
\def\mathdef2404#1{{}}\mathdef2404{memory type} &amp; {\mathit{memtype}} &amp;::=&amp;
  {\mathit{limits}}~{\mathit{share}} \\
\def\mathdef2404#1{{}}\mathdef2404{share} &amp; {\mathit{share}} &amp;::=&amp;
  {\mathsf{shared}} ~|~
  {\mathsf{unshared}} \\
\end{array}\end{split}\]</div>

メモリが`shared`であるか否か(複数のスレッドからアクセス可能な共有メモリであるか否か)を決めています。

</div></details>

## テーブル型

テーブル型は、テーブルを要素の型とサイズで分類するものです。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1595#1{{}}\mathdef1595{table type} &amp; {\mathit{tabletype}} &amp;::=&amp;
  {\mathit{limits}}~{\mathit{elemtype}} \\
\def\mathdef1595#1{{}}\mathdef1595{element type} &amp; {\mathit{elemtype}} &amp;::=&amp;
  {\mathsf{funcref}} \\
\end{array}\end{split}\]</div>

[メモリ型](#メモリ型)と同様に、テーブル型は最小サイズと最大サイズを示す[リミット](#リミット)によって制約を施されています。
リミットの数値の単位はテーブルの要素数です。

要素型の`funcref`は，すべての関数型の無限和です。
したがってこの`funcref`型のテーブルには関数への参照が含まれます。

### 付記

`WebAssembly`の将来のバージョンでは、他の要素型が導入されるかもしれません。

## グローバル型

グローバル型は、値を保持するグローバル変数の型です。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1595#1{{}}\mathdef1595{global type} &amp; {\mathit{globaltype}} &amp;::=&amp;
  {\mathit{mut}}~{\mathit{valtype}} \\
\def\mathdef1595#1{{}}\mathdef1595{mutability} &amp; {\mathit{mut}} &amp;::=&amp;
  {\mathsf{const}} ~|~
  {\mathsf{var}} \\
\end{array}\end{split}\]</div>

## 外部型

外部型は、[Import](#Import)と外部値を表す型です。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1595#1{{}}\mathdef1595{external types} &amp; {\mathit{externtype}} &amp;::=&amp;
  {\mathsf{func}}~{\mathit{functype}} ~|~
  {\mathsf{table}}~{\mathit{tabletype}} ~|~
  {\mathsf{mem}}~{\mathit{memtype}} ~|~
  {\mathsf{global}}~{\mathit{globaltype}} \\
\end{array}\end{split}\]</div>

### お約束

以下の補助的表記法は、外部型のシーケンスに対して定義されています。
これは特定の種類のエントリを順序を保持してフィルタリングします。

<ul>
  <li><span>\({\mathrm{funcs}}({\mathit{externtype}}^\ast) = [{\mathit{functype}} ~|~ ({\mathsf{func}}~{\mathit{functype}}) \in {\mathit{externtype}}^\ast]\)</span></li>
  <li><span>\({\mathrm{tables}}({\mathit{externtype}}^\ast) = [{\mathit{tabletype}} ~|~ ({\mathsf{table}}~{\mathit{tabletype}}) \in {\mathit{externtype}}^\ast]\)</span></li>
  <li><span>\({\mathrm{mems}}({\mathit{externtype}}^\ast) = [{\mathit{memtype}} ~|~ ({\mathsf{mem}}~{\mathit{memtype}}) \in {\mathit{externtype}}^\ast]\)</span></li>
  <li><span>\({\mathrm{globals}}({\mathit{externtype}}^\ast) = [{\mathit{globaltype}} ~|~ ({\mathsf{global}}~{\mathit{globaltype}}) \in {\mathit{externtype}}^\ast]\)</span></li>
</ul>

# 命令

`WebAssembly`のコードは命令のシーケンスで構成されています。
その計算モデルは命令が暗黙のオペランド・スタック上の値を操作し、引数の値を消費（ポップ）し、結果の値を生成または返す（プッシュ）という点で、スタック・マシンに基づいています。

スタックからの動的なオペランドに加えて、いくつかの命令は静的な即時引数、典型的にはインデックスや型のアノテーションを持ちます。
即時引数は命令を構成する一部です。

いくつかの命令は入れ子になった命令のシーケンスを括弧で囲むように構造化されています。

以下のセクションでは、命令をいくつかの異なるカテゴリに分類しています。

## 算術演算命令

算術演算命令は特定の型の数値に対する基本的な操作を提供します。
これらの操作はハードウェアで利用可能なそれぞれの操作と密接に一致しています。

<div>
\[\begin{split}\begin{array}{llcl}
\def\mathdef1519#1{{}}\mathdef1519{width} &amp; \mathit{nn}, \mathit{mm} &amp;::=&amp;
  \mathsf{32} ~|~ \mathsf{64} \\
\def\mathdef1519#1{{}}\mathdef1519{signedness} &amp; {\mathit{sx}} &amp;::=&amp;
  \mathsf{u} ~|~ \mathsf{s} \\
\def\mathdef1519#1{{}}\mathdef1519{instruction} &amp; {\mathit{instr}} &amp;::=&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{const}}~{\def\mathdef1556#1{{\mathit{i#1}}}\mathdef1556{\mathit{nn}}} ~|~
  \mathsf{f}\mathit{nn}\mathsf{.}{\mathsf{const}}~{\def\mathdef1557#1{{\mathit{f#1}}}\mathdef1557{\mathit{nn}}} \\&amp;&amp;|&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathit{iunop}} ~|~
  \mathsf{f}\mathit{nn}\mathsf{.}{\mathit{funop}} \\&amp;&amp;|&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathit{ibinop}} ~|~
  \mathsf{f}\mathit{nn}\mathsf{.}{\mathit{fbinop}} \\&amp;&amp;|&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathit{itestop}} \\&amp;&amp;|&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathit{irelop}} ~|~
  \mathsf{f}\mathit{nn}\mathsf{.}{\mathit{frelop}} \\&amp;&amp;|&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{extend}}\mathsf{8\_s} ~|~
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{extend}}\mathsf{16\_s} ~|~
  \mathsf{i64.}{\mathsf{extend}}\mathsf{32\_s} \\&amp;&amp;|&amp;
  \mathsf{i32.}{\mathsf{wrap}}\mathsf{\_i64} ~|~
  \mathsf{i64.}{\mathsf{extend}}\mathsf{\_i32}\mathsf{\_}{\mathit{sx}} ~|~
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{trunc}}\mathsf{\_f}\mathit{mm}\mathsf{\_}{\mathit{sx}} \\&amp;&amp;|&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{trunc}}\mathsf{\_sat\_f}\mathit{mm}\mathsf{\_}{\mathit{sx}} \\&amp;&amp;|&amp;
  \mathsf{f32.}{\mathsf{demote}}\mathsf{\_f64} ~|~
  \mathsf{f64.}{\mathsf{promote}}\mathsf{\_f32} ~|~
  \mathsf{f}\mathit{nn}\mathsf{.}{\mathsf{convert}}\mathsf{\_i}\mathit{mm}\mathsf{\_}{\mathit{sx}} \\&amp;&amp;|&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{reinterpret}}\mathsf{\_f}\mathit{nn} ~|~
  \mathsf{f}\mathit{nn}\mathsf{.}{\mathsf{reinterpret}}\mathsf{\_i}\mathit{nn} \\&amp;&amp;|&amp;
  \dots \\
\def\mathdef1519#1{{}}\mathdef1519{integer unary operator} &amp; {\mathit{iunop}} &amp;::=&amp;
  \mathsf{clz} ~|~
  \mathsf{ctz} ~|~
  \mathsf{popcnt} \\
\def\mathdef1519#1{{}}\mathdef1519{integer binary operator} &amp; {\mathit{ibinop}} &amp;::=&amp;
  \mathsf{add} ~|~
  \mathsf{sub} ~|~
  \mathsf{mul} ~|~
  \mathsf{div\_}{\mathit{sx}} ~|~
  \mathsf{rem\_}{\mathit{sx}} \\&amp;&amp;|&amp;
  \mathsf{and} ~|~
  \mathsf{or} ~|~
  \mathsf{xor} ~|~
  \mathsf{shl} ~|~
  \mathsf{shr\_}{\mathit{sx}} ~|~
  \mathsf{rotl} ~|~
  \mathsf{rotr} \\
\def\mathdef1519#1{{}}\mathdef1519{floating-point unary operator} &amp; {\mathit{funop}} &amp;::=&amp;
  \mathsf{abs} ~|~
  \mathsf{neg} ~|~
  \mathsf{sqrt} ~|~
  \mathsf{ceil} ~|~
  \mathsf{floor} ~|~
  \mathsf{trunc} ~|~
  \mathsf{nearest} \\
\def\mathdef1519#1{{}}\mathdef1519{floating-point binary operator} &amp; {\mathit{fbinop}} &amp;::=&amp;
  \mathsf{add} ~|~
  \mathsf{sub} ~|~
  \mathsf{mul} ~|~
  \mathsf{div} ~|~
  \mathsf{min} ~|~
  \mathsf{max} ~|~
  \mathsf{copysign} \\
\def\mathdef1519#1{{}}\mathdef1519{integer test operator} &amp; {\mathit{itestop}} &amp;::=&amp;
  \mathsf{eqz} \\
\def\mathdef1519#1{{}}\mathdef1519{integer relational operator} &amp; {\mathit{irelop}} &amp;::=&amp;
  \mathsf{eq} ~|~
  \mathsf{ne} ~|~
  \mathsf{lt\_}{\mathit{sx}} ~|~
  \mathsf{gt\_}{\mathit{sx}} ~|~
  \mathsf{le\_}{\mathit{sx}} ~|~
  \mathsf{ge\_}{\mathit{sx}} \\
\def\mathdef1519#1{{}}\mathdef1519{floating-point relational operator} &amp; {\mathit{frelop}} &amp;::=&amp;
  \mathsf{eq} ~|~
  \mathsf{ne} ~|~
  \mathsf{lt} ~|~
  \mathsf{gt} ~|~
  \mathsf{le} ~|~
  \mathsf{ge} \\
\end{array}\end{split}\]</div>

算術演算命令は、引数の値の型ごとにいくつかに分けられます。

- 定数
  - 静的に定義された定数を返します。
- 単項演算
  - 1つのオペランドを消費し、それぞれの型の1つの結果を返します。
- 二項演算
  - 2つのオペランドを消費し、それぞれの型の1つの結果を返します。
- テスト
  - 1つのオペランドを消費し、真偽値の1つの結果を返します。
- 比較
  - 2つのオペランドを消費し、真偽値の1つの結果を返します。
- 変換

整数に関連する命令の中には、符号化アノテーションsxによって、オペランドが符号なし整数として解釈されるか、符号付き整数として解釈されるかが区別されるものがあります。
その他の整数に関連する命令では、符号付き解釈に2の補数を使用することで、符号の有無に関係なく同じ動作をします。

### 表記上のお約束

以下の文法の短縮記法に従って演算子をグループ化しておくと便利です。

<div>
\[\begin{split}\begin{array}{llll}
\def\mathdef1519#1{{}}\mathdef1519{unary operator} &amp; {\mathit{unop}} &amp;::=&amp;
  {\mathit{iunop}} ~|~
  {\mathit{funop}} ~|~
  {\mathsf{extend}}{N}\mathsf{\_s} \\
\def\mathdef1519#1{{}}\mathdef1519{binary operator} &amp; {\mathit{binop}} &amp;::=&amp; {\mathit{ibinop}} ~|~ {\mathit{fbinop}} \\
\def\mathdef1519#1{{}}\mathdef1519{test operator} &amp; {\mathit{testop}} &amp;::=&amp; {\mathit{itestop}} \\
\def\mathdef1519#1{{}}\mathdef1519{relational operator} &amp; {\mathit{relop}} &amp;::=&amp; {\mathit{irelop}} ~|~ {\mathit{frelop}} \\
\def\mathdef1519#1{{}}\mathdef1519{conversion operator} &amp; {\mathit{cvtop}} &amp;::=&amp;
  {\mathsf{wrap}} ~|~
  {\mathsf{extend}} ~|~
  {\mathsf{trunc}} ~|~
  {\mathsf{trunc}}\mathsf{\_sat} ~|~
  {\mathsf{convert}} ~|~
  {\mathsf{demote}} ~|~
  {\mathsf{promote}} ~|~
  {\mathsf{reinterpret}} \\
\end{array}\end{split}\]</div>

## パラメトリック命令

このグループの命令は、任意の値型のオペランドを操作することができます。

<div>
\[\begin{split}\begin{array}{llcl}
\def\mathdef1519#1{{}}\mathdef1519{instruction} &amp; {\mathit{instr}} &amp;::=&amp;
  \dots \\&amp;&amp;|&amp;
  {\mathsf{drop}} \\&amp;&amp;|&amp;
  {\mathsf{select}}
\end{array}\end{split}\]</div>

- `drop`
  - スタック上に存在する値を1つ廃棄します。
- `select`
  - 3番目のオペランドがゼロかどうかで、最初の2つのオペランドのうちの1つを選択します。
  - 詳細については[実行/命令/パラメトリック命令/select](Execute#select)を読んでください。

## 変数命令

変数命令は、ローカル変数やグローバル変数に関するものです。

<div>
\[\begin{split}\begin{array}{llcl}
\def\mathdef1519#1{{}}\mathdef1519{instruction} &amp; {\mathit{instr}} &amp;::=&amp;
  \dots \\&amp;&amp;|&amp;
  {\mathsf{local.get}}~{\mathit{localidx}} \\&amp;&amp;|&amp;
  {\mathsf{local.set}}~{\mathit{localidx}} \\&amp;&amp;|&amp;
  {\mathsf{local.tee}}~{\mathit{localidx}} \\&amp;&amp;|&amp;
  {\mathsf{global.get}}~{\mathit{globalidx}} \\&amp;&amp;|&amp;
  {\mathsf{global.set}}~{\mathit{globalidx}} \\
\end{array}\end{split}\]</div>

これらの命令はそれぞれ変数の値を取得または設定します。

local.tee命令はlocal.setと似ていますが、引数を返します。
詳細については[実行/命令/変数命令/local.tee](Execute#local.tee)を読んでください。

## メモリ命令

メモリ命令はリニアメモリに関するものです。

<div>
\[\begin{split}\begin{array}{llcl}
\def\mathdef1519#1{{}}\mathdef1519{memory immediate} &amp; {\mathit{memarg}} &amp;::=&amp;
  \{ {\mathsf{offset}}~{\mathit{u32}}, {\mathsf{align}}~{\mathit{u32}} \} \\
\def\mathdef1519#1{{}}\mathdef1519{instruction} &amp; {\mathit{instr}} &amp;::=&amp;
  \dots \\&amp;&amp;|&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{load}}~{\mathit{memarg}} ~|~
  \mathsf{f}\mathit{nn}\mathsf{.}{\mathsf{load}}~{\mathit{memarg}} \\&amp;&amp;|&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{store}}~{\mathit{memarg}} ~|~
  \mathsf{f}\mathit{nn}\mathsf{.}{\mathsf{store}}~{\mathit{memarg}} \\&amp;&amp;|&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{load}}\mathsf{8\_}{\mathit{sx}}~{\mathit{memarg}} ~|~
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{load}}\mathsf{16\_}{\mathit{sx}}~{\mathit{memarg}} ~|~
  \mathsf{i64.}{\mathsf{load}}\mathsf{32\_}{\mathit{sx}}~{\mathit{memarg}} \\&amp;&amp;|&amp;
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{store}}\mathsf{8}~{\mathit{memarg}} ~|~
  \mathsf{i}\mathit{nn}\mathsf{.}{\mathsf{store}}\mathsf{16}~{\mathit{memarg}} ~|~
  \mathsf{i64.}{\mathsf{store}}\mathsf{32}~{\mathit{memarg}} \\&amp;&amp;|&amp;
  {\mathsf{memory.size}} \\&amp;&amp;|&amp;
  {\mathsf{memory.grow}} \\
\end{array}\end{split}\]</div>

メモリへのアクセスはロード命令とストア命令で行われます。
値の型に応じて命令は用意されています。

メモリ命令は即時に`memarg`で指定されたメモリを取ります。
`memarg`は`u32`型のoffsetと2の累乗の指数で表されるalignで構成されます。

整数のロードとストアでは、オプションでそれぞれの値型のビット幅よりも小さいストレージサイズを指定することができます。
ロードの場合には、適切な動作を選択するために符号拡張モード sx が必要となります。

静的なアドレスのoffsetが実行時に得られるオペランドアドレスに追加され、メモリがアクセスされるべき33ビットの有効アドレスが得られます。

すべての値はリトルエンディアンのバイト順で読み書きされます。
アクセスされたメモリ・バイトのいずれかが、メモリの現在のサイズによって暗示されるアドレス範囲外にある場合トラップ例外が発生します。

- memory.size 命令はメモリの現在のサイズを返します。
- memory.grow命令は、与えられたデルタ分だけメモリを成長させ、前のサイズを返します。どちらの命令もページサイズの単位で動作します。

### 付記

`WebAssembly`の将来のバージョンでは、64ビットのアドレス範囲を持つメモリ命令が提供されるかもしれません。

現在のバージョンの`WebAssembly`では、すべてのメモリ命令は暗黙的にメモリインデックス0として動作します。
この制約は将来のバージョンで除かれる可能性があります。

## 制御命令

制御命令は制御の流れに影響を与えるものです。

<div>
\[\begin{split}\begin{array}{llcl}
\def\mathdef1519#1{{}}\mathdef1519{block type} &amp; {\mathit{blocktype}} &amp;::=&amp;
  {\mathit{typeidx}} ~|~ {\mathit{valtype}}^? \\
\def\mathdef1519#1{{}}\mathdef1519{instruction} &amp; {\mathit{instr}} &amp;::=&amp;
  \dots \\&amp;&amp;|&amp;
  {\mathsf{nop}} \\&amp;&amp;|&amp;
  {\mathsf{unreachable}} \\&amp;&amp;|&amp;
  {\mathsf{block}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}} \\&amp;&amp;|&amp;
  {\mathsf{loop}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}} \\&amp;&amp;|&amp;
  {\mathsf{if}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{else}}~{\mathit{instr}}^\ast~{\mathsf{end}} \\&amp;&amp;|&amp;
  {\mathsf{br}}~{\mathit{labelidx}} \\&amp;&amp;|&amp;
  {\mathsf{br\_if}}~{\mathit{labelidx}} \\&amp;&amp;|&amp;
  {\mathsf{br\_table}}~{\mathit{vec}}({\mathit{labelidx}})~{\mathit{labelidx}} \\&amp;&amp;|&amp;
  {\mathsf{return}} \\&amp;&amp;|&amp;
  {\mathsf{call}}~{\mathit{funcidx}} \\&amp;&amp;|&amp;
  {\mathsf{call\_indirect}}~{\mathit{typeidx}} \\
\end{array}\end{split}\]</div>

- `nop`は何もしません。[^2]
- `unreachable`は無条件にトラップ例外を発生させます。
- ブロック命令、ループ命令、`if`命令は構造化された命令です。これらは、ブロックと呼ばれる命令の入れ子になったシーケンスを括弧で括り、`end`命令や`else`命令で終わるか、または擬似命令で区切られています。文法で規定されているように、これらの命令はうまく入れ子になっていなければなりません。

<div>構造化命令はアノテーションされたブロック型に従ってオペランドスタック上で入力を消費し、出力を生成することができます。
構造化命令は、適切な関数型を参照する型インデックスとして、または埋め込まれた値型として表記されます。(埋め込まれた値型は<span>\([] {\rightarrow} [{\mathit{valtype}}^?]\)</span>の短縮記法です。)</div>

各構造化制御命令は暗黙のラベルを導入します。
ラベルは、ラベルインデックスで識別され、ラベルを参照する分岐命令のターゲットとなります。

他のインデックス空間とは異なり、ラベルのインデックス付けは入れ子の深さによる相対的なものです。
ラベル0は参照する分岐命令を囲んでいる最も内側の構造化制御命令を参照します。
インデックスが増加するとより遠くにあるものを参照します。

その結果、ラベルは関連する構造化制御命令内からしか参照できません。
これはまた、分岐は対象となる制御構造のブロックから「break」して外側にのみ指示することができることを意味します。正確な挙動はその制御命令次第です。
ブロックの場合、または前方へのジャンプであれば、マッチング終了後に実行を再開します。
ループの場合は、ループの先頭への後方ジャンプです。

分岐命令にはいくつかの種類があります。

- `br`は無条件分岐
- `br_if`は条件分岐
- `br_table`はラベルベクトルを分岐先とします。オペランドがラベルベクトルの範囲外の時defaultラベルに分岐します。
- `return`は最外層のブロック(暗黙的には関数)を無条件で脱するための短縮命令です。スタックを関数呼び出し前の状態に巻き戻します。関数定義に戻り値が定義されている場合戻り値を巻き戻し後にスタックにpushします。

前方分岐は対象となるブロックの型の出力に応じたオペランドを要求します。
後方分岐は対象ブロックの型の入力に応じたオペランドを要求し、再起動されたブロックによって消費された値を表します。

`call`命令は別の関数を呼び出し、スタックから必要な引数を消費し、呼び出しの結果値を返します。

`call_indirect`命令は、オペランドをindex扱いしてテーブルから間接的に関数を呼び出します。
テーブルには`funcref`型の関数がありますが、呼び出したい関数のシグネチャと異なるシグネチャの関数が含まれている可能性があるため、呼び出し元は命令の`immediate`によってインデキシングされた関数型と実行時に照合され、一致しない場合はトラップ例外が発生して呼び出しが中止されます。

### 付記

ラベルインデックスのこの仕様により、構造化された制御フローが強制されます。
直感的には、ブロックやifを対象とした分岐は、ほとんどのC言語ではbreak文のように動作し、ループを対象とした分岐はcontinue文のように動作します。

現在のバージョンの`WebAssembly`では、`call_indirect`は暗黙的に0番目のテーブルを操作します。
この制限は将来のバージョンでは解除されるかもしれません。

## 式

関数本体、グローバルの初期化値、要素または[Dataセグメント](#Dataセグメント)のoffsetは式として与えられます。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1519#1{{}}\mathdef1519{expression} &amp; {\mathit{expr}} &amp;::=&amp;
  {\mathit{instr}}^\ast~{\mathsf{end}} \\
\end{array}\end{split}\]</div>

バリデーションによって式が定数に制限される場所では、許容される命令の集合が制限されています。

# モジュール

`WebAssembly`のプログラムは、デプロイ、ロード、コンパイルの単位であるモジュールにより構成されています。
モジュールは型・関数・テーブル・メモリ・グローバルの定義を収集します。
さらに、[Import](#Import)と[Export](#Export)を宣言します。
DataとElementのセグメントまたは開始関数を援用して初期化ロジックを提供します。

<div>
\[\begin{split}\begin{array}{lllll}
\def\mathdef1558#1{{}}\mathdef1558{module} &amp; {\mathit{module}} &amp;::=&amp; \{ &amp;
  {\mathsf{types}}~{\mathit{vec}}({\mathit{functype}}), \\&amp;&amp;&amp;&amp;
  {\mathsf{funcs}}~{\mathit{vec}}({\mathit{func}}), \\&amp;&amp;&amp;&amp;
  {\mathsf{tables}}~{\mathit{vec}}({\mathit{table}}), \\&amp;&amp;&amp;&amp;
  {\mathsf{mems}}~{\mathit{vec}}({\mathit{mem}}), \\&amp;&amp;&amp;&amp;
  {\mathsf{globals}}~{\mathit{vec}}({\mathit{global}}), \\&amp;&amp;&amp;&amp;
  {\mathsf{elem}}~{\mathit{vec}}({\mathit{elem}}), \\&amp;&amp;&amp;&amp;
  {\mathsf{data}}~{\mathit{vec}}({\mathit{data}}), \\&amp;&amp;&amp;&amp;
  {\mathsf{start}}~{\mathit{start}}^?, \\&amp;&amp;&amp;&amp;
  {\mathsf{imports}}~{\mathit{vec}}({\mathit{import}}), \\&amp;&amp;&amp;&amp;
  {\mathsf{exports}}~{\mathit{vec}}({\mathit{export}}) \quad\} \\
\end{array}\end{split}\]</div>

それぞれのベクトル(やモジュール全体)は空になり得ます。

## インデックス

モジュールの各定義は0から始まるインデックスで参照できます。

<div>
\[\begin{split}\begin{array}{llll}
\def\mathdef1558#1{{}}\mathdef1558{type index} &amp; {\mathit{typeidx}} &amp;::=&amp; {\mathit{u32}} \\
\def\mathdef1558#1{{}}\mathdef1558{function index} &amp; {\mathit{funcidx}} &amp;::=&amp; {\mathit{u32}} \\
\def\mathdef1558#1{{}}\mathdef1558{table index} &amp; {\mathit{tableidx}} &amp;::=&amp; {\mathit{u32}} \\
\def\mathdef1558#1{{}}\mathdef1558{memory index} &amp; {\mathit{memidx}} &amp;::=&amp; {\mathit{u32}} \\
\def\mathdef1558#1{{}}\mathdef1558{global index} &amp; {\mathit{globalidx}} &amp;::=&amp; {\mathit{u32}} \\
\def\mathdef1558#1{{}}\mathdef1558{local index} &amp; {\mathit{localidx}} &amp;::=&amp; {\mathit{u32}} \\
\def\mathdef1558#1{{}}\mathdef1558{label index} &amp; {\mathit{labelidx}} &amp;::=&amp; {\mathit{u32}} \\
\end{array}\end{split}\]</div>

関数、テーブル、メモリ、およびグローバルのインデックス空間には、同じモジュールで宣言されたそれぞれのImportが含まれます。
それぞれのImportはモジュールで定義されたそれぞれよりもインデックスが若いです。

ローカルのインデックス空間は、関数内からのみアクセス可能です
その関数の引数が先にインデックスを付与され、続いてローカル変数にインデックスが割り付けられます。

ラベルインデックスは、命令シーケンス内の[構造化制御命令](#制御命令)を参照します。

### 表記上のお約束

メタ変数`l`はラベルインデックスです。

メタ変数`x`,`y`は、他のインデックス空間のインデックスです。

## 型

モジュールの`types`コンポーネントは、関数型のベクトルを定義します。

モジュールで使用されるすべての関数型は、この`types`コンポーネント内で定義されなければなりません。
これらの型は型インデックスによって参照されます。

### 付記

`WebAssembly`の将来のバージョンでは、型定義の形式が追加される可能性があります。

## 関数

モジュールの`funcs`コンポーネントは、以下の構造を持つ関数のベクトルを定義します。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1558#1{{}}\mathdef1558{function} &amp; {\mathit{func}} &amp;::=&amp;
  \{ {\mathsf{type}}~{\mathit{typeidx}}, {\mathsf{locals}}~{\mathit{vec}}({\mathit{valtype}}), {\mathsf{body}}~{\mathit{expr}} \} \\
\end{array}\end{split}\]</div>

関数の`type`は、`typeidx`を通してシグネチャを宣言します。（モジュールで定義された型のみシグネチャと出来ます）。
関数の引数は関数固有の0から始まるローカルインデックスを通して参照されます。

`locals`ではmutableなローカル変数を型付きで宣言します。
これらの変数はローカルインデックスを通して参照されます。
最初のローカル変数のインデックスは引数の個数に合致します(引数に付与されていない最も小さい非負自然数)。

`body`は命令シーケンスです。
実行終了時にスタック上に関数の戻り値型ベクトルに合致する値を残している必要があります。

関数は、関数のImportを参照しない最小のインデックスから始まる関数インデックスを通して参照されます。

## テーブル

モジュールの`tables`コンポーネントは、テーブル型で記述されたテーブルのベクトルを定義します。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1558#1{{}}\mathdef1558{table} &amp; {\mathit{table}} &amp;::=&amp;
  \{ {\mathsf{type}}~{\mathit{tabletype}} \} \\
\end{array}\end{split}\]</div>

テーブルは特定のテーブル要素型の不透明な値のベクトルです。
テーブル型の[リミット](#リミット)の中の最小サイズはそのテーブルの初期サイズを指定します。
[リミット](#リミット)に最大サイズが存在する場合には後から成長できる最大サイズに上限を設定します。

テーブルはElementセグメントを通して初期化することができます。

テーブルはテーブルのImportを参照しない最小のインデックスから始まるテーブルインデックスを通して参照されます。
暗黙的にテーブルインデックス0を参照することが殆どです。

### 付記

現在のバージョンの`WebAssembly`では、1つのモジュールで最大1つのテーブルを定義またはImportすることができます。
モジュール初期化時にはこのテーブルの0番目を暗黙的に参照しています。

この制限は将来のバージョンでは除かれる可能性[^3]があります。

## メモリ

モジュールの`mems`コンポーネントは、メモリ型で記述されたリニアメモリ（略してメモリ）のベクトルを定義します。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1558#1{{}}\mathdef1558{memory} &amp; {\mathit{mem}} &amp;::=&amp;
  \{ {\mathsf{type}}~{\mathit{memtype}} \} \\
\end{array}\end{split}\]</div>

メモリは意味を問わない生のbyte列です。
メモリ型の[リミット](#リミット)における最小サイズはそのメモリの初期サイズを指定します。
[リミット](#リミット)に最大サイズが存在する場合には後から成長できる最大サイズに上限を設定します。
どちらもページサイズを単位としています。

メモリはDataセグメントを通して初期化することができます。

メモリはメモリのImportを参照しない最小のインデックスから始まるメモリインデックスを通して参照されます。
暗黙的にメモリインデックス0を参照することが殆どです。

### 付記

現在のバージョンの`WebAssembly`では、1つのモジュールで最大1つのメモリを定義またはImportすることができます。
モジュール初期化時にはこのメモリの0番目を暗黙的に参照しています。

この制限は将来のバージョンでは除かれる可能性[^3]があります。

## グローバル

モジュールの`globals`コンポーネントは、グローバル変数（略してグローバル）のベクトルを定義します。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1558#1{{}}\mathdef1558{global} &amp; {\mathit{global}} &amp;::=&amp;
  \{ {\mathsf{type}}~{\mathit{globaltype}}, {\mathsf{init}}~{\mathit{expr}} \} \\
\end{array}\end{split}\]</div>

各グローバルは、指定されたグローバル型の単一の値を格納します。また、その型は、グローバルが不変型か突然変異型かを指定します。さらに、各グローバルは、定数のイニシャライザ式で与えられた init 値で初期化されます。

グローバルはグローバルのImportを参照しない最小のインデックスから始まるグローバルインデックスを通して参照されます。

## Elementセグメント

[テーブル](#テーブル)の内容はデフォルトでは初期化されません。
モジュールの`elem`コンポーネントは、静的な要素のベクトルから与えられたテーブルの部分集合を初期化します。

<div>
\[\begin{split}\begin{array}{llll}
\def\mathdef1558#1{{}}\mathdef1558{element segment} &amp; {\mathit{elem}} &amp;::=&amp;
  \{ {\mathsf{table}}~{\mathit{tableidx}}, {\mathsf{offset}}~{\mathit{expr}}, {\mathsf{init}}~{\mathit{vec}}({\mathit{funcidx}}) \} \\
\end{array}\end{split}\]</div>

offsetは定数式です。

### 付記

現在のバージョンの`WebAssembly`では、1つのモジュールで最大1つのテーブルを定義またはImportすることができます。
故に現在唯一の有効な`tableidx`は0です。

## Dataセグメント

[メモリ](#メモリ)の初期内容は全て0のバイト列です。
モジュールの`data`コンポーネントは静的なバイト列から与えられたメモリの連続した部分を初期化します。

<div>
\[\begin{split}\begin{array}{llll}
\def\mathdef1558#1{{}}\mathdef1558{data segment} &amp; {\mathit{data}} &amp;::=&amp;
  \{ {\mathsf{data}}~{\mathit{memidx}}, {\mathsf{offset}}~{\mathit{expr}}, {\mathsf{init}}~{\mathit{vec}}({\mathit{byte}}) \} \\
\end{array}\end{split}\]</div>

offsetは定数式です。

### 付記

現在のバージョンの`WebAssembly`では、1つのモジュールで最大1つのメモリを定義またはImportすることができます。
故に現在唯一の有効な`memidx`は0です。

## 開始関数

モジュールの`start`コンポーネントは、テーブルとメモリが初期化された後、モジュールがインスタンス化されたときに自動的に起動される開始関数の関数インデックスを宣言します。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1558#1{{}}\mathdef1558{start function} &amp; {\mathit{start}} &amp;::=&amp;
  \{ {\mathsf{func}}~{\mathit{funcidx}} \} \\
\end{array}\end{split}\]</div>

### 付記

開始関数は、モジュールを初期化するためのものです。
初期化が完了するまではモジュールと`Export`は利用不能です。

## Export

モジュールの`exports`コンポーネントは、モジュールがインスタンス化されるとホスト環境からアクセス可能になるExportの集合を定義します。

<div>\[\begin{split}\begin{array}{llcl}
\def\mathdef1558#1{{}}\mathdef1558{export} &amp; {\mathit{export}} &amp;::=&amp;
  \{ {\mathsf{name}}~{\mathit{name}}, {\mathsf{desc}}~{\mathit{exportdesc}} \} \\
\def\mathdef1558#1{{}}\mathdef1558{export description} &amp; {\mathit{exportdesc}} &amp;::=&amp;
  {\mathsf{func}}~{\mathit{funcidx}} \\&amp;&amp;|&amp;
  {\mathsf{table}}~{\mathit{tableidx}} \\&amp;&amp;|&amp;
  {\mathsf{mem}}~{\mathit{memidx}} \\&amp;&amp;|&amp;
  {\mathsf{global}}~{\mathit{globalidx}} \\
\end{array}\end{split}\]</div>

各Exportは、ユニークな名前でラベル付けされます。
Export可能な定義は、関数、テーブル、メモリ、およびグローバルで、それぞれのインデックスを通して参照されます。

### 表記上のお約束

以下の補助表記法はExportのシーケンスに対して定義されています。
特定の種類のインデックスを順序を保持してフィルタリングします。

<ul>
  <li><span>\({\mathrm{funcs}}({\mathit{export}}^\ast) = [{\mathit{funcidx}} ~|~ {\mathsf{func}}~{\mathit{funcidx}} \in ({\mathit{export}}.{\mathsf{desc}})^\ast]\)</span></li>
  <li><span>\({\mathrm{tables}}({\mathit{export}}^\ast) = [{\mathit{tableidx}} ~|~ {\mathsf{table}}~{\mathit{tableidx}} \in ({\mathit{export}}.{\mathsf{desc}})^\ast]\)</span></li>
  <li><span>\({\mathrm{mems}}({\mathit{export}}^\ast) = [{\mathit{memidx}} ~|~ {\mathsf{mem}}~{\mathit{memidx}} \in ({\mathit{export}}.{\mathsf{desc}})^\ast]\)</span></li>
  <li><span>\({\mathrm{globals}}({\mathit{export}}^\ast) = [{\mathit{globalidx}} ~|~ {\mathsf{global}}~{\mathit{globalidx}} \in ({\mathit{export}}.{\mathsf{desc}})^\ast]\)</span></li>
</ul>

## Import

モジュールの`import`コンポーネントは、インスタンス化に必要なImportの集合を定義します。

<div>\[\begin{split}\begin{array}{llll}
\def\mathdef1558#1{{}}\mathdef1558{import} &amp; {\mathit{import}} &amp;::=&amp;
  \{ {\mathsf{module}}~{\mathit{name}}, {\mathsf{name}}~{\mathit{name}}, {\mathsf{desc}}~{\mathit{importdesc}} \} \\
\def\mathdef1558#1{{}}\mathdef1558{import description} &amp; {\mathit{importdesc}} &amp;::=&amp;
  {\mathsf{func}}~{\mathit{typeidx}} \\&amp;&amp;|&amp;
  {\mathsf{table}}~{\mathit{tabletype}} \\&amp;&amp;|&amp;
  {\mathsf{mem}}~{\mathit{memtype}} \\&amp;&amp;|&amp;
  {\mathsf{global}}~{\mathit{globaltype}} \\
\end{array}\end{split}\]</div>

各Importは2層からなる名前空間に所属します。モジュール名とそのモジュール内のエンティティの名前です。

Import可能な定義は、関数、テーブル、メモリ、およびグローバルです。

各Importは、インスタンス化時に提供される定義が一致する必要がある、それぞれの型インデックスを通して指定されます。

各Importは、それぞれのインデックス空間でインデックスを定義します。
各インデックス空間では、Importのインデックスは、モジュール自体に含まれる定義の最初のインデックスよりも前になります。

### 付記

Export名とは異なり、Import名は必ずしもユニークなものである必要はありません。

同じモジュール/名前の組み合わせを複数回Importすることも可能です。
そのようなImportは異なる種類のエンティティを含む異なる型の記述を持つこともありえるでしょう。

エンベッダがどう解釈するか次第ではありますが、このようなImportを持つモジュールをインスタンス化すること自体は可能です。
しかし、エンベッダはそのようなオーバーロードをサポートする必要はなく、`WebAssembly`の仕様自体もオーバーロードをサポートしていません。

# LINK

<footer>
  <nav>
    <ul>
      <li><a href="Introduction" rel="prev">Prev: はじめに</a></li>
      <li><a href="./">Top: Index</a></li>
      <li><a href="Validation" rel="next">Next: 検証</a></li>
    </ul>
    <a href="LICENSE" rel="license">LICENSE</a>
  </nav>
</footer>

[^1]: 訳注:SignalingNANは算術演算を行うと例外を発生させるNaNらしい。QuietNaNはNaNが伝播するらしい。
[^2]: 訳注:デバッガがブレークポイントを差し込むのに利用したりします。
[^3]: 訳注:[ほぼ確実にされる。](https://github.com/WebAssembly/design/blob/master/FutureFeatures.md#multiple-tables-and-memories)