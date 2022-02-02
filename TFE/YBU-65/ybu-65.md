# Storage Back up and Restore on Yugabyte Platform

## Introduction

In this hands-on lab, you will learn how to create a backup of a database then restore the database on Yugabyte Platform on AWS. The ability to backup and restore data in a straight forward and simple process is important to retain data in case of a failure, migration, or outage. Although we will backup and restore a PostgreSQL database, the procedure is nearly the same for a Cassandra database.

### Objective

As a sales engineer, I will demonstrate out to create a backup of the database and restore the database. 

## Prerequisites

A deployed Yugabyte Universe on AWS containing a populated PostgreSQL database. For information on how to deploy a Yugabyte Universe on Platform and run a workload, review the previous labs on Universe deployment for a multi-zone cluster.

The workload `SqlInserts` will be run on the Universe. This workload can be found on Platform as a docker container image, `yugabytedb/yb-sample-apps`.

The AWS credentials to access the S3 bucket; the access key ID and the secret access key for the IAM user account.

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
