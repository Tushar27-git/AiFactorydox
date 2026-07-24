# Enterprise AI Software Factory
# Architecture Design Specification (ADS)

> **Document ID:** ADS-021-v4
>
> **Document Name:** Workflow State Machine — Runtime & Orchestration
>
> **Version:** 2.0.0
>
> **Status:** Draft
>
> **Classification:** Core Runtime Specification
>
> **Importance:** CRITICAL
>
> **Depends On:** ADS-021-v1
>
> **Depends On:** ADS-021-v2
>
> **Depends On:** ADS-021-v3
>
> **Next:** ADS-021-v5 — Complete Workflow Simulation

---

# Executive Summary

This document defines the production runtime of the Workflow State Machine.

Previous documents defined architecture, algorithms and communication contracts.

This specification explains how the Workflow State Machine executes inside production infrastructure.

Topics include

- Runtime services
- Scheduler
- Worker orchestration
- Lease management
- Checkpoint persistence
- Failure recovery
- Queue topology
- Scaling
- Disaster recovery

---

# Runtime Philosophy

The runtime follows seven principles.

- Stateless orchestration
- Durable workflow state
- Event sourcing
- Deterministic execution
- Horizontal scalability
- Automatic recovery
- Complete observability

---

# Runtime Architecture

```mermaid
flowchart TB

Gateway["Enterprise API Gateway"]

Coordinator["Workflow Coordinator Cluster"]

Scheduler["Distributed Scheduler"]

Queue["Workflow Queue"]

Lease["Lease Manager"]

Workers["Workflow Workers"]

Checkpoint["Checkpoint Store"]

StateDB["Workflow State Database"]

Kafka["Event Bus"]

Observe["Observability"]

Gateway --> Coordinator

Coordinator --> Scheduler

Scheduler --> Queue

Queue --> Workers

Workers --> Lease

Workers --> Checkpoint

Workers --> StateDB

Workers --> Kafka

Kafka --> Observe

Observe --> Coordinator
```

Every workflow follows this runtime topology.

---

# Runtime Components

| Component | Responsibility |
|------------|----------------|
| Workflow Coordinator | Workflow orchestration |
| Scheduler | Worker assignment |
| Queue Manager | Workflow scheduling |
| Lease Manager | Workflow ownership |
| Worker Pool | State execution |
| Checkpoint Store | Recovery |
| State Database | Persistent workflow state |
| Event Bus | Event publication |
| Observability | Runtime telemetry |

Every service is independently deployable.

---

# Runtime Lifecycle

```mermaid
stateDiagram-v2

[*] --> Created

Created --> Queued

Queued --> Assigned

Assigned --> Executing

Executing --> Checkpoint

Checkpoint --> Executing

Executing --> Completed

Executing --> Failed

Failed --> Recovery

Recovery --> Assigned

Recovery --> Escalated

Completed --> [*]

Escalated --> [*]
```

---

# Workflow Coordinator

The Workflow Coordinator is responsible for

- Creating workflows
- Managing transitions
- Assigning workers
- Persisting state
- Emitting events
- Coordinating recovery

The coordinator never performs subsystem work.

---

# Distributed Scheduler

The Scheduler decides

- Which workflow executes next
- Which worker receives it
- Which queue it enters
- When execution begins

Scheduling inputs

- Priority
- Dependencies
- Resource availability
- Tenant quotas
- Worker health
- SLA policies

---

# Queue Topology

```text
Planning Queue

↓

TDD Queue

↓

Execution Queue

↓

QA Queue

↓

Security Queue

↓

Deployment Queue
```

Each queue is independent.

Each queue supports retries and dead-letter handling.

---

# Worker Pool

Workers are disposable.

Each worker

- Acquires workflow lease
- Executes one state transition
- Persists checkpoint
- Emits telemetry
- Releases lease

Workers never retain persistent state.

---

# Lease Manager

Workflow ownership is controlled through renewable leases.

Lease Metadata

- Workflow ID
- Worker ID
- Lease Start
- Lease Expiration
- Heartbeat
- Renewal Count

Expired leases return ownership to the Scheduler.

---

# Checkpoint Store

Every transition generates two checkpoints

```text
Pre-Transition

↓

Transition

↓

Post-Transition
```

Stored data

- Workflow Context
- State
- Owner
- Retry Counter
- Active Tasks
- Event Offset
- Timestamp

---

# State Persistence

Persistent state contains

```text
Workflow

↓

Current State

↓

History

↓

Artifacts

↓

Audit Trail

↓

Events
```

State is append-only.

---

# Runtime Communication

| Communication | Protocol |
|---------------|-----------|
| External APIs | HTTPS |
| Internal Services | gRPC |
| Event Streaming | Kafka |
| Agent Tools | MCP |
| Telemetry | OpenTelemetry |

---

# Cache Layers

```text
L1

Worker Cache

↓

L2

Workflow Cache

↓

L3

Organization Cache
```

Cached objects include

- Workflow Metadata
- Policy Decisions
- Transition Graphs
- Runtime Configuration

---

# Runtime Sequence

```mermaid
sequenceDiagram

Client->>Gateway: Create Workflow

Gateway->>Coordinator: Workflow Request

Coordinator->>Scheduler: Schedule

Scheduler->>Worker: Assign Lease

Worker->>Checkpoint Store: Save Checkpoint

Worker->>State Database: Persist State

Worker->>Kafka: Publish Event

Kafka-->>Coordinator: Workflow Updated

Coordinator-->>Client: Success
```

---

# Resource Allocation

Worker assignment considers

- Queue Depth
- Workflow Priority
- Estimated Runtime
- CPU
- Memory
- Tenant Limits
- Worker Health

Assignments are dynamic.

---

# Runtime Guarantees

The Workflow State Machine guarantees

- Deterministic execution
- Durable state
- Replay support
- Horizontal scalability
- Checkpoint recovery
- Immutable history
- Observable execution

---

# End of Part 1
