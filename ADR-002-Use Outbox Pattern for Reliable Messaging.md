# ADR-002: Use Outbox Pattern to Ensure Reliable Event Publishing in E-Commerce Platform

## Title

Use Outbox Pattern to Ensure Reliable Event Publishing in E-Commerce
Platform

## Context

In our event-driven e-commerce system, when a customer places an order:

1.  The Order Service saves the order in its database
2.  It publishes an `OrderCreated` event

However, the database and message broker are separate systems.

If the database commit succeeds but event publishing fails, other
services (Payment, Inventory) may not be notified.

This is known as the dual-write problem.

Distributed transactions such as two-phase commit are complex and not
scalable in microservices environments.

Therefore, we need a reliable solution.

## Decision

We will implement the Outbox Pattern.

The Order Service will:

-   Save order data and the corresponding event in the same database
    transaction
-   Store the event in an Outbox table
-   Use a background worker to read pending events
-   Publish them to the message broker
-   Mark them as SENT after successful publishing

## Rationale

The Outbox Pattern ensures atomicity between business data and event
storage because both are committed in the same database transaction.

Even if publishing fails, the background worker retries, ensuring
at-least-once delivery and eventual consistency.

## Consequences

### Pros

-   Prevents message loss
-   Solves the dual-write problem
-   No need for distributed transactions
-   Reliable event publishing

### Cons

-   Additional Outbox table
-   Requires background worker
-   Eventual consistency instead of immediate consistency
-   Consumers must handle duplicate events (idempotency)

## Sample Code

Example: Saving business data and event in the same transaction
(Python + SQLite)

``` python
import sqlite3
import json
from datetime import datetime

conn = sqlite3.connect("orders.db")
cursor = conn.cursor()

try:
    cursor.execute("BEGIN")

    cursor.execute(
        "INSERT INTO orders (id, amount) VALUES (?, ?)",
        (123, 250)
    )

    event = {
        "event_type": "OrderCreated",
        "order_id": 123,
        "amount": 250
    }

    cursor.execute(
        "INSERT INTO outbox (event_type, payload, status, created_at) VALUES (?, ?, ?, ?)",
        ("OrderCreated", json.dumps(event), "PENDING", datetime.utcnow())
    )

    conn.commit()

except Exception as e:
    conn.rollback()
    raise e

finally:
    conn.close()
```
