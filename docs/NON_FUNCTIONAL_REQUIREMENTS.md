# Non-Functional Requirements

## Document Metadata
- **Document Type**: Non-Functional Requirements Specification
- **Version**: 1.0
- **Author**: T. Vergilio
- **Context**: OTB Take-Home Exercise
- **Review Date**: 4 August 2025

## Executive Summary

This document defines the non-functional requirements for the flight search platform, establishing measurable quality attributes that support business objectives. These requirements ensure the platform delivers competitive performance, maintains operational resilience, and scales efficiently with business growth.

## Business Context

The non-functional requirements directly support OTB's strategic business objectives:

- **Revenue Protection**: High availability and data accuracy prevent booking failures and maintain customer trust
- **Competitive Advantage**: Sub-40ms response times and real-time data freshness differentiate against slower competitors
- **Operational Efficiency**: Cost-effective scaling and maintainability support sustainable business growth
- **Regulatory Compliance**: Security and audit capabilities meet ATOL and data protection requirements

## Requirements Traceability

All requirements trace back to business drivers and technical constraints identified in the design brief. Each requirement includes validation methods to ensure measurable achievement of quality targets. Infrastructure specifications and detailed implementation approaches are documented in the [High Availability Plan](HIGH_AVAILABILITY_PLAN.md).

## Reliability and Performance

<table style="border-collapse: collapse; width: 100%;">
<thead style="background-color: #27ae60; color: white;">
<tr>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Area</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Target / Constraint</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Source</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Implementation Reference</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Validation Method(s)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Availability</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">≥ 99.9% uptime. Platform must survive AZ loss or pod crashes</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: Technical Architecture</a> table and <a href="HIGH_AVAILABILITY_PLAN.md#risk-assessment-and-mitigation">Risk Assessment</a> section</td>
<td style="padding: 8px; border: 1px solid #ddd;">Chaos testing per <a href="HIGH_AVAILABILITY_PLAN.md#operational-excellence">HA Plan validation procedures</a></td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Durability</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">No data loss on AZ or node failure</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: Technical Architecture</a> - Kafka, Flink, and NoSQL replication strategies</td>
<td style="padding: 8px; border: 1px solid #ddd;">Simulated AZ loss and data consistency validation per <a href="HIGH_AVAILABILITY_PLAN.md#operational-excellence">HA Plan</a></td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Latency (p95)</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">≤ 40 ms end-to-end API search response</td>
<td style="padding: 8px; border: 1px solid #ddd;">Design Target</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: Technical Architecture</a> - API Runtime Pods and Redis cache configuration</td>
<td style="padding: 8px; border: 1px solid #ddd;">Load tests, profiling, RED metrics (Rate/Errors/Duration)</td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Read Throughput (Search)</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">API must handle ~3,000 QPS at peak traffic</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: API Runtime Pods</a></td>
<td style="padding: 8px; border: 1px solid #ddd;">Load tests simulating peak traffic patterns per <a href="HIGH_AVAILABILITY_PLAN.md#performance-under-load">HA Plan metrics</a></td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Ingest Throughput (Data)</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">Pipeline must process high-volume streams from 15+ providers without backpressure</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: Technical Architecture</a> - Kafka partitioning and Flink TaskManager scaling</td>
<td style="padding: 8px; border: 1px solid #ddd;">Soak tests per <a href="HIGH_AVAILABILITY_PLAN.md#performance-under-load">HA Plan</a>; monitor Kafka lag and Flink backpressure</td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Disaster Recovery</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">RPO < 5 s, RTO < 15 min for regional failure</td>
<td style="padding: 8px; border: 1px solid #ddd;">Design Target</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: Technical Architecture</a> - Pilot-light DR region implementation</td>
<td style="padding: 8px; border: 1px solid #ddd;">Fire-drill simulation per <a href="HIGH_AVAILABILITY_PLAN.md#operational-reviews">HA Plan quarterly testing schedule</a></td>
</tr>
</tbody>
</table>

---

## Data Handling and Observability

