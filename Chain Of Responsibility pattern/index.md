# Chain of Responsibility Pattern
- [Overview](#overview)
- [Real-World Analogy](#real-world-analogy)
- [Where it is Used](#where-it-is-used)
- [How it Works](#how-it-works)
- [Code Examples](#code-examples)
  - [C#](#c)
  - [Python](#python)
  - [Rust](#rust)
- [Dynamic Reordering](#dynamic-reordering)
- [Sources](#sources)

## Overview

The **Chain of Responsibility** is a behavioral design pattern that passes requests along a chain of handlers.
Upon receiving a request, each handler decides either to process it or pass it to the next handler in the chain, decoupling sender and receiver.

## Real-World Analogy

Consider a **Corporate Expense Approval** process:
1. **Team Lead:** Can approve expenses up to $500. Escalates higher requests to the Department Manager.
2. **Department Manager:** Can approve expenses up to $5,000. Escalates higher requests to the VP of Finance.
3. **VP of Finance:** Can approve expenses up to $50,000. Escalates higher requests to the CEO.

## Where it is Used

* **HTTP Middleware:** Frameworks like ASP.NET Core and Express.js pipe requests through a sequence of logging, rate-limiting, authentication, and routing handlers.
* **GUI Event Bubbling:** User input events (e.g., clicks) propagate from inner elements up through parent layout containers until handled.
* **Log Filters:** Logging libraries route log messages to different destinations (Console, Database, File) based on severity.

## How it Works

The pattern maintains handler execution order and delegation using three core concepts:
1. **The Link (`next` reference):** Each handler maintains a reference to the next handler in the chain, structuring it as a singly linked list.
2. **Base Delegation:** Successful handlers delegate execution to the next link by calling the base class method.
In C#, the `base` keyword is a language keyword referring to the parent class (`BaseHandler`).
In Python, the `super()` function is used. In Rust, handlers explicitly invoke the stored member.
If a check fails, delegation is bypassed, short-circuiting the chain.
3. **Fluent Chaining / Instantiation:** Links are set programmatically during startup or configuration, making it easy to rearrange handlers dynamically.

## Code Examples

Below is a **Request Validation Pipeline** using three handlers: **Rate Limiting**, **Authentication**, and **Authorization**.

### C#

```csharp
using System;

public record Request(string? User, string Role, bool IsUnderRateLimit);

public interface IHandler
{
    IHandler SetNext(IHandler next);
    bool Handle(Request req);
}

public abstract class BaseHandler : IHandler
{
    private IHandler? _next;

    public IHandler SetNext(IHandler next)
    {
        _next = next;
        return next;
    }

    public virtual bool Handle(Request req)
    {
        return _next == null || _next.Handle(req);
    }
}

public class RateLimitHandler : BaseHandler
{
    public override bool Handle(Request req)
    {
        if (req.IsUnderRateLimit)
        {
            return base.Handle(req);
        }
        Console.WriteLine("Blocked by Rate Limiter");
        return false;
    }
}

public class AuthenticationHandler : BaseHandler
{
    public override bool Handle(Request req)
    {
        if (!string.IsNullOrEmpty(req.User))
        {
            return base.Handle(req);
        }
        Console.WriteLine("Blocked by Authenticator");
        return false;
    }
}

public class AuthorizationHandler : BaseHandler
{
    public override bool Handle(Request req)
    {
        if (req.Role == "Admin")
        {
            return true;
        }
        Console.WriteLine("Blocked by Authorizer");
        return false;
    }
}

public class Program
{
    public static void Main()
    {
        var req = new Request("Alice", "Admin", true);

        var limit = new RateLimitHandler();
        var auth = new AuthenticationHandler();
        var authz = new AuthorizationHandler();

        limit.SetNext(auth).SetNext(authz);

        bool success = limit.Handle(req);
        Console.WriteLine($"Request allowed: {success}");
    }
}
```

### Python

```python
from typing import Optional

class Request:
    def __init__(self, user: Optional[str], role: str, is_under_rate_limit: bool):
        self.user = user
        self.role = role
        self.is_under_rate_limit = is_under_rate_limit

class Handler:
    def __init__(self):
        self._next: Optional[Handler] = None

    def set_next(self, next_handler: "Handler") -> "Handler":
        self._next = next_handler
        return next_handler

    def handle(self, req: Request) -> bool:
        if self._next:
            return self._next.handle(req)
        return True

class RateLimitHandler(Handler):
    def handle(self, req: Request) -> bool:
        if req.is_under_rate_limit:
            return super().handle(req)
        print("Blocked by Rate Limiter")
        return False

class AuthenticationHandler(Handler):
    def handle(self, req: Request) -> bool:
        if req.user:
            return super().handle(req)
        print("Blocked by Authenticator")
        return False

class AuthorizationHandler(Handler):
    def handle(self, req: Request) -> bool:
        if req.role == "Admin":
            return True
        print("Blocked by Authorizer")
        return False

if __name__ == "__main__":
    req = Request("Alice", "Admin", True)

    limit = RateLimitHandler()
    auth = AuthenticationHandler()
    authz = AuthorizationHandler()

    limit.set_next(auth).set_next(authz)

    success = limit.handle(req)
    print(f"Request allowed: {success}")
```

### Rust

```rust
struct Request {
    user: Option<String>,
    role: String,
    is_under_rate_limit: bool,
}

trait Handler {
    fn handle(&self, req: &Request) -> bool;
}

struct RateLimitHandler {
    next: Option<Box<dyn Handler>>,
}

impl RateLimitHandler {
    fn new(next: Option<Box<dyn Handler>>) -> Self {
        Self { next }
    }
}

impl Handler for RateLimitHandler {
    fn handle(&self, req: &Request) -> bool {
        if req.is_under_rate_limit {
            return match &self.next {
                Some(next_handler) => next_handler.handle(req),
                None => true,
            };
        }
        println!("Blocked by Rate Limiter");
        false
    }
}

struct AuthenticationHandler {
    next: Option<Box<dyn Handler>>,
}

impl AuthenticationHandler {
    fn new(next: Option<Box<dyn Handler>>) -> Self {
        Self { next }
    }
}

impl Handler for AuthenticationHandler {
    fn handle(&self, req: &Request) -> bool {
        if req.user.is_some() {
            return match &self.next {
                Some(next_handler) => next_handler.handle(req),
                None => true,
            };
        }
        println!("Blocked by Authenticator");
        false
    }
}

struct AuthorizationHandler {
    next: Option<Box<dyn Handler>>,
}

impl AuthorizationHandler {
    fn new(next: Option<Box<dyn Handler>>) -> Self {
        Self { next }
    }
}

impl Handler for AuthorizationHandler {
    fn handle(&self, req: &Request) -> bool {
        if req.role == "Admin" {
            return match &self.next {
                Some(next_handler) => true,
                None => true,
            };
        }
        println!("Blocked by Authorizer");
        false
    }
}

fn main() {
    let req = Request {
        user: Some("Alice".to_string()),
        role: "Admin".to_string(),
        is_under_rate_limit: true,
    };

    let authz = AuthorizationHandler::new(None);
    let auth = AuthenticationHandler::new(Some(Box::new(authz)));
    let limit = RateLimitHandler::new(Some(Box::new(auth)));

    let success = limit.handle(&req);
    println!("Request allowed: {}", success);
}
```

## Dynamic Reordering

By building pipelines dynamically, you can easily rearrange the order of handlers without modifying their core logic.

### Practical Scenario: DDoS Defense
* **Normal Mode:** `Authentication -> Authorization -> Rate Limiting`
  Authenticates user identity before checking rate limits (allows personalized limit policies).
* **Under DDoS Attack:** `Rate Limiting -> Authentication -> Authorization`
  Puts rate limiting first to reject spikes immediately, shielding the database/auth servers from expensive lookups.

### Reordering Configurations

#### C#
```csharp
var limit = new RateLimitHandler();
var auth = new AuthenticationHandler();
var authz = new AuthorizationHandler();

auth.SetNext(authz).SetNext(limit);
auth.Handle(req);

authz.SetNext(null!);
limit.SetNext(auth).SetNext(authz);
limit.Handle(req);
```

#### Python
```python
limit, auth, authz = RateLimitHandler(), AuthenticationHandler(), AuthorizationHandler()

auth.set_next(authz).set_next(limit)
auth.handle(req)

authz._next = None
limit.set_next(auth).set_next(authz)
limit.handle(req)
```

#### Rust
In Rust, the linked structure is configured during instantiation:
```rust
fn main() {
    let req = Request {
        user: Some("Alice".to_string()),
        role: "Admin".to_string(),
        is_under_rate_limit: true,
    };

    let authz = AuthorizationHandler::new(None);
    let auth = AuthenticationHandler::new(Some(Box::new(authz)));
    let limit = RateLimitHandler::new(Some(Box::new(auth)));

    let success = limit.handle(&req);
    println!("Request status: {}", success);
}
```

## Sources

- [Refactoring Guru: Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility)
- [Wikipedia: Chain of Responsibility Pattern](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)
- [Microsoft Learn: Middleware in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/)
