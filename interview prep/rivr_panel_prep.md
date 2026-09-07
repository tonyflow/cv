# Table of contents
1. Generalized GPU compute platform
2. Internal Developer Platform: What would it looks like?
	* Submit experiments?
3. Robotics MLOps pipeline 
4. VLA (Vision Language Action) model training. This is a model that processes what the robot sees and hears and reads to directly generate physical robot movements. It's a class of multimodal foundation models that integrates vision, language and actions.
5. Algorithms and neetcode problems I need to solve
	1. Number of Islands
	2. Rotting Oranges
	3. Walls and Gates
	4. Course Schedule
	5. Redundant Connection
	6. Network Delay Time
	7. Cheapest Flights Within K Stops
	8. Swim in Rising Water
	9. Alien Dictionary
	10. Robot Room Cleaner
	11. Dijstra & A*
6. How to ingest HQ video in a performant way
7. Reinforcement learning pipeline
8. Distributed systems based on key/value store design under here: https://bytebytego.com/courses/system-design-interview/design-a-key-value-store 
	* quorums
9. Questions to the panel: Check at the end of the document.
10. TODO
	* MAJOR! Go through the RL pipeline on ChatGPT ✅
		* There are two topics here (1) Locomotion policy deployment
		* RL training pipeline design
	* Go through PyTorch basic on GPT ✅
	* Go through distributed training modes ✅
	* Go thorugh on robot inference 
	* Go through sources of data for robotics pipelines ✅
	* Go through the release gates before deploying to production for robots ✅
	* Go through realtime data ingestion technics for robots (gRPC/MQTT/WebSocket)
	* Are you sure you can explain what Hydra is?
	* Remember model checkpoints and their usefulness
	* Go thrpugh the rivr interview prep question under the rivr project and check if I am missing something.
	* Algorithms to remember
		* BFS
		* Multi-source BFS
		* Bellman-ford negative edges directed with at most k edges used
		* Floyd warshall all pair shortest-path on weighted directed graphs
		* Dijstra
		* A* directed graph with direction heuristics
		* Topological sort
		* Union find
		* Alien dictionary provblem?
	* Go through RL basics? Maybe not
	* Simple coding exercises
	* Go through this document end to end before the interview
	* Exploration VS exploitation

---------------------------------------
# PyTorch basics
1. ```python
model.train() # training
model.eval() # validation and inference
```

2. `state_dict`
A dictionary containing state parameters like weights and biases. 

3. Saving checkpoints
4. Mixed precision training and quantization
5. Distributed training technics
	* `torch.distributed.DistributedDataParallel`
	* `torch.distributed.FullyShardedDataParallel`
6. Tensor is a multidimensional array
7. autograd is PyTorch's differentiation engine
8. What happens during training
	* Forward pass
	* Compute loss
	* Backward pass
	* Optimizer updates the weights
	
---------------------------------------
# Robotics basics
1. Occupancy maps / grids
2. A* compared to Dijkstra
3. Cost maps: More elaborate occupancy maps. Instead of just binary 1/0, the also include informations like grass=5, wall=1000, stairs=20 etc.
4. State space search?
5. SLAM(Simultaneous Localization and Mapping)


---------------------------------------
# SLAM
The main questions that it tries to answer is
1. Where am I?
2. How does the world around me look like?

There's a cyclic dependency between these two questions:
* In order to understand where I am, I need a map
* To build a map, I need to know where I am

This circular dependency is the core challenge.

When a robot enters a space it has never seen before, it starts in a random location. It receives sensor data from:
1. Lidar
2. Camera
3. IMU: Inertia Measurement Units. Measures robot orientation and gravitational forces.
4. Wheel encoder: Measure the rotation (and implicitly velocity of a wheel) as the robot is moving. They are essential for tracking speed, controlling motor accuracy and calculating the robot's position over time.

Using the data from the sensors the robot estimates a pose. A pose is a combinations of (x,y,Θ) in 2D or (x,y,z,roll, pitch, yaw) in 3D. This tuple can identify the orientation of the robot in space.

