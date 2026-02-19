ADR-001: Adopt Event-Driven Architecture
Title

Adopt Event-Driven Architecture for Service Communication

Context

Our system is built using a microservices architecture where each service owns its own database. Services must communicate with each other when business events occur, such as OrderCreated or PaymentCompleted.

Traditional synchronous communication (e.g., REST calls between services) tightly couples services and increases the risk of cascading failures. In addition, high scalability and asynchronous processing are required for handling growing workloads.

We need a reliable and scalable communication model between services.

Decision

We will adopt an Event-Driven Architecture (EDA) where services communicate by publishing and subscribing to events through a message broker (e.g., Kafka or RabbitMQ).

Each service:

Publishes events when a business state changes

Subscribes to relevant events from other services

Processes events asynchronously

Rationale

Event-Driven Architecture provides loose coupling between services. Services do not directly call each other but instead react to events.

This approach:

Improves scalability by enabling asynchronous processing

Reduces tight coupling between services

Enhances system resilience (one service failure does not immediately break others)

Supports distributed systems and cloud-native environments

EDA aligns well with microservices architecture principles.

Consequences
Pros – What becomes easier?

Better scalability

Loose coupling between services

Independent service deployment

Improved fault tolerance

Cons – What becomes more difficult?

Increased architectural complexity

Event monitoring and debugging become harder

Handling eventual consistency

Requires message broker infrastructure

Sample Code
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


finally:
    conn.close()
