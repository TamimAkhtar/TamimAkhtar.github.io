---
title: "My first note"
description: "Normal vs Applicative order of evaluation"
date: 2025-06-13
categories: [notes]
tags: [Normal order , applicative order, evaluation]
---

# 🚀 Normal Order vs Applicative Order in Java

When a function is applied to arguments, two main evaluation strategies can be used:

- **Applicative Order**: Evaluate all arguments first, then apply the function.
- **Normal Order**: Substitute arguments directly into the function and evaluate only when needed.

Let’s understand this using simple examples in Java.

---

## 🔁 Substitution Model (Applicative Order)

> <strong>“Evaluate the arguments and then apply the function.”</strong>

In this model, each argument is evaluated before the function body runs.