What are SLAM's major components?
1. Sensors
2. Odometry: Use of sensor data to estimate a robot's change in position and orientation over time relative to its starting point. It is essentially a form of dead reckoning.
3. Mapping: This is the result of the odometry. The robot is gradually building a map
4. Localization: The robot estimates the location inside the map
5. Loop closure: THe robot recognizes a previously visited location. The process can correct accumulated mapping and localization drift.


What makes SLAM hard?
1. Sensor noise
2. Dynamic Objects
3. Doors open
4. Drift
5. Compute cost

From no-SLAM to semi-SLAM to full-SLAM
1. Warehouse map is available. Robot starting point is known. Destination is known.
	* In this case, there's no SLAM needed. This is the case in warehouses.
2. Warehouse map is available. Robot DOES NOT know where it is. Destination is known.
	* The robot must localize itself. A* cannot even start. This is still not a full SLAM
3. Map does not exist. Robot DOES NOT know where it is. Destination is known.
	* Now the robot must estimate pose, build a map and plan the path.
	
How does a SLAM and path finding pipeline look like?
1. Goal
2. Sensors
3. SLAM (includes localization)
4. Current pose + map 
5. path planning
6. trajectory generation
7. motion controller
8. robot movement
9. new sensor data
10. SLAM updates pose + map
11. replan if necessary

	
## Example in the case of RIVR
There are two parts to the "Deliver a package directive"
1. The robot receives an address, it loads a map and it knows it's location on the map through GPS. The path finding does not require SLAM, except the case where the environment becomes dynamic e.g. people in way 
2. Once it reaches the address though another sort of navigation starts:
	* front yard
	* garden path
	* stairs
	* gate
	* porch
	Now the robot needs to
	1. localize precisely
	2. update the local map
	3. replan around obstacles

---------------------------------------
# Reinforcement learning policies
A policy dictates the robot's behavior. I can be anything from:
1. A neural network
2. A decision tree
3. A set of handwriten rules
4. A lookup table

A GPS navigation is an example: At this intersection turn left, at the next intersection turn left. The instructions are the policu and the software running inside the GPS device is the model.

* The policy is what decisions are made
* The model is how these decisions are generated


In modern RL world the words policy and model are used interchangeably e.g.
* I trained a model
* I trained a policy
What was actually deployed is a neural network


---------------------------------------
# Robotics MLOps pipeline

_tl;dr_ The pipeline is not linear; it is a closed-loop system. The fleet continuously generates data, failures are mined into new datasets, datasets are versioned, models are retrained, evaluated offline and in simulation, then deployed gradually back to the fleet with monitoring and rollback.

What are the basic components of a robotics MLOps pipeline
1. Robot fleet data
2. On-robot logging and buffering - very esoteric and could be ommitted in the setting of an interview
3. Cloud ingestion
4. Raw immutable data lake
5. Data curation and labeling
6. Versioned dataset registry
7. Training and experiment tracking
8. Offline evaluation
9. Simulation and evaluation
10. Safety validation and release gating
11. Staged deployment
12. Fleet monitoring
13. Failure mining


## Robot fleet data
The folling is a list of data points, the robots can provide - the list is not exhaustive:
1. Camera frames (video or photos)
2. Lidar points (x,y,z) - GeoJSONs
3. Radar detections
4. IMU data
5. Wheel encoder odometry
6. GPS ?
7. Planned path
8. Executed path
9. Control commands
10. Planner decisions
11. Pperator interventions
12. Faults/crashes/(near)misses

* Apart from raw data, we also log planned outputs, model predictions and ground-truth-ish data. 
* This is the data producer layer.

## On-robot logging/buffering
This is the data capture layer. Where are the data produced in the previous layer actually stored before being ingested?
The data of the previous stage are getting combined with
1. timestamps
2. sensor metadata
3. robot firmware version
4. model(s) version
5. map version
6. mission ID
7. route ID
8. battery state
9. network state

