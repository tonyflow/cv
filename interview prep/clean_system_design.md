# System

## Understand the problem and establish the design scope

### 1. Problem framing
Frame the problem:
* What are the boundaries of the system we are building and 
* What are its interfaces?
* Objectives and non-objectives

### 2. Functional requirements
* Which are the actors of the system?
* Which are the stakeholders? Do we actually need to talk about this?
* What are the features we want to implement?
* Let's describe 2-3 actor journeys based on these features
* Set priorities for the agreed upon functionalities


### 3. Non-functional requirements
These are the features that support the functional ones but are only implicitly surfaced to the users. They are supporting to the functional requirements
* Performance: Set percentiles for the endpoints
	* p95 of request of type X should be executed in less than 5 seconds.
* Durability: less than 1 object lost per 10^12 stored objects
* Availability
* Latency
	* e.g. low latency rate limit checks << 10ms
* Scalability
	* e.g. 1M rps
* Failure modes
	* e.g. return proper error codes 
	* e.g. proper error codes so that backpressure measures can be triggered accordingly.

The following are more esoteric and we could entertain these ideas only if the interview moves us towards them: 
* Recovery point objective: Maximum amount of recent data a system may los after a failure < 5 minutes
* Recovery Time Objective: The system can remain unavailable for X temporal units after a failure.
* Freshness e.g. new context visible within 10 seconds
* Reliability e.g. 
	* p99 of regional reliability
	* command success rate
	* degraded operation expectations and fallbacks
	* Recovery
* Correctness and invariants
	* ...
* Security and privacy
	* Authentication and authorization
* Safety (???)
* Cost and operability
* 

### 4. Scale and capacity
This is where we talk numbers
* Workload and growth assumptions
	* API reqs/sec
	* How does a request tranlate to database load?
	* X users per second/day/month year
* Architecture relevant calculations: This is where the back of the envelope calculations can come in handy e.g. all latencies an engineer should know
	* QPS for the database
	* Total HTTP load: reqs/sec
	* Total SQL storage
	* Total blob storage
	* Total NoSQL storage
* Capacity implications to our architecture

## Propose high-level design and get a buy in

### 1. Components
* Describe the high level components and their functionality, we will talk about their interfaces later on. You can now just talk about their main functionality

### 2. Contracts and state
* Decide on the data model
* Identify the access patterns (read heavy? write heavy?)
* State transitions e.g. a task transitions from suspended to running
* Describe the protocols (gRPC, HTTP, UDP, TCP other) and endpoints of the components you just drew
* What is the state passed around?

### 3. Complete design - draw the components
* Draw the components on Google Drawings

### 4. Journey walkthroughs
* Walk through the user journeys that we agreed upon in the functional requirements based onthe components you just drew

## Design deep dive

### 1. Highest risk analysis
* Identify high risk areas like
	* Scalability bottlenecks
	* Failure modes
	* Specialized compute or scheduling
	* Multi-region operation
	* Security risks
	* UDP, TCP?
	* Authentication

### 2. Targeted optimization
According to the areas identified in the previous step, we can end up talking about the following:
* partitioning ingestion by region/ vehicle, tenant or time
* autoscaling services based on number of requests and processing latency
* caching 
* batch processing
* data sharding
* pre-signed direct-to-object-store uploads
* database replicas and partitioning
* regional control plane with a global catalog
* GPU aware scheduling
* priority queues and quotas
* incremental processing
* rate limiting 
* regional load balancing
* database indexing
* data compression
* retention policies and cold storage
* security and least privilege principles
* data replication
* other?

## Wrap up

### 1. Operations and evolution
* Metrics, logs, traces and alerts
* Capacity and saturation signals Optional / Only if requested
* Deployments and rollbacks Optional / Only if requested
* Migrations Optional / Only if requested
	* Schema and data migration
	* Backfill
	* Dual-read and dual-write periods
	* Versioned contracts
	* cutover criteria e.g. backfill is 100% complete or error rate remains below 0.1%
	* validation and rollback 
	* removal of obsolete paths
* Rollouts - Optional / Only if requested
	* Canary and phased deployments
	* Feature flags
	* Backward compatibility
	* Rollback strategies
	* Success and abort metrics
* Trace and auditability
* Tests (unit, system/integration, e2e, modular)

### 2. Delivery plan and ownership - Optional / Only if requested
* Teams responsible
* Manpower allocated
* Milestones and dealine for each milestone

### 3. Epilogue
* Recap what has been said by re-stating what has been designed and if we have fulfilled and initial objectives (mainly the p0s)
* List 
	* critical path decisions made 
	* based on trade offs
* Remaing calculated or accepted risks

--

# From Meta Staff engineer
1. Requirements
	* Functional and non-functional
2. Core entities
	* Database entities or
3. API or interface
	* REST or gRPC
4. Data flow
5. High-level design deep dive
6. Deep dives
	* Should satisfy your non-functional requirements
	

## Candidate evaluation
1. Problem solving
2. Solution design
3. Technical excellence
4. Communication

## Fundamentals
1. Storage
	* various data storage models
	* sql
	* document stores
	* key value stores
	* acid properties
	* cap theorem of distributed systems
2. Scalability
	* vertical and horizontal scaling, 
	* partitioning/sharding, 
	* consistent hashing algorithm
3. Networking: OSI networking model. Most important from all these layers is
	3. Network layer: This is where the following happens
		* Load balancing
		* Firewalls
		* ACLs
	4. Transport layer: This is where the following happens
		* TCP
		* UDP
		* Request response lifecycle
	7. Application layer
		* http/https
		* rest/graphql/gRPC
		* RESTful semantinc
		* DNS resoliution
		* Websockets VS SSE
4. Latency, throughput and performance
	* RAM access: 100ns
	* SSD access 0.2ms
	* HDD access 2ms
	* Cross region 10ms
	* Cross region 50ms
5. Fault tolerance
	* Failure modes
	*  
6. CAP theorem
	* In modern distributed systems intermitten connectivity is a fact, your system should definitely be build in a way that partition tolerance is handle partitions. Then the main decision becomes whether your design should prioritize availability or consistency.

## Components
1. Databases
2. Cache
3. Message queues
4. Load balancer
5. Blob storage
6. CDN: just a cache for static content like images and videos

## Problems
What his interviews on Hello Interview