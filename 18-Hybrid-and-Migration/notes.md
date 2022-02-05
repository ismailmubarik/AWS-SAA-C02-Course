## Hybrid Enviroment and Migration

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

![image](https://user-images.githubusercontent.com/33827177/149440312-9df113a3-439e-4989-8198-05a8c64fb47e.png)

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

With BGP AWS VPC and the routers can exchange information about their multiple links (talk to each about which networks are on the AWS side and which are on the customer side)  on the fly. In addition they can exchange information about the state of each link and adjust routing on the fly. This allows multiple links to be used at once b/w the two locations. And hence the Dynamic VPN offer a more robust and HA architecture.

With Dynamic Routes you can still add routes to the route table statically. Or you can make the whole architecture dynamic by enabling Route Propagation.

#### Considerations

![image](https://user-images.githubusercontent.com/33827177/149442658-ad16cbb3-bc1a-4068-811c-043ed365d3b0.png)

 A single VPN Connection with two tunnels: Speed Cap on VPN 1.25Gbps (This is the speed that AWS supports. The real speed will also depend on the speed at the customer side because there is an overhead of encryption and decryption (VPN uses encryption IPSec) the overhead can be significant. For exams if a speed greater than 1.25 Gbps is required then you can't use VPN.

Latency Considerations - this is inconsistent because it uses the public internet. SO in exam if low latency is required then may be use Direct Connect and not VPN

Cost - AWS hourly, GB out cost, data cap

Setup of hours or less

Great as a backup especially for Direct Connect (DX). We can also start with the VPN since it is quick to provision and request a DX which sometime takes weeks or months and when the DX is ready termiante the VPN or keep using it as a backup.

### AWS Direct Connect (DX)

This is a 1 Gpbs or 10 Gbps Network Port into AWS

This is at a DX Location (1000-Base-LX or 10GBASE-LR)

This is a cross connect to your customer router (requires VLANS/BGP)

You can connect to a partner router if extending to your location.

The port needs to be arranged to connect somewhere else and connect to
your hardware.

This is a single fiber optic cable from the DX port to your network.

VIFS are multiple virtual interfaces (VIFS) over one DX

- Private VIF (VPC)
- Public VIF (Public Zone Services)

Has one physical cable with no high availability and no encryption.

Can take weeks or month for physica cable to be installed.

**Public VIF** is only public services, not public internet.

**Private VIF** is one VPC

DX Port Provisioning is likely quick, the cross-connect takes longer.

Generally use a VPN first then bring a DX in and leave VPN as backup.

40 Gbps with aggregation

It does not use public internet and provides consistently low latency.

DX provides NO ENCRYPTION and needs to be managed on a per application basis.

### AWS Transit Gateway (TGW)

Network transit hub to connect VPCs to on premises networks

Significantly reduces network complexity.

There is a single network object which makes it HA and scalable.

Attachment to other network types.

VPC attachments are configured with a subnet in each AZ where service
is required.

You can use these for cross-region peering attachment.

Can share between accounts using AWS RAM

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
