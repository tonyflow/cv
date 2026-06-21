# Data integration workers
* The data integration pipelines of the company was a collection of customizations
* For every data integration an engineer
	* Needed to write a client e.g. mqtt, kafka, cloud storage, you name it
	* Create a new entry in the containers that 
	* For every new source, we had to change the code and redeploy - with downtime
	* Only FDEs and people who wrote code could add a new integration
	* One customer could only have only one type of data ingestion e.g. Kafka
* I wanted to have a framework that 
	* Could be resused for all different types of live data
	* Different integration types could work simultaneously
	* Could scale with our needs to use k8s
* I entertained different scheduling frameworks for this but most of them had conventions that were going against our constraints. For example, we wanted to be able to pause and resume live data sources at will.
* And this is where the upload channels come into play:
	* The upload channels is an orchestration framework that provide customizable live data integation
	* The entrypoint is a upload channel manager
	* The upload channel manager received different commands through Redis/RabbitMQ e.g. start consuming, stop consuming, delete channel, replay messages etc
	* For every data source it spawns one of many consumers. 
	* It can be many per channel since we might need to consume different data types and apply different transformations

* And what about the infra? Where does this live? How does it scale?
	0. Each pod exposes a prometheus metric
	1. Prometheis scapes the metrics
	2. KEDA reads metrics
	1. KEDA scales integration-workers
	2. Each worker starts and registers workerId + heartbeat in Redis.
	3. Assignment service sees a new healthy worker, creates/updates bindings.
	4. Worker consumes its assigned queue.

* The last but maybe the most important part was the UI - since this is where the feature stops having an engineer as its main user and clients can also configure their integrations
* Many more features we added one top
	* Error stats
	* Replays of failed messages
	* Start consumption from specific timestamps
	
* One the great features of this is that we can spawn multiple data integration containers per data type ingestion
	* 5 containers for MQTT data
	* 10 containers for Kafka data
	* 100 containers for different cloud storage integrations
	* The backend and the manager running on the data integration container. Every new container creates a new 

# ETL Pipeline
* The existing implementation for parsers and loaders was very involved hierarchy of pixins that an FDE would need to appropriately compose to achieve a goal

* The structure and hierarchy was so involved that FDEs were giving up on finding the right classes and re-implemented interfaces and different functionalities that already existed.

* The pipelines had evolved organically into this

* On top of this - since the pipelines we mostly written by data scientists - many data exploratory technologies were used which are not very common in OLAP schemes. For example, the semantic enriching before storing in the database was performed using Pandas.

* The ETL is very simple conceptually. It is composes of 3 parts:
	* A raw data provider: This is the authority for reading raw data. There is a raw data provider that knows how to read Excel data, another that knows how to reade JSON and so on. There are very few raw data providers. You can also call them readers.
	* A transformer: This is where the business logic reside
	* And a loader:
	
* The most difficult part of this project is not coming up with the interfaces to fit the ETL functionality. 
* These interfaces are ubiquotous and most data pipelines abide by them.
* The most difficult part was:
	* Coordinating the effort of multiple people: Some
	* And the most difficult was building an adapter so that the old system could still work with the new interfaces.
	* The old system was still in use from multiple customers.
	* One cannot just wipe it out the old system and start using the new one

* And once the adapter for the old system was in place we could
	* Expose the new parsers to the UI both for live data and manual uplaods
	* 

# Kubernetes migration
* As mentioned in my CV we were working on single node manchines. This is how the company came to be


# Distributed system for vision pipelines
* DVC for data set tracking - not relevant
* Sentinel nodes
* Courier in cases where the inspections were needed on the cloud
* Network file system like NFS or
* HTTP server whose sole purpose was artifact distribution
	* YAMLs/JSONs were used to 
* RabbitMQ and MQTT were used for inspection details tranfer
* For cloud deployments we used AWS Lustre
	

