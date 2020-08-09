# Lab 1 -  Replicating to Cloudera Hadoop

## Before You Begin
### Environment details 

- Hadoop 3.0.0-cdh6.3.2
- GoldenGate 19c for BigData
- Hadoop client libraries at *** /opt/Cloudera/parcels/CDH/lib/hadoop/client/ ***


### Introduction
 This lab will guide through the steps required to start a replication to Cloudera Hadoop.

### Objectives
- Replicate from trail files on the target machine to Cloudera Hadoop.

### Time to Complete
Approximately 30 minutes


### What Do You Need?
You will need:
- Putty if you are using windows machine
- Creae target path like below 
```
[root@hol ~]# sudo -u hdfs hdfs dfs -chown oracle:oinstall  /ogg1
[root@hol ~]# sudo -u hdfs hdfs dfs -chown -R oracle:oinstall  /ogg1
```


1. Kindly create hdfs props and prm files under dirprm goldengate home directory

```
[oracle@hol ~]$cd <GOLDENGATE_FOR_BIGDATA_HOME>/dirprm
[oracle@hol dirprm]$ vi hdfs.props

```
Copy paste the below text in hdfs.props.


## Cloudera Hadoop properties for GG4BD

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
gg.classpath=/opt/Cloudera/parcels/CDH/lib/hadoop/client/*:/etc/hadoop/conf
#Sample gg.classpath for HDP
#gg.classpath=/usr/hdp/current/hadoop-client/client/*:/etc/hadoop/conf

javawriter.bootoptions=-Xmx512m -Xmhdfs2m -Djava.class.path=ggjava/ggjava.jar
```
Save the text using wq!

2. Edit the Replicat Parameter file to include the schema_name & table_name that needs to be replicated.
```
[oracle@hol ~]$cd <GOLDENGATE_FOR_BIGDATA_HOME>/dirprm
[oracle@hol dirprm]$ vi rhdfs.prm

```
Copy paste the below text in rhdfs.prm
```
REPLICAT rhdfs
-- Trail file for this example is located in "AdapterExamples/trail" directory
-- Command to add REPLICAT
-- add replicat rhdfs, exttrail AdapterExamples/trail/tr
TARGETDB LIBFILE libggjava.so SET property=dirprm/hdfs.props
REPORTCOUNT EVERY 1 MINUTES, RATE
GROUPTRANSOPS 10000
MAP QASOURCE.*, TARGET QASOURCE.*;


```
3. Add the replicat with the below command.

```
[oracle@hol dirprm]$ ../ggsci

Oracle GoldenGate for Big Data
Version 19.1.0.0.4 (Build 001)

Oracle GoldenGate Command Interpreter
Version 19.1.0.0.2 OGGCORE_OGGADP.19.1.0.0.2_PLATFORMS_200508.0538
Linux, x64, 64bit (optimized), Generic on May  8 2020 07:21:07
Operating system character set identified as UTF-8.

Copyright (C) 1995, 2019, Oracle and/or its affiliates. All rights reserved.



GGSCI (hol) 1> add replicat rhdfs, exttrail AdapterExamples/trail/tr
REPLICAT added.


GGSCI (hol) 2> info all

Program     Status      Group       Lag at Chkpt  Time Since Chkpt

MANAGER     RUNNING
REPLICAT    STOPPED     RHDFS       00:00:00      00:00:03


GGSCI (hol) 3> start RHDFS

Sending START request to MANAGER ...
REPLICAT RHDFS starting


GGSCI (hol) 4> info all

Program     Status      Group       Lag at Chkpt  Time Since Chkpt

MANAGER     RUNNING
REPLICAT    RUNNING     RHDFS       00:00:00      00:00:04


GGSCI (hol) 5> stats rhdfs

Sending STATS request to REPLICAT RHDFS ...

Start of Statistics at 2020-07-31 16:18:24.

Replicating from QASOURCE.TCUSTMER to QASOURCE.TCUSTMER:

*** Total statistics since 2020-07-31 14:39:24 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

*** Daily statistics since 2020-07-31 14:39:24 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

*** Hourly statistics since 2020-07-31 14:39:24 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

*** Latest statistics since 2020-07-31 14:39:24 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

Replicating from QASOURCE.TCUSTORD to QASOURCE.TCUSTORD:

*** Total statistics since 2020-07-31 14:39:24 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

*** Daily statistics since 2020-07-31 14:39:24 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

*** Hourly statistics since 2020-07-31 14:39:24 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

*** Latest statistics since 2020-07-31 14:39:24 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

End of Statistics.


GGSCI (hol) 6> 

```
4. Crosscheck at Cloudera Hadoop for the replicat’s status

```
[oracle@hol ggbd]$ hdfs dfs -ls /ogg1
Found 2 items
-rw-r--r--   3 oracle oinstall        905 2020-07-31 14:39 /ogg1/QASOURCE.TCUSTMER_RHDFS_2020-07-31_14-39-24.538.txt
-rw-r--r--   3 oracle oinstall       2305 2020-07-31 14:39 /ogg1/QASOURCE.TCUSTORD_RHDFS_2020-07-31_14-39-25.896.txt
``` 


You have completed replication to Cloudera hadoop :sunglasses: ! Great Job! 	:wink:

<a href="https://github.com/smadhukumar/Fiserv" target="_blank">Click here to return</a>
