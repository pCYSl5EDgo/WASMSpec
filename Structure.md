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
    <li>Terminal symbols (atoms) are written in sans-serif font: <span class="math notranslate nohighlight">\(\mathsf{i32}, \mathsf{end}\)</span>.</li>
    <li>Nonterminal symbols are written in italic font: <span class="math notranslate nohighlight">\(\mathit{valtype}, \mathit{instr}\)</span>.</li>
    <li><span class="math notranslate nohighlight">\(A^n\)</span> is a sequence of <span class="math notranslate nohighlight">\(n\geq 0\)</span> iterations  of <span class="math notranslate nohighlight">\(A\)</span>.</li>
    <li><span class="math notranslate nohighlight">\(A^\ast\)</span> is a possibly empty sequence of iterations of <span class="math notranslate nohighlight">\(A\)</span>.
    (This is a shorthand for <span class="math notranslate nohighlight">\(A^n\)</span> used where <span class="math notranslate nohighlight">\(n\)</span> is not relevant.)</li>
    <li><span class="math notranslate nohighlight">\(A^+\)</span> is a non-empty sequence of iterations of <span class="math notranslate nohighlight">\(A\)</span>.
    (This is a shorthand for <span class="math notranslate nohighlight">\(A^n\)</span> where <span class="math notranslate nohighlight">\(n \geq 1\)</span>.)</li>
    <li><span class="math notranslate nohighlight">\(A^?\)</span> is an optional occurrence of <span class="math notranslate nohighlight">\(A\)</span>.
    (This is a shorthand for <span class="math notranslate nohighlight">\(A^n\)</span> where <span class="math notranslate nohighlight">\(n \leq 1\)</span>.)</li>
    <li>Productions are written <span class="math notranslate nohighlight">\(\mathit{sym} ::= A_1 ~|~ \dots ~|~ A_n\)</span>.</li>
    <li>Large productions may be split into multiple definitions, indicated by ending the first one with explicit ellipses, <span class="math notranslate nohighlight">\(\mathit{sym} ::= A_1 ~|~ \dots\)</span>, and starting continuations with ellipses, <span class="math notranslate nohighlight">\(\mathit{sym} ::= \dots ~|~ A_2\)</span>.</li>
    <li>Some productions are augmented with side conditions in parentheses, “<span class="math notranslate nohighlight">\((\mathrel{\mbox{if}} \mathit{condition})\)</span>”, that provide a shorthand for a combinatorial expansion of the production into many separate cases.</li>
</ul>