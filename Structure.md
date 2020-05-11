<script async="async" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">MathJax.Hub.Config({"TeX": {"MAXBUFFER": 30720}})</script>

# 慣例

`WebAssembly`は複数の具体的な表現（そのバイナリ形式とテキスト形式）を持つプログラミング言語です。
両方とも共通の構造に対応しています。
簡潔さのためにこの構造は抽象構文の形で記述されています。
この仕様のすべての部分はこの抽象構文の観点から定義されています。

## 文法表記法

抽象構文の文法規則を定義する際には以下の規則を採用しています。

<ul class="simple">
    <li>終端記号（アトム）はサンセリフフォントで表記します: <span class="math notranslate nohighlight">\(\mathsf{i32}, \mathsf{end}\)</span>。</li>
    <li>非末端記号はイタリック体で表記します: <span class="math notranslate nohighlight">\(\mathit{valtype}, \mathit{instr}\)</span>。</li>
    <li><span class="math notranslate nohighlight">\(A^n\)</span>は<span class="math notranslate nohighlight">\(A\)</span>の<span class="math notranslate nohighlight">\(n\geq 0\)</span>回の繰り返しです。</li>
    <li><span class="math notranslate nohighlight">\(A^\ast\)</span>は<span class="math notranslate nohighlight">\(A\)の繰り返しの空のシーケンスの可能性があります（これは n が関係ない場合に使用される<span class="math notranslate nohighlight">\(A^n\)</span>の略記法です）。</span></li>
    <li><span class="math notranslate nohighlight">\(A^+\)</span>は<span class="math notranslate nohighlight">\(A\)</span> の繰り返しの空ではないシーケンスです（これは n≧1 の場合の<span class="math notranslate nohighlight">\(A^n\)</span>の速記法です）。</li>
    <li><span class="math notranslate nohighlight">\(A^?\)</span>は<span class="math notranslate nohighlight">\(A\)</span>が1つ以下存在するかもしれないことを示します。</li>
    <li>積和は<span class="math notranslate nohighlight">\(\mathit{sym} ::= A_1 ~|~ \dots ~|~ A_n\)</span>と表記します。</li>
    <li>大きな積和は複数の定義に分割されることがあり、連続体<span class="math notranslate nohighlight">\(\mathit{sym} ::= \dots ~|~ A_2\)</span>を省略記号で開始し、<span class="math notranslate nohighlight">\(\mathit{sym} ::= A_1 ~|~ \dots\)</span>で明示的に終わらせることで示されます。</li>
    <li>いくつかの積和は括弧内に付記された条件“<span class="math notranslate nohighlight">\((\mathrel{\mbox{if}} \mathit{condition})\)</span>”で補強されていますが、これは積和を多くの別々のケースに組み合わせて拡張するための略記法です。</li>
</ul>