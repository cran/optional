[![Build Status](https://travis-ci.org/antoinechampion/optional.svg?branch=master)](https://travis-ci.org/antoinechampion/optional) [![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/optional)](https://cran.r-project.org/package=optional) ![CRAN\ Downloads](https://cranlogs.r-pkg.org/badges/optional)

<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">

<head>

<meta charset="utf-8">
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<meta name="generator" content="pandoc" />

<meta name="viewport" content="width=device-width, initial-scale=1">

<meta name="author" content="Antoine Champion" />

<meta name="date" content="2017-07-27" />

<div id="installtion" class="section level2">
<h2>Installation</h2>
<p>You can install this package from this repository by running <code>devtools::install_github("antoinechampion/optional")</code>, or from the CRAN by running <code>install.packages("optional")</code>.
</div>

<div id="introduction" class="section level2">
<h2>Introduction</h2>
<p>This package adds an <code>optional</code> type, similar to <code>Option</code> in F#, OCaml and Scala, to <code>Maybe</code> in Haskell, and to nullable types in <code>C#</code>.</p>
<p>It should be used instead of <code>NULL</code> for values that might be missing or otherwise invalid.</p>
<p>This package also introduces pattern matching.</p>
</div>
<div id="using-the-optional-type" class="section level2">
<h2>Using the optional type</h2>
<p><code>optional</code> is an object wrapper which indicates whether the object is valid or not.</p>
<div id="declaring-an-optional-object" class="section level3">
<h3>Declaring an optional object</h3>
<p>An optional variable can be set to <code>some(object)</code> or to <code>none</code>.</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r">a &lt;-<span class="st"> </span><span class="kw">some</span>(<span class="dv">5</span>)
<span class="kw">class</span>(a)</code></pre></div>
<pre><code>## [1] &quot;optional&quot;</code></pre>
<p>Operators and print will have the same behavior with an <code>optional</code> than with its base type.</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r">a <span class="op">==</span><span class="st"> </span><span class="dv">5</span></code></pre></div>
<pre><code>## [1] TRUE</code></pre>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r">a</code></pre></div>
<pre><code>## [1] 5</code></pre>
<p>Note that <code>some(some(obj))</code> equals <code>some(obj)</code> and that <code>some(none)</code> equals <code>FALSE</code>.</p>
</div>
<div id="optionals-on-functions" class="section level3">
<h3>Optionals on functions</h3>
<p>Given a function <code>f()</code>, to handle properly <code>optional</code> arguments and wraps its return type into an <code>optional</code>, one should use <code>make_opt()</code> the following way:</p>
<pre><code>f_opt &lt;- make_opt(f)</code></pre>
<ol style="list-style-type: decimal">
<li>Every <code>optional</code> argument passed to <code>f_opt()</code> will be converted to its original type before being sent to <code>f()</code>. If one or more of them is <code>none</code>, several behaviors are available (see <code>?make_opt</code>).</li>
<li>If <code>f()</code> returns null, or if an error is thrown during its execution, then <code>f_opt()</code> returns <code>none</code>. Else it will return <code>some(f(...))</code>.</li>
</ol>
<p>For instance:</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r">c_opt &lt;-<span class="st"> </span><span class="kw">make_opt</span>(c)
<span class="kw">c_opt</span>(<span class="kw">some</span>(<span class="dv">2</span>), none, <span class="kw">some</span>(<span class="dv">5</span>))</code></pre></div>
<pre><code>## [1] 2 5</code></pre>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class="kw">c_opt</span>()</code></pre></div>
<pre><code>## [1] &quot;None&quot;</code></pre>
</div>
</div>
<div id="pattern-matching" class="section level2">
<h2>Pattern matching</h2>
<p>Patterns are used in many functional languages in order to process variables in an exhaustive way.</p>
<p>The syntax is the following:</p>
<pre><code>match_with( variable,
pattern , result-function,
...</code></pre>
<p>If <code>variable</code> matches a <code>pattern</code>, <code>result-function</code> is called. For comparing optional types, it is a better habit to use <code>match_with()</code> rather than a conditional statement.</p>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class="kw">library</span>(magrittr)

a &lt;-<span class="st"> </span><span class="dv">5</span>
<span class="kw">match_with</span>(a,
  . <span class="op">%&gt;%</span><span class="st"> </span><span class="kw">some</span>(.),          paste,
  none,                   <span class="cf">function</span>() <span class="st">&quot;Error!&quot;</span>
)</code></pre></div>
<pre><code>## [1] &quot;5&quot;</code></pre>
<ol style="list-style-type: decimal">
<li>Each <code>pattern</code> can be either:
<ul>
<li>an object or a primitive type (direct comparison with <code>variable</code>),</li>
<li>a list (match if <code>variable</code> is in the list),</li>
<li>a <code>magrittr</code> functional sequence that matches if it returns <code>variable</code>. The dot <code>.</code> denotes the variable to be matched.</li>
</ul></li>
<li>If <code>result-function</code> takes no arguments, it will be called as is. Else, the only argument that will be sent is <code>variable</code>. You can also use the fallthrough function <code>fallthrough()</code> to permit the matching to continue even if the current pattern is matched.</li>
</ol>
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r">a &lt;-<span class="st"> </span><span class="dv">4</span>
<span class="kw">match_with</span>(a,
  <span class="dv">1</span>,                 <span class="cf">function</span>() <span class="st">&quot;Matched exact value&quot;</span>,
  <span class="kw">list</span>(<span class="dv">2</span>, <span class="dv">3</span>, <span class="dv">4</span>),     <span class="kw">fallthrough</span>(<span class="cf">function</span>() <span class="st">&quot;Matched in list&quot;</span>),
  . <span class="op">%&gt;%</span><span class="st"> </span><span class="cf">if</span> (. <span class="op">&gt;</span><span class="st"> </span><span class="dv">3</span>)., <span class="cf">function</span>(x) <span class="kw">paste0</span>(<span class="st">&quot;Matched in condition: &quot;</span>,x,<span class="st">&quot;&gt;3&quot;</span>)
)</code></pre></div>
<pre><code>## [1] &quot;Matched in list&quot;           &quot;Matched in condition: 4&gt;3&quot;</code></pre>
</div>

</body>
</html>
