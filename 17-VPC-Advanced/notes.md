## VPC Advanced

### VPC Flow Logs

Capture packet metadata, not packet contents. Things like source and
destination Is, source and destiantion ports, and packet size, etc.. Anything which could be observed
from the outside of the packet. Anything related to the flow of data through a VPC. If you need to capture 
packet data you might want to install a Packet Sniffer on an EC2 instance.

![image](https://user-images.githubusercontent.com/33827177/149243531-562be8c4-e865-42e3-9f66-d78912f5b0a1.png)

Flow Logs works by attaching virtual monitors within a VPC. These can be applied at three different levels:

- VPC Level : Monitors all ENIs in every subnet within in that VPC
- Subnets Level: Monitors ENIs in that subnet
- Interface Level: Monitors ENIs directly

VPC flow logs are not realtime.

Flow Logs capture what are called Flow Log Records.

Desination can be S3 or CloudWatch logs. Each of this comes with a trade-off.
If you use S3 you are able to access the Flow Logs directly and  integrate it with 
a 3rd-party monitoring solution. If you use Cloudwatch Logs you can stream that data
into different locations and you can access it either promgmatically or using CW Logs console.
You can use Athena to query Flow Logs stored in S3 using SQL like language.

![image](https://user-images.githubusercontent.com/33827177/149243787-42cb89c6-9ce9-4344-bc43-b4fbdc257926.png)

Visually the flow log works as show above. We have a VPC with a private (Blue) a subnet and public (Green) subnet. Our Catagram 
application server is running in the public subnet. The application's database is in the private subnet which has a primary instance
and a duplicated standby instance. Flow Logs can be at VPC, Subnet, and Interface levels.

Childs of interfaces inherit the flow low monitoring from higher groups.

The packet will always have source followed by destination.

![image](https://user-images.githubusercontent.com/33827177/149244723-76700b8c-da3b-49ac-ab70-088caf8a36db.png)

Shown above are 2 Flow Log Records. In the first one, Bob's IP address is in pink and the application servers private 
IP is in blue. The order is always source and then destination in both the IP and port nummber. Since this is a PING
we don't have source and destination port numbers directly after the source/destination IPs which is usually the case.
The Highlighed 1 is a protocol number.
The second to last item shows if traffic was accepted or rejected. This basically shows if the traffic was accepted or rejected
(i.e. whether the traffic was blocked or not by the SG and NACL). As can be show in the first Flow Log record it was
accepted. IF its a SG then only one line will show in Flow Log. Remember a SG is Stateful and so if anything is allowed the Response
is automatically allowed. But if we have a NACL on the instances subnet which allows inbound but blocks outbound then it can cause a
second line like in the case above i.e. a reject.
If in the exam we have Flow Log Records for the same traffic flow and we have an Accept followed by a Reject then it means that
a SG and NACL  has been used and they are restricting traffic.

VPC Flow Logs dont log all kinds of traffic. The following are excluded:

1. Metadata Service: Any access to the meta data service such as the one running in EC2 instance
2. Time Server Request: 169.254.169.123
3. DHCP
4. AWS DNS Server

### Egress-Only Internet Gateway

Its a type of IGW that only allows connection initiated from inside the VPC to outside

IPv4 addresses are private or public

NAT allows private IPs to access public networks. It will not allow externally
initiated connections.

![image](https://user-images.githubusercontent.com/33827177/149246291-c5a894a1-d701-44d6-9fad-890495cd80a8.png)

Using IPv6, all IPs are public. ***And NAT isn't usable with IPv6. and so how do
we restrict outbound connectivity for IPv6 instances***

Internet Gateway (IPv6) allows all IPs **in** and **out**

Egress-only is **outbound only** for IPv6. It is exactly the same as
NAT, only outbound only.

![image](https://user-images.githubusercontent.com/33827177/149246806-62a80fc5-db51-4479-bd3a-067186f7aed4.png)

To configure the Egress-only gateway, you must add default IPv6 route ::/0
added to RT with eigw-id as target.

Any Response traffic will be allowed because the Egress-Only IGW are stateful devices. What will not be allowed
is inbound traffic.

You can use a traditiona IGW for PIv4 instances with a public version IPv4 IP and IPv6 but in this case the traffic 
will be both In bound and Outbound in a bi-directional way. If you want to restrict in bound traffic for IPv6 instances
you need to use Egress_nly IGW like you would use NATGW for IPv4.

### VPC Endpoints (Gateway)

Provide private access to S3 and DynamoDB. Meaning they allow a private only resource inside a VPC or any resource
inside a private only VPC to access S3 and DynamoDB. Remember S3 and DynamoDB are public services.

Normally when you want to access a public service fomr with a VPC, you
need infrastructure and configuration to make this happen. Meaning Normally
this is an internet Gateway created and attached to the VPC. And then for the resources
inside the VPC you need to grant them a public IPv4 address and an IPv6 address or use a NATGW for instances with
private addresses to access public services.

A Gateway Endpoint allows you access these public services i.e. S3 and DynamoDB without implementing the infrastructure
just mentioned.

The way this works is that you create a Gateway Endpoint and these are created per service per region (what does it mean??).
For example, you create a Gateway Endpoint for an S3 in us-east-1 Northern Virginia region and associate with 1 or more subnets
in a particular VPC.

A Gateway Endpoint does not actually go into a VPC. What happens is that when you allocate a gateway endpoint to a subnet, a prefix 
list is added to the route table as a destination. And the target of the Prefix List is the Gateway Endpoint. A prefix List is just like what you would
find on an normal route. But its an object, a logical entity that reprsents a service like S3 or DynamoDB

Any traffic destined for S3, goes via the gateway endpoint. ***The gateway
endpoint is highly available for all AZs in a region by default. It DOES NOT
go into a VPC***

With a gateway endpoint you set which subnet will be used with it and
it will configure automatically the route on the route table for those subnets
with these prefix lists.

A gateway endpoint is a VPC endpoint object. Repeat: It is HA. Does not go into a VPC and is highly AZ

Endpoint policy is used to control what things can be accessed by the endpoint. For example a particular subset
of S3 buckets. This is an ideal solution for A private VPC and you want the instances inside the VPC to access
certain S3 buckets only.

Gateway endpoints can only be used to access services in the same region.
Can't access cross-region services. For example, can't access S3 bucket
in southeast-2 region from/through a Gateway Endpoint in use-east-1 regions. 

In Summary a Gateway Endpoint supports two use cases:

1. A private VPC and you want instances to access certain S3 or DynamoDB buckets only.

S3 can be set to private only by allowing access ONLY from
a specific gateway endpoint. For anything else, the implicit deny will apply. This will be done by
applying a bucket policy.

***Limitation of Gateway Endpoint: They are only accessible from inside that specific VPC. They are logical
GW objects and you can access logical GWs created inside a VPC from  that VPC***

![image](https://user-images.githubusercontent.com/33827177/149252580-d5836153-a050-4caa-92d7-3c8302862483.png)

The problem with giving access to private IP addresses via the NATGW is that the resources within the private instances
have access to public internet access eithe directly or indirectly. If you want resources inside the VPC to access certain
S3 buckets and not the public internet then it is not possible with NATGW. 

![image](https://user-images.githubusercontent.com/33827177/149253083-46abce21-97cb-40d8-8dd0-f913b12556e3.png)

So if you are in a highly regulated industry for which you need to create a Private VPC with no access to public internet but
require access to say an S3 bucket, then you need to use Gateway Endpoints. And so Gateway Endpoint will be associated with a subnet.
A Prefix List will be added to the Route Table attached to the subnet. The private instance within the private subnet will access the
Gateway Endpoint via the VPC router and to the S3 bucket. Note: A public IP address won't be used in this case i.e. neither through NATGW 
nor through IGW

![image](https://user-images.githubusercontent.com/33827177/149253588-9a9dafc3-804b-4290-ac5b-28632cb02fc3.png)

### VPC Endpoints (Interface)

Provide private access to AWS Public Services. So instances that are private or inside private only VPCs
Anything EXCEPT S3 and DynamoDB historically. But now S3 is supported. So for S3 you can use either Interface Endpoint
or Gatewy Endpoint.

These are not HA by default and are added to specific subnets inside a VPC. One subnet as you know means
one AZ. So if the AZ fails functionality provided by the Interface Endpoint inside that AZ fails.

Must add one endpoint for one subnet per AZ

Since Interface Endpoints are just interfaces inside the VPC and so you are able to use Security Groups to
control access to that Network Interface from a networking perspective. This is something you can't do with Gateway Endpoints

You still have the options to use Endpoint policies to restrict what can be accessed using
the Interface Endpoint.

*** Interface Enpoints currently ONLY supports TCP and IPv4 at the moment.***

![image](https://user-images.githubusercontent.com/33827177/149256608-171290e9-2e7e-466e-a1f2-f5f12110ae9f.png)

Behind the scenes, it uses PrivateLink which is a product that allows external 
(AWS or 3rd-party) services to be injected into your VPC and be given Network
Interfaces inside the VPC subnet. Private link is how Interface Endpoints operate but it is also how you can deploy
3rd party application and services directly into your VPC. This especially useful if you are in a heavily regulated 
industry where you want to access to 3rd-party services (like update services) inside a private VPC. You can do it 
without creating any additional infrastructure, you just use Private Link and inject that service's network interface
directly into the private VPC

Interface Endpoints don't work like Gateway Endpoints. They provide similar type of functionality but via a different mechanism
Gateway endpoints use a prefix list which is a logical representation of a service and the prefix list is added to the route table.
Interface Endpoints primarily use DNS. Interface Endpoints are just Network Interfaces inside your VPC. They have a private IP within 
the range of the subnet they are placed in...

The way this works is that when you create an Interface Endpoints in particular region for a particular service you get a
new DNS name for that service. And that DNS name can be used to access that service via the interface Endpoints.

e.g. vpce-123-xyz.sns.us-east-1.vpce.amazonaws.com

The above address resolved to the private IP address of the interface endpoint. 
You can configure your applications to use the above given end-point specific DNS name for accessing the service directly without requiring public
IP addressing.

Interface Endpoints are actually given multiple DNS names:

- Regional DNS Name: Which is 1 single DNS name for whatever AZ to access the interface Endpoint. Good for HA
- Zonal DNS Name: Which resolves to 1 specific interace within the 1 specific AZ.

Either of those two points of endpoints can be used

Interface Endpoints also offers PrivateDNS and what it does is associate a Route53 private hosted zone with your VPC.
This private hosted zone carries a replacement DNS record for the default service endpoint DNS name. It basically
overrides the default service DNS with a new version that points at your Interface Endpoint. This option is now enabled by default.
And it means that your application can now use Interface Endpoints without being modified?? So this makes it much easier for applications
running a VPC to use interface Endpoints (But How??) 

Without using Interface Endpoints accessing a service like SNS from within a public subnet inside a VPC would work like this: The instance using the SNS
would resolve the default service endpoint which is sns.useeast-1.amazonaws.com. It would resolve this name to a public space IP address (instance)
and the traffic would be routed via the VPC router via the IGW and out to the SNS service.
Private Instances would also work like this but without having access to a public IP address they wouldnt be able to get past the IGW

![image](https://user-images.githubusercontent.com/33827177/149259084-92cd49a9-9cbe-41f9-94c3-2dcceef23eee.png)

IF we change this architecture and add a Interface Endpoint, then if private DNS isn't used the services which continue to use the service default DNS
would still leave the VPC via VPC router and then the IGW and so on. But the services which choose to use the endpoint specific DNS name, they would resolve 
that name to the interface endpoint's private address. The endpoint is a private interface to the service that is configured for in this case SNS. ANd so the 
traffic could then flow via the interface endpoint service without requiring IPv4 public addressing. Its as if the SNS service has been injected into the VPC in
a secure way.

![image](https://user-images.githubusercontent.com/33827177/152273469-b461efe9-b893-425d-9d8c-c26beb7454de.png)

If we utilize private DNS, it makes it even easier...The private DNS replaces the services default DNS. So even clients which haven't been reconfigured to use the
endpoint specific DNS (and so they are still using service default DNS name) will not go via the interface endpoint. So in the example below using private DNS overrides
the default SNS service endpoint name. So sns.us-east-1.amazonaws.com rather than resolving to a public IP address belonging to the SNS service, its overriden and it will
now resolved to the 'private IP address of the Interface Endpoint'. The advantage is that is that services that cannot be modified to use the endpoint specific DNS name will
also utilize the interface endpoint

![image](https://user-images.githubusercontent.com/33827177/152274590-d51d08a8-cdca-4595-8c75-82d3e5889a09.png)

### Exam Top Up
1. Gateway Endpoints work using prefix lists and route tables so they never require making changes to the applications. The application thinks that it is essentially communicating directly with S3 or DynamoDB. All we are doing by using a GW endpoint is the route that traffic flow uses. Instead of using an Internet GW and requiring a public IP addresses ot goes via a GW endpoint and keeps using a Private IP address.
2. Interface endpoint uses a DNS and a private IP address for the interface endpoint. You can use the endpoint specific DNS name which would require to make changes to the application so that it uses the DNS name (which would resolve to the private address of the Interface Endpoint to access the public service like SNS) or you can enable private DNS which overrides the default and allows unmodified applications to access the services using the Interface Endpoint. Interface Endpoint don't use routing, they use DNS. The DNS name resolves to the private IP address of the interface endpoint and that is used for connectivity with the public service

### VPC Peering
VPC Peering a service that lets you create a Direct Encrypted Network link between two VPCs.

***One peering connections only links two and only two VPCs.***

Works in the same or cross region (Some limitations when connecting cross region VPCs) and in the same or across accounts.

VPC peers have the option to allow Public Hostnames to resolve to
private interal IP addresses.

Same region VPC can reference each other by using  SG ID. So you can do the same efficient refercing and nesting that you can do when two isntanes are in the same VPC.
This allows for the same nesting of security groups within that region. Only work with same Region VPCs though

In different regions, you can still utilize SGs but you will need to reference IP addresses ir IP ranges.

IF VPC peers are within the same region then you can do the logical referencing of the entire SG and that massively improves efficiency of the security of VPC peers.

VPC peering connects **ONLY TWO**

VPC Peering ***DOES NOT*** support **transitive peering**

If you want to connect 3 VPCs, you need 3 connections. You can't route
through interconnected VPCs. i.e. if you have a VPC A peered to VPC B and the VPC B peered to C that does not mean that VPC A and C are also peered.

When you create a VPC peering connection between two VPCs, you are creating a logical gateway object inside of both of the VPCs. And to fully configure 
connectivity between thoe VPCs you need to configure routing i.e. route tables with routes on them pointing at the remote VPC IP address range and using the 
VPC peering connection gateway object as the target.

To ensure free flow of traffic b/w VPC we need to ensure that SGs and NACLs are configured to allow the flow

***VPC Peering Connections CANNOT be created with overlapping VPC CIDRs***

![image](https://user-images.githubusercontent.com/33827177/149428512-5db2209c-a1c9-47d3-8f2e-6dbea1753da8.png)

When a VPC peering connection is established between 2 VPCs a logical gateway object is created inside each VPC. Step 2 is to configure routing tables within each VPC and associate these tables with subnets. These routing tables have the remote VPC CIDR and as the target the VPC peering gateway object. So when instance in VPC A wants to communicate with an instance in VPC B it would send the traffic to VPC Logical Gateway object. If we want to allow bi-directional communication this configurations of route tables + targets would need to be done in all route tables of the VPCs.



