# Create a Yugabyte Multi-Zone Universe using Platform (Anywhere)

## Introduction

In this hands-on deployment lab, you will create a Yugabyte Universe that consists of a three node cluster in a multi-zone topology. Each node will reside in the same region, but in a different availability zone. The purpose is to demonstrate YugabyteDBs ability to provide high availability and workload distribution. The added resiliency in a multi-zone architecture protects against potential failures in case resources in a single availability zone become unavailable. This topology can withstand a single zone failure, but not two or more.

The architecture of this three-node database cluster includes a stand-alone server that hosts the Platform management component and an additional driver server that will host test applications and tools. 

> **TODO:** add screenshot of architecture diagram
## Prerequisites

Before creating a universe, the cloud provider environment must first be configured according to the specifications found on the [Yugabyte docs page on cloud configuration.](https://docs.yugabyte.com/latest/yugabyte-platform/configure-yugabyte-platform/set-up-cloud-provider/aws/) This is to secure the database as well as create access points in the VPC to allow YugabyteDB to connect and communicate with the different nodes in the cluster.

> **Important:** Make a careful note which region contains the VPC and EC2 instance.

In this guide, we will focus on deployment with AWS.

## Checklist of necessary steps to install Yugabyte Platform

1. Complete AWS prerequisites (IAM role, access keys, Routing Table Entry, Security Group, VPC + subnets, Internet Gateway)

2. Obtain AWS IAM role access ID and secret

3. Obtain Yugaware license file (.rli) from Yugabyte Representative

4. Create AWS EC2 instance to run Yugaware Platform

5. Install Yugaware platform on EC2 server

6. Create AWS provider in Yugaware Platform

7. Create a YugabyteDB Universe via Yugaware Platform


<!-- ## User Stories

* As a user, I want to deploy a 3 node cluster, one node in each availability zone.

* As a user, I want to each node to reside in a different availability zone. -->

### Launch the EC2

Once the VPC, Security Group, IAM role, Subnets, Routing Table, and Internet Gateway have been set up, we can proceed with launching the EC2 instance using an AMI.

To begin, log into the Amazon account and navigate to the EC2 console to launch an instance with the following specifications:

* Instance type: c5.2xlarge

* IAM instance template: CentOS 7.9.2009 x86_64 - ami-00e87074e52e6c9f9 or a suitable privately created CentOS 7.9 image

* Network VPC and Subnets: <as created in the prerequisite>

* Disk storage: gp3 16GB for sample workloads, not ephemeral

* VPC network: <as created in the prerequisite>

* Security Group: <as created in the prerequisite>

* SSH access key: <generate and save a new key if required>

In this lab, we will be using a `c5.2xlarge`. The specifications of this instance type allow for the necessary processing power, 8 cores and 16GiB RAM, that are necessary to demonstrate Yugabyte Platform effectively as well as run sample workloads.

On a production workload, it is recommended to use a minimum of 16 cores and 32 GiB memory, if not more depending on the client. (i.e. AWS EC2 instance type c5.4xlarge)

The EBS volume can also be increased if a larger workload is desired for a demonstration.

> **Important:** When creating the EC2 instance, ensure the Public IP address setting is ENABLED if it is desired to connect to the Platform server from outside the AWS VPC (eg. from users’ workstations or applications running in Kubernetes environments).

SSH connect into the EC2 once it is running to install the tools and the Platform server.

### Install Replicated

Before we can continue, check to ensure that the necessary tools are available in your EC2, namely `wget` and `curl`, with the commands, `wget --version` and `curl -version`.

If these tools are not available, install them now.




## Platform Installation

## Create a Universe

## Terminate a Universe

# Create a Yugabyte Multi-Zone Universe using Platform (Anywhere) with AWS

## Introduction

In this guide, we will be deploying a three node cluster, with each node residing in a separate availability zone. A multi-zone deployment will enable the system to withstand a failure in one of the nodes with no impact on the system.



### Launch the EC2

Once the VPC, Security Group, IAM role, In this guide, we will be using a `c5.2xlarge`. The specifications in this instance type allow for the amount of processing power, with 8 cores and 16GiB RAM necessary to demonstration Platform effectively.

The EBS volume can also be increased if a large workload is necessary for a demonstration. 

For the VPC configuration, Security Group, and IAM roles details, please refer to the [Yugabyte docs page on cloud configuration for Platform.](https://docs.yugabyte.com/latest/yugabyte-platform/configure-yugabyte-platform/set-up-cloud-provider/aws/) 

### Platform server 

Before we can continue, check to ensure that the necessary tools are available in your EC2, namely `wget` and `curl`, with the commands, `wget --version` and `curl -version`.

If these tools are not available, install them now.

```


## Create a Universe



## Terminate a Universe