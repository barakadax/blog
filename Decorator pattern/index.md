# Decorator Pattern
- [What is the Decorator Pattern](#what-is-the-decorator-pattern)
- [The Ice Cream Analogy](#the-ice-cream-analogy)
- [Code Examples](#code-examples)
  - [C#](#c)
  - [Python](#python)
  - [Rust](#rust)
- [Sources](#sources)

## What is the Decorator Pattern

The **Decorator Pattern** is a structural design pattern that allows you to dynamically attach new behaviors to objects by placing them inside special wrapper objects that contain these behaviors.

Instead of subclassing (which extends behavior statically at compile-time), you "wrap" the original object.
The wrapper implements the same interface as the wrapped object, delegating the requests to it while adding pre- and post-processing logic before or after.

## The Ice Cream Analogy

Imagine your friend is heading out to **buy ice cream** (this is our core operation).
You want to modify this behavior by introducing extra steps, but you don't want to change their core task of buying ice cream.
So, you wrap their trip with a decorator:

1. **Before** they leave, you decorate their action: *"Hey, buy one for me as well!"*
2. They perform the **Core Action**: They go and buy ice cream.
3. **After** they return and finish, you decorate their action again: *"Hey, throw the garbage away!"*

By wrapping the core action, you've added new behaviors before and after it without changing how ice cream is bought.

## Code Examples

### C#

In C#, we implement this by creating a wrapper class that implements the same interface as the target class and accepts the target class instance through its constructor.

```csharp
using System;

public interface IIceCreamRunner {
    void BuyIceCream();
}

// Concrete component (Core Action)
public class SimpleRunner : IIceCreamRunner {
    public void BuyIceCream() {
        Console.WriteLine("Buying ice cream...");
    }
}

// Decorator
public class HelpfulRunner : IIceCreamRunner {
    private readonly IIceCreamRunner _inner;

    public HelpfulRunner(IIceCreamRunner inner) {
        _inner = inner;
    }

    public void BuyIceCream() {
        Console.WriteLine("Before: Hey, bring me an ice cream too!");
        _inner.BuyIceCream(); // Delegate core action
        Console.WriteLine("After: Hey, throw away the garbage when you're done!");
    }
}

public class Program {
    public static void Main() {
        IIceCreamRunner runner = new HelpfulRunner(new SimpleRunner());
        runner.BuyIceCream();
    }
}
```

### Python

Python has native support for decorators using the `@` symbol, making it incredibly simple to wrap functions.

```python
def ice_cream_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before: Hey, bring me an ice cream too!")
        result = func(*args, **kwargs) # Delegate core action
        print("After: Hey, throw away the garbage when you're done!")
        return result
    return wrapper

@ice_cream_decorator
def buy_ice_cream():
    print("Buying ice cream...")

# Calling the decorated function
buy_ice_cream()
```

### Rust

In Rust, we can implement the decorator pattern using traits and generics to wrap behaviors statically or dynamically.

```rust
trait IceCreamRunner {
    fn buy_ice_cream(&self);
}

// Concrete component (Core Action)
struct SimpleRunner;
impl IceCreamRunner for SimpleRunner {
    fn buy_ice_cream(&self) {
        println!("Buying ice cream...");
    }
}

// Decorator wrapping another type implementing IceCreamRunner
struct HelpfulRunner<T: IceCreamRunner> {
    inner: T,
}

impl<T: IceCreamRunner> IceCreamRunner for HelpfulRunner<T> {
    fn buy_ice_cream(&self) {
        println!("Before: Hey, bring me an ice cream too!");
        self.inner.buy_ice_cream(); // Delegate core action
        println!("After: Hey, throw away the garbage when you're done!");
    }
}

fn main() {
    let runner = HelpfulRunner { inner: SimpleRunner };
    runner.buy_ice_cream();
}
```

## Sources

- [Refactoring Guru: Decorator Pattern](https://refactoring.guru/design-patterns/decorator)
- [Decorator Pattern (Wikipedia)](https://en.wikipedia.org/wiki/Decorator_pattern)
