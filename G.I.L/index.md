# Global Interpreter Lock (GIL)
- [What it is](#what-it-is)
- [Why we need it](#why-we-need-it)
- [How it works](#how-it-works)
- [Pros and Cons](#pros-and-cons)
- [Sources](#sources)

## What it is

The Global Interpreter Lock (GIL) is a **mutex** (mutual exclusion lock) for interpreters to synchronize the execution of threads,
Preventing multiple native threads from executing bytecode at the same time within a single process.

## Why we need it

The GIL exists in interpreters for historical and technical reasons, primarily centered around simplicity and thread safety for the interpreter's internal state.

### Preventing race conditions
Without a lock, concurrent threads could access and modify the same internal data structures simultaneously,
leading to data corruption or crashes. The GIL provides a coarse-grained synchronization mechanism that avoids these issues
but at the cost of parallelism.

### Memory management
Many interpreters use tracking mechanisms like **reference counting** for memory management. When an object is referenced,
its count increases; when it's dereferenced, it decreases.
In a multi-threaded environment without a GIL, two threads could simultaneously modify tracing data or reference counts
of the same object, leading to memory leaks or premature deallocation.

---

## How it works

When a thread wants to run, it must first acquire the GIL.
1. **Acquire**: The thread waits to acquire the global lock.
2. **Execute**: It executes for a fixed interval (based on time or a number of instructions).
3. **Release**: The thread releases the lock, allowing other threads to compete for it.

This cycle continues throughout the execution of the process, ensuring that only one thread is executing bytecode at any given moment.

---

## Pros and Cons

### Pros
- **Simplicity**: Easier to implement the interpreter and its memory management.
- **Single-threaded Performance**: Faster for single-threaded applications since there is no overhead from multiple fine-grained locks.
- **Native Extension Compatibility**: Simplifies the API for writing native extensions.

### Cons
- **No True Parallelism**: CPU-bound tasks cannot take advantage of multiple cores within a single process.
- **Bottleneck**: Can become a major performance bottleneck for multi-threaded CPU-heavy applications.

---

## Sources

- [Wikipedia: Global Interpreter Lock](https://en.wikipedia.org/wiki/Global_interpreter_lock)
