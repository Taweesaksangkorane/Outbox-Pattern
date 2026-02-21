# ADR-001: Adopt Event-Driven Architecture for Service Communication in E-Commerce Platform

## Title

Adopt Event-Driven Architecture for Service Communication in E-Commerce
Platform

## Context

Our system is an e-commerce platform where customers can place orders
online. The system consists of multiple microservices, including:

-   Order Service
-   Payment Service
-   Inventory Service
-   Notification Service

Each service owns its own database and operates independently.

When a customer places an order, multiple services must react to that
event. For example:

-   The Payment Service must process payment
-   The Inventory Service must reserve stock
-   The Notification Service must send confirmation

Using synchronous REST communication would tightly couple services and
increase the risk of cascading failures.

Therefore, a scalable and loosely coupled communication model is
required.

## Decision

We will adopt an Event-Driven Architecture (EDA).

Services will communicate by publishing and subscribing to events
through a message broker (e.g., Kafka or RabbitMQ).

## Rationale

Event-Driven Architecture enables loose coupling between services.
Instead of calling each other directly, services react to business
events.

This approach:

-   Improves scalability through asynchronous processing
-   Reduces service coupling
-   Increases resilience
-   Supports distributed deployment

This design introduces eventual consistency, which is acceptable for our
e-commerce domain.

## Consequences

### Pros

-   Better scalability
-   Independent service deployment
-   Reduced cascading failures
-   Supports high traffic workloads

### Cons

-   Increased architectural complexity
-   Harder debugging and event tracing
-   Eventual consistency between services
-   Requires message broker infrastructure

## Sample Code

Example: Publishing an event (Python with RabbitMQ)

``` python
import json
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost')
)
channel = connection.channel()

channel.queue_declare(queue='order_events', durable=True)

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

print("Event published successfully")
connection.close()
```
