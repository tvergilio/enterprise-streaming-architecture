# High Availability and Scaling Plan

## Document Metadata
- **Document Type**: Infrastructure Architecture Specification
- **Version**: 1.0
- **Author**: T. Vergilio
- **Context**: OTB Take-Home Exercise
- **Review Date**: 4 August 2025

## Executive Summary

This document defines the high availability, disaster recovery, and scaling strategies for the flight search platform. The design targets 99.9%+ uptime (≤ 8.77 h/year) whilst supporting elastic scaling from baseline operations to peak holiday traffic surges, aligned with business continuity requirements and cost optimisation objectives.

## Business Context

### Availability Requirements
- **Revenue Impact**: Platform outages estimated at £100k/hour during peak booking periods
- **Customer Experience**: Sub-40ms response times must be maintained during failure scenarios
- **Regulatory Compliance**: ATOL requirements mandate audit trail preservation during incidents
- **Market Competition**: High availability as competitive differentiator in travel search market

### Scaling Drivers  
- **Traffic Patterns**: 50+ million searches daily with 10x peak surge during holiday booking windows
- **Geographic Expansion**: Multi-region deployment supporting European market growth
- **Supplier Growth**: Architecture must accommodate rapid integration of new travel providers
- **Cost Efficiency**: Infrastructure costs must scale proportionally with business volume

## Architecture Principles

1. **No Single Points of Failure**: All critical components deployed across multiple availability zones
2. **Graceful Degradation**: Service continues with reduced functionality during partial outages
3. **Elastic Scaling**: Auto-scaling based on demand patterns with cost optimisation
4. **Data Consistency**: Strong consistency for financial data, eventual (≤ 1 s) consistency acceptable for search index
5. **Recovery Automation**: Automated failover and recovery with minimal manual intervention

## Technical Architecture

> **Deployment Assumption**
> The platform is deployed in a mainstream public cloud environment that offers managed or self-hosted options for distributed data stores, container orchestration and object storage.
> Exact vendor choices (AWS, GCP, Azure, on-prem Kubernetes) can be selected to meet cost, skillset and regulatory constraints.

| Tier / Service                                          | Deployment Specification                                                                                                                                       | Replication and Failover                                                                                       | Design Rationale                                                                                                                                  |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Kafka (distributed log)**                             | 3 brokers, one per Availability Zone; baseline 24 vCPU each; topic partitions sized to peak ingest (e.g. 24 for `supplier.raw`).                                                | Replication factor 3, minimum in‑sync replicas 2. Broker or AZ failure triggers automatic leader re‑election.  | Based on 40 million searches/day and 10,000 msg/s supplier feed. Designed with 100 % headroom for peak. Enables Flink's end‑to‑end exactly‑once processing. A 7-day topic retention policy supports replayability requirements. |
| **Flink JobManager (HA)**                               | 3 replicas (one per AZ) managed by orchestrator.                                                                                                                                | Leader election < 5 s. Checkpoint to object storage every 30 s; savepoints before deployment or schema change. | Recovery objective < 30 s. Savepoints are durable snapshots used for controlled updates and operational recoverability.                                     |
| **Flink TaskManagers**                                  | Autoscaled: baseline 3 workers per AZ, peaks to 10+ at holiday surge; each worker 4 vCPU and 16 GB RAM.                                                                         | Stateless; restarted on failure. Remaining TaskManagers resume work from latest checkpoint.                    | Kafka consumer lag maintained < 5 s at p95 throughput. Core stream transforms: normalise, enrich, deduplicate, anomaly detection.                           |
| **In‑memory Hot Cache (Redis)**                         | 1 primary in AZ‑a plus 2 replicas in AZ‑b and AZ‑c (cluster mode).                                                                                                              | Automatic promotion ≈ 10 s; client retries on failure.                                                         | Meets targets: 85% hit ratio. TTL 60 s, LFU eviction, \~2 GB per shard.                                                |
| **Search Index (ElasticSearch, OpenSearch or similar)** | 3 data nodes (one per AZ). Optional 3 dedicated masters.                                                                                                                        | 1 replica per shard; cluster remains green on AZ failure.                                                      | Brief states document search must support ranking and faceting. Newly indexed documents searchable within 1 s (refresh interval). No fares indexed.         |
| **Durable State Store (NoSQL)**                         | Distributed key‑value or document store (e.g. DynamoDB deployed across 3 AZs or self-hosted MongoDB). Tuned for fast lookups and optional strong consistency on critical paths. | Synchronous replication across AZs. Tunable consistency. Automatic failover.                                   | Optimised for cache miss fallback. Meets brief: p95 latency ≤ 250 ms on cache-miss path. Supports price lookups keyed by flight ID.                                              |
| **API Runtime Pods**                                    | Stateless containers. Autoscaled from a baseline of 2 to 8+ replicas per AZ during peak periods.                                                                                                                    | Kubernetes restarts failed pods; load balancer excludes unhealthy instances.                                   | Aggregates index and pricing for result set. Meets brief: p95 search latency 40 ms.                                                                         |
| **Ingestion Adapters**                                  | One per AZ minimum; autoscaled based on Kafka lag.                                                                                                                              | Autoscaler monitors Kafka lag and launches or restarts ingestion tasks as needed.                              | Handles 15 suppliers. Canonical JSON conversion. Target ingest lag < 30 s.                                                                                  |
| **Flink Checkpoints (State Backend)**                   | Key-value storage (RocksDB). ≈ 50 MB per checkpoint every 30 s.                                                                                                                 | Atomic write and restore. Restart resumes from latest checkpoint.                                              | Durable state for exactly-once semantics and operator recovery.                                                                                             |
| **Raw Events Lake (Audit Trail)**                       | Object storage (e.g. S3) for raw ingest via Kafka or Flink side output. ≈ 8 GB/day.                                                                                             | Regionally replicated. Disaster recovery via cross‑region sync.                                                | 1-year retention for audit, compliance, and replay.                                                                                                         |
| **Pilot‑light Disaster Recovery Region**                | Asynchronous log replication, global DB replica, read‑only Redis shard, index snapshot, idle Flink JM.                                                                          | Recovery Point Objective < 5 s; Recovery Time Objective ≈ 15 min with IaC‑based restart.                       | Meets spec risk assumption of £100 k/hr outage. Annual disaster recovery drills. Promoted if primary region down > 5 min.                                   |
| **Data Lifecycle and Governance**                       | Raw Kafka events streamed to object storage (e.g. S3). Lifecycle: Standard → Infrequent Access → Glacier. Athena queries over curated Parquet. | Regionally replicated. Raw events retained for replay. Curated tables support analytics and audits. | 1-year retention meets regulatory needs. Replay enables price anomaly investigation and recovery scenarios. |