## Cloud ingestion
There are a few ways that this can be done:
1. Synchronous / Real-time
	* Everything that is stored on ROS
2. A-synchronous / Offline or in-batch

The cloud ingestion layer should handle:
1. Authentication
2. Robot identity
3. Upload reusability
4. Schema validation: More on this during data curation.
5. Deduplication
6. Metadata extraction
7. Routing to storage

A robot may upload one large log file plus some metadata like
```json
{
  "robot_id": "robot-041",
  "mission_id": "mission-abc",
  "software_version": "2026.06.12",
  "model_version": "perception-v37",
  "started_at": "...",
  "ended_at": "...",
  "event_tags": ["staircase", "pedestrian", "replan"]
}
```

This is the integration layer: How are we moving the data from the robot into our data lake

## Raw immutable data lake
* This where all the robot fleet data end up. 
* It is usually an object store that the pipeline integrating the data uses to store the robot data.
* Although it's a data lake (or bronze) data set, data can be located like so
```
s3://robot-data/raw/robot_id=.../date=.../mission_id=...
```
This provides:
1. auditability
2. reproducability
3. debuggability
4. ability to rebuild dataset later

Raw data are IMMUTABLE!

## Data curation and labeling
Some of the steps might be:
1. Data quality and validation checks
2. Synchronization/Quantization (are all data points in sync)
	* During the ingestion, we miight have data in the following form
	```
	t = 0.000 camera frame
	t = 0.004 IMU sample
	t = 0.010 wheel encoder sample
	t = 0.050 lidar scan
	t = 0.055 model prediction
	t = 0.060 planner command
	```
	This is one frame but the timestamps are different, we can create clusters of data and assign the timestamp. 
	Some of the ways this can happen:
	1. Create the axis/timeline yourself and the epsilon that each data point should be away from ingestion-defined timestamp
	2. DBSCAN it and create the clusters using ML
	3. Throw it in a neural network
3. Sensor calibration validation
4. Data cleaning
5. Data enrichment
6. Scenario extraction - see below
7. Labeling: Examples depend on the model
	* Scenario labeling e.g. stairs, curbs, wet pavement, dogs, glass doors, night scenes, sensor glare, occlusions, wheel slip, replanning events, narrow passages
	* Based on the model we could have 
		* object detection labels e.g. bounding boxes 
		* semantic segmentation masks e.g. per-pixel classes: every pixel is labeled as sidewalk, grass, stairs, pedestrian etc.
		* traversability labels e.g. can/cannot pass from here 
		* depth occupancy labels e.g. free/occupied 3D space 
		* failure labels robot e.g. got stuck/colliede 
		* human intervention labels e.g. teleoerator took action
	
	In robotics, a lot of labels can be weakly or automatically generated.

8. Dataset construction

## Dataset/version registry
After the data have been curated we need to define an IMMUTALE TRAINING & EVALUATION dataset.

* Select the dimensions based on which we want to version the data
	* raw log IDs: tracks back raw data
	* labels
	* calibration version
	* schema version
	* train/val/test split
	* filtering criteria
	* generation code commit

* Use a tool like DVC that keeps metadata in git and actual dataset in an object store backend.

Example:
```
dataset: traversability-v12
includes:
  42k stair samples
  18k curb samples
  11k wet-surface samples
  90k normal sidewalk samples

excludes:
  corrupted calibration
  pre-2026 camera firmware
```

## Training and experiment tracking

In this stage, the training is taking place and we record everything that is necessary to reproduce the run
* code commit
* container image
* dataset version
* hyperparameters
* model architecture (???)
* pretrained checkpoint
* Random seed: Should be random during training but it should be tracked for reproducability
* training metrics e.g. training loss, training accuracy
* validation metrics e.g. mAP, IoU, precision/recall
* artifacts - resulting model 
* hardware the training was ran on

