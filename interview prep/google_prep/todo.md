1. Read system design book 2-3
2. IO / CPU / Mem
3. Read sharding and 
	* Sharding can used ANY SHORT OD DATABASES: SQL,NoSQL, Redis, Cassandra
	* Good harding keys have:
		1. High cardinality
		2. even distribution
		3. aligns with queries??
	* Distribution can be performed based on
		1. ranges
		2. hash based: hash(key)% number of shards
			* Rebalancing is a problem 
		3. Directory based sharding
			* We keep a map of shark key to shard
	* Problems:
		* Hot spots: Avoid hot spots by using a composite key
		* Dedicated celebrity shard that uses special kind of hardware
		* Cross shard operations:
			* Cache results of cross-shard operation
			* Create materialized views of cross-shard operations or any precomputation as a matter of fact
			* Denormalization: There is some redundancy on the data but you have certain operation executing faster
			* Consistency: Check the Alice and Bob example where one gives money to someone else but this someone else is located on another shard
				* In this case we have a 2-phase-coordinator that handles this
				* Another solution is the SAGA pattern. This is a sequence of operations where each one was a compensating action that happens if this particular action fails.
4. Partinioning
5. Encoding and compression
6. Merkle trees 
7. Bloom filters 
8. MinSketch count
9. http and grpc protocols
10. CAP theorem - how to think around it
	* Is the fact that 2 users are not exposed to (do not get to see) the same state? Then we need to prioririze consistency. e.g. 2 users trying to book the same seat in an airplane
	* In any other case availabiltiy can be prioritized: e.g. a social media platform showing an outdated feed
	* If you choose to support consistency, there are different tools around it
		* implement distributed transactions
		* limit to single node - if it's a single instance we do not have such issues
		* discuss consensus protocols
		* accept higher latency
		* Example tools
			* Postgres
			* Tra RDBMS
			* Spanner
			* NoSQL with strong consistency (DynamoDB)
	* If we choose to support availability
		* use multiple replicas
		* CDC: Change data capture and send the changes as event to a data streaming platform like Kafka
		* Eventual consistency is fine
	* Consistency has different flavors
		* Strong consistency is what we're talking about mostly here
		* Causal consistency: we cannot have a comment replying to another comment appearing before the comment it replies to
		* Read-your-writes consistency: I should have an immediate view of what I have updated.
		* Eventual consistency
11. Types of indexes
	* btrees are only good for 1 dimensional data?
	* hash index
	* Geospatial indexes
		* geohashing
		* quad trees: geospatial index
		* r-trees
	* Inverted index: Elastic Search uses this - or Lucene. It is quite popular for document databases.
12. Batching transactions by using a 
13. Posgress supports 4 TPS
14. Redis supports 400 TPS
15. Geohashing - redis supports heohashinh
16. Rate limit algorithms
	* Token bucket
	* Leaking bucket
	* Fixed-window counter
	* Sliding window counter
17. Top-k implementation
	* Sliding window
	* Tumbling window
18. Distributed locks using Redis and a TTL
19. Consistent hashing
20. OLAP vs OLTP
21. TSPs estimates
	* Postgres: 1k - 10k
	* Postgres tuned: 10k - 50k
	* Redis (small inmem): 50k - 200k
	* Redis hosted: 200k - 1M
	* TODO how many HTTP served per second
22. Message queues
	* Acks
	* Delivery guarantees
		* at least once
		* at most once
		* exactly once
	* Use cases:
		* Async work
		* bursty traffic
		* Decoupling
		* Reliability 
	* Increase parallelism by
		* Introducing partitions that multiple consumers can consume from
	* In case of a lot of load, we can introduce backpressure: this can be as easy as having a rate limiter to the producer returning service unavailable responses to let them know that they should slow down
		* Alerting is also a countermeasure
	* Dead Letter Queues
	* Durability of qeueus
23. Cache
24. Idempotency
25. Concurrency and parallelism
26. 
27. Blob storage
28. DNS server and resolver
29. OSI framework
30. Proxy and reverse proxy
31. Networking: OSI networking model. Most important from all these layers is
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
32. Networking
	* Same region latency
	* Cross region latency
33. Latency
34. Vertical VS Horizontal scaling
35. Vertical partinioning
36. Normalization and denormalization
37. Quorums and leader election strategies
38. Content Delivery Networks
39. Websockets
40. Webhook
41. Rate limiting
42. API Gateway: Describes a role in the architecture - not a specific framework or product category
	* Authentication
	* Rate limiting
	* Logging
	* Monitoring
	* Request routing
	* TLS termination
43. Write scaling strategies
	* Batching
	* Aggregation  
		* e.g. bucketrization per hour
		* timescale's segmentBy
		* MapReduce operation e.g. Spark
		* Approximation COUNT MIN SKETCH
44. Checkpointing
	* Flink hourly windows
45. WAL - Write Ahead Logging
46. Separate data and metadata
47. Trade-offs and compromises
48. Race conditions
49. Concurrency and parallelism
50. Atomicity
	* This is not just a characteristic of SQL database. Redis can support this as well.
51. Useful HTTP codes
	* 429: Too many requests
52. Interesting headers
	* X-RateLimit-Limit
	* X-RateLimit-Remaining
	* X-RateLimit-Reset
53. Observable patterns
54. polyglor persistence: is the notion of keeping
55. How do we stream tokens back to the user so that we do not necessarilty wait for the inference service to produce an entire response to actually return
56. Difference in protocols when we're using wifi and wired connections over cellular data.
	* When using wifi and network cards that can support GBits of transfer per second then having streaming bidirectional communication is cheaprer
	* Especially when compared with cellular data when we need to PAY for the packages transfered upon every request and response.
57. Offline non-posix object stores that we can use
	* Rust FS
	* MinIO
58. Cache stampede: It happens when one hot cache entry expires and many concurrent requests simultaneously miss the cache and hit the backing DB/service to recompute the same value. The following are a few mitigations of this phenomenon:
	* Request coalescing: One request acquires lock and recomputes. This allows all subsequent requests to read from the updated hot value.
	* TTL jitter: Add different TTLs to different entries so that they all expire in different times and not at the same time so that at time X we can a concentrated run on the backend service.
	* Backend refreshes in the background. So clients might read stale data and 
	* Optimistic locking on the cache level: This allows a type of update on the cache but it does not protect the backend from stampede
59. Techniques for warming up the cache
60. Future/Promise and asynch request handling
61. Proxy and Reverse Proxy
62. Pagination for webserver reults 
63. Adaptive load shedding is a mechanism where a service dynamically rejects some incoming requests when it detects that it is approaching overload, rather than using a fixed traffic threshold.
64. Think about stateless and stateful services. Stateless services are very cheap to regenerate e.g. a pod that provides a web socket streaming tokens to a chat service 
65. Sampling techniques
	* Reservoir sampling: Maintain a uniform random sampling of a stream of unknown length
	* Stratified sampling: Divide data into groups/windows and sample within each
	* Systematic sampling: Keep every kth observation. Easy to miss spikes
	* min/max sampling: keep min + max within each time bucket.
	* m4 sampling: keep first, last, min, max for each bucket
	* anomaly aware sampling: explicitly retain anomalous points + sample the normal population. 



















