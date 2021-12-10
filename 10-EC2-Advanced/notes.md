## Advanced EC2

### Bootstrapping EC2 using User Data

Bootstrapping is a process where scripts or other config steps can be run when
an instance is first launched. This allows an instance to be brough to service
in a particular configured state.

In systems automation, bootstrapping allows the system to self configure
or perform some self configuration steps. In AWS this is
**EC2 Build Automation**.

This could perform some software installs and post install configs.

Bootstrapping is done using user data - accessed via the meta-deta IP

<http://169.254.169.254/latest/user-data>

Anything you pass in is executed by the instance OS. 
### Exam Powerup: User Data is excecuted only once on launch. If you update the User Data and restart the instance it won't be executed again.
EC2 doesn't validate/interpret the user data. You can tell EC2 to pass in delete the data on the Boot Volume and the data will be passed ad there is a process on the OS which run it as a root user
The OS needs to understand the user data.

#### Bootstrapping Architecture

An AMI is used to launch an EC2 instance in the usual way to create
an EBS volume that is attached to the EC2 instance. This is based on the
block mapping inside the AMI.

Now the EC2 service provides some userdata through to the EC2 instance.
There is software within the OS running on the EC2 instance which is designed 
to look at the metadata IP for any user data. If it sees any user data, it executes
this on launch of that instance.
![image](https://user-images.githubusercontent.com/33827177/144942423-266cd90c-80e9-4b33-9488-eec532417304.png)
This is treated like any other script the OS runs. At the end of running
the script, the instance will be in either:

- Running state and ready for service
- Bad config but still likely running
  - The instance will probably still pass its checks
  - It will not be configured as you excpected

#### User Data Key Points

EC2 doesn't know what the user data contains, it's just a block of data.

The user data is not secure, anyone can see what gets passed in. For this
reason it is important not to pass passwords or long term credentials.

User data is limited to 16 KB in size. Anything larger than this will
need to pass a script to download the larger set of data.

User data can be modified if you stop the instance, change the user
data, then restart the instance.

The contents are only excecuted once at launch.

#### Boot-Time-To-Service-Time

How quickly after you launch an instance is it ready for service. This
includes the time for EC2 to configure the instance and any software
downloads that are needed for the user.

When looking at an AMI, this can be measured in minutes.

AMI baking will front load the time needed.
![image](https://user-images.githubusercontent.com/33827177/144943524-6fa0248f-a464-4fcd-972e-85f17023e407.png)
The optimal way is to use AMI baking and Bootstraping. When you have an application installation process which is 90% installation and 10% confugration, then you should
AMI bake the 90% part and Bootstrap the 10% confugration part

### AWS::CloudFormation::Init

**cfn-init** is a helper script installed on EC2 OS.

This is a simple configuration management system.

Procedural (User Data) vs Desired State (cfn-init)...cfninit can be also procedural and run commands line by line like user data

Ti can make sure Packages are installed even with awareness of specific versions. It can manipulate OS groups and users. It can download sources and extract them onlto the local isntance even using authentication. It can create files with certain content , permission and ownerships. It can run commands and test if certain conditions are true after the commands ahve run. It can even control services on instance like ensuring a service started at the boot of the instance or enabled to be started.

This is executed as any other command by passing into the instance with
the user-data.

This is provided with directives via Metadata and AWS::CloudFormation::Init on
a CFN resource

#### cfn-init explained

Starts off with a **cloud formation template**
This has a logical resource within it which is to create an EC2 instance.

This has specific section called `Metadata:` and "AWS::CloudFormation::Init" and this where the cfn:init configuration is stored.

The cfn:init command itself is executed from the user data which is passed into the instance

![image](https://user-images.githubusercontent.com/33827177/144954512-fdbce591-b212-4d40-ae57-e3e54591a150.png)

cfn-init gets variables passed into the userdata by cloud formation

It knows the desired state desired by the user and can work towards a
final stated configuration

This can monitor the userdata and change things as the EC2 data changes unlike the user data which executes once. If the metadata changes cfn:init will update instance...

#### CreationPolicy and Signals

The template has a specific part designated signals. Without Signals the resource creation process inside CloudFormation is dumb. For example, you have a template which is used to create a stack which creates an EC2 instance, let say you pass some User Data which is run and then the instance is marked as complete. The problem we don't know if the resource actually completed successfully in cases we are doing extra configuration through User Data. CloudFormation has created the resource and passed in the user data but CF doesnt understand the User Data. If there is a problem with user data if the boot strapping doesn't work CF wouldn't know but the instance would be marked as complete.

![image](https://user-images.githubusercontent.com/33827177/144958124-59b08880-e619-4fa6-9885-cc4c933299b7.png)

A creation policy is added to a logical resource inside CF template. It is provided a
timeout value say 15 minutes as given below. A template is used to create a stack which creates an EC2 instance but at this point CF waits. It doesn't move the instance into create complete state when EC2 signals that its been created successfully. Instead it wait for a signal from the resource itself. The 'cfn-signal' command is given the stackid, the resource and the region.

The resource itself will trigger a signal that cloud formation if the resource creation has been done successfully or there was an error.
### EC2 Instance Roles

IAM roles are the best practice ways for services to be granted permissions.

Roles that an instance can assume to grant permissions for resources within.

Starts with an IAM role with a permissions policy.

EC2 instance role allows the EC2 service to assume that role but we need someway of delivering those credentials into the EC2 instance.

The **instance profile** is the item that allows the permissions inside
the instance.

When you create an instance role in the console, an instance profile is created in the same name but if you use the command line or 
CloudFormation or Command Line you have to create these two things separately
![image](https://user-images.githubusercontent.com/33827177/145029541-92b6c936-57a0-48e8-b1ef-94595a391425.png)
When IAM roles are assumed, you are provided temporary roles based on the
permission assigned to that role. These credentials are passed through
instance **meta-data**.

EC2 and the secure token service ensure the credentials never expire.

Key facts

- Credentials are inside meta-data
- Specifically inside iam/security-credentials/role-name
- automatically rotated - always valid as long as the instance role attached to the instance
- the resources need to check the meta-data periodically
- should always use roles compared to storing long term credentials
- CLI tools use role credentials automatically

### AWS System Manager Parameter Store

Passing secrets into an EC2 instance is bad practice because anyone
who has access to the meta-data has access to the secrets.

Parameter store allows for storage of **configuration** and **secrets**

Parameter Store lets you create parameters which has name and a value. Value is the part stat
stores the actual configuration.

Many AWS services integrate with the Parameter Store natively.

Parameter Store offers to store three difeerent kinds of paramters:

- Strings
- StringList
- SecureString

You can store license codes, database (connection) strings like hostnam or ports, and full configs and passwords.

Parameter store allows for hierarchies and versioning.

It can store plaintext and ciphertext. This integrates with **kms** to
encrypt passwords.

Allows for public parameters such as the latest AMI parameter to be stored
and referenced for EC2 creating.

This is a public service so any services needs access to the public space end points or
to be an AWS service.

Applications, EC2 instances, lambda functions can all request access to
parameter store.

This is tied closely to IAM and could use long term credentials such
as access keys, or short term use of IAM roles.
![image](https://user-images.githubusercontent.com/33827177/145036562-8e066420-0400-4735-9758-a8609fcac8fc.png)
If the parameters are encrypted then KMS will be involved and appropriate permissions to the CMK inside the KMS will be required

Allows for simple or complex sets of parameters. For example, myDBpassword, or create hierarchical structures like /wordpress/ which might DBUser and DBPassword inside it and can be accessed by /wordpress/DBUser and /wordpress/DBPassword respectively.

Permissions are flexible and they can be set on individual parameters or whole trees. It supports versioning. Changes occuring to parameter can/may spawn events which can start processes in other AWS services.
### System and Application Logging on EC2

Cloudwatch monitors the outside metrics of an instance.
Cloudwatch logs is for logging

Neither natively capture data inside an EC2 instance.

A CloudWatch Agent is required for inside data i.e. OS visible data. It sends this data into CW or CloudWatch Logs

For CW to function, it needs configuration and permissions in addition
to having to install the cloud watch agent.

The cloudwatch agent needs to know what information to inject
into cloud watch and cloud watch logs.

The agent needs some permissions to interact with AWS. This is done with an
IAM role as best practice. The IAM role has permissions to interact
with CW logs. The IAM role is attached to the instance which provides
the instance and anything running on the instance, permissions to manage
CW logs.
![image](https://user-images.githubusercontent.com/33827177/145138499-1d4a3881-0d58-468c-a396-9a0bfdd3d1d0.png)
The data requested is then injected in CW logs.

There is one log group for each individual log we want to capture

And then within each log group, there is one log stream for each instance that performs this logging or is injecting that data

To do this for one instance, we can do it manually i.e. log in to the instance, install the Agent, configure it, attach a role and start injecting the data.

But to do it at scale, you will need to autmate the process and use CloudFormation to use the Agent Configuration for every instance you provision.

We can use parameter store to store the configuration for the CW agent.

### EC2 Placement Groups
Normally when you launch an EC2 instance its physical location is selected by AWS with placing in an EC2 host makes the most sense within an AZ its launched in
Placement Groups allows you to influence placements ensuring placements are either physically close together or not depending your preference
#### Cluser - Pack instances close together

Achieves the highest level of performance available with EC2.

Best practice is to launch all of the instances within that group at the
same time.

If you launch with 9 instances and AWS places you in a place with capacity
for 12, you are now limited in how many you can add.

Cluser placements need to be part of the same AZ. The idea with cluster
placement groups are generally the same rack, but they can even be the same
EC2 host.
![image](https://user-images.githubusercontent.com/33827177/145561056-01b78269-8d2c-4cb8-b152-7c94d74fd2b7.png)
All members have direct connections to each other. They can achieve
10 Gbps single stream vs 5 Gbps normally. They also have the lowest
latency and max PPS possible in AWS.

If the hardware fails, the entire cluster will fail.

##### Cluster Exams

Clusters can't span AZs. The first AZ used will lock down the cluster.

They can span VPC peers - but impacts performance

Requires a supported instance type. Cluster Placement Groups are not support by every type of instance.

Best practice to use the same type of instance and launch all at once.

This is the only way to achieve **10Gbps SINGLE stream**, other data metrics
assume multiple streams.

#### Spread - Keep instances seperated

This provides the best resillience and availability.

Spread groups can span multiple AZs. Information will be put on distinct
racks with their own network or power supply. There is a limit of 7 instances
per AZ. The more AZs in a region, the more instances inside a spread placement
group.
![image](https://user-images.githubusercontent.com/33827177/145561548-f5bb04e6-7834-41fb-a9a8-b7077acbc6c7.png)
##### Spread Exams

Provides the highest level of availability and resillience. Each instance
by default runs from a different rack (Each rack has its own network and power source).

7 instances per AZ is a hard limit.

Not supported for dedicated instances or hosts.

Use case: small number of critical instances that need to be kept seperated
from each other. Several mirrors of an application or file server.

In short, if you want to maximize the availibility of your application go with Spread Placement Groups

#### Partition - groups of instances spread apart

Instances can be placed into a specific parition, or have EC2 make that decision on your behalf.
![image](https://user-images.githubusercontent.com/33827177/145563542-1ee0f9f5-94ae-4eca-bfbc-5c5abc619024.png)
If you launch 10 instances into 10 different partition and a problem occurs with one partition's 
networking or power, it will at most take out one instance. If you launch 10 into a single instance
then you will lose all if a problem occurs.

The main difference is you can launch as many instances in each partition
as you desire.

When you launch a partition group, you can allow AWS decide or you can
specifically decide.

They are designed for huge scale parallel processing systems

Partition Placement Groups provide visibility into the partion 
which enables us to see which instance is in which partition and 
we can share this information with topology aware applications like HDFS, HBase, and Cassandra

##### Parition Exams

7 paritions maximum for each AZ

Instances can be placed into a specific parition, or AWS can pick natively.

This is not supported on dedicated hosts.

Great for HDFS, HBase, and Cassandra

### EC2 Dedicated Hosts

EC2 host allocated to you in its entirety.

Pay for the host itself which is designed for a family of instances. e.g. a1, c5, m5

No instance charges...you pay for the whole capacity of the host

You can pay for a host on-demand or reservation with 1 or 3 year terms.

The host hardware has physical sockets and cores. This dictates to how
many instances can be run.
![image](https://user-images.githubusercontent.com/33827177/145564191-15c31fcc-bdcb-4586-b277-eef60a4e9bc1.png)
Hosts are designed for a specific size and family. If you purchase one host, you
configure what type of instances you want to run on it. With the older
system you cannot mix and match. 

The new nitro system allows for mixing and matching host size.
![image](https://user-images.githubusercontent.com/33827177/145564396-23f8dda3-a6d9-4de4-9596-70f35bee1886.png)
#### Dedicated Hosts Limitations

AMI Limits, some versions can't be used - RHEL, SUSE Linux & Windows AMIs aren't supported

Amazon RDS instances are not supported

Placement groups are not supported for dedicated hosts.

Hosts can be shared with other organization accounts using **RAM** which is the Resource Access Manager.

The owner of the dedicated host can see the instances other organization create in the dedicate host but 
the other guest organization cannot see other intances. But the owner cannot control any of the instances
created by other organizations though

This is mostly used for licensing problems related to ports.

### Enchanced Networking

Enchanced networking isued to improve the overall performance of EC2 networking (Required for Cluster placement Groups)
Enhanced networking uses SR-IOV - The physical network interface is aware
of the virtualization. Instead of presenting itself as a single NIC which the host needs to manage it offers what can be
called Logical Cards per physical card. Each instance is given exclusive access to one logical of a physical network interface card.

There is no charge for this and is available on most EC2 types.

It allows for higher IO and lower host CPU usage

This provides more bandwidth and higher packet per seconds
Networking' in picture below

In general this provides lower latency.
![image](https://user-images.githubusercontent.com/33827177/145567386-cd4cd4dc-d034-4f44-9426-f8e46a280bde.png)
#### EBS Optimized

Historically network was shared, data and EBS.

EBS optimized there has been dedicated capacity for EBS. Most instances support
andh ave this enabled by default.

Some support, but enabling costs extra. This is generally enabled and comes
with standard instances.