A robot might employ many different models
1. Object detection.
2. Semantic segmentation: per-pixel classification, every pixel is labeled as sidewalk, grass, curb, stairs, pedestrian etc.
3. Traversability prediction: Predict whether the robot can safely drive through an area. Output is often a traversability score or binary decision.
4. Terrain classification: Classify terrain types like sidewalk, gravel, grass, mud, stairs, wet pavement.
5. Visual odometry: Estimating robot motion using camera images instead of wheel encoders
7. Local costmap prediction
8. Failure prediction

## Offline evaluation
This is about evaluating the model on static datasets before simulation or deployment.
* outside simulation environment
* before deployment to the actual robots

Metrics like the following should be tracked:
* precision / recall
* train loss
* evaluation accuracy and evaluation loss
* mAP: Mean Average Precision
* IoU: Intersection over Union. Measures overlap between prediction and ground truth.
* Calibration error
* False traversable rate: Percentage of truly unsage areas incorrectly predicted as traverseable.
* false obstacle rate: Percentage of safe areas incorrectly predicted as obstacles. Causes unecessary avoidance and inefficiency.
* terrain class accuracy
* colission prediction error
* path feasibility accuracy
* intervention prediction recall

## Simulation/replay evaluation
This is robotics specific stage that most MLOps pipelines lack.

There are two options here:
1. Feed recorded logs (video feed, lidar IMU planner state camera, commands) through the new model
2. Simulation evaluation where the robot is evaluated inside a synthetic or reconstructed environment.

We want to answer questions like:
1. Would the new model have made better predictions on real historical data?
2. Would the robot complete the task safely under varied conditions?

## Safety validation and release gating
Hardening rollouts. These are certain release gates that the molde should pass before production roll-out:
1. no regression on safety critical scenarios
2. better or equal performance on golden datasets
3. bounded latency on target hardware
4. bounded memory usage
5. deterministic startup behavior
6. compatible with robot software version
7. passes simulation scenario suite
8. passes shadow-mode comparison

## Staged deployment
Before rolloing out to the entire fleet
1. 1 robot internal testing
2. shadow mode
3. Small canary/pilot fleet: Canary fleet could be segmented by risk (???)
4. Regional subset e.g. 50 robots in area X
5. Larger fleet percentage (25% of the fleet, 50% of the fleet, 60% of the fleet etc)
6. Full deployment

Deployment must include:
1. model artifacts
2. config
3. runtime container
4. feature flags
5. rollback plan
6. compatibility constraints

## Fleet monitoring
ML Monitoring
1. Prediction confidence
2. Class distribution drift: Prediction distribution changes over time e.g. Model used to see 5% stairs but now sees 25% of stairs because robots were deployed in a new neighborhood.
3. Model latency
4. Model errors

Robot monitoring;
1. (Operator) intervention rate
2. Stuck rate
3. Collisions / near-collision events
4. Localization failures
5. CPU/GPU mem usage
6. Network upload backlog
7. Fall rate
8. Battery impact
9. Crashes/restarts
10. Operator complaints
11. Slip events
12. Generic error tracking

Production quality is measured by full task outcomes not just model outcomes.

## Failure mining & new datasets
In this phase we are mining specific for failures:
1. Robot got stuck
2. Human teleoperator intervened
3. Planned oscillated
4. Near collision
5. Unexpected obstacleROute abandonded
6. ...

We can fine-tune soecific for the failure cases:
1. Failure case
2. Data points
3. Label
4. Dataset version
5. Retrain
6. Evaluate
7. Redeploy

### Rollback
Every robot should retain a copy of its last working policy locally whose deployment can be triggered under certain events.

### Instead of a summary
The MLOps pipeline focuses on:
1. data collection
2. versioning
3. training
4. simulation
5. rollout
6. monitoring
7. feedback loops

The points in-between are mostly detailed/expanded phases or robotics-related phases.

