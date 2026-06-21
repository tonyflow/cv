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
9. Questions to the panel
	* You're looking for a senior engineer. Not a junior, not a principle. What are your expectations from such a hire in a horizon 1,3 and 6 months?
	* What is the ballpark of data you're retrieving from every robot?
	* What is the size of your robot fleet.
	* Are you expecting the engineer to be on standby for any reason?
	* 
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
2. Where does the world arounf me look like?

There's a cyclic dependency between these two questions:
* In order to understand where I am, I need a map
* To build a map, I need to know where I am

This circular dependency is the core challenge.

When a robot enters a space it has never seen before, it starts in a random location. It receives sensor data from:
1. Lidar
2. Camera
3. IMU: Inertia Measurement Units. Measures robot orientation and gravitational forces.
4. Wheel encoder: Measure the rotation (and implicitly velocity of a wheel) when as the robot is moving. They are essential for tracking speed, controlling motor accuracy and calculating the robot's position over time.

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
1. The robot receives an address, it loads a map and it knows it's location on the map through GPS. The path finding does not require SLAM, excpe the case where the environment becomes dynamic e.g. people in way 
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
1. camera frames
2. lidar points (x,y,z)
3. radar detections
4. IMU data
5. wheel encoder odometry
6. GPS ?
7. planned path
8. executed path
9. control commands
10. planner decisions
11. operator interventions
12. faults/crashes/(near)misses

* Apart from raw data, we also log plannet outputs, model predictions and ground-truth-ish data. 
* This is the data producer layer.

## On-robot logging/buffering
This is the data capture layer. Where are the data produced in the previous layer actually storeed?
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
2. A-synchronous / Offline or in-batch

The cloud ingestion layer should handle:
1. Authentication
2. Robot identity
3. Upload reusability
4. Schema validation
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
	
	In robotics, a lot of labels can eb weakly or automatically generated.

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
1. object detection
2. semantic segmentation: per-pixel classification, every pixel is labeled as sidewalk, grass, curb, stairs, pedestrian etc.
3. traversability prediction: Predict whether the robot can safely drive through an area. Output is often a traversability score or binary decision.
4. terrain classification: Classify terrain types like sidewalk, gravel, grass, mud, stairs, wet pavement.
5. visual odometry: Estimating robot motion using camera images instead of wheel encoders
6. place recognition
7. local costmap prediction
8. failure prediction

## Offline evaluation
This is about evaluating the model on static datasets before simulation or deployment.
* outside simulation environment
* before deployment to the actual robots

Metrics like the following should be tracked:
* precision / recall
* mAP: Mean Average Precision
* IoU: Intersection over Union. Measures overlap between prediction and ground truth.
* calibration error
* false traversable rate: Percentage of truly unsage areas incorrectly predicted as traverseable.
* false obstacle rate: Percentage of safe areas incorrectly predicted as obstacles. Causes unecessary avoidance and inefficiency.
* terrain class accuracy
* colission prediction error
* path feasibility accuracy
* intervention prediction recall

## Simulation/replay evaluation
This is robotics specific stage that most MLOps pipelines lack.

There are two options here:
1. Feed recorded logs (video feed, lidar IMU planner state camera, commands) through the new model
2. Simulation evaluation where the robot is evaluated inside a sunthetic or reconstructed environment.

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
3. small canary fleet
4. sepcific geography
5. larger fleet percentage
6. full deployment

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
1. Intervention rate
2. Stuck rate
3. Collisions / near-collision events
4. Localization failures
5. CPU/GPU mem usage
6. Network upload backlog

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
# Questions to the panel
1. Djibril asked me about knowledge of Java during the first inteview. Which parts of your stack are written in Java? Are you using it extensively?
2. How severe is the friction due to research engineers not having the appropriate infrastructure to run their experiments?
3. Tell me about your scaling plans: Eric mentioned that you performed 10k deliveries last year in the States. What is the plan for this year?
4. I am currently a lead of the Platform team. Although I do not care about titles I want to know that there is the potential to grow into such a role here as well and work with multiple teams simultaneously.












