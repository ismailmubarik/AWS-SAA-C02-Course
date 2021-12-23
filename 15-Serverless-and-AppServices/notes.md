## Serverless and AppServices

### Architecture Evolution

Youtube

Once a video is uploaded to Youtube, it creates different versions in
different quality levels. Historically the most popular way to make this work
was a monolithic architecture.

#### Monolithic
![image](https://user-images.githubusercontent.com/33827177/147016320-c51ac63c-c8d2-48c8-b3f6-3fce364f3164.png)
- It fails together. One error will bring the whole system down.
- Scales together. Everything expects to be running on the same compute hardware
- Bill together. All components are always running and always incurring charges.

This is the least cost effective way to architect systems.

#### Tiered

The different components can be on the same server or different servers.

The components are coupled together because the endpoints connect together.

You can independently increase the size of the server that is running each application tier.

We can utilize load balancers in between tiers to add capacity.

The tiers are still coupled, the upload tier **expects** and requires
processing to respond. If the Processing fails completely, the upload tier
will fail because it does not recieve a response. So the upload tier needs at least one processing
tier to ex![image](https://user-images.githubusercontent.com/33827177/147016689-c9978d18-8f29-451a-a6ab-3c3cbee24107.png)ist. 
![image](https://user-images.githubusercontent.com/33827177/147016967-5d401547-1528-433e-a00a-70425d490705.png)
If there is a backload in one tier, it will impact the other tiers and the
customer experience.

Even if there is no job to be processed, the middle tier will need to be
running because otherwise it would fail. So we need at least one istance of Upload, Processing, and Storage tier.
This means if demand is zero we will still have one instance each

#### Evolving with Queues

This is a system that accepts messages off a queue. Often times
they are **FIFO** (first in, first out)

Instead of passing data into the processing tier. It will store this in an S3
bucket as well as detailing the information into the queue. This now
moves towards the next slot in the queue.

The upload tier doesn't expect an immediate answer from the processing tier.
The queue has the location of the S3 bucket as well as the location
of the information.
It sends an asycn message. While this is happening, the upload can add more
messages to the queue.
![image](https://user-images.githubusercontent.com/33827177/147017140-2a295001-2028-4297-9321-30f7ca689fcf.png)

The queue will have an autoscaling group to increase capacity at the end and
process appropriately. Similarly, once the jobs are processes, they are deleted
from the queue. If we now less jobs remaining the ASG can delete one processing
instance.
![image](https://user-images.githubusercontent.com/33827177/147017172-e16870d8-c5d6-473f-98d8-08c8fd727ca5.png)

![image](https://user-images.githubusercontent.com/33827177/147017497-6c937df0-c7ca-4d0c-a3d6-0b81857a9b39.png)

The autoscaling group will only bring up servers as their needed. Using the Queues components are decoupled and
can scale independently

#### Event Driven Architecture
![image](https://user-images.githubusercontent.com/33827177/147017773-e4b2ac2b-b103-4cd8-afb7-aa5921c80d7f.png)
Event producers - interact with customers or systems monitoring components.
They produce events in reaction to something

Event consumers - pieces of software waiting for events to occur.

Services can be producers and consumers at once.

In either case, there are no resources waiting around to be used. That is neither the producers or the consumers.
![image](https://user-images.githubusercontent.com/33827177/147019982-bed3705a-efca-4011-94ee-8517b3d91603.png)
Event router is needed for event driven architecture that also manages
an event bus.

#### Highlights

- No constant running or waiting for things
- Producer generate events when something happens
- Clicks, events, errors, actions
- Events are delivered to consumers of events
- Actions are taken and the system returns to waiting

Mature event driven architecture only consumes resources while handling
events. 

### AWS Lambda

![image](https://user-images.githubusercontent.com/33827177/147021073-01b56e58-6f81-4073-8dba-ac3259531285.png)

#### Lambda Architecture
Every time a Lambda functions is invoked a new runtime environment is
created with all the component that the functiond needs. Once runs it terminates.
If we want to run again a new runtime environment is invoked (there are exceptions though). Lambda functions are
state less. Meaning no  data is stored from previous runs.

You also define the resources that your Lambda runtime environment gets. Minimum is 128 MB. Max is 3 GB. 64 MB Steps.
You dont control the virtual CPU. This scales with memory. Also some disk space.

Best practice is to make it very small and very specialized.
The runtime enviroment will match the language the script is written in.
The runtime enviroment is lime a mini container provided with resources.
![image](https://user-images.githubusercontent.com/33827177/147021286-3bd120c1-9633-4d6f-ad38-3cab2fc2d3fd.png)
Lambda functions can be given an IAM role, **execution role**. Whenever that
function executes, the code inside has access to permissions to other products and services
![image](https://user-images.githubusercontent.com/33827177/147021943-85a5229a-daeb-4675-8eea-25ae3d11e779.png)

This can be **event-driven** or **manual** invocation many ways.

Each time you invoke a lambda function, the enviroment is new and clean

These are by default public services and can access any websites. By default
they cannot access private VPC resources.

Once they are configured, they can only access resources within a VPC.

Should always use AWS services for input and output.

Lambda functions can run up to 15 minutes. That is the max limit.

Lambda has two networking modes. Public and VPC Networking

For public networking we start with an AWS environment where Lambda is running. 
![image](https://user-images.githubusercontent.com/33827177/147022377-7aa938b7-108f-4a00-a674-6030ada4b2fb.png)

For VPC networking mode we have a private subnet inside the VPC. By default it does not have access to public internet or services unless
the private subnet is configured
![image](https://user-images.githubusercontent.com/33827177/147022685-ee5538d9-fab5-4997-a94d-26fdf9aa96d8.png)

VPC Lambda functions actually dont run within the VPC they used to  work like FarGate. We have Lambda Service VPC and Customer VPC. Each Lambda service when invoked would create and Elastic network within the customer VPC. Configuring the Elastic Network takes time and doesnt scale well
![image](https://user-images.githubusercontent.com/33827177/147022813-fcf7d4ad-1f66-478c-b323-1ebe9ed10f09.png)
Now it uses Elastic Network Interfaces based on subnets and security groups
![image](https://user-images.githubusercontent.com/33827177/147023077-c59ff71b-c12f-41b1-895b-cb32cea8e723.png)

Rule of Thumb: Treat any Lambda in a VPC like any other service running in the VPC. It will follow the rules and permission policies

![image](https://user-images.githubusercontent.com/33827177/147023215-e33da237-2bfc-4235-b91e-1e9773e63059.png)

![image](https://user-images.githubusercontent.com/33827177/147024015-0d940e01-a3b2-4925-8ffd-0cac2b649b06.png)

### Lambda Function Invocation
![image](https://user-images.githubusercontent.com/33827177/147024635-ad1ceaba-aeb2-4206-aa57-14ec057a648c.png)

![image](https://user-images.githubusercontent.com/33827177/147024913-94f64b87-4da6-4ae0-a1b7-d0bcd18d4a2c.png)

![image](https://user-images.githubusercontent.com/33827177/147024999-a12b23c3-1e01-451a-86b2-bc83ac8e0bc3.png)

### Lambda Versions
![image](https://user-images.githubusercontent.com/33827177/147025346-17867668-5d53-4548-a3c9-4e38fad3a47a.png)

### Lambda Startup Times
![image](https://user-images.githubusercontent.com/33827177/147170831-56994e97-07ff-4dd4-ac82-f63f5b5313f7.png)


#### Key Considerations

- Currently 15 min (900s) execution limit
- Assume each execution gets a new runtime environment
- Use the execution role which is assumed when needed
- Always load data from other services from public API's or S3
- Store data to other services (e.g. S3)
- 1M free requests and 400,000 GB-seconds of compute per month

### CloudWatch Events and EventBridge

Delivers near real time stream of system events that describe changes in AWS
products and services. EventsBridge will be replacing CW Events.

EventsBridge can also handle events from third parties. Both share the same
underlying architecture. AWS is now encouraging a migration to EB.

#### Key Concepts

They can observe if X happens at Y time(s), do Z. This is basically
CW Events V2.

Both systems have a default Event bus for a particular AWS account.

In CW Events, there is only one bus (implicit), this is not exposed.
EventBridge can have additional event buses. These can be interacted with
in the same way as the default bus.

Rules match incoming events or schedules. The rule matches an event and routes
that event to one or more targets as you define on that rule. e.g. invoke a specific Lambda function

Architecturally at the heart of event bridge is the default event bus. The default is the default Event Bus.

Within Eventbridge we have rules. Rules are created and they are linked to specific Event Bus

The default event bus is moving packets of JSON data. The data (json) in the event bus can be used by the targets (e.g. lambda)

### API Gateway

Application Programming Interface (API)

This is a way that applications or services can communicate with each other.

Endpoints are used to access services. Each service has its own endpoint
in its own region.

When you request AWS stops an EC2 instance, the message is set to the API
in that region for that resource.

APIs also perform authentication using passwords or keys. API authorizes
each service and needs your permissions verified each time.

#### Authentication

#### Authorization

API gateway is an AWS managed service that provides managed API endpoints.
Allows you to create, publish, monitor, and secure APIs as a service.

Billed based on the number of API calls as well as data transfered.

This can be used for serverless architecture to provide an entry point
for that design.

This is great during an architecture evolution.

Step 1:
Create a managed API and point at the existing monolithic application.

Step 2:
Using API gateway allows the buisness to evolve along the way slowly.
This might move some of the data to fargate and aurora architecture.

Step 3:
Move to a full serverless architecture with DynamoDB

### Serverless

This is not one single thing, you manage few if any servers.
Applications are a collection of small and specialized functions.

These functions are stateless and run in ephemeral environments.
Each time they run, they obtain the data they need each time.

Generally, everything is event driven. While not being used, there should
be little to no cost due to compute not being used.

Should use managed services when possible.
![image](https://user-images.githubusercontent.com/33827177/147171554-fd17652d-8ee2-4820-b257-e7afc72cc111.png)
#### Example of Serverless

She browses to a static website that is running the uploader. The JS runs
directly from the web browser which provides the frontend for the PetTUbe Application

We use a third party auth provider, google in this case. Authenticate via
**token**.

AWS cannot use tokens provided by third parties. In this case another
service **Cognito** is called. This swaps the third party token for AWS
credentials.

It uses these temporary credentials to upload a video to S3 bucket.

The bucket will generate an event once it has completed the upload.

When the Video is uploaded to the Originals Bucket it is configured to
generate and event. That event contain details of the video upload.
The event will trigger a lambda to process the video. The Lambda function
creates jobs within the AWS Elastic transcoder service (managed service by 
AWS which mainipulate the media like generate media of different sizes, etc.) 
The transcoder will get the original S3 bucket video  location and will use this for its workload.
![image](https://user-images.githubusercontent.com/33827177/147172988-74e7059f-25a6-4b71-981f-150fa8a0415d.png)
These will be added to a new transcode bucket and will put an entry into
DynamoDB.

The user can then interact with another Lambda which will allow her to
pull the media from the transcode bucket using the dynamoDB entry.

### Simple Notification Service (SNS)

HA, Durable, and Secure service.

This is a public service which needs access to the public endpoint. This
allows anyone from the public internet to access it.

Messages are under 256KB in size.

SNS topics are the base entity of SNS. On these topics the permissions are 
controlled and most configurations for SNS are defined.

SNS has a concept of a publisher. A publisher sends messages to a topic. 
Topics can have subscribers which recieve messages. They can come in many
different forms like HTTP(s) end points, Email addresses(-JSON), SQS Queues, 
Mobile Push notifications, SMS Messages & Lambda.

Things like APis can be subscribers and producers at the same time.

You can create topics inside the SNS.

SNS used across AWS for notifications e.g. CloudWatch and CloudFormation
![image](https://user-images.githubusercontent.com/33827177/147173545-d99d6d21-6a29-4883-9bcc-49e6c03bbcd4.png)
By default all topics will recieve the message, you can put filters on
those lines to make sure they don't trigger additional lambdas.

You can use fanout to process different flows to SQS Queues

Offers:

- Delivery Status including HTTP, Lambda, SQS
- Delivery Retries - Reliable Delivery
- HA (within a Region) and Scalable (within a Regional and can cope with a range of workloads from nothing to highly transactional workloads)
- SSE (server side encryption) meaning on-disk encryption
- Topics can be used cross-account via Topic Policy (like resource policy)

### AWS Step Functions
![image](https://user-images.githubusercontent.com/33827177/147174422-d619448a-f63c-4e4f-865e-fb6291574c0e.png)
There are many problems with lambdas limitations that can be solved with
a state machine.

This is a serverless workflow

- Start
- States
- End

States are **things** which occur inside these workflows.

Maximum duration is 1 year

Standard workflow and express. At a high level, standard is the default
and has a 1 year workflow. Express is for high volume event processing 
workloads like IoT and highly transactional such as IoT, streaming data processing, 
mobile app backends and can run upto 5 minutes

Started via API Gateway, IoT Rules, EventBridge, Lambda.

Amazon States Languate (ASL) - JSON template

These use IAM Roles for permissions.
In summary step functions lets you create state machines. State machines are long running serverless workflows. They have start and end and in between they have states. States can be directional points or they can be tasks
#### States

- Succeed & Fail : Will wait until either is achieved
- Wait : will wait until specific date and time or for period of time
- Choice : different path is determined based on an input
- Parallel : will create parallel branches based on a choice
- Map : accepts a list of things e.g. a list of orders and for each order the Map function a set of actions
- Task : Single unit of work ( can be integrated lambda, batch, dynamoDB, ECS, SNS,SQS, Glue, SageMaker, EMR, Step Functions)
![image](https://user-images.githubusercontent.com/33827177/147175694-47a88994-ef91-42bc-95d8-0bf7535c3dbf.png)

### API Gateway - Referesher

![image](https://user-images.githubusercontent.com/33827177/147175895-255c6237-9536-4eb0-b4e3-943924b93d99.png)

![image](https://user-images.githubusercontent.com/33827177/147176087-b6002af8-c904-42ab-8662-d98fcdb25ed4.png)

![image](https://user-images.githubusercontent.com/33827177/147176119-73fa44e9-e443-468e-a93f-e147a3653203.png)

![image](https://user-images.githubusercontent.com/33827177/147176223-97b23dd5-6638-4edf-8900-3d8879cced3b.png)

![image](https://user-images.githubusercontent.com/33827177/147176515-00bbeb50-a5c4-44f0-8842-620f2cd57e6c.png)

![image](https://user-images.githubusercontent.com/33827177/147176768-6be76a3e-aa88-4f99-b384-dc62efd4f42b.png)

![image](https://user-images.githubusercontent.com/33827177/147176851-fe3205b5-39fa-41d8-9037-f5baf1553664.png)

### Simple Queue Service (SQS)

Provides managed message queues, fully managed, highly available.

Replication happens within a region by default.

- Standard : 
- FIFO queue : 

Messages up to 256KB in size. These should link to larger sets of data.

Polling is checking for any messages on the queue. When a client polls
and recieves messages, they are hidden due to **visibility timeout**.

This is the amount of time that a client can wait to work on the messages.

If a client recieves messages on the queue and finishes on that workload
it can delete the message. If the client doesn't delete the message, then
it will reappear on the queue. The queue will put the message back in
and make sure a different client can retry that workload.

**Dead-letter queue** if a message is recieved multiple times but is unable
to be finished, this puts it into a different workload to try and fix
the corruption.

ASG can scale and lambdas can be invoked based on queue length.

#### Highlights

Two types of queue

Standard - multi-lane HW
guarantee the order and at least once delivery.

FIFO - single lane road with no way to overtake
guarantee the order and at exactly once delivery
3,000 messages per second with batching or up to 300 messages second without

Billed on **requests** not messages. A request is a single request to SQS
One request can send 1 - 10 messages up to 64KB total.

Requests can return 0 messages. The more frequently you poll a SQS Queue,
the less effective it is.

Two ways to poll

- short (immediate) : uses 1 request and can return 0 or more messages. If the
queue is empty, it will return 0 and try again. This hurts queues that stay
short

- long (waitTimeSeconds) : it will wait for up to 20 seconds for messages
to arrive on the queue. It will sit and wait if none currently exist.

Messages can live on SQS Queue for up to 15 days. They offer KMS encryption
at rest.

Access is based on identity policies or a queue policy.

### Kinesis

This is a scalable streaming service. It is designed to inject data from
lots of devices or lots of applications.

Producers send data into a Kinesis Stream.

The stream can scale from low to near infinite data rates.

Highly available public service by design.

Streams store a 24-hour moving window of data. Can be increased to 7 days.
Data that is 24 hours and a second more is replaced by new data entering
the stream.

Kinesis includes the store within it for the amount of data
that can be ingested during a 24 hour period. However much you ingest during
24 hours, that's included.

Multiple consumers can access data from that moving window.
One might look at data points once per hour while another looks at data in
real time.

Each shard can have 1MB/s for ingestion and 2MB/s consumption.

**Kinesis data records (1MB)** are stored accross shards and are the blocks
of data for a stream.

**Kinesis Firehose** connects to a Kinesis stream. It can move the data
from a stream onto S3 or another service.

### SQS vs Kinesis

Is this about the ingestion of data or is it about the worker pools.

Large throughput or large numbers of devices, it is likely Kinesis.

SQS has 1 thing sending messages to the queue. One consumption group from
that tier.

Allow for async communications where the sender and reciever don't care
about what the other is doing. Once the message is processed, it is deleted.

Kinesis is desiged for huge scale ingestion with multiple consumers. Rolling
window for multiple consumers.

Designed for data ingestion, analytics, monitoring, app clicks.
