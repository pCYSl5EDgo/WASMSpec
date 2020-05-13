<script async="async" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/latest.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/x-mathjax-config">MathJax.Hub.Config({"TeX": {"MAXBUFFER": 30720}})</script>

# 表記上のお約束

## コンテキスト

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

<ul>
    <li>When spelling out a context, empty fields are omitted.</li>
    <li><span>\(C,\mathsf{field}\,A^\ast\)</span> denotes the same context as <span>\(C\)</span> but with the elements <span>\(A^\ast\)</span> prepended to its <span>\(\mathsf{field}\)</span> component sequence.</li>
</ul>

### 付記

## 散文的表記法

### 付記

## 形式的表記法

<div>\[\frac{
  \mathit{premise}_1 \qquad \mathit{premise}_2 \qquad \dots \qquad \mathit{premise}_n
}{
  \mathit{conclusion}
}\]</div>

### 付記

<div>\[\frac{
}{
  C \vdash {\mathsf{i32}}.{\mathsf{add}} : [{\mathsf{i32}}~{\mathsf{i32}}] {\rightarrow} [{\mathsf{i32}}]
}\]</div>

<div>\[\frac{
  C.{\mathsf{locals}}[x] = t
}{
  C \vdash {\mathsf{local.get}}~x : [] {\rightarrow} [t]
}\]</div>

<div>\[\frac{
  C \vdash {\mathit{blocktype}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
  \qquad
  C,{\mathsf{label}}\,[t_2^\ast] \vdash {\mathit{instr}}^\ast : [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C \vdash {\mathsf{block}}~{\mathit{blocktype}}~{\mathit{instr}}^\ast~{\mathsf{end}} : [t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

# 型

## リミット

<h3><span>\(\{ {\mathsf{min}}~n, {\mathsf{max}}~m^? \}\)</span></h3>

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

<h3><span>\({\mathit{typeidx}}\)</span></h3>

<ul>
    <li>The type <span>\(C.{\mathsf{types}}[{\mathit{typeidx}}]\)</span> must be defined in the context.</li>
    <li>Then the block type is valid as <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\(C.{\mathsf{types}}[{\mathit{typeidx}}]\)</span>.</li>
</ul>

<div>\[\frac{
  C.{\mathsf{types}}[{\mathit{typeidx}}] = {\mathit{functype}}
}{
  C {\vdash} {\mathit{typeidx}} : {\mathit{functype}}
}\]</div>

<h3><span>\([{\mathit{valtype}}^?]\)</span></h3>

<ul>
    <li>The block type is valid as <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\([] {\rightarrow} [{\mathit{valtype}}^?]\)</span>.</li>
</ul>

<div>\[\frac{
}{
  C {\vdash} [{\mathit{valtype}}^?] : [] {\rightarrow} [{\mathit{valtype}}^?]
}\]</div>

## 関数型

<h3><span>\([t_1^n] {\rightarrow} [t_2^m]\)</span></h3>

<div>\[\frac{
}{
  {\vdash} [t_1^\ast] {\rightarrow} [t_2^\ast] \mathrel{\mbox{ok}}
}\]</div>

## テーブル型

<h3><span>\({\mathit{limits}}~{\mathit{elemtype}}\)</span></h3>

<ul>
    <li>The limits <span>\({\mathit{limits}}\)</span> must be <a class="reference internal" href="#valid-limits"><span class="std std-ref">valid</span></a> within range <span>\(2^{32}\)</span>.</li>
    <li>Then the table type is valid.</li>
</ul>

<div>\[\frac{
  {\vdash} {\mathit{limits}} : 2^{32}
}{
  {\vdash} {\mathit{limits}}~{\mathit{elemtype}} \mathrel{\mbox{ok}}
}\]</div>

## メモリ型

<h3><span>\({\mathit{limits}}\)</span></h3>

<ul>
    <li>The limits <span>\({\mathit{limits}}\)</span> must be <a class="reference internal" href="#valid-limits"><span class="std std-ref">valid</span></a> within range <span>\(2^{16}\)</span>.</li>
    <li>Then the memory type is valid.</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{limits}} : 2^{16}
}{
  {\vdash} {\mathit{limits}} \mathrel{\mbox{ok}}
}\]</div>

## グローバル型

<h3><span>\({\mathit{mut}}~{\mathit{valtype}}\)</span></h3>

<div>\[\frac{
}{
  {\vdash} {\mathit{mut}}~{\mathit{valtype}} \mathrel{\mbox{ok}}
}\]</div>


## 外部型

<h3><span>\({\mathsf{func}}~{\mathit{functype}}\)</span></h3>

<ul>
    <li>The <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\({\mathit{functype}}\)</span> must be <a class="reference internal" href="#valid-functype"><span class="std std-ref">valid</span></a>.</li>
    <li>Then the external type is valid.</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{functype}} \mathrel{\mbox{ok}}
}{
  {\vdash} {\mathsf{func}}~{\mathit{functype}} \mathrel{\mbox{ok}}
}\]</div>

