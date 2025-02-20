## HA and Scaling
### Global Service Location and Discovery: When you type in Netlflix dot com into you browser, what happens? How does you machine figure where to point at?
![image](https://user-images.githubusercontent.com/33827177/146112947-bd3ca45d-ba89-4004-8bbd-e3bc97943fe7.png)
DNS Part: On AWS DNS component is implemented via route 53. Route 53 is flexible and can be configured in any number of ways.
Netflix client will use the DNS for initial service discovery. Netflix would have configured the DNS to point at one or more service entry points i.e primary and secondary. Let's say there is a primary location for Netflix in the US and if that fails Australia will be used a secondary. Another valid configurationc can be to send customer to the nearest
location. 

### Content Delivery and Optimization: How does the content or data of an application get delivered in an optimized way, globally?
Content Delivery Network delivery method can be used to cache content globally as close to the users as possible to improve performance. The locations from the global location as and when required

### Global Health Check & Failover: Detecting if Infrastructure in one location is healthy and moving the customers to new location as or when required
The AWS global architecture has health checks which can determine if a location is healthy and direct all traffic to the primary if everything is OK or direct customers to
the secondary in case of problems.

Assuming the user is directed to the US location for application's content/data consumption. The traffic will enter one specific region of AWS. Depending on the architecture
this might be entering into a VPC or public space AWS services. The purpose of the web tier is to act as an entry point for regional application component

### Reginonal Entry Point
Initially communication or traffic from the customer will enter using the web tier. Generally, this will be a Regional based AWS service Application Load Balancer or API gateway
depending on the architecture the application uses. The purpose of the web tier is to act as an entry point for regional application component.
![image](https://user-images.githubusercontent.com/33827177/146115377-766059b0-ef09-45ec-b008-f086882ebdbc.png)

### Scaling and Resilience

### Application services and components
The functionality provided to the customer via the web tier of is provided by the compute tier using servides like EC2, Lambda or Elastic Container Service. The computer tier would consume storage like EBS, EFS or S3(for media storage)
You will also find that many global architecture utilizing CloudFront. The global content delivery network within AWS. And CLoudFront is capable of using S3 as an origin for media. So S3 can store media such as movies and these might be cached by CloudFront
In additon to file storage an application may require data storage like RDS, Aurora, DynamoDB, or RedShift. But in order to improve performance most applications dont access the database, instead they go via a caching layer through applications like Elastic Cache for general caching or DyanmoDB Accelerator DAX. Only access database if data isn't available on the caching application. Accessing/Reads from Cache is cheap as compared to databases. Also better performance

### ELB Evolution
Classic Load balancers can load balance http, https as well as lower level protocols but they aren't layer 7 devices. They dont understand http and they can't make decisions on http protocol features. For example, Classic CLB only support 1 SSL per CLB so larger deployments you might user 100s or 1000s load balancer. This can be consolidated to 1 single v2 load balancer. In exam, default to v2 load balancer
![image](https://user-images.githubusercontent.com/33827177/146115727-4f804ff4-4f34-4ac9-9c76-4e7e1efab9fb.png)

Application Load Balander (ALB) -v2 are truly layer 7 devices
![image](https://user-images.githubusercontent.com/33827177/146116224-2e9c1358-0b3e-4612-ad19-db75f9ab3b02.png)

### Load Balancer
It is the job of  a load balancer to accept connections from the customers and then distribute to backend compute. It means the user is abstracted away from the physical infrastructure. So the amount of infrastructure can change(increase/decrease) or fail and repaired without the customer knowing
### ELB Architecture
We have a VPC with 2 AZs. AZ A and AZ B. We have got subnets some public and some private. We have a Bob a user and 2 load balancers. The load balancer accepts connection from the user. When provisioning a load balancers, you have to decide a bunch of configurations one of them is to accept both IPv4 and IPv6 or just IPv4. We also need to pick the AZ the load balancer would use. Specifically, you will be picking 1 subnet in each AZs. Based on the subnets you pick the when you provision a Load balancer, it places one or more LB nodes in each nodes. The way you pick which AZ goes into is by picking one and only one of the subnet in that specific AZ. 

Incoming load is distributed equally to the individual nodes of the LB. These nodes scale so if one fails another one is provisioned if traffic increases additional LB nodes are provisioned inside each AZ

Aanother choice while configuring the LB, you need to choose whether it would be internal or internet facing. This choice controls the IP addressing of the LB nodes. If you choose public then the nodes are given private and public addresses. Only private ip addresses if private facing is chosen.

Once connection to the LB nodes is made, the LB nodes can make connections to instances that are registered with the LB.
![image](https://user-images.githubusercontent.com/33827177/146276016-be9cbbbd-f7db-4ceb-8fa0-3876d9115069.png)

A LB requires 8 free IPs to functions, so with the 5 reserved IPs for AWS we are looking for a /28 subnets (11 free IPs) although aWs suggest /27 for scaling. So if in exam there is no option with /27 then /28 is the right choice for the minimum number of IPs and subnet.

Internal load balancers work the same like internet facing LBs, they are used to separate application tiers.
![image](https://user-images.githubusercontent.com/33827177/146277868-47ac6309-5bba-452a-bbe6-9a2d6b026572.png)

### ELB Architecture Example:
We have a multi-tier Application. Its inside a VPC and inside that an AZ. We have an internet facing load balancer. Then a Web Autoscaling Group providing the front-end capability of the App. Then another LB which is internal (with only private IPs) and then an APP ASG. And then Aurora DB. 
![image](https://user-images.githubusercontent.com/33827177/146276941-ba809f7f-de45-4cf8-8344-ae169cef9558.png)
Without a LB the instances need to have awareness of each other. e.g. Bob would need an awareness of connecting to a specific web instance which will be 

Bob doesnt know which Web instance is he connecting with caz he connects with them through LB nodes

### Cross Zone Load Balancing
![image](https://user-images.githubusercontent.com/33827177/146277338-a588b7a3-3072-4fb4-8cae-3c24536cc082.png)

### Load Balancer Consolidation
![image](https://user-images.githubusercontent.com/33827177/146279725-4de277e9-d661-4d15-9795-2cc1e1138f0d.png)

### Application Load Balancer
![image](https://user-images.githubusercontent.com/33827177/146280024-397ef4c0-d39f-4833-b094-0d5ba1009dc3.png)

![image](https://user-images.githubusercontent.com/33827177/146280607-64974415-aa4f-4536-9283-344089b983aa.png)

IF you need an end to end encrypted connection to your application without the connection getting terminated at 
the LB then we need to use Network LB instead of Application LB. The network balancer can configure a listener 
which forwards TCP traffic directly to instances and can thus have unbroken encryption. 

![image](https://user-images.githubusercontent.com/33827177/146280918-4ef92fdc-a945-4393-b780-4bdc61bb17c6.png)

![image](https://user-images.githubusercontent.com/33827177/146281490-332c91d6-722e-44bf-969c-3bdf85f80647.png)

![image](https://user-images.githubusercontent.com/33827177/146281550-baf56b17-63c8-49eb-808b-277e2e081c5b.png)

### Launch Configurations and Lanuch Templates
![image](https://user-images.githubusercontent.com/33827177/146281890-ff090f64-1950-4b5c-bbfc-53c9c45566e5.png)

![image](https://user-images.githubusercontent.com/33827177/146282015-063ee0e9-0ebc-47f2-859a-98be52cbc58b.png)

### Load Balancing Fundamentals. 

Without load balancing, it is difficult to scale.

The user connects to a load balancer that is set to listens on port 80 and 443

Within AWS, the ports the load balancer will listen to is called
the **listener**.

The user is connected to the load balancer and not the actual server.

Behind the load balancer, there is an application server. At a high
level when Bob connects to the load balancer, it distributes that to
servers on the application server.

As long as 1+ servers are available, the LB is operational. Clients
shouldn't see errors.

#### LB Exam Powerup

Clients connect to the **listener** of the load balancer

The load balancer connects to one or more **targets** or servers

Listener connection - one connection between the client and listener
Backend connection - one connection between load balancer and backend instance

Client is abstracted away from individual servers

Used for high availability, fault tolerance, and scaling

### Application Load Balancer (ALB)

ALB is a layer 7 - it is capable of inspecting data that passes through
it and can understand the application layer

It can take action based on things from that protocol

These are scalable and highly available.

Internet facing or Internal

Internal load balancer is used for inside a VPC only

Listens on the outside and sends to target groups

#### Cross zone load balancing

Each node that is part of the load balancer is able to distribute load
even if its not in the same AZ. It is the reason we can achieve a balanced
distribution of connections behind a load balancer.

It can also provide health checks on the target servers.

If all instances are shown as healthy, it can distribute evenly.

ALB can support a wide array of targets. An individual target can be a
member of multiple groups.

#### ALB Exam Powerup

**Targets** are lambda functions or EC2 instances that are directed towards.

Rules are path based or host based.

Support EC2, EKS, Lambda, HTTPS, HTTP/2 and websockets

ALB can use SNI for multiple SSL certs - host based rules

AWS does not suggest using Classic Load Balancer (CLB), these are legacy.

### Launch Configuration and Templates

LC and LT key concepts. They are documents which allow you to config an EC2
instance in advance. You can configure userdata and IAM role along with
networking and security groups.

Launch templates
- Provide T2/T3 Unlimited, placement groups with more.  
- Newer and recommended to use over launch configurations, they include the latest features and improvements.
- Supports versioning of templates.
- Can be used to save time when provisioning EC2 instances from the console UI / CLI.

If you need to adjust a configuration, you must make a new one and launch it.

### Auto Scaling Groups

ASG provides Automatic scaling and self-healing for EC2. Can be alsoed used to implement self-healing architecture as part of the scaling or in isolation

They make use of configurations defined within Launch Templates or Launch Configurations to know what to provision.

They use one version specific version of a configuration they're assigned with. You can assign either a LT or LC but not both

Minimum, Desired, and Maximum Size.

Provision or Terminate instances to keep at the desired capacity

Scaling Policies automate based on metrics like CPU load or a schedule
![image](https://user-images.githubusercontent.com/33827177/146287677-1b984667-414a-40f7-92c2-40bdc1ebadf0.png)

Auto Scaling Groups will try to keep the AZs equal with the number of EC2
instances.

#### Scaling Policies

Manual Scaling - manually adjust the desired capacity

Scheduled Scaling - time based adjustments e.g. Sales or known period of high/low usage. In exam if you have know periods of high/low usage Scheduled Scaling can be the potential answer

Dynamic Scaling

- Simple : If CPU is above 50%, add one to capacity. iF below 50 removwe 1
- Stepped : If CPU usage is above 50%, add one, if above 80% add three. And the same in reverse
- Target Tracking: Desired aggregate CPU = 40%, ASG will achieve this. Not all metrics for target tracking. Some that do are avg cpu utilizatiion, etc.

Cooldown Period - How long to wait at the end of a scaling action before scaling again. Helps avoid chaotic changes and billing due to minimum billing periods

Always use cool downs to avoid rapid scaling.

AGS can use the load balancer health checks rather than EC2. If an EC2 instance fails. The ASG terminates it and provisions another one. Called Self-Healing

Autoscaling Groups are free  

Think about more, smaller instances to allow granularity and cost reduction

You should use ALB with autoscaling groups - abstraction and elasticity

ASG defines WHEN (instances are launched) and WHERE (which sunnets are launched into). Launch Template defines WHAT ( whatisntances are launched and what configuration).

### ASG + Load Balancers
The real powder of ASG comes from their ability to integrate with Load Balancers

![image](https://user-images.githubusercontent.com/33827177/146288990-b6133c41-fa6f-4c2b-a7a9-5fe8738f3578.png)
You need to be careful while using Application Load Balancer Health Checks. For example, if your application has complext logic in it and you are checking a static HTML page then health check may respond as Okay even though the Application is failing. Inversely, if your application uses Database and your health checks a page with some DB requirements and the database fails the health check can return a fail state even though the DB not the application has failed. Your EC2 instance would be reprovisioned
![image](https://user-images.githubusercontent.com/33827177/146289374-80d845a7-566f-4ad1-b76d-dde53989dd9c.png)

### ASG SCALING POLICIES
![image](https://user-images.githubusercontent.com/33827177/146291342-c8f10168-1cd8-4e01-8069-a5accc76b472.png)

![image](https://user-images.githubusercontent.com/33827177/146291482-23573f84-c432-4c45-94d4-d676f9e887fd.png)
Not Flexivlw A RHW Amw 2 instances are added or removed irrespective of high a jump above/below 50%

![image](https://user-images.githubusercontent.com/33827177/146291819-7aa16fc8-58e8-4832-82a7-ffed939548d5.png)

### ASG LifeCycle Hooks
![image](https://user-images.githubusercontent.com/33827177/146292902-786b4c52-fede-4543-a019-4bf6cf328a81.png)

![image](https://user-images.githubusercontent.com/33827177/146293114-0e91acce-3efa-43dd-b59c-ba535df95d68.png)

### ASG HealthCheck Comparison

![image](https://user-images.githubusercontent.com/33827177/146294847-99a12849-6c74-444b-9309-aadf41857c58.png)
The grace period needs to be sufficiently lolong to allow system launch, bootstrapping and application start. Otherwise, what will happen is that if the grace period isn't sufficiently long you can be in a situation where the Health Check starts before the application has finished configuring. This would result in it being viewed unhealthy terminated. And this would be repeated until a sufficiently long graceperiod is chosen

### Scaling Processes
Launch & Terminate: IF Launch is set to SUSPEND then the application won't scale out. If termiante is set to SUSPEND then application won't terminate
![image](https://user-images.githubusercontent.com/33827177/146289687-11fb753f-6a0d-4025-8dd8-930bffc1dee1.png)
Standby - Won't get effected by what the ASG does.
### Network Load Balancer (NLB)

Part of AWS Version 2 series of load balances.

NLB's are Layer 4, only understand TCP and UDP.

Can't understand or interpret HTTP or HTTPs, for these reason they are much
faster in latency. You should default to NLB if http is not used.

There is nothing stopping NLB from load balancing on HTTP just by data.

Rapid scaling - **millions of requests per second**

Only member of the load balancing family that can be provided a static IP.
There is 1 interface per AZ. Can also use Elastic IPs (whitelisting)

Can do SSL pass through.

NLB can load balance non HTTP/S applications, doesn't care about anything
above TCP/UDP. This means it can handle load balancing for FTP or things
that aren't HTTP or HTTPS.

### SSL Offload and Session Stickiness

#### Bridging - Default mode

One or more clients makes one or more connections to a load balancer.
The load balancer is configured so the **listener** uses HTTPS, SSL connections
occur between the client and the load balancer.

The load balancer then needs an SSL certificate that matches the domain name
of the application.

If you need to be careful of where your certificates are stored, you may
have a problem with this system.

ELB initiates a new SSL connection to backend instances with a removed
HTTPS certificate. This can take actions based on the content of the HTTP.

It needs to decrypt any data that is being encrypted by the client.

The EC2 will need matching SSL certificates.

Needs the compute for the cryptographic operations. Every EC2 instance must
peform these cryptographic operations.
![image](https://user-images.githubusercontent.com/33827177/146297467-afb99a47-e050-46a1-a631-23d3dabd8a9f.png)
#### Pass-through

The client connects, but the load balancer passes the connection along without
decrypting the data at all. The instances still need the SSL certificates,
but the load balancer does not.

The load balancer is configured for TCP, it can see the source or destinations,
but it never touches the encrypted connection. The certificate never
needs to be seen by AWS.

Negative is you don't get any load balancing based on the HTTP part
because that is never exposed to the load balancer.

#### Offload

Clients connect to the load balancer using HTTPS and are terminated on the
load balancer. The LB needs an SSL certificate to decrypt the data, but
on the backend the data is sent via HTTP. While there is a certificate
required on the load balancer, this is not needed on the LB.

Data is in plaintext form across AWS's network. Not a problem for most.

#### Connection Stickiness

If there is no stickiness, each time the customer logs on they will have
a stateless experience. If the state is stored on a particular server,
sessions can't be load balanced across multiple servers.

Session Stickiness is an option. If enabled, the first time a user makes a
request, the load balancer generates a cookie called AWSALB. A valid duration
is between one second and seven days. For this time, sessions will be sent to
the same backend instance. This will happen until:
![image](https://user-images.githubusercontent.com/33827177/146298184-e6c61954-100a-4a56-bb56-5837fa241967.png)
- If we have a server failure, then the user will be moved to a different
server.
- The cookie could expire, the whole process will repeat and will recieve a
new cookie and the process will start again.

The proble with this method is that it could cause uneven load on backend because irrespective of the load on say EC2-2 server the user will still sent to the EC2-2 server.
Applications should therefore be designed to use stateless servers meaning holding the session or user state somewhere else externally, like DynamoDB. If this method is used the EC2-2 instance would be stateless and the load balancer would be able to load balance without using cookies in a fair way.

### Gateway Load Balancer
![image](https://user-images.githubusercontent.com/33827177/147007954-d7c71f2e-3b8b-431e-ae01-729c0973198c.png)

AWS has networking products that can do what a GWLB does but a company might be required to use a 3rd-party networking and/or security product (for various like in-house expertise, formal requirements, or a specific which only a specfic vendor's hardware has. So in that case you will use a 3rd-party device with AWS and to do that in a manageable way you need a GLWB.

GWLB endpoints are like VPC interface endpoints with some key improvements. 

The GWLB balances packets aross multiple instances. These are normal EC2 instances running security softwares. The GWLB sends and receive packets to the security devices without any alterations. The security device reviews packets as they are sent/received. The source or destination on the packets might have source or destination IPs which might work on the original network but not on the network on which the security appliances are hosted. So the GWLB uses a protocol called GENEVE
![image](https://user-images.githubusercontent.com/33827177/147008860-10968cd5-2a92-49df-b9d9-70d2c4bb7b95.png)

The GWLB endpoint is different than the VPC endpoint in that it can be added to a route table as a next hop

The gateway load balancer ensure flow stickiness. So one flow of data will always use one appliance so it allows that appliance to monitor the state of the packet. Provides reselience by providing multiple security appliance so if one fails packets are moved over to another security appliance

![image](https://user-images.githubusercontent.com/33827177/147009354-3bcfd1ef-4435-439d-b7ea-6d427e9803f9.png)

We have a Catagram Application running in a pair of private subnets, behind and Application Load Balander which is running in pair of public subnets. We have a separate VPC with a pair of security appliances inside an auto-scaling group so they can grow and shrink based on load. This is how it works

1. We start with a client which is accessing the catagram.io app. It first the internet gateway which has a gateway route table. 
2. The gateway sends the traffic to the GWLB endpoint
3. The traffic is send to the GWLB itself. The packet has original IP addresses. The packets are encapsulated using GENEVE and sent to security appliance.
4. Packets are send back to the GWLB the encapsulation is stript and the packet is sent to the GWLB endpoint
5. Packets sends packets to ALB and then the application

![image](https://user-images.githubusercontent.com/33827177/147009696-64123d85-2cd2-41a6-861e-955b0d5144ad.png)

The sameway it works on return journey

![image](https://user-images.githubusercontent.com/33827177/147010543-3fc21649-b6b9-449b-8fdb-8ba0ca86a48c.png)

