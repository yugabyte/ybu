# Storage Back up and Restore on Yugabyte Platform

## Introduction

In this hands-on lab, you will learn how to create a backup of a database then restore the database on Yugabyte Platform on AWS. The ability to backup and restore data in a straight forward and simple process is important to retain data in case of a failure, migration, or outage. Although we will backup and restore a PostgreSQL database, the procedure is nearly the same for a Cassandra database.

### Objective

As a sales engineer, I will demonstrate out to create a backup of the database and restore the database. 

## Prerequisites

A deployed Yugabyte Universe on AWS containing a populated PostgreSQL database. For information on how to deploy a Yugabyte Universe on Platform and run a workload, review the previous labs on Universe deployment for a multi-zone cluster.

The Yugabyte Universe in this lab has been deployed on a `c5.4xlarge` EC2 instance using a CentOS 7 image.

In this lab, we have run the workload `SqlInserts` on the Universe, but this lab will work on any YSQL table. This workload can be found on the Platform as a docker container image, `yugabytedb/yb-sample-apps`. Details on how to run a workload on a Yugabyte Universe can be found in a previous lab.

You need the AWS credentials to access the S3 bucket, namely the access key ID and the secret access key for the IAM user account.

You need the Yugabyte Platform credentials for the Yugabyte Universe that is installed on the EC2 instance.

You need the `.pem` key in order to access the EC2 instance that contains the Yugabyte Platform console.

## Checklist to create and restore a database

* Create an S3 bucket that will store the database backup.

* Enable the backup on the Yugabyte Universe.

* Drop the table.

* Verify the Yugabyte Universe no longer contains the table.

* Restore the database.

* Verify in the Yugabyte Universe that the database has been restored.

## Create the S3 bucket

In this step, we will create an S3 bucket with the proper permissions so the Yugabyte Universe can backup and restore from the S3 bucket.

Navigate to the S3 console in AWS and select the region that contains the Yugabyte Universe. In this case it will be `us-west-2`.

Name the bucket using the first four letters of your email address. For example, `mkim-ybu-storage-backup-us-west-2`.

> **Important:** Bucket names must be unique in a given region.

Next, use the default settings listed below and select the "Create Bucket" button.

| Property Name | Value |
|----|-----|
| AWS Region | us-west-2 (Oregon) |
| ACLs | Disabled |
| Block all public access | Enabled |
| Bucket Versioning | Disable |
| Default encryption | Disable |
| Object Lock| Disable |

### Bucket Permissions

Once the bucket has been created, you will be navigated to the Buckets list. 

Select the bucket that was just created and view the "Permissions" tab.

Review the following image to confirm that the user is able to write and read the objects that will be created in this bucket:

![ACL bucket permission for write and read access for the Yugabyte Universe.](./assets/images/100-s3_bucket_acl_1366x768.png)

Note that in the proceeding image, you have designated the bucket owner has list and write access to the objects in this S3 bucket. This permission is setup by default. This will allow the Yugabyte Universe to store and upload the database from the S3 bucket.

Keep a copy of the bucket name handy since you will need this when creating the backup in the Yugabyte Platform console.

## Create a Backup

In the last step you created an S3 bucket, that will store the database backup. In this step, you will create a database back up and store it in the S3 bucket.

### Verify that the Universe Contains a Table

Navigate to the Yugabyte Platform console on the EC2 instance public IPv4 on port 8800. Once you have signed in using your credentials, you will be navigated to the Universe Overview page.

Select the Universe that is running the workload, `SqlInserts`. For information on how to deploy a Universe and run a workload, review previous labs for details.

Once a Universe is selected, you will be navigated to the Universe details page that will present the "Overview" tab by default as shown in the following image:

![Description of this action.](./assets/images/150-universe_overview_1366x768.png)

Notice in the proceeding image, that in this Yugabyte Universe, there is a three node cluster that's running a single YSQL table.

Selecting the "Tables" tab, we can verify that the `postgresqlkeyvalue` table is located in the Yugabyte Universe as shown in the following image:

![Description of this action.](./assets/images/260-table_1366x768.png)

Note in the proceeding image, the table type is YSQL and the keyspace is `postgres`. Keyspace is another term used for database, where the table is located.

Now that we have verified that the table is currently available in the Yugabyte Universe, you will make a backup and store it in the S3 bucket you created earlier.

