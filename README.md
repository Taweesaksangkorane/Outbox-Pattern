# Outbox-Pattern
✅ ADR-001: Adopt Event-Driven Architecture
Title
Adopt Event-Driven Architecture for Service Communication
________________________________________
Context
Our system is built using a microservices architecture where each service owns its own database. Services must communicate with each other when business events occur, such as OrderCreated or PaymentCompleted.
Traditional synchronous communication (e.g., REST calls between services) tightly couples services and increases the risk of cascading failures. In addition, high scalability and asynchronous processing are required for handling growing workloads.
We need a reliable and scalable communication model between services.
________________________________________
Decision
We will adopt an Event-Driven Architecture (EDA) where services communicate by publishing and subscribing to events through a message broker (e.g., Kafka or RabbitMQ).
Each service:
•	Publishes events when a business state changes
•	Subscribes to relevant events from other services
•	Processes events asynchronously
________________________________________
Rationale
Event-Driven Architecture provides loose coupling between services. Services do not directly call each other but instead react to events.
This approach:
•	Improves scalability by enabling asynchronous processing
•	Reduces tight coupling between services
•	Enhances system resilience (one service failure does not immediately break others)
•	Supports distributed systems and cloud-native environments
EDA aligns well with microservices architecture principles.
________________________________________
Consequences
Pros – What becomes easier?
•	Better scalability
•	Loose coupling between services
•	Independent service deployment
•	Improved fault tolerance
Cons – What becomes more difficult?
•	Increased architectural complexity
•	Event monitoring and debugging become harder
•	Handling eventual consistency
•	Requires message broker infrastructure
________________________________________
Sample code
# Example: Publishing an event (simplified)
import json
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost')
)
channel = connection.channel()

channel.queue_declare(queue='order_events')

event = {
    "event_type": "OrderCreated",
    "order_id": 123,
    "amount": 250
}

channel.basic_publish(
    exchange='',
    routing_key='order_events',
    body=json.dumps(event)
)

print("Event published")
connection.close()
________________________________________
✅ ADR-002: Use Outbox Pattern for Reliable Messaging
________________________________________
Title
Use Outbox Pattern to Ensure Reliable Event Publishing
________________________________________
Context
In an event-driven microservices architecture, when a service performs a business operation (e.g., creating an order), it must:
1.	Save data to its database
2.	Publish an event to a message broker
However, the database and message broker are separate systems and do not share a transaction.
If the database commit succeeds but event publishing fails (or vice versa), the system becomes inconsistent.
Distributed transactions (e.g., two-phase commit) are complex, slow, and not scalable in microservices environments.
We need a reliable way to guarantee that events are not lost.
________________________________________
Decision
We will implement the Outbox Pattern.
The service will:
•	Save business data and event data into the same database transaction
•	Store the event in a dedicated Outbox table
•	Use a background worker to read events from the Outbox table
•	Publish events asynchronously to the message broker
•	Mark events as SENT after successful publishing
________________________________________
Rationale
The Outbox Pattern ensures atomicity between business data and event storage because both are committed in the same database transaction.
If the transaction succeeds, the event is guaranteed to exist in the Outbox table.
Publishing is handled separately and retried if it fails, ensuring at-least-once delivery without using distributed transactions.
This approach improves reliability while maintaining scalability.
________________________________________
Consequences
Pros – What becomes easier?
•	Reliable event delivery
•	No need for distributed transactions
•	Increased data consistency
•	Production-proven pattern
Cons – What becomes more difficult?
•	Requires additional Outbox table
•	Requires background worker process
•	Eventual consistency instead of immediate consistency
•	Possible duplicate message handling needed
________________________________________
Sample code
# Example: Saving business data + outbox event in same transaction
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
