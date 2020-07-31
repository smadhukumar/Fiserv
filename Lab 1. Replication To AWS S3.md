# Lab 1 -  Oracle to CloudERA Hadoop

## Before You Begin
-Hadoop 3.0.0-cdh6.3.2
-GoldenGate 19c for BigData
-Hadoop client libraries at ***/opt/cloudera/parcels/CDH/lib/hadoop/client/* ***


### Introduction
 This lab will guide through the steps required to start a replication to CloudERA Hadoop.

### Objectives
- Replicate from trail files on the target machine to CloudERA Hadoop.

### Time to Complete
Approximately 30 minutes


### What Do You Need?
You will need:
- Putty if you are using windows machine


1. Kindly create hdfs props and prm files under dirprm goldengate home directory

```
[oracle@hol ~]$cd <GOLDENGATE_FOR_BIGDATA_HOME>/dirprm
[oracle@hol dirprm]$ vi hdfs.props


```
Copy paste the below text in s3.props.


## CloudERA Hadoop properties for GG4BD

```
gg.handlerlist=hdfs

gg.handler.hdfs.type=hdfs
gg.handler.hdfs.includeTokens=false
gg.handler.hdfs.maxFileSize=1g
gg.handler.hdfs.pathMappingTemplate=/ogg1
gg.handler.hdfs.fileRollInterval=0
gg.handler.hdfs.inactivityRollInterval=0
gg.handler.hdfs.fileNameMappingTemplate=${fullyQualifiedTableName}_${groupName}_${currentTimestamp}.txt
gg.handler.hdfs.partitionByTable=true
gg.handler.hdfs.rollOnMetadataChange=true
gg.handler.hdfs.authType=none
gg.handler.hdfs.format=delimitedtext
gg.handler.hdfs.format.includeColumnNames=true

gg.handler.hdfs.mode=tx

goldengate.userexit.writers=javawriter
javawriter.stats.display=TRUE
javawriter.stats.full=TRUE

gg.log=log4j
gg.log.level=INFO

gg.report.time=30sec

#Sample gg.classpath for Apache Hadoop
#gg.classpath=/var/lib/hadoop/share/hadoop/common/*:/var/lib/hadoop/share/hadoop/common/lib/*:/var/lib/hadoop/share/hadoop/hdfs/*:/var/lib/hadoop/share/hadoop/hdfs/lib/*:/var/lib/hadoop/etc/hadoop/:
#Sample gg.classpath for CDH
gg.classpath=/opt/cloudera/parcels/CDH/lib/hadoop/client/*:/etc/hadoop/conf
#Sample gg.classpath for HDP
#gg.classpath=/usr/hdp/current/hadoop-client/client/*:/etc/hadoop/conf

javawriter.bootoptions=-Xmx512m -Xms32m -Djava.class.path=ggjava/ggjava.jar
```
Save the text using wq!

2. Edit the Replicat Parameter file to include the schema_name & table_name that needs to be replicated.
```
REPLICAT S3
-- Trail file for this example is located in "AdapterExamples/trail" directory
-- Command to add REPLICAT
-- add replicat fw, exttrail AdapterExamples/trail/tr
-- SETENV(GGS_JAVAUSEREXIT_CONF = 'dirprm/fw.props')
TARGETDB LIBFILE libggjava.so SET property=dirprm/f.props
REPORTCOUNT EVERY 1 MINUTES, RATE
GROUPTRANSOPS 1000
MAP QASOURCE.*, TARGET QASOURCE.*;

```
3. Add the replicat with the below command.

```
add replicat s3, exttrail AdapterExamples/trail/tr

```
4. Crosscheck for CloudERA Hadoop replicat’s status, RBA and stats.
Once you get the stats, you can view the s3.log from. /dirrpt directory which gives information about data sent to kinesis data stream and operations performed.
![](/images/s3/s3_002.JPG)


