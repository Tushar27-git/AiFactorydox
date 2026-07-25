# Enterprise AI Software Factory
# Architecture Design Specification (ADS)

> **Document ID:** ADS-038-v3
>
> **Document Name:** Enterprise Event Streaming, Messaging & Real-Time Data Platform — APIs, Events & Contracts
>
> **Version:** 1.0.0
>
> **Classification:** Enterprise Platform Plane
>
> **Importance:** CRITICAL
>
> **Depends On:** ADS-038-v1
>
> **Depends On:** ADS-038-v2
>
> **Next:** ADS-038-v4 — Runtime & Event Infrastructure

---

# Executive Summary

This document specifies the APIs, messaging contracts, event schemas, security policies, and integration interfaces used by the Enterprise Event Streaming Platform.

Every event contract is versioned.

Every topic is governed.

Every API is observable.

---

# REST APIs

## API-038-001

### Publish Event

```http
POST /events/v1/publish
```

Request

```json
{
  "eventType": "OrderCreated",
  "topic": "orders.created",
  "tenant": "tenant-a",
  "payload": {},
  "metadata": {},
  "traceId": "TRC-932001"
}
```

Response

```json
{
  "eventId": "EV-10021",
  "eventRecordId": "ER-24011",
  "publicationStatus": "Published"
}
```

---

## API-038-002

### Register Topic

```http
POST /events/v1/topics
```

Registers a governed event topic.

---

## API-038-003

### Register Schema

```http
POST /events/v1/schemas
```

Registers a new schema version in the Schema Registry.

---

## API-038-004

### Replay Events

```http
POST /events/v1/replay
```

Initiates a replay operation for an authorized scope.

---

## API-038-005

### Query Event

```http
GET /events/v1/events/{eventId}
```

Returns immutable event metadata.

---

# gRPC Service

```protobuf
service EventStreamingService {

  rpc PublishEvent(EventRequest)
      returns (EventResponse);

  rpc RegisterTopic(TopicRequest)
      returns (TopicResponse);

  rpc RegisterSchema(SchemaRequest)
      returns (SchemaResponse);

  rpc ReplayEvents(ReplayRequest)
      returns (ReplayResponse);

  rpc QueryEvent(QueryRequest)
      returns (EventRecord);
}
```

---

# Core Schemas

## Event

```yaml
event:

  eventId:

  eventType:

  source:

  tenant:

  schemaVersion:

  payload:

  metadata:

  timestamp:

  traceId:
```

---

## Event Record

```yaml
eventRecord:

  eventRecordId:

  event:

  topic:

  partition:

  offset:

  producer:

  publicationStatus:

  deliveryGuarantee:

  retentionPolicy:

  publishedAt:
```

---

## Stream Record

```yaml
streamRecord:

  streamRecordId:

  stream:

  topics:

  consumerGroups:

  retentionPolicy:

  deliveryGuarantee:

  operationalStatus:

  throughput:

  lastEvaluatedAt:
```

---

## Topic Record

```yaml
topicRecord:

  topicRecordId:

  topicName:

  partitions:

  replicationFactor:

  retentionPolicy:

  schemaRegistry:

  accessPolicy:

  operationalStatus:

  createdAt:
```

---

# MCP Tools

The platform exposes the following tools.

- publish_event
- query_event
- replay_events
- register_topic
- register_schema
- create_consumer_group
- stream_health
- topic_diagnostics

---

# Platform Events

## EVT-038-001

EventPublished

---

## EVT-038-002

SchemaRegistered

---

## EVT-038-003

TopicCreated

---

## EVT-038-004

ConsumerGroupCreated

---

## EVT-038-005

ReplayStarted

---

## EVT-038-006

ReplayCompleted

---

## EVT-038-007

DeadLetterQueued

---

## EVT-038-008

RetentionExpired

---

# Event Flow

```mermaid
flowchart LR

Producer

-->

EventGateway

-->

SchemaRegistry

-->

EventBroker

-->

StreamProcessor

-->

Consumer

-->

EventLedger
```

Every interaction produces immutable operational evidence.

---

# Request Validation Pipeline

Every API request follows the same validation pipeline.

```text
Receive Request

↓

Authenticate Producer

↓

Authorize Topic Access

↓

Validate Event Schema

↓

Validate Payload

↓

Apply Governance Policies

↓

Publish Event

↓

Persist Event Record

↓

Return Response
```

No event is accepted without completing the validation pipeline.

---

# Authentication

Supported authentication mechanisms

