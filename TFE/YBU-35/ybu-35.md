# Run a Workload on a Multi-node Universe

## Introduction

In this hands-on lab, you will use xCluster replication to connect two independent YugabyteDB universes in a multi-region topology. The reason why a client would want this type of topology is to create high availability and resiliency in different geographical regions.

Up until now, you have created a single cluster that is geographically distributed across different availability zones. You can also stretch a cluster across different geographic regions as well. These topologies are considered synchronous replication.

But many clients often can't justify the additional complexity or operational costs involved with a synchronous replication in a multi-regional topology. These clients can use the asynchronous replication offered by xCluster to save and simplify, although the trade off is a decrease in consistency from transactional to timeline consistency due to the asynchronous nature of the replication.

For a deeper dive into the different types of regional replication methodologies and their use cases, review the [Yugabyte docs on multi-region deployment](https://docs.yugabyte.com/latest/explore/multi-region-deployments/).

In this lab, you will connect two independent three node multi-zone clusters. You will use AWS as the cloud provider to deploy both clusters. One cluster will be in located in us-west-2 and the other will be in the us-east-1. You will begin by implementing unidirectional replication, meaning a source cluster will replicate its data to a target cluster. Any writes made to the source cluster will be replicated to the target cluster. Any writes made to the target cluster however, will not be replicated to the source cluster. Hence the unidirectional data flow only goes from the source to the target, in our case the west to the east. This is known as an Active-Passive set up. 

After that, you will implement a bidirectional replication, meaning both clusters will replicate data changes to the other cluster region. This is known as an Active-Active replication.

### Objective

As a sales engineer, I want to connect two YugabyteDB clusters located in different regions to demonstrate xCluster replication.

## Requirements

* A deployed YugabyteDB cluster on AWS in us-east-1.

* A deployed YugabyteDB cluster on AWS in us-west-2.

* Both `.pem` key files to connect to both YugabyteDB servers hosting Platform.

## Create a Peering Connection

To connect two regions together in AWS, a peering connection is needed to link the regional VPCs together. This route will allow a cluster to replicate data to the other cluster.

A virtual private cloud or VPC is an isolated virtual network located in a region. For more information about [how VPCs work, visit the AWS docs.](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) 

### Create the peering connection request

To create a peering connection you must assign requester and accepter roles to each VPC. In this lab, you will assign the VPC in the west to become the requester and the VPC in the east region to be the accepter. This designation is actually arbitrary and can easily be reversed, but should be designated in order to avoid confusion, especially if more VPCs need to be connected. For a closer look at [VPC peering visit the AWS docs.](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)

To create the peering connection request following the instructions:

* Navigate to the AWS VPC console in the us-west-2 region 

* Select the Peering Connections option located on the left menu in the Virtual Private Cloud section as shown in the following image:

![VPC in US West (Oregon).](./assets/images/100-vpc_west_1600x700.png)

* Select the **Create peering connection** button on the top right side of the page.

* Name the peering connection using the standard naming convention, pc-<4-letter-username>-usw2-use1, to state the regions that will be connected, for example, **pc-CKIM-usw2-use1**.

* Select the local VPC in that region that will be connected, in this case, vpc-CKIM-usw2.

* Once the VPC is selected, note that the CIDR block is automatically populated.

> **Important:** In order for the peering connections to work, the CIDR blocks cannot overlap. This would cause a conflict and make the peering connection fail.

* Select the region that contains the VPC you wish to connect to, in this case us-east-1.

* Enter the VPC ID of the accepter VPC in the east region.

* Add the Name tag, **pc-CKIM-usw2-use1**.

* Select **Create peering connection**.

* A success message states that the peering connection request has been created, however to make this connection active, this request must be accepted.

### Accept the peering connection

* Navigate to the us-east-1 VPC console.

* Select the peering connections option in the menu on the left side of the page.

* 



### Verify Workload is Running in the Universe

In the last step, you ran a YSQL workload on our Yugabyte Universe. In this step, you will verify the workload is running and review the metrics tools. 

Navigate back to the Yugabyte Platform Console and select the Universe that contains the workload. On the Universe details page in the "Overview" tab, you can see following activity:

![The Universe details page displays the metrics of the workload.](./assets/images/200-workload_metrics_1366x768.png)

On the "Overview" tab, the "Total Ops/Sec" displays the reads and writes being performed by the workload. In the "Tables" window, we can see a YSQL table has been inserted into the database.

Select the "Tables" tab to see that a table, `postgresqlkeyvalue`. Review the "Health" and "Metrics" tabs to measure the performance of Yugabyte.

## Reflection

The purpose of this lab was to demonstrate how to execute a YSQL workload on the three node multi-zone Universe.

Multiple workloads can now be added to the Universe to benchmark performance. High availability and resiliency can also be demonstrated by removing a node. 