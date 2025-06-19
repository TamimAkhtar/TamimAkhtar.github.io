---
title: "Normal vs Applicative order of evaluation"
description: "from CS61A"
date: 2025-06-13
categories: [notes]
tags: [Normal order , applicative order, evaluation]
---

When a function is applied to arguments, two evaluation strategies can be used:

- **Applicative Order**: Evaluate all arguments first, then apply the function.
- **Normal Order**: Substitute arguments directly into the function and evaluate only when needed.

Let’s understand this using simple examples in Java.

---

## 🔁 Substitution Model (Applicative Order)

> <strong>“Evaluate the arguments and then apply the function.”</strong>

In this model, each argument is evaluated before the function body runs.

```java
public class Main {
    public static void main(String[] args) {
        int result = sumOfSquares(6 - 1, 7 * 2);
        System.out.println(result);
    }

    public static int sumOfSquares(int x, int y) {
        return square(x) + square(y);
    }

    public static int square(int n) {
        return n * n;
    }
}
```
### 🧮 Evaluation Steps

1. Evaluate arguments:
   - `6 - 1 → 5`
   - `7 * 2 → 14`

2. Function becomes:
   ```java
   square(5) + square(14)
   → (5 * 5) + (14 * 14)
   → 25 + 196
   → 221
   ```

   ## 🔁 Normal Order

> <strong>“Fully expand, then reduce.”</strong>

In this method, arguments are not evaluated upfront until their values are needed. Instead, expressions are substituted for formal parameters and until an expression involving only primitive operators is remaining.

### Same function call:
```java
sumOfSquares(6 - 1, 7 * 2)
```

Substitution would look like:
```java
square(6 - 1) + square(7 * 2)
→ (6 - 1) * (6 - 1) + (7 * 2) * (7 * 2)
→ 5 * 5 + 14 * 14
→ 25 + 196
→ 221
```

Even though we get the **same result**, the process is different — expressions like `6 - 1` and `7 * 2` may be evaluated multiple times.

---
## 🖼️ Visual Comparison

<table>
  <thead>
    <tr>
      <th>Applicative Order</th>
      <th>Normal Order</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Evaluate arguments first:<br><code>square(5) + square(14)</code></td>
      <td>Substitute expressions:<br><code>square(6 - 1) + square(7 * 2)</code></td>
    </tr>
    <tr>
      <td><code>(5 * 5) + (14 * 14)</code></td>
      <td><code>(6 - 1) * (6 - 1) + (7 * 2) * (7 * 2)</code></td>
    </tr>
    <tr>
      <td>→ 25 + 196 → 221</td>
      <td>→ 25 + 196 → 221</td>
    </tr>
  </tbody>
</table>

---

## ⚠️ Non-Termination in Applicative Order

Applicative order can cause problems if an argument expression doesn't terminate (e.g., infinite recursion or error).

### 🧨 Example

```java
public class Main {
    public static void main(String[] args) {
        int max = (3 < 4) ? (5 + 5) : (1 / 0); // Divide-by-zero is never needed
        System.out.println(max);
    }
}
```

### In Applicative Order

Even though the condition is `true`, **both** `(5 + 5)` and `(1 / 0)` are evaluated first:

```
→ (3 < 4) → true
→ Evaluate both sides:
    (5 + 5) → 10
    (1 / 0) → ❌ Error: Division by zero
```

### In Normal Order

Only the branch needed is evaluated:

```
→ (3 < 4) → true
→ Use only (5 + 5)
→ Result: 10 ✅
```
## 👇 Some Takeaways
<ul>

<li>The purpose of these evaluation methods is to help us think about procedure application, not to provide a description of how the interpreter really works.</li>
<li>If we are doing purely functional programming, using either evaluation method returs the same result. Especially, Applicative order only works for non-mutable data when states remain unchanged.</li>
<li>Most functional programming languages use lazy evaluation in which we delay function evaluation such that it avoids multiple evaluation of the same function; This is a combination of both Applicative and Normal order evaluation.</li>
</ul>

---

References
1. https://sookocheff.com/post/fp/evaluating-lambda-expressions/
2. Structure and Interpretations of Computer Programs - The Wizard book

