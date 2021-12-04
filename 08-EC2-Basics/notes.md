## ELASTIC COMPUTE CLOUD (EC2) BASICS

### Virtualization 101

EC2 provides Infrastructure as a Service (IAAS Product)

Servers are configured in three sections without virtualization

- CPU hardware
- Kernel
  - Operating system
  - Runs in Privileged mode and can interact with the hardware directly
- User Mode
  - Runs applications
  - Can make a **system call** to the Kernel to interact with the hardware
  - If an app tries to interact with the hardware without a system call, it
  will cause a system error and can crash the server or at minimum the app
![image](https://user-images.githubusercontent.com/33827177/144336308-f1399a9a-c8a9-4950-9f64-d01b84d865bf.png)

![image](https://user-images.githubusercontent.com/33827177/144336426-24efa092-e4c5-4c78-86ab-6ab8e6aaf704.png)

#### Emulated Virtualization - Software Virtualization

A host OS operated on the hardware and it included a hypervisor
The software ran in priviledged mode and had full access to the hardware on the
server.

Guest operating systems were wrapped in a virtual machine and had devices
mapped into their OS to emulate real hardware. Drivers such as graphics cards
were all software emulated to allow the process to run properly.

The guest OS still believed they were running on real hardware and tried
to take control of the hardware. The areas were not real and only allocated
space to them for the moment.
![image](https://user-images.githubusercontent.com/33827177/144336607-1a46160f-c2a8-4b6b-a85f-07a9bbe27ec5.png)
The hypervisor performs **binary translation**. Any calls attempted to make
are intercepted and translated in software on the way. The guest OS needs no
modification, but it slows the guest OS down a lot.

#### Para-Virtualization

Guest OS are still running in containers, except they do not use slow
binary translation. This only works on a small subset of OS that can be
modified. The OS is modified to change the **system calls** to **user calls**.
Instead of calling on the hardware, they call on the hypervisor called
hypercalls. Areas of the OS call the HV instead of the hardware. They need
to be modified for the particular vendor peforming the para-virtualization.

The OS becomes virtualization aware and got faster, but this was still a
software trick.
![image](https://user-images.githubusercontent.com/33827177/144336819-d12cb1b2-a819-4f58-9589-4110d6d6dce0.png)

#### Hardware Assisted Virtualization

The physical hardware itself is virtualization aware. The CPU has specific
functions so the hypervisor can come in and support. When guest OS try to run
priviledged instructions, they are trapped by the CPU and do not halt
the process. They are redirected to the hypervisor from the hardware.

What matters for a virtual machine is the input and output operations such
as network transfer and disk IO. The problem is multiple OS try to access
the same piece of hardware but they get caught up on sharing.
![image](https://user-images.githubusercontent.com/33827177/144337006-922e5fd9-f6a0-4240-b93f-8e363850ef4b.png)
#### SR-IOV (Singe Route IO virtualization)
In this case the Network cards are aware of the virtualizations...
Allows a network or any addon card to present itself as many mini cards.
As far as the hardware is concerned, they are real dedicated cards for their
use. No translation needs to be done by the hypervisor. The physical card
handles their own process internally. In EC2 this feature is called
**enchanced networking**. This means less CPU usage for the guest.
![image](https://user-images.githubusercontent.com/33827177/144337253-c8ec071d-3935-44b5-9e36-9e85150dffbe.png)

### EC2 Architecture and Resilience

- EC2 instances are virtual machines (OS+Resources)
- Run on EC2 hosts
- Shared hosts 
  - Every customer is isolated even on the same shared hardware
-  Dedicated hosts
  - Dedicated hosts pay for entire host, don't pay for instances
- AZ resilient service. They run within only one AZ system. If AZ fails, then Host Fails and the EC2 will fail

Example AZ with an EC2 host

- Run within single AZ
- Local hardware such as CPU and memory
- Also have instance store
  - Temporary
  - If instance moves hosts, the storage is lost
- Networking
  - Storage networking
  - Data networking

When instances are provisioned within a specific subnet within a VPC
A primary elastic network interface is provisioned in a subnet which
maps to the physical hardware on the EC2 host. Subnets are also within
one specific AZ. Instances can have multiple network interfaces, even within
different subnets so long as they're within the same AZ.

EC2 can make use of remote storage, Elastic Block Store (EBS).

EBS also runs within a specific AZ. You can't access them cross zone.

EBS allows you to allocate volumes of persistent storage to instances
within the same AZ.

An instance runs on a specific host. If you restart the instance
it will stay on that host.

Instances stay on the host until either

- The host fails or is taken down by AWS
- The instance is stopped and then started, different than restarted.

The instance will be relocated to another host in the same AZ. Instances
cannot move to different AZs. Everything about their hardware is locked within
one specific AZ.
![image](https://user-images.githubusercontent.com/33827177/144338559-c51d025f-44df-4d07-abae-3f45b78e4753.png)
A migration is taking a copy of an instance and moving it to a different AZ.
![image](https://user-images.githubusercontent.com/33827177/144338718-98a00c1c-248b-4f7d-8678-448cfe05bf51.png)

Instances of different size can share a host in general, instances of the same type and generation (version, year, etc.) will occupy the same host.
The only difference will generally be their size. So in essence a host can have instances of different customers which are different in sizes
So if we provision two different type of instances they may end up on different zones

#### EC2 Strengths

Traditional OS+Application compute need. A specific OS with a specific hardware
setup.

Long running compute needs. Many other AWS services have run time limits.

Server style applications

- things waiting for network response
- burst or stead-load
- monolithic application stack
  - middle ware or specific run time components running on an operating system
- migrating application workloads or disaster recovery
  - existing applications running on a server and a backup system to intervene

### EC2 Instance Types

Raw CPU, memory, local storage capacity, and type
Resource ratios: Some might have more CPU while others have more storage.
Instance type will also influence Storage and Data network Bandwidth we get

Instance type can influence the architecture and vendor.
AMD CPU vs Intel CPU

Influcences features and capabilities with that instance e.g. GPU or FPGAs

#### EC2 Categories

General Purpose - default steady state workloads with even resources
Compute Optimized - Media processing, scientific modeling and gaming
Memory Optimized - Processing large in-memory datasets
Accelerated Computing - Hardware GPU, FPGAs
Storage Optimized - Large amounts of super fast local storage. Massive amounts
of IO per second. Elastic search and analytic workloads.

#### Naming Scheme

R5dn.8xlarge - whole thing is the instance type. When in doubt give the
full instance type

- 1st char: Instance family. No expectation to remember all the details
- 2nd char: Instance generation. Generally always select the newest generation.
- char after period: Instance size. Memory and CPU considerations.
  - often easier to scale system with a larger number of smaller instance sizes
- 3rd char - before period: additional capabilities
  - a: amd cpu
  - d: nvme storage
  - n: network optimized
  - e: extra capacity for ram or storage
![image](https://user-images.githubusercontent.com/33827177/144341994-42b7c515-95df-48ed-bc5a-5180fa16ad2f.png)

https://ec2instances.info/

NOte: EC2 instant connect using GUI can only connect to instances which have public addressing. You can download EC2 instance connect CLI tool which has the capability to connect to private EC2 instances as long you have private connectivity using a VPN or directconnect. There is another connection method called "Session Manager" which can connect to both public and private EC2 instances

### Storage Refresher

**Direct** local attached storage - these are physical disks on EC2

**Network** attached storage - volumes delivered over the network (EBS)

**Ephemeral** storage - temporary storage. Instant store attached to EC2 host

**Persistent** storage - permanent storage that lives on past the lifetime
of the instance. EBS is persistent storage.

#### Three types of storage

Block Storage - volume presented to the OS as a collection of blocks. No
structure beyond that. These are mountable and bootable. The OS will
create a file system on top of this, NTFS or EXT3 and then it mounts
it as a drive or a root volume on Linux. Spinning hard disks or SSD. This
could also be delivered by a physical volume. Has no built in structure.
You can mount an EBS volume or boot off an EBS volume.

File Storage - Presented as a file share with a structure. You access the
files by traversing the storage. You cannot boot from storage, but you
can mount it.

Object Storage - It is a flat collection of objects. An object can be anything
with or without attached metadata. To retrieve the object, you need to provide
the key and then the value will be returned. This is not mountable or
bootable. It scales very well and can have simultanious access.
In general, if you want a storage to boot from or a high performance storage inside an OS it would be Block Storage. If you want to share a FIle system shared by multiple different servers or that can be access by different services then File Storage should be used. Lastly, if you want largescale read and writes or building a collection of 
media then Object Storage is recommended.
#### Storage Performance

IO Block Size - size of the wheels. This determines how to split up the data.
IOPS - speed of an engine rev (RPM). How many reads or writes a storage
system can accomidate in a second.
Throughput - end speed of the race car. This can be influenced by the network
speed for network storage. Expressed in MB/s (megabyte per second).

Block Size * IOPS = Throughput

This isn't the only part of the chain, but it is a simplification.

A system might have a throughput cap. The IOPS might decrease as the block
size increases.

### Elastic Block Store (EBS)

Allocate block storage **volumes** to instances.
EBS Volumes are isolated to one AZ. EBS in one AZ is different then another AZ

And EBS volume can be detached from one instance and then reattached to another. If an EC2 instance moves between different EC2 hosts then EBS volumes follows it. If an instances stops and restarts the volume is maintained.
![image](https://user-images.githubusercontent.com/33827177/144427479-beaff6fe-b971-46f4-951b-e1dae26cff40.png)
- The data is highly available and resilient for that AZ.
- All of the data is replicated within that AZ. The entire AZ must have
a major fault to go down.
- Different physical storage types available (SSD/HDD)
- Varying level of performance (IOPS, T-put)
- Billed as GB/month.
  - If you provision a 1TB for an entire month, you're billed as such.
  - If you have half of the data, you are billed for half of the month.
![image](https://user-images.githubusercontent.com/33827177/144427927-9e3f2164-33ce-4153-982b-14e1df12c0a7.png)

Four types of Volumes:

- General purpose SSD (gp2)
- Provisioned IOPS SSD (io1)
- T-put optimized HDD (st1)
- Cold HDD (sc1)

Each volume has a dominant performance attribute

SSD based focus on maximum IOPS such as a database
HDD based focus on throughput for logs or media storage.

#### General Purpose SSD (gp2)

Uses a performance bucket architecture based on the iops it can deliever.

The GP2 starts with 5,400,000 IOPS allocated. It is available instantly.

You can consume the capacity quickly or slowly over the life of the volume.

The capacity is filled back based upon the volume size.

Min of 100 iops added back to the bucket per second.
Above that, there are 3 IOPS/GiB of volume size. The max is 16,000 IOPS.
![image](https://user-images.githubusercontent.com/33827177/144429766-a325db87-4625-4cbf-aedd-7d479cc878df.png)
This is the **baseline performance**

This should be the default for boot volumes and some data volumes.

Can only be attached to one volume at a time.
#### General Purpose SSD (gp3)
Removes the credit bucket architecture of gp2 for sth simpler. Every gp3 volume regardless of size starts with 3000 IOPs and can transfer 125 MiB/s. Volume sizes can be 1 GB to 16 TB.
![image](https://user-images.githubusercontent.com/33827177/144430447-c6f97513-7d2c-40c9-8172-73e53209443e.png)

#### Provisioned IOPS SSD (io1)

50:1 IOPS to GiB Ratio

You pay for capacity and the IOPs set on the volume. This is good if your
volume size is small but need a lot of IOPS.

64,000 is the max IOPS per volume assuming 16 KiB I/O.

Good for latency sensitive workloads such as mongoDB.

Multi-attach allows them to attach to multiple EC2 instances at once.

If it mentions high IOPS, or mentioning latency. Small volume sizes.

#### HDD Volume Types

st1 - throughput
sc1 - cold

Both have 4 shared characteristics

- great value
- great throughput
- 500 GiB - 16 TiB
- Neither can be used for instance boot volumes. Cannot boot from them.

Good for price conscious.
Great for high throughput vs IOPs.

Good for streaming data on a hard disk. Media conversion with large amounts of
storage.

Frequently accessed high throughput intensive workload

- log processing
- data warehouses

The access patterns should be sequential

##### st1

Starts at 1 TiB of credit per TiB of volume size.

40 MB/s baseline per TiB
Burst of 250 MB/s per TiB
Max t-put of 500 MB/s

##### sc1

Designed for less frequently accessed data

Fills slower:

12 MB/s baseline per TiB
Burst of 80 MB/s per TiB
Max t-put of 250 MB/s

There is a massive inefficency for small reads and writes. These are based on
large block sizes of data in a sequential way

#### Exam Power Up

Volumes are created in an AZ, isolated in that AZ
If an AZ fails, the volume is impacted.
Highly available and resilient in that AZ. The only reason for failure is
if the whole AZ fails.
Generally one volume to one instance, but multi-attach occurs.
Has a GB/m fee regardless of instance state.
EBS maxes at 80k IOPS per instance and 64k vol (io1)
Max 2375 MB/s per instance, 1000 MiB/s (vol) (io1)

### EC2 Instance Store

Local physical storage that instances can utilize attached to an instance.

**block storage** devices. They're just like EBS but they're local.

The volumes are physically connected to one EC2 host. They are isolated to
that one specific host.

Instances on that host can access them.

Highest storage performance in AWS.

They are included in instance price, use it or lose it.

They can be attached ONLY at launch. Cannot be attached later.

Architecturally, each instance has a collection of ephemeral volumes that are
locked to that specific host. Even though an instance is been alocated a
specific number of volumes, the data is locked to the host.

Instances can move between hosts for many reasons:

- If an instance is stopped and started, that migrates hosts.
- If a host undergoes AWS maintenance, it will be wiped.
- If you change the type of an instance, these will be lost.
- If a physical hardware fails, then the data is gone.

The number, size, and performance of instance store volumes vary based on the
type of instance used. Some instances do not have any instance store volumes
at all.

#### Instance Store Performance

This is much higher than EBS can provide. These volumes
perform at much higher volumes than EBS.

#### Exam Powerup

- Instance store volumes are local to EC2 host.
- Can only be added at launch time. Cannot be added later.
- Any data on instance store data is lost if it gets moved, or resized.
- Highest data performance in all of AWS.
- You pay for it anyway, it's included in the price.
- TEMPORARY

### EBS vs Instance Store

If the read/write can be handled by EBS, that should be default.

When to use EBS

- Highly available and reliable in an AZ. Can self correct against HW issues.
- Persist independently from EC2 instances.
  - Can be removed or reattached.
  - You can terminated instance and keep the data.
- Multi-attach feature of **io1**
  - Can create a multi shared volume.
- Region resilient backups.
- Require up to 64,000 IOPS and 1,000 MiB/s per volume
- Require up to 80,000 IOPS and 2,375 MB/s per instance

When to use Instance Store

- Great value, they're included in the cost of an instance.
- More than 80,000 IOPS and 2,375 MB/s
- If you need temporary storage, or can handle volativity.
- Stateless services, where the server holds nothing of value.
- Rigid lifecycle link between storage and the instance.
  - This ensures the data is erased when the instance goes down.

### Snapshots, restore, and fast snapshot restore

EBS Snapshots

Efficent way to backup EBS volumes to S3. Protect data against AZ issues.
Can be used to migrate data between hosts.

The data becomes region resilient.

Snapshots are incremental volume copies to S3.

The first is a **full copy** of `data` on the volume. This can take some time.

EBS won't be impacted, but will take time in the background.

Future snaps are incremental, consume less space and are quicker to perform.

If you delete an incremental snapshot, it moves data to ensure subsequent
snapshots will work properly.

Volumes can be created (restored) from snapshots. Snapshots can be used to
move EBS volumes between AZs. Snapshots can be used to migrate data between
volumes.

#### Snapshot and volume performance

When creating a new EBS volume without a snapshot, the performance is
available immediately.

If you restore a snapshot, it does it lazily.

If you restore a volume, it will transfer it slowly in the background.

If you attempt to read data that hasn't been restored yet, it will
pull it from S3, but this will achieve lower levels of performance than reading
from EBS directly.

You can force a read of all data immediately. This is something you would do
right when using a volume.

You could also use Fast Snapshot Restore (FSR) - Immediate restore

You can have up to 50 snaps per region. This is set on the snap and AZ.

FSR is not free and can get expensive with lost of different snapshots.

You can force the same response by reading every block manually using DD or
another tool in the OS.

#### Snapshot Consumption and Billing

They are billed using a GB/month metric.

20 GB stored for half a month, represents 10 GB-month.

This is used data, not allocated data. If you have a 40 GB volume but only
use 10 GB, you will only be charged for the allocated data. This is not true
for EBS itself.

The data is incrementally stored which means doing a snapshot every 5 minutes
will not necessarily increase the charge as opposed to doing one every hour.

#### EBS Encryption

Provides at rest encryption for block volumes and snapshopts.

When you don't have EBS encryption, the volume is not encrypted.
The physical hardware itself may be performing at rest encryption, but
that is a seperate thing.

When you set up an EBS volume initially, EBS uses KMS and a customer master key.
This can be the ebs default (CMK) which is refered to as `aws/ebs` or it
could be a customer managed CMK which you manage yourself.

That key is used by EBS when an encrypted volume is created. The CMK
generates an encrypted data encryption key which is stored with the volume with
on the physical disk. This key can only be encrypted by KMS when a role with
the proper permissions makes the request.

When the volume is first used, EBS asks CMS to decrypt the key and stores
the decrypted key in memory on the EC2 host while it's being used. At all
other times it's stored on the volume in encrypted form.

When the EC2 instance is using the encrypted volume, it can use the
decrypted data encryption key to move data on and off the volume. It is used
for all cryptographic operations when data is being used to and from the
volume.

When data is stored at rest, it is stored as **Ciphertext**.

If the EBS volume is ever moved, the key is discarded.
![image](https://user-images.githubusercontent.com/33827177/144686391-5dd8b130-490d-4316-a52d-a9a41c88c2a6.png)
If a snapshot is made of an encrypted EBS volume, the same data encryption
key is used for that snapshot. Anything made from this snapshot is also
encrypted in the same way.

Everytime you create a new EBS volume from scratch, it creates a new
data encryption key.

##### EBS Exam Power Up

AWS accounts can be set to encrypt EBS volumes by default.
It will use the default CMK unless a different one is chosen.
Each volume uses 1 unique DEK (data encryption key)
Snapshots and future volume use the same DEK
Can't change a volume to NOT be encrypted. You could mount an unencrypted
volume and copy things over but you can't change the original volume.
###The OS isn't aware of the encryption, there is no performance loss. The data
uses AES256
If an exam question does not use AES256, or it suggests you need an OS to
encrypt or hold the keys, then you need to perform full disk encryption
at the operating system level.
You can perform full disk encryption on an unencrypted or encrypted EBS
volume. For Full Disk encryption which is done at the OS level there is 
going to be a performance penalty for the encryption/decryption process.

### EC2 Network Interfaces, Instance IPs and DNS

An EC2 instance starts with at least one ENI - elastic network interface.

An instance may have ENIs in seperate subnets, but everything must be
within one AZ.

When you launch an instance with Security Groups, they are on the
network interface and not the instance.

#### Elastic Network Interface

Has these properties

- MAC address
- Primary IPv4 private address
  - From the range of the subnet the ENI is within.
  - 10.16.0.10 will be static and not change for the lifetime of the instance
  - Given a DNS name that is associated with the address
    - ip-10-16-0-10.ec2.internal
    - only resolvable inside the VPC and always points to private IP address
- 0 or more secondary private IP addresses
- 0 or 1 public IPv4 address given two ways
- 1 elastic IP address per IPv4 address. Elastic IP address are public IPv4 address and these are different then normal IPv4 address
- 0 or more IPv6 addresses which are all public. With IPv6 there is no definition of public or private addresses. All are public addresses

  - Instance must manually be set to recieve an IPv4 addr
  - Default settings into a subnet which automatically allocates an IPv4
  - Dynamic IP that is not fixed
  - If you stop an instance the address is deallocated.
  - When you start up again, it is given a brand new IPv4 address
  - Restarting the instance will not change the IP address
  - Changing between EC2 hosts will change the address
  - They are allocated a public DNS name.
  - Public DNS name will resolve to the primary
  private IPv4 address of the instance
  - Outside of the VPC, the DNS will resolve to the public IP address.
  - Allows one single DNS name for an instance, and allows traffic to resolve
  to an internal address inside the VPC and the public will resolve to a public
  IP address.

- 1 elastic IP per private IPv4 address
  - Can have 1 public elastic interface per private IP address on this interface
  - Allocated to your AWS account
  - Can associate with a private IP on the primary interface or
  on the secondary interface.
  - If you are using a public IPv4 and assign an elastic IP, the original IPv4
  address will be lost. There is no way to recover the original address.

- 0 or more IPv6 address on the interface
  - These are by default public addresses
- Security groups
  - applied to network interfaces
  - will impact all IP addresses on that interface
  - if you need different IP addresses impacted by different security
  groups, then you need to make multiple interfaces and apply different
  security groups to those interfaces
- Source / destination checks
  - if traffic is on the interface, it will be discarded if it is not
  from going to or coming from one of the IP addresses

Secondary interfaces function in all the same ways as primary interfaces except
you can detach interfaces and move them to other EC2 instances.

#### Exam Power Ups

Legacy software is licensed using a mac address.
If you provision a secondary ENI to a specific license, you can move
around the license to different EC2 instances.

Multi homed (subnets) management and data.

Different security groups are attached to different interfaces.

The OS doesn't see the IPv4 public address.

You always configure the private IPv4 private address on the interface.

Never configure an OS with a public IPv4 address.

IPv4 Public IPs are Dynamic, starting and stopping will kill it

Public DNS for a given instance will resolve to the primary private IP
address in a VPC. If you have instance to instance communication within
the VPC, it will never leave the VPC. It does not need to touch the internet
gateway.

#### Launching Manual Install of Wordpress on single EC2 instance
This is going to be a working Wordpress website without High Availibility features because it will be running on a single EC2 instance and it is going to be architecturally
monolithic i.e. both the application and the database
We will have the A4L VPC and we will deploy the Wordpress in a single subnet WEBA public subnet.
![image](https://user-images.githubusercontent.com/33827177/144689375-532f5e2f-5484-4487-b57a-587fd97c7477.png)

### Amazon Machine Image (AMI)

Images of EC2.

AMI's can be used to launch EC2 instance.

- When you launch an EC2 instance, you are using an Amazon provided AMI.
- Can be Amazon or community provided
- Marketplace (can include commercial software)
  - Will charge you for the instance cost and an extra cost for the AMI
- Regional - Each individual region has it own set of AMIs and each AMI has a unique ID ami-0a3464564535nb6h4545bh56
  - ami-`random set of chars` for the same distribution of AMI in different regions
- An AMI can only be used in the region it is in
- Controls permissions
  - Default only your account can use it
  - Can be set to be public
  - Can have specific AWS accounts on the AMI
- Can create an AMI from an existing EC2 instance to capture the current config

#### AMI Lifecycle

Launch

EBS volumes are attached to EC2 devices using block IDs

- BOOT /dev/xvda
- DATA /dev/xvdf

Configure

- Can customize the instance from applications or volume sizes

Create Image

- Once it has been customized, an AMI can be created from that
- AMI contains:
  - Permissions: who can use it
  - EBS snapshots are created from attached volumes
    - Block device mapping links the snapshot IDs and a device ID for
    each snapshot.

Launch

When launching an instance, the snapshots are used to create new EBS
volumes in the same AZ as that of the EC2 instance's image (or the instances within other AZs of that region) and contain the same block device mapping.

An AMI itself doesn't contain any Data Volume. An AMI is a container, it references snapshot that are created from the original EBS volumes of an EC2 instance and so you can take them to provision new instances with the exactly same Data and configuration of volumes
![image](https://user-images.githubusercontent.com/33827177/144692701-bf8c530f-ede1-46cf-b40d-bf1a576ba916.png)

#### Exam Powerups

AMI can only be used in one region
AMI Baking: creating an AMI from a configuration instance.
An AMI cannot be edited. If you need to update an AMI, launch an instance,
make changes, then make new AMI
Can be copied between regions (includes its snapshots)
Remember permissions by default are your account only

Billing: AMI contains snapshots so the Billing is for the storage capacity for the EBS snapshots that the AMI references

### EC2 Pricing Models

#### On-Demand Instances

- Hourly rate based on OS, size, options, etc
- Billed in seconds (60s min) or hourly
  - Depends on the OS
- Default pricing model
- No long-term commitments or upfront payments
- New or uncertain application requirements
- Short-term, spiky, or unpredictable workloads which can't tolerate
any disruption.
![image](https://user-images.githubusercontent.com/33827177/144696299-1a67e4bb-c281-4420-a36c-a355e5e4b095.png)

#### Spot Instances

Up to 90% off on-demand
Depends on the spare capacity
You can set a maximum hourly rate in a certain AZ in a certain region.
If the max size you set is above the spot price, you pay for the instance.
As the spot price increases, you'll keep paying until the price increases.
![image](https://user-images.githubusercontent.com/33827177/144696412-92509c07-aa30-43f7-8dfc-d872c4d44714.png)

Once this price increases too much, it will terminate the instance. Therefore, Spot instances shouldn't be viewed as reliable

![image](https://user-images.githubusercontent.com/33827177/144696432-c9a9bcf7-702f-4fdb-b361-a14fd95f5697.png)
Never use SPOT for workload which can't tolerate interruptions. No matter how high you set your Spot price there is always a chance for your instance to be terminated. So domain controllers, mail servers, company websites, etc. shouldn't use Spot

![image](https://user-images.githubusercontent.com/33827177/144696581-65549b40-ad7e-4a55-ac3c-c2c6b5c1e8eb.png)

#### Reserved Instance (Now on as Standard Reserved)
If you need access to the cheapest EC2 running 24/7 365 days an year then use this.
Up to 75% off on-demand
The trade off is commitment.
You're buying capacity in advance for 1 or 3 years.
Flexibility on how to pay

- All up front
- Partial upfront
- No upfront
![image](https://user-images.githubusercontent.com/33827177/144696904-1ba1cfa2-cbdc-49c6-9ce2-d604d706a2c1.png)
Best discounts are for 3 years all up front
Reserved in region, or AZ with capacity reservation
If you lock a reservation in a specific AZ then you will reap benefits only if you when launching instances in that AZ but it also reserves capacity
If you purchase reservation for a region it doesn't reserve capacity
Reserved takes priority for AZ capacity but it can benefit any instances which are launch in that region???
Can perform scheduled reservation when you can commit to specific time windows.

If you have a known steady state usage, email usage, domain server.
Cheapest option with no tolerance for distruption.

### Dedicated Hosts
- These hosts come with all the resources that you would expect from a physical host like number of CPU/Cores, Meomry, Local Storage, etc.
- You pay for the host. Any instance that run on the host has no per second charge. You can lauch any number of instances up to the complete resource capacity of the host
Exmaple:
You are using a software that is license based on teh Sockets/Cores of a machine
![image](https://user-images.githubusercontent.com/33827177/144697100-898251e1-75a2-4c24-a19a-78d243e4aa74.png)
Host Affinity - Starting and Stoping an instance means it will remain on the same host
### Dedicated Instances
![image](https://user-images.githubusercontent.com/33827177/144697179-7307500e-fec2-4ffa-b69c-4c5827d6bdf0.png)

#### Scheduled Reserved Instance
![image](https://user-images.githubusercontent.com/33827177/144697342-f9b80b04-1d99-4678-a78a-58ca3aac5bde.png)

### Capacity Reservations
![image](https://user-images.githubusercontent.com/33827177/144697494-caeb072a-dd6d-41b7-89f3-db0c5942a160.png)
Capacity reservations don't have the 1 or 3 year commitment requirements tht you need for reserved instances

### EC2 Savings Plan
Its like Reserved instances but instead of focusing on a particular type of instance in an AZ or region you are making a commitment of an hourly spend for a 1 or 3 year term
![image](https://user-images.githubusercontent.com/33827177/144697700-78aa0f7f-ece6-4c53-80c3-5b6484e0b374.png)

### Instance Status Checks and Autorecovery

Every instance has two high level status checks

#### System Status Checks

Failure of this check could indicate software or hardware problems of the EC2
service or the host.

#### Instance Status Checks

This is specific to the file system or has a corrupted Kernel

Assuming you haven't launched an instance, this is a problem and needs to be
fixed.

#### Create Status Check Alarm

This feature has four options

- Recover this instance: can be a number of steps depending on the failure
- Stop this instance
- Terminate this instance: useful in a cluster
- Reboot this instance:

### Horizontal and Vertical Scaling

#### Vertical Scaling

As customer load increases, the server may need to grow to handle more data.
The server can increase in capacity, but this will require a reboot.
Often times vertical scaling can only occur during planned outages.
Larger instances also carry a $ premium compared to smaller instances.
There is an upper cap on performance - instance size.
No application modification is needed.
Works for all applications, even monoliths (all code in one app)

#### Horizontal Scaling

As the customer load increases, this adds additional capacity.
Instead of one running copy of an application, you can have multiple versions
running on each server.
This requires a load balancer.
When customers try to access an application, the load balancer ensures the
servers get equal parts of the load.

Sessions are everything.
With horizontal scaling you can shift between instances equally.
This requires either application support or off-host sessions.
This means the servers are **stateless**, the app stores session data elsewhere.

No distruption while scaling up or down.

No real limits to scaling.

Uses smaller instances so you pay less, allows for better granulairity.

### Instance Metadata

EC2 service provides data to instances
Accessible inside all instances

Memorize **<http://169.254.169.254/latest/meta-data/>

Meta-data contains:

- enviroment the instance is in
- networking is the big reason why
- authentication information
  - instances can be used to gain access on other services
- user-data
- NO AUTHENTICATION or ENCRYPTED
  - Anyone who can gain access and it can and will get exposed
  - Can be restricted by local firewall
