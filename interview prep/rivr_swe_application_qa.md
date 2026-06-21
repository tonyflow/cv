# Questions

1. Can you provide an example of a project where you implemented or maintained a substantial part of the stack? Please specify the architecture that you used and any problems that you had to overcome, sharing any permissible details.
	* Revamp the data integration layer for the entire ethon platform. Created a worker paradigm where every ingestion type was equipped with multiple consumer wrappers and an upload manager to orchestrate them. The backend (assignment service) can assign sources to be ingested from each integration pod. This affected all data ingestion pipelines and their scale. The different implementations - of both consumers and managers - could accommodate traffic of message brokers, Kafka clusters, Object Stores etc. The scaling was based on Kubernetes. An additional data source to worker assignment service was used during the scaling processs. The scaling reasoning worked as follows:
	1. Backend receives new ingestion request for sourceId=S123, writes desired state to Postgres (enabled), and emits a control message keyed by sourceId (or records a pending command) for RabbitMQ delivery.  
	2. Assignment service reconcile loop checks ownership/capacity (owned_count < 10) across live workers.  
	3. If capacity exists, assignment service atomically sets source -> worker, ensures RabbitMQ binding (source.S123 -> q.worker-N), marks source assigned, and the worker starts consuming.  
	4. If capacity does not exist, assignment service marks source unassigned and updates scaling signal metrics (e.g., integration_sources_unassigned, total_required_sources).  
	5. Prometheus scrapes worker + assignment-service metrics; KEDA evaluates the configured query and computes desired replicas (for example ceil(total_required/10)).  
	6. KEDA updates DEployment replicas; Kubernetes provisions new worker pod(s).  
	7. New worker becomes Ready, registers/heartbeats (or is discovered via Deployment ordinal), and is considered live capacity.  
	8. The same assignment reconcile loop runs again, picks pending unassigned sources, writes 	source -> worker, applies binding(s), and ingestion begins on the new worker queue.  
	9. Workers continuously publish runtime metrics (integration_sources_owned, etc.), feeding the next scaling/rebalancing decisions.
	
	Clarifications:
	1.THIS IS INCORRECT! Workers are defined as a stateful set due to how easy it is to refer to them with ordinals (worker-0, worker-1). It's easier that way for the assignment service to know what is the cluster DNS that the scheduler provisioned and needs to be included in the new database state.
	2. There are two levels of orchestration here. (1) from the assignment servive (2) from the upload manager inside the runnng worker.
	3. Upon restart, bindings' state on Postgres is re-read and items re-assigned.


One more for how the autoscaling for upload channel deployments can scale. First of all the moving parts:
1. A data integration deployment can accomodate up to X upload channels 
2. A data integration deployment can accomodate upload channels using different credentials e.g. Can serve two channels from bucket A and and 8 channels from bucket B.
2. Every upload channel corresponds to one consumer wrapper
3. The communication between the backend and the data integration deployments is NOT done through Redis anymore but using RabbitMQ - we will talk about why this should be the case.
4. An assignment service is introduced that checks the state of active upload channels.
5. The assignment service acts as the orchestrator between the user asking for a new data ingestion and Kubernetes provisioning new data integration deployments.
6. To do so the assignment service has a very thin HTTP layer that exposes crucial Prometheus metrics about the state of the upload channels.
7. The cluster Prometheus metrics server can read these metrics and provision new deployments.

How do these actors interplay when a request for creating a new upload channel arrives at the backend:
1. Backend receives new ingestion request for sourceId=S123, writes desired state to Postgres (enabled), and emits a control message keyed by sourceId (routing key) for RabbitMQ delivery. The message is published to a RabbitMQ exchange and there's no binding for this routing key for this exchange to a queue so the message remains in the exchange unprocessed.
2. Assignment service reconcile loop checks ownership/capacity (owned_count < 10) across live workers.  
3. If capacity exists, assignment service atomically sets source -> worker, ensures RabbitMQ binding (source.S123 -> q.worker-N), marks source assigned, and the worker starts consuming.  
4. If capacity does not exist, assignment service marks source unassigned and updates scaling signal metrics (e.g., integration_sources_unassigned, total_required_sources).  
5. Prometheus scrapes worker + assignment-service metrics; KEDA evaluates the configured query and computes desired replicas (for example ceil(total_required/10)).  
6. KEDA updates DEployment replicas; Kubernetes provisions new worker pod(s).  
7. New worker becomes Ready, registers/heartbeats (or is discovered via Deployment ordinal), and is considered live capacity.  
8. The same assignment reconcile loop runs again, picks pending unassigned sources, writes 	source -> worker, applies binding(s), and ingestion begins on the new worker queue.  
9. Workers continuously publish runtime metrics (integration_sources_owned, etc.), feeding the next scaling/rebalancing decisions.

2. Can you discuss a time when you identified a major integration issue during testing? How did you diagnose and resolve it?
	* KEDA autoscaling for Celery workers was based on "ready" messages on an AMQP broker. This resulted in tasks being killed if all messages were consumed by workers (empty RMQ queues), the tasks were taking more than the KEDA cooldown period to finish, and the replica count for the specified worker type was set to zero. The fix was as easy as using the HTTP protocol to query for unacked messages. Hitting the nail on the head about what was causing the issues was more involved. 
	


# Disclaimer
1. I am not actively looking for a job I do have my eyes open about exceptional opportunities and I think RIVR is something like this.
2. I do not care about titles and that is why I applied to your senior roles although I consider having moved further in the engineering ladder, leading a team of 6.
3. Although I do not consider roles important I consider 
	* Responsibility
	* Impact and 
	* Remuneration
	important. So if I have:
	* The responsibility I want
	* The opportunity to have a large blast radius
	* and the base remuneration I want
	we can continue this conversation :)
4. I applied to your senior software engineer robot platform as well and that is position that really struck my attention. Even more than the MLOps role.
5. ...


* data ingestion
	* 
* finetuning
* inference - annotation -region
* evaluation


### Questions to answer before main interview process
1. MLJobs using kJobs.
2. Re-inforcement learning basics.
3. Why didn't we use Ray for fine-tuning?
4. time-slicing and multi-instance GPU implementations for training pipelines.
5. How do we make sure that changes on TF or jsonnet do not end up on both stg / prod and dev.