- OAuth 2.1
- OpenID Connect (OIDC)
- Mutual TLS (mTLS)
- JWT Bearer Tokens
- API Keys (Policy Controlled)
- SPIFFE / SPIRE Workload Identity

Every producer possesses a verifiable identity.

---

# Authorization

Access decisions evaluate

- Tenant
- Topic
- Producer Identity
- Consumer Group
- Event Type
- Schema Version
- Governance Policies

Authorization flow

```text
Request

↓

Identity

↓

Policy Engine

↓

Permit / Deny

↓

Audit Record
```

Authorization remains centrally governed.

---

# Consumer Group Record

Every managed consumer group is represented by an immutable Consumer Group Record.

```yaml
consumerGroupRecord:

  consumerGroupRecordId:

  consumerGroup:

  subscribedTopics:

  assignedPartitions:

  consumerInstances:

  rebalanceStrategy:

  offsetPolicy:

  operationalStatus:

  lastHeartbeatAt:
```

Consumer coordination remains independently managed from Topic and Stream state.

---

# Event Contracts

Every event contract defines

- Event Name
- Schema Version
- Required Fields
- Optional Fields
- Compatibility Policy
- Deprecation Timeline

Contracts are version-controlled.

---

# Schema Evolution Rules

Supported compatibility modes

| Mode | Description |
|------|-------------|
| Backward | New consumers read old events |
| Forward | Old consumers read new events |
| Full | Bidirectional compatibility |
| None | Breaking changes allowed with approval |

Compatibility policies are enforced by the Schema Registry.

---

# Governance Policies

Every topic defines

- Maximum Message Size
- Allowed Producers
- Allowed Consumers
- Retention Duration
- Replay Permissions
- Encryption Policy
- Compliance Classification

Policies remain immutable until versioned.

---

# Distributed Tracing

Trace propagation

```text
Producer

↓

Event Gateway

↓

Event Broker

↓

Stream Processor

↓

Consumer

↓

Event Ledger
```

Each stage appends OpenTelemetry spans using the shared Trace ID.

---

# Prometheus Metrics

```text
event_publications_total

event_publication_failures_total

topic_registrations_total

schema_registrations_total

consumer_groups_total

consumer_rebalances_total

event_replay_requests_total

dead_letter_queue_size

event_delivery_latency_seconds

schema_validation_duration_seconds
```

Metrics expose real-time operational health.

---

# Structured Logging

Example

```json
{
  "eventId":"EV-10021",
  "eventRecord":"ER-24011",
  "topic":"orders.created",
  "consumerGroup":"analytics-service",
  "traceId":"TRC-932001",
  "publicationStatus":"Published"
}
```

Logs remain immutable and correlated.

---

# API Error Model

Standard error response

```json
{
  "code":"EVENT_SCHEMA_VALIDATION_FAILED",
  "message":"Payload does not conform to schema version 3.",
  "traceId":"TRC-932001",
  "timestamp":"2027-05-18T14:22:41Z"
}
```

All errors are auditable.

---

# Architecture Decision Records

## ADR-038-04

### Decision

Require schema validation before accepting any event.

### Status

Accepted

### Reason

Prevents invalid events from entering enterprise streams.

---

## ADR-038-05

### Decision

Separate Consumer Group Records from Stream and Topic Records.

### Status

Accepted

### Reason

Consumer lifecycle evolves independently from stream configuration.

---

## ADR-038-06

### Decision

Version all event contracts.

### Status

Accepted

### Reason

Supports long-term compatibility and controlled schema evolution.

---

# Operational Readiness Scorecard

| Capability | Status |
|------------|--------|
| REST APIs | ✅ Required |
| gRPC APIs | ✅ Required |
| Schema Registry | ✅ Required |
| Topic Governance | ✅ Required |
| Consumer Coordination | ✅ Required |
| OpenTelemetry | ✅ Required |
| Prometheus Metrics | ✅ Required |
| Immutable Contracts | ✅ Required |

---

# Related Documents

ADS-021-v5 — Workflow Kernel

ADS-023-v5 — Knowledge & Memory Platform

ADS-025-v5 — Compute & Infrastructure Platform

ADS-026-v5 — Security Platform

ADS-027-v5 — Observability Platform

ADS-030-v5 — Integration & Ecosystem Platform

ADS-037-v5 — Enterprise Edge, IoT & Cyber-Physical Systems Platform

ADS-038-v1 — Architecture

ADS-038-v2 — Event Algorithms & Lifecycle

ADS-038-v4 — Runtime & Event Infrastructure

---

# End of Document
