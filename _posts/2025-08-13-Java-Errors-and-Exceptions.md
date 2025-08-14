---
title: "Handling Java Errors and Exceptions"
description: "How to deal with Java Exception handling"
date: 2025-08-13
categories: [notes]
tags: [Exceptions, Errors, Java]
---

# ⚖️ Java Exceptions and Errors

![Java Exceptions Hierarchy](assets/lib/java_exceptions_hierarchy.png)

As Java is an object-oriented language, all exceptions are considered objects of special classes. The class hierarchy of Java Exceptions and Errors can be seen above, where the base class is `java.lang.Throwable`, which has two direct subclasses: **Error** and **Exception**.

---

## 💥 Error Class
Represents low-level exceptions in the JVM, such as:
- `OutOfMemoryError`
- `StackOverflowError`

---

## ⚠️ Exception Class

### 🚀 Runtime / Unchecked Exceptions
- Checked during program execution.
- Does not prevent code from compiling.

### 📋 Checked Exceptions
- Checked by the compiler at compile time.
- The compiler checks if the programmer expects the occurance of such exceptions in a program.  
- If a programmer expects such exceptions, they must indicate it; otherwise, the compiler cannot compile the program.

Here, the programmer indicates that the `Scanner` constructor might throw a `FileNotFoundException` in method declaration.
As a result, the method caller will need to either:
- Handle the exception internally, or  
- Throw it further to its caller method.

**Example:**
```java
public static String readLineFromFile() throws FileNotFoundException {
    Scanner scanner = new Scanner(new File("file.txt"));
}
```

---

## 🛠 General Exception Handling
An exception interrupts the normal execution of a program. After a line of code throws an exception, the Java runtime system attempts to find a suitable handler for it.

Such a handler can be located in the same method where the exception occurred, or in the calling method. As soon as a handler is found and executed, the exception is handled, and the program continues normally.

---

### ✅ Suitable Handler 1: `try` - `catch` Block
- If an exception occurs in the `try` block, execute the code in the `catch` block.
- Supports several catch blocks to handle multiple exceptions.

### 🔄 Suitable Handler 2: `try` - `catch` - `finally` Block
- The statements in the `finally` block always execute, regardless of whether an exception occurs in the `try` block.

---

### ⚠️ Warning Handler: Declaring Exceptions in Method Signatures
Declaring that a method **may throw** an exception; required for Checked Exceptions.

```java
public static String readLineFromFile() throws FileNotFoundException {
    Scanner scanner = new Scanner(new File("file.txt"));
}
```

---

## 📦 Array Exceptions

### ❌ NullPointerException
Since arrays are reference types, their variable can be `null` and checking a null array might throw an exception. Avoid this by using:

```java
int[] numbers = null;
int size = numbers.length; // Throws NPE
```
**Fix:**
```java
int size = numbers == null ? 0 : numbers.length;
```

### 🚫 ArrayIndexOutOfBoundsException
A fairly common exception that occurs when trying to access a non-existent element of the array.  
**Check before accessing:**
```java
if (index < 0 || index > hardCodedArray.length - 1) {
    System.out.println("The index is out of bounds.");
} else {
    System.out.println(hardCodedArray[index]);
}
```

---

## 🎯 Be proactive and throw some Exceptions yourself
Any object of the `Throwable` class or its subclasses can be thrown using the `throw` statement.

The common practice is to throw an exception when and only when the method preconditions are broken, that is when it cannot be performed under the current conditions.

If a client can reasonably be expected to recover from an exception, make it a checked exception. If a client cannot do anything to recover, make it an unchecked exception.


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
We know that Java Input Streams should be closed after they are used to release system resources. This example shows why you have to think carefully when handling Exceptions and think of all possible scenarios and program behaviour carefully.


**Naive aka how Tamim would do it example:**
```java
Reader reader = null;

try {
    reader = new FileReader("file.txt"); //assume file exists
    // code which may throw an exception
} finally {
    reader.close();
}
```
We put the `close()` method in `finally` block so exceptions cannot affect the invocation of the `close()` method.  
However, what if the `close()` method raises an exception itself? So we will have 2 exceptions, one was raised in the try block and the other one was raised in the `finally` block. So Exception2 will be raised outside the method and we wont even know that try block raised an exception at all.

---

### 🛡 Improved Handling
Okay, lets upgrade our code so we handle Exception2 right after it was thrown and this way the method will return Exception1.

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

This works but we still want to save information about both Exceptions instead of just Exception1.

---

## ♻️ try-with-resources(what a giga-chad)
Automatically closes resources that implement `AutoCloseable`:

```java
try (Reader reader1 = new FileReader("file1.txt");
     Reader reader2 = new FileReader("file2.txt")) {
    // some code
}
```

All the objects declared in the round parenthesis will have a close method implicitly invoked. This guarantees closing all resources in a proper way. We can also have a finally block if needed.

Now let's go back to our two-exceptions case. If both the try block and close method throw exceptions Exception1 and Exception2, we will still get information on both exceptions:

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

So all outside resources like web or database connections should be closed. These can be wrapped by the try-with-resources statement so if there's an exception in the body, all resources will still be released properly. 

---

## 🐾 Assertions(a little bit gay ngl)

The part before the colon specifies the boolean expression that should be checked, and when it evaluates to false, an error is thrown. The part after it specifies the message that describes the error.
Now, if we run the code with the -ea flag (java -ea BrokenInvariants), the program will throw an error and terminate right in the Cat constructor:

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
- ✅ If condition true → nothing happens.
- ❌ If condition false & assertions enabled → throws `AssertionError` and stop the program unless caught.
- 🚫 If assertions disabled (default) → the assert line is ignored entirely.

---