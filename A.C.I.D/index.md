# ACID
- [Atomicity](#atomicity)
- [Consistency](#consistency)
- [Isolation](#isolation)
- [Durability](#durability)
- [Best Code Practices: The Actor Model](#best-code-practices-the-actor-model)
- [Where to Use ACID](#where-to-use-acid)
- [Sources](#sources)

## Atomicity

Atomicity ensures that a transaction is treated as a single, indivisible unit of work.
Either all operations within the transaction succeed, or none do.
If any operation fails, the entire transaction is rolled back, leaving the current state without any change.

*Example:* In a bank transfer, money must be debited from Account A and credited to Account B.
If the credit operation fails, the debit must be rolled back to prevent money from disappearing or being duplicated.

## Consistency

Consistency ensures that a transaction only transitions the database from one valid state to another, maintaining all predefined schema constraints, rules, and triggers.
It guarantees that database invariants are never violated.

*Example:* If a bank database has a constraint that account balances cannot drop below zero, a transaction attempting to withdraw more than the available balance will be rejected to maintain consistency.

## Isolation

Isolation ensures that concurrent transactions execute without interfering with each other.
The intermediate state of a transaction is invisible to other concurrent transactions, producing the same database state as if they were executed sequentially.

*Example:* If User A attempts to withdraw 50 from a shared account with a 60 balance, they enter an intermediate state where the database calculates the new balance ($10) but hasn't finalized (committed) the change yet.
Database isolation ensures that if User B checks the account balance at that exact time, they will either see the original $60 or be forced to wait until User A finishes.
User B will never see an intermediate, uncommitted "temporary" state, preventing them from making decisions based on data that might roll back.

## Durability

Durability guarantees that once a transaction commits, its changes are permanently saved in non-volatile storage.
The updates will not be lost even in the event of a system crash, power outage, or database restart.

*Example:* Once a bank transaction is confirmed as completed, the record of that transaction is guaranteed to survive a sudden database server failure.
For more on data persistency, see the [3-2-1 backup rule](https://barakadax.github.io/blog?article=3-2-1%20backup%20rule) article.

## Best Code Practices: The Actor Model

Implementing ACID transactions in distributed systems can be challenging.
A modern best practice is using [Actor models](https://barakadax.github.io/blog?article=Actor%20model).
In the Actor model, each actor manages its own state and processes incoming messages sequentially from a single queue.
Since an actor handles only one message at a time, it naturally guarantees **Isolation** without requiring database row locks or complex multi-thread synchronization.
Furthermore, because state transitions are localized, the actor can validate business rules before modifying state, ensuring **Consistency** in a single-threaded execution model.
By using virtual actor frameworks like Microsoft Orleans in C#, you receive out-of-the-box database persistence which guarantees **Durability** and service discovery.
Service discovery ensures a 'single-activation' (enforcing **Isolation**), meaning no duplicate actor instances exist across a distributed environment like multiple Kubernetes pods.
While a single actor secures its own state, transactions spanning *multiple* actors require coordination.
This is typically handled using:
- **Saga pattern:** Compensating actions for rollback.
- **Two-Phase Commit (2PC):** A coordinator prevents inconsistencies across nodes (Orleans has built-in transactions).

## Where to Use ACID

ACID principles are essential when data integrity and correctness are more critical than raw system performance.
Common use cases include:
- Banking and financial systems
- E-commerce transactions and checkout systems
- Booking and reservation platforms
- Medical and healthcare databases
- Authentication and authorization systems
- Data pipelines

## Sources

- [ACID (Wikipedia)](https://en.wikipedia.org/wiki/ACID)
- [Database Transaction (Wikipedia)](https://en.wikipedia.org/wiki/Database_transaction)
- [Orleans transactions](https://learn.microsoft.com/en-us/dotnet/orleans/grains/transactions)
