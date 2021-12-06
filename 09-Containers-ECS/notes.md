## 09-Containers
What we refer to as Virtualization should be called OS Virtualization which involves running multiple OS on the same hardware
### Intro to Containers

Virtualisation Problems

AWS EC2 Host : base
AWS Hypervisor (Nitro)

If you run a virtual machine with 4 GB ram and 40 GB disk,
the OS can comsume 60-70% of the disk and little available
memory.
![image](https://user-images.githubusercontent.com/33827177/144770838-11d34aae-019c-424e-89ec-7ef708948687.png)
If all of the OS use the same or similar resources, they are
duplicates. Every restart must manipulate the entire OS.

What you want to do in order to run applications in their own
isolated enviroments for their applications to run? Do we really want separate OS each taking disk space and resources? NO...

Containers handle things much differently. We have Host Hardware and a Host OS. Running on top of the OS is a Container Engine like Docker.
A container is similar to a virtual machine because it provides an isolated environment in which an application can run.
But unlike a virtual machine, a container runs within a host OS as an isolated process. It is isolated from other processess but it can use
the host OS for networking, file I/O, etc.

Example Scenario: If the OS is Linux, it can run Docker as container Engine. Linux + Docker can run a container. That container would run
as a single process within Linux potentially with other container processes. But inside the container, it's like an isolated OS, it has its
own file system and it can run child processes.

![image](https://user-images.githubusercontent.com/33827177/144771522-997bde47-2d7d-4530-9875-16aad20f4d2d.png)

#### Image Anatomy

Container is a running image of docker image.

These are stacks of layers and not a monolithic disk image.

Docker Images are created by a Docker file

Each line of a docker image creates a new filesystem layer.

Images are created from scratch or a base image. e.g in the example below a based image of CentOS 7 is created as
a base image a thin layer of file system that can run CentOS 7 disctibution.

The next line adds another layer for a webserver in this case APACHE

Images contain read only layers, images are layer onto images.

The next line adds another file system layer for a total of 3
![image](https://user-images.githubusercontent.com/33827177/144771894-2d0d6e2c-6b99-4f52-8be5-0afbd8b405a9.png)

In reality the Docker Image is upside down with the based image at the botton, then the webserver and the final layer of customization

Docker container is the same as a docker image (Docker Image is read only), except it
has an additional READ/WRITE layer of the container. All the data of the application is stored in the READ/WRITE layer of the container
![image](https://user-images.githubusercontent.com/33827177/144772274-28f3683a-a8a1-4804-bd9e-6e7079458c0b.png)
If you have lots of containers with very similar base
structures, they will share the parts that overlap. For example, in the image above the R/W layers are different so the other layers are shared. Imagine 200 containers sharing the 3 layers...

The other layers are reused between containers.

#### Container Registry

Registry or hub of container images.

Dockerfile can create a container image where it gets stored
in the container registry.

Docker hosts can run many containers between one or more images.

#### Container Key Concepts

Dockerfiles are used to build images

Portable and always run as expected. Anywhere there is a compatable host,
it will run exactly as you intended. --> Portability and Consistency

![image](https://user-images.githubusercontent.com/33827177/144772415-61a34241-ad75-4b3b-b6e3-e8bb2d758000.png)

Containers are super lightweight, use the host OS for the
heavy lifting, but otherwise are isolated. File system layers
are shared when possible.

Containers only run the application and enviroment it needs very little memory.

Ports are isoalted so anything it runs need to be **exposed** to allow outside world to access the application its running. 
This can be done by exposing ports of the host for example, TCP port 80 the host and beyond.

Application stacks can be multi container. Meaning you can use, multiple containers in a single architecture running different applications. e.g. a database container, application container, ML container

### Elastic Container Service (ECS) Concepts

Accepts containers and instructions you provide.

ECS allows you to create a cluster. Clusters are where containers run from.

Container images will be located on a registry.
AWS provides ECR (elastic container registry). You can use that or Docker Hub. Just in this case you won't require have to deal with permission, etc.

Container definition is like a pointer which tells ECS where the container image is store and which port it is going to be exposed, and similarly other info. needed to run the container

Task definition represents a self-contained application. Task definition represents the application as a whole and stores whatever
information is needed to run the application. It could have one container defined inside it or many.
The task definition stores information about the resources used by the task., so CPU and Memory. They also store the networking mode that task uses. It also stores the compatibility i.e whether the task will work on EC2 mode or Fargate. It also stores the IAM  Role the task will assume

Container is just a pointer to where the container is stored and what port is
exposed. The rest is defined at the task definition.
https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_ContainerDefinition.html

Task definitions store the resources and networking used by the task. It stores
the compatability of how the container runs. It also stores **task role**, an
IAM role that allows the task complete its task. This is the best practice
way to give containers the access needed for other AWS resources.
https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_TaskDefinition.html
A lot of tasks will only have one container definition, but a task could
include one or more containers.

Task is not by itself highly available and it doesn't scale on its own.

ECS Service is configured via a Service Definition. This defines how many copies of the task
is allowed to run to load balance. It can add capacity and reselience (The service will replace failed tasks. 
You can deploy a Load Balancer in front of a service to provide scalability
and high availability.
![image](https://user-images.githubusercontent.com/33827177/144780073-0ce78396-6153-4e40-a17b-4e4914668a5e.png)
Tasks or services get deployed to an ECS cluster.

**Container definition** the image and the ports that will be used.

**Task definition** the task role and security is defined. This is an IAM
role that is assumed. The temporary credentials allow access to AWS products
and services.

**Task role** IAM role which the task assumes.

**Service** how many copies of a task you want to run for scaling and
high availability.

### ECS Cluster Types

ECS Cluster manages

- Scheduling and Orchestration
- Cluster manager
- Placement engine

#### EC2 mode

ECS cluster is created within a VPC. It benefits from the multiple AZs that
are within that VPC.

You specify an initial size which will drive an **auto scaling group**

These are just like EC2 insances with containers running in them. You will see them in your account and can even connect with them.

It is to be noted when they are provisioned you will be paying for them regardless of what containers are running on them??

ECS using EC2 mode is not a serverless solution, you need to worry about capacity for your cluster.

The container instances are not delivered as a managed service, they are managed as normal EC2 instances through ECS tooling and you need to worry about availibility and capacity
![image](https://user-images.githubusercontent.com/33827177/144934955-6f8713f6-dbcc-4b78-8385-c0e58270582c.png)
This is good because you can use spot pricing or prepaid EC2 servers.

#### Fargate mode

Removes more of the management overhead from ECS, no need to manage EC2.

There is a **fargate shared infrastructure** which allows all customers
to access from the same pool of resources.

Fargate deployment still uses a cluster with a VPC where AZs are specified.
![Uploading image.png…]()
For ECS tasks, they are injected into the VPC. Each task is given an
elastic network interface which has an IP address within the VPC. They then
run like an VPC resource.

You only pay for the container resources you use based on the resources they consume. So you've no visibility of the host costs, you dont need to manage host, provision hosts or worry about their availibility or scalibility

#### EC2 vs ECS(EC2) vs Fargate

If you already are using containers, use **ECS**

**EC2 mode** is good for a large workload with price consciousness and are not worried about effort. This allows for
spot pricing and prepayment.

Large workload but overhead conscious **Fargate**

Small or burst style workloads **Fargate** makes sense

Batch or periodic workloads **Fargate**
