# Lab 4. Replication To MongoDB

## Before You Begin
### Environment details 

- MongoDB  version v4.4.0
- GoldenGate 19c for BigData -Version 19.1.0.0.4 (Build 001)


### Introduction
 This lab will guide through the steps required to start a replication to  MongoDB.

### Objectives
- Replicate from trail files on the target machine to  MongoDB Server.

### Time to Complete
Approximately 30 minutes


### What Do You Need?
You will need:
- Putty if you are using windows machine
- The MongoDB Java Driver is required for Oracle GoldenGate for Big Data to connect and stream data to MongoDB. The minimum required version of the MongoDB Java Driver is 3.4.3.  The MongoDB Java Driver is not included in the Oracle GoldenGate for Big Data product. You must download the driver from:

https://docs.mongodb.com/ecosystem/drivers/java/#download-upgrade

Select mongo-java-driver and the 3.4.3 version to download the recommended driver JAR file.

You must configure the gg.classpath variable to load the MongoDB Java Driver JAR at runtime. For example: gg.classpath=/home/mongodb/mongo-java-driver-3.4.3.jar



1. Kindly create mongo props and prm files under dirprm goldengate home directory

```
[oracle@ogg-monitor ~]$cd <GOLDENGATE_FOR_BIGDATA_HOME>/dirprm
[oracle@ogg-monitor dirprm]$ vi rmongo.props
```
Copy paste the below text in rmongo.props.

##  rmongo properties for GG4BD

```
gg.handlerlist=mongodb
gg.handler.mongodb.type=mongodb

#The following handler properties are optional.
#Please refer to the Oracle GoldenGate for BigData documentation
#for details about the configuration.


gg.handler.mongodb.clientURI=mongodb://localhost:27017/
#gg.handler.mongodb.Host=<MongoDBServer address>
#gg.handler.mongodb.Port=<MongoDBServer port>
#gg.handler.mongodb.WriteConcern={ w: <value>, wtimeout: <number> }
#gg.handler.mongodb.AuthenticationMechanism=GSSAPI,MONGODB_CR,MONGODB_X509,PLAIN,SCRAM_SHA_1
#gg.handler.mongodb.UserName=<Authentication username>
#gg.handler.mongodb.Password=<Authentication password>
#gg.handler.mongodb.Source=<Authentication source>
#gg.handler.mongodb.ServerAddressList=localhost1:27017,localhost2:27018,localhost3:27019,...
#gg.handler.mongodb.BulkWrite=<false|true>
#gg.handler.mongodb.CheckMaxRowSizeLimit=<true|false>


goldengate.userexit.writers=javawriter
javawriter.stats.display=TRUE
javawriter.stats.full=TRUE
gg.log=log4j
gg.log.level=INFO
gg.report.time=30sec

#Path to MongoDB Java driver.
# maven co-ordinates
#        <dependency>
#            <groupId>org.mongodb</groupId>
#            <artifactId>mongo-java-driver</artifactId>
#            <version>3.2.2</version>
#        </dependency>
#gg.classpath=/path/to/mongodb/java/driver/mongo-java-driver-3.2.2.jar
gg.classpath=/home/opc/mongo_jar/mongo-java-driver-3.4.3.jar

javawriter.bootoptions=-Xmx512m -Xms32m -Djava.class.path=.:ggjava/ggjava.jar:./dirprm


```
Save the text using wq!

2. Edit the Replicat Parameter file to include the schema_name & table_name that needs to be replicated.
```
[oracle@ogg-monitor ~]$cd <GOLDENGATE_FOR_BIGDATA_HOME>/dirprm
[oracle@ogg-monitor dirprm]$ vi rmongo.prm

```
Copy paste the below text in rmongo.prm
```
REPLICAT rmongo
-- Trail file for this example is located in "AdapterExamples/trail" directory
-- Command to add REPLICAT
-- add replicat rmongo, exttrail AdapterExamples/trail/tr
TARGETDB LIBFILE libggjava.so SET property=dirprm/mongo.props
REPORTCOUNT EVERY 1 MINUTES, RATE
GROUPTRANSOPS 1000
MAP QASOURCE.*, TARGET QASOURCE.*;


```

3. Add the replicat with the below command.

