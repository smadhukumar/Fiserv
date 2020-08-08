# Lab 3 -  Replicating to Apache Kafka

## Before You Begin
### Environment details 

- Apache  Kafka 2.6.0
- GoldenGate 19c for BigData -Version 19.1.0.0.4 (Build 001)
- kafka client libraries at *** /u01/ggbd/kafka-2.6.0-src/ ***


### Introduction
 This lab will guide through the steps required to start a replication to Apache Kafka.

### Objectives
- Replicate from trail files on the target machine to Apache Kafka.

### Time to Complete
Approximately 30 minutes


### What Do You Need?
You will need:
- Putty if you are using windows machine
- Apache kafka libraries 
```
[oracle@hol ggbd]$ wget http://apachemirror.wuchna.com/kafka/2.6.0/kafka_2.13-2.6.0.tgz
--2020-08-07 11:13:10--  http://apachemirror.wuchna.com/kafka/2.6.0/kafka_2.13-2.6.0.tgz
Resolving apachemirror.wuchna.com (apachemirror.wuchna.com)... 159.65.154.237
Connecting to apachemirror.wuchna.com (apachemirror.wuchna.com)|159.65.154.237|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 65537909 (63M) [application/x-gzip]
Saving to: ‘kafka_2.13-2.6.0.tgz’

100%[=============================================================================================>] 65,537,909  12.9MB/s   in 6.4s

[2020-08-07 11:13:17 (9.71 MB/s) - ‘kafka_2.13-2.6.0.tgz’ saved [65537909/65537909]

[oracle@hol ggbd]$ tar -zxvf kafka_2.13-2.6.0.tgz
[oracle@hol ggbd]$ cd kafka_2.13-2.6.0/
[oracle@hol kafka_2.13-2.6.0]$ ls -lrth
total 56K
-rw-r--r--. 1 oracle oinstall  337 Jul 28 18:16 NOTICE
-rw-r--r--. 1 oracle oinstall  30K Jul 28 18:16 LICENSE
drwxr-xr-x. 2 oracle oinstall 4.0K Jul 28 18:23 config
drwxr-xr-x. 3 oracle oinstall 4.0K Jul 28 18:23 bin
drwxr-xr-x. 2 oracle oinstall   44 Jul 28 18:23 site-docs
drwxr-xr-x. 2 oracle oinstall 8.0K Aug  7 11:42 libs
[oracle@hol kafka_2.13-2.6.0]$
```


1. Kindly create kafka props and prm files under dirprm goldengate home directory

```
[oracle@hol ~]$cd <GOLDENGATE_FOR_BIGDATA_HOME>/dirprm
[oracle@hol dirprm]$ vi kafka.props
```
Copy paste the below text in hdfs.props.

## Apache Kafka properties for GG4BD

```
gg.handlerlist = kafkahandler
gg.handler.kafkahandler.type=kafka
gg.handler.kafkahandler.KafkaProducerConfigFile=custom_kafka_producer.properties
#The following resolves the topic name using the short table name
## Topic name is madhuGoldenGate
gg.handler.kafkahandler.topicMappingTemplate=madhuGoldenGate
#The following selects the message key using the concatenated primary keys
gg.handler.kafkahandler.keyMappingTemplate=${primaryKeys}
gg.handler.kafkahandler.format=avro_op
gg.handler.kafkahandler.SchemaTopicName=mySchemaTopic
gg.handler.kafkahandler.BlockingSend =false
gg.handler.kafkahandler.includeTokens=false
gg.handler.kafkahandler.mode=op
#gg.handler.kafkahandler.metaHeadersTemplate=${alltokens}
gg.handler.kafkahandler.transactionsEnabled=false

goldengate.userexit.writers=javawriter
javawriter.stats.display=TRUE
javawriter.stats.full=TRUE

gg.log=log4j
gg.log.level=INFO

gg.report.time=30sec

#Sample gg.classpath for Apache Kafka
gg.classpath=dirprm/:/u01/ggbd/kafka_2.13-2.6.0/libs/*
#Sample gg.classpath for HDP
#gg.classpath=/etc/kafka/conf:/usr/hdp/current/kafka-broker/libs/*

javawriter.bootoptions=-Xmx512m -Xms32m -Djava.class.path=ggjava/ggjava.jar
```
Save the text using wq!

