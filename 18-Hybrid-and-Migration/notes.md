## Hybrid Enviroment and Migration

### Border Gateway Protocol (BGP)
BGP is a routing protocol used to control how data flows from points A to points B and then C and arrives at the destination D. Both AWS Direct Connect and Dynamic VPN utilize BGP.
BGP as a system is made up of lots of self-managing networks known as Autonomous Systems. These are like black boxes onlcy concerned with network routing in and out of the AS.

BGP Operates over TCP and port 179 thus ensuring Error Correction and Flow Control

Not Automatic - Meaning peering connection between two AS of the BGP have to manually configured abd once configured these two AS and communicate with each other information about Network Topologies.

An AS will learn Network Topology information fron all the peering connections it has and it will also communicate this learnt informationw to all its peers. This is how the BGP scales up like a ripple...

![image](https://user-images.githubusercontent.com/33827177/153313817-d6737ebb-d114-43bf-8c0d-0891add37888.png)

![image](https://user-images.githubusercontent.com/33827177/153314713-ea97bacd-8035-4ca4-8a39-93400f5a3e78.png)

![image](https://user-images.githubusercontent.com/33827177/153314723-664a951c-58d8-4b70-aa75-22302734c928.png)

![image](https://user-images.githubusercontent.com/33827177/153314831-1c875fd8-718c-47e3-b4b3-e4d79cffacd2.png)

![image](https://user-images.githubusercontent.com/33827177/153315362-5a9140a4-86bb-4f20-a925-ddfc991af3e4.png)

BGP always considers the shortest path not the speed/latency. AS path prepending can be used if one path is prefered due to say speed/latency.

![image](https://user-images.githubusercontent.com/33827177/153315461-079773ff-94f5-4354-a1b6-0becd91ad0b0.png)

### AWS Site-to-Site VPN

The quickets way to create a network link b/w an AWS environment and sth that is not on AWS (on-premises, another cloud environment or a data centre)

![image](https://user-images.githubusercontent.com/33827177/152662370-5b23e550-2507-4f49-b4d9-b49d2a94a324.png)

A logical connection between a VPC and on-premise network encrypted
using IPSec, running over the public internet.

***This can be full High Availability if you design it correctly***

Quick to provision, less than an hour.

VPNs connect VPCs and on-prem

Virtual Private Gateway (VGW) another logical gateway object is the target on one or more route tables.
It is something you create and associate with a single VPC and is a target of one or more route tables (associatd with subnets)

Customer Gateway (CGW). Can refer to two different things

- logical configuration within AWS
- also the thing that configuration represent i.e. the physical on-premises router which the VPC connects to

So when you see CGW it can actually refer to any of the two above.

VPN connection between the VGW and CGW itself which stores the config and is linked to VGW and CGW

### AWS Site-to-Site VPN
We have A4L VPC with three private subnets in the AZs. We have AWS Public Zone where AWS public services operate from. We have a Public Internet connected to AWS Public Zone and then the Corporate office of A4L using an IP range of 192168.10.0/24.
Step-1 to create a VPN connecion is to gather information like the IP address range of the VPC, IP range of the on-premises network and the IP address of the physical router on the customer premises. Once we have all this information we will create a VGW and attach it to the A4L VP.
Our physical router would have an external IP address and for this router we will create a logical gateway object i.e. CGW. This is a logical configuration entity that refences the physical router in the A4L corporate office. In this case we need to define its public IP address so that logical CGW entity matches the physical router. 

(The VGW is a HA gateway object and behind the scenes it has physical endpoints in different AZs with public IPv4 addresses meaning the VGW is HA by design. It means one end point can fail but other endpoints and hence VGW will continue to provide service. *** But that doesn't mean that the 'whole thing' is HA ??? ***)

The next step is to create a VPN connection inside AWS. There are both static and dynamic VPNs. We shown above is a static VPN. When creating a VPN you need to link it to a VGW. This means that it can use the end points which the VGW provides. We also need to specify the customer gateways that we will be using and when we do 2 VPN tunnels are created. One b/w each Endpoint and the physical on-premises router. As long at keast connection between the VPN and an Endpoint is active the connection between the VPC and the on-premise router will be active. So this a partially HA design. If one AZ on the AWS side fail the other AZ endpoint will continue work and so the conneciton will be active.

A VPN tunnel is an ecnrypted channel through which data can flow between a VPC and a on-premise network.

Since shown above is a static VPN we have to statitcally configure the VPN connection with IP addressing information. Meaning we have to tell AWS the ip range used on the on-premise corporate A4L network and we have to configure the on-premises side so that it knows the IP range used by the AWS VPC.

So why is this arhcitecture partially HA and not fully HA?? Well the fault is on the side of the on-premises network. If the physical router fails the VPC connection fails. So it is highly available on the customer side but not on the customer side. 

![image](https://user-images.githubusercontent.com/33827177/152662678-bf2cc8bf-fc4c-4afd-8c49-2b9f92adda65.png)

So how do you make it HA? Add a redundant router with a separate inernet connection and ideally placing it in a separate building. This will creatre a separate VPN connection and the VGW will have addtional separate endpoints in each AZ for this redundant VPN which will then connect with the redundant router on the customer end.

![image](https://user-images.githubusercontent.com/33827177/152662729-4a8decff-ca73-4d10-a777-e4b5ab27a440.png)

***Static vs Dunamic VPN***
The dynamic VPN uses a protocol called BGP i.e. Border Gateway Protocol. But if the customer router doesn't support BGP then Dynamic VPN cannot be supported.

The benefit of the Static VPN is that it is simple and just uses IPSec and works alomost anywhere i.e. with any routers.
But with Static VPN you are restriced on things like Load-Balancing and multi-connection failover. So if you need advanced HA or you a multiple connections, if the VPNs need
to work with Direct Connect then you need to use Dynamic VPN

![image](https://user-images.githubusercontent.com/33827177/152663058-83e909ca-34f8-422e-a9cd-2d8024424986.png)

With BGP AWS VPC and the routers can exchange information about their multiple links (talk to each about which networks are on the AWS side and which are on the customer side)  on the fly. In addition they can exchange information about the state of each link and adjust routing on the fly. This allows multiple links to be used at once b/w the two locations. And hence the Dynamic VPN offer a more robust and HA architecture.

With Dynamic Routes you can still add routes to the route table statically. Or you can make the whole architecture dynamic by enabling 'Route Propagation'.

#### Considerations

![image](https://user-images.githubusercontent.com/33827177/149442658-ad16cbb3-bc1a-4068-811c-043ed365d3b0.png)

A single VPN Connection with two tunnels: Speed Cap on VPN 1.25Gbps (This is the speed that AWS supports. The real speed will also depend on the speed at the customer side because there is an overhead of encryption and decryption (VPN uses encryption IPSec) the overhead can be significant. For exams if a speed greater than 1.25 Gbps is required then you can't use VPN.

![image](https://user-images.githubusercontent.com/33827177/152663072-5e18ef3a-9f2e-4f59-bc1e-f5b417ef888e.png)

There is also a limit on the speed of the VGW as a whole and its also 1.25 Gbps and it is for all VPN connections connecting to the VGW

Latency Considerations - Latency is incosistent because it uses the public internet. The latency can be high since multiple Hops over the public internet are required. So in exam if low latency is required then may be use Direct Connect and not VPN

Cost - AWS hourly cost , GB (data Transfer charge) out cost, data cap (from internet connection package, May be?)

Setup of hours or less

Great as a backup especially for Direct Connect (DX). We can also start with the VPN since it is quick to provision and request a DX which sometime takes weeks or months and when the DX is ready terminate the VPN or keep using it as a backup.

### AWS Direct Connect (DX)

AWS Direct Connect is an actual physical connection into the AWS network.

This is a 1 Gpbs or 10 Gbps Network Port into AWS. A DX is not a connection to be precise, it is a port assigned to an AWS account

This is at a DX Location (1000-Base-LX or 10GBASE-LR) which are located in Data Centers globally.

For Larger Organizations, the DX is a cross connect meaning a direct connection to your customer router (also inside a DX Location but how??)i.e. from the DX port location 
to your router (and this requires your router to support VLANS/BGP)

For smaller Organizations, you can connect to a partner router if extending to your location.

The port needs to be arranged to connect somewhere else and connect to
your hardware.

Conceptually (not actually) think of this as a single fiber optic cable from the DX port to your network.

VIFS are multiple virtual interfaces (VIFS) over one DX. Each VIFs is a VLAN and a BGP connection between your router and the AWS DX router

- Private VIF (VPC): are associated with VPGW and connect to a single VPC and they are used to provide connectivity between a VPC and your private network. You can have as many private VIFS on top of a DX connections as you want.
- Public VIF (Public Zone Services): Provides direct connections to AWS public services like AWS S3, DynamoDB, SNS, SQS, etc.

![image](https://user-images.githubusercontent.com/33827177/153317808-b1d62f9a-5edc-4042-a723-1bcddcfeb583.png)

**Has one physical cable with no high availability and no encryption.**

![image](https://user-images.githubusercontent.com/33827177/153318293-065056db-7fd6-440a-b9e5-83532dfa135e.png)

Can take weeks or month for physical cable to be installed.

**Public VIF** is only public services, not public internet.

**Private VIF** is one VPC

DX Port Provisioning is likely quick, the cross-connect takes longer.

Generally use a VPN first then bring a DX in and leave VPN as backup.

40 Gbps with aggregation i.e. 4 10 GB ports

It does not use public internet and provides consistently low latency.

DX provides NO ENCRYPTION and needs to be managed on a per application basis.

![image](https://user-images.githubusercontent.com/33827177/153318617-6f23662b-6bb6-4419-9e2f-48bbdbfc01d3.png)

### Direct Connection Resilience

![image](https://user-images.githubusercontent.com/33827177/153323983-3744928f-e0e8-4767-aad7-dd963f88311d.png)

![image](https://user-images.githubusercontent.com/33827177/153324222-eb877df6-4211-4569-9ef2-151f6ebc5b63.png)

![image](https://user-images.githubusercontent.com/33827177/153324381-abd4c59e-22e5-4349-bcc6-1bca6cb0b94d.png)

![image](https://user-images.githubusercontent.com/33827177/153325266-b71b825d-f0b4-487b-a8c0-45d2572d2f76.png)

### AWS Transit Gateway (TGW)

![image](https://user-images.githubusercontent.com/33827177/153325685-914335c2-0886-4d1d-af43-d68a28c938ad.png)

Network transit hub to connect VPCs to on premises networks using site to site VPNs and DX

Significantly reduces network complexity within AWS

This is a single network object which makes it HA and scalable.

Attachment to other network objects within AWS and on-premises network

![image](https://user-images.githubusercontent.com/33827177/153325912-4ef74aa1-0977-4c5e-8098-d8e6cc2a1ab1.png)

VPC attachments are configured with a subnet in each AZ where service
is required.

![image](https://user-images.githubusercontent.com/33827177/153326965-99cae7ae-45a7-4b56-a707-1795dba9876a.png)

You can use these for cross-region peering attachment.

Can share between accounts using AWS Resource Access Manager (RAM) which is an AWS service that allows you to share products and services b/w different accounts

![image](https://user-images.githubusercontent.com/33827177/153327257-c0093d94-a3c9-44e4-b7c1-a3c0bbbaacb5.png)

### Storage Gateway

Hybrid Storage Virtual Application (On-premise)

Scenarios

Extend storange of File and Volume Storage into AWS.
Keep volume storage backups into AWS.
Tape backups into AWS. Can act as emulation layer.

Migration of extisting infrastructure into AWS slowly.

- Tape Gateway (VTL) Mode
  - Virtual Tapes are stored on S3

- File Mode : SMB and NFS
  - File Storage Backed by S3 Objects

- Volume Mode (Gateway Stored)
  - Block Storage backed by S3 and EBS
  - Great for disaster recovery
  - Data is kept locally
  - Awesome for migrations

- Volume Mode (Cache Mode)
  - Data as added to gateway is not stored locally.
  - Backup to EBS Snapshots
  - Primarily stored on AWS
  - Great for limited local storage capacity.

### Snowball / Edge / Snowmobile

Move large amounts of data IN and OUT of AWS

Physical storage the size of a suitcase or truck.

Ordered from AWS, use, then return.

#### Snowball

Anything on Snowball uses KMS
50TB or 80TB Capacity
1 Gbps or 10 Gbps
This makes sense from 10 TB to 10 TB and over many premises.
This only includes storage

#### Snowball Edge

Both storage and compute
Larger capacity vs snowball
10 Gbps or up to 100 Gbps

Storage optimized (with EC2) includes 1TB SSD
Compute optimized
Compute with GPU as above with GPU

These are great for remote sites when ingestion is needed

#### Snowmobile

Portable data center within a shipping container on a truck.

This is a special order and is not available in high volume.
Ideal for single location where 10 PB+ is required.

Up to 100 PB per snowmobile.

This is not economical for multi-site for sub 10 PB

### AWS Directory Service

This is a managed service with lots of use cases.

Stores objects, users, groups, computers, servers, file Shares with
a structure.

Multiple trees can be grouped into a forest.

Commonly used in Windows Environments.

Sign in to multiple devices with the same username/password provides
central management for assets.

#### AWS managed implementation

Runs within a VPC as a private service.

Provides HA by deploying into multiple AZs.

Certain services in AWS need a directory, Amazon Workspaces.

To join EC2 instances to a domain you need a directory.

Can be isolated or integrated with existing on-prem system.

Could act as a proxy back to on-premises.

#### Picking the Modes

Simple AD should be default

Microsoft AD is anything with Windows or if it needs a trust relationship
with on-prem. This is not an emulation.

AD Connector - Use AWS services without storing any directory info in the
cloud, it proxies to your on-prem directory.

### AWS DataSync

Data transfer service TO and FROM AWS.

This is used for migrations or for large amounts of data processing transfers.

Designed to work at huge scales. Each agent can handle 10 GB and each job
can handle 50 million files.

This keeps metadata.

Has built in data validation to ensure the data matches.

Each agent is about 100 TB per day.

Can use bandwidth limiters to avoid customer impact

Has incrememetal and scheduled transfer options

Compression and encryption is also supported

It does data validation and automatic recovery from transit errors.

AWS service integration with S3, EFS, FSx for Windows servers.

Pay as you use product.

The data is encrypted in transit and all of the data transfer in parts.

#### Components

Task is a job within datasync and defines what is going from where to where

Agent is software to read and write to on prem such as NFS or SMB

Location is the FROM and TO

### FSx for Windows File Server

Fully managed native windows file servers/shares
Designed for integration with windows environments

Integrates with Directory Service or Self-Managed AD

Single or Multi-AZ within a VPC.

Can perform on-demand and scheduled backups.

File systems can be access using VPC, Peering, VPN, Direct Connect. Native
windows filesystem or Directory Services.

#### Words to look for

VSS - User Driven Restores
Native file system accesible over SMB

Windows permissions model

Product supports DFS, scale out file share.

Managed - no file server admin

Integrates with DS and your own directory.
