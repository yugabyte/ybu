# Run a Workload on a Multi-node Universe

## Introduction

In this hands-on lab, you will use xCluster replication to connect two independent YugabyteDB universes in a multi-region topology. The reason why a client would want this type of topology is to create high availability and resiliency in different geographical regions.

Up until now, you have created a single cluster that is geographically distributed across different availability zones. You can also stretch a cluster across different geographic regions as well. These topologies are considered synchronous replication.

But many clients often can't justify the additional complexity or operational costs involved with a synchronous replication in a multi-regional topology. These clients can use the asynchronous replication offered by xCluster to save and simplify, although the trade off is a decrease in consistency from transactional to timeline consistency due to the asynchronous nature of the replication.

For a deeper dive into the different types of regional replication methodologies and their use cases, review the [Yugabyte docs on multi-region deployment](https://docs.yugabyte.com/latest/explore/multi-region-deployments/).

In this lab, you will connect two independent three node multi-zone clusters. You will use AWS as the cloud provider to deploy both clusters. One cluster will be in located in us-west-2 and the other will be in the us-east-1. You will begin by implementing unidirectional replication, meaning a source cluster will replicate its data to a target cluster. Any writes made to the source cluster will be replicated to the target cluster. Any writes made to the target cluster however, will not be replicated to the source cluster. Hence the unidirectional data flow only goes from the source to the target, in our case the west to the east. This is known as an Active-Passive set up. 

After that, you will implement a bidirectional replication, meaning both clusters will replicate data changes to the other cluster region. This is known as an Active-Active replication.

It is important to note that with asynchronous replication, there is no longer a dependency on the Raft consensus algorithm. Instead, the YugabyteDB layer uses the change data capture (CDC) tool to ensure that changes in the data are automatically applied to remote data repositories. Similar to the Raft consensus, CDC is applied at the DocDB layer and therefore works with both YSQL and YCQL APIs. For a closer look at the [CDC technology take a look at the Yugabyte Docs.](https://docs.yugabyte.com/latest/architecture/docdb-replication/change-data-capture/)

> **Important:** Although the Raft Consensus synchronous replication method guarantees transactional consistency, CDC asynchronous replication can only offer timeline consistency. [For a deeper dive on the different types of consistency models in relation to replication view the following article on Wikipedia.](https://en.wikipedia.org/wiki/Consistency_model#Consistency_and_replication)

### Objective

As a sales engineer, I want to connect two YugabyteDB clusters located in different regions to demonstrate xCluster replication.

## Requirements

* A deployed YugabyteDB cluster on AWS in us-east-1 on Platform version 2.11.2.

* A deployed YugabyteDB cluster on AWS in us-west-2 on Platform version 2.11.2..

* Both `.pem` key files to connect of their respective YugabyteDB servers hosting Platform.

* Yugabyte Platform credentials

## Cluster setup

In the initial set up for both clusters, there are a couple things to keep in mind. 

First, it's recommended to use the same versions of Platform to avoid any configuration errors. This lab demonstrates xCluster replication with two cluster on Platform version 2.11.2. To get a detailed step by step lab on how to upgrade your Platform version, take a look at **LAB: Platform Version Update**

Second, you must change the length of time the YB-Master will declare a node unhealthy since this topology requires remote network communication. This is done with a G-Flag called **--leader_failure_max_missed_heartbeat_periods**. We will extend this time from the default value of 6 seconds to 10 seconds. You can do this in the Platform UI by following the step by step directions in **LAB: Rolling Config Changes**.

## Create a Peering Connection

To connect two regions together in AWS, a peering connection is needed to link the regional VPCs together. This route will allow a cluster to replicate data to the other cluster.

A virtual private cloud or VPC is an isolated virtual network located in a region. For more information about [how VPCs work, visit the AWS docs.](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) 

### Create the peering connection request

To create a peering connection you must assign requester and accepter roles to each VPC. In this lab, you will assign the VPC in the west to become the requester and the VPC in the east region to be the accepter. This designation is actually arbitrary and can easily be reversed, but should be designated in order to avoid confusion, especially if more VPCs need to be connected. For a closer look at [VPC peering visit the AWS docs.](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)

To create the peering connection request following the instructions:

* Navigate to the AWS VPC console in the us-west-2 region in the browser. 

* Make a note of the VPC id for the requester VPC in the us-west-2 region. You will need this later to accept the peer connection. 

* Make a note of the CIDR block for the requester VPC. This will be added to the routing table in the accepter VPC later in this lesson.  

* Select the Peering Connections option located on the left menu in the Virtual Private Cloud section as shown in the following image:

![VPC in US West (Oregon).](./assets/images/100-vpc_west_1600x700.png)

* Select the **Create peering connection** button on the top right side of the page.

* Name the peering connection using the standard naming convention, pc-<4-letter-username>-usw2-use1, to state the regions that will be connected, for example, **pc-CKIM-usw2-use1**.

* Select the local VPC in that region that will be connected, in this case, vpc-CKIM-usw2.

* Once the VPC is selected, note that the CIDR block is automatically populated.

> **Important:** In order for the peering connections to work, the CIDR blocks cannot overlap. This would cause a conflict and make the peering connection fail.

* Select the region that contains the VPC you wish to connect to, in this case **us-east-1**.

* Enter the VPC ID of the accepter VPC, located in the **us-east-1** region.

* Add the Name tag, pc-CKIM-usw2-use1, to identify the regions being connected.

* Select **Create peering connection** to provision this network connection.

* If successful, a message states that the peering connection request has been created, however to make this connection active, this request must be accepted.

### Accept the peering connection request

* Navigate to the us-east-1 VPC console in the browser.

* Make a note of the VPC id for the accepter VPC in the us-east-1 region. You will need this later to validate the peer connection.

* Make a note of the CIDR block for the accepter VPC. This will be added to the routing table in the requester VPC later in this lesson.

* Select the **Peering Connections** option in the menu on the left side of the page.

* Select the radio button of the peering connection that is in the state **Pending acceptance**. The peering connection can further be validated by comparing the VPC IDs of the two VPCs being connected.
  
* Select the Action drop down at the top right of the page.

* Select **Accept request**.

* Verify the VPC IDs are correct and select **Accept request**.

Now that the peering connection has been created, one last step is needed to ensure this network connection has been made. You need to add the CIDR block to the routing table of each VPC. This ensures that requests will be routed to the correct address.

### Add the peering connection routes

Now that the peering connection has been established, you must add the final step to route the requests to the correct address. You will do this by adding a route to the routing table in the respective VPC. By adding the respective CIDR block of each VPC, you can enable bidirectional communication between the two clusters.

* In the us-east-1 VPC console, select **Your VPCs** to view all the VPCs in this region.

* Select the accepter VPC, in this case vpc-SLUE-use1. This will navigate you to the details page of the accepter VPC. 

* Select the **Main route table** to navigate to the Route tables page.

* Scroll down to and select the **Routes** tab.

* Select **Edit routes**.

* Select **Add route**.

* In the Destination field add the CIDR block for the requester VPC in the west region.

* In the Target field select **Peering Connection** and add the connection you created, for example, **pc-MKIM-usw2-use1**.

* Select Save changes.

* Go the the us-west-2 region and add the CIDR block for the accepter VPC for the routing table for same peering connection and save the changes.

Now the network route between the VPCs has been established, you need to verify that the connection work.

One way to test this connection is to SSH into the EC2 instance hosting Platform in the west region. Then try to SSH from that EC2 instance into the EC2 instance in the east region.

You will need to carefully copy the respective .pem key file with a notepad or text editor, not an IDE, to the EC2 instance in order to establish this connection. Remember to delete this key once this test has completed for security concerns.

## Create xCluster replication

Now that the network provisioning work has been completed, the remaining tasks to create an xCluster with unidirectional replication will be accomplished with Yugabyte Platform.

In this section here are the steps you'll follow to create a unidirectional replication:

* Create the identical table in both clusters
 
* Retrieve the universe ID

* Retrieve the Master addresses

* Retrieve the table ID

* Configure the universe with the replication command

### Create the table to replicate

> **Important:** First you must create a table on both clusters that are identical. With xCluster replication, you must designate which table(s) will be replicated. xCluster allows you to be selective about which tables are mission critical and HA.

In both clusters, create a `keyvalue` table using the following steps:

* SSH into the EC2 instance that hosts Yugabyte Platform. The prompt will change to **[centos@platform ~]$** if the EC2 was launched with centos. 

* SSH into the node that contains the YB-Master leader. You can find this information in the Yugabyte Platform UI. Once you sign in and select the universe, select the **Nodes** tab. Then select the Leader, indicated as the Master (Leader) in **Processes**. Then choose **Connect** from the **Actions** drop down list. Execute this script in the EC2 instance terminal that was accessed in the previous step to be connected to the node that contains the YB-Master leader. 

If successful, the prompt will change again to display the IP address and the user as shown in the following example:

```bash
[yugabyte@ip-172-152-110-225 ~]$
```
* Next enter the following script command to access the YSQL shell:

```bash
/home/yugabyte/tserver/bin/ysqlsh -h <my-node-IP-address> -p 5433 -U yugabyte
```

Use the IP address of the node in the proceeding script command.

* Connect to the `postgres` database with the following command:

```bash
yugabyte=# \c postgres
```

* Create the **keyvalue** table with the following command:
  
```bash
CREATE TABLE keyvalue (k int, v int, PRIMARY KEY (k));
```

* Populate the table with data with the following command:

```bash
INSERT INTO keyvalue VALUES (1, 2);
```

* Verify the table and data are in the table with the following command:

```bash
SELECT * FROM keyvalue;
```

If the table and data were populated successfully, you will get the following response in the terminal:

```bash
k | v
---+---
 1 | 2
```

### Get the cluster identifier and master addresses in the source cluster

Now that the table has been created, you will need to retain the identifier id for the cluster and node's master addresses in order to configure the xCluster replication.

* Navigate to the source cluster node YSQL shell terminal, and run the **exit** command. This will close the YSQL shell and return to the node instance terminal.

* In the terminal for the node, enter the following command to view the configuration for the cluster as shown in the following script command:
  
```bash
[yugabyte@ip-172-152-110-225 ~]$ cat /home/yugabyte/tserver/conf/server.conf
```

This command returns the data needed to assign xCluster replication. The key properties you must write down are the **tserver_master_addrs** and the **cluster_uuid** for the SOURCE cluster.

```bash
--placement_cloud=aws
--placement_region=us-west-2
...
--tserver_master_addrs=172.151.37.55:7100,172.151.62.129:7100,172.151.78.12:7100
...
--cluster_uuid=e069287c-534b-41e8-83c8-c0659c0bacea
...
--leader_failure_max_missed_heartbeat_periods=10
```

> **Pro Tip:** These G-flag attributes enables fine tuning of the YugabyteDB cluster which makes the YugabyteDB run more efficiently and productively to a clients' apps specific needs.

Take special note that the following properties and their values in the proceeding response:

* tserver_master_addrs

* cluster_uuid

* leader_failure_max_missed_heartbeat_periods

The **tserver_master_addrs** are the needed to use the `yb-admin` tool to enable xCluster replication. This property is referred to as the **master_addresses** in Yugabyte Docs. Note that these master addresses are assigned to port 7100. Here is an example of what the **master_addresses** looks like, 172.152.110.225:7100,172.152.79.76:7100,172.152.87.11:7100 for a three node cluster.

The **cluster_uuid** is the unique identifier for the universe. This will be used to identify the source and target universes. Here is an example of what a **cluster_uuid** looks like: e069287c-534b-41e8-83c8-c0659c0bacea.

Note that the G-flag property, **leader_failure_max_missed_heartbeat_periods**, has the value of 10. You assigned this value at the beginning of the lesson in order to enable the master to take into account the larger latency involved with geographically distant communication.

### Get the table id 

In order to correctly configure cross cluster replication, you need to not only identify the cluster ids, but also the table ids. Note that you will connect one table in this lab, but multiple tables can also be configured for replication.

To retrieve the table id, you will use the **yb-admin** utility. 

The **yb-admin** utility is a powerful command line interface that is used to manage the cluster's attributes. You can retrieve important information in much more detail than possible in the Platform UI by using the commands such as **list all master** to find out the master addresses as well as which node contains the leader. You can also get information about the cluster and if there are any current producers or replications currently set up on the cluster with the command, **get_universe_config**. For more  Refer to the [Yugabyte docs to find a detailed list of commands for the yb-admin utility.](https://docs.yugabyte.com/latest/admin/yb-admin/)

> **Important:** In order the use the yb-admin utility, the master addresses of the nodes must be added for every command. Currently there is an [open GitHub issue](https://github.com/yugabyte/yugabyte-db/issues/8844) to disassociate this dependency.

To retrieve the table id using the **yb-admin** utility enter the following script in the terminal of the leader node:

```bash
/home/yugabyte/master/bin/yb-admin \
    -master_addresses \ <master address in source cluster> \
    list_tables include_table_id | grep postgres.keyvalue
```

Replace the master addresses in your source cluster in the statement above. Note that the list tables search will look for the table **postgres.keyvalue**.

Now that the cluster id, master addresses, and table id have been retrieved for the source cluster, repeat the proceeding steps for the target cluster in the east region as well. You will need this information for the xCluster replication.

### Configure unidirectional replication

In order to create a replication of the table, **keyvalues**, in a cross cluster asynchronous replication, you will need to configure the target cluster to replicate from the source cluster. You will use **yb-admin** to administer this replication pattern.

In the target cluster's leader node's terminal, populate the following command with the information retrieved from the previous step:

```bash
/home/yugabyte/master/bin/yb-admin \
-master_addresses \
<target-master-addresses> \
setup_universe_replication <source-universe_uuid> <source_master_addresses> <source-table-ids>
```

For more information regarding these attributes visit the [Yugabyte Docs regarding setup for universe replication.](https://docs.yugabyte.com/latest/admin/yb-admin/#setup-universe-replication)

> **Pro tip:** The IP addresses for the master addresses and the tserver addresses use the IP address of the nodes. The distinction between the master addresses and tserver addresses are the port numbers, 7100 for masters and 9100 for the tservers respectively.

If the proceeding script ran successfully 

## Reflection

The purpose of this lab was to demonstrate how to execute a YSQL workload on the three node multi-zone Universe.

Multiple workloads can now be added to the Universe to benchmark performance. High availability and resiliency can also be demonstrated by removing a node. 