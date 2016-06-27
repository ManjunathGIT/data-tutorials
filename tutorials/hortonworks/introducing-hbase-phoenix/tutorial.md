---
layout: tutorial
title: Introduction to HBase Concepts, Apache Phoenix and New Backup & Restore Utility in HBase
tutorial-id: 650
tutorial-series: Hello World
tutorial-version: hdp-2.5.0
intro-page: true
components: [ hbase, phoenix, ambari ]
---

## Introduction

HBase is a distributed column-oriented database built on top of the Hadoop file system. It is an open-source project and is horizontally scalable. HBase is a data model that is similar to Google’s big table designed to provide quick random access to huge amounts of unstructured data. It leverages the fault tolerance provided by the Hadoop File System (HDFS).

It is a part of the Hadoop ecosystem that provides random real-time read/write access to data in the Hadoop File System.

Tables in HBase are sorted by row. The table schema defines only column families, which are the key value pairs. A table have multiple column families and each column family can have any number of columns. Subsequent column values are stored contiguously on the disk. Each cell value of the table has a timestamp.
For more information, refer HBase documentation [here](http://hortonworks.com/apache/hbase/).

In this tutorial, we are going to walk you through some basic HBase shell commands, how to use Apache Phoenix along with HBase and a new Backup & Restore Utility introduced in HDP 2.5.

## Prerequisites

- [Download Hortonworks 2.5 Sandbox Technical Preview](http://hortonworks.com/tech-preview-hdp-2-5/)
- Complete the [Learning the Ropes of the Hortonworks Sandbox tutorial,](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/) you will need it for logging into Ambari as an administrator user.

## Outline

- [1: Start HBase](#start-hbase)
    - [1.1: View the HBase Services page](#view-hbase-service)
    - [1.2: Start HBase Service](#start-hbase-service)
- [2: Enter HBase Shell](#enter-hbase-shell)
- [3: Try out Some General Commands](#general-commands)
    - [3.1: version](#version)
    - [3.2: whoami](#whoami)
- [4: Data Definition Language Commands in HBase](#ddl-hbase)
    - [4.1: Create](#create)
    - [4.2: List](#list)
    - [4.3: Describe](#describe)
- [5: Data Manipulation Commands in HBase](#dml-hbase)
    - [5.1: Scan](#scan)
    - [5.2: Put](#put)
    - [5.3: Get](#get)
- [6: Apache Phoenix Introduction](#phoenix-introduction)
- [7: Enabling Phoenix by Ambari](#enable-phoenix)
- [8: Launch Phoenix Shell](#launch-phoenix-shell)
- [9: Create Phoenix Table on existing HBase table](#create-phoenix-table)
- [10: Inserting Data via Phoenix](#inserting-data)
- [11: HBase Backup & Restore Introduction](#hbase-backup-restore-introduction)
- [12: Creating a Full Backup](#create-full-backup)
- [13: Backup Sets](#backup-sets)
- [14: Restoring a Backup](#restore-backup)

## 1. Start HBase <a id="start-hbase"></a>

### 1.1 View the HBase Services page <a id="view-hbase-service"></a>

Started by logging into Ambari as an admin user. From the Dashboard page of Ambari, click on `HBase` from the list of installed services.

![hbaseServiceOnOff](/assets/introducing-hbase-phoenix/hbase_service_on_off_iot.png)

### 1.2 Start HBase Service <a id="start-hbase-service"></a>

From the HBase page, click on Service Actions -> `Start`

![starthbaseService](/assets/introducing-hbase-phoenix/start_hbase_service_iot.png)

Check the box and click on Confirm Start:

![confirmhbaseStartIot](/assets/introducing-hbase-phoenix/confirm_hbase_start_iot.png)

Wait for HBase to start (It may take a few minutes to turn green)

![hbaseStartedIot](/assets/introducing-hbase-phoenix/hbase_started_iot.png)

## 2. Enter HBase Shell <a id="enter-hbase_shell"></a>

HBase comes with an interactive shell from where you can communicate with HBase components and perform operations on them.

First SSH into the Hortonworks Sandbox with the command:

~~~
ssh root@127.0.0.1 -p 2222
~~~

![sshTerminal](/assets/introducing-hbase-phoenix/sshTerminal.png)

Switch the user to hbase.

~~~
su hbase
~~~

![switchTohbase](/assets/introducing-hbase-phoenix/switch_to_hbase.png)

Type `hbase shell` and you will see the following screen:

![enterhbaseShell](/assets/introducing-hbase-phoenix/enter_HBase_shell.png)

To exit the interactive shell, type `exit` or use `<ctrl+c>`. But wait, it is time to explore more features of the shell.

## 3. Try out Some General Commands <a id="general-commands"></a>

As you are now inside **HBase shell**, let’s try out some general commands:

### 3.1 version <a id="version"></a>

Type `version` to know the version of HBase being used.

![versionHBase](/assets/introducing-hbase-phoenix/version_HBase.png)

### 3.2 whoami <a id="whoami"></a>

Type `whoami`  to know the information about the user.

![whoamiHBase](/assets/introducing-hbase-phoenix/whoami_HBase.png)

## 4. Data Definition Language Commands in HBase <a id="ddl-hbase"></a>

These are the commands that operate on tables in HBase.

### 4.1 Create <a id="create"></a>

The syntax to create a table in HBase is `create '<table_name>','<column_family_name>'`. Let’s create a table called '**driver_dangerous_event'** with a column family as **events**. Type:

~~~
create 'driver_dangerous_event','events'
~~~

![create_table](/assets/introducing-hbase-phoenix/create_table.png)

### 4.2 List <a id="list"></a>

Type `list` to check whether the table is created or not. It will display the list of all the user tables in HBase as shown below:

![list_table](/assets/introducing-hbase-phoenix/list_table.png)

### 4.3 Describe <a id="describe"></a>

Type `describe 'driver_dangerous_event'`  to see the description of the table.

![describe_table](/assets/introducing-hbase-phoenix/describe_table.png)

## 5. Data Manipulation Commands in HBase <a id="dml-hbase"></a>

Let’s import some data into the table which you created. The file data.csv is the collected data from the events. It contains records like driverId, truckId, eventTime, eventType, location, etc.

Open a new terminal and ssh into the Sandbox. Download the data.csv file and let’s copy the file in HDFS,

~~~
ssh root@127.0.0.1 -p 2222

curl -o data.csv https://raw.githubusercontent.com/hortonworks/tutorials/hdp/assets/data.csv

hadoop fs -copyFromLocal data.csv /tmp
~~~

![copyFromLocal_data_csv](/assets/introducing-hbase-phoenix/copyFromLocal_data_csv.png)

Now execute the `LoadTsv` from hbase user statement as following:

~~~
su hbase

hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=,  -Dimporttsv.columns="HBASE_ROW_KEY,events:driverId,events:driverName,events:eventTime,events:eventType,events:latitudeColumn,events:longitudeColumn,events:routeId,events:routeName,events:truckId" driver_dangerous_event hdfs://sandbox.hortonworks.com:/tmp/data.csv
~~~

![importtsv_command](/assets/introducing-hbase-phoenix/importtsv_command.png)

Wait for 20 seconds, you will see something like this:

![importtsv_output](/assets/introducing-hbase-phoenix/importtsv_output.png)

Now let’s check whether the data got imported in the table `driver_dangerous_events` or not. Go back to the `hbase shell`.

### 5.1 Scan <a id="scan"></a>

The scan command is used to view the data in the HBase table. Type `scan 'driver_dangerous_event'`.
You will see all the data present in the table with row keys and the different values for different columns in a column family.

![scan_command](/assets/introducing-hbase-phoenix/scan_command.png)

### 5.2 Put <a id="put"></a>

Using put command, you can insert rows in a HBase table. The syntax of put command is as follows:
`put '<table_name>','row1','<column_family:column_name>','value'`

Copy following lines to put the data in the table.

~~~
put 'driver_dangerous_event','4','events:driverId','78'
put 'driver_dangerous_event','4','events:driverName','Carl'
put 'driver_dangerous_event','4','events:eventTime','2016-09-23 03:25:03.567'
put 'driver_dangerous_event','4','events:eventType','Normal'
put 'driver_dangerous_event','4','events:latitudeColumn','37.484938'
put 'driver_dangerous_event','4','events:longitudeColumn','-119.966284'
put 'driver_dangerous_event','4','events:routeId','845'
put 'driver_dangerous_event','4','events:routeName','Santa Clara to San Diego'
put 'driver_dangerous_event','4','events:truckId','637'
~~~

![put_command](/assets/introducing-hbase-phoenix/put_command.png)

Now let’s view a data from scan command.

~~~
scan 'driver_dangerous_event'
~~~

![scan_command1](/assets/introducing-hbase-phoenix/scan_command1.png)

You can also update an existing cell value using the `put` command. The syntax for replacing is same as inserting a new value. Just make sure, you select a correct row, column family and column to update.

So let’s update a route name value of row key 4, from `'Santa Clara to San Diego'` to `'Santa Clara to Los Angeles'`. Type the following command in HBase shell:

~~~
put 'driver_dangerous_event','4','events:routeName','Santa Clara to Los Angeles'
~~~

![update_table](/assets/introducing-hbase-phoenix/update_table.png)

Now scan the table to see the updated data:

~~~
scan 'driver_dangerous_event'
~~~

![scan_command2](/assets/introducing-hbase-phoenix/scan_command2.png)

### 5.3 Get <a id="get"></a>

It is used to read the data from HBase table. It gives a single row of data at a time. Syntax for get command is:
`get '<table_name>','<row_number>'`

Try typing `get 'driver_dangerous_event','1'` in the shell. You will see the all the column families (in our case, there is only 1 column family) along with all the columns in the row.

![get_command](/assets/introducing-hbase-phoenix/get_command.png)

You can also read a specific column from get command. The syntax is as follows:
`get 'table_name', 'row_number', {COLUMN ⇒ 'column_family:column-name '}`

Type the following statement to get the details from the row 1 and the driverName of column family events.

~~~
get 'driver_dangerous_event','1',{COLUMN => 'events:driverName'}
~~~

![get_command_column](/assets/introducing-hbase-phoenix/get_command_column.png)

## 6. Apache Phoenix Introduction <a id="phoenix-introduction"></a>

Apache Phoenix is a SQL abstraction layer for interacting with HBase. Phoenix lets you
create and interact with tables in the form of typical DDL/DML statements via its standard JDBC API. With the driver APIs, Phoenix translates SQL to native HBase API calls. Consequently, Phoenix provides a SQL skin for working with data and objects stored in the NoSQL schema of HBase.

For more information about Phoenix capabilities, see the [Apache Phoenix website](#https://phoenix.apache.org/).

## 7. Enabling Phoenix <a id="enable-phoenix"></a>

There is no separate installation required for Phoenix. You can enable Phoenix with Ambari:

1. Go to Ambari and select Services tab > HBase > Configs tab.

![config_HBase](/assets/introducing-hbase-phoenix/config_HBase.png)

2. Scroll down to the Phoenix SQL settings.

![phoenix_tab](/assets/introducing-hbase-phoenix/phoenix_tab.png)

3. Click the `Enable Phoenix` slider button.

![enable_phoenix](/assets/introducing-hbase-phoenix/enable_phoenix.png)

4. Scroll up and click `Save` to get your config change reflected.

![save_phoenix_setting](/assets/introducing-hbase-phoenix/save_phoenix_setting.png)

A pop up will come to write about the change that you are making. Type **Enabled Phoenix** in it and click `Save`.

![save_phoenix_setting_popup](/assets/introducing-hbase-phoenix/save_phoenix_setting_popup.png)

After pressing `Save`, you will get a popup like this:

![save_configuration_changes](/assets/introducing-hbase-phoenix/save_configuration_changes.png)

5. Restart HBase service.

![restart_HBase_service](/assets/introducing-hbase-phoenix/restart_HBase_service.png)

## 8. Launch Phoenix Shell <a id="launch-phoenix-shell"></a>

To connect to Phoenix, you need to specify the zookeeper quorum and in the sandbox, it is localhost. Change the working directory as per your HDP version. To launch it, execute the following commands:

~~~
cd /usr/hdp/2.5.0.0-817/phoenix/bin

./sqlline.py localhost
~~~

Your Phoenix shell will look like this:

![enter_phoenix_shell](/assets/introducing-hbase-phoenix/enter_phoenix_shell.png)

## 9. Create Phoenix Table on existing HBase table <a id="create-phoenix-table"></a>

Let’s create an HBase table from Phoenix. You can create a Phoenix table/view on a pre-existing HBase table. There is no need to move the data to Phoenix or convert it. A table can be created with the same name.

> **NOTE**: The DDL used to create the table is case sensitive and if HBase table name is in lowercase, you have to put the name in between double quotes. In HBase, you don’t model the possible KeyValues or the structure of the row key. This is the information you specify in Phoenix and beyond the table and column family.

Create a Phoenix table from existing HBase table by writing a code like this:

~~~
create table "driver_dangerous_event" ("row" VARCHAR primary key,"events"."driverId" VARCHAR,"events"."driverName" VARCHAR,
"events"."eventTime" VARCHAR,"events"."eventType" VARCHAR,"events"."latitudeColumn" VARCHAR,
"events"."longitudeColumn" VARCHAR,"events"."routeId" VARCHAR,"events"."routeName" VARCHAR,
"events"."truckId" VARCHAR);
~~~

![create_phoenix_table](/assets/introducing-hbase-phoenix/create_table_phoenix.png)

You can view the HBase table data from this Phoenix table.

~~~
select * from "driver_dangerous_event";
~~~

![select_data_phoenix](/assets/introducing-hbase-phoenix/select_data_phoenix.png)

If you want to change the view from horizontal to vertical, type the following command in the shell and then try to view the data again:

~~~
!outputformat vertical

select * from "driver_dangerous_event";
~~~

![select_data_phoenix1](/assets/introducing-hbase-phoenix/select_data_phoenix1.png)

So with all existing HBase tables, you can query them with SQL now. You can point your Business Intelligence tools and Reporting Tools and other tools which work with SQL and query HBase as if it was another SQL database with the help of Phoenix.

## 10: Inserting Data via Phoenix <a id="inserting-data"></a>

You can insert the data using `UPSERT` command. It inserts if not present and updates otherwise the value in the table. The list of columns is optional and if not present, the values will map to the column in the order they are declared in the schema. Copy the `UPSERT` statement given below and then view the newly added row.

~~~
UPSERT INTO "driver_dangerous_event" values('5','23','Matt','2016-02-29 12:35:21.739','Abnormal','23.385908','-101.384927','249','San Tomas to San Mateo','814');

select * from "driver_dangerous_event";
~~~

![upsert_data_phoenix](/assets/introducing-hbase-phoenix/upsert_data_phoenix.png)

You will see a newly added row:

![upsert_data_phoenix1](/assets/introducing-hbase-phoenix/upsert_data_phoenix1.png)

## 11. HBase Backup & Restore Introduction <a id="hbase-backup-restore-introduction"></a>

The HBase backup-and-restore utility helps ensure that enterprises using HBase as a data repository can recover from the data loss or failures. HBase incremental backups enable more efficient and effective capture of HBase table images than previous attempts at serial backup-and-restore solutions, such as those that only used HBase Export and Import APIs. The full backup must be exported. The backup utility has an option to restore the data of the baseline full backup, which serves as the foundation on which incremental backups can be applied to return to you the data of a backup that was performed at a later point in time.

## 12. Creating a Full Backup <a id="create-full-backup"></a>

The first step in running the backup-and-restore utilities is to capture the complete data set in a separate image from the source. The syntax for creating HBase backup is as follows:

`hbase backup create {{ full | incremental } {backup_root_path} {[tables] | [-set backup_set_name]}} [[-silent] | [-w number_of_workers] | [-b bandwidth_per_worker]]`

### Arguments:-

1. full | incremental - Full argument takes the full backup image. Incremental argument creates an incremental backup that has an image of data changes since the full backup or the previous incremental backup.

2. backup_root_path - the root path where you want to store your backup image.

3. tables (optional) - Specify the table or tables to backup. If no tables are specified, all tables are backed up.

4. -set backup_set_name (optional)- Calls an existing backup set in the command.

5. -silent (optional) - Ensures that the progress of the backup is not displayed on the screen.

6. -w number_of_workers (optional) - Specifies the number of parallel workers to copy data of the backup.

7. -b bandwidth_per_worker (optional) - Specifies the bandwidth of the worker in MB per second.

Now create a full backup of table `driver_dangerous_event` with 3 parallel workers.

~~~
hbase backup create full hdfs://sandbox.hortonworks.com:8020/user/hbase/backup driver_dangerous_event -w 3  
~~~

![create_full_backup_image](/assets/introducing-hbase-phoenix/create_full_backup.png)

You check whether the backup of your table is created in HDFS or not.

~~~
hadoop fs -ls /user/hbase/backup
~~~

![view_full_backup](/assets/introducing-hbase-phoenix/view_full_backup.png)

One more way to check whether the backup is taken or not is by running:

~~~
hbase backup history
~~~

![backup_history](/assets/introducing-hbase-phoenix/backup_history.png)

Note the backup_ID which will be used while restoring the data.

## 13. Backup Sets <a id="backup-sets"></a>

You can create a group of tables into a set so that it reduces the amount of repetitive inputs of table names. You can then use the `-set argument` to invoke named backup set in either hbase backup create or hbase backup restore utility. Syntax to create a backup set is:

`hbase backup set {[add] | [remove] | [list] | [describe] | [delete]} backup_set_name tables`

If you run the **hbase backup set add** command and specify a backup set name that does not yet exist on your system, a new set is created. If you run the command with the name of an existing backup set name, then the tables that you specify are added to the set.

### Arguments:

1. add  - Add tables to a backup set. Specify a backup_set_name value after this argument to create a backup set.
remove - Removes tables from the set. Specify the tables to remove in the tables argument.

2. list - Lists all backup sets.

3. describe - Use this subcommand to display on the screen a description of a backup set. This subcommand must precede a valid value for the backup_set_name value.

4. delete - Deletes a backup set. Enter the value for the backup_set_name option directly after the **hbase backup set delete** command.

5. backup_set_name (optional) - Used to assign or invoke a set name.

6. tables (optional) - list of tables to include in the backup set.

Now create a backup set called event which has a table driver_dangerous_event.

~~~
hbase backup set add event driver_dangerous_event
~~~

![create_backup_set](/assets/introducing-hbase-phoenix/create_backup_set.png)

Let’s check whether our set is added or not using list:

~~~
hbase backup set list
~~~

![view_backup_set](/assets/introducing-hbase-phoenix/view_backup_set.png)

## 14. Restoring a Backup <a id="restore-backup"></a>

The syntax for running a restore utility is as follows:

`hbase restore {[backup_root_path] | [backup_ID] | [tables]} [[table_mapping] | [-overwrite]]`

### Arguments:

1. backup_root_path - Specifies the parent location of the stored backup image.

2. backup_ID - The backup ID that uniquely identifies the backup image to be restored.

3. tables - Table or tables to be restored.

4. table_mapping (optional)- Directs the utility to restore data in the tables that are specified in the tables option. Each table must be mapped prior to running the command.

5. -overwrite  (optional) - Overwrites an existing table if there is one with the same name in the target restore location.

6. -automatic (optional) - Restores both the backup image and all the dependencies following the correct order.

Let’s drop a table so that restore utility can be tested. To drop a table in HBase, you first have to disable it and then drop it.

Run the following commands to drop the table:

~~~
disable 'driver_dangerous_event'

drop 'driver_dangerous_event'
~~~

![drop_table](/assets/introducing-hbase-phoenix/drop_table.png)

Now let’s restore the backup of this table which you created earlier in the tutorial:

> **NOTE**: Copy the same backup ID that you got while doing **hbase backup history**

~~~
hbase restore /user/hbase/backup backup_1466560117119 driver_dangerous_event -automatic
~~~

![restore_command_result](/assets/introducing-hbase-phoenix/restore_command_resultrev.png)

You can view the result at the end of this command’s execution.

![restore_command](/assets/introducing-hbase-phoenix/restore_command.png)