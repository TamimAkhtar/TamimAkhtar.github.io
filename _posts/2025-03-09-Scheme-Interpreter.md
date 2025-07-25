---
title: "Writing a Scheme Interpreter in Scheme"
description: "Simple evaluator for Scheme using substitution model"
date: 2025-03-09
categories: [Blog]
tags: [Scheme Interpreter/Evaluator]
---

<p>Today, we’ll discuss a Scheme Interpreter written in the Scheme language. This version is based on the <strong>UC Berkeley CS61A (2011)</strong> course by Brian Harvey. The interpreter runs in a <strong>REPL environment</strong>, executing code line by line.</p>

<p>To keep things simple, we’ll define some rules:</p>

<ol>
  <li>We'll call our evaluator <strong>Scheme-1</strong>, and all procedure names will end in <code>-1</code> to avoid confusion with STK (a real Scheme interpreter).</li>
  <li>Scheme-1 uses the <strong>substitution model</strong> and does not include <code>define</code>. Therefore, procedures can only be defined using <code>lambda</code>.</li>
</ol>

<h2>Expression Types in Scheme-1</h2>

<p>There are four basic expression types in Scheme, which we’ll handle as follows in Scheme-1:</p>

<h3>1. Self-Evaluating Expressions</h3>

<p>These include numbers, <code>#t</code>, <code>#f</code>, and other constant values.</p>

<ul>
  <li>STK’s primitive procedures (<code>+</code>  <code>-</code>  <code>/</code>  <code>*</code>  <code>cons</code>  etc.) are treated as self-evaluating constants.</li>
  <li>Since Scheme-1 lacks an environment system with global bindings for functions like STK, we use STK’s evaluator to retrieve values directly instead of treating these as functions.</li>
</ul>

<h3>2. Symbols (Variables)</h3>

<ul>
  <li>In the substitution model, local variables are not evaluated directly because we will substitute the actual argument values in the procedure body before evaluating the procedure itself. So by the time we evaluate the procedure itself, all variables should have been replaced by argument values. </li>
  <li>Therefore, the only variable names left are primitive Scheme functions, which we evaluate using STK’s built-in <code>eval</code>.</li>
  <li>This allows us to use global primitive functions without maintaining a symbol table or a full environment system.</li>
</ul>

<h3>3. Special Forms</h3>

<p>Special forms follow special evaluation rules (not normal applicative-order evaluation). Scheme-1 only supports:</p>

