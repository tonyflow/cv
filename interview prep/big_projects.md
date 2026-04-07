# Tag manager data ingestion redesign
## Problem statement


# DSAR pipelines 
## Problem statement

## Cross-team coordination
- Product
- Legal
- Engineering


# MSA
## Problem statement
One of the responsibilities of the data consolidation team I was in was to provide a DSL for customer success engineers to build data sources. These represented different sources from the side of the customer which could produce attributes for the user profiles. This DSL though did not allow for the combination of these data sources.
- For example, we were producing probabilistic demographic attributes e.g. male 33%
- A different age data source might have better accuracy for the same user. Without the sources combination both ages were ingested/stored and then whoever created the audience should be aware of this difference in order to target users better.
- We need a new DSL through which the customer success engineers could define which data sources could be combined and in that way so that new attributes could be end up directly in the user profiles.

## Changes required
- New DSL library
- How do you aggregate these two data sources?
	- Crux: read both/all and alias the required attribute so that the aggregation framework 

# ID Management Optimization
## Problem statement
- The DMP ingested different data from different data sources. These data sources use different IDs to represent the same user.
- In order to connect these different data points we used a process of data syncing which was nothing more than ingesting tuples of IDs which belong together. 
- By ingesting for example tuple (a,b) we knew that all data arriving under ID a and ID b referred to the same user. 
- An important thing to mention here is that internally these IDs were not used. 
- Internally, the user was represented by a representative ID X. 
- This ID is exactly the same as the representative element in a union find data structure. The external IDs were only visible at the edges of the system: input/import and output/export. In order to keep track of these IDs and the user profiles we used two data bases:
	1. ID matcher: Database responsible for keeping track of ID components as a set of connected components in a huge graph like structure. It was indeed an implementation of the union/find or dynamic search algorithms and data structure. The lifecycle of IDs in this database was based on TTL and once one ID on a graph was "touched" then all IDs' TTLs on the component were updated.
	2. The profile cluster: This was were all the user profiles were stored. Alongside with all the user attributes, the IDs are needed, since there are certain directions when it comes to exporting profiles and these directions are based on IDs. The lifecycle of IDs was also based on TTLs on the level of the profile. This meant that expired IDs still existed if the profile they were assigned to existed. On top of that, all incoming IDs for a user were naively appended to the profile. This meant that new IDs were sitting alongside super outdated user IDs. This created an obvious misallignment between what the ID matching database and the profiling database were storing. 
- The ever increasing frequency of ID syncing was - in turn - increasing the number of IDs stored in these two databases. 
- So the goal of the project was to mitigate the ID load on these two databases
- Useless IDs were stored for an inditerminate amount of time.
- And this was in principle the crux of the puzzle. Strong identifiers were given the same lifecycle oportunities with less strong - non deterministic - identifiers.
- There was also the option of changing the customer requirements and ingesting less IDs by somehow assessing the gravity of an ID type but no customers wanted to go down that road. For them more IDs equals more data and more data equals better profiles.
- We were at the limit of throwing money to the problem: Paying 50k a month for both clusters


## Solution
### ID Matching
- The ID matching side was not particularly involved. The problem here was that the lifetime of all IDs in an ID component was perscribed by the age of the latest ID which was touched. For example in a component with 100 IDs where only 3 were strong and deterministic ones which received traffic, the other 97 were just tagging along and remaining useless just because we were ingesting data for other 3. The solution here was the introduction of retention classes per ID type. All IDs were assgined the base retention class and each ID was individually promoted to a new retention class based on the timestamp when it was touched.
- Nevertheless, the tricky thing was coming up with a sane retention class configuration:
	- How long should the base retention class be? 
	- How many retention classes should we have? 
	- What should be maximum we should allow an ID to remain into the system?
- In order to answer these questions we had to analyse the recurrence patterns of users per customer. For example if we receive at least two events for most users in a period of 3 days, then the base retention should be at least 3 days. So that we allow these recurring users to remain into the systems. Users who do not follow this pattern are labelled as less important and the corresponding IDs should expire after 3 days so that they can also alleviate the system from the load.
- Another important thing is that the data ingested using the expired IDs do NOT expire. We just expire the IDs themselves.
- Since this change was more contained, it was "easier" to implement but still tricky. Mostly as I mentioned because of the analysis that had to be performed in order to configure the system properly and not loose too many IDs.


### Profiling
- The profiling database was more involved to handle. The main complexity here is that:
	- it sits almost at the end of the data processing: just before the export of the profiles to the interested customers
	- this cluster consolidates the traffic of both the real-time and batch system 
- What is the criteria based on which the IDs aspect in the profile cluster get updated?
- This was the solution that was followed in brief: 
	- The profiles cluster consolidated a multitude of different data sources which could potentially carry ID information. 
	- These data sources are either S3 buckets or Kafka topics. 
		- The S3 buckets were created from pipelines which were - at some point in their execution - querying the ID matcher service
		- The Kafka tropics were populated by real-time services which were - at some point in their execution - querying the ID matcher service
	- All the clients of the ID matcher service should populate the aspects sent downstream with ID timestamps
	- In this way the profile cluster knows, when was this snapshot of IDs taken!
	- Thus the solution is to equip all aspects consumed by the profile service with timestamped IDs.
	- In that way, the service can upsert the latest ID aspect and discard any deprecated one
- Since the profile consolidation involves multiple sources, multiple pipelines and real-time services had to be adjusted in a way that allows the correct ID aspect resolution from the side of the service.
- New data model had to be introduced
- New schemas had to be introduced carrying these timestamps

#### Synchronization issues
- The cluster was processing events in real-time in a large distributed system and there were definitely synchronization issues. Checking the timestamps though - once all clients populated them - should resolve these issues.
- Nevertheless, in the first few days there might be events carrying timestamps and events not carrying timestmaps thus fallback mechanisms had to be introduced in the profile cluster: 
	- Timestamped events were given precedence
	- In case of a mistake, eventual consistency should fix the issue
- Refactoring old code along the way


## Testing and Rolling out
### ID Matching
- Initially retention classes were configured in a way that we would avoid aggressive IDs expiration. We had to configure the retention in a way that was in par with the existing implementation
- Afterwards
	- Base classes should have remained the same
	- Higher retention classes should increase in time


## Results
### ID Matching
- This resulted in an average of ~15% in average. Customers with first party identifiers were heavily impacted but these customers were 1-2 that's why the overall decrease remained small.

### Profiling
- Up to 20% decrease in size. The amazing fact here is that these IDs were used to send audience diffs to ad servers and because users which were more active that other were significantly affected, these diffs we decreased up to 40%. This is a 40% decrease in netword traffic which result in a much more timely audience delivery.