2. Edit the Replicat Parameter file to include the schema_name & table_name that needs to be replicated.
```
[oracle@hol ~]$cd <GOLDENGATE_FOR_BIGDATA_HOME>/dirprm
[oracle@hol dirprm]$ vi rkafka.prm

```
Copy paste the below text in rkafka.prm
```
REPLICAT rkafka
-- Trail file for this example is located in "AdapterExamples/trail" directory
-- Command to add REPLICAT
-- add replicat rkafka, exttrail AdapterExamples/trail/tr
TARGETDB LIBFILE libggjava.so SET property=dirprm/kafka.props
REPORTCOUNT EVERY 1 MINUTES, RATE
GROUPTRANSOPS 10000
MAP QASOURCE.*, TARGET QASOURCE.*;


```
3. Add Kafka producer property file to custom_kafka_producer.properties

Copy paste the below text in custom_kafka_producer.properties
- hostname = hol.sub05051224240.integration.oraclevcn.com
- Port = 9092

```
#bootstrap.servers=hostname:port
bootstrap.servers=hol.sub05051224240.integration.oraclevcn.com:9092
acks=1
reconnect.backoff.ms=1000

value.serializer=org.apache.kafka.common.serialization.ByteArraySerializer
key.serializer=org.apache.kafka.common.serialization.ByteArraySerializer
# 100KB per partition
batch.size=16384
linger.ms=0

```
4. Add the replicat with the below command.

```
[oracle@hol dirprm]$ ../ggsci

Oracle GoldenGate for Big Data
Version 19.1.0.0.4 (Build 001)

Oracle GoldenGate Command Interpreter
Version 19.1.0.0.2 OGGCORE_OGGADP.19.1.0.0.2_PLATFORMS_200508.0538
Linux, x64, 64bit (optimized), Generic on May  8 2020 07:21:07
Operating system character set identified as UTF-8.

Copyright (C) 1995, 2019, Oracle and/or its affiliates. All rights reserved.



GGSCI (hol) 1> add replicat rkafka, exttrail AdapterExamples/trail/tr
REPLICAT added.


GGSCI (hol) 2> info all

Program     Status      Group       Lag at Chkpt  Time Since Chkpt

MANAGER     RUNNING
REPLICAT    STOPPED     RKAFKA      00:00:00      00:00:07


GGSCI (hol) 3> start RKAFKA

Sending START request to MANAGER ...
REPLICAT RKAFKA starting


GGSCI (hol) 4> info all

Program     Status      Group       Lag at Chkpt  Time Since Chkpt

MANAGER     RUNNING
REPLICAT    RUNNING     RKAFKA      00:00:00      00:00:04


GGSCI (hol) 5>  stats rkafka

Sending STATS request to REPLICAT RKAFKA ...

Start of Statistics at 2020-08-08 17:55:48.

Replicating from QASOURCE.TCUSTMER to QASOURCE.TCUSTMER:

*** Total statistics since 2020-08-08 17:55:25 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

*** Daily statistics since 2020-08-08 17:55:25 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

*** Hourly statistics since 2020-08-08 17:55:25 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

*** Latest statistics since 2020-08-08 17:55:25 ***
        Total inserts                                      5.00
        Total updates                                      1.00
        Total deletes                                      0.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                   6.00

Replicating from QASOURCE.TCUSTORD to QASOURCE.TCUSTORD:

*** Total statistics since 2020-08-08 17:55:25 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

*** Daily statistics since 2020-08-08 17:55:25 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

*** Hourly statistics since 2020-08-08 17:55:25 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

*** Latest statistics since 2020-08-08 17:55:25 ***
        Total inserts                                      5.00
        Total updates                                      3.00
        Total deletes                                      2.00
        Total upserts                                      0.00
        Total discards                                     0.00
        Total operations                                  10.00

End of Statistics.


GGSCI (hol) 6> 

```
4. Crosscheck at Apache Kafka for the replicat’s status