### Infrastructure Best Practices

- **Immutable Infrastructure**: All stateful tiers are recreated via IaC (Terraform/Pulumi), further reducing recovery RTO through consistent, repeatable deployments.

- **Readiness / Liveness Probes**: Kubernetes probes enforce "fail fast" policies on API Runtime Pods and Ingestion Adapters, ensuring unhealthy instances are quickly removed from load balancer rotation.

## Risk Assessment and Mitigation

### High-Impact Risks
- **Multi-AZ Failure**: Primary region unavailable for >5 minutes
  - **Mitigation**: Pilot-light disaster recovery region with 15-minute RTO
  - **Business Impact**: Managed through automated failover and customer communication
  
- **Data Loss During Processing**: Stream processing failures causing financial discrepancies  
  - **Mitigation**: At-least-once processing with idempotent writes, 30-second checkpointing
  - **Business Impact**: Prevented through exactly-once semantics and audit trail preservation

- **Cache Poisoning**: Stale pricing data served to customers
  - **Mitigation**: 60-second TTL with LFU eviction, cache invalidation events
  - **Business Impact**: Revenue protection through real-time cache coherence

### Operational Excellence
- **Monitoring**: Comprehensive observability across all tiers with automated alerting
- **Testing**: Quarterly disaster recovery drills and chaos engineering practices  
- **Documentation**: Runbooks for common failure scenarios and escalation procedures
- **Skills Development**: Cross-training on critical system components and recovery procedures

## Success Metrics

### Availability Targets
- **System Uptime**: 99.9%+ (≤ 8.77 hours downtime annually)
- **Recovery Time Objective**: 15 minutes for complete region failure
- **Recovery Point Objective**: <5 seconds data loss maximum
- **Degraded Service**: Maintain 70% capacity during single AZ failure

### Performance Under Load
- **Search Latency**: P95 ≤ 40ms during 3,000+ QPS peak traffic
- **Data Freshness**: Supplier updates searchable within 1 second
- **Cache Hit Ratio**: 70%+ for pricing lookups during normal operations
- **Stream Lag**: Kafka processing lag maintained <5 seconds at P95

## Governance and Review

### Operational Reviews
- **Daily**: Infrastructure health checks and capacity planning
- **Weekly**: Performance metrics review and cost optimisation assessment  
- **Monthly**: Disaster recovery preparedness and runbook updates
- **Quarterly**: Full disaster recovery drill and architecture evolution planning. Update capacity model against latest peak traffic data.

### Decision Review Triggers
- **Availability SLA Breaches**: Systematic review of incident causes and architecture improvements
- **Cost Variance**: Infrastructure spend >20% above budget triggers optimisation review
- **Technology Evolution**: Major cloud provider feature releases enabling improved resilience
- **Business Growth**: Traffic patterns exceeding current capacity planning assumptions

---

**Document History:**
- v1.0 (3 August 2025): Initial high availability specification 
