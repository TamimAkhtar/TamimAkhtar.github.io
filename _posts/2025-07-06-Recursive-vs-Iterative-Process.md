---
title: "Recursive vs Iterative Process"
description: "Comparing 2 imp programming techniques"
date: 2025-07-06
categories: [notes]
tags: [Recursion, Iteration, CS61A]
---

The immediate issue for today is the difference between a linear recursive process and an iterative process.

## 🔁 Recursive Process

```scheme
(define (count sent)
  (if (empty? sent)
      0
      (+ 1 (count (bf sent)))))
```

This function counts the number of words in a sentence. It takes Θ(N) time. It also requires Θ(N) space, not counting the space for the sentence itself, because Scheme has to keep track of N pending computations during the processing:

(count '(she loves you))

= (+ 1 (count '(loves you)))

= (+ 1 (+ 1 (count '(you))))

= (+ 1 (+ 1 (+ 1 (count '()))))

= (+ 1 (+ 1 (+ 1 0)))

= (+ 1 (+ 1 1))

= (+ 1 2)

= 3

So in recursion, we have to wait for the next one in line to return a value and all that information takes up space in computer memory and is stored waiting for the next in line to return something for it to return a value itself.

When we get halfway through this chart and compute (count ’()), we aren’t finished with the entire problem. We have to remember to add 1 to the result three times. Each of those remembered tasks requires some space in memory until it’s finished.

## 🔁 Iterative Process

```scheme
(define (count sent)
  (define (iter wds result)
    (if (empty? wds)
        result
        (iter (bf wds) (+ result 1))))
  (iter sent 0))
```

This time, we don’t have to remember uncompleted tasks; when we reach the base case of the recursion, we have the answer to the entire problem:

(iter '(she loves you) 0)

→ (iter '(loves you) 1)

→ (iter '(you) 2)

→ (iter '() 3)

→ 3

When a process has this structure, Scheme does not need extra memory to remember all the unfinished tasks during the computation.

We are doing the work on the way in rather than doing work on the way out.
All the work is done prior to the recursive call and there's no need to wait for intermediate answers. So we don't need to use more and more memory as the size of the problem increases.

## 🧠 Some Takeaways

Recursive: We have a chain of deferred operations, interpreter should keep track of the operations it has to perform later on.

Iterative: We only need to keep track of current variables at any given time.

Note that both these procedures are recursive, as in the syntax used for writing both of them is recursive. The process of how they evolve is different.