---------------------------------------
# MLOps pipeline for a Locomotion RL policy
Before we start: Locomotion is the act of moving from one place to another.
The expected steps for deploying and maintaining a locomotion ML policy are the following - in similar fashion the the general robotics MLOps pipeline:
1. Data generation in simulation - Isaac Sim
2. Distributed training: Ray on PyTorch
	* Data distribution
	* Model distribution
	* Pipeline distribution
	* Fully sharded
3. Experiment tracking
4. Hyperparameter management (Hydra) (???)
5. Checkpointing (???)
6. Dataset versioning DVC
7. Evaluation gates
8. Promotion to production
9. Rollback strategy

---------------------------------------
# Model Checkpoints
A model checkpoint is a saved snapshot of a machine learning model at a specific point during training. The checkpoint is a snapshort of:
* Model weights
* Config

They are useful for:
1. Failure recovery
2. Selecting the best model: It's not necessarily the model corresponding to the end of the training cycle the one with the best evaluation results
3. Experimentation
	* Compare different training runs
	* Fine-tune from an existing model
	* Rollback to an earlier version of the model
4. Rollbacks: If for any reason deployed checkpoint A is failing on production, we can rollabck to checkpoint B which has been battle tested.

---------------------------------------
# Deep neural nets parallel training

## One GPU for all
CUDA underneath - or Apples metal shaders what whatever they are using

## Data parallelism
1. Each GPU has a full copy of the model
2. Forward pass is identical on all GPUs and is performed independently. This step is where the loss is computed
3. The gradients are synchronized: This synchronization is happening using different frameworks like 
	* PyTorch DDP
	* NCCL
	* Horovod
4. Once the gradients have been synchronized, the weights are identical on all GPUs

This method represents 80-90% of the distributed training happening in production systems.

## Model Parallelism
1. Each GPUs is assigned a subset of the layers of the model
	* GPU0:layers 1-10
	* GPU1:layers 10-20
	
2. The control flow is: Input => GPU0 => GPU1 => output

This approach is used when the model cannot fit into the memory of 1 GPU. I have never implemented such a parallelism.

## Pipeline Parallelism
```
GPU1: Embeddings
GPU2: Transformer blocks 1-12
GPU3: Transformer blocks 13-24
GPU4: Output head
```

This like the model parallelism but we are at the same time passing batches of data through each GPU

G0 layers 0-10    b0.    b1     b2  b3
G1 layers 10-15   None   b0     b1  b2
G2 layers 15-20   None.  None   b0  b1 and so on

## Hybrid: 3D Parallelism on data, model and parallelism
Such hybrid models are deployed for frontier models deployments e.g. GPT4

## Fully Sharded Data Parallel (FSDP)
A modern alternative to Distributed Data Parallel. Instead of every GPU holding:
* Model
* Optimizer
* Gradients

each GPU stores only a shard
```
GPU1: 25%
GPU2: 25%
GPU3: 25%
GPU4: 25%
```
and weights are gathered only when they are needed.

* It enables much less memory consumption on larger models. e.g. 
	1. https://docs.pytorch.org/docs/2.12/fsdp.html
	2. Deep Speed 0: https://www.deepspeed.ai/tutorials/zero/?utm_source=chatgpt.com


---------------------------------------
# Questions to the panel
5. You're looking for a senior engineer. Not a junior, not a principle. What are your expectations from such a hire in a horizon 1,3 and 6 months?
6. What is the ballpark of data you're retrieving from every robot?
8. What is the ballpark of the data you are processing and how often do you ingest data if you have no real time data ingestion.
9. 
7. What is the size of your robot fleet.
1. Djibril asked me about knowledge of Java during the first inteview. Which parts of your stack are written in Java? Are you using it extensively?
2. How severe is the friction due to research engineers not having the appropriate infrastructure to run their experiments?
3. Tell me about your scaling plans: Eric mentioned that you performed 10k deliveries last year in the States. What is the plan for this year?
4. I am currently a lead of the Platform team. Although I do not care about titles I want to know that there is the potential to grow into such a role here as well and work with multiple teams simultaneously.
8. Are you expecting the engineer to be on standby for any reason?












