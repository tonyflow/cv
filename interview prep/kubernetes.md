## Overview

### How is a VM different than a container?

#### Virtual Machines

A virtual machine is an emulation of a physical computer that runs an operating system and applications just like a physical computer.

##### Isolation
VMs provide full isolation at the hardware level. Each VM runs its own operating system (OS) on top of a hypervisor, which abstracts the underlying hardware.

##### Components
Hypervisor: A software layer that creates and manages VMs by abstracting the physical hardware.
Guest OS: Each VM has its own complete operating system.
Virtualized Hardware: VMs simulate hardware components like CPU, memory, and storage.

##### Resource Overhead
VMs have higher resource overhead because each VM includes a full OS, its own kernel, and virtualized hardware.

##### Startup Time
VMs generally have longer startup times because booting an entire OS takes more time.

##### Use Cases
VMs are suitable for running multiple different operating systems on a single physical server, legacy application support, and scenarios where strong isolation is required.

#### Containers

A container is a lightweight, standalone, and executable package that includes everything needed to run a piece of software, including the code, runtime, system tools, libraries, and settings.

##### Isolation
Containers provide isolation at the application level. They share the host OS kernel but run in isolated user spaces.

##### Components
Container Engine: Software like Docker that manages containers.
Container: A package that includes the application and its dependencies but shares the host OS kernel.

##### Resource Overhead
Containers have lower resource overhead because they share the host OS kernel and do not require a separate OS for each instance.

##### Startup Time
Containers have faster startup times because they do not need to boot a full OS.

##### Use Cases
Containers are ideal for microservices, continuous integration/continuous deployment (CI/CD) pipelines, scalable web applications, and environments where rapid startup and efficient resource utilization are important.

## Kubernetes Components
- When you deploy Kybernetes you get a ***cluster***
- The cluster consists from a set of worker machines called ***nodes*** which run containerized applications. Each cluster will have at least one worker machine / node.
- The worker node hosts ***pods*** which are components
- The ***control plane*** manages the worker nodes and the pods in the cluster.

### Control Plane components
#### kubeapi-server
The API server is the frontend of the Kubernetes control plane

#### etcd
Consistent and highly available key value store used as a back-up store for all of cluster's data

#### kube-scheduler
Watches for newly creted pods which are not assigned to any node and selects and node to run them on.

#### kube-controller-manager
Runs controller processes. There are many different types of controllers
- Node controller: Notice and react when nodes go down
- Job controller: Watches for job objects that represent one-off tasks, then creates pods to run these tasks to completion
- EndpointSlice controller: Populate endpoint-slice objects to create connections between pods and services
- ServiceAccount Controller: Create defailt service accounts for namespaces

#### cloud-controller-manager
Link your cluster to your cloud provider API


### Node components
Node components run on every node and provide the Kurnetes runtime environment

#### kubelet
An agent that runs on every node making sure that containers are running in pods

#### kube-proxy
Maintains network rules no nodes

#### Container runtime
It is responsible for managing and execution and lifecycle of containers in a Kubernetes system

### Other addons
- DNS
- Web UI
- Container Resource monitoring
- Network plugins


### The Kubernetes API
The API lets you query and manipulate the state of API objects like
- Pods
- Namespaces
- ConfigMaps
- Events