```
[oracle@ogg-monitor dirprm]$ ../ggsci

Oracle GoldenGate for Big Data
Version 19.1.0.0.4 (Build 001)

Oracle GoldenGate Command Interpreter
Version 19.1.0.0.2 OGGCORE_OGGADP.19.1.0.0.2_PLATFORMS_200508.0538
Linux, x64, 64bit (optimized), Generic on May  8 2020 07:21:07
Operating system character set identified as UTF-8.

Copyright (C) 1995, 2019, Oracle and/or its affiliates. All rights reserved.



GGSCI (ogg-monitor) 1> info all

Program     Status      Group       Lag at Chkpt  Time Since Chkpt

MANAGER     RUNNING

GGSCI (ogg-monitor) 2> view params  rmongo

REPLICAT rmongo
-- Trail file for this example is located in "AdapterExamples/trail" directory
-- Command to add REPLICAT
-- add replicat rmongo, exttrail AdapterExamples/trail/tr
TARGETDB LIBFILE libggjava.so SET property=dirprm/mongo.props
REPORTCOUNT EVERY 1 MINUTES, RATE
GROUPTRANSOPS 1000
MAP QASOURCE.*, TARGET QASOURCE.*;



GGSCI (ogg-monitor) 3> add replicat rmongo, exttrail AdapterExamples/trail/tr
REPLICAT added.


GGSCI (ogg-monitor) 4> start replicat rmongo

Sending START request to MANAGER ...
REPLICAT RMONGO starting


GGSCI (ogg-monitor) 5> info all

Program     Status      Group       Lag at Chkpt  Time Since Chkpt

MANAGER     RUNNING
REPLICAT    RUNNING     RMONGO      00:00:00      00:00:15


GGSCI (ogg-monitor) 6> !
info all

Program     Status      Group       Lag at Chkpt  Time Since Chkpt

MANAGER     RUNNING
REPLICAT    RUNNING     RMONGO      00:00:01      00:00:00


GGSCI (ogg-monitor) 7>stats RMONGO

Sending STATS request to REPLICAT RMONGO ...

Start of Statistics at 2020-10-15 12:05:47.

Replicating from QASOURCE.TCUSTMER to QASOURCE.TCUSTMER:

*** Total statistics since 2020-10-15 12:05:37 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

*** Daily statistics since 2020-10-15 12:05:37 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

*** Hourly statistics since 2020-10-15 12:05:37 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

*** Latest statistics since 2020-10-15 12:05:37 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

Replicating from QASOURCE.TCUSTORD to QASOURCE.TCUSTORD:

*** Total statistics since 2020-10-15 12:05:37 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

*** Daily statistics since 2020-10-15 12:05:37 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

*** Hourly statistics since 2020-10-15 12:05:37 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

*** Latest statistics since 2020-10-15 12:05:37 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

End of Statistics.


GGSCI (ogg-monitor) 8> 

```
4. Crosscheck at MongoDB for the replicat’s status

```
[root@ogg-monitor ~]#    mongo
MongoDB shell version v4.4.0
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("a138ea72-9709-4917-afc2-3441638919ef") }
MongoDB server version: 4.4.0
---
The server generated these startup warnings when booting:
        2020-10-15T11:02:35.706+00:00: ***** SERVER RESTARTED *****
        2020-10-15T11:02:36.086+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
---
---
        Enable MongoDB's free cloud-based monitoring service, which will then receive and display
        metrics about your deployment (disk utilization, CPU, operation statistics, etc).

        The monitoring data will be available on a MongoDB website with a unique URL accessible to you
        and anyone you share the URL with. MongoDB may use this information to make product
        improvements and to suggest MongoDB products and deployment options to you.

        To enable free monitoring, run the following command: db.enableFreeMonitoring()
        To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---

> show dbs
QASOURCE  0.000GB
admin     0.000GB
config    0.000GB
local     0.000GB
madhu     0.000GB
> use QASOURCE;
switched to db QASOURCE
> show collections
TCUSTMER
TCUSTORD
>

``` 

You have completed replication to  MongoDB :sunglasses: ! Great Job! 	:wink:

<a href="https://github.com/smadhukumar/Fiserv" target="_blank">Click here to return</a>

