# Role Deep Dive

Since this is more about role specific skills, we can focus on the MLOps lifecycle and projects from the past 2 years that reflect work related to the different parts of the MLOps lifecycle:

The MLOps lifecycle is:
1. Data preparation / curation
2. Model development 
	* Data Driven Testing I was involved
 	* Model development I was not really involved since it was performed mostly from the researchers using a SLURM local machine for experiments.
3. Model training
	* Data selection / upload / annotation
4. Review and evaluation
	* Separate pipeline for this
5. Inference is performed in multiple ways during this lifecycle
6. Monitoring
	* Kubernetes monitoring
	* Triton monitoring 
	* MLFlow / Aim monitoring


--- We want to docus on recent projects challenges and problem solving

------------------------

## Data preparation / curation
* Initially the data integration was customer-specific, we would need to write new code for every new customer whether or not we already had for example a cloud storage integration or some kind of data broker integration. There were a few ways of integrating data but they were
	* dispersed, 
	* fragmented and 
	* they were not-following any specific patterns. 
* On top of that, when you're curating data for ML models, usually FDEs or research engineers have to apply certain business logic before these data can be stored and used for training. 
* And finally once you're done with this and you want to accomodate 100 customers, then you need your features and pipelines to scale

1. Data integration
	* Upload manager abstraction
	* Inside the upload manager there were multiple consumer wrapper abstractions
	* Each manager could handle multiple consumer wrappers
	* Each manager was a k8s deployment
	* Each manager could handle one type of ingestion
	* So an s3 upload manager living on a k8s deployment was spawning multiple consumer wrappers that could consume from different s3 buckets and ingesting different data types. Data type is not necessarily a primitive but a internal platform concept.
	* The end result of this step is a "bronze" dataset of raw data. There's no post-processing applied at the time of the integration. 
2. Curation and business logic application using an ETL pipeline
	* Implementation of an ETL pipeline were research engineers can focus mainly on the T part
	* The extract part was mostly repeated: images, csv files, json files, excel files. Theere are more or less standard ways of reading and extract this data so it was rare that an engineer would need to implement an extractor.
	* The L part of the ETL which represents the loaders was also quite standardized. The loaders are receivers of canonicalized data and they are also rarely touched. If a change needs to be done there, this means that the canonicalized schema itself has changed and it also means that the database needs to change.
	* Thus we're left with a small T step which is the transformation step and we need to regularly deploy these Ts in order apply business logic to the incoming data points.
3. Scale
	* Split a worker container per ingestion category (AMQP brokers, Kafka clusters, Cloud Storage). The integration flow looked like the followin

### Challenges during this phase
* Create a custom worker paradigm instead of using an existing one like Celery
	* The reason was mainly that Celery tasks are largely opaque after beeing scheduled.
	* Custom error handling and retries became hard to reason around
	* We wanted to start and stop these data stream and Celery once a worker item is spawned it cannot be retrieved and adjust its state, we wanted a manager that can have access to these workers threads and their state and can start and stop them at will.
* Secrets management when creating new access points 
* Arbitrary code execution for even faster transformation logic deployment.
* Redis for work item submission, this was a mistake for many reasons
	* No priorities
	* No state, on restarts state was lost
	* In the end we replaced it with RabbitMQ
* TRADEOFF and things that we left behind: There was a data mapping feature that we decided not to port because of the functionality provided by the new paradigm: The speed in which the data were ingested allowed for fast iteration thus an old piece of code that allowed for correct ingestion from day one by prevewing or inspecting the data was not relevant anymore.

------------------------
MLFlow
* Team
	* Hardware
		*  
	* and software
		* 20%
		* 35 swes
		* 5 teams
			* system one: reinforcement learnng
				* AWS Batch
				* Simuluation data (?)
				* Locally executing script to run multiple experiments 
			* system two: learned path VLA 6 team
				* Learning from data sets
				* 10k deliveries double digit TBs
					* Simuluation data (?)
					* Robot connected to cloud
						* Functions without cloud
						* it is not airgapped
						* telemetry
						* map
						* data offload TB per robot per data
							* Not RT
					* Flight (instead of Ray)
					* There's a k8s operator
					* RGB data - rendering 
			* Platform: software on the robot SLAM
				* Multi- distributed GPUs
				* Automation are in progress
			* Cloud team 
			* SLAM team 
			* DevOps person 
* Tech stack
	* Kubernetes setup
* 


------------------------

## Model fine tuning
The main objective was to build a fine-tuning system available from either the main platform or our internal tools with certain requirements:
* Multi-job queueing
* Read annotated training data from an object store
* Read base models from a file system - initial object stores, then FXLustre
* The PyTorch code is containerized and the pipeline loads it on trigger
* Run the jobs
* Notofy on completion
* The foundational models are not trained on the GPU environment and this only happens on our local cluster.
* Initially there's no multi-node/multi-process training. We are using a single-node that can run multiple fine tuning jobs on one GPU. This is a multi-tenant scalable compute environment. But we are not doing any distributed training for now.
* Leverage GPU compute

### Challenges and tradeoffs during this phase
* The biggest question was wether we can go straight to Kubernetes with this. We were not fully on k8s yet and we wanted to build a system that is in-line with our existing job sumbission framework but can me moved to Kubernetes when our infrastructure was mature enough.

* Research engineers were arleady using SLURM on our in-house GPU cluster.

* We entertained different solutions from k8s native jobs, Kubeflow, Ray, Sagemaker and completely custom ones.

