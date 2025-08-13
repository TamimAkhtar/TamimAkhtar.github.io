---
title: "Handling Java Errors and Exceptions"
description: "How to deal with Java Exception handling"
date: 2025-08-13
categories: [notes]
tags: [Exceptions, Errors, Java]
---

As Java is an object oriented langauge, all exceptions are considered objects of special classes. The class hierarchy of Java Exceptions and Errors can be seen below where the base class is Java.lang.Throwable which has 2 direct subclasses, Error and Exception.

Error class represents: Low level exceptions in JVM like OutofMemoryError, StackOverflowError

Exception Class:
Runtime / Unchecked Exceptions: Checked for during program execution, doesnt prevent a code from compiling.
Checked Exceptions: Checked for by the compiler during compiling. The compiler checks wether the programmer expects the occurence of such exceptions in a program. If a programmer expects such exceptions, they have to indicate it otherwise the compiler cannot compile the program:

Public static String readLineFromFile() throws FileNOtFoundEXception() {
    Scanner scanner= new Scanner(newFile("file.txt"))
}

Here the programmer indicates that the Scanner constructor might throw a FileNotFoundException in method decleration. As a result, the method caller will need to decide whether to either handle the exception internally or throw it further to its caller method.

General Exception Handling:
An exception interrupts the normal execution of a program. After a line of code throws an exception, the Java runtime system attempts to find a suitable handler for it. Such a handler can be located in the same method where the exception occurred or in the calling method. As soon as a suitable handler is found and executed, the exception is considered as handled, and the program runs normally.

Suitable Handler 1: try - catch block -> if an exception occurs in try block, execute the code in catch block. Supports several catch blocks to handle multiple exceptions

Suitable Handler 2: try - catch - finally block (all statements present in finally block will always execute regardless of wether an exception occurs in try block )

Warning Handler: throw exception in method handler -> like declaring a warning that this method might throw an exception

Array Exceptions
NullPointerException: since Array is a reference type, its variable can be null and checking a null array might throw an exception: Avoid this by using: 

int[] numbers = null;
int size = numbers.length; // It throws an NPE
int size = numbers == null ? 0 : numbers.length;

Array Index out of Bounds Exception:This is a fairly common exception that occurs while attempting to access a non-existent element of the array. Check this by using:

 if (index < 0 || index > hardCodedArray.length - 1) {
            System.out.println("The index is out of bounds.");
        } else {
            System.out.println(hardCodedArray[index]);
        }
    }
}

Be Proactive and Throw Exceptions yourself

Any object of the Throwable class and all its subclasses can be thrown using the throw statement. 
The common practice is to throw an exception when and only when the method preconditions are broken, that is when it cannot be performed under the current conditions.
If a client can reasonably be expected to recover from an exception, make it a checked exception. If a client cannot do anything to recover, make it an unchecked exception.


For example, let's take a look at the following method that reads text from a file. In case the file is not found, the method throws an IOException:

public static String readTextFromFile(String path) throws IOException {
    // find a file by the specified path    

    if (!found) {
        throw new IOException("The file " + path + " is not found"); //throwing a checked exception
    }
    ..........................
}

 public void deposit(long amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Incorrect sum " + amount); //throws unchecked Exceptions
        }
        
        if (amount >= 100_000_000L) {
            throw new IllegalArgumentException("Too large amount"); //throws unchecked Exception
        }
        
        balance += amount;
    }

Assertions:

public Cat(String name, int age) {
    assert (age >= 0) : "Invalid age";
    this.name = name;
    this.age = age;
}

The part before the colon specifies the boolean expression that should be checked, and when it evaluates to false, an error is thrown. The part after it specifies the message that describes the error.
Now, if we run the code with the -ea flag (java -ea BrokenInvariants), the program will throw an error and terminate right in the Cat constructor:

Behavior:

If the condition is true, nothing happens.

If the condition is false and assertions are enabled (java -ea), it throws an AssertionError and usually stops the program unless caught.

If assertions are disabled (default), the assert line is ignored completely (as if it doesn’t exist).

Try with Resources
We know that Java Input Streams should be closed after they are used to release resources.