### Backup the Database

In the last step, you verified that a YSQL table was available in the Yugabyte Universe. In this step, you will back up the database to an S3 bucket.

On the Universes details page, select the "Backups" tab. This will navigate you to the following page:

![On the Backups tab, create a backup of the database.](./assets/images/300-create_backup_1600x700.png)

Select the "Create Backup" button in the proceeding image to navigate to a form as shown in the following image:

![Description of this action.](./assets/images/450-create_backup_form_1600x700.png)

Select the following properties:

| Property Name | Value | Default |
|----|----|----|
| Language | YSQL | Yes |
| Storage | S3 Storage | Yes |
| Namespace | postgres | No |
| Encrypt Backup | Disabled | Yes |
| Parallel Threads | 8 | Yes |
| Number of Days to Retain Backup | n/a | Yes |

Keep all the default values except the Namespace. This value is the name of the database that contains the table, `postgresqlkeyvalue`, in this case, `postgres`, as we noted in the previous step. There is also an option to backup multiple namespaces if needed. 

> **Important:** Several backups can be created that target particular namespaces. Backups can also be set on an automated schedule.

In a few moments, the backup of the database will complete. Refresh the page to update the status of the backup. Once the status is "Completed", you will proceed to the next steps; to drop the table and restore it from the backup.

To verify the backup was stored in the S3 bucket, you can run the view the S3 console to view the backup in the S3 bucket:

![The backup folder in the S3 bucket has been newly created.](./assets/images/800-universe_backup_1366x768.png)

Now that the backup has been verified, the next step is to drop the table. To do this, you need to connect to the database with the YSQL shell. Since the table is distributed on the nodes, you will need the connect to a node in the cluster first.

To get the SSH command to connect to a node, navigate to the "Nodes" tab on the Universes details page and select the "Actions" drop down located in the Primary Cluster window as shown in the following image:

![Select the dropdown for a node in the cluster.](./assets/images/460-node_connect_1600x700.png)

Select "Connect" and copy the command to SSH into this node.

```bash
sudo ssh -i /opt/yugabyte/yugaware/data/keys/be5f73e4-3bfe-4b55-8c1c-0aa144be10d6/yb-dev-aws_be5f73e4-3bfe-4b55-8c1c-0aa144be10d6-key.pem -ostricthostkeychecking=no -p 22 yugabyte@<node-IP>
```

As can be seen the the proceeding statement, the `.pem` key for the node instance is located on the EC2 instance that contains Yugabyte Platform. This value was automatically generated when the Universe was created. You will be accessing the node through port 22 with the user name `yugabyte`.

Make a note of this command since you will use it to SSH into the node.

## Drop the Table

In the last step you created a backup of the database, `postgres` that contains the table, `postgresqlkeyvalue`. In this step, you will drop the table, then restore the table from the backup in the S3 bucket.

In this step, you will need the `.pem` file to SSH into the EC2 instance.

In order to drop the table, you will SSH into the EC2 instance that contains the Yugabyte Platform console, connect to a node, then connect to the YSQL shell. 

### Connect to the Node

In the CLI, navigate to the directory that contains the `.pem` file and execute the following command:

```bash
ssh -i "ybu-yugaware.pem" centos@<my-EC2-public-IPv4>
```

Once connected to the EC2 instance, paste the SSH command you noted earlier to connect to the node.

The connection is established if the prompt in the CLI reflects the user, `yugabyte`, and the IP address of the node. Here is an example of what the connection will look like:

```bash
[yugabyte@ip-172-151-37-55 ~]$
```

### Connect to the Database with the YSQL Shell

In the last step, you connected to the node. In this step we will open the YSQL shell in order to connect to the database and drop the table.

Run the following command to connect to the YSQL shell:

```bash
/home/yugabyte/tserver/bin/ysqlsh -h <my-node-IP> -p 5433 -U yugabyte
```

In the proceeding command, you ran the binary file `ysqlsh` with the declarations for the host, port, and user.

