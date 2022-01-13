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

Provide private access to S3 and DynamoDB.

Normally when you want to access a public service through a VPC, you
need infrastructure to make this happen.

When you allocate a gateway endpoint to a subnet, a prefix list is added
to the route table.

The target is the gateway endpoint.

Any traffic destined for S3, goes via the gateway endpoint. The gateway
endpoint is highly available for all AZs in a region by default.

With a gateway endpoint you set which subnet will be used with it and
it will configure automatically.

A gateway endpoint is a VPC endpoint object.

Endpoint policy is used to control what things can be accessed by the endpoint.

Gateway endpoints can only be used to access services in the same region.
Can't access cross-region services.

S3 buckets can be set to private only by allowing access ONLY from
a gateway endpoint. For anything else, the implicit deny will apply.

They are only accessible from inside that specific VPC.

### VPC Endpoints (Interface)

Provide private access to AWS Public Services
Anything EXCEPT S3 and DynamoDB

These are not HA by default and are added to specific subnets.

Must add one endpoint for one subnet per AZ

Network access controlled via security groups.

You can use Endpoint policies to restrict what can be accessed with
the endpoint.

ONLY TCP and IPv4 at the moment.

Behind the scenes, it uses PrivateLink.

Endpoint provides a **NEW** service endpoint DNS

e.g. vpce-123-xyz.sns.us-east-1.vpce.amazonaws.com

- Endpoint : Regional DNS
- Endpoint : Zonal DNS

Either of those two points of endpoints can be used

PrivateDNS overrides the default DNS for services.

### VPC Peering

Direct encrypted network link between two VPCs

Works in the same or cross region and in the same or across accounts.

VPC peers have the option to allow Public Hostnames to resolve to
private IPs.

Same region SG's can reference peer security groups. This allows for the
same nesting of security groups within that region.

In different regions, you need to reference IP peers.

VPC peering connects **ONLY TWO**

VPC Peering does not support **transitive peering**

If you want to connect 3 VPCs, you need 3 connections. You can't route
through interconnected VPCs.

You are creating a logical gateway object in one VPC.

VPC Peering Connections CANNOT be created with overlapping VPC CIDRs