<table style="border-collapse: collapse; width: 100%;">
<thead style="background-color: #3498db; color: white;">
<tr>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Area</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Target / Constraint</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Source</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Implementation Reference</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Validation Method(s)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Data Quality</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">Must detect and reconcile pricing anomalies and conflicting flight information</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">Flink deduplication with source ranking and outlier detection algorithms</td>
<td style="padding: 8px; border: 1px solid #ddd;">Simulated conflict injection, anomaly detection metrics, alert thresholds</td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Reporting and Analytics</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">Support for generating business reports on route coverage, availability, and data freshness</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: Data Lifecycle and Governance</a> with curated Parquet tables</td>
<td style="padding: 8px; border: 1px solid #ddd;">Generate sample reports and validate data against source systems</td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Index Freshness</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">≤ 1 s delay between ingestion and searchability</td>
<td style="padding: 8px; border: 1px solid #ddd;">Design Target</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: Search Index  configuration</a></td>
<td style="padding: 8px; border: 1px solid #ddd;">Track p95 ingestion-to-query delta</td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Retention</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">Raw events must be archived for ≥ 1 year</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: Raw Events Lake</a> with S3 lifecycle policies</td>
<td style="padding: 8px; border: 1px solid #ddd;">Confirm S3 lifecycle config per <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan specifications</a></td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Replayability</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">Minimum 3-day replay window for pipeline recovery</td>
<td style="padding: 8px; border: 1px solid #ddd;">Design Target</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: Technical Architecture</a> - Kafka topic retention and Flink savepoint strategy</td>
<td style="padding: 8px; border: 1px solid #ddd;">Run manual replay dry-run per <a href="HIGH_AVAILABILITY_PLAN.md#operational-reviews">HA Plan procedures</a></td>
</tr>
</tbody>
</table>

---

## Operational Qualities

<table style="border-collapse: collapse; width: 100%;">
<thead style="background-color: #f39c12; color: white;">
<tr>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Area</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Target / Constraint</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Source</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Implementation Reference</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Validation Method(s)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Security</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">Must comply with fare display rules and data protection expectations</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">TLS in transit, IAM-scoped access, audit logs, encryption at rest</td>
<td style="padding: 8px; border: 1px solid #ddd;">Config reviews, automated scans, compliance scanning</td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Maintainability</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">System must support onboarding new supplier feeds quickly</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: Ingestion Adapters</a></td>
<td style="padding: 8px; border: 1px solid #ddd;">Time-to-integrate new feed; error rate during first sync</td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Cost Efficiency</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">Minimise cloud spend through dynamic scaling and optimal pricing models</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">See <a href="HIGH_AVAILABILITY_PLAN.md#technical-architecture">HA Plan: Technical Architecture</a> across all compute tiers</td>
<td style="padding: 8px; border: 1px solid #ddd;">Track cost-per-search metrics; automated budget alerts</td>
</tr>
</tbody>
</table>

---

## Business Features

<table style="border-collapse: collapse; width: 100%;">
<thead style="background-color: #9b59b6; color: white;">
<tr>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Area</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Target / Constraint</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Source</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Implementation Reference</strong></th>
<th style="padding: 8px; text-align: left; border: 1px solid #ddd;"><strong>Validation Method(s)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Search Flexibility and Segmentation¹</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">Must support tailored results by brand, locale, and customer segment</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">Search-time filtering and boosting using OpenSearch field tags</td>
<td style="padding: 8px; border: 1px solid #ddd;">Integration tests with personalised search scenarios</td>
</tr>
<tr>
<td style="padding: 8px; border: 1px solid #ddd;"><strong>Dynamic Configuration Support</strong></td>
<td style="padding: 8px; border: 1px solid #ddd;">Must support pricing variants by brand/segment with minimal code change</td>
<td style="padding: 8px; border: 1px solid #ddd;">Brief</td>
<td style="padding: 8px; border: 1px solid #ddd;">Config-driven fare modifiers with dynamic targeting capabilities</td>
<td style="padding: 8px; border: 1px solid #ddd;">Variant tracking dashboards, feature flag logs, rollout monitoring</td>
</tr>
</tbody>
</table>

¹ For future hyper-personalisation, the platform could integrate OpenSearch with OpenAI using Retrieval-Augmented Generation (RAG), enabling dynamic content generation based on user context and preferences. This would support advanced features like personalised fare suggestions, loyalty programme prompts, and tailored travel packages. While not required in the current scope, this platform allows evolution toward hyper-personalisation using RAG.

---

## Requirements Governance

### Review and Validation
- **Performance Testing**: Load testing campaigns validate throughput and latency requirements under peak conditions
- **Chaos Engineering**: Regular failure injection tests verify availability and recovery requirements
- **Business Metrics**: Continuous monitoring ensures requirements align with evolving business objectives

### Change Management
- **Impact Assessment**: Changes to non-functional requirements trigger architecture impact analysis
- **Stakeholder Approval**: Performance target modifications require business stakeholder sign-off
- **Version Control**: All requirement changes tracked with rationale and approval authority

### Success Criteria
- **Technical Compliance**: All measurable targets achieved within specified tolerances
- **Business Value**: Requirements demonstrably support revenue protection and competitive positioning
- **Operational Readiness**: Validation methods successfully integrated into operational processes

---

**Document History:**
- v1.0 (3 August 2025): Initial NFR specification 