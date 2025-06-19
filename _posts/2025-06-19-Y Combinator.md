---
title: "Y Combinator"
description: "from CS61A"
date: 2025-06-19
categories: [notes]
tags: [Y Combinator, CS61A]
---

The Y Combinator is truly fascinating in terms of recursion and functional programming.
Suppose I want to define a simple function that computes the factorial of a number using recursion. 
In a typical programming language, I would define it like this:

```scheme
(define (factorial n)
  (if (= n 0) 1
      (* n (factorial (- n 1)))))
```
However, I can also define this procedure without having to name the function itself and without using the 'define' keyword.
I can do this by using the Y combinator as follows:

```scheme
(
(lambda (f)
  (lambda (n)(f f n)))
  (lambda (fun x)
    (if (= x 0) 1
        (* x (fun fun (- x 1))))))
```

So here, we have 3 lambda functions. Understand that the first lambda takes a function f as an argument which is the innermost
lambda function. lets call that function z. Note that the outer lambda itself returns a lambda function.

So I have basically (lambda (n)(z z n)).

lets call this with an argument 3

so I have (z z 3). 

This means that I am calling a function named 'a' with 2 arguments, a and 3. This is how I am able to call the function itself.

so when I call (a a 3), it will evaluate to (if (= 3 0) 1 (* 3 (a a (- 3 1))))

This will keep going until n is 0, at which point it will return 1.

The job of the Y combinator is to provide the function with itself as an argument.
