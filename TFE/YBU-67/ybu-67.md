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
## About this lab

In this hands-on lab, you will perform a rolling configuration change of a cluster on Yugabyte Platform. A rolling configuration change is one method of replacing current nodes for new ones that are required due to an update to the underlying infrastructure. You will accomplish this task by updating the G-flag on the Yugabyte Platform Admin console as well as adding a timeout flag. 

The reason why the underlying infrastructure will need updating is because software intrinsically gets stale over time. Whether its bug fixes, version upgrades, dependency changes, or feature additions, regular maintenance is a required task for highly resilient systems. 

"If you are not moving forward, you are moving backward." ~Gorbachev

A rolling change is a common method to implement a configuration change to a cluster of nodes since there is minimal change in reliably and performance of the database and therefore lacks any degradation in performance to the user. Of course, how this rolling configuration change occurs is a big factor in how efficient the upgrade is performed.

For context, here's a [list of deployment policies at AWS.](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.rolling-version-deploy.html)

### Objective

As a sales engineer, I want to perform a rolling configuration change to my Yugabyte Universe.

### Requirements

In this lab, you will need a Yugabyte Universe running on Platform. The Universe will need to be running several sample workloads in order to monitor changes in latency and performance when the rolling configuration takes place.


### Edit configuration flags

"Adding and modifying configuration flags for your YB-Master and YB-TServer nodes in a YugabyteDB universe allows you to resolve issues, improve performance, and customize functionality." ~ [Yugabyte Docs](https://docs.yugabyte.com/latest/yugabyte-platform/manage-deployments/edit-config-flags/)