<h3><span>\({\mathsf{table}}~{\mathit{tabletype}}\)</span></h3>

<ul>
    <li>The <a class="reference internal" href="../syntax/types.html#syntax-tabletype"><span class="std std-ref">table type</span></a> <span>\({\mathit{tabletype}}\)</span> must be <a class="reference internal" href="#valid-tabletype"><span class="std std-ref">valid</span></a>.</li>
    <li>Then the external type is valid.</li>
</ul>

<div>\[\frac{
  {\vdash} {\mathit{tabletype}} \mathrel{\mbox{ok}}
}{
  {\vdash} {\mathsf{table}}~{\mathit{tabletype}} \mathrel{\mbox{ok}}
}\]</div>
</div>

<h3><span>\({\mathsf{mem}}~{\mathit{memtype}}\)</span></h3>

<ul>
    <li>The <a class="reference internal" href="../syntax/types.html#syntax-memtype"><span class="std std-ref">memory type</span></a> <span>\({\mathit{memtype}}\)</span> must be <a class="reference internal" href="#valid-memtype"><span class="std std-ref">valid</span></a>.</li>
    <li>Then the external type is valid.</li>
</ul>

<div>\[\frac{
  {\vdash} {\mathit{memtype}} \mathrel{\mbox{ok}}
}{
  {\vdash} {\mathsf{mem}}~{\mathit{memtype}} \mathrel{\mbox{ok}}
}\]</div>

<h3><span>\({\mathsf{global}}~{\mathit{globaltype}}\)</span></h3>

<ul>
    <li>The <a class="reference internal" href="../syntax/types.html#syntax-globaltype"><span class="std std-ref">global type</span></a> <span>\({\mathit{globaltype}}\)</span> must be <a class="reference internal" href="#valid-globaltype"><span class="std std-ref">valid</span></a>.</li>
    <li>Then the external type is valid.</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{globaltype}} \mathrel{\mbox{ok}}
}{
  {\vdash} {\mathsf{global}}~{\mathit{globaltype}} \mathrel{\mbox{ok}}
}\]</div>

# 命令

<div><span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span></div>

### 付記

<div><span>\([{\mathsf{i32}}~{\mathsf{i32}}] {\rightarrow} [{\mathsf{i32}}]\)</span></div>


---

Typing extends to <a class="reference internal" href="#valid-instr-seq"><span class="std std-ref">instruction sequences</span></a> <span>\({\mathit{instr}}^\ast\)</span>.
Such a sequence has a <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> if the accumulative effect of executing the instructions is consuming values of types <span>\(t_1^\ast\)</span> off the operand stack and pushing new values of types <span>\(t_2^\ast\)</span>.
<p id="polymorphism">For some instructions, the typing rules do not fully constrain the type,
and therefore allow for multiple types.
Such instructions are called <em>polymorphic</em>.
Two degrees of polymorphism can be distinguished:
<ul>
    <li><em>value-polymorphic</em>:
the <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value type</span></a> <span>\(t\)</span> of one or several individual operands is unconstrained.
That is the case for all <a class="reference internal" href="#valid-instr-parametric"><span class="std std-ref">parametric instructions</span></a> like <span>\({\mathsf{drop}}\)</span> and <span>\({\mathsf{select}}\)</span>.</li>
    <li><em>stack-polymorphic</em>:
the entire (or most of the) <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> of the instruction is unconstrained.
That is the case for all <a class="reference internal" href="#valid-instr-control"><span class="std std-ref">control instructions</span></a> that perform an <em>unconditional control transfer</em>, such as <span>\({\mathsf{unreachable}}\)</span>, <span>\({\mathsf{br}}\)</span>, <span>\({\mathsf{br\_table}}\)</span>, and <span>\({\mathsf{return}}\)</span>.</li>
</ul>


### 付記

For example, the <span>\({\mathsf{select}}\)</span> instruction is valid with type <span>\([t~t~{\mathsf{i32}}] {\rightarrow} [t]\)</span>, for any possible <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value type</span></a> <span>\(t\)</span>.   Consequently, both instruction sequences
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
    <li>The instruction is valid with type <span>\([t] {\rightarrow} []\)</span>, for any <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value type</span></a> <span>\(t\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} {\mathsf{drop}} : [t] {\rightarrow} []
}\]</div>

