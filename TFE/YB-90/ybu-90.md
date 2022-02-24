
⏰ Lab time is 105 min
---

## About this lab
In this lab, you will run a couple of the workloads in the the **Yahoo! Cloud Serving Benchmark (YCSB)** for Yugabyte. The lab consists of these activities and uses the YSQL API:
* Download and install the Yugabyte Client
* Download and install the Yugabyte YCBS benchmark
* Run individual workloads
* Experiment on your own
* Clean up


### About YCSB
The [Yahoo! Cloud Serving Benchmark (YCSB)](https://github.com/brianfrankcooper/YCSB/wiki) consists of the YCSB client and the Core workload. The YCSB client is an extensible workload generator which runs not only the Core workload but also any custom workloads. YCSB supports running benchmarks for many databases. 

The YCSB client is a Java program and ideally runs on at least one dedicated host. The number of CPU cores determines the upper thread count limits for both the loading and execution phases of the workload. For large database clusters, the benchmark often requires multiple YCSB clients running on individual dedicated hosts.

The Core workload defines a simple mix of read, insert, update, and scan operations. A parameter file and workload properties defines the relative frequency of each operation.  For the Core workload, the YCSB Client assumes that there is a default database table named `usertable`. By design, the `usertable` table is a flexible schema and allows for additional columns at runtime.

The YCSB client supports three commands:
- load
- run
- shell

The load and run commands require a binding properties reference to a specific database. The load command requires a workload file and optional parameters. The load command processes the loading section of the workload file. The run command is similar in signature to the load command parameters. However, the run command processes the transaction section of the workload file. The shell command serves as a shell for the defined database. Basic is the default database definition. As there is no database for Basic, you can use Basic to test that the YCSB client is available on a host.



### About the YCSB for Yugabyte
The [YCSB for Yugabyte](https://github.com/yugabyte/YCSB) is a fork of the original YCSB. The YCSB for Yugabyte supports running YSQL and YQCL workloads.

You can run the YCSB for Yugabyte in several ways. One way is to use the utility shell scripts, `run_jdbc.sh`, `run_ycql.sh`, and `run_ysql.sh`. The utility scripts load the data for and then run each workload in the the `workloads` directory. The scripts support the following arguments:

```
--ip                         IP of node to connect to
--recordcount                Number of rows
```


YCSB for Yugabyte contains the following bindings:

| Binding | Description |
|---------|-------------|
| YugabyteCQL | This binding contains the related JARs to enable YCSB to work with YugabyteDB using YCQL. |
| YugabyteSQL | This contains the related JARs to enable YCSB to work with YugabyteDB using YSQL.
| YugabyteSQL2Keys | This binding uses 2 column keys and supports `workloade`. The drivers support YSQL. |

 
YCSB for Yugabyte also contains a `db.properties` file. Modify this file to specify connection properties to your database for both YSQL and YCQL.



### Requirements
The utility scripts in YCSB for Yugabyte require the YSQL and YCQL shells. You must download these shells to your YCSB EC2 host. You must also add these shells to the your PATH variable. In order to complete this lab, you'll need the following:
* Your AWS Account
* A Yugabyte Universe managed by Yugabyte Platform
* A YCSB EC2 host running Centos 7 or similar in your VPC dedicated with the following installed:
  *  Java 1.8 (open JDK is fine)
  *  ysqlsh and/or ycqlsh



## Download and install the Yugabyte Client
The Yugabyte Client contains both the YCQL and YSQL Shells. 

* Using your `.pem` key, secure shell into your YCSB EC2 host, replacing the values for **PUBLIC_IPV4_HOST_YCSB** and **kp-NAME-aws.pem**.

```bash
PUBLIC_IPV4_HOST_YCSB = '<PUBLIC_IPV4_HOST_YCSB>'
MY_PEM_FILE = '<kp-NAME-aws.pem>'

ssh -i ~/.ssh/${MY_PEM_FILE} centos@${PUBLIC_IPV4_EC2_TPCC}

```

* Once connected, install `wget` and Java, if needed.
  
```bash
sudo yum install -y wget java
```

* Download Yugabyte Client.

```bash
cd /tmp
wget -q -O - https://downloads.yugabyte.com/get_clients.sh | sh 

```

* Add `ysqlsh` to your path.

```bash
export PATH=$PATH:/tmp/yugabyte-client-2.6/ysqlsh
```

* Optionally, add `ycqlsh` to your path.
  
```bash
export PATH=$PATH:/tmp/yugabyte-client-2.6/ycqlsh
```




## Download and install the Yugabyte YCBS benchmark
Follow these steps to download and install the benchmark on your EC2 host where you will run the YCSB benchmark.

* On your YCSB EC2 host, download and uncompress the `tpcc.targ.gz` file.
  
```bash
cd /tmp
wget https://github.com/yugabyte/YCSB/releases/download/1.0/ycsb.tar.gz
tar -zxvf ycsb.tar.gz
cd YCSB/
ls -l
```


### Configure the YCSB database properties file

Prior to loading data and running a benchmark, you need to configure the YCSB application to connect to your cluster. It is important to specify connection strings for all the nodes in the Yugabyte cluster. Here are the steps to follow for YSQL:

* Using VIM, edit the `db.properties` file in the YCSB directory `[i = Insert mode ]`.

```bash
vim db.properties
```  

* For the `db.url`, specify the connection string for each Yugabyte cluster node. Use the Private IPv4 address for each node. Use a semi-colon (`;`) delimiter for the connection strings.  
  
```bash
db.url=jdbc:postgresql://<PRIVATE_IPV4_YB_NODE_1>:5433/ycsb;jdbc:postgresql://<PRIVATE_IPV4_YB_NODE_2>:5433/ycsb;jdbc:postgresql://<PRIVATE_IPV4_YB_NODE_3>:5433/ycsb
```

* Change both the value of `db.user` and `db.passwd` to administrator user or similar user.
  
```bash
db.user=<MY_DATABASE_USER>
db.passwd=<MY_DATABASE_USER_PASSWORD>
```

* Save your changes `[ esc = Read-only mode  |  :wq! = force save quit ]` and exit.

* Verify your changes.
  
```bash
cat db.properties
```  


## Run individual workloads
Rather than running all the workloads in YCSB, you will run workloads individually. To run an individual workload, you will first create the `ycsb` database and the `usertable` table.

### Create the ycsb database and usertable table
Using the `ysqlsh` client, connect to a node in your Yugabyte cluster and create the `ycsb` database. Here are the steps:

* Connect to your database with `ysqlsh`, replacing the values for **MY_DATABASE_USER** and **PRIVATE_IPV4_YB_NODE_1**.

```bash
MY_DATABASE_USER = 'yugabyte'
PRIVATE_IPV4_YB_NODE_1 = '<PRIVATE_IPV4_YB_NODE_1>''
cd /tmp/yugabyte-client-2.6/bin/
./ysqlsh -h ${PRIVATE_IPV4_YB_NODE_1} -d yugabyte -U ${MY_DATABASE_USER}
```

* In YSQL Shell, create the **ycsb** database.

```sql
CREATE DATABASE ycsb;

```

* Next, connect to the database and create the **usertable** table.
  
```sql
\c ycsb

CREATE TABLE usertable (
  YCSB_KEY VARCHAR(255) PRIMARY KEY,
  FIELD0 TEXT, 
  FIELD1 TEXT, 
  FIELD2 TEXT, 
  FIELD3 TEXT,
  FIELD4 TEXT,
  FIELD5 TEXT,
  FIELD6 TEXT,
  FIELD7 TEXT,
  FIELD8 TEXT,
  FIELD9 TEXT);

exit;
```

### Load the data for a workload
After creating the database and the table, you need to load the data for the workload. The workload in this step is `workloada`. Here are the steps:

* Using the `load` command, load the data for `workloada`.

```bash
cd /tmp/YCSB/
./bin/ycsb load yugabyteSQL -s \
  -P db.properties  \
  -P workloads/workloada  \
  -p recordcount=10000  \
  -p operationcount=100000  \
  -p threadcount=32  \
  -p maxexecutiontime=180

```

* Verify the success of the load
```bash
[INSERT], Return=OK, 10000
```

* Next, you can run the workload.

```bash
cd /tmp/YCSB/
./bin/ycsb run yugabyteSQL -s \
  -P db.properties  \
  -P workloads/workloada  \
  -p recordcount=10000  \
  -p operationcount=100000  \
  -p threadcount=64  \
  -p maxexecutiontime=180

```

* Review the results of the workload benchmark.

```bash
[OVERALL], RunTime(ms), 15635
[OVERALL], Throughput(ops/sec), 1603.1232133351
```


* Run workloadb and review the benchmark results.
  
```bash
cd /tmp/YCSB/
./bin/ycsb run yugabyteSQL -s \
  -P db.properties  \
  -P workloads/workloadb  \
  -p recordcount=10000  \
  -p operationcount=100000  \
  -p threadcount=32  \
  -p maxexecutiontime=180

```


## Experiment on your own

Now that you have the YCSB benchmark up and running, you can experiment. Here a some ideas:

*  In Yugabyte Platform, for your universe with your Yugabyte cluster, view the dashboard and metrics while a workload runs.
*  Double the nodes in the cluster and run the previous benchmark. How does doubling the size of nodes affect the overall throughput metric?
*  Select a different workload in the workloads directory to run and review how this impacts the dashboard.


## Clean up
When you are done, you will want to complete the following tasks:
* Terminate your EC2 instance for running the benchmarks.
* Resize your cluster back to just three nodes with a replication factor of 3.
* Drop the `ycsb` database.
* If this is your last lab in this course, terminate all of your EC2 instances in AWS.


## Reflection
YCSB is a common benchmark that can show how various databases perform. YCSB for Yugabyte allows you to generate results for specific workloads. By doubling the number of nodes in a Yugabyte cluster, you can demonstrate linear scale.
