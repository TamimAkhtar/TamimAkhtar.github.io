---
title: "Insertion Sort using Recursive Functional Programming"
description: "How beautiful recursion can be!"
date: 2025-06-23
categories: [notes]
tags: [Insertion Sort, CS61A]
---

This is an amazing algorithm, it really impresses me how powerful recursion is in general and doing this with functional programming makes this even more beautiful.

## The Code

```scheme
(define (sort sent)
    (if (empty? sent) '()
    (insert (first sent)(sort(bf sent)))))

(define (insert num sent)
    (cond ((empty? sent) (se num))
          ((< num (first sent))(se num sent))
          (else (se (first sent)(insert num (bf sent))))))
```

The basic idea is as follows:

I have a sentence of numbers, I start with comparing the second number with the one before it, if they are in ascending order, good, I move ahead to the third number.
Now if the third number is smaller than the second one, I compare it with the second number and also the first number and position it accordingly.

The logic is as follows:

- The sort function recursively breaks the sentence down to its last element.

- Then, each number is inserted into the correct position of the sentence that's already being built up in sorted order.

Take this sentence for example:

```scheme
'(11 10 8 5 15)
```

When passed through sort, the recursion transforms it into:

```scheme
(insert 11 (insert 10 (insert 8 (insert 5 (insert 15 '() )))))
```

So what happens is that each element waits until the recursion builds the sentence it's supposed to fit into. I get to the end of the sentence.

I want to compare every number. So I start with my sort method, it makes me go to the end of the sentence to the very last number while getting every element in the sentence ready for comparing.

Now, the insert method handles the actual comparison:

- If the sentence is empty, return a new one with just the number.
- If the number is smaller than the first element, it goes right there—at the front.
- Otherwise, we keep the current first element and try inserting the number into the rest of the list.

This guarantees that every number is inserted into an already sorted sentence.

My insert method is really simple, it takes a number and a sentence. It compares the number with the first element of sentence, if the number is smaller than the first element, that means that the sentence is sorted. Due to my sort method and how recursion works from inside to out, I will start comparing the last element with the elements before. 

So lets evaluate the part (insert 5 '15) > evaluates to '(5 15)

(insert 8 '(5 15)) > compare 8 with 5. As 5 is smaller, switch 5 with 8 and then also verify that the sent '(8 15) is sorted. If not, sort that as well so we end up with '(5 8 15).

Notice that senetence will always be sorted via this method before it is passed to insert.

```scheme
(insert 15 '()) → '(15)
(insert 5 '(15)) → '(5 15)
(insert 8 '(5 15)) → compare 8 with 5 → keep 5 → insert 8 into '(15)
                   → compare 8 with 15 → insert before 15 → result: '(5 8 15)
(insert 10 '(5 8 15)) → compare 10 with 5 → keep 5
                      → compare 10 with 8 → keep 8
                      → compare 10 with 15 → insert before 15 → result: '(5 8 10 15)
(insert 11 '(5 8 10 15)) → compare 11 with 5, 8, 10 → keep all
                         → compare 11 with 15 → insert before 15 → result: **'(5 8 10 11 15)**
```