> **Deep Dive:** [For a closer look at the `ysqlsh` and the shell commands, visit the official Yugabyte docs for more detailed instructions.](https://docs.yugabyte.com/latest/admin/ysqlsh/)

You have successfully connected to the YSQL shell if the prompt now looks like this:

```bash
yugabyte=#
```

Now you can directly connect to the table, view logs, and use PostgreSQL DDL or DML to make changes to the data layer.

First, run the following command to list the databases:

```bash
\l
```

This will result in the following response in the CLI:

```bash
                                   List of databases
      Name       |  Owner   | Encoding | Collate |    Ctype    |   Access privileges
-----------------+----------+----------+---------+-------------+-----------------------
 postgres        | postgres | UTF8     | C       | en_US.UTF-8 |
 system_platform | postgres | UTF8     | C       | en_US.UTF-8 |
 template0       | postgres | UTF8     | C       | en_US.UTF-8 | =c/postgres          +
                 |          |          |         |             | postgres=CTc/postgres
 template1       | postgres | UTF8     | C       | en_US.UTF-8 | =c/postgres          +
                 |          |          |         |             | postgres=CTc/postgres
 yugabyte        | postgres | UTF8     | C       | en_US.UTF-8 |
(5 rows)

yugabyte=#
```

Connect to the `postgres` database with the following command:

```bash
\c postgres
```

The proceeding command will return the following result:

```bash
yugabyte=# \c postgres
You are now connected to database "postgres" as user "yugabyte".
postgres=#
```

> **Pro Tip:** As a general practice, it is good to turn on the `\timing` feature to measure response time of the SQL operations.

To list the tables, run the following command in the ysqlsh:

```bash
\dt
```

The will result in the following response in the CLI.

```bash
               List of relations
 Schema |        Name        | Type  |  Owner
--------+--------------------+-------+----------
 public | postgresqlkeyvalue | table | postgres
(1 row)

postgres=#
```

Determine the number of rows in this table with the following command:

```SQL
SELECT count(*) from postgresqlkeyvalue;
```

This will create the following response in the CLI:

```bash
---------
 2000001
(1 row)

Time: 2850.394 ms (00:02.850)
postgres=#
```

Note there are 2,000,001 rows in this table.

Now that you have established the table is present and populated with data, you will drop this table with the following command:

```bash
DROP TABLE postgresqlkeyvalue;
```

This command will result in the following response in the CLI:

```bash
DROP TABLE
Time: 199.763 ms
postgres=#
```

### Verify the Table has been Dropped

Now run the `\dt` or `SELECT count(*) from postgresqlkeyvalue;` commands to verify the table is no longer available. The CLI will result in the following response:

```bash
Did not find any relations
```

Leave this YSQL shell open so you can verify the table has been restored.

Navigate back to the Yugabyte Platform console in the browser to verify that the `postgresqlkeyvalue` table is no longer available. Select the "Tables" tab in the Universe Details page to see the following image:

![Description of this action.](./assets/images/500-drop_table_1366x768.png)

Note in the proceeding image that there are no longer any available tables in this Universe.

## Restore the Table

In the last step, you dropped the table then verified the table is no longer present in the Universe. In this step, you will restore the table and verify the data remains consistent with the data prior to the DROP command.

On the Yugabyte Platform console, in the Universe details page, select the "Backups" tab. This will display the following image: 

![Select the backup to restore the table.](./assets/images/700-restore_backup_1600x700.png)

In the proceeding image, you will see the backup you created in a previous step. Select the "Actions" button for this backup and choose the "Restore Backup" option as shown in the following image:

![Restore this backup to the current universe.](./assets/images/750-restore_data_1600x700.png)

In the proceeding form, select the default options for the current Universe and the keyspace or database, `postgres`, keeping the same number of parallel threads. 

> **Pro Tip:** Increasing the parallel thread capacity will result in a faster backup or backup restore. This decision depends on the size of the backup as well as the resource capacity of the EC2 instances.

Once the backup restore has completed, check the "Tables" tab in the Universes details page to see if the table has been restored.

### Verify the Table has been Restored

In the last step, the table was restored. In this step you will verify the data was restored with no data loss.

Navigate back to the CLI that is connected to the database via the YSQL shell. Run the following command:

```bash
SELECT count(*) from postgresqlkeyvalue;
```

You will see the following response in the CLI:

```bash
---------
 2000001
(1 row)
```

From the proceeding response from the SQL query, we have verified that the table has been restored and there has been no data loss from the backup and restore process.

## Reflection

In this lab, you created a backup of the YSQL database and restored it. You also connected to a node to access the database in order to remove the table.