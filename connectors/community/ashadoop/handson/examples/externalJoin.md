---
title: Map-Reduce Example - Sessionization, User Profile and External Join
description: Sessionization, User Profile and External Join Map-Reduce Example using Aerospike Connector
---

This is a three part example. It uses web server log data stored in HDFS. In step 1, the log data is sessionized and output to Aerospike. In step 2, userids in the web server logs are assigned a profile (age and gender, generated randomly). Finally in step 3, sessionization data is again extracted from web server logs (just like step 1) and then joined with userid profile data read from Aerospike, the joined data is re-written back to Aerospike into a new set.

### Get the data set

Data for this example can be obtained from:

 http://ita.ee.lbl.gov/html/contrib/WorldCup.html

The data provided is in binary format. They also provide a tool called `recreate` to convert the data to text (HDFS requires data in text format).

 
Example of using the `recreate` tool to convert binary files to text on a MacBook:  

- Download WorldCup_tools.tar and open a terminal on a MacBook.

```asciidoc
MacBookPro:$cd Downloads
MacBookPro:$tar -xvf WorldCup_tools.tar
MacBookPro:$cd /Users/yourname/Downloads/ita_public_tools
```   
- Make the `recreate` executable in bin directory.

```asciidoc
MacBookPro:$make recreate
```
 
- Sample conversion:

```asciidoc
MacBookPro:$gzip -dc input/test_log.gz | bin/recreate state/object_mappings.sort > output/recreate.out
``` 

- Get the test data 

Get the data files as shown below. Download `wc_day51_1.gz` and `wc_day52_2.gz` 
using the download link to `~/Downloads/ita_public_tools/input` as follows:

