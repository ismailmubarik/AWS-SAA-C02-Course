## Virtual Private Cloud (VPC)

### VPC Sizing and Structure
VPC Sizing and Structure is about designing an invidual IP plan for your organization and it includes designing the networking within that plan AWS VPC.
A VPC is essentially a private network inside the AWS. When designing a VPC the first and important thing to decide is the IP range for the VPC network i.e. the VPC CIDR.
It critically important to figure out in advance, what IP range the VPC would use even if that range is made up of multiple smaller ranges. It is critically important because
it is not easy to change later and as an architect is would save you a world of pain down the line if you get it right from the begining

![image](https://user-images.githubusercontent.com/33827177/148704989-a86260ca-1f89-489e-9f89-89ceca2da359.png)

You need to consifder other networks you will use or the nodes in your VPC would interact with..Overlapping or duplication of IPs would make network communication difficult. So
choosing wisely is important. So be mindful of IP ranges other VPC, CLoud environments, On-premises, Partners and Vendors use.

VPC Structures need to be figured out as well. For a given IP range it will be broken down further like into  Web tier, database tier, application tier (Tiers separate application components and allow different security policies to be applied).

### Case Study

![image](https://user-images.githubusercontent.com/33827177/148705034-a3153be5-73a1-4f9d-b536-e43ea8d5cea5.png)

We wil have to avoid the IP ranges already used by Animals for life network because our network would need to communicate with it. That is the IP ranges 192.168.0/24, 10.0.0.0/16, 172.31.0.0/16 is out of bounds. Besides that we cannot use the IP ranges for London, New York, and Seattle Office. Additonally, the GCP pilot for previois cloud pilot had a range of 10.128.0.0 --> 10.255.255.255 will also not be used.

![image](https://user-images.githubusercontent.com/33827177/148705221-13a1dd29-1f82-4b72-b529-e42793ebc3a9.png)\

Besides that the Azure network is using the same IP address range that the default AWS VPC uses i.e. 172.31.0.0/16. So we can't use the AWS default VPC IP

### More Considerations

Limit on VPC sizing in AWS: A VPC can be at the smallest a /28 network meaning 28 bits for network and 4 for IPs 
![image](https://user-images.githubusercontent.com/33827177/148705357-8e39db6b-dcfb-4f1d-8f51-fcde37ea6a86.png)

Try to avoid 10.0 and 10.1 in fact anything from 10.0 to 10.10. We cant use 10.128. to 10.255 because that is usually used by Google Cloud. 
A good idea is to start from 10.16. So we have a good range from 10.16. to 10.127 inclusive which we can use to create our network.  

![image](https://user-images.githubusercontent.com/33827177/148706085-3c06365f-1fe4-45bb-aef3-a6375b99971d.png)

How many IP ranges a business would require can be based on how many AWS regions your applicaiton would operate it. Be cautious and consider the maximum number of regions the business could operate in and then add more as a buffer.

### VPX Sizing

![image](https://user-images.githubusercontent.com/33827177/148706256-4af4eb3b-cb66-4687-849a-a40ef7aebb9f.png)

You can't just use a VPC to launch services into in AWS. Services uses subnets which is where IP addresses are allocated from. VPC services run from within subnets not directly from the VPC. Since a VPC is located in AZs. The first decision we have to make is that how many AZs your VPC will use. This decison impacts High Availibility and Reselience and  this depends somewhat on the Region the VPC is in because some Regions are limited in the number of AZs they have. Some have 3 some have more.
.
A good trick is to choose at least 3 caz it will work anywhere and then use 1 spare. So use 4 AZs. This means we will have to split the VPC into at least 4 smaller networks. So for example, if we started with /16s we will 4 /18s.

![image](https://user-images.githubusercontent.com/33827177/148707058-26d8650b-30d9-4b82-a7a0-6620ff391a17.png)

In addition we have tiers. A good idea is to use 4 tiers inside each network of the AZ. Now each Tier will have its own subnet in each availibity zone. So a total of 16 subnets.
(i.e. 4 Webm App, DB, and Spare subnets). So it means if we chose a /16 IP range for the whole VPC then each of the 16 subnets would need to fit into the /16 VPC. A /16 VPC split into 16 subnets results in 16 smaller network ranges of /20 (Remember each time the /16 prefix is increased by 1 it creates 2 networks of half sizes. So a /20 would create a 16 subnets of 16th the size of original VPC.

Now that we know that we need 16 subnets we can start with with a /17 VPC. Then each subnet would /21. Or we can start with /18 and then each subnet would /22.

https://github.com/acantril/aws-sa-associate-saac02/blob/master/07-VPC-Basics/01_vpc_sizing_and_structure/A4L_IPPlan.pdf

### Proposal

![image](https://user-images.githubusercontent.com/33827177/148707272-777d2267-ba60-45e4-94f9-f3575ccd6a72.png)

### Networking Refresher

#### IPv4 - RFC 791 (1981)

Represented different ways for people vs computer.

4 numbers from 0 to 255 seperated by a period

There is just over 4 billion addresses.

This was not very flexible because it was either too small or large for
some corporations. Some IP addresses was always left unused.

##### Class A range

Started at 0.0.0.0 and ended at 127.255.255.255.
Split into 128 class A networks
Handed out to large companies

##### Class B Range

Half the range of class A.
Started at 128.0.0.0 and ends at 191.255.255.255.

##### Class C range

Half of range class B
Started at 192.0.0.0 and ends at 223.255.255.255.

##### Class D and E

Outside of the scope of this class.

#### Internet / Private IPs

Defined by a standard RFC1918.

These can't communicate over the internet and are used internally only

One class A network: 10.0.0.0 - 10.255.255.255
16 Class B networks: 172.16.0.0 - 172.31.255.255
256 Class C networks: 192.168.0.0 - 192.168.255.255

#### Classless inter-domain routing (CIDR)

Not limited by the traditional methods.

**10.0.0.0/16** - 10.0.0.0 is the first address on the network and /16 is the size
of the network called the prefix. The bigger the prefix, the smaller the network
and the smaller the prefix, the bigger the network.

/16 provides 65,536 addresses. If you want to split this network, its easy

Just provide the following
10.0.0.0/17 and 10.0.128.0/17

You can split this again into a /18 if you like.
Just be careful to note the starting range properly.

This is called **subnetting**

#### IP address notations to remember

- **0.0.0.0/0** means all IP addresses
- **10.0.0.0/8** means 10.ANYTHING - Class A
- **10.0.0.0/16** means 10.0.ANYTHING - Class B
- **10.0.0.0/24** means 10.0.0.ANYTHING - Class C
- **10.0.0.0/32** means only 1 IP address

10.0.0.0/16 is the equivalent of 1,2,3,4 as a password. You should consider
other ranges that people might use to ensure it does not overlap.

#### Packets

Packet has a source IP address and a destination IP address. It also has
some data that the source IP wants to communicate with the destination IP.

TCP and UDP are protocols built on top of IP.

TCPIP means TCP running with IP
UDPIP means UDP running with IP

TCP/UDP Segment has a source and destination port number.

This allows devices to have multiple conversations at the same time.

In AWS when data goes through network devices, filters can be set based on
IP addresses and port numbers.

#### IPv6 - RFC 8200 (2017)

2001:0db8:28ac:0000:0000:82ae:3910:7334

In this case the value is hex and there are two octets per spacing or one hextet

The redudant zeros can be removed to create:

2001:0db8:28ac:0:0:82ae:3910:7334

or you can remove them entirely once per address like so

2001:0db8:28ac::82ae:3910:7334

##### Format

Each address is 128 bits long. They are addressed by the start of the network
and the prefix.

Since each grouping is 16 values, we can multiple the groups by this to achieve
the prefix.

2001:0db8:28ac::/48 really means the network starts at

2001:0db8:28ac:0000:0000:0000:0000:0000 and finishes at
2001:0db8:28ac:ffff:ffff:ffff:ffff:ffff

::/0 represents all IPv6 addresses

### VPC Sizing and Structure

Things to keep in mind

- What size should the VPC be. This will limit the how many services can fit into that VPC because each service occupies one or more IPs and they occupy a space inside the VPC.
- Are there any networks we can't use or we need to interact with and hence can't use?
- Be mindful of ranges other VPC's use or are used in other cloud environments, On-premise, used by Partners and/or Vendors, etc.
- Try to predict the future uses
- VPC structure with tiers and resilience (availability) zones

VPC min /28 network (16 IP)
VPC max /16 (65456 IP)

Personal preference for 10.x.y.z range

Avoid common range 10.0 or 10.1, include up to 10.10

Suggest starting of 10.16

Reserve 2+ network ranges per region being used per account.
Think of the highest region you will operate in and add extra as a buffer.

An example:

- 3 regions in US, Europe, Aus (5) x 2 - assume 4 AWS accounts
- Total of 40 ranges

![image](https://user-images.githubusercontent.com/33827177/144335118-d10a6837-5931-4e76-8a4f-164aa1301006.png)

VPC services run from within subnets, not directly from the VPC

![image](https://user-images.githubusercontent.com/33827177/144335320-88f68621-7c23-493a-aa5e-430fd640cfa2.png)

![image](https://user-images.githubusercontent.com/33827177/144335864-2f049faf-ea91-463d-ab0c-3945c42ae755.png)

#### How to size VPC

Subnet is located in one availability zone. How many AZ's will your VPC use?
Some regions have more AZ's, but they all have at least 3. It is a good practice
to start splitting the network into at least 4 AZ's.

This would be split into tiers (web, application, db, spare)

Taking a /16 subnet and splititng it 16 ways will make a /20

### Custom VPC

![image](https://user-images.githubusercontent.com/33827177/148708817-cf30641a-fd2e-41fe-8cfe-fcb36fdb1b28.png)

We will also be creating an Internet Gateway which will give resources in the VPC public access. We also be creating NAT Gateways private instances outgoing only access. We also be creating a Bastion Host which lets us connect securely into the VPC. We also be looking to Network Access Control List (NACL) to secure the VPC as well data transfer cost for any data that moves in and around the VPC.

VPCs are regionally isolated and regionally reselient. Meaning a VPC is created in a region and it operates from all of the AZs in that region. It lets you create isolated networks in AWS. Meaning even in a single region you can have multiple isolated networks.

Nothing is allowed IN or OUT of a VPC without explicit configuraiton. Its a network boundary and it provides an isolated Blast Radius. Meaning if you have a problem e.g. a resource or a set of resources inside a VPC are exploited the impact is limited to the VPC or anything connected to it...

Default VPC setup by AWS have the same static structure i.e. 1 subnet per AZ using the same IP address ranges and require no configuration. Custom VPC are the exact opposite as they let you create networks with almost any configurations which can range from a simple VPC to a complex multi-tiered one. Custom VPC also support Hybrid Networking.

You can pick Default or Dedicated Tenancy Model: This control whether the resources created in the VPC have shared hardware or dedicated

![image](https://user-images.githubusercontent.com/33827177/148709346-c4ab33d7-5a1e-4dae-8698-a7e8b4613d22.png)

- Regional Service - All AZs in that region
- Allows isolated networks inside AWS
- Nothing IN or OUT without explicit configuration
- Flexible configuration - simple or multi-tier
- Allow connection to other cloud or on-prem networking
- Default or Dedicated Tenancy
  - ***Default*** allows on a per resource basis(decision later on when you provision resources) to go on Dedicated Hardware or Shared Hardware.
  - ***Dedicated**** locks any resourced created in that VPC to be on dedicated always! Meaning if you choose Dedicated at VPC level then its locked in and resources will always be in on dedicated hardware
  - Should pick default most of the time as hardware which comes at a cost premium.

#### VPC Facts

- Can use IPv4 Private CIDR block and public IPs but the Private CIDR block is the main method of communicatio for the VPC. Use public when you want to make resources public. When you want the resources to communicate with public internet, or the AWS public zone or you want to allow connections to the resources from the public internent.
- Allocated 1 mandatory primary private IPv4 CIDR blocks
  - Min /28 prefix (16 IP)
  - Max /16 prefix (65,536 IP)
- Can add secondary IPv4 CIDR Blocks
  - Max of 5, can be increased with a support ticket
  - When thinking of VPC, it has a pool of private IPv4 addresses and can
  use public addresses when needed.
- Another option is a single assigned IPv6 /56 CIDR block
  - Still being matured, not everything works the same level of features as it does for IPv4
  - With increasing use of IPv6, this should be thought of as default
  - Range is either allocated by AWS, or private addresses can be used
  that are already owned. 
  - IPv6 does not have the concept of private and public, they are all publicly routalbe by default as
  public by default. But if you do use them in AWS you still have to explicitly allow connectivity to and from the public internet

#### DNS
AWS VPC also have a fully featured DNS. Provided by Route 53

Available on the base IP address of the VPC + 2.

If the VPC is 10.0.0.0 then the DNS IP will be 10.0.0.2

Two important settings which can cause issues.
These are found in the actions area of a VPC

- enableDnsHostnames: **Edit DNS hostnames**
  - Indicates whether instances with public IP addresses
in a VPC are given public DNS hostnames. If this is set to true, instances
will get get public DNS hostnames and viceversa.
- enableDnsSupport: **Edit DNS resolution**
  - indidcates if DNS resolution is enabled or disabled in the VPC
  - If True: instances in the VPC can use the dns ip address. **VPC + 2**
  - If False this is not available

If in the world or real exam there is a scenario where you have issues with DNS, these two should be the settings and Turn ON/OFF as appropriate.

### VPC Subnets

This is what services run from inside VPCs. They add structure, functionality and reselience to VPC.

![image](https://user-images.githubusercontent.com/33827177/148855684-1c348b55-8290-4a1a-a0eb-b3f91da93bb3.png)

This is where we will start which is basically a VPC skeleton. And the goal is to turn the above into what we have given below:

![image](https://user-images.githubusercontent.com/33827177/148856162-bea26fac-4531-4382-8e3b-e4d39ebad790.png)

You will notice that the Web Tier subnets are blue whereas they were green in the figure before that. That is because the Web Tier subnets are Private subnets right now. In
AWS blue stands for Private and Green for Public subnets. This is because subnets in AWS VPC start of as private and they need to be configured to become public.

***Exam PowerUp***

AZ Resilient subnetwork of a VPC. This runs within a particular AZ.

Runs inside of an AZ. If the AZ fails, the subnet and services also fail.

We put different components of our infrastructure into different AZs. So if the AZ fails only that component like Web, Database, etc. fail...

***1 subnet can only be only in 1 AZ
1 AZ can have zero or many subnets***

A subnet by default has IPv4 networking and is allocated and IPv4 CIDR
This IPv4 CIDR is a subset of the VPC CIDR block.

Cannot overlap with any other subnets in that VPC

Subnet can optionally be allocated IPv6 CIDR block as long as the VPC is also enabled for IPv6
(The range that is allocated for individual subnets is /64 range and it is a subset of /56 VPC. /56 can have 256 /64 subnets)

Subnets can by default communicate with other subnets in the VPC by default.

#### Reserved IP addresses

There are five IP addresses within every VPC subnet that you cannot use.
Whatever size of the subnet, the IP addresses are five less than you expect.

Lets assume that the subnet we are talking about is 10.16.16.0/20 (Range: 10.16.16.0 - 10.16.31.255 (So a total of 16 IPs)

- Network address: 10.16.16.0 --> This isn't just true for AWS its true for any IP network
- Network + 1: 10.16.16.1 - VPC Router
- Network + 2: 10.16.16.2 - Reserved for DNS
- Network + 3: 10.16.16.3 - Reserved for future AWS use
- Broadcast Address: 10.16.31.255 (Last IP in subnet) (Broadcast is not supported inside a VPC but nevertheless the 255 address is reserved. 

Keep this in mind when making smaller VPCs and subnets. Because the subnet has a total of 16 IPs but only 11 are available

VPC has a configuration object applied to it called DHCP Option Set.
This is how computing devices recieve IP addresses automatically. There is
one option set applied to a VPC at one time and this flows through
to subnets.

![image](https://user-images.githubusercontent.com/33827177/148858360-59165729-bda7-43a4-b4b9-8e4dbb445805.png)

- This can be changed, can create new ones, but you cannot edit one.
- If you want to change the settings
  - You can create a new one
  - Change the VPC allocation to the new one
  - Delete the old one

On every Subnet you can also define 2 different IP allocation options:

- Auto Assign IPv4 address
  - This will create a public IP address in addition to their private subnet
- Auto Assign IPv6 address
  - For this to work, the subnet and VPC need an allocation.

These are defined at the subnet level and flow down.

### VPC Routing and Internet Gateway
The Internet Gateway within the VPC enables communication from and to the VPC from the AWS Public Zone and the Public internet.

A VPC Router is a highly available device which is present in every VPC both default and custom. It moves traffic from and to VPC.

Highly available device available in every VPC which moves traffic from
somewhere to somewhere else.

Router has a network interface in every subnet in the VPC. **network + 1** address of the subnet.
By default Routers withou any custom configurations only routes traffic between subnets. For example, if an EC2 instance in one subnet wants
to communicate to another node in another subnet, the VPC router will route the traffic.

Controlled by 'route tables' defines what the VPC router will do with traffic
when data leaves that subnet.

A VPC is created with a main route table. If you don't associate a custom
route table with a subnet, it uses the main route table of the VPC.

If you do associate a custom route table you create with a subnet, then the
main route table is disassociated. A subnet can only have one route table
associated at a time, but one route table can be associated by many subnets.

#### Route Tables

![image](https://user-images.githubusercontent.com/33827177/148867677-cd2100f5-a491-4ad9-94f7-deff1306102a.png)

A route table is just a list of routes. When traffic leaves a subnet that the a route table is associated with, the VPC router reviews the IP packet and looks for the destination address. It looks at the Route Table and it identifies all of the routes which match the destination address. The destination field on the route table can match a 
exactly 1 specific IP address i.e. an IP with /32 prefix (Remeber the higher the prefix the higher the priority). The destination can also be a network match meaning a whole network of which an destination IP is a part. It can also be a default route. i.e. 0.0.0.0/0 destination. If traffic packets leaving a subnet has a destination which matches exactly 1 IP, then it is selected. If multiple routes match meaning /32 IP match or a /16 network match and there is of course 0.0.0.0/0 match then the prefix is used a priority i.e. the higher prefix one is selectted.

Once a single field in a route table is selected i.e. the  route with the highest priority, the router forward the traffic to that destination which is determined by the target field on the route. The target field will either point at the AWS Gateway or will say 'Local' (destination is in the VPC itself in which case the VPC will direct the traffic to the destination directly).

All route tables have at least one route i.e. the local route which matches the VPC CIDR range and it lets the VPC router know that the any packet with IP destination within the CIDR range is a local IP and hence the packets can be delivered directly.

***Exam PowerUp***
A route table are attached to Zero or more subnets. A subnet has to have a route table. Its either the VPC's Main Route Table or a custom one.

The local routes can never be updated. They always take priority. THey cannot be changed or edited. They match the VPC CIDR range. They are exception to the rule according to which the more specific/higher the IP prefix is the more priority it is given. Local route always take priority.

If the VPC is also IPv6 enabled, they it will also another local route matching the IPv6 CIDR range.

The higher the prefix, the more specific the route, thus higher priority. 

When target says **local** that means the VPC can route to the VPC itself.

Local routes always take priorty and can never be updated.

#### Internet Gateway

Regional resilient gateway attached to a VPC. You do not need an internet gateway per availibility zone. It is regionally resilient by design and one IGW will cover all the AZs which they VPC is using

DO NOT NEED one per AZ. One IGW will cover all AZ's in a region

1 VPC can have no IGW which would make it entierly prviate or it can have 1 IGW

![image](https://user-images.githubusercontent.com/33827177/148868126-6867b245-ae76-4fdc-bf42-13480fee0e5e.png)

A IGW can be created and attached to no VPC, but it can only be attached
to one VPC at a time at which point it is valid in all the AZ that the VPC uses.

The IGW Runs from the border of the VPC and the AWS public zone

It is what allows Gateway traffic between the VPC and the internet or AWS
Public Zones (S3, SQS, SNS, etc.)

It is a managed gateway so AWS handles the performance and from your perspective it simply works.

#### Using IGW

![image](https://user-images.githubusercontent.com/33827177/148869295-7fdace63-a018-44d5-8585-c21f404b79bd.png)

Once an IGW is created and attached to VPC it is available for use inside the VPC and we can use it as a target in the route tables. We then create a custom route table and attach it with the Web subnet (Note: We have different subnets like Web, App, etc.). Then we add IPv4 and IPv6 default routes to the route table with the target being the IGW. And finally we configure the subnet to allocate IPv4 and IPv6 (optional) addresses be default. 
At the end of the above steps, the subnet is now classified as a public subnet.


***Exam Scenario***
In this example, an EC2 instance has:

- Private IP address of 10.16.16.20
- Public address of 43.250.192.20

![image](https://user-images.githubusercontent.com/33827177/148869744-3c0d769c-0319-4386-9d15-2eeb97f0fbf7.png)

***The public address is not public and not really connected to the EC2 instance itself. 
The EC2 instance or any node within a subnet is never aware of its public IP. Don't fall 
for exam questions that tries to convince you otherwise***
Instead, the IGW creates a record that links the instance's private IP
to the public IP. This is why when an EC2 instance is created it only
sees the private IP address. This is IMPORTANT. ***For IPv4 it is not configured
in the OS with the public address.***

When the linux instance wants to communicate with the linux update service,
it makes a packet of data.

The packet has a source address of the EC2 instance and a destination address
of the linux update server. At this point the packet is not configured with
any public addressing and could not reach the linux update server.

The packet arrives at the internet gateway.

The IGW sees this is from the EC2 instance and analyzes the source IP address.
It changes the packet source IP address from the linux EC2 server and puts
on the public IP address that is routed from that instance. The IGW then
pushes that packet on the public internet.

On the return, the inverse happens. As far as it is concerned it does not know
about the private address and instead uses the instance's public IP address.

![image](https://user-images.githubusercontent.com/33827177/148870202-395aa6f2-d34a-4c3d-8b05-ae47869dedf9.png)

If the instance uses an IPv6 address, that public address is good to go. The IGW
does not translate the packet and only pushes it to an internet server and then back again.

#### Bastion Host / Jumpbox

It is an instance in a public subnet in a VPC.

These are used to allow incoming management connections.

Once connected, you can then go on to access internal only VPC resources.

Used as a management point or as an entry point at a private VPC.

This is an inbound management point. Can be configured to only allow
specific IP addresses or to authenticate with SSH. It can also integrate
with your on premise identification service. You can configure them exactly
how you need but they are generally the only entry point to a highly secure
VPC.

Historically, they were the only way to manage private EC2 instance. There are now
alternative ways to do that now.

### Stateful vs Stateless Firewalls

A connection is uniquely identified by a source port and IP and client/destination port and IP.

![image](https://user-images.githubusercontent.com/33827177/149034992-d61fe492-df95-4a7d-9900-d1bd1771d7ea.png)

There are 2 things to consider when dealing with Firewall rules:

1. Each connections between a Client & Server has 2 components i.e. The Request and The Response
2. The Response is always in opposite direction to the request but the direction of the request is not always outbound or inbound. It depends on the perspective

![image](https://user-images.githubusercontent.com/33827177/149035529-7c930182-415f-4f14-964f-40150d7212d9.png)

A Stateless Firewall doesn't understand the state of connections. It seas the connection from Client to the Server and the Response Connections from Server to the Client as two individual parts. And to allow or deny a connection/request you need two rules.

![image](https://user-images.githubusercontent.com/33827177/149037584-4aeb1c63-2d50-4c41-954f-6cd26b8a0493.png)

Two important points about Stateless Firewalls:

1. For Any Servers that accept connections and initiate connections (common for servers that accept connections from clients but also need to do software updates) you will need two rules for the inbound and outbound connections and they will need to be inverse of each other. And Similarly, there will be two rules for connections between the Server and the Update Server

3. The request connection to a server will always be to a well know port but the request connection to the client will always be to an unknown ephemeral port. And since the firewall is stateless and it has no way to know which port is used for the response, you will therefore have to allow a full range of ephemeral ports for respone connections.

For Stateful Firewalls you don't need to allow a a full range of ephemeral ports for respone connections because the Firwall can identify which post is being used and allow it based on it being a response to a request.

![image](https://user-images.githubusercontent.com/33827177/149037337-5e9db85e-29f3-4c71-93e4-ad9cdcc3c4db.png)

### Network Access Control List (NACL)

Network Access Control Lists (NACLs) are a type of security filter
(like firewalls) which can filter traffic as it enters or leaves a subnet.

All VPCs have a default NACL, this is associated with all subnets of that VPC
by default.

NACLs are stateless which means they don't know if traffic is request or response, its all about direction.

NACLs acan explicitly ALLOW or DENY traffic.

![image](https://user-images.githubusercontent.com/33827177/149039434-eea77e41-891b-4605-a3ec-6ebc7065a5a4.png)

Rules are processed in order .First the NACL determines if inbound or outbound rules apply. Then it starts from
the lowest rule number, it evaluates traffic against each indvidual rule until there is a match. Then rule is either
allowed or denined based on that rule and then processing stops. Its important to understand that if there is a DENY
and ALLOW rule which matches the same traffic but if the DENY rule comes first then the allow rule might never be processed
  
NACLs are used when traffic enters or leaves a subnet.
Since they are attached to a subnet and not a resource, they only filter
data as it crosses in

![image](https://user-images.githubusercontent.com/33827177/149039776-c2db4f93-cf0b-464e-b50f-be1b07e0fb07.png)

Lets say Bob initiates a connection to the WEB server. If we have NACL around the WEB subnet we will need inbound and outbound rule on the WEB NACL.
The Web Server might want to communicate witht the APP server. And in this case the traffic will cross two TCP subnet boundaries. The Web subnet and
the APP subnet boundary. So we are going to need an Outbound rule on the Web Outbound boundary and inbound rule at the App subnet boundary. For the response
from the APP to the WEB will also need two rules because it crosses two subnet boundaries (Outbound Rule on the App Subnet Boundary and Inbound Rule on the Web
Subnet boundary).

![image](https://user-images.githubusercontent.com/33827177/149040524-01f2e4bb-1e3f-488a-9c02-8f9d3fa1884b.png)

VPC has a Default NACL. All traffic is allowed because because it has a default Allow and a Deny. So they aren't used and they do nothing.

![image](https://user-images.githubusercontent.com/33827177/149040769-9d312e6c-ce08-48ad-b05c-80f78a38d542.png)

![image](https://user-images.githubusercontent.com/33827177/149040809-c5ddc02b-5af7-4df9-965f-8fd1ee8d02f1.png)

If two EC2 instances in a VPC communicate, it is not involved because the NACL is not used for intra subnet communications.

NACL are not aware of any logical resources. They only allow or deny IPs/CIDR, Ports and protocols.

![image](https://user-images.githubusercontent.com/33827177/149041232-a56cc341-028f-48bc-afc9-eb7240d43a47.png)

You are trying to ssh into the bastion host

- the traffic leaves the machine
- crosses the internet
- uses **inbound** rules

The action can be for the traffic to **cross** or **deny** the traffic.

Each rule has the following fields related to traffic

- type
- protocol: tcp, udp, or icmp
- port range

Examples:

- ssh: tcp port 22
- http: tcp port 80
- https: tcp port 443
- ping traffic: icmp

Inbound rule: Source - who traffic is from
Outbound rule: Destination - who traffic is destined to

If all of those fields match, then the first rule will either allow or deny.

The rule at the bottom with * is the **implicit deny**
This cannot be edited and is defaulted on each rule list.
If no other rules match the traffic being evaluated, it will be denied.

There is also rule 100, which is a catch all allow. This can be edited.

#### NACLs example below

- Bob wants to view a blog using https(tcp/443)
- We need a NACL rule to allow TCP on port 443.
- All IP communication has two parts
  - Initiation
  - Response
- Bob is initiating a connection to the server to ask for a webpage
  - This is the initiation
- Server will respond with an **Ephemeral** port
- Bob talks to the webserver connecting to a port on that server (tcp/443)
  - This is a well known port number
- Bob's pc tells the server it can talk to back to Bob on a specific port
  - Wide range from port 1024, 65535
  - That response is outbound traffic
- When using NACLs, you must add an outbound port for the response traffic
as well as the inbounding port. This is the ephemeral port.
- If the webserver is not managing the apps server, it must communicate
back on a different port.
- The data is moving out of the web subnet and in on the app subnet
  - This complicates things

#### Exam powerup

NACLs are stateless and so see initiation and reponse phases of a connection
and 1 inbound and 1 outbound stream requiring two roles (one IN one OUT)

NACLs are attached to subnets and only filter data as it crosses the
subnet boundary. Two EC2 instances in the same subnet will not check against
the NACLs when moving data.

Can explicitly allow and deny traffic. If you need to block one particular
thing, you need to use NACLs.

They only see IPs, ports, protocols, and other network connections.
No logical resources can be changed with them.

NACLs cannot be assigned to specific AWS resources.

Use with security groups to add explicit deny (Bad IPs/nets)

One subnet can only be assigned to one NACL at a time.

NACLs are processed in order starting at the lowest rule number until
it gets to the catch all. A rule with a lower rule number will be processed
before another rule with a higher rule number.

### Security Groups

Security Groups (SGs) are another security feature of AWS VPC ... only unlike NACLs they are attached to AWS resources, not VPC subnets.

SGs offer a few advantages vs NACLs in that they can recognize AWS resources and filter based on them, they can reference other SGs and also themselves.

But.. SGs are not capable of explicitly blocking traffic - so often require assistance from NACLs

Security Groups operate above the NACL on the OSI Layer so they have more features.

SGs are not attached to instances or subnets butr rather to Elastic Network Insterfaces (ENIs). So even if you see that a SG is being attached to an instance
what is actually happening is that it is being attached to the Network Interface of that instance

![image](https://user-images.githubusercontent.com/33827177/149043496-ffc44657-8c75-4b7d-85cf-dc4eccaf6084.png)

We have a public subnet containing a EC2 instance with an ENI. Consider a SG as sth that surrounds a NI. It has inbound and outbound rules like a NACL.

![image](https://user-images.githubusercontent.com/33827177/149044418-837df106-ab3c-4cdf-828d-f859e6890e8a.png)

SGs cannot explicitly block traffic. For example, in the scenario above if you are allowing 0.0.0.0/0 to access the instance on TCP port 443 meaning the whole IPv4 internet. Then you can't block anything specific. What if Bob is a Bad actor but you can't block him since you can't explicitly Deny an IP.

Security Groups are capable of using Logical Referencing. 

A VPC with a public WEB subnet and a prviate APP subnet. Inside the Web Subnet is the Catagram Web Instance and inside the APP subnet is the application backend instance.
Both of these are protected by SGs. i.e. A4L/APP & A4L/WEB.

![image](https://user-images.githubusercontent.com/33827177/149045617-f19b2537-2088-4075-afd9-7118e33a73cd.png)

To enable communication b/w APP and WEB we reference the WEB SG. The APP inbound rule allows the port 1337 but it references as the source a Logical resource i.e. the A4L-Web SG. Using a logical resource in this way actually referencesanything which has the SG asociated with it. So this means that anything which has A4L-WEB attache to them can access whatever has A4L-APP attache to them using TCP port 1337.

This means we don't have to worry about IPs or CIDR ranges. Another benefit is that it scales really well. So any instances added to APP or WEB subnet will automatically have the SG assigned automatically.

![image](https://user-images.githubusercontent.com/33827177/149045746-cc4a0e9a-92b5-41e0-99fd-5d492e1e4684.png)

![image](https://user-images.githubusercontent.com/33827177/149046410-53b18086-0e99-4ea4-9e47-a59e5a56c830.png)

Logical Referencing allows even more functionality. They allow self-referencing. For example, we have a private subnet with an ever changing number of APP instances. Right now its 3 but might be 3, 30 or 1. We can create a SG that allows incoming communication Port 1337 from the Web SG (So the Web Instance can connect). But it also has a self-referential rule which allows all traffic. Sof if this rule is attached to all instances can receive all traffic from all other instances in the SG. IP changes are handled automatically which is useful if the instances are within an AutoScaling Group that is provisioning/terminating resources based on the load. Also simplifies intra-app communications.

Important to remeber that while NACL allows to explicitly deny traffic SGs dont. So you would generally, use to block traffic from Bad actors and use SGs to allow traffic to your VPC based resources because of its capability to use Logical Based and Self Referencing.

An EC2 instance has one more more attached network interfaces.
The network interfaces and not the IP itself is given the private ip addresses.

When data is sent, it goes through this network interface. This is located
in one and only one AZ.

SGs are boundaries which can filter traffic. Instead of being attached
to a subnet, they are assigned to an AWS resource.

When Bob browses, it must pass through the security group to the network
interface of the EC2 interface, the webserver.

The security group is attached to a resource and not a security subnet.

Just like NACLs, security groups have two sets of rules.

NACLs are stateless. For a single communication between Bob and the server,
it has two seperate and stateless points of communication.

SGs are stateful. When Bob communicates with the server, the response with the
traffic is viewed as the same communication. This means only one rule is
required, one inbound rule. Any return traffic is automatically included
in the allow.

This makes them easier to deal with.

SG understand AWS logical resources so they're not limit to IP traffic only.
This means they can have a source and destination referencing the instance
and not the IP.

The SG can reference itself and will do so when creating the default.

SG's have a hidden implicit **Deny**. Anything that is not allowed in the rule
set for the SG is implicitly denied.

SG cannot explicit deny anything.

Wordpress Example: Allow access using port 443 from anywhere. What
if Bob is evil and wants to damage it. The security group will allow anything
from anywhere on that port.

NACLs are used in conjunction with SGs to do explicit denys.

#### Exam Powerups

SGs are stateful, they see traffic and response as the same rule.
Can filter based on AWS logical resources or other SGs
SGs have an implicit deny and can explcit allow
SG CANNOT EXPLICITLY DENY

NACLs are used when products cannot use SGs, NAT Gateways.
NACLs are used when adding explicit deny, bad IPs or bad actors
SG is the default almost everwhere because they are stateful
NACLs are associated with a subnet and only filter traffic that crosses
that boundary. If they are in the same subnet, it will not do anything

### Network Address Translation (NAT) gateway

![image](https://user-images.githubusercontent.com/33827177/149051608-a3be5adc-26e2-4019-b3f4-8acbe580d466.png)

Set of different processes that can adjust IP packets by changing
their source or destination addresses.

The IGW actually performs a type of NAT called Static NAT. It is how
a resource can be allocated with a private IPv4 address and when the packet
passes through the IGW it is given a public address and sent forth. The reverse is
done on return. And this is how an IGW implements Public IPv4 Addressing.

IP masquerading, hides entire CIDR block behind one IP. This is many private
IPs attached to one public IP.

This gives private CIDR range **outgoing** internet access.

Incoming connections don't work. Outgoing connections can get a response
wrapped together, but incoming cannot initiate.

Two ways to provide NAT:

1. Configure an EC2 instance to provide NAT 
2. Managed NAT Gateway by AWS

So how do we enable the instance inside the Private APP subnet to peform certain functions that requires access to public newtorking for example, software updates. We can make the subnet public but we might now want to do architecturally. We can host a software update server inside the VPC. but that comes with an admin overhead.

![image](https://user-images.githubusercontent.com/33827177/149052901-a7840334-f373-4d18-b9a4-0464aa659daa.png)

NAT offers a 3rd option. We provision a NAT inside a public subnet. A public subnet allows public IP addresses. The public subnet has a default Route Table attached to it which provides default public version IPv4 routes pointing the IGW. So the NAT Gateway has a public IP that is routable across the public internet and so it is able to send data out and get data back in.
The privte subnet can also have a Rout Table but it can be different than the Public Subnet one. We can configure the Route Table in the Private Subnet to point at the NAT Gateway.

Lets say instance one generates some data e.g. it is looking for software updates. It has a source and destination IP. The packet is routed to the NAT Gateway (NATGW). The NATGW makes a record of this data packets. It source the destination the packet is for. The source IP address of the instance sending the packet and other details lie ports, etc. that will help it identify this specific communication in the future. All this data is stored in a Translation Table. The NATGW then changes the sources address of the packet to its own address (i.e. the source address of NATGW). Now if this was a network other than AWS it would have instead assigned a public IP address to it. But nothing inside an AWS VPC has a public IP directly attached to it. This function is performed by the IGW. So the NATGW has a default route pointing at the IGW. And so the packet is sent to the IGW by the VPC router. The IGW knows that the packetis from the NATGW and the NATGW has a public IPv4 IP associated with it. So the IGW changes the source address to the NATGW's public IPv4 address.

![image](https://user-images.githubusercontent.com/33827177/149054267-e580f067-ec12-4191-b826-11ff851d4fd8.png)

![image](https://user-images.githubusercontent.com/33827177/149054311-4cc22b96-6934-47b1-a968-493f29d6a503.png)

![image](https://user-images.githubusercontent.com/33827177/149054173-2b6fbf26-5c3c-43be-8602-d0b515a00655.png)

The job of the NATGW is to allow multiple private IPs to masquerade behind a single IP.

A NATGW uses a special type of IPv4 address called Elastic IPs. These are IPv4 IPs that are static as in they don't change. These IP addresses are allocated to your account in a region. 

NATGW are AZ resilient. They can recover from a HW failure inside an AZ but if an AZ entirely fails then the NATGW fails.

![image](https://user-images.githubusercontent.com/33827177/149054912-8ef495fc-00a4-477e-b679-8a704e7a19c8.png)

NATGW can get costly if you have one in each AZ of a region. So its important to think about optimal VPC design.

You can have multiple NATGW and split your subnets across multiple provisoned products in the same AZ.

For NATGW you are chared an houryly rate multiplied by the number of NATGW you have. 

***Exam PowerUp***: They have 2 charges. Hourly charges (Partial hours are counted as full hours) i.e. 4 c/hr. There is also a data processing charge i.e. 4 c/GB. 

#### Key facts

Must run from a public subnet to allow for public IP address. They can be used for whatever you want until they are de-allocated.

Uses Elastic IPs (Static IPv4 Public)

- Don't change
- Allocated to your account

![image](https://user-images.githubusercontent.com/33827177/149058434-87a6a56e-42db-41d7-bc1f-e0c9de6c8a00.png)

NATGW are AZ resilient service (HA in that AZ). THEY ARE NOT REGIONALLY RESILIENT. So you need a NATGW in each AZ in the above example. An IGW in comparison Regionally Resilient.

For a fully region resillient service, you must deploy
one NATGW in each AZ
RT in each AZ with NATGW as target

Managed service, scales up to 45 Gbps. Can deploy multiple NATGW to increase
bandwidth.

$ is charged on duration and data volume

NATGW is highly available in **one** AZ. If that AZ fails, there is
no recovery. You must deploy one in each AZ you use for region resillience.

#### Nat Instance vs NATGW

NAT used to be provided by NAT instances that are just EC2 instances to provide NAT functionality. It essentially all data on its NIC when that NIC is not either the source or destination. So if NAT instance is running it will possibly receive some data with source other than itself meaning some other instance or resource in the VPC and the destination will be a host on the internet. So by default the traffic would be dropped.
***If you need to allow an EC2 instance to function as a NAT instance, then you need to disable a feature called 'Source/Destination Checks'--> Can be disable using the Console UI, the CLI or the API.***

***Exam Power Up:*** If you ever need to use a NAT instance, by default an EC2 instance filters all the traffic it sends or receives.

![image](https://user-images.githubusercontent.com/33827177/149058759-db882b80-2eb4-48c3-b78b-fc9b12e1fc00.png)

Architecturaly NAT instances and NATGWs are kind of the same. Both need apublic IP and need to run in a public subnet. They both need a functional IGW.

NATGW should be the default for most situations as a NAT EC2 instance is not recommended by AWS. But there are scenarios where you might consider using a NAT instance:

***Exam Powerup:***

Performance Criteria:
If you value availibility, Bandwidth, low maintenance and high performance then use NATGW. Its custom design to do Network Address Translation and it scales well
NAT instance is limited by the performance of the EC2 instance it is running on. Since it is running on a general EC2 instance it isnt able to achieve the same lelvel of performance as a NATGW. 

Availibility Criteria: A NAT instance is running a single EC2 instance running inside an AZ. If the EC2 hardware, storage, netowrk, etc. fails, the NAT would fail. And it will fail if the AZ entirely fails. So A NATGW has some benefits over a NAT instance in terms of availibility. Inside a single AZ its HA. It can automatically recover. It can automatically scale. So it removes risks of outage. The only way a NATGW fails is if the entire AZ fails (So provision multiple NATGW across different AZs to ensure availibility. For max availibility one NATGW in every AZ in a public subnet).
 
Cost Criteria: A NAT instance can be cheaper if cost is a consideration. At high volumes of data it can be significantly cheaper. Or at really low volume or if yours is a test VPC. You can use a Free-Tier instance for low volume or test scenarios. A NATGW scales automatically so if your data volumes increases the bill will increase automatically. But if you want to put a limit you can use a limited EC2 instance with known and low specs to contain the bills. A NATGW is also not free-tier eligible.

You can connect to a NAT instance like any other EC2 instance. You can multipurpose them like using it both as a NAT instance and Bastion Host. You can also use them for port forwarding. So you can have a port on the instance externally that can be connected to over the public internet and have it forwarded onto an instance inside the VPC.

***NATGW cannot do port forwarding or be a bastion server***

NAT instances are just EC2 instances so you can filter traffic using the network ACL on the subnet that the instance is in or through SG associated with that instance.

Important For Exam:
***NATGW don't support SG. You can only use NACL with NATGW*** Important For Exam

Any IPv6 IP can communicate directly with the AWS Public Zone and the Public Internet as long as you don't have any NACL or SGs

![image](https://user-images.githubusercontent.com/33827177/149062812-d07624b8-48b3-48e9-9884-10f76c1248de.png)