```
[oracle@hol ggbd]$ cd kafka_2.13-2.6.0/
[oracle@hol kafka_2.13-2.6.0]$ bin/kafka-topics.sh --list --bootstrap-server localhost:9092
CUSTOGG
mySchemaTopic
[oracle@hol kafka_2.13-2.6.0]$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic CUSTOGG  --from-beginning
"QASOURCE.TCUSTMERI42015-11-05 18:45:36.00000042020-08-08T17:55:25.894000(00000000000000001956CUST_CODWILLBG SOFTWARE CO.SEATTLEWA
"QASOURCE.TCUSTMERI42015-11-05 18:45:36.00000042020-08-08T17:55:27.050000(00000000000000002126CUST_CODJANE ROCKY FLYER INC.
DENVERCO
"QASOURCE.TCUSTORDI42015-11-05 18:45:36.00000042020-08-08T17:55:27.058000(0000000000000000292CUST_CODEORDER_DATEPRODUCT_CODEORDER_IWILL&1994-09-30 15:33:00CARb@▒@Y@
"QASOURCE.TCUSTORDI42015-11-05 18:45:36.00000042020-08-08T17:55:27.063000(0000000000000000310CUST_CODEORDER_DATEPRODUCT_CODEORDER_IJANE&1995-11-11 13:52:00
PLANEp@▒EAY@
"QASOURCE.TCUSTMERI42015-11-05 18:45:39.00000042020-08-08T17:55:27.064000(00000000000000003286CUST_CODDAVE$DAVE'S PLANES INC.TALLAHASSEEFL
"QASOURCE.TCUSTMERI42015-11-05 18:45:39.00000042020-08-08T17:55:27.065000(00000000000000003462CUST_CODBILL BILL'S USED CARS
DENVERCO
"QASOURCE.TCUSTMERI42015-11-05 18:45:39.00000042020-08-08T17:55:27.065001(00000000000000003600CUST_CODEANNANN'S BOATSSEATTLEWA
"QASOURCE.TCUSTORDI42015-11-05 18:45:39.00000042020-08-08T17:55:27.066000(0000000000000000373CUST_CODEORDER_DATEPRODUCT_CODEORDER_IBILL&1995-12-31 15:00:00CAR▒@L▒@Y@
"QASOURCE.TCUSTORDI42015-11-05 18:45:39.00000042020-08-08T17:55:27.067000(0000000000000000394CUST_CODEORDER_DATEPRODUCT_CODEORDER_IBILL&1996-01-01 00:00:00
TRUCK▒t@d▒@Y@
"QASOURCE.TCUSTORDI42015-11-05 18:45:39.00000042020-08-08T17:55:27.070000(0000000000000000412CUST_CODEORDER_DATEPRODUCT_CODEORDER_IDAVE&1993-11-03 07:51:35
PLANE▒@▒zAi@
"QASOURCE.TCUSTORDU42015-11-05 18:45:39.00000042020-08-08T17:55:27.071000(0000000000000000430CUST_CODEORDER_DATEPRODUCT_CODEORDER_IBILL&1995-12-31 15:00:00CAR▒@L▒@YBILL&1995-12-31 15:00:00CAR▒@X▒@
"QASOURCE.TCUSTORDU42015-11-05 18:45:39.00000042020-08-08T17:55:27.072000(0000000000000000458CUST_CODEORDER_DATEPRODUCT_CODEORDER_IBILL&1996-01-01 00:00:00
TRUCK▒t@d▒@YBILL&1996-01-01 00:00:00
TRUCK▒t@j▒@
"QASOURCE.TCUSTORDU42015-11-05 18:45:39.00000042020-08-08T17:55:27.073000(0000000000000000484CUST_CODEORDER_DATEPRODUCT_CODEORDER_IWILL&1994-09-30 15:33:00CARb@▒@YWILL&1994-09-30 15:33:00CARb@"▒@
"QASOURCE.TCUSTMERU42015-11-05 18:45:39.00000042020-08-08T17:55:27.079000(00000000000000005100CUST_CODEANNANN'S BOATSSEATTLEWAANNNEW YORKNY
"QASOURCE.TCUSTORDD42015-11-05 18:45:39.00000042020-08-08T17:55:27.080000(0000000000000000527CUST_CODEORDER_DATEPRODUCT_CODEORDER_IDAVE&1993-11-03 07:51:35
PLANE▒@▒zAi@
"QASOURCE.TCUSTORDD42015-11-05 18:45:39.00000042020-08-08T17:55:27.085000(0000000000000000548CUST_CODEORDER_DATEPRODUCT_CODEORDER_IJANE&1995-11-11 13:52:00
PLANEp@▒EAY@


^CProcessed a total of 16 messages


``` 

You have completed replication to Kafka! Great Job!

<a href="https://github.com/smadhukumar/Fiserv" target="_blank">Click here to return</a>

