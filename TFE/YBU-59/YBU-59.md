---
---
## About this lab
In this hands-on lab, you'll create the following AWS cloud infrastructure resources and objects required to deploy Yugabyte Platform and a Yugabyte Universe in a given AWS region:
- VPC
- Routing table
- Subnet
- Internet gateway
- Security group
- Key Pair
- EC2



## Requirements
Most importantly, you need a user account that is associated with  the ATS presales AWS account. Connect with IT Support (#it-support Slack channel). The IT Support team will provide access to this AWS account using Okta.



### About AWS resources, the Name tag, and the recommended naming convention
An object or resource in AWS Cloud has one or more **Tags**. A **Tag** is a key-value pair. Most objects have a default tag, the **Name** tag. The key is `Name`. You specify the value for this key.

The Name tag helps you identify the resources and objects that you create. As a matter of practice, always specify a value for the **Name** tag using a recommended naming convention. The recommended naming convention in this lab encapsulates the object type, you as a the creator, and the region and/or availability zone where the resource or object exists.

#### Why?
You are responsible for terminating and deleting your cloud resources. Therefore, you need to be able to easily identify the resources and objects that you create. Using the recommended naming convention, you will easily be able to identify what is yours among possibly hundreds of objects in the AWS account.  

> **Warning:** To avoid accidentally terminating or deleting the incorrect resources, please embrace the practice of using a recommended naming conventions.  Don't be the new team member that deletes the resources for a live POC !!! 


#### Recommended naming convention
Since you'll need to identify and later terminate or delete the resources and objects that your create in AWS, you must employ a standardized naming convention for all your resources and objects that you create. 

The recommended naming convention consists of several elements or parts, in this order, when applicable:
* an object or resource abbreviation and prefix
* a personal identifier
* a region abbreviation
* an availability zone abbreviation, when applicable

The following sections will start with a personal identifier. An object or resource name is typically a composite of all elements or parts.

##### Personal identifier abbreviation
For your personal identifier, you'll use the first 4 letters of your Yugabyte email address. If your email is less than four characters, you can add additional letters, A to Z. Here are some examples:

| Email | Personal identifier|
|:-|:-|
| sluersen@yugabyte.com | SLUE |
| mkim@yugabyte.com | MKIM |
| rao@yugabyte.com | RAOZ |


##### Region abbreviation
Using an abbreviation for a region name will help you quickly identify a resource or object in a given region. Quick identification minimizes costly mistakes. Here are examples of region abbreviations for AWS:

| Region | Region abbreviation | 
|:-|:-|
| US East (N. Virginia) us-east-1 | use1 |
| US East (Ohio) us-east-2 | use2 |
| US West (N. California) us-west-1 | usw1 |
| US West (Oregon)  us-west-2 | usw2 |
| Europe (Frankfurt)  eu-central-1 | euc1 |
| Europe (Ireland)  eu-west-1 | euw1 |
| Europe (London)  eu-west-2 | euw2 |



##### Availability zone abbreviation
An availability zone (AZ) represents an data center in a given region. There is often two or more availability zones in a given region. Here are some examples of availability zone abbreviations:

| Availability zone | AZ short name | Region + AZ short name |
|:-|:-|:--|
| US East (N. Virginia) us-east-1a | 1a-az4 |use1-1a-az4|
| US East (N. Virginia) us-east-1b | 1b-az6 |use1-1b-az6|
| US East (N. Virginia) us-east-1c | 1c-az1 |use1-1c-az1|
| US East (N. Virginia) us-east-1d | 1d-az2 |use1-1d-az2|
| US East (N. Virginia) us-east-1e | 1e-az3 |use1-1e-az3|
| US East (N. Virginia) us-east-1f | 1f-az5 |use1-1f-az5|



##### Resource and object prefix and abbreviations
In most cases, you can use a prefix for the object identifier, with Security Group names being one of the exceptions. As such, resource and object tag names are composite names that include your personal identifier, region, and when appblicable, an availability zone. Here are some examples:


| Resource / Object | Prefix  | Example Name |  Description | 
|:-|:-|:-|:-|
| VPC | vpc | `vpc-SLUE-use1` | A VPC is an isolated portion of the AWS Cloud populated by AWS objects, such as Amazon EC2 instances. |
| Subnet| sb | `sb-SLUE-use1-1a-az4` |  A subnet is a range of IP addresses in your VPC. You can launch AWS resources, such as EC2 instances, into a specific subnet. Subnets can be either public or private. The example details the region and zone. [Info](https://docs.aws.amazon.com/en_us/console/vpc/subnets) |
| Route Table | rt | `rt-SLUE-use1` | A route table specifies how packets are forwarded between the subnets within your VPC, the internet, and your VPN connection. [Info](https://docs.aws.amazon.com/en_us/console/vpc/route-tables/create-route-table) |
| Internet Gateway | igw | `igw-SLUE-use1` | An internet gateway is a virtual router that connects a VPC to the internet. [Info](https://docs.aws.amazon.com/en_us/console/vpc/internet-gateways#working-with-igw) |
| NAT Gateway | nat | `nat-SLUE-use1` | A highly available, managed Network Address Translation (NAT) service that instances in private subnets can use to connect to services in other VPCs, on-premises networks, or the internet. [Info](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) | 
| Peering connection | pcx | `pcx-SLUE-use1` | A VPC peering connection is a networking connection between two VPCs that enables you to route traffic between them privately. [Info](https://docs.aws.amazon.com/en_us/console/vpc/peering/create) | 
| Network ACLs | acl | `acl-SLUE-use1` | A network ACL is an optional layer of security that acts as a firewall for controlling traffic in and out of a subnet. [Info](https://docs.aws.amazon.com/en_us/console/vpc/network-acls#CreateACL) | 
| Security Group | sg | `SLUE-sg-use1` | A security group acts as a virtual firewall for your instance to control inbound and outbound traffic. It is specific to a region. You can not use **sg** as a prefix, so use your personal identifer as the prefix and then **sg**. [Info](https://docs.aws.amazon.com/en_us/console/ec2/security-groups) | 
| Key Pair | kp | `kp-SLUE-aws` | A key pair, consisting of a private key and a public key, is a set of security credentials that you use to prove your identity when connecting to an EC2 instance. A key pair results in a private key file format, either a `.pem` or `.ppk` file. You need this file to secure shell (SSH) into a EC2 instance. You need to keep this private key file secure. Only share it internally using Keybase (not Slack, GDrive, or email).  In AWS, you can copy a key pair from one region to another region. For this reason, the example shows the cloud provider name, `aws`, rather than a regional identifier. | 
| EC2| ec2 | `ec2-SLUE-use1-1a-az4-platform` | With EC2, you can create  virtual machines, or instances, that run on the AWS Cloud. This example includes ther region, the zone, and the purpose of the ec2 instances, for example `platform`.|


#### Summary
When completing the labs in this course, practice using the recommended naming convention. You'll find that this practice will helps you minimize errors and find objects quickly.

--- 
---

## Create your AWS cloud infrastructure
In this lab, you will create and edit various resources and objects required in a region to deploy the Yugabyte Platform and a Yugabyte Universe such as:
- a VPC
- a routing table
- a subnet
- an internet gateway
- a security group
- a key pair

>Note: In upcoming labs, you will create one or more EC2 instance using a specific AMI.


### Sign in to the AWS console
To sign in using SSO and Okta, follow these steps:
* In Google Chrome, open [Yugabyte Okta Applications](https://yugabyte.okta.com/app/UserHome).
* Select **AWS SSO**.
* In the Single Sign-On AWS App, select **AWS Account → yugabyte-ats-presales → Managament console**.


### Create your VPC for the us-east-1 region
After signing in, you can start creating your resources. The first resource that you will create is your Virtual Private Cloud (VPC). A VPC is an isolated portion of the AWS Cloud populated by your AWS objects, such as Amazon EC2 instances. 

Here are the steps to create your VPC:
* In the AWS Console, in the global navigation bar, in the Region drop down list, select **US East (N. Virgina) us-east-1**.
* In the global navigation bar, search for VPC, and then select **Services** → **VPC**.
* Next, select the **VPCs** card.
* In **Your VPCs**, select **Create VPC**.
* Specify the following properties:

| Property | Value |
|:-|:-|
| Name tag | Use a composite name for your VPC that includes your prefix and the select region, such as: `vpc-SLUE-use1`|
| IPv4 CIDR block | `IPv4 CIDR manual inut` |
| IPv4 CIDR | Enter a value for private range that allows for one or more  subnet ranges |


Your VPC requires an IPv4 CIDR block. To determine your IPv4 CIDR block, you can use a command-line tool such as `ipcalc` in terminal or a website such as [IPCALC](http://jodies.de/ipcalc).

Subnets are typically private, so use a private range. Here are some examples: 

| Class | Range |
|:- |:-|
| Class A | `10.0.0.0 – 10.255.255.255` |
| Class B | `172.16.0.0 – 172.31.255.255` |
| Class C | `192.168.0.0 – 192.168.255.255` |


Your VPC IPv4 CIDR block must be able to accomadate one or more subnets within the range without create conflicts for the subnet ranges.  Here is an example of a VPC IPv4 Class B private range with non-conflicting ranges for three subnets:

| Type | IPv4 CIDR | 
|:- |:-|
| VPC | `172.21.0.0/16`|
| Subnet 1 | `172.21.10.0/24` | 
| Subnet 2 | `172.21.12.0/24` | 
| Subnet 3 | `172.21.14.0/24` |


* To continue defining your VPC, leave the following properties set to their default values:

| Property | Value |
|:-|:-|
| IPv6 CIDR block | No IPv6 CIDR block |
| Tenancy | Default | 

* Select **Create VPC**.
* In Details view, select **Actions → Edit DNS hostnames**.
* In Edit DNS hostnames, select **Enable**.
* Select **Save Changes**.
* In the **Your VPCs** list view, find your VPC, and select the **VPC ID** link.
* In Details, select **Actions → Edit DNS resolution**.
* In Edit DNS resolution, select **Enable**.
* Select **Save Changes**.



### Create a subnet for your VPC
 A subnet is a range of IP addresses in your VPC. You can launch AWS resources, such as EC2 instances, into a specific subnet. Subnets can be either public or private. Here are the steps to create one or more subnets for your VPC:

* In the left sidebar, in Virtual Private Cloud, select **Subnets**.
* In the Subnets list view, select **Create subnet**.
* In **Create subnet**, specify the following properties:

| Property | Value |
|:-|:-|
| VPC - VPC ID | In the drop down list, select your VPC. |
| Subnet settings - Subnet name |  Specify the name of your subnet using the recommended naming convention for the region and availablity zone, for example, `sb-SLUE-use1-1a-az4` |
| Subnet settings - Avaiablity zone | Select the availablity zone for the subnet. Make sure your Subnet name encodes this selection, for example, `sb-SLUE-use1-1f-az5`. |
| Subnet settings -IPv4 CIDR blockInfo | Specify the IPv4 CIDR block for the subnet so that is does not conflict with any other subnets| 

 Here is an example of a VPC IPv4 Class B private range with non-conflicting subnets:

| Type | IPv4 CIDR | 
|:- |:-|
| VPC | `172.21.0.0/16`|
| Subnet 1 | `172.21.10.0/24` | 
| Subnet 2 | `172.21.12.0/24` | 
| Subnet 3 | `172.21.14.0/24` |


### Define VPC route table properties
A route table specifies how packets are forwarded between the subnets within your VPC, the internet, and your VPN connection. When you created your VPC, AWS automatically creates a main route table for your VPC. You need to give this main route table a name and also explicitly associate your subnet or subnets with it.

Here are the steps to edit the properties of the exisitng route table:

* In the left sidebar, in Virtual Private Cloud, select **Your VPCs**.
* In Filter VPCS, enter the name of your VPC, e.g, `vpc-SLUE-use1`.
* In the list view, select your VPC row.
* In the tabs view, in the Details tab,in Main route table, select the route table link.
* In Route tables, select **Actions → View details**.
* In Details, select **Actions → Manage tags**, and specify a value for the Name key property.


| Property | Value |
|:-|:-|
| Tags - Key - Name | Specify the name of your route table using the recommended naming convention for the region, for example, `rt-SLUE-use1` |

* Select **Save**.
* In Details, select **Actions → Edit subnet associations**.
* In Edit subnet associations, select your subnets in the list view.
* Verify the Selected subnets.
* Select **Save associations**.


### Create an Internet gateway
An internet gateway is a virtual router that connects your VPC to the internet. Here are the steps to create your internet gateway:

* In the left sidebar, in Virtual Private Cloud, select **Internet Gateways**.
* In Internet Gateways, select **Create internet gateway**.
* In Create internet gateway, specify the following properties:

| Property | Value |
|:-|:-|
| Internet gateway settings - Name tag | Specify the name of your internet gateway using the recommended naming convention for the region, for example, `igw-SLUE-use1` |

* In the Internet gateway details,  select **Actions → Attach to VPC**.
* In Attach to VPC, in VPC, in the drop down list, select your VPC.
* Select **Attach internet gateway**.



 ### Edit your route table
 Now that you have an internet gateway for your VPC, the next steps are to add your internet gateway to your route table:
* In the left sidebar, in Virtual Private Cloud, select **Route Tables**.
* If needed, use the Filter to find your route table.
* Select your route table in the list view.
* In the tab view, first select the **Routes** tab an then select **Edit routes**.
* In Edit routes, select **Add route**.
* In the new row, specifc the following column values:

| Column | Value | Comment | 
|:-|:-|:-|
| Destination | `0.0.0.0/0` | This is the IPv4 address for the entire internet, meaning this is a public internet route |
| Target | `igw-SLUE-use1` | First select Internet Gateway in the list. Then select your internet gateway that you just created. |

* Select **Save changes**.
* In the Routes tab view, confirm that there is a noew route destination of `0.0.0.0/0` with your  internet gateway.


### Edit your Network ACL
When you created your subnet, AWS created a Network ACL (Access Control List) for you. A network ACL is an optional layer of security that acts as a firewall for controlling traffic in and out of a subnet. Follow these steps to identify and name your Network ACL:

* In the left sidebar, in Security, select **Network ACLs**.
* To help identify your Network ACLs, you can filter the list using the name of your VPC or subnet.
* In the list view, select the Network ACL for your subnet.
* Select **Actions → Manage tags**.
* For the key property, enter `Name`.
* For the value property, enter the name of your Network ACL, such as `acl-SLUE-use1` .
* Select **Save**.


### Create a Security Group
A security group acts as a virtual firewall for your instance to control inbound and outbound traffic. It is specific to a region and consists of various rules for allowed traffic over certain internet protocols and ports.

> Note: You can not use **sg** as a prefix for the security group name, so use your personal identifer as the prefix and then **sg**. You can, however, use **sg** as a prefix for the object name.

Here are the steps to create your security group for your VPC:
* In the left sidebar, in Security, select **Security Groups**.
* Select **Create security group**.
* In Create security group, in Basic details, specifc the following properties:


| Property | Value |
|:-|:-|
| Basic details - Security group name  | Specify the name of your security group using the recommended naming convention, for example, `SLUE-sg-use1` |
| Basic details - Description | Enter a short description, for example, `sg-SLUE-use1`. |

* In Inbound rules, select **Add rule** to create a new rule row for the following:

| Type | Protocol |  Port range | Source (1) | Source (2) | Description - optional |
|:-|:-|-:|:-|-:|:-|
| SSH         | TCP  | 22    | Anywhere IpV4 | `0.0.0.0/0 ` | SSH |
| HTTP        | TCP  | 80    | Anywhere IpV4 | `0.0.0.0/0 ` | Web YB Platform |
| HTTPS       | TCP  | 443   | Anywhere IpV4 | `0.0.0.0/0 ` | Web YB Platform | 
| Custom      | TCP  | 5433  | Anywhere IpV4 | `0.0.0.0/0 ` | API endpoint YSQL |
| Custom TCP  | TCP  | 9042  | Anywhere IpV4 | `0.0.0.0/0 ` | API endpoint YCQL |
| Custom TCP  | TCP  | 7000  | Anywhere IpV4 | `0.0.0.0/0 ` | Web YB-Master Admin |
| Custom TCP  | TCP  | 7100  | Custom | Your VPC IPv4 CIDR Block  |  Internodal RPC YB-Master |
| Custom TCP  | TCP  | 8080  | Anywhere IpV4 | `0.0.0.0/0 ` |  Web YB Platform Http Alternate |
| Custom TCP  | TCP  | 8800  | Anywhere IpV4 | `0.0.0.0/0 ` |  Web Replicated for Platform
| Custom TCP  | TCP  | 9000  | Anywhere IpV4 | `0.0.0.0/0 ` |  Web YB-TServer Admin
| Custom TCP  | TCP  | 9090  | Anywhere IpV4 | `0.0.0.0/0 ` |  Web Prometheus
| Custom TCP  | TCP  | 9100  | Custom | Your VPC IPv4 CIDR Block  |  Internodal RPC YB-TServer |
| Custom TCP  | TCP  | 9300  | Anywhere IpV4 | `0.0.0.0/0 ` |   Prometheus Node targets |
| Custom TCP  | TCP	 | 12000 | Custom | Your VPC IPv4 CIDR Block  |    YB-TServer API YCQL |
| Custom TCP  | TCP	 | 13000 | Custom | Your VPC IPv4 CIDR Block  |   YB-TServer API YSQL|


* In Outbound rules, leave the **All traffic** `0.0.0.0/0 `  settings.
* In Tags - optional, select Add new tag. 
* In the Tag row, enter `Name` for the key, and for the value, specify the name of the resource, such as `sg-SLUE-use1`. 
* Select **Create security group**.
* Verify the succesfuly creation of the security group.


##### Additional resources
Here are some helpful links in Yugabyte documentation related to Security Groups:
- [https://docs.yugabyte.com/latest/reference/configuration/default-ports/#root](https://docs.yugabyte.com/latest/reference/configuration/default-ports/#root)
- [https://docs.yugabyte.com/latest/yugabyte-platform/install-yugabyte-platform/prepare-environment/aws/#create-a-new-security-group-optional](https://docs.yugabyte.com/latest/reference/configuration/default-ports/#root)




TODO: Create Key pair

TODO: * Optional: Depending on your region, add at least one more subnet by selecting **Add new subnet** and specify the subnet properties. 
* Select **Create subnet**.
