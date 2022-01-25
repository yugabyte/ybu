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

4. Create AWS EC2 instance to run Yugaware platform

5. Install Yugaware platform on EC2 server

6. Create AWS provider in Yugaware platform

7. Create a YugabyteDB Universe via Yugaware platform


<!-- ## User Stories

* As a user, I want to deploy a 3 node cluster, one node in each availability zone.

* As a user, I want to each node to reside in a different availability zone. -->

### Launch the EC2

Once the VPC, Security Group, IAM role, Subnets, Routing Table, and Internet Gateway have been set up, we can proceed with launching the EC2 instance using an AMI. The EC2 will perform as the server that will host the Yugaware platform.

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

> **Important:** When creating the EC2 instance, ensure the Public IP address setting is ENABLED if it is desired to connect to the server from outside the AWS VPC (eg. from users’ workstations or applications running in Kubernetes environments).

SSH connect into the EC2 once it is running to configure the environment for the Yugaware platform.

## Install the Yugaware Platform

Now the the server is running, the following steps will configure and install the Platform server.

### Initial Config

Once connected to the server by SSH, perform the initial configure steps to install the Yugaware platform.

1. Set the hostname `sudo hostnamectl set-hostname platform`

2. Update the OS: `sudo yum -y update`

3. Install `wget`: `sudo yum install -y wget`

4. Install `curl`: `sudo yum install -y curl`

### Install Replicated on the Server

Yugabyte uses a 3rd party tool called Replicated, to manage license and version control for the Yugaware platform component. This Replicated tool runs as a set of Docker containers, on a linux server. An install script is provided which installs the correct version of Docker, then downloads and runs the required containers from the Replicated repository. This tool is installed on the Platform server (the same server on which Yugaware platform is to be installed).

In order to complete this step the Platform 2.11(latest) license, `.rli` extension, from a Yugabyte representative must to received first.

Logged in as the user, `centos`, run the following command on the Platform server to run the Yugaware platform:

```bash
curl -sSL https://get.replicated.com/docker | sudo bash
```

Once the script has run, in the terminal, a series of prompts will begin.

```bash
Do you want to the IP address of the machine?
```

Reply with the default answer: Yes

```bash
Do you need a proxy to connect to the internet?
```

Reply with the default answer: No

Once these questions have been answered, Replicated will be installed on your server.

To verify that the Replicated has been installed on your server, you will receive the following output in your terminal:

```bash
Operator installation successful
To continue the installation, visit the following URL in your browser:
  http://xx.xxx.xxx.xx:8800
To create an alias for the replicated cli command run the following in your current shell or log out and log back in:
  source /etc/replicated.alias
```

The url in your message will reflect the public IP of your server. At port 8800 will be the Replicated interface that will allow us to complete the Yugaware installation.

### Complete the Yugaware Installation with the .rli license

In the last step, we configured the server and installed Replicated next step we will interface with Replicated and verify the Yugabyte license.

Navigate to the IP address of your server at port 8800 in your browser to connect to Replicated's interface.
You will see the following webform:

> **TODO: Image** Replicated UI form

1. Select "Continue to Setup"

2. Next, you will see a warning page regarding security risks due to a self-signed certificate.

3. Select "Advanced" and then select "Accept the risk and continue".

4. On the next screen titled HTTPS for admin console, we will select "Use the Self-Signed Cert".

5. On the resulting pop-up, select "Continue without a hostname". 
Once the internal setup has completed, the next screen will prompt you to Upload your license.

6. Select "Choose license" to upload the `.rli` file from your machine.

7. Once uploaded, select "Online" on the next screen and select "Continue".

8. The next screen will offer a list of Yugaware versions covered by the license.
   Select the "Latest" version, 2.11.

9. The next screen will progress through a long list of pre-flight checks which will turn to green once verified.
   Select "Continue" to proceed with the installation.

10. The next screen titled Settings will consist of a web form. Make sure that the field "Hostname" contains the server's IPV4 public IP.

11. Leave the other fields defaults as-is and select "Save" at the bottom of the screen.

12. A dialog box will appear. Select the button "Settings Saved - Restart Now". This will begin the Yugaware installation process.

13. The next screen will be the Replicated dashboard that indicates the status of the Yugaware installation. 
    Once the status has changed
    

## Create a Universe



## Terminate a Universe