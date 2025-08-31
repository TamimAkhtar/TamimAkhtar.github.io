---
title: "Memory Management in Java: Stack, Heap, and Thread Memory"
description: "How Java memory works, Top of the line view"
date: 2025-08-31
categories: [notes]
tags: [Java, Memory, Stack, Heap, Threads]
---

Memory Allocation in Java is divided into 2 categories, Stack and Heap Memory. Both are used for different purposes and both have different characteristics:

## Stack Memory
Stores local variables, Method Calls, References to objects, organized as LIFO

Temporary memory that is used for function calls. Automatically allocated when a function starts and cleared when a function ends. Data exists only during the function's execution. A new call stack is allocated for each thread. So main thread has its own call stack for and worker thread will have its own call stack.

## Heap Memory
Stores objects and dynamic data allocated at runtime, Instance variables (inside objects), Class variables (static fields)

Share memory area for the whole process (all threads). Memory allocated to store objects (which are usually created using new keyword). Size depends on the data types and fields defined in its class. It is organized by the Garbage Collector. Objects lives beyond the method execution.

### Example: Stack and Heap Memory

```java
public class Main {
    public static void main(String[] args) {
        int a = 10;              // Stored in main thread's call stack
        String b = "CS";
        Person p = new Person(); // 'p' (reference) on stack, object in heap
        p.name = "Alice";        // 'name' field stored in heap (inside Person object)
    }
}

class Person {
    static String xyz = "Tamim";
    String name;
}
```

Main Thread's Call Stack stores
1. local variable a 
2. local reference p to Person object
3. local String reference b

Heap Memory stores:
1. Person object
2. field name of Person object
3. String Reference xyz
4. String "Tamim" (stored in a special part of heap called String pool since strings are also objects in java)
5. String "CS"


### Example: Server with Executor

```java
public static void main(String[] args) {
    try (ServerSocket server = new ServerSocket(LISTENING_ON_PORT)) {
        System.out.println("Server started!");
        Gson gson = new Gson();
        ExecutorService executor = Executors.newFixedThreadPool(5);

        while (true) {
            Socket client = server.accept(); // waits for a request
            executor.submit(() -> {
                try {
                    worker(client, gson); // run in its own thread
                } catch (Exception e) {
                    e.printStackTrace();
                }
            });
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```


In the above method, we are using Java executor which encapsulates 5 threads in a single pool and puts incoming requests in an internal queue and executes each request in a different thread. So everytime we receive a request, we call a worker method that runs in its own thread.

## Memory Flow When Requests Arrive

### First Request

When first request arrives, server.accept() creates a socket object in the heap (lets call it Socket Object 1)

**Main Thread's Stack Frame stores:**
- client reference to Socket object 1

**Heap Memory stores:**
- Socket object 1

**Thread 1 Stack Frame (called by worker) stores:**
- client reference to Socket object 1

### Second Request

When a second request arrives, server.accept() creates a new socket object in heap (lets call it Socket object 2)

**Main Thread's Stack Frame stores:**
- client reference to Socket object 2 (reassigned to new object)

**Heap Memory stores:**
- Socket object 1 (stays as long as there's a reference to it example in the thread 1 of executor, once no thread references it anymore, it becomes eligible for garbage collection)
- Socket object 2

**Thread 1 Stack Frame (called by worker) stores:**
- client reference to Socket object 1

**Thread 2 Stack Frame (called by worker) stores:**
- client reference to Socket object 2

So when a thread is called to executes its task, it gets its own stack frame with a parameter that points to the socket object in heap. So each thread has a private copy of the Socket reference in its own stack. Thats why threads can work independently with different socket objects. Each Thread's stack frame will also have their own local variables which may be declared in worker method.

### Lifecycle of Stack Frames

When a method is invoked, a new frame is pushed to the current thread’s stack.

When the method returns, its frame is popped and local variables are discarded.

Heap objects survive as long as they are referenced by any stack frame or static variable.
