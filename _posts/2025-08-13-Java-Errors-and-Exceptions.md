---
title: "Handling Java Errors and Exceptions"
description: "How to deal with Java Exception handling"
date: 2025-08-13
categories: [notes]
tags: [Exceptions, Errors, Java]
---

# ⚖️ Java Exceptions and Errors

As Java is an **object-oriented language**, all exceptions are considered objects of special classes. The class hierarchy of Java Exceptions and Errors can be seen above, where the base class is `java.lang.Throwable`, which has two direct subclasses: **Error** and **Exception**.

---

## 💥 Error Class
Represents low-level exceptions in the JVM, such as:
- `OutOfMemoryError`
- `StackOverflowError`

---

## ⚠️ Exception Class

### 🚀 Runtime / Unchecked Exceptions
- Checked **during program execution**.
- Do not prevent code from compiling.

### 📋 Checked Exceptions
- Checked **by the compiler** at compile time.
- The compiler checks whether the programmer expects such exceptions in a program.  
- If a programmer expects such exceptions, they must indicate it; otherwise, the compiler cannot compile the program.

**Example:**
```java
public static String readLineFromFile() throws FileNotFoundException {
    Scanner scanner = new Scanner(new File("file.txt"));
}
```
Here, the programmer indicates that the `Scanner` constructor might throw a `FileNotFoundException`.  
As a result, the **method caller** will need to either:
- Handle the exception internally, or  
- Throw it further to its caller method.

---

## 🛠 General Exception Handling
An exception **interrupts** the normal execution of a program. After a line of code throws an exception, the Java runtime system attempts to find a **suitable handler**.

Such a handler can be:
- Located **in the same method** where the exception occurred, or  
- In the **calling method**.

Once a handler is found and executed, the exception is **handled**, and the program continues normally.

---

### ✅ Suitable Handler 1: `try` - `catch` Block
- If an exception occurs in the `try` block, execute the code in the `catch` block.
- Supports **several catch blocks** to handle multiple exceptions.

### 🔄 Suitable Handler 2: `try` - `catch` - `finally` Block
- The statements in the `finally` block always execute, regardless of whether an exception occurs in the `try` block.

---

### ⚠️ Warning Handler: Declaring Exceptions in Method Signatures
Declaring that a method **may throw** an exception:

```java
public static String readLineFromFile() throws FileNotFoundException {
    Scanner scanner = new Scanner(new File("file.txt"));
}
```

---

## 📦 Array Exceptions

### ❌ NullPointerException
Since arrays are reference types, their variable can be `null`.  
Example:
```java
int[] numbers = null;
int size = numbers.length; // Throws NPE
```
**Fix:**
```java
int size = numbers == null ? 0 : numbers.length;
```

### 🚫 ArrayIndexOutOfBoundsException
Occurs when trying to access a **non-existent element** of the array.  
**Check before accessing:**
```java
if (index < 0 || index > hardCodedArray.length - 1) {
    System.out.println("The index is out of bounds.");
} else {
    System.out.println(hardCodedArray[index]);
}
```

---

## 🎯 Throwing Exceptions Yourself
Any object of the `Throwable` class or its subclasses can be thrown using the `throw` statement.

**Best practices:**
- If a client can reasonably recover → **make it a checked exception**.
- If a client cannot recover → **make it an unchecked exception**.

**Examples:**

☑ Checked Exception:
```java
public static String readTextFromFile(String path) throws IOException {
    if (!found) {
        throw new IOException("The file " + path + " is not found");
    }
}
```

🚫 Unchecked Exception:
```java
public void deposit(long amount) {
    if (amount <= 0) {
        throw new IllegalArgumentException("Incorrect sum " + amount);
    }
    if (amount >= 100_000_000L) {
        throw new IllegalArgumentException("Too large amount");
    }
    balance += amount;
}
```

---

## 📜 Handling Input Stream Exceptions
Java Input Streams should be **closed after use** to release system resources.

**Naive example:**
```java
Reader reader = null;

try {
    reader = new FileReader("file.txt");
    // code which may throw an exception
} finally {
    reader.close();
}
```
The `close()` method is in `finally` so exceptions don’t prevent it from running.  
**But**: If `close()` itself throws an exception, it will hide the original exception from the `try` block.

---

### 🛡 Improved Handling
```java
void readFile() throws IOException {
    Reader reader = null;
    try {
        reader = new FileReader("file.txt");
        throw new RuntimeException("Exception1");
    } finally {
        try {
            reader.close(); // throws Exception2
        } catch (Exception e) {
            // handle Exception2
        }
    }
}
```

---

## ♻️ try-with-resources
Automatically closes resources that implement `AutoCloseable`:

```java
try (Reader reader1 = new FileReader("file1.txt");
     Reader reader2 = new FileReader("file2.txt")) {
    // some code
}
```

If both the `try` block and the `close()` method throw exceptions:
```java
void readFile() throws IOException {
    try (Reader reader = new FileReader("file.txt")) {
        throw new RuntimeException("Exception1");
    }
}
```
Output:
```
Exception in thread "main" java.lang.RuntimeException: Exception1
    at ...
    Suppressed: java.lang.RuntimeException: Exception2
        at ...
```

---

## 🐾 Assertions
```java
public Cat(String name, int age) {
    assert (age >= 0) : "Invalid age";
    this.name = name;
    this.age = age;
}
```
Run with:
```bash
java -ea BrokenInvariants
```

**Behavior:**
- ✅ Condition true → nothing happens.
- ❌ Condition false & assertions enabled → throws `AssertionError`.
- 🚫 Assertions disabled (default) → ignored entirely.

---