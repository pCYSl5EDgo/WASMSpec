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

<div>以下の形式の積和は、フィールド<span>\(\mathsf{field}_i\)</span>の固定セットをそれぞれ「値」<span>\(A_i\)</span>にマッピングするレコードとして解釈されます。</div>
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
\def\mathdef1632#1{{}}\mathdef1632{unsigned integer} &amp; \href{../syntax/values.html#syntax-int}{\mathit{u}N} &amp;::=&amp;
  0 ~|~ 1 ~|~ \dots ~|~ 2^N{-}1 \\
\def\mathdef1632#1{{}}\mathdef1632{signed integer} &amp; \href{../syntax/values.html#syntax-int}{\mathit{s}N} &amp;::=&amp;
  -2^{N-1} ~|~ \dots ~|~ {-}1 ~|~ 0 ~|~ 1 ~|~ \dots ~|~ 2^{N-1}{-}1 \\
\def\mathdef1632#1{{}}\mathdef1632{uninterpreted integer} &amp; \href{../syntax/values.html#syntax-int}{\mathit{i}N} &amp;::=&amp;
  \href{../syntax/values.html#syntax-int}{\mathit{u}N} \\
\end{array}\end{split}\]</div>

後者のクラスは解釈されない整数を定義し、その符号化の解釈はコンテキストに応じて変化します。
抽象構文では、これらは符号なしの値として表現されます。
しかし、一部の操作では2の補数解釈に基づいて符号付きに変換されます。

### 付記

この仕様で使用される主な整数型は、u32、u64、s32、s64、i8、i16、i32、i64です。
しかし他の整数型もまた浮動小数点数の定義などにおいて補助的な構文として使用されます。

### 表記上のお約束

<ul>
    <li>メタ変数<span class="math notranslate nohighlight">\(m, n, i\)</span>は整数型を表します。</li>
    <li>数値は上の文法のように単純な算術で表すことができます。整数型の演算<span class="math notranslate nohighlight">\(2^N\)</span>をシーケンス<span class="math notranslate nohighlight">\((1)^N\)</span>などから区別するため後者のシーケンス表現は括弧で括られることとします。</li>
</ul>

## 浮動小数点数

浮動小数点データは、IEEE 754-2019規格（セクション3.3）のそれぞれのバイナリフォーマットに対応する32ビットまたは64ビットの値を表します。

## 名前

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