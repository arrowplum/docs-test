---
title: Apache Hadoop CLI 
description: A quick reference guide to hadoop command line interface. 
---

# HDFS Command Line Interface

Users can interact with HDFS by executing shell like commands.

USAGE:  
 `hadoop fs - <command> - <option> <URI>`

Some distributions use `dfs` instead of `fs` and `hdfs` instead of `hadoop`.

Alternate USAGE:   
`hdfs dfs - <command> - <option> <URI>`

URI format is `scheme://host:port/path`  
e.g  `hdfs://localhost/`  will list files from `/` directory

Most commands behave like Unix commands.    
`ls`, `cat`, `du`  

Supports HDFS specific operations. e.g changing replication. To get help type `–help` or display help for a specific command.  
e.g `hadoop fs –help <command_name>`

**Relative path:** It works relative to user’s home directory, where home directory is `/user/<username>`
```asciidoc
[-ls <path>]
[-lsr <path>]
[-df [<path>]]
[-du <path>]
[-dus <path>]
[-count[-q] <path>]
[-mv <src> <dst>]
[-cp <src> <dst>]
[-rm [-skipTrash] <path>]
[-rmr [-skipTrash] <path>]
[-expunge]
[-put <localsrc> ... <dst>]
[-copyFromLocal <localsrc> ... <dst>]
[-moveFromLocal <localsrc> ... <dst>]
[-get [-ignoreCrc] [-crc] <src> <localdst>]
[-getmerge <src> <localdst> [addnl]]
[-cat <src>]
[-text <src>]
[-copyToLocal [-ignoreCrc] [-crc] <src> <localdst>]
[-moveToLocal [-crc] <src> <localdst>]
[-mkdir <path>]
[-touchz <path>]
[-test -[ezd] <path>]
[-stat [format] <path>]
[-tail [-f] <file>]
[-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
[-chown [-R] [OWNER][:[GROUP]] PATH...]
[-chgrp [-R] GROUP PATH...]
[-help [cmd]]
```
`cat` – stream source to standard output  
`$ hadoop fs –cat /dir/readme.txt`

`cp` – copy file in hdfs from source to destination  
`$ hadoop fs –cp /dir/file1 /otherdir/file1`  

`ls` – list file/s with stats in a directory  
`$ hadoop fs –ls /dir`  

