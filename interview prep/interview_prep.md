# Introduction

The product is an agentic platform with 3 main products:
1. Vision defect detection and classification. Using context aware and unaware models.
2. Causal inference on process models
3. Anomaly/outlier detection on time-series
All these products operate on different domains of the manufacturing industry and they are generic to be used from chemical uses cases to automotive use cases. On top of all the products there are agents interacting with each product's APIs.

I am currently the lead of the platform team that is sitting in the middle of the product/features teams and accomodates all their needs in terms of data and operations.

We are not the typical platform team in the sense that we are not all Devops. I have a heavy operations background but I cannot call myself a devops. The team consitutes of 3 mostly full stack engineers - one of which is also customer facing - and 3 DevOps.

For example:
1. For the vision team we:
	* Architected the GPU compute infrastructure that runs our foundation models' fine-tuning pipeline.
	* Building a distributed system that enables performant inference on edge devices and collects feedback on these edge devices that can be used for fine-tuning our foundation model.
	* Provision GPU node pools on Kubernetes clusters for pre-training inference e.g. image annotation, region type transfer and post-training inference e.g. evaluation runs.
2. Our causal inference algorithms identifies causal relations in manufacturing process models. There 2 important parts there:
	* Data modeling: The context layer is the most important part of our causal inference algorithm. And the complexity comes from its ability to represent complex relationships by still mainting its generic nature.
	* To accomodate this we have designed and mainting an e2e data pipeline: 
		* From a worker model for data integration to
		* A composable parser paradigm for extract/reading the raw information and applying business and customer-specific transformation logic to
		* Semantic enrichment just before perstisting the data to disk
	* LightGBM compute
3. The anomaly/outlier detection is most straight forward uses case that we have:
	* For rule based and lower dimensions the approach is very simple since we just need to check incoming values against certain spec limits
	* For higher dimensions we are using principal component analysis for dimentionality redunction, then compute a Mahalanobis distance (Hotelling's score) and then it's a bit of black magic when it comes to finding the correct threshold and decide which samples are anomalous and which are not.
	

Kubernetes migrations
* 2 implementations areas on 5 gates
* 2 implementation areas
	* Terraform:
		* EKS
		* core customer modules
	* jsonnet
		* batteries applications
			* grafana k8s monitoring
			* network policies
			* argocd
			* ...
		* Product specific application
		

* Migration Gates
	* Parity with EC2
	* backups and restores
	* EC2 to EKS migration process
	* dev experience: 
		* local development, 
		* live syncs of local and remote clusters
		* documentation
	* EKS to EC2 recovery process
	

# Older
In the past almost 4 years I have been working for 1PlusX and I was part of two teams:
* Data consolidation
	1. Data ingestion: All different kinds of ways. Building the pipelines real-time or not that allow these data to enter the system
		- In-browser Javascript SDKs
		- S3 buckets
		- GCP
		- Data streaming platforms aka Kafka topics
		- REST APIs
	2. Canonicalization: Provide a uniform schema so that downstream pipelines do not need to reprocess the data - at least not in terms of schema changes
	3. ID tracking
		- Mention disjoint sets
	4. Provide an attribute definition DSL
* Audience
	1. Teams sits at the very end of the data processing and consolidates all data sources from all different teams into user profiles
	2. Clean room: Safe data exchange between data vendors
