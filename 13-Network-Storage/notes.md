## Network Storage

### EFS Architecture
EFS provides network based file systems that can be mounted within linux EC2 instances 

EFS file system can used  i.e. mounted on many EC2 instances at once (so the data can be shared)

Allows to store media for post outside the EC2 instances and so the media isn't lost when EC2 instances are added and removed

EFS moves the instances closer to a stateless.

EFS like EBS exists separately then the EC2. EBS is block storage while EFS is file storage

EFS is an implementation of NFSv4

EFS Filesystems can be mounted witin EC2 Linux instances. Linux uses tree structures, so devices can be mounted into folder in the hierarchy 
For example, EFS filesystem can be mounted into /NFS/Media

Using Amazon EFS with Microsoft Windows–based Amazon EC2 instances is not supported.

Media can be shared between many EC2 instances.

This is a private service, access is via mount targets inside a VPC.

You can access from on-premises (i.e. outside the VPC) with VPN or DX so long as access is configured.

#### Elastic File System Explained

EFS runs inside a VPC.
EFS includes POSIX permissions. These are made available via mount targets.
![image](https://user-images.githubusercontent.com/33827177/146006794-3d366382-d1bb-4b5d-840b-f2a64dbfae67.png)
For a fully highly available system you need a mount target for each AZ the
system runs in.

You can use hybrid networking to connect to the same mount targets.
![image](https://user-images.githubusercontent.com/33827177/146007916-bfe0729d-d2e4-49d1-9165-ba35c99901d7.png)

#### EFS Exam Powerup

EFS is Linux Only

Two performance modes, general purpose and max I/O performance mode.

General purpose = default for 99.9% of uses

Bursting and provisioned throughput modes.

Two storage classes available, standard and infrequent access.

Lifecycle policies can move data between EFS