<h3><span>\({\mathsf{select}}\)</span></h3>

<ul>
    <li>The instruction is valid with type <span>\([t~t~{\mathsf{i32}}] {\rightarrow} [t]\)</span>, for any <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value type</span></a> <span>\(t\)</span>.</li>
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
    <li>Let <span>\(t\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value type</span></a> <span>\(C.{\mathsf{locals}}[x]\)</span>.</li>
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
    <li>Let <span>\(t\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value type</span></a> <span>\(C.{\mathsf{locals}}[x]\)</span>.</li>
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
    <li>Let <span>\(t\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value type</span></a> <span>\(C.{\mathsf{locals}}[x]\)</span>.</li>
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
    <li>Let <span>\({\mathit{mut}}~t\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-globaltype"><span class="std std-ref">global type</span></a> <span>\(C.{\mathsf{globals}}[x]\)</span>.</li>
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
    <li>Let <span>\({\mathit{mut}}~t\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-globaltype"><span class="std std-ref">global type</span></a> <span>\(C.{\mathsf{globals}}[x]\)</span>.</li>
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
    <li>The alignment <span>\(2^{{\mathit{memarg}}.{\mathsf{align}}}\)</span> must not be larger than the <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">bit width</span></a> of <span>\(t\)</span> divided by <span>\(8\)</span>.</li>
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
    <li>The alignment <span>\(2^{{\mathit{memarg}}.{\mathsf{align}}}\)</span> must not be larger than the <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">bit width</span></a> of <span>\(t\)</span> divided by <span>\(8\)</span>.</li>
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
    <li>The instruction is valid with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>, for any sequences of <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value types</span></a> <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>.</li>
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
    <li>The <a class="reference internal" href="../syntax/instructions.html#syntax-blocktype"><span class="std std-ref">block type</span></a> must be <a class="reference internal" href="types.html#valid-blocktype"><span class="std std-ref">valid</span></a> as some <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
    <li>Let <span>\(C'\)</span> be the same <a class="reference internal" href="conventions.html#context"><span class="std std-ref">context</span></a> as <span>\(C\)</span>, but with the <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> <span>\([t_2^\ast]\)</span> prepended to the <span>\({\mathsf{labels}}\)</span> vector.</li>
    <li>Under context <span>\(C'\)</span>,
the instruction sequence <span>\({\mathit{instr}}^\ast\)</span> must be <a class="reference internal" href="#valid-instr-seq"><span class="std std-ref">valid</span></a> with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
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
    <li>The <a class="reference internal" href="../syntax/instructions.html#syntax-blocktype"><span class="std std-ref">block type</span></a> must be <a class="reference internal" href="types.html#valid-blocktype"><span class="std std-ref">valid</span></a> as some <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
    <li>Let <span>\(C'\)</span> be the same <a class="reference internal" href="conventions.html#context"><span class="std std-ref">context</span></a> as <span>\(C\)</span>, but with the <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> <span>\([t_1^\ast]\)</span> prepended to the <span>\({\mathsf{labels}}\)</span> vector.</li>
    <li>Under context <span>\(C'\)</span>,
the instruction sequence <span>\({\mathit{instr}}^\ast\)</span> must be <a class="reference internal" href="#valid-instr-seq"><span class="std std-ref">valid</span></a> with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
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
    <li>The <a class="reference internal" href="../syntax/instructions.html#syntax-blocktype"><span class="std std-ref">block type</span></a> must be <a class="reference internal" href="types.html#valid-blocktype"><span class="std std-ref">valid</span></a> as some <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
    <li>Let <span>\(C'\)</span> be the same <a class="reference internal" href="conventions.html#context"><span class="std std-ref">context</span></a> as <span>\(C\)</span>, but with the <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> <span>\([t_2^\ast]\)</span> prepended to the <span>\({\mathsf{labels}}\)</span> vector.</li>
    <li>Under context <span>\(C'\)</span>,
the instruction sequence <span>\({\mathit{instr}}_1^\ast\)</span> must be <a class="reference internal" href="#valid-instr-seq"><span class="std std-ref">valid</span></a> with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
    <li>Under context <span>\(C'\)</span>,
the instruction sequence <span>\({\mathit{instr}}_2^\ast\)</span> must be <a class="reference internal" href="#valid-instr-seq"><span class="std std-ref">valid</span></a> with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
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
    <li>Let <span>\([t^\ast]\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> <span>\(C.{\mathsf{labels}}[l]\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]\)</span>, for any sequences of <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value types</span></a> <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{labels}}[l] = [t^\ast]
}{
  C {\vdash} {\mathsf{br}}~l : [t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

### 付記

The <a class="reference internal" href="../syntax/modules.html#syntax-labelidx"><span class="std std-ref">label index</span></a> space in the <a class="reference internal" href="conventions.html#context"><span class="std std-ref">context</span></a> <span>\(C\)</span> contains the most recent label first, so that <span>\(C.{\mathsf{labels}}[l]\)</span> performs a relative lookup as expected.
The <span>\({\mathsf{br}}\)</span> instruction is <a class="reference internal" href="#polymorphism"><span class="std std-ref">stack-polymorphic</span></a>.

---

<h3><span>\({\mathsf{br\_if}}~l\)</span></h3>

<ul>
    <li>The label <span>\(C.{\mathsf{labels}}[l]\)</span> must be defined in the context.</li>
    <li>Let <span>\([t^\ast]\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> <span>\(C.{\mathsf{labels}}[l]\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t^\ast~{\mathsf{i32}}] {\rightarrow} [t^\ast]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{labels}}[l] = [t^\ast]
}{
  C {\vdash} {\mathsf{br\_if}}~l : [t^\ast~{\mathsf{i32}}] {\rightarrow} [t^\ast]
}\]</div>

### 付記

The <a class="reference internal" href="../syntax/modules.html#syntax-labelidx"><span class="std std-ref">label index</span></a> space in the <a class="reference internal" href="conventions.html#context"><span class="std std-ref">context</span></a> <span>\(C\)</span> contains the most recent label first, so that <span>\(C.{\mathsf{labels}}[l]\)</span> performs a relative lookup as expected.

---

<h3><span>\({\mathsf{br\_table}}~l^\ast~l_N\)</span></h3>

<ul>
    <li>The label <span>\(C.{\mathsf{labels}}[l_N]\)</span> must be defined in the context.</li>
    <li>Let <span>\([t^\ast]\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> <span>\(C.{\mathsf{labels}}[l_N]\)</span>.</li>
    <li>For all <span>\(l_i\)</span> in <span>\(l^\ast\)</span>,
the label <span>\(C.{\mathsf{labels}}[l_i]\)</span> must be defined in the context.</li>
    <li>For all <span>\(l_i\)</span> in <span>\(l^\ast\)</span>,
<span>\(C.{\mathsf{labels}}[l_i]\)</span> must be <span>\([t^\ast]\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t_1^\ast~t^\ast~{\mathsf{i32}}] {\rightarrow} [t_2^\ast]\)</span>, for any sequences of <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value types</span></a> <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>.</li>
</ul>
<div>\[\frac{
  (C.{\mathsf{labels}}[l] = [t^\ast])^\ast
  \qquad
  C.{\mathsf{labels}}[l_N] = [t^\ast]
}{
  C {\vdash} {\mathsf{br\_table}}~l^\ast~l_N : [t_1^\ast~t^\ast~{\mathsf{i32}}] {\rightarrow} [t_2^\ast]
}\]</div>

### 付記

The <a class="reference internal" href="../syntax/modules.html#syntax-labelidx"><span class="std std-ref">label index</span></a> space in the <a class="reference internal" href="conventions.html#context"><span class="std std-ref">context</span></a> <span>\(C\)</span> contains the most recent label first, so that <span>\(C.{\mathsf{labels}}[l_i]\)</span> performs a relative lookup as expected.
The <span>\({\mathsf{br\_table}}\)</span> instruction is <a class="reference internal" href="#polymorphism"><span class="std std-ref">stack-polymorphic</span></a>.

---

<h3><span>\({\mathsf{return}}\)</span></h3>

<ul>
    <li>The return type <span>\(C.{\mathsf{return}}\)</span> must not be absent in the context.</li>
    <li>Let <span>\([t^\ast]\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> of <span>\(C.{\mathsf{return}}\)</span>.</li>
    <li>Then the instruction is valid with type <span>\([t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]\)</span>, for any sequences of <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value types</span></a> <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{return}} = [t^\ast]
}{
  C {\vdash} {\mathsf{return}} : [t_1^\ast~t^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

### 付記

The <span>\({\mathsf{return}}\)</span> instruction is <a class="reference internal" href="#polymorphism"><span class="std std-ref">stack-polymorphic</span></a>.
<span>\(C.{\mathsf{return}}\)</span> is absent (set to <span>\(\epsilon\)</span>) when validating an <a class="reference internal" href="#valid-expr"><span class="std std-ref">expression</span></a> that is not a function body.
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
    <li>Let <span>\({\mathit{limits}}~{\mathit{elemtype}}\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-tabletype"><span class="std std-ref">table type</span></a> <span>\(C.{\mathsf{tables}}[0]\)</span>.</li>
    <li>The <a class="reference internal" href="../syntax/types.html#syntax-elemtype"><span class="std std-ref">element type</span></a> <span>\({\mathit{elemtype}}\)</span> must be <span>\({\mathsf{funcref}}\)</span>.</li>
    <li>The type <span>\(C.{\mathsf{types}}[x]\)</span> must be defined in the context.</li>
    <li>Let <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\(C.{\mathsf{types}}[x]\)</span>.</li>
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
    <li>The empty instruction sequence is valid with type <span>\([t^\ast] {\rightarrow} [t^\ast]\)</span>,
for any sequence of <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value types</span></a> <span>\(t^\ast\)</span>.</li>
</ul>
<div>\[\frac{
}{
  C {\vdash} \epsilon : [t^\ast] {\rightarrow} [t^\ast]
}\]</div>

<h3>空でない命令シーケンス: <span>\({\mathit{instr}}^\ast~{\mathit{instr}}_N\)</span></h3>

<ul>
    <li>The instruction sequence <span>\({\mathit{instr}}^\ast\)</span> must be valid with type <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>,
for some sequences of <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value types</span></a> <span>\(t_1^\ast\)</span> and <span>\(t_2^\ast\)</span>.</li>
    <li>The instruction <span>\({\mathit{instr}}_N\)</span> must be valid with type <span>\([t^\ast] {\rightarrow} [t_3^\ast]\)</span>,
for some sequences of <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value types</span></a> <span>\(t^\ast\)</span> and <span>\(t_3^\ast\)</span>.</li>
    <li>There must be a sequence of <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value types</span></a> <span>\(t_0^\ast\)</span>,
such that <span>\(t_2^\ast = t_0^\ast~t^\ast\)</span>.</li>
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
    <li>The instruction sequence <span>\({\mathit{instr}}^\ast\)</span> must be <a class="reference internal" href="#valid-instr-seq"><span class="std std-ref">valid</span></a> with type <span>\([] {\rightarrow} [t^\ast]\)</span>,
for some <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> <span>\([t^\ast]\)</span>.</li>
    <li>Then the expression is valid with <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> <span>\([t^\ast]\)</span>.</li>
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
    <li>or of the form <span>\({\mathsf{global.get}}~x\)</span>, in which case <span>\(C.{\mathsf{globals}}[x]\)</span> must be a <a class="reference internal" href="../syntax/types.html#syntax-globaltype"><span class="std std-ref">global type</span></a> of the form <span>\({\mathsf{const}}~t\)</span>.</li>
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
    <li>Let <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\(C.{\mathsf{types}}[x]\)</span>.</li>
    <li>Let <span>\(C'\)</span> be the same <a class="reference internal" href="conventions.html#context"><span class="std std-ref">context</span></a> as <span>\(C\)</span>,
but with:
<ul>
    <li><span>\({\mathsf{locals}}\)</span> set to the sequence of <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">value types</span></a> <span>\(t_1^\ast~t^\ast\)</span>, concatenating parameters and locals,</li>
    <li><span>\({\mathsf{labels}}\)</span> set to the singular sequence containing only <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">result type</span></a> <span>\([t_2^\ast]\)</span>.</li>
    <li><span>\({\mathsf{return}}\)</span> set to the <a class="reference internal" href="../syntax/types.html#syntax-valtype"><span class="std std-ref">result type</span></a> <span>\([t_2^\ast]\)</span>.</li>
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
    <li>The <a class="reference internal" href="../syntax/types.html#syntax-tabletype"><span class="std std-ref">table type</span></a> <span>\({\mathit{tabletype}}\)</span> must be <a class="reference internal" href="types.html#valid-tabletype"><span class="std std-ref">valid</span></a>.</li>
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
    <li>The <a class="reference internal" href="../syntax/types.html#syntax-memtype"><span class="std std-ref">memory type</span></a> <span>\({\mathit{memtype}}\)</span> must be <a class="reference internal" href="types.html#valid-memtype"><span class="std std-ref">valid</span></a>.</li>
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
    <li>The <a class="reference internal" href="../syntax/types.html#syntax-globaltype"><span class="std std-ref">global type</span></a> <span>\({\mathit{mut}}~t\)</span> must be <a class="reference internal" href="types.html#valid-globaltype"><span class="std std-ref">valid</span></a>.</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be <a class="reference internal" href="instructions.html#valid-expr"><span class="std std-ref">valid</span></a> with <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> <span>\([t]\)</span>.</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be <a class="reference internal" href="instructions.html#valid-constant"><span class="std std-ref">constant</span></a>.</li>
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
    <li>Let <span>\({\mathit{limits}}~{\mathit{elemtype}}\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-tabletype"><span class="std std-ref">table type</span></a> <span>\(C.{\mathsf{tables}}[x]\)</span>.</li>
    <li>The <a class="reference internal" href="../syntax/types.html#syntax-elemtype"><span class="std std-ref">element type</span></a> <span>\({\mathit{elemtype}}\)</span> must be <span>\({\mathsf{funcref}}\)</span>.</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be <a class="reference internal" href="instructions.html#valid-expr"><span class="std std-ref">valid</span></a> with <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> <span>\([{\mathsf{i32}}]\)</span>.</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be <a class="reference internal" href="instructions.html#valid-constant"><span class="std std-ref">constant</span></a>.</li>
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
    <li>The expression <span>\({\mathit{expr}}\)</span> must be <a class="reference internal" href="instructions.html#valid-expr"><span class="std std-ref">valid</span></a> with <a class="reference internal" href="../syntax/types.html#syntax-resulttype"><span class="std std-ref">result type</span></a> <span>\([{\mathsf{i32}}]\)</span>.</li>
    <li>The expression <span>\({\mathit{expr}}\)</span> must be <a class="reference internal" href="instructions.html#valid-constant"><span class="std std-ref">constant</span></a>.</li>
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
    <li>The export description <span>\({\mathit{exportdesc}}\)</span> must be valid with <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external type</span></a> <span>\({\mathit{externtype}}\)</span>.</li>
    <li>Then the export is valid with <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external type</span></a> <span>\({\mathit{externtype}}\)</span>.</li>
</ul>
<div>\[\frac{
  C {\vdash} {\mathit{exportdesc}} : {\mathit{externtype}}
}{
  C {\vdash} \{ {\mathsf{name}}~{\mathit{name}}, {\mathsf{desc}}~{\mathit{exportdesc}} \} : {\mathit{externtype}}
}\]</div>

<h3><span>\({\mathsf{func}}~x\)</span></h3>
<ul>
    <li>The function <span>\(C.{\mathsf{funcs}}[x]\)</span> must be defined in the context.</li>
    <li>Then the export description is valid with <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external type</span></a> <span>\({\mathsf{func}}~C.{\mathsf{funcs}}[x]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{funcs}}[x] = {\mathit{functype}}
}{
  C {\vdash} {\mathsf{func}}~x : {\mathsf{func}}~{\mathit{functype}}
}\]</div>

<h3><span>\({\mathsf{table}}~x\)</span></h3>
<ul>
    <li>The table <span>\(C.{\mathsf{tables}}[x]\)</span> must be defined in the context.</li>
    <li>Then the export description is valid with <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external type</span></a> <span>\({\mathsf{table}}~C.{\mathsf{tables}}[x]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{tables}}[x] = {\mathit{tabletype}}
}{
  C {\vdash} {\mathsf{table}}~x : {\mathsf{table}}~{\mathit{tabletype}}
}\]</div>

<h3><span>\({\mathsf{mem}}~x\)</span></h3>
<ul>
    <li>The memory <span>\(C.{\mathsf{mems}}[x]\)</span> must be defined in the context.</li>
    <li>Then the export description is valid with <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external type</span></a> <span>\({\mathsf{mem}}~C.{\mathsf{mems}}[x]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{mems}}[x] = {\mathit{memtype}}
}{
  C {\vdash} {\mathsf{mem}}~x : {\mathsf{mem}}~{\mathit{memtype}}
}\]</div>

<h3><span>\({\mathsf{global}}~x\)</span></h3>
<ul>
    <li>The global <span>\(C.{\mathsf{globals}}[x]\)</span> must be defined in the context.</li>
    <li>Then the export description is valid with <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external type</span></a> <span>\({\mathsf{global}}~C.{\mathsf{globals}}[x]\)</span>.</li>
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
    <li>Let <span>\([t_1^\ast] {\rightarrow} [t_2^\ast]\)</span> be the <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\(C.{\mathsf{types}}[x]\)</span>.</li>
    <li>Then the import description is valid with type <span>\({\mathsf{func}}~[t_1^\ast] {\rightarrow} [t_2^\ast]\)</span>.</li>
</ul>
<div>\[\frac{
  C.{\mathsf{types}}[x] = [t_1^\ast] {\rightarrow} [t_2^\ast]
}{
  C {\vdash} {\mathsf{func}}~x : {\mathsf{func}}~[t_1^\ast] {\rightarrow} [t_2^\ast]
}\]</div>

<h3><span>\({\mathsf{table}}~{\mathit{tabletype}}\)</span></h3>
<ul>
    <li>The table type <span>\({\mathit{tabletype}}\)</span> must be <a class="reference internal" href="types.html#valid-tabletype"><span class="std std-ref">valid</span></a>.</li>
    <li>Then the import description is valid with type <span>\({\mathsf{table}}~{\mathit{tabletype}}\)</span>.</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{tabletype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} {\mathsf{table}}~{\mathit{tabletype}} : {\mathsf{table}}~{\mathit{tabletype}}
}\]</div>

<h3><span>\({\mathsf{mem}}~{\mathit{memtype}}\)</span></h3>
<ul>
    <li>The memory type <span>\({\mathit{memtype}}\)</span> must be <a class="reference internal" href="types.html#valid-memtype"><span class="std std-ref">valid</span></a>.</li>
    <li>Then the import description is valid with type <span>\({\mathsf{mem}}~{\mathit{memtype}}\)</span>.</li>
</ul>
<div>\[\frac{
  {\vdash} {\mathit{memtype}} \mathrel{\mbox{ok}}
}{
  C {\vdash} {\mathsf{mem}}~{\mathit{memtype}} : {\mathsf{mem}}~{\mathit{memtype}}
}\]</div>

<h3><span>\({\mathsf{global}}~{\mathit{globaltype}}\)</span></h3>
<ul>
    <li>The global type <span>\({\mathit{globaltype}}\)</span> must be <a class="reference internal" href="types.html#valid-globaltype"><span class="std std-ref">valid</span></a>.</li>
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
    <li>Let <span>\(C\)</span> be a <a class="reference internal" href="conventions.html#context"><span class="std std-ref">context</span></a> where:
        <ul>
            <li><span>\(C.{\mathsf{types}}\)</span> is <span>\({\mathit{module}}.{\mathsf{types}}\)</span>,</li>
            <li><span>\(C.{\mathsf{funcs}}\)</span> is <span>\({\mathrm{funcs}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{ft}^\ast\)</span>,
        with the import’s <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external types</span></a> <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function types</span></a> <span>\(\mathit{ft}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{tables}}\)</span> is <span>\({\mathrm{tables}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{tt}^\ast\)</span>,
        with the import’s <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external types</span></a> <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-tabletype"><span class="std std-ref">table types</span></a> <span>\(\mathit{tt}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{mems}}\)</span> is <span>\({\mathrm{mems}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{mt}^\ast\)</span>,
        with the import’s <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external types</span></a> <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-memtype"><span class="std std-ref">memory types</span></a> <span>\(\mathit{mt}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{globals}}\)</span> is <span>\({\mathrm{globals}}(\mathit{it}^\ast)\)</span> concatenated with <span>\(\mathit{gt}^\ast\)</span>,
        with the import’s <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external types</span></a> <span>\(\mathit{it}^\ast\)</span> and the internal <a     class="reference internal" href="../syntax/types.html#syntax-globaltype"><span class="std std-ref">global types</span></a> <span>\(\mathit{gt}^\ast\)</span> as determined below,</li>
            <li><span>\(C.{\mathsf{locals}}\)</span> is empty,</li>
            <li><span>\(C.{\mathsf{labels}}\)</span> is empty,</li>
            <li><span>\(C.{\mathsf{return}}\)</span> is empty.</li>
        </ul>
    </li>
    <li>Let <span>\(C'\)</span> be the <a class="reference internal" href="conventions.html#context"><span class="std std-ref">context</span></a> where <span>\(C'.{\mathsf{globals}}\)</span> is the sequence <span>\({\mathrm{globals}}(\mathit{it}^\ast)\)</span> and all other fields are empty.</li>
    <li>Under the context <span>\(C\)</span>:
        <ul>
            <li>For each <span>\({\mathit{functype}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{types}}\)</span>,
        the <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\({\mathit{functype}}_i\)</span> must be <a class="reference internal" href="types.html#valid-functype"><span class="std std-ref">valid</span></a>.</li>
            <li>For each <span>\({\mathit{func}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{funcs}}\)</span>,
        the definition <span>\({\mathit{func}}_i\)</span> must be <a class="reference internal" href="#valid-func"><span class="std std-ref">valid</span></a> with a <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function type</span></a> <span>\(\mathit{ft}_i\)</span>.</li>
            <li>For each <span>\({\mathit{table}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{tables}}\)</span>,
        the definition <span>\({\mathit{table}}_i\)</span> must be <a class="reference internal" href="#valid-table"><span class="std std-ref">valid</span></a> with a <a class="reference internal" href="../syntax/types.html#syntax-tabletype"><span class="std std-ref">table type</span></a> <span>\(\mathit{tt}_i\)</span>.</li>
            <li>For each <span>\({\mathit{mem}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{mems}}\)</span>,
        the definition <span>\({\mathit{mem}}_i\)</span> must be <a class="reference internal" href="#valid-mem"><span class="std std-ref">valid</span></a> with a <a class="reference internal" href="../syntax/types.html#syntax-memtype"><span class="std std-ref">memory type</span></a> <span>\(\mathit{mt}_i\)</span>.</li>
            <li>For each <span>\({\mathit{global}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{globals}}\)</span>:
            <ul>
                <li>Under the context <span>\(C'\)</span>, the definition <span>\({\mathit{global}}_i\)</span> must be <a class="reference internal" href="#valid-global"><span class="std std-ref">valid</span></a> with a <a class="reference internal" href="../syntax/types.html#syntax-globaltype"><span class="std std-ref">global type</span></a> <span>\(\mathit{gt}_i\)</span>.</li>
            </ul>
            </li>
            <li>For each <span>\({\mathit{elem}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{elem}}\)</span>,
        the segment <span>\({\mathit{elem}}_i\)</span> must be <a class="reference internal" href="#valid-elem"><span class="std std-ref">valid</span></a>.</li>
            <li>For each <span>\({\mathit{data}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{data}}\)</span>, the segment <span>\({\mathit{data}}_i\)</span> must be <a class="reference internal" href="#valid-data"><span class="std std-ref">valid</span></a>.</li>
            <li>If <span>\({\mathit{module}}.{\mathsf{start}}\)</span> is non-empty, then <span>\({\mathit{module}}.{\mathsf{start}}\)</span> must be <a class="reference internal" href="#valid-start"><span class="std std-ref">valid</span></a>.</li>
            <li>For each <span>\({\mathit{import}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{imports}}\)</span>, the segment <span>\({\mathit{import}}_i\)</span> must be <a class="reference internal" href="#valid-import"><span class="std std-ref">valid</span></a> with an <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external type</span></a> <span>\(\mathit{it}_i\)</span>.</li>
            <li>For each <span>\({\mathit{export}}_i\)</span> in <span>\({\mathit{module}}.{\mathsf{exports}}\)</span>, the segment <span>\({\mathit{export}}_i\)</span> must be <a class="reference internal" href="#valid-export"><span class="std std-ref">valid</span></a> with <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external type</span></a> <span>\(\mathit{et}_i\)</span>.</li>
        </ul>
    </li>
    <li>The length of <span>\(C.{\mathsf{tables}}\)</span> must not be larger than <span>\(1\)</span>.</li>
    <li>The length of <span>\(C.{\mathsf{mems}}\)</span> must not be larger than <span>\(1\)</span>.</li>
    <li>All export names <span>\({\mathit{export}}_i.{\mathsf{name}}\)</span> must be different.</li>
    <li>Let <span>\(\mathit{ft}^\ast\)</span> be the concatenation of the internal <a class="reference internal" href="../syntax/types.html#syntax-functype"><span class="std std-ref">function types</span></a> <span>\(\mathit{ft}_i\)</span>, in index order.</li>
    <li>Let <span>\(\mathit{tt}^\ast\)</span> be the concatenation of the internal <a class="reference internal" href="../syntax/types.html#syntax-tabletype"><span class="std std-ref">table types</span></a> <span>\(\mathit{tt}_i\)</span>, in index order.</li>
    <li>Let <span>\(\mathit{mt}^\ast\)</span> be the concatenation of the internal <a class="reference internal" href="../syntax/types.html#syntax-memtype"><span class="std std-ref">memory types</span></a> <span>\(\mathit{mt}_i\)</span>, in index order.</li>
    <li>Let <span>\(\mathit{gt}^\ast\)</span> be the concatenation of the internal <a class="reference internal" href="../syntax/types.html#syntax-globaltype"><span class="std std-ref">global types</span></a> <span>\(\mathit{gt}_i\)</span>, in index order.</li>
    <li>Let <span>\(\mathit{it}^\ast\)</span> be the concatenation of <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external types</span></a> <span>\(\mathit{it}_i\)</span> of the imports, in index order.</li>
    <li>Let <span>\(\mathit{et}^\ast\)</span> be the concatenation of <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external types</span></a> <span>\(\mathit{et}_i\)</span> of the exports, in index order.</li>
    <li>Then the module is valid with <a class="reference internal" href="../syntax/types.html#syntax-externtype"><span class="std std-ref">external types</span></a> <span>\(\mathit{it}^\ast {\rightarrow} \mathit{et}^\ast\)</span>.</li>
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