# 40% reduction
* Start using webp as an original image format
* Stopped storing thumbnails
	* Scaling the image on demand for UI requests made much more sense for the purposes we were using the images
	* Of course during training the 

# Fine-tuning pipeline
* First of all I want to describe a bit the situation
* We had a vision product that was not part of the main platform
* The vision product provided defect detection and classification and was being re-designed from the ground up
* In parallel, our main platform was migrated to k8s
* There was a dilemma here: do we push both (1) the integration of the vision product inside the main platform and (2) the py-torch pipelines training pipelines running inside k8s or (3) go with a more conservative compute approach
* Many ideas were entertained:
	* All in one GPU machine with a subprocess as the training pipeline: The entire platform is deployed there and we spawn subprocess when we want to train
	* All-in-one GPU Machine with a Celery task as the training pipeline: The entire platform is deployed there and we spawn Celery tasks when we want to train
	* All-in-one GPU Machine with a separate container
	* Separate GPU machines per customer
	* AWS Batch with Event bridge using EC2 instances with ECS as the capacity provider
		* Separate queues and prioritization for our tasks
		* Apart from GPU tasks we could reuse it for any kind of compute intensive jobs
		* 





# Human and management skills
* Before becoming an engineering success manager I could see that the most potent leadership principle was leading by example
* Even when I was under leader that I disliked I could see that my entire mindset shifted when they did something that I could use in my work
* I could look up to them when the provided engineering value
* That is why I was the also follow the IC path and not the people managers path: I could see that exceptional engineers get pushed and inspired by exceptional engineers.
* Having said that I always try take a smaller or bigger part in the individual projects of the team.
* I allocate projects based on skills:
	* Engineer A is amazing at tooling: He introduced nix and aleviated the need of engineers to always keep track with dependedabot versions that they need to install on every depedency change.
	* Engineer A is very passionate about CI/CD. He did amazing work with
	* Engineer A craves for interfacing with the customer 
	* Engineer A is an algorithmic wiz: He turned the efficiency of our LightGBM algorithm upside down. We are creating a solution space of causal graph. And even one of them can be GBs of data. He replaced the adjacency list implementation with bitmaps and drops the storage footprint by 300%. It was amazing
	* Engineer A was very passionate about distributed system. I could think of a better fit than k8s for them.
	
* Based on this I feel like I have created a team that is productive
* What is missing though is more cohesiveness: I feel like we can do much better in having a common direction for the team:
	* We had a direction for some time when many people were working on the k8s migration but when this was delivered most people had the tendency to isolate
	* I don't know how this will evolve but I would like to see more cohesiveness
	
# Questions
1. I have read your blog posts about Titan and the Model Factory: How close is the everyday work of the data platform team to these components?
2. How close is the data platform team collaborating with the researchers?
	* Close collboration
	* Embed people in the team for a while
	* Researh engineering is the group
3. Can you talk to me a bit about the technologies that you're using?
4. How is the data platform team currently organized?
	* 5 people
	* Internal initiatives from the team
	* Platform liability
	* And research requirements
	* Model of particular size: Log-term project
5. What is your take of the responsibilities of this role? I have read the opening but it would interesting to get your take on things.	
	* 
6. nvidia
	* 10k GPUs

















# 
* Research engineer
* Agents
	* research from platform to product
	



* key value storage system
* database that runs on a single machine for now
* APIs:
	* store(k,v)
	* get(k)
	* read(k1,k2) all the values between the two keys
* Keys are either byte arrays or strings
	* User chooe the byte array
	* Pick one and stick
* Constraints on the size of keys of values
	* Maximum of 1MBs for both keys and values(??)
	* p50 4KBs
* Users access the data
	* Multiple concurrent clients
	* The write more than they read
	* More write replicas
* Full data size might be large
	* Billions of such objects
	* RAM cannot accomodate all these values 
		* (LRU,MRU) + Disk
* Durability
	* Backups
	* Retain as much data as possible




class KeyValueStorage




