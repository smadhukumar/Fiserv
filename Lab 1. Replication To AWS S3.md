# Lab 1 -  Oracle to AWS S3

## Before You Begin
-AWS SDK 
-Create a S3 bucket on your AWS Instance, Follow the link for reference-
https://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html

-Attach the following policies to the newly/existed user to allow access and GET/Put Operations on AWS S3:
AmazonS3FullAccess in AWS.

-You need to attach the following inline policy as json:

```
{
  "Version": "2012-10-17",
  "Statement": [{
      "Sid": "VisualEditor1",
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject"],
      "Resource": [
        "arn:aws:s3:::examplebucket/*"
      ]
   }
}

```


### Introduction
 This lab will guide through the steps required to start a replication of data between an Oracle database and AWS S3.

### Objectives
- Replicate from trail files on the target machine to AWS S3.

### Time to Complete
Approximately 30 minutes


### What Do You Need?
You will need:
- Putty if you are using windows machine


1. Kindly create S3 props and prm files under dirprm goldengate home directory

```
[oracle@gg4bd-target01 ggbd_home1]$ cd dirprm
[oracle@gg4bd-target01 dirprm]$ vi s3.props

```
Copy paste the below text in s3.props.


## AWS S3 properties for GG4BD

```
gg.handlerlist=filewriter
#The File Writer Handler
gg.handler.filewriter.type=filewriter
gg.handler.filewriter.mode=op
gg.handler.filewriter.pathMappingTemplate=./dirout
gg.handler.filewriter.stateFileDirectory=./dirsta
gg.handler.filewriter.fileNameMappingTemplate=${fullyQualifiedTableName}_${currentTimestamp}.txt
gg.handler.filewriter.fileRollInterval=7m
gg.handler.filewriter.finalizeAction=delete
gg.handler.filewriter.inactivityRollInterval=7m
gg.handler.filewriter.format=avro_row_ocf
gg.handler.filewriter.includetokens=true
gg.handler.filewriter.partitionByTable=true

#Selecting the Parquet Event Handler
gg.handler.filewriter.eventHandler=s3
gg.handler.filewriter.rollOnShutdown=true

#The S3 Event Handler
gg.eventhandler.s3.type=s3
gg.eventhandler.s3.region=ap-south-1
gg.eventhandler.s3.bucketMappingTemplate=oraclegg
gg.eventhandler.s3.pathMappingTemplate=oraclegg
gg.eventhandler.s3.finalizeAction=none

goldengate.userexit.timestamp=utc
goldengate.userexit.writers=javawriter
javawriter.stats.display=TRUE
javawriter.stats.full=TRUE
gg.log=log4j
gg.log.level=INFO
gg.includeggtokens=true
gg.report.time=30sec
#Set the classpath for AWS S3  - Need the AWS Java SDK
gg.classpath=/u01/app/aws/aws-java-sdk-1.11.718/lib/*:/u01/app/aws/aws-java-sdk-1.11.718/third-party/lib/*

#Set the heap memory
javawriter.bootoptions=-Xmx512m -Xms32m -Djava.class.path=.:ggjava/ggjava.jar:./dirprm -Daws.accessKeyId=<access-key-of-new-created-user> -Daws.secretKey=<secret-ke-new-created-user>

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
4. Crosscheck for AWS S3 replicat’s status, RBA and stats.
Once you get the stats, you can view the s3.log from. /dirrpt directory which gives information about data sent to kinesis data stream and operations performed.
![](/images/s3/s3_002.JPG)


