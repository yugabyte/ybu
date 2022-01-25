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

## Platform Server Configuration

Now the the Platform server is running, the following steps will configure and install the Platform server.

### Platform Server Initial Configuration

1. Set the hostname `sudo hostnamectl set-hostname platform`

2. Update the OS: `sudo yum -y update`

3. Install `wget`: `sudo yum install -y wget`

4. Install `curl`: `sudo yum install -y curl`


### Install Replicated on the Platform Server

Yugabyte uses a 3rd party tool called Replicated, to manage license and version control for the Yugaware Platform component. This Replicated tool runs as a set of Docker containers, on a linux server. An install script is provided which installs the correct version of Docker, then downloads and runs the required containers from the Replicated repository. This tool is installed on the Platform server (the same server on which Yugaware Platform is to be installed).

In order to complete this step the Platform 2.11(latest) license, `.rli` extension, from a Yugabyte representative must to received first.

Logged in as the user, `centos`, run the following command on the Platform server to run the Yugaware Platform:

```bash
curl -sSL https://get.replicated.com/docker | sudo bash
```

To verify that the Replicated has successfully installed the Yugaware platform on your server, you will receive the following output in your terminal:

```bash
Operator installation successful
To continue the installation, visit the following URL in your browser:
  http://34.105.238.92:8800
To create an alias for the replicated cli command run the following in your current shell or log out and log back in:
  source /etc/replicated.alias
```

The url in your message will reflect the public IP of your server.

The Yugaware Platform will now be available on port 8800. 
Navigate to the interface to complete the Yugaware installation.


## Create a Universe



## Terminate a Universe