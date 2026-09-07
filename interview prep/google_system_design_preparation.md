## Preparation

Write a short numbered list:
* P0: essential use cases;
* P1: valuable extensions;
* out of scope;
* one representative end-to-end user journey.

## Questions
* I do not undertand how percentiles are important to the actual implementation. Give me an example of how an implementation can change when the requirement is 500 ms at p99 instead of 500 ms at p50


## 1. Functional requirements
* Who are the actors?
* What are the use cases?
* Deliverables: Write a short numbered list:
	* P0: essential use cases;
	* P1: valuable extensions;
	* out of scope;
	* one representative end-to-end user journey.
	
## 2. Non-functional requirements
1. Availability and durability
	* How is this measured
2. Latency
	* How is this measured
	* telemetry ingestion delay
	* time from model approval to fleet availability?
	* inference latency?
3. Safety and isolation TODO
4. Security and privacy TODO
5. Reproducability and auditability TODO

## 3. Scale and capacity estimates: Show me the numbers!
Do enough arithmetic to validate an architecture
* Number of vehicles, developers, models, workflows
* Messages or files produced per vehicle
* Average peak data ingestion rate
* Daily raw data volume
* Metadata volume
* Model artifact size and number of versions
* Concurrent workflows and average workflow duration
* Telemetry retention period
* read/wite traffic patterns
* Example: "With 100,000 connected vehicles, if 10% are active concurrently and each uploads an average of 50 KB/s of selected telemetry, average ingress is roughly 500 MB/s, or 43 TB/day before compression. Because incidents and regional reconnects create bursts, I would design for at least five times average throughput."
	* Break down: 0,1 * 100,000 = 10,000
	* 10,000 * 50 KB/s = 500,000 KB/s = 500 MBs/s
	* 60 seconds * 60 minutes * 24 hours = 86,400 seconds a day
	* 86,400 * 500 =43,200,000 MBs = 43,200 GBs = 43 TBs of data daily before compression
	* Design for 5 times this throughput
* Connect decision to
	* Streaming partition coint
	* object-store capacity
	* metadata database size
	* upload protocol and buffering TODO
	* offiline VS online processing
	* regional architecture
	* retention and tiering
* Explain how sampling, event-triggered collection, compression, local buffering, and retention policies affect the design.

## 4. Contracts and data model
* Define the contracts that clarify component responsibility
```
POST /v1/datasets
POST /v1/training-runs
GET /v1/runs/{run_id}
```
* And request bodies - e.g for asynchronous operations
```json
{
  "deploymentId": "dep-123",
  "status": "PENDING_VALIDATION"
}
```
* Important contracts should include
	1. idempotency key TODO
	2. caller identity
	3. target fleet TODO
	4. artifact version and digest
	5. compatibility constraints
	6. policy or approval reference
	7. request correlation identifier
	8. desired state TODO
* Core entities - database model and data classes from the side of the backend TODO: Is this actually database contract or something else?
	* dataset and immutable dataset version
	* training run
	* model and model version
	* evaluation report
	
* Storage choices
	* Object stores for immutable raw data, datasets, models, logs and large artifacts
	* relational storage for transactional metadata and lifecycle state
	* event log or message system for asyncrhonous coordination TODO: what is an event log give me an example
	* a search or analytics store

TODO: Explain the following in more detail
```
Discuss consistency at the boundary where it matters. For example, model approval and deployment creation may require transactional consistency.
```

## 5. First design

## 6. e2e flow

--
Between 6 and 7 there might be a longer optimization discussion. Optimization might include but not be limited to
* partition data per region
* partitioning data per region
* autoscaling data consumers from queue depth and processing latency
* caching artifact, manifests and frequently requested metadata
* batching small telemetry messages
* pre-signed uploads/downloads
* database read-replicas and partitioning
* regional control planes with a global catalog
* GPU-aware scheduling
* priority queues and quotas
* incremental processing rather than full data set recomputations
* other...

A good tradeoff phrasing could
> The bottleneck is X. I propose Y. It improves Z, but costs us A and introduces risk B.
--

## 7. Staff-Level Deep Dive options
* Safe fleet rollout
* Data and model lineage
* Multi-region operations and intermittent connectivity
* Safety and policy enforcement
* 

## 8. Reliability, Safety, and Operations

## 9. Delivery / Stakeholdrs / Evolution
> The main organizational dependency is the contract between the central deployment platform and the vehicle runtime. I would assign explicit owners to both sides and treat compatibility and rollback as versioned interfaces.

## 10. Deadlines and effort




## 11. Migration
In case we need to migrate from an existing solution to a new one

## 12. Validate & Conclude
Summarize:
1. Main user journey
2. Architectural boundary
3. The two most important design decisions
4. THe largest trade-off
5. The main unresolve risk
6. How the system evolves


## When to optomize




