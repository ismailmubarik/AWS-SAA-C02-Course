## DynamoDB

### DynamoDB Architecture

NoSQL Public Database as a Service (DBaaS)

Wide column Key/Value database.

Not like RDS which is a Database Server as a Product. This is only the
database.

You can take full control and provisioned capacity or use on-demand
mode and set it and forget it.

This is highly resilient, across AZs and optionally globally resilient.

Data is replicated across multiple storage nodes by default so doesn't need
to be set up or managed.

Really fast, single digit millisecond access to data.

Supports backups and encryption at rest.

It allows event-driven integration. Do things when data changes.

#### Dynamo DB Tables

A table is a grouping of items which share the same primary key.
There is no limit to the number of items in a table.

When configuring Dynamo, you need to pick a primary key from two types

- Simple (Partition)
- Composite (Partition and Sort)

Every item in the table needs a unique primary key.

The attributes may or may not be there. This is not necessary.

Each item can be at most 400KB in size. This includes the primary key and
attributes.

In DynamoDB, capacity means speed. If you choose on-demand capacity model
you don't have to worry about capacity. You only pay for the operations
for the table.

If you choose provisioned capacity, you must set this on a per table basis.

Capacity is set per WCU or RCU

1 WCU means you can write 1KB per second to that table
1 RCU means you can read 4KB per second for that table

#### Backups

On-demand Backups : Similar to manual RDS snapshots. Full backup of the table
that is retained until you manually remove that backup. This can be used to
restore data in the same region or cross-region. You can adjust indexes, or
adjust encryption settings.

Point-in-time Recovery : Must be enabled on each table and is off by
default. This allows continous record of changes for 35 days to allow you to
replay any point in that window to a 1 second granularity.

#### Considerations

If you see NoSQL, you should jump towards DynamoDB.

If you see relational data, this is not DynamoDB.

If you see key value and DynamoDB is an answer, this is likely the proper
choice.

Access to Dynamo is from the console, CLI, or API. There is NoSQL
available.

Billing based on:

- RCU and WCU
- Storage on that table
- Additional features on that table

Can purchase reserved capacity with a cheaper rate for a longer term commit.

### DynamoDB Operations, Consistency, and Performance

Reading and Writing

On-Demand - unknown or unpredictable load on a table. This is also good
for as little admin overhead as possible. Pay a price per million
Read or Write units. This is as much as 5 times the price as provisioned.

Provisioned - RCU and WCU set on a per table basis.

Every operation consumes at least 1 RCU/WCU

1 RCU = 1 x 4KB read operation per second. This rounds up.
1 WCU = 1 x 1KB write operation per second.

Every single table has a WCU and RCU burst pool. This is 500 seconds
of RCU or WCU as set by the table.

#### Query

You have to pick one partition key value to start.

The partition key can be the sensor unit, the sort key can be the day of the
week you want to look at.

Query accepts a single PK value and **optionally** a SK or range.
Capacity consumed is the size of all returned items. Further filtering
discards data, capacity is still consumed.

In this example you can only query for one weather station.

If you Query a PK it can return all fields items that match. It is always
more efficent to pull as much data as needed per query to save RCU.

You have to query for at least one item of PK and are charged for the
response of that query operation.

If you filter data and only look at one attribute, you will still be
charged for pulling all the attributes against that query.

#### Scan

Least efficent when pulling data from Dynamo, but the most flexible.

Scan moves through the table item by item consuming the capacity
of every item. Even if you consume less than the whole table, it will
charge based on that. It adds up all the values scanned and will charge
rounding up.

#### DynamoDB Consistency Model

Eventually Consistent : easier to impliment and scales better

Strongly (Immediatly) Consistent : more costly to achieve

Every piece of data is replicated between storage node. There is one
Leader storage node and every other node follows.

Writes are always directed to the **leader node**. Once the leader
is complete, it is **consistent**. Once the leader node has the new
data it immediatly starts the process of replication. This typically
takes miliseconds and assumes the lack of any faults on the storage nodes.

Eventual consistent reads check 1/3 nodes. Could be unlucky with stale data
if a node is checked before replication completes. You get a discount
for this risk.

A strongly consistent read always uses the leader node and is less
scalable.

Not every application can tolerate eventual consistency. If you have a stock
database or medical information, you must use strongly consistent reads.

If you can tolerate the cost savings you can scale better.

#### WCU Calculation

You need to store 10 items per second with 2.5K average size per item.

Calculate WCU per item, round up, then multiply by average per second.

(2.5 KB / 1 KB) = 3 * 10 p/s = 30 WCU

#### RCU Calculation

Need to retrieve 10 items per second with 2.5K average size per item

Calculate RCU per item, round up, then mu

(2.5 KB / 4 KB) = 1 * 10 p/s = 10 RCU for strongly consistent
5 RCU for eventually consistent.

### DynamoDB Streams and Triggers

DymanoDB stream is a time ordered list of changes to items in a DynamoDB
table. A stream is a 24 hour rolling window of the changes.

It uses Kinesis streams on the backend.

This is enabled on a per table basis. This records

