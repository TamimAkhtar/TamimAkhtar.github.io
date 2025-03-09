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

<h2>The “read-eval-print loop” (REPL):</h2>

```scheme
(define (scheme-1)
  (display "Scheme-1: ")
  (flush)
  (print (eval-1 (read)))
  (scheme-1))
```

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

<h2>Evaluating an Expression:</h2>

<p><li>EVAL-1</li> takes an expression and returns its value. Scheme expressions have sub-expressions which can have sub-expressions, so it is not surprising to see tree recursion involved in <i>EVAL-1</i> to deal with hierarchies, while recursively evaluating all the subexpressions as well. </p>

<ul>
  <li>The read procedure turns the expression “foo to (quote foo). The value of (quote foo) is foo – the second element of the expression.</li>
  <li>To evaluate the expression (IF A B C), we first evaluate A; then if A is true, we evaluate B; if A is false, we evaluate C.</li>
  <li>The value of the lambda expression is the expression itself and there is no work to do until we call the actual procedure. </li>
  <li>To evaluate a procedure call, we recursively evaluate all the subexpressions. </li>
</ul>

```scheme
(define (eval-1 exp)
  (cond ((constant? exp) exp) 
	((symbol? exp) (eval exp))	; use STK's EVAL
	((quote-exp? exp) (cadr exp))
	((if-exp? exp)
	 (if (eval-1 (cadr exp))
	     (eval-1 (caddr exp))
	     (eval-1 (cadddr exp))))
	((lambda-exp? exp) exp)
	((define-exp? exp)
	 (eval (list 'define (cadr exp) (maybe-quote (eval-1 (caddr exp))))))
	((pair? exp) (apply-1 (eval-1 (car exp))      ; procedure call
			         (map eval-1 (cdr exp))))         ; turning actual argument expressions into argument values by tree recursion
	(else (error "bad expr: " exp))))
```