<ul>
  <li><code>if</code> , <code>quote</code> , <code>lambda</code> , <code>and</code> , <code>let</code>. </li>
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
(define and-exp? (exp-checker 'and))
(define let-exp? (exp-checker 'let))

(define (let-names exp)
  (map car (cadr exp)))

(define (let-values exp)
  (map cadr (cadr exp)))

(define let-body caddr)
```

<h2>Evaluating an Expression:</h2>

<p><i>eval-1</i> takes an expression and returns its value. Scheme expressions have sub-expressions which can have sub-expressions, it is not surprising to see tree recursion involved in <i>eval-1</i> to deal with hierarchies (subexpressions). </p>

<ul>
  <li>The read procedure turns the expression “foo to <code>(quote foo)</code>. The value of <code>(quote foo)</code> is <code>foo</code> – the second element of the expression.</li>
  <li>The special forms are checked before the <code>pair?</code> test because special forms are also pairs and must be caught before we interpret them as ordinary pair calls </li>
  <li>The value of the lambda expression is the expression itself and there is no work to do until we call the actual procedure. </li>
  <li>To evaluate a procedure call, we recursively evaluate all the subexpressions. </li>
  <li>LET is really a lambda combined with a procedure call, and one way we can handle LET expressions is just to rearrange the text to get <code> ((lambda (name1 name2 ...) body) value1 value2 ...) </code>. </li>
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
  ((and-exp? exp) (eval-and-1 (cdr exp)))
  ((let-exp? exp) (eval-let exp))
	((pair? exp) (apply-1 (eval-1 (car exp))      ; procedure call
			         (map eval-1 (cdr exp))))         ; turning actual argument expressions into argument values by tree recursion
	(else (error "bad expr: " exp))))

(define (eval-and-1 subexps)
  (if (null? subexps) #t
      (let ((result (eval-1(car subexps))))
        (cond ((null? (cdr subexps)) result)
              ((equal? result #f) #f)
              (else (eval-and-1 (cdr subexps)))))))

(define (eval-let exp)
  (apply-1 (list 'lambda (let-names exp) (let-body exp))
	   (map eval-1 (let-values exp))))
```

<h2>Procedure invocation:</h2>

<p><i>apply-1</i> takes an expression which should only have values (substituted by <i>eval-1</i> procedure) and no notation. It then calls the relevant procedure on those values. </p>

<ul>
  <li><i>apply-1</i> takes 2 arguments: a procedure and a list of argument values.</li>
  <li>There are 2 kinds of procedures: primitive and LAMBDA-created. Primitive procedures are called using STK’s eval procedure.</li>
  <li>In Scheme-1, the value of a LAMBDA expression is the expression itself. We substitute the actual arguments(<i>args</i>) for the formal parameters (<i>cadr proc</i>) in the body (<i>caddr proc</i>). The result of this substitution is an expression which we then evaluate with <i>eval-1.</i></li>
</ul>

```scheme
(define (apply-1 proc args)
  (cond ((procedure? proc)	; use STK's APPLY
	 (apply proc args))
	((lambda-exp? proc)
	 (eval-1 (substitute (caddr proc)   ; the body
			     (cadr proc)    ; the formal parameters
			     args           ; the actual arguments
			     '())))	    ; bound-vars, see below
	(else (error "bad proc: " proc))))
```

<h2>Substituting argument values:</h2>

<p>The <i>substitute</i> procedure takes an expression (body of the procedure), formal parameters, argument values and bound variables as input and returns the procedure body with all argument variables substituted in place of the formal parameters. </p>

<p><i>Substitute</i> replaces actual arguments for free references to the corresponding formal parameters. For example, given the expression:</p>

```scheme
((lambda (x y)
   ((lambda (x) (+ x y))
    (* x y)))
 5 8)
```

<p>The body of the procedure we're calling is <code>((lambda (x) (+ x y))(* x y))</code> and we want to substitute 5 for X and 8 for Y, but the result should be <code>((lambda (x) (+ x 8))(* 5 8))</code> and not  <code>((lambda (5) (+ 5 8))(* 5 8))</code>.</p>

<p>To make this work, in its recursive calls, <i>substitute</i> keeps a list of bound variables in the current subexpression -- ones that shouldn't be substituted for in its argument <i>bound</i>.  This argument is the empty list in the top-level call to <i>substitute</i> from <i>apply-1</i>. So we call <i>substitute</i> on each element of the procedure body and if we encounter a variable that isn’t a bound variable, we replace the argument with that variable.</p>

<p>To deal with nested lambda expressions, the procedure returns the inner lambda expression with the unbound variables substituted. The inner lambda expression is returned with its argument and needs to go through eval-1 procedure which will again recursively call the substitute procedure on the lambda expression.</p>

<p><i>Maybe-quote</i> ensures that non-self-evaluating values stay quoted. For example, consider the expression <code>((lambda (x) (first x)) 'foo)</code>. The actual argument value is <code>foo</code>, but we want the result of the substitution to be <code>(first 'foo)</code> and not <code>(first foo)</code>.</p>

```scheme
(define (substitute exp params args bound)
  (cond ((constant? exp) exp)
        ((symbol? exp) ;check for variables
         (if (memq exp bound)
             exp
             (lookup exp params args)))
        ((quote-exp? exp) exp)
        ((lambda-exp? exp) ;checks for nested lambda
         (list 'lambda
               (cadr exp)
               (substitute (caddr exp) params args (append bound (cadr exp)))))
        (else (map (lambda (subexp) (substitute subexp params args bound))
                   exp))))

(define (lookup name params args)
  (cond ((null? params) name)
	((eq? name (car params)) (maybe-quote (car args)))
	(else (lookup name (cdr params) (cdr args)))))

(define (maybe-quote value)
  (cond ((lambda-exp? value) value)
	((constant? value) value)
	((procedure? value) value)	; Stk procedure
	(else (list 'quote value))))
```

<details>
  <summary>map-1 procedure</summary>

```scheme
(define (map-1 fn seq) ;STK's primitive procedure 'map' applied as map-1
  (if (empty? seq) '()
        (cons (apply-1 fn (list(eval-1(car seq))))(map-1 fn (cdr seq)))))
```

</details>