- Inserts
- Updates
- Deletes

Different view types influence what is in the stream.

There are four view types that it can be configured with:

- KEYS_ONLY : only shows the item that was modified
- NEW_IMAGE : shows the final state for that item
- OLD_IMAGE : shows the initial state before the change
- NEW_AND_OLD_IMAGES : shows both before and after the change

Pre or post change state might be empty if you use
**insert** or **delete**

### DynamoDB Local (LSI) and Global (GSI) Secondary Indexes

Great for improving data retrival in DynamoDB.

Query can only work on 1 partition key value at a time.

Indexes are a way to provide an alternative view on table data.

You have the ability to choose which attributes are projected
to the table.

#### Local Secondary Indexes (LSI)

Alternative view on base table data. These must be created with a
table in the beginning. This cannot be added later.

Maximum of 5 LSI's per base table.

Uses the same partition key, but different sort key.

Shares the RCU and WCU with the table.

Can use:

- ALL
- KEYS_ONLY
- INCLUDE

Only items from the base table that have an attribute for the
new value of the sort key, this can be limited on the attribute.

This ensures the LSI will only include data that you want to view.

It makes a smaller table and makes **scan** operates easier.

#### Global Secondary Index (GSI)

Can be created at any time and much more flexible.

There is a default limit of 20 GSIs for each table.

This allows for alternative PK and SK.

GSI will have their own RCU and WCU allocations.

You can then choose which attributes are included in this table.

GSIs are **always** eventually consistent. Replication between
base and GSI is Async

#### LSI and GSI Considerations

Must be careful which projections (KEYS_ONLY, INCLUDE, ALL). This can
eat up capacity.

If you don't project a specific attribute, but you require the attribute
later, it will then fetch the data later.

This means you should try to plan what will be used on the front.

**GSI as default** and only use LSI when **strong consistency** is required

Indexes are designed when data is in a base table needs an alternative
access pattern. This is great for a security team or data science team
to look at other attributes from the original purpose.

### DynamoDB Streams and Lambda Triggers

