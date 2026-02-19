# ADR-001: Adopt Event-Driven Architecture

## Title

Adopt Event-Driven Architecture for Service Communication

## Context

Our system is designed using a microservices architecture, where each service owns its own database and operates independently. Services must communicate when business events occur, such as OrderCreated or PaymentCompleted.

Traditional synchronous communication (e.g., REST calls between services) tightly couples services and increases the risk of cascading failures. If one service becomes unavailable, dependent services may also fail.

In addition, the system must support high scalability and asynchronous processing to handle increasing workloads efficiently.

Therefore, a reliable and scalable communication model between services is required.

## Decision

We will adopt an Event-Driven Architecture (EDA) where services communicate by publishing and subscribing to events through a message broker (e.g., Kafka or RabbitMQ).

Each service will:

- Publish events when a business state changes

- Subscribe to relevant events from other services

- Process events asynchronously

## Rationale

Event-Driven Architecture provides loose coupling between services. Instead of directly calling one another, services react to events emitted by other services.

This approach:

- Improves scalability by enabling asynchronous processing

- Reduces tight coupling between services

- Enhances system resilience (a failure in one service does not immediately break others)

- Supports distributed systems and cloud-native environments

Therefore, EDA is a suitable architectural style for scalable and resilient microservices systems.

## Consequences

### Pros – What becomes easier?

- Better scalability

- Loose coupling between services

- Independent service deployment

- Improved fault tolerance

### Cons – What becomes more difficult?

- Increased architectural complexity

- Event monitoring and debugging become harder

- Handling eventual consistency

- Requires message broker infrastructure

## Sample Code

Example: Publishing an event (Python with RabbitMQ)

```python
import json
import pika

# Connect to message broker
connection = pika.BlockingConnection(
    pika.ConnectionParameters(host='localhost')
)
channel = connection.channel()

# Declare durable queue
channel.queue_declare(queue='order_events', durable=True)

# Create event
event = {
    "event_type": "OrderCreated",
    "order_id": 123,
    "amount": 250
}

# Publish event
channel.basic_publish(
    exchange='',
    routing_key='order_events',
    body=json.dumps(event)
)

print("✓ Event published successfully")
connection.close()
```
