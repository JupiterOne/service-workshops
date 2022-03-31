# AWS Cost Analysis

In this workshop, we will go over queries that are useful for identify resources that can result in increasing costs and attack surface within your AWS environment. Examples will include identifying DataStore usage, unused resources, as well as how to evaluates the costs and risk of these entities across your AWS organization within the JupiterOne platform. This workshop will include several customizable Alert Rules, as well as an Insight Dashboard useful for continually monitoring your AWS environment's ongoing changes.

This concept is relevant to Managers and Directors who need to understand the resources and costs across an AWS organization. 

**Prerequisite**

This workshop requires the AWS integration. This workshop is highly relevant to JupiterOne user's leveraging AWS Organization level ingestion.


## DataStore Usage

JupiterOne does not have visibility into what is stored by your data storage entities. However, there is context that can be discovered that will identify redundancies, as well as over and under utilization across your organization.

### S3 Bucket Usage

In organization leveraging the S3 Service, bucket size will benefit from monitoring. 

#### S3 Buckets Ordered By Size
```
find aws_s3_bucket with bucketSizeBytes!=undefined as b 
return b.bucketSizeBytes as Size, 
b.bucketName as "Name", 
b.createdOn as "Creation Date",  
b.accountId ORDER BY Size DESC
```

This query will show a list of S3 buckets across the entire organization, ordered by Size. This will also contain a creation date, the bucket's name, as well as accountId associated with the bucket.

>This query leverages the `ORDER BY` function. This can be used to order values within the `RETURN`. Columns can be ordered `ASC` or `DESC`.

#### S3 Average Account Bucket Size

When performing bucket analysis at organization with a large amount of AWS accounts, it can be helpful to have a average bucket size per AWS account.

```
find aws_s3_bucket with bucketSizeBytes!=undefined as b 
return b.accountId as Account, 
SUM(b.bucketSizeBytes) as TotalSize, 
AVG(b.bucketSizeBytes) as AverageSize ORDER BY AverageSize DESC
```
This query will return the Account and its average bucket size sorted from largest to smallest, and total bucket bytes.

>This query uses the `AVG` function. The `AVG` function can be used with properties with integer values to produce an average. This query also uses the `SUM` function. The `SUM` function can be used with properties with integer values to produce a total.

#### S3 Monthly Cost By Account

A monthly bucket cost report can be generated using current S3 state.

```
find aws_s3_bucket with bucketSizeBytes!=undefined as b 
return b.accountId as Account, 
SUM(b.bucketSizeBytes) / 1000000000 * 0.023 as monthlyCost 
ORDER BY monthlyCost DESC
```

This query shows the monthly cost per account, ordered by monthly cost.

> Mathematical operations can be used in queries on integer values in queries. Operations include `+`, `-`, `/`, and `*`.

### Backup Costs

Visibility into current backup usage is another area that can be helpful when analyzing costs.

#### Current Backup Costs Across Org

Current backup costs can be calculated across accounts in an organization.

```
FIND (aws_db_cluster_snapshot|aws_db_snapshot) as snapshot
  RETURN
    snapshot.tag.AccountName as name,
    sum(snapshot.allocatedStorage) * 0.02 as value
```

This query will cost of backups per AWS account.

#### Current Disks not used by Hosts or for Backup

A list of storage disks that not being leveraged for backup or by a Host can help provide visibility into unnecessary storage costs.

```
Find Disk with _class!='Backup' that !uses Host
```

This query will return a list of Disk entities currently not in use.

>The `Disk` class contains several entity types including EBS Snapshots and EBS volumes. It should be noted that this query will return results from other Services outside of an AWS environnement.

#### Storage Analysis

Having a general overview of storage usage across an AWS environment is also important when performing a cost analysis exercise. 


```
FIND aws_db_cluster_snapshot with _source!="system-mapper" and allocatedStorage!=undefined AS s return s.allocatedStorage as Size, s.displayName, s.createdOn ORDER By Size DESC
```

```
Find aws_ebs_volume with size!=undefined as d RETURN d.size as Size, d.arn, d.createdOn ORDER BY Size DESC
```

```
Find aws_ebs_snapshot with volumeSize!=undefined as d RETURN d.volumeSize as Size, d.arn, d.createdOn ORDER BY Size DESC
```

These three queries show current Snapshot usage across an org.


## Unused Resources

Having visibility into the utilization of costly resources can be helpful when addressing the costs in an organization

### EC2 Usage

One common area of cost related issues can be the EC2 Service. There are a number of issues that can cause AWS environments to incur costs due to incorrect usage.

#### EC2 Usage Across Organization

Having visibility into current instance types across org can be beneficial when understanding the EC2 costs in an organization.

```
find aws_instance with instanceType!=undefined as a return a.accountId, a.instanceType, count(a)
```
This query will display current EC2 Instance Types across accounts, as a count total.



#### Inactive Instances



```
find Host with active = false and stateChangedOn < date.now - 60 day
```

>This query uses the `date.now` function. The date specified should be modified to an organization's desired timeline.

#### Inactive Instances with Volumes


```
Code or Query - If a query is used, make sure to provide a RETURN value
```


### Elastic IP Usage 

Elastic IP addresses is another costly service that can result in costs and security issues when not in use.

#### EIP Usage Across Organization

```
Code or Query - If a query is used, make sure to provide a RETURN value
```

#### EIP Not Currently Utilized

```
Code or Query - If a query is used, make sure to provide a RETURN value
```




## Resource Evaluation

Describe the concept and what relevant topics JupiterOne covers relating to this concept


### 
AWS EC2 Instances by 

Describe the _class, _type or relationship that is useful in understanding this concept


```
find unique aws_instance with instanceType!=undefined as a return a.instanceType, count(a)
```


_Explain the results of the query for the reader to understand. If <code>RETURN TREE</code> is used, include a screenshot explaining what they may see.</em> The results in this query show a User's name obtained through their <code>vendor_user</code> account

Explain a J1 specific concept in the memo. Think of these as things the user should know, but are not crucial to completing the task.

Alert/Dashboard/Question

Include an example of an alert or dashboard that may be relevant to this use case. Make sure to highlight where the nuance of customizing the alert should be.


### 
Conclusion
