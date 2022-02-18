<!--  Rolling Configuration Change using G Flags -->
<!-- Naming convention
VPC
vpc-SLUE-use1
vpc-&lt;MY_4_CHAR&gt;-use1

Subnet
sb-SLUE-use1-1a-az4

Routing Table
rt-SLUE-use1

Internet Gateway
igw-SLUE-use1

Security Group
sg-SLUE-use1
S3 Bucket
s3-SLUE-use1
s3-&lt;MY_4_CHAR&gt;-use1
IAM role - EC2
iamrole-YBU

NAT gateway
nat-SLUE-use1

Universe
universe-slue-use1

Peering Connection
pcx-SLUE-use1

ACL
acl-SLUE-use1

pem key
kp-SLUE-use1

ec2-SLUE-use1-1a-az4-platform
ec2-&lt;MY_4_CHAR&gt;-use1-1a-az4-platform

#### Personal identifier abbreviation
For your personal identifier, you'll use the first 4 letters of your Yugabyte email address. If your email is less than four characters, you can add additional letters, A to Z. Here are some examples:

| Email | Personal identifier abbreviation |
|:-|:-|
| `sluersen@yugabyte.com` | `SLUE` |
| `mkim@yugabyte.com` | `MKIM` |
| `rao@yugabyte.com` | `RAOZ` |

MY_4_CHAR = '<MY_4_CHAR>'
echo $MY_4_CHAR
cd
mkdir .ssh
ls -a | grep .ssh
chmod 400 ~/Downloads/kp-${MY_4_CHAR}-aws.pem
mv ~/Downloads/kp-${MY_4_CHAR}-aws.pem ~/.ssh/kp-${MY_4_CHAR}-aws.pem
cat ~/.ssh/kp-${MY_4_CHAR}-aws.pem
```
 -->
### About this lab

In this hands-on lab, you will perform a rolling configuration change of a cluster on Yugabyte Platform. A rolling configuration change is one method of replacing current nodes for new ones that are required due to an update to the underlying infrastructure. You will accomplish this task by updating the G-flag on the Yugabyte Platform Admin console as well as adding a timeout flag. 

The main reason why a cloud environment needs to be updated is because software intrinsically gets stale over time. Whether it's bug fixes, version upgrades, dependency changes, or feature additions; regular maintenance is a required task for highly resilient systems. 

"If you are not moving forward, you are moving backward." ~ Gorbachev

A rolling change is a common method to implement a configuration change to a cluster of nodes since there is minimal change in reliably and performance of the database.

### Objective

As a sales engineer, I want to perform a rolling configuration change to my Yugabyte Universe without a noticeable change in the cluster's performance.

### Requirements

Here are the requirements for this lab:

* A deployed YugabyteDB cluster (a Universe)

* Yugabyte Platform credentials

* A .pem  file that you can use to connect to the EC2 instance of your YugabyteDB Platform host

* The Yugabyte Universe must be running a workload.

### Resources

To find how to deploy a Yugabyte Universe on Platform review **LAB: Create a Yugabyte Multi-Zone Universe using Yugabyte Platform**. 
To find how to run a sample workload on Yugabyte Platform, review **LAB: Run YB-Sample-Apps from Yugabyte Platform.**

### Edit configuration flags

"Adding and modifying configuration flags for your YB-Master and YB-TServer nodes in a YugabyteDB universe allows you to resolve issues, improve performance, and customize functionality." ~ [Yugabyte Docs](https://docs.yugabyte.com/latest/yugabyte-platform/manage-deployments/edit-config-flags/)

In order to edit a configuration flag for a Yugabyte Universe, navigate to the Yugabyte Platform Admin console located at the public IP of your EC2 instance. If created from `LAB: Create a Yugabyte Multi-Zone Universe using Yugabyte Platform` it will look like the following:

`ec2-<MY-4-CHAR>-use1-1a-az4-platform`

Where `<MY-4-CHAR>` is the four letter name identifier found in your email address.

Once signed in the Yugabyte Platform Admin console, select the YugabyteDB universe that contains the running workload. In this demo, it is running the workloads, `SqlInserts`, `SqlSecondaryIndex`, and `SqlSnapshotTxns` from the repo `yb-sample-apps`. This is why there are four tables with reads and writes occurring the Universe as seen in the following image:

![The Universe is currently running several workloads.](./assets/images/100-universe_1920x830.png) 

Keeping track of the performance of this app is important in order to test the universe for performance degradation due to the rolling config change.

On the Universe details page, select the More dropdown located under the profile icon on the top right corner.