* Decision to stay away from Sagemaker and get trapped in its ecosystem for trainings, 
* It was a hard decision to stay away from Ray and Kubeflow. They both provide
	* complex and dynamic dependency graphs
	* mixed GPU/CPU training
	* Distributed Python code execution
* Kubeflow also has a nice PyTorch operator for native distributed training.

* We ended up selecting a native AWS solution and used AWS Batch. Batch is composed of 3 main parts: job definitions, queues and compute environments. 

* AWS Batch was a cloud-native version of Slurm which aligned with how the researchers were used to interface with training environments.

* AWS Batch did not just remove the complexity of provisioning compute ourselves. 

* With AWS Batch we were able to change from EC2 compute to EKS compute with one Terraform line change and point to our EKS clusters when they were ready :)

* There was no requirement for warm compute luckily. We only required warm pools duringinference and we would tackle this using Kubernetes.

* Multiple jobs can be enqueued and prioritized which is a great free-feature of AWS Batch.

* Fine-tuning is not time-critical thus we could bring spot instances in the mix

* Versioning problem: Platform version could be bound to job definitions and job definition were bound with training image version.
	* This also had a drawback because we had to commit new Terraform code for every job defition.

* The code changes needed were minimal

* Costs. No additional charged to actually using AWS Batch and its queues. We are just paying for the compute.


* Collocation of GPU jobs under the same environment was another concern but this became of matter of configuration using Batch.

* Reading based models from object stores was slow and had to move to Amazon Lustre
	* Shared model cache
	* Faster parallel reads
	

* Next steps since now we're running our main platform on k8s would be move to k8s jobs
	* shared jobs specs for job definitions
	* shared Argo app and a separate NodePool
	* with an nvidia operator exposing the requested GPU resources
	* Karpenter creating the GPU nodes
	* and Kueue for job prioritization.
	* customer namespaces 
	* Spread topologies here would allow us to spread the compute across multiple availability zones.



* Different queues for different customers mapping to different compute environment could accomodate privacy concerns that some customers might have.



------------------------

## Annotation and evaluation process aka Inference
* Before the fine-tuning pipelines are triggered, the data need to be uploaded and labeled. 
* We talked about how the data are loaded.
* Once they are loaded 
Annotation of images uses
1. SAM2 for auto-annotation
2. Base foundation model for region transfer if you want to annotate 100 images
3. Older fine-tuned models for region transfer if you want to annotate 100 images

* We entertains using different inference servers like:
	* Triton
	* Ray Serve
	* Torch Serve (limited support for nvidia Jetson)
	* BertoML
	* KServe 
	* and even building our own. 
	The decision came down to Triton for the following reasons:
		1. Support for different repositories s3, local
		2. Support for different model formats ONNX models is the prefered one for now
		3. Defining model ensembles was very easy and intuitive
		3. Plays well with the nvidia operator we used on Kubernetes and would be later be helpful for
		4. Sagemaker multi-model endpoints: You can use sagemaker as a repository

### Challenges and tradeoffs during this phase
* How many Triton instances?
	* Warm instance for magic wand (SAM Annotations) and region transfer and
	* Cold instance for evaluation
	
* What is the footprint of extracted models and what is their loading time?
* Should we use Lustre there as well
	

------------------------
## Data Driven Testing
TODO

------------------------

## Distribution of inference results and model artifacts from edge devices back to the main platform
TODO

### Challenges and tradeoffs during this phase
1. SQLlite for state: one lock problem, when multiple inspections were taking place, the inspection results had to be queued before stored and sent to our MQTT broker
	* The solution was to remove all state from the edge devices. The broker was the state.
2. Artifact distribution
3. Authentication: On which internet accessible layer of the factory is the data hub pushing data to the main platform sitting? How does the courier authenticates with the platform? How do the edge pollers authenticate with the courier?
	* One certificate for the data hub pushing data to the platform, another certificate for the edge pollers pushing data to the data hub. The second certificate is wildcarded so that we do not need to create a new one every time a new edge poller needs to be integrated.



------------------------
## Interesting problems with Kubernetes
1. Sync waves applied inside the scope of one app and not and not in between different apps. This resulted in PodMonitors failing when the corresponding Monitoring app that installed the required custom resource was not installed.
	* The solutions are either imposed order or App of Apps where sync waves (priorities) can be imposed in between apps.
	
2. Cluster decommissioning
	* The solution was introducing pre-destroy actions for EC2 Node class and attached nodes, LBs, their VPCs and their attached security groups.

3. Physical and logical backups: How and when should they be performed
4. Migration plan between EC2 and EKS
	* Reusing secrets
	* Reuse resources
	* DNS conflicts that we had to create DNS switchers
	
------------------------

### Questions to answer before main interview process
1. MLJobs using kJobs.
	* Kueue for queues and prioritization
	* Shared compute as part of shared cluster resources / batteries
	* g6 as NodePool instance types
	* Karpenter node pools for AWS GPU compute environment
	* Service accounts for IAM roles
	* Job definition job spec
	Overall: backend creates Kubernetes Jobs directly, Karpenter creates graphics processing unit nodes on demand, Pod Identity gives the training pod bucket permissions, and Kubernetes Job status replaces Amazon Web Services Batch status.
2. Re-inforcement learning basics.
3. Why didn't we use Ray or Kubeflow for fine-tuning?
	* Answered above
4. time-slicing and multi-instance GPU implementations for training pipelines.
5. How do we make sure that changes on TF or jsonnet do not end up on both stg / prod and dev.


------------------------
### Motivation
TODO: Write a motivation for wanting to join Amazon RIVR and the robotics worlds
