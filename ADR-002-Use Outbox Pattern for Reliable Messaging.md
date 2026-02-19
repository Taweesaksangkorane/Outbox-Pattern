# ADR-002: Use Outbox Pattern for Reliable Messaging

## Title

Use Outbox Pattern to Ensure Reliable Event Publishing


## Context
In an event-driven microservices architecture, when a service performs a business operation (e.g., creating an order), it must:
1.	Save data to its database
2.	Publish an event to a message broker
However, the database and message broker are separate systems and do not share a transaction.
If the database commit succeeds but event publishing fails (or vice versa), the system becomes inconsistent.
Distributed transactions (e.g., two-phase commit) are complex, slow, and not scalable in microservices environments.
We need a reliable way to guarantee that events are not lost.


## Decision
We will implement the Outbox Pattern.

**The service will:**
- Save business data and event data into the same database transaction
- Store the event in a dedicated Outbox table
- Use a background worker to read events from the Outbox table
- Publish events asynchronously to the message broker
- Mark events as SENT after successful publishing


## Rationale
The Outbox Pattern ensures atomicity between business data and event storage because both are committed in the same database transaction.
If the transaction succeeds, the event is guaranteed to exist in the Outbox table.


## This approach improves reliability while maintaining scalability.
## Consequences

### Pros – What becomes easier?
- Reliable event delivery
- No need for distributed transactions
- Increased data consistency
- Production-proven pattern

### Cons – What becomes more difficult?
- Requires additional Outbox table
- Requires background worker process
- Eventual consistency instead of immediate consistency
- Possible duplicate message handling needed


## Sample Code

**Example: Saving business data + outbox event in same transaction (Python with SQLite)**

```python
import sqlite3
import json
from datetime import datetime

conn = sqlite3.connect("orders.db")
cursor = conn.cursor()

try:
    cursor.execute("BEGIN")

    # Save business data
    cursor.execute(
        "INSERT INTO orders (id, amount) VALUES (?, ?)",
        (123, 250)
    )

    # Save event to outbox table
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
