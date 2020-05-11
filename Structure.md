<script async="async" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">MathJax.Hub.Config({"TeX": {"MAXBUFFER": 30720}})</script>

# 慣例

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

以下の形式の積和は、フィールド<span class="math notranslate nohighlight">\(\mathsf{field}_i\)</span>の固定セットをそれぞれ「値」<span>\(A_i\)</span>にマッピングするレコードとして解釈されます。

<footer>
    <nav>
        <a href="Introduction" rel="prev">Prev: Introduction</a>
        <a href="./">Top: Index</a>
    </nav>
</footer>