<center>
{{#figure "" "Downloading World Cup Test Data" size="medium" }}
<img src="/docs/connectors/assets/images/DownloadingWCTestData.png">
{{/figure}}
</center>

- Extract the data as shown below.

```asciidoc
MacBookPro:ita_public_tools$ gzip -dc input/wc_day52_1.gz | bin/recreate state/object_mappings.sort > output/wc_day52_1.out
Initializing
Reading Access Log
0
1000000
2000000
3000000
4000000
5000000
6000000
6999999
Printing Results
    Total Requests: 6999999
       Total Bytes: 31015800544
Mean Transfer Size: 4430.829282
     Max Client ID: 1817715
     Max Object ID: 55185
        Start Time: 897948001
       Finish Time: 897970055
      Out of Order: 0
Terminating

MacBookPro:ita_public_tools$ gzip -dc input/wc_day52_2.gz | bin/recreate state/object_mappings.sort > output/wc_day52_2.out
```

- Copy files from MacBook to user `hdclient`’s home directory on `ztg-client`.

```asciidoc
MacBookPro:ita_public_tools$ cd output
MacBookPro:output$scp wc_day52_1.out hdclient@x.y.z.209:~
hdclient@x.y.z.209's password: 
wc_day52_1.out 100%  612MB  10.0MB/s   01:01    
MacBookPro:output$ scp wc_day52_2.out hdclient@x.y.z.209:~
hdclient@x.y.z.209's password: 
wc_day52_2.out 100%  611MB   7.6MB/s   01:21    
```

- Move data to HDFS from `ztg-client` local filesystem.

```asciidoc
hdclient@ztg-client:~$ hdfs dfs -mkdir /tmp/worldcup

hdclient@ztg-client:~$ hdfs dfs -copyFromLocal ~/wc_day52_1.out /tmp/worldcup/wc_day52_1.log

hdclient@ztg-client:~$ hdfs dfs -copyFromLocal ~/wc_day52_2.out /tmp/worldcup/wc_day52_2.log
```

**Check:**

```asciidoc
hdclient@ztg-client:~$ hdfs dfs -ls /tmp/worldcup
Found 2 items
-rw-r--r--   3 hdclient supergroup  641300226 2015-06-30 12:42 /tmp/worldcup/wc_day52_1.log
-rw-r--r--   3 hdclient supergroup  640853121 2015-06-30 12:44 /tmp/worldcup/wc_day52_2.log
hdclient@ztg-client:~$
```

### Step 1: Identify sessions

In this step (`sesssion_rollup.jar`), we take the web server log data and sessionize it. We identify sessions by looking at the timestamps of user activity, identified by userid and demarcate a session if the gap exceeds twenty minutes.

The web server log files data will reside on HDFS. The session data will be written to Aerospike into `test:sessions`.

Typical log line entries are shown below:

```asciidoc
946628 - - [15/Jun/1998:22:00:01 +0000] "GET /images/blank.gif HTTP/1.1" 200 85
1781297 - - [15/Jun/1998:22:00:01 +0000] "GET /images/nav_bg_bottom.jpg HTTP/1.0" 200 8389
1781297 - - [15/Jun/1998:22:00:01 +0000] "GET /english/images/nav_news_off.gif HTTP/1.0" 200 853
```

Here, `966628` is the user id, two dash characters, followed by the timestamp `[15/Jun/1998:22:00:01 +0000]` and then the HTTP call plus two other data items. There are 7 fields in total.


<center>
{{#figure "" "Session Rollup Example" size="large" }}
<img src="/docs/connectors/assets/images/SessionRollupExample.png">
{{/figure}}
</center>

**Source Code:**

```asciidoc
~/aerospike-hadoop/examples/session_rollup/src/main/java/com/aerospike/hadoop/examples/sessionrollup/SessionRollup.java
```

- Run the session_rollup example.

```asciidoc
hdclient@ztg-client:~$ cd aerospike-hadoop
hdclient@ztg-client:~/aerospike-hadoop$ hadoop jar 	./examples/session_rollup/build/libs/session_rollup.jar -D aerospike.output.host=ztg-client -D aerospike.output.namespace=test -D aerospike.output.setname=sessions -D mapred.reduce.tasks=30 	/tmp/worldcup/wc_day52_1.log /tmp/worldcup/wc_day52_2.log
```
- Check results on Aerospike using AQL: 
 
**Note:** session_rollup.jar writes records in Aerospike using Primary Key = hash of userid and start time.

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop$ aql
aql> select * from test.sessions
+---------+--------------+--------------+-------+
| userid  | start        | end          | nhits |
+---------+--------------+--------------+-------+
| 92379   | 897983088000 | 897984349000 | 34    |
| 744382  | 897948334000 | 897948392000 | 2     |
| 1521723 | 897994991000 | 897995171000 | 87    |
| 53493   | 897974060000 | 897974569000 | 187   |
| 680860  | 897948612000 | 897949083000 | 62    |
| 1595044 | 897985429000 | 897985923000 | 174   |
| 1535024 | 897991912000 | 897994251000 | 255   |
. . .
```

To better investigate the data, create a secondary index on userid and search by userid.

```asciidoc
aql> create index useridndx on test.sessions (userid) NUMERIC
OK, 1 index added.

aql> select * from test.sessions where userid = 946628
+--------+--------------+--------------+-------+
| userid | start        | end          | nhits |
+--------+--------------+--------------+-------+
| 946628 | 897948001000 | 897948015000 | 15    |
+--------+--------------+--------------+-------+
1 row in set (0.005 secs)

aql> select * from test.sessions where userid = 369157
+--------+--------------+--------------+-------+
| userid | start        | end          | nhits |
+--------+--------------+--------------+-------+
| 369157 | 897987168000 | 897988759000 | 165   |
| 369157 | 897979400000 | 897979439000 | 8     |
| 369157 | 897974355000 | 897974643000 | 107   |
| 369157 | 897982167000 | 897982663000 | 96    |
+--------+--------------+--------------+-------+
4 rows in set (0.003 secs)

aql> exit
```

### Step 2: Generate Profiles by userid

In this step, we will again read the web server log data in HDFS,
use the userid as the key in the map phase assigning each instance a value of 1,
then reduce by userid, ignore the value of 1 from the map operation and
assign randomly generated age and gender values to the userid key in the reduce phase.
The reducer will write this data on Aerospike into the `test:profiles` set.

<center>
{{#figure "" "Generate Profiles by userid" size="large" }}
<img src="/docs/connectors/assets/images/GenerateProfilesExample.png">
{{/figure}}
</center>

**Source Code:**

```asciidoc
~/aerospike-hadoop/examples/generate_profiles/src/main/java/com/aerospike/hadoop/examples/generateprofiles/GenerateProfiles.java
```
- Run the `generate_profiles` example.

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop$ hadoop jar 	./examples/generate_profiles/build/libs/generate_profiles.jar -D aerospike.output.host=ztg-client -D aerospike.output.namespace=test -D aerospike.output.setname=profiles -D mapred.reduce.tasks=30 	/tmp/worldcup/wc_day52_1.log 	/tmp/worldcup/wc_day52_2.log
```

- Check results on Aerospike using AQL:

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop$ aql
Aerospike Query
Copyright 2013 Aerospike. All rights reserved.

aql> select * from test.profiles
+---------+-----+--------+
| userid  | age | isMale |
+---------+-----+--------+
| 17224   | 44  | 0      |
| 104797  | 57  | 1      |
| 866372  | 32  | 0      |
| 869891  | 31  | 1      |
| 779266  | 46  | 0      |
. . . 

aql> select * from test.profiles where PK = 946628
+--------+-----+--------+
| userid | age | isMale |
+--------+-----+--------+
| 946628 | 48  | 0      |
+--------+-----+--------+
1 row in set (0.000 secs)

aql> exit
```

### Step 3: Perform External Join

In this step, sessionization data is again extracted from web server logs 
(just like step 1) and then joined with userid profile data read from 
Aerospike `test:profiles` set. The joined data is re-written back 
to Aerospike into a new set, `test:session2`. 


<center>
{{#figure "" "External Join Example" size="large" }}
<img src="/docs/connectors/assets/images/ExternalJoinExample.png">
{{/figure}}
</center>

**Source Code:**

```asciidoc
~/aerospike-hadoop/examples/external_join/src/main/java/com/aerospike/hadoop/examples/externaljoin/ExternalJoin.java
```
- Run the `external_join` example.

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop$ hadoop jar ./examples/external_join/build/libs/external_join.jar -D aerospike.input.host=ztg-client -D aerospike.input.namespace=test -D aerospike.input.setname=profiles -D aerospike.output.host=ztg-client -D aerospike.output.namespace=test -D aerospike.output.setname=sessions2 -D mapred.reduce.tasks=30 /tmp/worldcup/wc_day52_1.log /tmp/worldcup/wc_day52_2.log

15/06/30 13:19:08 INFO externaljoin.ExternalJoin: run starting
. . .
15/06/30 13:21:26 INFO externaljoin.ExternalJoin: finished
hdclient@ztg-client:~/aerospike-hadoop$
```

- Check results of external join using AQL. 

The data in `test:sessions2` is written with Primary Key which is SHA-256 hash of userid and start time.   

```asciidoc
hdclient@ztg-client:~/aerospike-hadoop$ aql
Aerospike Query
Copyright 2013 Aerospike. All rights reserved.

aql> select * from test.sessions2
+---------+--------------+--------------+-------+-----+--------+
| userid  | start        | end          | nhits | age | isMale |
+---------+--------------+--------------+-------+-----+--------+
| 218974  | 897958349000 | 897958372000 | 11    | 34  | 0      |
| 22543   | 897964928000 | 897966405000 | 285   | 43  | 1      |
| 7841    | 897983516000 | 897983524000 | 2     | 21  | 1      |
| 256876  | 897958917000 | 897958927000 | 8     | 56  | 0      |
| 1468715 | 897972168000 | 897972305000 | 83    | 55  | 1      |
. . .
We can create an index on userid in AQL to query specific userid data.

aql> create index useridndx2 on test.sessions2 (userid) NUMERIC
OK, 1 index added.

aql> select * from test.sessions2 where userid = 946628
+--------+--------------+--------------+-------+-----+--------+
| userid | start        | end          | nhits | age | isMale |
+--------+--------------+--------------+-------+-----+--------+
| 946628 | 897948001000 | 897948015000 | 15    | 48  | 0      |
+--------+--------------+--------------+-------+-----+--------+
1 row in set (0.005 secs)

aql> exit
```

