# Architecture Trade-off Analysis

Before choosing a final architecture, two core options were evaluated:

* Hybrid Real-Time (Streaming): Uses continuous ingestion with Kafka and Flink to keep data fresh and indexed within seconds. Prioritises accuracy and availability at the cost of increased operational complexity.

* Serverless Batch + Cache: Uses scheduled jobs to process provider files and populate a cache periodically. Simpler to operate and cheaper at low volume, but introduces data staleness and cache consistency issues that conflict with the brief’s core goals.


| Criteria | Hybrid Real-Time (Streaming) | Serverless Batch + Cache |
|----------|------------------------------|---------------------------|
| **Latency (Data Freshness)** | **Very Low (Seconds).** Data is processed continuously, ensuring search results reflect current availability within seconds. This directly supports the goal of providing the "most accurate pricing and availability information". | **High (Minutes to Hours).** Latency is tied to the batch schedule (e.g., every 15 minutes). This delay means customers see outdated information, failing the "fresh" results requirement. |
| **Operational Effort** | **Higher.** Requires managing and monitoring stateful, distributed systems like Kafka and Flink. Even with managed services, it demands specialised skills to maintain a mission-critical pipeline. | **Lower.** The serverless model abstracts away infrastructure management. Teams focus on function code, and the cloud provider handles scaling and availability, simplifying operations. |
| **Cost** | **Higher Baseline Cost.** The core infrastructure is always-on, incurring costs even during idle periods. This model becomes highly cost-efficient at the high, sustained volumes the platform expects. | **Lower Baseline Cost.** A pay-per-use model means no cost for idle time. This is very efficient for low or unpredictable traffic but can become expensive at sustained high volumes. |
| **Risk** | **Implementation Risk.** The architecture is more complex to build and requires specialised engineering skills. Mitigation: The design is robust, proven, and the complexity is necessary to meet core business goals. | **Business Risk.** The inherent latency means the platform cannot deliver on its primary goal of accurate, real-time data. This is a critical failure that directly impacts revenue and customer trust. |