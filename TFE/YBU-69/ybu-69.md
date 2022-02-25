## Styles
<div class="fr-view">
	<style>
		td,
		th,
		tr {
			padding: 8px;
		}
		
		blockquote {
			margin: 10px 0;
			padding: 15px 0;
			border: none !important;
		}
		
		blockquote p {
			box-shadow: 2px 2px 10px grey;
			padding: 15px;
			border-left: 2px solid rgb(57, 11, 192)
		}
		
		img {
			border: 1px solid black;
		}
		
		.code-highlight {
			font-family: "Courier New", monospace;
			font-size: 16px;
			padding-left: 4px;
			color: rgb(217, 0, 0);
		}
		
		.bash {
			background: black;
			color: white;
			font-weight: bold;
			font-family: "Courier New", monospace;
			width: 80%;
			padding: 10px;
		}

	</style>

  ⏰ Lab time is 20 min | [slack feedback](https://yugabyte.slack.com/archives/C03176Y6BU0)
***

## About this lab

In this hands-on lab, you will run a YSQL workload on a multi-node Yugabyte Universe with YB Sample Apps, a docker image that you can pull to run pre-canned workloads.

## About YB Sample Apps
The YB Sample App is a  Java program. Yugabyte Platform contains a docker image with this Java application. If you connect to your Yugabyte Platform host using SSH,  you can run a container of the image. The YB Sample App mimics the workload of an application. The SqlInserts workload is one of many different sample workloads. This workload inserts unique string keys into a postgresqlkeyvalue table. YB Sample Apps creates the table and the INSERT statements.

There are a total of 21 sample workloads that you can run from the docker image. For a full description, visit the yb-sample-apps GitHub repository. The docker image supports additional Yugabyte Query Layer APIs such as YCQL.

## Objective
As a sales engineer, I want to run a sample workload for a given Yugabyte Cluster so that the cluster's related performance metrics can be reviewed.

## Requirements
Here are the requirements for this lab:

A deployed YugabyteDB cluster (a Universe)
Yugabyte Platform credentials
A .pem  file that you can use to connect to the EC2 instance of your YugabyteDB Platform host

## Verify the Universe is operational
Here are the steps to verify that the Universe is operational:

Navigate to the public IP of the EC2 instance that hosts Yugabyte Platform.
Sign in using your credentials.
View your universes in your Dashboard.

In the Dashboard, select your universe
View the Universe details that will be running the workload. 
Verify that the universe is ready and that the Primary Cluster has 3 nodes.

Copy the CLI command
Here are the steps to copy the CLI command:

In the Tab bar, select Actions. 
In the Actions menu, select Run Sample Apps.
In the Run Sample Apps dialog, select the default YSQL tab.
In Usage, select Copy.

Paste the copied CLI docker command into a text file so that you can use it later.

Run a YSQL workload
With an operational universe and the YB Sample Apps docker image, you can now run the docker container. Here are the steps to connect to your Yugabyte Platform host and run the container:

Use SSH to connect to your EC2 instance.
Important: In order to connect to the Platform server, you will need the .pem key that was downloaded when the EC2 instance was launched.

Execute your copied docker run command or copy the following command, replacing the <PrivateIPv4_NODE_X> tokens.
sudo docker run -d yugabytedb/yb-sample-apps --workload SqlInserts --nodes <PrivateIPv4_NODE_1>:5433,<PrivateIPv4_NODE_2>:5433,<PrivateIPv4_NODE_3>:5433
Important: In order to run the proceeding workload script on a Universe that has a password authenticated YSQL database or TLS encryption in transit, it is necessary to add the user, password, and path of the locally stored .crt and .key files. You many need to secure copy these files to your EC2 host and then copy these files into your docker container. By default the user is yugabyte. For more details, review  TLS encryption in transit.

Verify the running workload
Here are the steps to verify in Yugabyte Platform that the workload is running: 

In the left sidebar, select Universes.
Select your Universe.
In the Overview tab, view the CPU Usage gauge, the Total Ops / Sec line chart, and the Average Latency line chart.

In the Universe Details, select the Tables tab and verify the postgresqlkeyvalue table.
Select the Metrics tabs to review the workload performance. The following displays the Nodes metrics.

Not only is the CPU usage being measured, but also how much memory, latency, system load, and ops/second for reads and writes are also being monitored.

## Stop the Workload
Here are the steps to stop the workload from running:

In the SSH of your Yugabyte Platform host, stop the workload using  docker stop:
sudo docker rm $(docker stop $(docker ps -a -q --filter ancestor=yugabytedb/yb-sample-apps --format="{{.ID}}"))
Exit the SSH of your Yugabyte Platform host.


## Reflection
The purpose of this lab was to illustrate how a YSQL workload for a sample application performs on a Yugabyte cluster. As a learning outcome, you now know how to not only perform a basic benchmark of cluster performance, but also use YB Sample Apps to quickly demonstrate cluster resiliency and high availability.





That's it.
To continue, in the footer bar of this Course Player, please select  NEXT → .


Important: Universe termination
If you do not plan on continuing with this course, you need to delete and terminate your universe. Deleting a universe will remove the underlying infrastructure in AWS. Pausing a universe however will not. Here's how to delete your universe:

Go to the Universe details page.
Select Actions.
Select Delete Universe from the drop down list.
Enter the name of the universe.
Check the Ignore Errors and Force Delete box.
Check the Delete Backups box.
Select Yes.
</div>