![image](https://user-images.githubusercontent.com/33827177/147798240-75e4b433-46e9-4669-9100-b6263e1be448.png)

![image](https://user-images.githubusercontent.com/33827177/147798433-43ec71eb-a3e5-4df1-ab44-79c35d46aef6.png)

#### Trigger Concepts

Allow for actions to take place in the event of a change in data

Item change generates an event which contains the data which
was changed. The specifics depend on the view type.

The action is taken using that data. This will combine the
capabilities of stream and lambda.

This is great for reporting and analytics in the event of changes.

Good for data aggregation for stock or voting apps. This
can provide messages or notifications and eliminates the
need to poll databases. Similarly, can be used for Messaging or Notifications

The advantage of Streams & Triggers is that rather than having to poll data when nothing has happened and 
in the process consuming RCU, you can respond when the event happens and consume minimum RCU and then may be WCU

![image](https://user-images.githubusercontent.com/33827177/147798613-005c786a-7b9c-4e22-a957-212982019e6f.png)

### DynamoDB Global Tables

Global tables provide multi-master cross-region replication.

Tables are created in multiple AWS regions. In one of the tables, you
configure the links between all of the tables.

DynamoDB will enable replication between all of the tables.

Between the tables, **last writer wins** in conflict resolution.

Reads and Writes can occur to any region and are replicated
generally within a second.

Strongly Consistent reads **only** in the same region as writes. Eventually consistent in others

![image](https://user-images.githubusercontent.com/33827177/147798801-3c103d78-5ab7-433d-b3e6-92a5b3afb9c0.png)

Replication is generally sub-second (eventually-consistent) and depends on the region load.

**global eventual consistency** but can have same region strongly consistent

Provides Global HA, performance and/or disaster recovery easily.

### DynamoDB Accelerator (DAX)

This is an in memory cache for the application. That is unlike, other caches it 
is integrated with the DynamoDB product itself so susbtantially improves performance.

Traditional Cache : The application needs to access some data and checks
the cache. If there is no data, it is a cache miss and it pulls from
the database. This then updates the cache with the new data. Next times
there will be a cache hit and it will be faster

![image](https://user-images.githubusercontent.com/33827177/147799286-eb347a4b-744d-499d-84e2-668d59432765.png)

DAX : The application instance has DAX SDK added on. DAX and dynamoDB are one
in the same. If DAX has the data then the data is returned direclty. If not
it will talk to Dynamo and get the data. This is one set of API calls and
is much easier for the developers.

#### DAX Architecture

DAX runs from within a VPC and is designed to be deployed to multiple
AZs in that VPC. It must be deployed accross availablity zones to ensure
it is available.

There is a primary node which is a Read and Write node which flows
out to read replicas on other AZs. The DAX SDK communicates with the Read
Write node.

DAX maintains 2 two different caches. The first is the Item cache which holds 
invidual items which can be retrieved via **GetItem** or **Batch GetItem**. 
You must specify the items partition and sort key if present.

![image](https://user-images.githubusercontent.com/33827177/147799625-0fa7258f-c911-4c29-87cd-cec04bd61817.png)

There is also a query cache which holds the parameters used for the
original query or scan and the data returned as a result. Whole query 
or scan operations can be rerun and return the same cache data.

DAX is accesed like RDS> Every DAX cluster has an endpoint which can return 
data in microseconds in case of cache hit

Any cache misses can be returned in single digit miliseconds.

When writing data to DAX, it can use write-through. Data is written to the
database, then written to DAX.

DAX is an efficient way of interacting with DynamoDB because architecturally it
abstracts away from DynamoDB. You think you are interacting with a single product
using a single set of API but behind the scene DAX is handling the caching read and write
and as a result improving performance.

#### DAX Exam PowerUp

Primary node which writes and Replicas which read

Nodes are HA, if this fails there will be an election and secondary nodes
will be made primary.

DAX is an In-memory cache - scaling. If you are performing the same operations, you
can achieve faster reads and reduced costs.

With DAX you can scale up or scale out.

DAX supports write-through. If you write data to DynamoDB, you can
use the DAX SDK which will write the data to both DynamoDB and the cache

While DynamoDB is a public service, DAX is not as it is deployed within a VPC. So Anything
that uses that DAX will have to be deployed inside the VPC.

Any questions which talk about caching with DynamoDB, assume it is DAX.

**To Summarize**, DAX is an in-memory cache that is designed to read operations by an order of magnitude
taking you down from single digit milliseconds when using DynamoDB natively to microseconds if you use DAX.
**When to Use?** If you are reading the same data over and over again, then you should look into in-memory
cache. Now choosing between DAX and an in-memory cache comes down to how much admin overhead you want to manage
because DAX is integrated with DynamoDB because it uses the same SDK for accessing the cache and the DynamoDB 
both together as a single abstract entity DAX offers siginificant less workload. If you have a Read-heavy or bursty
workload then DAX provides increased throughputs and significant cost reductions. If your application applies large 
RCU values onto a table then these can get expensive quickly and you are better of using DAX. If you find that your 
app has read-heavy operations on the same set of data then consider DAX. If you care for supe-low response time for Reads 
consider DAX for your app. If your application is sensitive to consistency of data then DAX is not suitable. If your application
is Write heavy with very little Read operations then DAX is not advisable.

### Amazon Athena

![image](https://user-images.githubusercontent.com/33827177/147800634-335a57ea-d67b-46fe-8d04-6450e6c45dba.png)

You can take data stored in S3 and perform Ad-hoc queries on data. Pay
only for the data consumed.

This is serverless.

Start off with structured, semi structured data stored in S3.

This uses **schema-on-read**, the original data is never changed
and remains on S3 in its original form.

This modifies data in flight when its read.

Normally you need to make a table and then load the data in.

With Athena you create a schema and load data on this schema on the fly in
a relational style way without changing the data.

The output of a query can be sent to other services and can be
performed in an event driven way.

#### Athena Explained

The source data is stored on S3 and Athena can read from this data.

In Athena you are defining a way to get the original data and defining
how it should show up for what you want to see.

![image](https://user-images.githubusercontent.com/33827177/147800750-0d3c67c8-b002-4c8b-b77a-19f13a154eb9.png)

Tables are defined in advance in a data catalog and data is projected
through when read. It allows SQL-like queries on data without transforming
the data itself.

This can be saved in the console or fed to other visualation tools.

You can optimize the original data set. How??

Exam Power Up?
If you are given a scenario where structured, semi-structured or unstructured data is stored on S3 and you need to 
perform Ad-hoc queries and you are charged only for the queries then Athena might be answer...

### Demo Example
![image](https://user-images.githubusercontent.com/33827177/147801433-d4178e88-7219-4e80-bf79-8dac9d2875fc.png)

### AWS Elastic Cache
In-memory database for application with high performance requirements.

Consider RDS which is a managed product that delivers Database as a service. Databases stores data **persistenly** on a disc and hence its performance is limited compared to cache. An In-memory cache holds data in memory which orders of magnitude faster than the disc but the data is not persistent, so it can only be used for temporary data.

![image](https://user-images.githubusercontent.com/33827177/147838017-b19e068c-b4a0-4b66-8731-cc7e9ca32378.png)

Can be used to store session data for stateless application (This is generally used in highly scalable and elastic and fault tolerant environment where users can't notice if an components fail and hence stateless applications are required.)

You cannot just use ElasticCache your application code needs to change. Your application should understand or be compatible with caching architecture. For example, the application should be programmed to check the cache for data and if there is a cache miss then check the database. So in exam, if a questions says no application/code changes required then ElasticCache isn't the answer.

![image](https://user-images.githubusercontent.com/33827177/147838272-e466cce6-e541-4bd3-8102-6ff6b05f3424.png)

![image](https://user-images.githubusercontent.com/33827177/147838313-8be1f45b-aff0-48a8-aef2-2c6bc781f916.png)

![image](https://user-images.githubusercontent.com/33827177/147838400-27e3fe5a-248c-4bec-bf32-464f76794728.png)

### AWS RedShift








