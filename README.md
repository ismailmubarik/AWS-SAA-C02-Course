# 1. SAA-C02 Notes

> These are my personal notes from Adrian Cantrill's (SAA-C02) course.Learning Aids from [aws-sa-associate-saac02](https://github.com/acantril/aws-sa-associate-saac02). There may be errors, so please purchase his course to get the original content and show support <https://learn.cantrill.io.>

**Table of Contents**

- [1.1. Cloud Computing Fundamentals](#11-cloud-computing-fundamentals)
- [1.2. AWS-Fundamentals](#12-aws-fundamentals)
- [1.3. IAM-Accounts-AWS-Organizations](#13-iam-accounts-aws-organizations)
- [1.4. Simple-Storage-Service-(S3)](#14-simple-storage-service-s3)
- [1.5. Virtual-Private-Cloud-VPC](#15-virtual-private-cloud-vpc)
- [1.6. Elastic-Cloud-Compute-EC2](#16-elastic-cloud-compute-ec2)
- [1.7. Containers-and-ECS](#17-containers-and-ecs)
- [1.8. Advanced-EC2](#18-advanced-ec2)
- [1.9. Route-53](#19-route-53)
- [1.10. Relational-Database-Service-RDS](#110-relational-database-service-rds)
- [1.11. Network-Storage-EFS](#111-network-storage-efs)
- [1.12. HA-and-Scaling](#112-ha-and-scaling)
- [1.13. Serverless-and-App-Services](#113-serverless-and-app-services)
- [1.14. CDN-and-Optimization](#114-cdn-and-optimization)
- [1.15. Advanced-VPC](#115-advanced-vpc)
- [1.16. Hybrid-and-Migration](#116-hybrid-and-migration)
- [1.17. Security-Deployment-Operations](#117-security-deployment-operations)
- [1.18. NoSQL-and-DynamoDB](#118-nosql-and-dynamodb)

---

## 1.1. Cloud Computing Fundamentals

Cloud computing provides

1. On-Demand Self-Service: Provision and terminate capabilities as needed using a UI/CLI without human interaction. Capabilties mean virtual machines, databases, etc.
2. Broad Network Access: Access services over any networks on any devices using
standard protocols and methods (http(s), remote desktop, vpn, etc..)
3. Resource Pooling (& Location Independence): Economies of scale, cheaper service. Resources are pooled to served multiple consumers using multi-tenant model...Furthermore, there is a sense of location independence...no contol or knowledge over the exact location of the resources.
4. Rapid Elasticity: Scale up and down automatically in response to system load. Secondly, to the consumer the capabilities or resources available for provisioning should appear to be unlimited
5. Measured Service: Usage can be monitored, controlled, reported and Billed. Pay only for what you consume. If its a truly cloud platform it shouldn't charge anything in advance.

### 1.1.1. Public vs Private vs Multi Cloud

- Public Cloud: using 1 public cloud such as AWS, Azure, Google Cloud.
- Private Cloud: using on-premises real cloud.  Examples include AWS Outpost, Azure Stack, Anthos. There is a difference between private cloud and having on premise infrastructure such as VmWare, HyperV or ZenServer. Private cloud must still meet 5 requirements but are just dedicated to a company.
- Multi-Cloud: using more than 1 public cloud in one deployment. For example, deploying part of your system/web-application on say AWS and other on GCP. Another example is using a third-party application that provides single management window or "single pane of glass". This is not advisable as it abstracts away from invidual environments relying on lowest common feature set. So you lose what makes each vendor unique.
- Hybrid Cloud: using public and private clouds in one environment generally from the same vendor
  - This is **NOT** using Public Cloud and Legacy on-premises hardware. 
  - Hybrid Networking or Hybrid Envirionment: Public Cloud + Traditional On-premise Networking Infrastructure

### 1.1.2. Cloud Service Models

The *Infrastructure Stack* or *Application Stack* contains multiple components
that make up the total service. There are parts that **you** manage as well
as portions the **vendor** manages. The portions the vendor manages and you
are charged for is the **unit of consumption**

1. On-Premises: The individual manages all components from data to facilities.
Provides the most flexibility, but also most IT intensive.
2. Data Center Hosting: Place equipment in a building managed by a vendor.
You pay for the facilities only. Similar to On-Premise except the facility is managed by the vendor
3. Infrastructure as a Service (IaaS): Vendor manages facilities and everything
else related to servers up to the OS. You pay per second or minute for the OS
used to the vendor. Lose some flexibility, but big risk reductions. e.g. AWS EC2
4. Platform as a Service (PaaS): Good for running an application only. The
unit of consumption is the runtime environment 9e.g. python runtime environment). You manage the application
and the data, but the vendor manges all else.
5. Software as a Service (SaaS): You consume the software as a service. This
can be Outlook or Netflix. There are almost no risks or additional costs, but
very little control.

There are additional services such as *Function as a Service*,
*Container as a Service*, and *DataBase as a Service* which be explained later.

---

## 1.2. AWS-Fundamentals

### AWS Support Plans

- Basic (free)
- Developer (one user, general guidance)
- Business (multiple users, personal guidance)
- Enterprise (Technical account manager)

### 1.2.1. Public vs Private Services


Refers to the networking only, not permissions.

- Public Internet: AWS is a public cloud platform and connected to the public
internet. It is not on the public internet, but is next to it.
- AWS Public Zone: Attached to the Public Internet.
S3 Bucket is hosted in the Public Zone, not all services are.
Just because you connect to a public service,
that does not mean you have permissions to access it.
- AWS Private Zone: No direct connectivity is allowed between the AWS Private
Zone and the public cloud unless this is configured for that service.
This is done by taking a part of the private service and projecting it into the
AWS public zone which allows public internet to make inbound or outbound
connections. Means we can allow certain public connections to our AWS private cloud. THis has to be done explicitly as by default the AWS private as by default the only things that connect to the private cloud are entities that are in the same private network or other newtorks with which private networking links are created such as other On-Premise networks within the private cloud.
Summary: AWS Public can be connected to over the public internet. AWS Public Services are not on the internet in fact they are hosted in a zone called AWS Public Cloud. THe private zone which is islated from the public internet and AWS public cloud can be sub divided using Virtual Private Clouds VPC. But private running private services can be allowed outoging connections or in some cases outside connections can be also made through IP projection into the AWS  Public cloud.

![image](https://user-images.githubusercontent.com/33827177/142623420-4888481a-6410-4e84-abdd-38f5fb25e935.png)



### 1.2.2. AWS Global Infrastructure

#### 1.2.2.1. Regions

AWS Region is an area of the world they have selected for a full deployment of
AWS infrastructure.

Areas such as countries or states

- Ohio
- California
- Singapore
- Beijing
- London
- Paris

AWS can only deploy regions as fast as their planning allows.
Regions are often not near their customers.

#### 1.2.2.2. AWS Edge Locations

Local distribution points. Useful for services such as Netflix so they can store
data closer to customers for low latency high speed transfers.

If a customer wants to access data stored in Brisbane, they will stream data
from the Sydney Region through an Edge Location hosted in Brisbane.

#### 1.2.2.3. AWS Management

Regions are connected together with high speed networking.
Some services such as EC2 need to be selected in a region.
Some services are global such as IAM

#### 1.2.2.4. Region's 3 Benefits

- Geographical Separation
  - Useful for natural disasters
  - Provide isolated fault domain
  - Regions are 100% isolated
- Geopolitical Separation
  - Different laws change how things are accessed
  - Stability from political events
- Location Control
  - Tune architecture for performance
  - Duplicate infrastructure at closer points to customers

### 1.2.3. Regions and AZs

Region Name: Asia Pacific (Sydney)
Region Code: ap-southeast-2

AWS will provide between 2 and 6 AZs per region.
AZs are isolated compute, storage, networking, power, and facilities.
Components are allowed to distribute load and resilience by using multiple zones.
Availbility Zones doesnt mean a data center. An AZ can be a single data center or be part of multiple DCs. AWS does not provide that visibility of what an AZ is. Just that it is isolated from other AZs and AZs are connected to each other with high speed redundant networks.

![image](https://user-images.githubusercontent.com/33827177/142625055-16488b4b-4c88-4598-8544-2c1234af6961.png)

Service can be placed across multiple availability zones to make them reselient. For example, VPC or Virtual Private Network which is a way to create a Private network. It can be placed across multiple AZs to provide reselience.

#### 1.2.3.1. Service Resilience
Services can be placed multiple availibility zones to make them reselient. 
1. Globally Resilient: IAM or Route 53. No way for them to go down. Data is
replicated throughout multiple regions.
2. Region Resilient: Operate as separate services in each region. Generally
replicate data to multiple AZs in that region meaning if an AZ fails a service can continue to operate in that region but if the whole region as a whole fails then the service cannot operate.
3. AZ Resilient: Run from a single AZ. If an AZ fails then a service at that AZ may fail. It is possible for hardware to fail in an
AZ and the service to keep running because of redundant equipment, but should not be relied on.

### 1.2.4. AWS Default VPC

VPC is a virtual network inside of AWS.
A VPC is within 1 account and 1 region which makes it regionally resilient. It runs from multiple availibility zones within within a region.
A VPC is private and isolated until decided otherwise. Services deployed in a VPC can communicate with one another but they are isolated from other VPCs and from the AWS public zone and the public internet unless otherwise confugred. 

We can have only One default VPC per region. But can have many custom VPCs which are all private
by default. Default VPC comes preconfigured in a very specific way. A custom VPC needs to be configured end to end and are 100% by default. They can be configured in anyways as long as we stay within the rules of VPC. Custom VPCs are flexibile then Deafault VPCs

#### 1.2.4.1. Default VPC Facts

VPC CIDR - defines start and end ranges of the VPC.
IP CIDR of a default VPC is always: **172.31.0.0/16**

A region has multiple AZs and each of those is an independent pool of infrastructure. Since a VPC is regionally  reselient and the way it is done is by assigning each subnet to an AZ. This assignment of a subnet is set on creation and can never be changed. 

The default VPC is always configured to have one subnet in each AZ in the region by default. If an AZ fails that subnet fails but the other subnets in other AZs continue to operate.

Subnets are given one section of the IP ranges for the default service. 
They are configured to provide anything that is deployed inside those subnets with public IPv4 addresses. 

![image](https://user-images.githubusercontent.com/33827177/142739826-b2e13394-143b-42bb-bb88-e1d440684316.png)

In general do not use the Default VPC in a region because it is not flexible.

Default VPC is large because it uses the /16 range.
A subnet /20 is smaller because the higher the / number is, the smaller the grouping. A/17 is half the size of a /16 and that is why 2 /17 can fit in a /16.

Two /17's will fit into a /16, sixteen /20 subnets can fit into one /16.

### 1.2.5. Elastic Compute Cloud (EC2)

Default compute service. Provides access to virtual machines called instances.

#### 1.2.5.1. Infrastructure as as Service (IaaS)

The unit of consumption is an instance.
An EC2 instance is configured to launch into a single VPC subnet.
Private service by default, public access must be configured.
The VPC needs to support public access. If you use a custom VPC then you must
handle the networking on your own.

EC2 deploys into one AZ meaning its only Az resilient. Thus, if it fails, the instance fails.

Different sizes and capabilities. All use On-Demand Billing - Per second.
Only pay for what you consume.

Local on-host storage or **Elastic Block Storage** which network storage made available to the EC2 instance.

Pricing based on:

- CPU
- Memory - Super fast memory to store data that is currently being worked on like RAM I guess?
- Storage - Local on-host storage or Elastic Block Storage (EBS) which is a network storage made available to the EC2 instance. It is for medium term data?
- Networking
Extra cost for any commercial software the instance deploys with.
EC2 has an attribute called state
#### 1.2.5.2. Running State

Charged for all four categories.

- Running on a physical host using CPU.
- Using memory even with no processing.
- OS and its data are stored on disk, which is allocated to you.
- Networking is always ready to transfer information.

#### 1.2.5.3. Stopped State

Charged for EBS storage  only.

- No CPU resources are being consumed
- No memory is being used
- Networking is not running
- Storage is allocated to the instance for the OS together with any applications.

#### 1.2.5.4. Terminated State

No charges, deletes the disk and prevents all future charges.

![image](https://user-images.githubusercontent.com/33827177/142740616-a070bb50-5ad2-46ae-a568-d6d13b4bab36.png)


#### 1.2.5.5. AMI (Server Image)

AMI can be used to create an instance or can be created from an instance.
AMIs in one region are not available from other regions.

Contains:

- Permissions: controls which accounts can and can't use the AMI and can be set to the following:

  - Public - Anyone can launch it.

  - Owner - Implicit allow, only the owner can use it to spin up new instances

  - Explicit - Owner grants access to AMI for specific AWS accounts

- Root Volume: contains the **Boot Volume** i.e. the C drive in windows or the root volume in Linux

- Block Device Mapping: links the volumes that the AMI has and
how they're presented to the operating system. Determines which volume is a
boot volume and which volume is a data volume.


#### 1.2.5.6. Connecting to EC2

AMI Types:

- Amazon Quick Start AMIs
- AWS Marketplace AMIs
- Community AMIs
- Private AMIs

- Windows using RDP (Remote Desktop Protocol), Port 3389
- Linux SSH protocol, Port 22

Login to the instance using an SSH key pair.
Private Key - Stored on local machine to initiate connection.
Public Key - AWS places this key on the instance.

Configurring Security Group: Think of them like mini firewalls. They are attached to the network interfaces of anything that goes inside of VPC. A security group can be connected to 1 EC2 instance, 2 or more Ec2 instances or zero instacnes

### 1.2.6. S3 (Default Storage Service)

Global Storage platform. Runs from all regions and is a public service.
Can be accessed anywhere from the internet with an unlimited amount of users. Offers unlimited data.

S3 service is regional based service because data never leaves that region unless otherwise configured to. S3 is regionally resilient because data is replicated across AZs. Meaning failure in an AZ can be tolerated because data will remain to be available.

Economical and ideal for Movies, Audio, Photos, Text, Large Data Sets.

Can be accessed via UI/CLI/API/HTTP. 
This should be the default storage platform. S3 delivers two main things: Objects & Buckets.

S3 is an object storage, not file, or block storage.
You can't mount an S3 Bucket.

#### 1.2.6.1. Objects

Can be thought of a file. e.g movies, Big data, etc. Two main components:

- Object Key: File name in a bucket
- Value: Data or contents of the object
  - Zero bytes to 5 TB

Other components:

- Version ID
- Metadata
- Access Control
- Sub resources

#### 1.2.6.2. Buckets

Buckets are containers for objects.

- Created in a specific AWS Region.
- Data has a primary home region. Will not leave this region unless told.
- Blast Radius = Region. That is if a disastor/emergency occurs due to which the service goes down or say data is corrupted the effect of that will be limited to the region.
- Unlimited number of Objects
- Name is globally unique. e.g. name of bucket is koaladata means we can only use that name if nobody else is using it already.
- All objects are stored within the bucket at the root level. Meaning it has flat structure. Meaning there are no folders within folders with file like a File System. But if we do listing on S3 bucket we will see what we think as Folders even though UI presents it like that but its not that case. For example, /old/Koala1.jpg, /old/koala2.jpg are basically named like that and as a result AWS presents it like the koala1 or koala2 is placed in a old folder. FOlders are actually referred to as prefixes in AWS S3 as they are part of the object name.


If the objects name starts with a slash such as `/old/Koala1.jpg` the UI will
present this as a folder. In actuality this is not true, there are no folders.

#### Exam Power Up:
Bucket names are globally unique.
3-63 characters, all lower case no underscores
Start with a lowercase letter or a number
Can't be IP formatted e.g. 1.1.1.1
Buckets- 100 soft limit in an AWS account. Not per region across all entire AWS account. Can be requested to be increased to 1000 but not more. So 100 is a soft limit and  1000 is a Hard Limit
Unlimited objects in bucket, 0 bytes to 5TB
Key= Name, Value= Data
S3 is an object store - not file or block based storage. 
a. If you want to access the whole of something say an audio file, then its a candidate for S3 storage.
b. Likewise its not block storage. Meaning you can't mount and S3 bucket as K:\ or /images. Block storages (virtual hardrives) are what you mount to VMs. in EC2 its the EBS that you mount to VMs. Block storage is limited to only one entiy accessing it a time. Only one instance can access a block storage in EBS. Where is S3 which is an object storage can be accessed by  multiple entities.
S3 is great for offloading for example, blog with huge vidoes pictures instead of storing it on expensive compute instance you can configure your blog to point users to S3 directly.

We should use AWS S3 for INPUT and/or OUTPUT to many AWS products.

### 1.2.7. CloudFormation Basics

CloudFormation templates can be used to create, update, modify, and delete infrastructure.

They can be written in YAML or JSON. An example is provided below.

```YAML
## This is not mandatory. The AWSTemplateFormatVersion section (optional) identifies the capabilities 
## of the template. If you don't specify it, CloudFormation will use the latest version
AWSTemplateFormatVersion: "version date"

## Give details as to what this template does.
## If you are using bot the Description & AWSTemplateFormatVersion then the Description section, 
## MUST immediately follow the AWSTemplateFormatVersion.
Description:
  A sample template

## It can control how the different things in CloudFormation template are  presented through the
## the AWS console. You can specify grouping, control order, add descriptions ad labels. Its a way
## you can force the UI presents the template.The bigger your template, the more likely
## this section is needed. USed for other things as well.
Metadata:
  template metadata

## Prompt the user for more data. Name of something, size of instance,
## data validation
Parameters:
  set of parameters

## Another optional section. Allows lookup tables, not used often
Mappings:
  set of mappings

## Decision making in the template. Things will only occur if a condition is met.
## Step 1: create condition
## Step 2: use the condition to do something else in the template
Conditions:
  set of conditions

Transform:
  set of transforms

## The only mandatory field of this section
Resources:
  set of resources

## Once the template is finished it can return data or information.
## Could return the instance ID or setup address of a word press blog.
Outputs:
  set of outputs
```

### 1.2.8. Resources

An example which creates an EC2 instance

```YAML
Resources:
  Instance: ## Logical Resource
    Type: 'AWS::EC2::Instance' ## This is what will be created
    Properties: ## Configure the resources in a particular way
      ImageId: !Ref LatestAmiId
      Instance Type: !Ref Instance Type
      KeyName: !Ref Keyname
```

Resources inside a CloudFormation template are called logical resources in this case instance.
Once a template is created, AWS will make a stack. This is a living and active
representation of a template. One template can create infinite amount of stacks.

For any **Logical Resources** in the stack,
CF will make a corresponding **Physical Resources** in your AWS account.

![image](https://user-images.githubusercontent.com/33827177/142791120-dda4104b-ba5f-4892-afe3-3516c6d030f6.png)


It is cloud formations job to keep the logical and physical resources in sync.

A template can be updated and then used to update the same stack.

![image](https://user-images.githubusercontent.com/33827177/142791140-219c4b5d-58c2-4e17-beda-ff58754c9cd5.png)


### 1.2.9. CloudWatch Basics

CloudWatch is a support service used by almost all AWS services. It collects and manages operational data on your behalf. 

Three products in one

- Metrics: data relating to AWS products, apps, on-prem solutions
- CloudWatch Logs: AWS Products, Apps, on-premises
- CloudWatch Events: event hub
  - If an AWS service does something, CW events can perform another action
  - Generate an event to do something at a certain time of day or time of week.

![image](https://user-images.githubusercontent.com/33827177/142957124-66cb35f1-4150-4335-9109-b1f4d9920b5a.png)

#### 1.2.9.1. Namespace

Container for monitoring data.
Naming can be anything so long as it's not `AWS/service` such as `AWS/EC2`.
This is used for all metric data of that service

#### 1.2.9.2. Metric

Time ordered set of data points such as:

- CPU Usage
- Network IN/OUT
- Disk IO

![image](https://user-images.githubusercontent.com/33827177/142957240-8ba87758-18a4-4e1e-99ac-1577f002f20d.png)

All the above three data points are gathered natively by default

CloudWatch is a public service and it can be used inside AWS, within on-premises envirionment and even in other cloud platforms (the last 2 can be done through Cloud Watch agent). Meaning it is not for a specific server. This could get things from different servers. 

Anytime CPU Utilization is reported, the **Datapoint** will report:

- Timestamp = 2019-12-03
- Value = 98.3

**Dimensions** could be used to get metrics for a specific instance or type of instance, among others. They separate data points for different **things** or
**perspectives** within the same metric.

![image](https://user-images.githubusercontent.com/33827177/142957459-4bb576a1-920d-4761-92b0-08f328cfadce.png)

#### 1.2.9.3. Alarms

Has two states `ok` or `alarm`. A notification could be sent to an SNS topic or an action could be performed based on an alarm state. For example, Billing Alarm.
Third state can be insufficient data state. Not a problem, just wait.

### 1.2.10. Shared Responsibility Model

AWS: Responsible for security **OF** the cloud

Customer: Responsible for security **IN** the cloud

![image](https://user-images.githubusercontent.com/33827177/142747146-b875c1a6-76a4-45ff-a9f2-e59de64fa26f.png)


### 1.2.11. High Availability (HA), Fault-Tolerance (FT) and Disaster Recovery (DR)

#### 1.2.11.1. High Availability (HA)

- Aims to **ensure** an agreed level of operational **performance**, usually
**uptime**, for a **higher than normal period**
- Instead of diagnosing the issue, if you have a process ready to replace it, it can be fixed quickly and probably in an automated way.
- Spare infrastructure ready to switch customers over to in the event of a disaster to minimize downtime
- User disruption is not ideal, but is allowed
  - The user might have a small disruption or might need to log back in.
- Maximizing a system's uptime
  - 99.9% (Three 9's) = 8.7 hours downtime per year.
  - 99.999 (Five 9's) = 5.26 minutes downtime per year.

#### 1.2.11.2. Fault-Tolerance (FT)

- System can **continue operating properly**
in the event of the **failure of some** (one or more faults within) of its
**components**
- Fault tolerance is much more complicated than high availability and more
expensive. Outages must be minimized and the system needs levels of
redundancy.
- An airplane is an example of system that needs Fault Tolerance. It has
more engines than it needs so it can operate through failure.

Example:
A patient is waiting for a life saving surgery and is under anesthetic.
While being monitored, the life support system is dosing medicine.
This type of system cannot only be highly available, even a movement of
interruption is deadly.

Most people think that HA means operating through faiure. HA is just about maximizing up-time. Fault-tolerance is whats means operating through failure. HA can be accomplished by having spare equipments and having automated processes that would kick in in the event of failure. With Fault Tolerance you need to not only minimize outages just like HA but also design the system in a way so that it can tolerate failure. This means for a system to be faul tolerant it should not only have redundancy but also designed in a such away where it can route traffic/processes around failed components. 

![image](https://user-images.githubusercontent.com/33827177/142962386-5d0e2b55-c4d3-4387-94ad-22f6ab23ca32.png)

#### 1.2.11.3. Disaster Recovery (DR)

- Set of policies, tools and procedures to **enable the recovery** or
**continuation** of **vital** technology infrastructure and systems
**following a natural or human-induced disaster**.
- DR can largely be automated to eliminate the time for recovery and errors.

This involves:

- Pre-planning
  - Ensure plans are in place for extra hardware
  - Do not store backups at the same site as the system
- DR Processes
  - Cloud machines ready when needed

This is designed to keep the crucial and non replaceable parts of the
system in place.

Used when HA and FT don't work.

![image](https://user-images.githubusercontent.com/33827177/142962748-9f31a125-7843-4929-bc73-3255e85c7d9e.png)

### 1.2.12. Domain Name System (DNS)

DNS is a discovery service. Translates machines into humans and vice-versa.
It is a huge database which is globally distributed.

The conversion from www.amazon.com to 104.98.34.131 happens when your laptiop/device directly communicates with the DNS server OR it talks to a 
DNS resolver server (which is either running on your router or your ISP) to find and convert the Domain name to an IP address. That piece of information is called a zone and the way it is stored is called zone file. The Zone File is stored on a Name Server. So we query the Zone File within the Name Server (NS) and get IP address for www.amazon.com
i.e. 104.98.34.131. our laptop can communicate with the Amazon Web Server. 
The Zone file can be located anywhere on 1 or 2 out of potentially millions of Name Servers i.e. NS

![image](https://user-images.githubusercontent.com/33827177/148627476-6347a9f5-e2a7-43a7-bc2b-b3db22e91052.png)

Parts of the DNS system

- DNS Client: Piece of software running on the OS for a device you're using.
- Resolver: Software on your device or server which queries DNS on your behalf.
- Zone: A part of the DNS database.
  - This would be amazon.com
  - What the data is, its substance
- Zone file: physical database for a zone
  - How physically that data is stored
- Nameserver: where zone files are hosted

Steps:

Find the Nameserver which hosts a particular zone file.
Query that Nameserver for a record that is in that zone file.
It then passes the information back to the DNS client.

#### 1.2.12.1. DNS Root

The starting point of DNS.
DNS names are read right to left with multiple parts separated by periods.

`www.netflix.com.`

The last period is assumed to be there in a browser when it's not present.
The DNS Root is hosted on DNS Root Servers (13). These are hosted
by 12 major companies.

![image](https://user-images.githubusercontent.com/33827177/142965067-a27d04a4-2668-4239-91e5-4b696c5f5f42.png)

**Root Hints** is a pointer to the DNS Root servers provided by the OS vendor

Process

1. DNS client asks DNS Resolver for IP address of a given DNS name.
2. Using the Root Hints file, the DNS Resolver communicates with one or
more of the root servers to access the root zone and begin the process
of finding the IP address.

The Root Zone is organized by IANA (Internet Assigned Numbers Authority).
Their job is to manage the contents of the root zone. IANA is in charge
of the DNS system because they control the root zone.

![image](https://user-images.githubusercontent.com/33827177/142965272-c83fec5f-962c-43cf-954b-806986b8603a.png)


#### 1.2.12.2. DNS Hierarchy

Assuming a laptop is querying DNS directly for www.amazon.com and using
a root hints file to know how to access a root server and query the root zone.

- When something is trusted in DNS, it is an **authority**.
- One piece can be authoritative for root.
- One piece can be authoritative for amazon.com
- The root zone is the start and the only thing trusted in DNS.
- The root zone can delegate a part of itself to another zone or entity.
- That someone else then becomes authoritative for just the part that's delegated.
- The root zone is just a database of the top level domains.

The top level domains are the only thing immediately to the left of the root in a DNS name.

- `.com` or `.org` are generic top level domains (gTLD)
- `.uk` is a country code top level domain (ccTLD)

![image](https://user-images.githubusercontent.com/33827177/142965566-3949bf41-ac6c-41e5-b545-fd4b8a225e3b.png)

IANA doesn't run anything in the DNS apart from the root zone. They delegate management to other organizations known as registries for example, .com is delegated to 'VeriSign'

![image](https://user-images.githubusercontent.com/33827177/142965935-af43b0aa-4062-4c78-9797-78ef02da3fdb.png)

![image](https://user-images.githubusercontent.com/33827177/142966179-254693f5-d43e-4e40-aa44-9dbf491e2f5b.png)

**Registry** maintains the zones for a TLD (e.g .ORG)
**Registrar** has relationships with the .org TLD zone manager
allowing domain registration

### 1.2.13. Route53 Fundamentals
Route53 is AWS's managed product. It allows you to:
- Registers domains
- Can host zone files on managed nameservers
- This is a global service (so single database), no need to pick a region
- Globally Resilience because its distributed across regions
  - Can operate with failure in one or more regions

#### 1.2.13.1. Register Domains

Has relationships with all major registries (registrar)

- Route 53 will check with the top level domain to see if the name is available
- Route 53 creates a zone file for the domain to be registered
- Allocates nameservers for that zone
  - Generally four of these for one individual zone
  - This is a hosted zone
  - The zone file will be put on these four managed nameservers
- Route 53 will communicate with the `.org` registry and add the nameserver records 
into the zone file for that top level domain.
  - This is done with a nameserver record (NS).

#### 1.2.13.2. Route53 Details

**Zone files** in AWS
Hosted on four managed name servers

- Can be **public** or **private** (linked to one or more VPCs)

![image](https://user-images.githubusercontent.com/33827177/142967387-f94d44ea-be56-4ccb-983c-5cc31e197b7c.png)

### 1.2.14. DNS Record

![image](https://user-images.githubusercontent.com/33827177/142968897-6675f3ad-6b72-48d1-8182-53b3bedef8ad.png)

- Nameserver (NS): Allows delegation to occur in the DNS.
- A and AAAA Records: Maps the host to a v4 or v6 host type respectively. Most of the time
you will make both types of record, A and AAAA.
- CNAME (CanonicalName) Record Type: Allows DNS shortcuts to reduce admin overhead because if the IP of the server changes, we only need to update the A record...
CNAMES cannot point directly to an IP address, only another name.

![image](https://user-images.githubusercontent.com/33827177/142969221-bf44492e-0e46-4e2f-8266-0be75cbf316a.png)

- MX records: How emails are sent. They have two main parts:
  - Priority: Lower values for the priority field are higher priority.
  - Value
    - If it is just a host, it will not have a dot on the right. It is assumed
to be part of the same zone as the host.
    - If you include a dot on the right, it is a ***fully qualified domain name***
- TXT Record: Allows you to add arbitrary text to a domain.
One common usage is to prove domain ownership.
![image](https://user-images.githubusercontent.com/33827177/142969497-c8510e05-81b8-4bd8-882c-4f8d434d3534.png)

#### 1.2.14.1. TTL - Time To Live

This is a numeric setting on DNS records in seconds.
Allows the admin to specify how long the query can be stored
at the resolver server.
If you need to upgrade the records, it is smart to lower the TTL value first or leave them permanently low.
The TTL has to always obey the TTL values but that configuration can be changed locally by the admin of the resolver server

![image](https://user-images.githubusercontent.com/33827177/142969693-686c991f-bffc-444f-a29e-73169e0f032e.png)

Getting the answer from an Authoritative Source is known as an
**Authoritative Answer**.

If another client queries the same thing, they will get back a
**Non-Authoritative** response.

---

## 1.3. IAM-Accounts-AWS-Organizations

### 1.3.1. IAM Identity Policies

Identity Policies are attached to AWS Identities which are
IAM users, IAM groups, and IAM roles. These are a set of security statements
that ALLOW or DENY access to AWS resources.

When an identity attempts to access AWS resources, that identity needs
to prove who it is to AWS, a process known as **Authentication**.
Once authenticated, that identity is known as an **authenticated identity**

#### 1.3.1.1. Statement Components

- Statement ID (SID): Optional field that should help describe
  - The resource you're interacting
  - The actions you're trying to perform
- Effect: is either `allow` or `deny`.
  - It is possible to be allowed and denied at the same time
- Action are formatted `service:operation`. There are three options:
  - specific individual action
  - wildcard as an action
  - list of multiple independent actions
- Resource: similar to action except for format `arn:aws:s3:::catgifs`

#### 1.3.1.2. Priority Level

- Explicit Deny: Denies access to a particular resource cannot be overruled.
- Explicit Allow: Allows access so long there is not an explicit deny.
- Default Deny (Implicit): IAM identities start off with no resource access.
![image](https://user-images.githubusercontent.com/33827177/143322732-2112c2ab-0dc4-433f-b4e9-26768edd38af.png)

#### 1.3.1.3. Inline Policies and Managed Policies

- Inline Policy: grants access and assigned on each accounts individually. This is done by assigning a policy to say each user invidually through a JSON file. The problem is that you would have to make changes to each manually if you want to say make changes to identical roles and their policies. Shpould be only used when want to assign Special or Exceptional Allow or Deny to an invidual identity
- Managed Policy (best practice): one policy is applied to all users at once. Reusable --> can be assigned to multiple users and Low Management Overhead

There are two main types of Managed Policies. AWS Managed policy which are created and managed policies. And Customer Managed policies tweaked to the exact need of the customer.
### 1.3.2. IAM Users

Identity used for anything requiring **long-term** AWS access

- Humans
- Applications
- Service Accounts

If you can name a thing to use the AWS account, this is an IAM user.

When a **principal** wants to **request** to perform an action,
it will **authenticate** against an identity within IAM. An IAM user is an
identity which can be used in this way.

There are two ways to authenticate:

- Username and Password
- Access Keys (CLI) --> used both by humans and applications

Authentication: Once the **Principal** has authenticated, it becomes an **authenticated identity** 
Authorization: Once authenticated, AWS knows which policy applies to that identity. AWS authorizes or denies access an identity by checking the policy that applies to that identity.

![image](https://user-images.githubusercontent.com/33827177/143327442-80a5f35b-64dc-4afb-9f26-64f783c3d334.png)

#### 1.3.2.1. Amazon Resource Name (ARN)

Uniquely identify resources within any AWS accounts.

This allows you to refer to a single or group of resources.
This prevents individual resources from the same account but in
different regions from being confused as ARN can identify them individually...

ARN generally follows the same format:

```bash
arn:partition:service:region:account-id:resource-id
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id
```

- partition: almost always `aws` unless it is china `aws-cn`
- region: can be a double colon (::) if that doesn't matter
- account-id: the account that owns the resource
  - EC2 needs this
  - S3 does not need account-id because its globally unique
- resource-type/id: changes based on the resource

An example that leads to confusion:

- arn:aws:s3:::catgifs
  - This references an actual bucket
- arn:aws:s3:::catgifs/*
  - This refers to objects in that bucket, but not the bucket itself.

These two ARNs do not overlap.

#### 1.3.2.2. IAM FACTS

- 5,000 IAM users per account
- IAM user can be a member of a maximum 10 groups
- ![image](https://user-images.githubusercontent.com/33827177/143328429-25b38b11-0bb1-4e0d-b19d-7a1dd9aef452.png)

### 1.3.3. IAM Groups

Containers for users. **You cannot login to IAM groups** They have no
credentials of their own. Used solely for management of IAM users.

Groups bring two benefits

1. Effective administrative style management of users based on the team
2. Groups can have Inline and Managed policies attached.

AWS merges all of the policies from all groups the user is in together.

- The 5000 IAM user limit applies to groups.
- There is **no all users** IAM group.
  - You can create a group and add all users into that group, but it needs to be
created and managed on your own.
- No Nesting: You cannot have groups within groups.
- 300 Group Limit per account. This can be fixed with a support ticket.

**Resource Policy** A bucket can have a policy associated with that bucket.
It does so by referencing the identity using an ARN (Amazon Reference Name).
A policy on a resource can reference IAM users and IAM roles by the ARN.
A bucket can give access to one or more users or one or more roles.

**GROUPS ARE NOT A TRUE IDENTITY**
**THEY CAN'T BE REFERENCED AS A PRINCIPAL IN A POLICY**

An S3 Resource cannot grant access to a group, it is not an identity.
Groups are used to allow permissions to be assigned to IAM users.

### 1.3.4. IAM Roles

A single thing that uses an identity is an IAM User.

IAM Roles are also identities that are used by large groups of individuals.
If have more than 5000 principals, it could be a candidate for an IAM Role.
![image](https://user-images.githubusercontent.com/33827177/143505351-dc777af3-6415-4a81-ac1a-b47c765f5818.png)

IAM Roles are **assumed** you become that role.

This can be used short term by other identities.

IAM Users can have inline or managed policies which control which permissions
the identity gets within AWS

Policies which grant, allow or deny, permissions based on their associations.

IAM Roles have two types of roles can be attached.

- Trust Policy: Specifies which identities are allowed to assume the role.
- Permissions Policy: Specifies what the role is allowed to do.

Trust policy can reference different things. It can reference identities in the same account i.e other IAM users, other roles and even other AWs services such as EC2. The trust policy can even reference identities in other AWS accounts. It can allow anonymous usage of that account...

If an identity is allowed on the **Trust Policy**, it is given a set
of **Temporary Security Credentials**. Similar to access keys except they
are time limited to expire. The identity will need to renew them by
reassuming the role.

Every time the **Temporary Security Credentials** are used, the access
is checked against the **Permissions Policy**. If you change the policy, the
permissions of the temp credentials also change.

![image](https://user-images.githubusercontent.com/33827177/143505786-b37bc4dc-e08f-483c-96bd-80386de08d96.png)

Roles are real identities and can be referenced within resource policies.

Secure Token Service (sts:AssumeRole) this is what generates the temporary
security credentials (TSC).

### 1.3.5. When to use IAM Roles
Lambda functions as with most AWS things has no permission by default...
Instead of hard coding access key to the Lambda function. We can create a role Lambda Execution role.
Lambda Execution Role.
For a given lambda function, you cannot determine the number of principals
which suggested a Role might be the ideal identity to use.

- Trust Policy: to trust the Lambda Service
- Permission Policy: to grant access to AWS services.

When this is run, it uses the sts:AssumeRole to security credentials to access AWS resources. For example, when the code is
running in run time environment, the run time environment assumes the roles and then the run time environment cana ccess say
S3, etc.
![image](https://user-images.githubusercontent.com/33827177/143506212-c037d6e3-7d19-4dee-b3d1-9d557cabb29b.png)

It is better when possible to use an IAM Role versus attaching a policy.

#### 1.3.5.1. Emergency or out of the usual situations

Break Glass Situation - There is a key for something the team does not
normally have access to. When you break the glass, you must have a reason
to do.
A role can have an Emergency Role which will allow further access if
its really needed.
![image](https://user-images.githubusercontent.com/33827177/143506720-8efc56ef-2740-44f7-a1f6-ccc75fc7c8b4.png)

#### 1.3.5.2. Adding AWS into existing corp environment

You may have an existing identity provider you are trying to allow access to.
This may offer SSO (Single Sign On) or over 5000 identities.
This is useful to reuse your existing identities for AWS.
External accounts can't be used to access AWS directly.
To solve this, you allow an IAM role in the AWS account to be assumed
by one of the active directories.
**ID Federation** allowing an external service the ability to assume a role.
![image](https://user-images.githubusercontent.com/33827177/143506794-3c652927-2866-4609-8d64-f34da2254a9a.png)

#### 1.3.5.3. Making an app with 1,000,000 users

**Web Identity Federation** uses IAM roles to allow broader access.
These allow you to use an existing web identity such as google, facebook, or
twitter to grant access to the app.
We can trust these web identities and allow those identities to assume
an IAM role to access web resources such as DynamoDB.
No AWS Credentials are stored on the application.
Can scale quickly and beyond.
![image](https://user-images.githubusercontent.com/33827177/143506890-8c2cbf5b-a1ca-434f-bf81-fa08282a71db.png)

#### 1.3.5.4. Cross Account Access

You can use a role in the partner account and use that to upload objects
to AWS resources.
![image](https://user-images.githubusercontent.com/33827177/143506939-9055bc0d-3ab6-43f7-a8b9-b3edc5017423.png)

### 1.3.6. AWS Organizations

Without an organization, each AWS account needs it's own set of IAM users
as well as individual payment methods.
If you have more than 5 to 10 accounts, you would want to use an org.

Take a single AWS account **standard AWS account** and create an org.
The standard AWS account then becomes the **master account**.
The master account can invite other existing standard AWS accounts. They will
need to approve their joining to the org.

When standard AWS accounts become part of the org, they
become **member accounts**.
Organizations can only have one **master accounts** and zero or more
**member accounts**

#### 1.3.6.1. Organization Root

This is a container that can hold AWS member accounts or the master account.
It could also contain **organizational units** which can contain other
units or member accounts.

#### 1.3.6.2. Consolidated billing

The individual billing for the member accounts is removed and they pass their
billing to the master account.
Inside an AWS organization, you get a single monthly bill for the master
account which covers all the billing for each users.
Can offer a discount with consolidation of reservations and volume discounts

#### 1.3.6.3. Create new accounts in an org

Adding accounts in an organization is easy with only an email needed.
You no longer need IAM users in each accounts. You can use IAM roles
to change these.
It is best to have a single AWS account only used for login.
Some enterprises may use an AWS account while smaller ones may use the master.

#### 1.3.6.4. Role Switching

Allows you to switch between accounts from the command line

### 1.3.7. Service Control Policies

Can be used to restrict what member accounts in an org can do.

JSON policy document that can be attached:

- To the org as a whole by attaching to the root container.
- A specific Organizational Unit
- A specific member only.

The master account cannot be restricted by SCPs which means this
should not be used because it is a security risk.

SCPs limit what the account, **including root** can do inside that account.
They don't grant permissions themselves, just act as a barrier.

#### 1.3.7.1. Allow List vs Deny List

Deny list is the default.

When you enable SCP on your org, AWS applies `FullAWSAccess`. This means
SCPs have no effect because nothing is restricted. It has zero influence
by themselves.

```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": "*",
    "Resource": "*"
  }
}
```

SCPs by themselves don't grant permissions. When SCPs are enabled,
there is an implicit deny.

You must then add any services you want to Deny such as `DenyS3`

```json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Deny",
    "Action": "s3:*",
    "Resource": "*"
  }
}
```

**Deny List** is a good default because it allows for the use of growing
services offered by AWS. A lot less admin overhead.

**Allow List** allows you to be conscience of your costs.

- To begin, you must remove the `FullAWSAccess` list
- Then, specify which services need to be allowed access.
- Example `AllowS3EC2` is below

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
        "Effect": "Allow",
        "Action": [
            "s3:*",
            "ec2:*"
        ],
    "Resource": "*"
    }
  ]
}
```

### 1.3.8. CloudWatch Logs

This is a public service, this can be used from AWS VPC or on premise
environment.

This allows to **store**, **monitor** and **access** logging data.

- This is a piece of information data and a timestamp
- Can be more fields, but at least these two

Comes with some builtin Integrations with AWS servies like EC2, VPC Flow Logs, Lambda, CloudTrail, R53 and more...
Security is provided with IAM roles or Service roles. For services outside AWS to log data in CloudWatch Logs we will need unified CloudWatch agent.
Can generate metrics based on logs **metric filter**
![image](https://user-images.githubusercontent.com/33827177/143789983-e8634073-f59a-4380-abd5-90982dd07f61.png)

#### 1.3.8.1. Architecture of CloudWatch Logs

It is a regional service `us-east-1`

Need logging sources such as external APIs or databases. This sends
information as **log events**. These are stored in **log streams**. This is a
sequence of log events from the same source.

**Log Groups** are containers for multiple logs streams of the same
type of logging. This also stores configuration settings such as
retention settings and permissions.

Once the settings are defined on a log group, they apply to all log streams
in that log group. Metric filters are also applied on the log groups.

### 1.3.9. CloudTrail Essentials

Concerned with who did what.

Logs API calls or activities as **CloudTrail Event**

Stores the last 90 days of events in the **Event History**. This is enabled
by default and is no additional cost.

To customize the service you need to create a new **trail**.
Two types of events. Default only logs Management Events

- Management Events:
Provide information about management operations performed on resources
in the AWS account. Create an EC2 instance or terminating one.

- Data Events:
Objects being uploaded to S3 or a Lambda function being invoked. This is not
enabled by default and must be enabled for that trail.

#### 1.3.9.1. CloudTrail Trail

Logs events for the AWS region it is created in. It is a regional service.

Once created, it can operate in two ways

- One region trail
- All region trail
  - Collection of trails in all regions
  - When new regions are added, they will be added to this trail automatically

Most services log events in the region they occur. The trail then must be
a one region trail in that region or an all region trail to log that event.

A small number of services log events globally to one region. Global services
such as IAM or STS or CloudFront always log their events to `us-east-1`

A trail must have this enabled to have this logged.

AWS services are largely split into regional services or global services.

When the services log, they log in the region they are created or
to `us-east-1` if they are a global service.

A trail can store events in an S3 bucket as a compressed JSON file. It can
also use CloudWatch Logs to output the data.

CloudTrail products can create an organizational trail. This allows a single
management point for all the APIs and management events for that org.

#### 1.3.9.2. CloudTrail Exam PowerUp

- It is enabled by default for 90 days without S3
- Trails are how you configure S3 and CWLogs
- Management events are only saved by default
- IAM, STS, CloudFront are Global Service events and log to `us-east-1`
  - Trail must be enabled to do this
- NOT REALTIME - There is a delay. Approximately 15 minute delay

#### 1.3.9.3. CloudTrail Pricing

<https://aws.amazon.com/cloudtrail/pricing/>

---

## 1.4. Simple-Storage-Service-(S3)

### 1.4.1. S3 Security

**S3 is private by default!** The only identity which has any initial
access to an S3 bucket is the account root user of the account which owns that
bucket.

#### 1.4.1.1. S3 Bucket Policy

This is a **resource policy**

- controls who has access to that resource
- can allow or deny access from different accounts
- can allow or deny anonymous principals
  - this is explicitly declared in the bucket policy itself.

Different from an **identity policy**

- controls what that identity can access
- can only be attached to identities in your own account
  - no way of giving an identity in another account access to a bucket.

Each bucket can only have one policy, but it can have multiple statements.
I an identity inside one AWS account is accessing a bucket in the same account, then the effective access is the combination of all the applicable identiyt policies plus resource policy (in this case bucket policy)
For anonymous principal only resource policy applies as it is an unauthenticated user and hence no identity policy applies
Cross-Account Access: If an identity in an external policy tries to access a resource in your account, then your resource policy as well as their identity policy applies

#### 1.4.1.2. ACLs (Legacy)

A way to apply a subresource to objects and buckets.
These are legacy and AWS does not recommend their use.
They are inflexible and allow simple permissions.
![image](https://user-images.githubusercontent.com/33827177/143794657-e309731d-9bde-4250-903b-0fa4b6c2177b.png)
#### 1.4.1.3. S3 Exam PowerUp

When to use Identity Policy or Bucket Policy:

Identity

- Controlling high mix of different resources.
  - Not every service supports resource policies.
- Want to manage permissions all in one place, use IAM.
- Must have access to all accounts accessing the information.

Bucket

- Managing permissions on a specific product.
- If you need anonymous or cross account access.

ACLs: NEVER - unless you must.

### 1.4.2. S3 Static Hosting

Normal access is via AWS APIs.
This allows access via HTTP using a web browser.

When you enable static website hosting you need two HTML files:

- index document
  - default page returned from a website
  - entry point for most websites
- error document
  - similar to index, but only when something goes wrong

Static website hosting creates a **website endpoint**.

This is influenced by the bucket name and region it is in.
This cannot be changed.

You can use a custom domain for a bucket, but then the bucket name matters.
The name of the bucket must match the domain.

#### 1.4.2.1. Offloading

Instead of using EC2 to host an entire website, the compute service
can generate a HTML file which points to the resources hosted on a static
bucket. This ensures the media is retrieved from S3 and not EC2.

#### 1.4.2.2. Out-of-band pages

This may be an error page to display maintenance if the server goes offline.
We could then change our DNS and move customers to a backup website on S3.

#### 1.4.2.3. S3 Pricing

- Cost to store data, per GB / month fee
  - Prorated for less than a GB or month.
- Data transfer fee
  - Data in is always free
  - Data out is a per GB charge
- Each operation has a cost per 1000 operations.
  - Can add up for static website hosting with many requests.

### 1.4.3. Object Versioning and MFA Delete
![image](https://user-images.githubusercontent.com/33827177/143798619-79486e42-539a-4688-843c-d9329a748555.png)

Without Versioning:

- Each object is identified solely by the object key, it's name.
- If you modify an object, the original of that object is replaced.
- The attribute, **ID of object**, is set to **null**.

Versioning

- This is off by default.
- Once it is turned on, it cannot be turned off.
- Versioning can be suspended and enabled again.
- This allows for multiple versions of objects within a bucket.
- Objects which would modify objects **generate a new version** instead.

The latest or current version is always returned when an object version
is not requested.
![image](https://user-images.githubusercontent.com/33827177/143798993-96cb2fdd-d391-4099-9486-9bac2c127987.png)

![image](https://user-images.githubusercontent.com/33827177/143799139-a6890c12-3b21-43be-b98d-efdc3588227f.png)

![image](https://user-images.githubusercontent.com/33827177/143799222-8361ccc7-35a4-4672-a377-691c6f0d3d12.png)

![image](https://user-images.githubusercontent.com/33827177/143799298-d38d602f-1d68-458f-8b7c-f269a7caaf15.png)


When an object is deleted, AWS 
