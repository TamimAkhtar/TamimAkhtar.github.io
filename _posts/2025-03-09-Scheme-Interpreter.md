---
title: "Writing a Scheme Interpreter in Scheme"
description: "Simple evaluator for Scheme using substitution model"
date: 2025-03-09
categories: [CS61A]
tags: [Scheme Interpreter/Evaluator]
---

<p>Today, we’ll discuss a Scheme Interpreter written in the Scheme language. This version is based on the <strong>UC Berkeley CS61A (2011)</strong> course by Brian Harvey. The interpreter runs in a <strong>REPL environment</strong>, executing code piece by piece.</p>

<p>To keep things simple, we’ll define some rules:</p>

<ol>
  <li>We'll call our evaluator <strong>Scheme-1</strong>, and all procedure names will end in <code>-1</code> to avoid confusion with STK (a real Scheme interpreter).</li>
  <li>Scheme-1 uses the <strong>substitution model</strong> and does not include <code>define</code>. Procedures can only be defined using <code>lambda</code>.</li>
</ol>

<h2>Expression Types in Scheme-1</h2>

<p>There are four basic expression types in Scheme, which we’ll handle as follows in Scheme-1:</p>

<h3>1. Self-Evaluating Expressions</h3>

<p>These include numbers, <code>#t</code>, <code>#f</code>, and other constant values.</p>

<ul>
  <li>STK’s primitive procedures (<code>+</code>, <code>-</code>, <code>/</code>, <code>*</code>, <code>cons</code>, etc.) are treated as self-evaluating constants.</li>
  <li>Since Scheme-1 lacks an environment system with global bindings for functions like STK, we use STK’s evaluator to retrieve values directly instead of treating these as functions.</li>
</ul>

<h3>2. Symbols (Variables)</h3>

<ul>
  <li>In the substitution model, local variables are not evaluated directly because we will substitute the actual argument values in the procedure body before evaluating the procedure itself. So by the time we evaluate the procedure itself, all variables hsould have been replaced by argument values. </li>
  <li>Therefore, the only variable names left are primitive Scheme functions, which we evaluate using STK’s built-in <code>eval</code>.</li>
  <li>This allows us to use global primitive functions without maintaining a symbol table or a full environment system.</li>
</ul>

<h3>3. Special Forms</h3>

<p>Special forms follow special evaluation rules (not normal applicative-order evaluation). Scheme-1 only supports:</p>

<ul>
  <li><code>if</code></li>
  <li><code>quote</code></li>
  <li><code>lambda</code></li>
</ul>

<h3>4. Procedure Calls</h3>

<p>To evaluate a procedure call:</p>

<ol>
  <li>Recursively evaluate all subexpressions using <code>Eval-1</code>.</li>
  <li>Call <code>Apply-1</code> to handle the procedure invocation.</li>
</ol>

<h2>Some Trivial Helper Procedures</h2>

<p>These procedures help us identify the types of expressions we defined earlier.</p>

```scheme
(define (constant? exp)
  (or (number? exp) (boolean? exp) (string? exp) (procedure? exp)))

(define (exp-checker type)
  (lambda (exp) (and (pair? exp) (eq? (car exp) type))))

(define quote-exp? (exp-checker 'quote))
(define if-exp? (exp-checker 'if))
(define lambda-exp? (exp-checker 'lambda))
(define define-exp? (exp-checker 'define))
```



