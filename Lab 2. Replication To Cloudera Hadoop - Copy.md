# Lab 1 -  oracle GoldenGate forBigdat Installation

## Before You Begin
### Environment details 

- Linux 64 bit OS
- GoldenGate 19c for BigData


### Introduction
 This lab will guide through the steps required for installation of the GoldenGate for Bigdata.

### Time to Complete
Approximately 30 minutes


### What Do You Need?
You will need:
- Putty if you are using windows machine

## Installation of GG4BD

1. Download the GoldenGate for bigdata binaries from Oracle site 

```
- click on the link https://www.oracle.com/in/middleware/technologies/goldengate-downloads.html
- Select **"Oracle GoldenGate for Big Data 19.1.0.0.1 on Linux x86-64"**

```
![](/images/bd/bd_1.png)

2. accpet the Oracle license agreement and press Download button
![](/images/bd/bd_2.png)

3. login the Oracel SSO login portal by passing registered usename and press
 ![](/images/bd/bd_3.png)

4. Save the Oracle Bigdata Binaries Files to Local desktop
![](/images/bd/bd_4.png)


5. Winscp the GG for Bigdata binaries to target GG bigdata environment 


6. Unzip and untar the gg4bd binaries 

```
[oracle@hol Downloads]$ mkdir /u01/ggbd
[oracle@hol Downloads]$ cd /u01/ggbd/
[oracle@hol ggbd]$ ll
total 314396
-rw-r--r--. 1 oracle oinstall      1371 Oct  8  2019 OGGBD-19.1.0.0-README.txt
-rw-r--r--. 1 oracle oinstall    339834 Oct  8  2019 OGG_BigData_19.1.0.0.1_Release_Notes.pdf
-rw-rw-r--. 1 oracle oinstall 321597440 Sep 25  2019 OGG_BigData_Linux_x64_19.1.0.0.1.tar
[oracle@hol ggbd]$ tar -xvf OGG_BigData_Linux_x64_19.1.0.0.1.tar
```
7. set the environment variable 

```
[oracle@hol ggbd]$ export JAVA_HOME=<<JAVA_HOME>
[oracle@hol ggbd]$ export PATH=$JAVA_HOME/bin:$PATH
[oracle@hol ggbd]$ export LD_LIBRARY_PATH=$JAVA_HOME/jre/lib/amd64/server

```

8. go to GG4BD untar folder location and do the below steps

```

[oracle@hol ggbd]$ pwd
/u01/ggbd
[oracle@hol ggbd]$ ./ggsci

Oracle GoldenGate for Big Data
Version 19.1.0.0.1 (Build 003)

Oracle GoldenGate Command Interpreter
Version 19.1.0.0.2 OGGCORE_OGGADP.19.1.0.0.2_PLATFORMS_190916.0039
Linux, x64, 64bit (optimized), Generic on Sep 16 2019 02:12:32
Operating system character set identified as UTF-8.

Copyright (C) 1995, 2019, Oracle and/or its affiliates. All rights reserved.



GGSCI (hol) 1> create subdirs

Creating subdirectories under current directory /u01/ggbd

Parameter file                 /u01/ggbd/dirprm: created.
Report file                    /u01/ggbd/dirrpt: created.
Checkpoint file                /u01/ggbd/dirchk: created.
Process status files           /u01/ggbd/dirpcs: created.
SQL script files               /u01/ggbd/dirsql: created.
Database definitions files     /u01/ggbd/dirdef: created.
Extract data files             /u01/ggbd/dirdat: created.
Temporary files                /u01/ggbd/dirtmp: created.
Credential store files         /u01/ggbd/dircrd: created.
Masterkey wallet files         /u01/ggbd/dirwlt: created.
Dump files                     /u01/ggbd/dirdmp: created.


GGSCI (hol) 2> edit params mgr



GGSCI (hol) 3> view params mgr

PORT 7810


GGSCI (hol) 4> start mgr
Manager started.


GGSCI (hol) 5> info mgr

Manager is running (IP port TCP:hol.7810, Process ID 1935).


GGSCI (hol) 6> info all

Program     Status      Group       Lag at Chkpt  Time Since Chkpt

MANAGER     RUNNING


```






