# ADR-0001: Choose Async Messaging Platform

**Date:** 2026-01-15
**Status:** Accepted
**Deciders:** Platform Team, Integration Architects

## Context

Our e-commerce platform needs to decouple the order processing pipeline from
inventory, fulfillment, and notification services. Currently these are direct
synchronous HTTP calls, which creates cascading failures when downstream services
are slow or unavailable. We evaluated async messaging options.

## Decision

We will adopt **Apache Kafka** as our primary event streaming platform for
domain events (OrderPlaced, InventoryReserved, ShipmentCreated).
RabbitMQ will be retained for task queues where work-queue semantics are
needed (e.g., email sending, PDF generation).

## Options Considered

### Option A: Apache Kafka
- Pros: High throughput, log retention (replay), strong ecosystem, consumer groups
- Pros: Event sourcing friendly — retained log is the source of truth
- Cons: Operational complexity, requires Zookeeper (or KRaft), steep learning curve
- Cons: Not ideal for task queues — no built-in dead-letter queue semantics out of box

### Option B: RabbitMQ (for everything)
- Pros: Simpler ops, excellent task queue semantics, dead-letter exchange built-in
- Pros: Team already has experience
- Cons: Messages deleted on consumption — no replay capability
- Cons: Throughput ceiling lower than Kafka at our projected order volumes

### Option C: AWS SQS/SNS
- Pros: Zero ops, managed, integrates with existing AWS footprint
- Cons: Vendor lock-in — multi-cloud strategy makes this a risk
- Cons: No log retention / replay; SNS fan-out patterns less flexible than Kafka topics

## Consequences

- **Easier:** Downstream services can replay events for audit, reprocessing, debugging
- **Harder:** Team needs Kafka operational training; local dev setup is heavier
- **Creates:** Decision needed on Kafka topic naming conventions (see ADR-0002)
- **Creates:** Decision needed on schema registry (Avro vs Protobuf vs JSON Schema)
- **Accepted trade-off:** We are accepting operational complexity in exchange for
  replay capability and the ability to add new consumers without touching producers.
