# Core thesis
Gemini is a 
* Deterministic command coordinator
* Vehicle-side policy enforcement point
* Converts a model proposal into a short-lived state-bound idempotent command

# Framework
1. Frame and scope: Boundary, ecosystem, assumptions and exclusions
	* Boundary: What are we building and what do we interface with
		* Not implementing models training and inference implementation from the side of Google cloud
		* Only in vehicle runtime
		* Not implementing climate system API - just consuming it
	* Assumptions and exclusions
		* Vehicles in multiple regions
		* manufacturer exposes high-level capabilities
		* Intermitten connectivity
		* Authentication/authorization comes from google identity federation
2. Functional requirements
	* Actors and stakeholders
		* drivers
		* vehicle platform
		* cloud API provider
		* automaker tool intergation
	* Journeys
		* Set my side to 21 degrees
		* Charger discovery and navigation
	* Set priorities
		* p0
			* Session and context: streaming voice/text, interruption/cancel, locale, authentication, current route and sage state snapshot
			* capability discovery: vehicle specific tool catalog, 
			* deterministic authorization: policy decision, state precnditions, condirmations, safety/distraction gating.
			* reliable execution lifecycle
			* user feedback and audit
		* p1
			* roadside and charging transactions
			* multi-step goals
			* develop ecocystem
2. Non-functional
	* Performance
		* Voice responsivenes: stream speach and model output, ackowledge long provider workflows early
			* p95 < 1.2 seconds
			* p99 < 2.5 seconds
		* Climate decision: Preserve a local path without a cloud roundtrip
			* p95 < 300 milliseconds
			* p99 < 700 seconds
		* Connected vehicle mutation: something actually happens on the vehicle and the actor realized this
			* p95 < 3 seconds
			* p99 < 8 seconds
		* Cloud navigation
			* p95 < 3 first useful result below 3 seconds
	* Reliability
		* Regional session reliability: p99. A cloud outage still leaves a bounded vehicle in local mode. Cloud becomes unavailable but the local system can still serve an approved request for setting the cabin temperature.
		* Command success rate: Authorized Successful Commands / All Commands Ration > X
		* Degraded operation: If connectivity issues are present, notify and not queue the request to be executed async.
		* Recovery: Unknown outcomes remain visible and are reconciled e.g. the vehicle applies the temperature changes but the success response is lost. we can reconcile by re-consuming the temperature changed API.
	* Correctness or invariants
		* A model output does not reach an actuator or a consequential cloud service without authorization
		* The system checks current state before a physical action: does not open an open window
		* A driver confirmation can approve only the exact command shown not a previous one
		* An older delayed command cannot overwrite newer driver intent
		* Losing cloud connectivity cannot increase permission. For example, offline mode cannot start a purchase that requires a cloud identify check.
	* Security and privacy
		* Vehicle proves identity using hardware based keys - serial numbers
		* Short-lived credentials backed by oauth spec 
		* Trusted driver ID
		* Encrypted service connections
		* Least privilege tool scopes
		* Encrypted storage
		* Separate personal info from operational identifiers: we are not storing driver names - but owner IDs
	* Safety
		* Expose only high level capabilities as tools and assign risk level to each of them
		* Again the vehicle checks current state imeediately before physical action
	* Cost and operability
		* Handle simple low-risk requests in the vehicle when possible
		* Limit the number of model tokens and external function calls HOW?
		* Track cost and errors per vehicle, model version

The non-functional requirements lead to 2 main main deign decisions:
* The vehicle makes the final decision
* A small approved functions remains available when the network is not available
			
3. Scale and capacity	
	"I will now estimate traffic and storage only when the result changes the design. e.g. 500k state events /second require a separate event path instead of a command database."
	* Workload and growth assumptions => This refers mostly to the API level
		* Fleet: 20M, 4M daily active
		* Conversation 80M turnes/day;25k peak turns/s during commuting/ event bursts
		* Commands that change state: 5k peak tool commands / s, less than 10/vehicle
		* vehicle state: 100k - 500k filtered events
		* payload: command messages < 16kB
	* Architecture-relevant calculations
		* model requests running at the same time: 80m/day is about 930/second. Peak is 25k/second, about 27 times the average.
			* 2 seconds / request => 50k requests are running at once on peak: This is because 25k are getting completed and 25k are arriving
			* 30% contigency gives a normal global target of 50k + 0.3 * 50k = 50k + 15k = 65k
		* Commands per day
			* 8M /day
			* 1 command + database indexes use 4 kbs
			* 32GBs/day
			* 2.9 TBs for 90 days before copies and audit archives
		* Vehicle state data arriving int he cloud
			* 500k events/sec
			* 300 bytes per event
			* 150 megabytes  / sec
			* keeping that peak rate for a full data this is 12TBs
			* A separat event system stores this
		* The system receives 5k commands /sec globally
			* <10 per vehicle
			* Store commands with a vehicle identifier as a key

4. Contracts and first state
	* API 
	* state machine
	* complete hybrid architecture
4. Complete design
	* Optimization
5. High-risk deep dive
5. Reliability and operations
6. Evolution and ownership
6. Close

framing => functional => non-functional => contracts and state => complete design - high risk analysis => reliability and operations => evolution and ownership


Coarser granularity from bytebytego
* Understand the problem and establish design scope
	* problem framing
	* functional
	* non-functional
* Propose high-level design and get a buy in
	* contracts and state
	* complete design
	* journey walkthroughs
	* pressure-point identification and potential optimizations
* Design deep dive
	* highest-risk analysis and targeted optimization of that risk
* Wrap up 
	* Reliability and operations
	* evolution and ownership


TODO: Where does the CDN sit and how is it used during an HTTP request lifecycle?

1452