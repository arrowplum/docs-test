---
title: asmonitor - Creating New Tables
description: Learn how you can create new or modify existing tables to organize the information that is displayed by the asmonitor command.
breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: Aerospike Monitor
    url: /docs/v3/Aerospike Monitor.html
---

{{#note}}
Deprecated from Aerospike Tools Versions >= 3.9.0. Use asadm instead.
{{/note}}

The various views in asmonitor are shown with the help of tables.

`asmonitor` allows you to create persistent tables for monitoring certain statistics.  The list of statistics that you can monitor is in the reference.

Once you create your table definition, you can display the table and the table name will be shown in the list of available tables.

For example, if you wanted to monitor the number of pending limit errors, your `asmonitor`session to create the table definition and display the table would look like this:

```
Monitor> createtable
Create new table (response n if you do not want to create) (Y/n)? y
Enter Table name : PendingLimit
table name= PendingLimit
Type of table (node,namespace,xdr,sets) : node
Table type= node
---------------
List of keys allowed for reference: tscan_pending , err_rw_request_not_found , partition_ref_count , rw_err_dup_internal , ....[List of all available stat names] , stat_proxy_errs ,
---------------
Enter the Number of Columns in the table(excluding 1st column as it will be IP:port) : 1
1st column will be Node IP
Enter Name of 1 Column in the table: Pending Limit Errors
Enter key for Pending Limit Errors Column in the table: err_rw_pending_limit
Table PendingLimit created!
Monitor> info
Namespace     Node          PendingLimit  SETS          XDR          
Monitor> info PendingLimit
 
 === PENDINGLIMIT ===
Node IP                 Pending
                          Limit
                         Errors
u10.aerospike.local         86
u11.aerospike.local          0
u9.aerospike.local          12
No. of rows: 3
Monitor>
```



If you do not like the table pre-created or if you want to replace certain columns with columns you care about, asmonitor has a command edittable, which will let you do that.

For example, if you want to see the number of client connections to each node, you can replace a column in the standard Node table with the column that you wish to see.  The asmonitor console session would look like this:

```
Monitor> edittable
Do you want to Edit or delete a table (response n if you do not want to edit/delete) (Y/n)? y
Enter Table name : Node
table name= Node
Do you want to Delete this table (response y if you want to Delete) (y/N)?
---------------
List of keys allowed for reference: tscan_pending , err_rw_request_not_found , partition_ref_count , rw_err_dup_internal , ....[List of all available stat names] , stat_proxy_errs ,
---------------
Do you want to delete or replace Cluster Size (no response to next) (d/r)?
Do you want to delete or replace Node id (no response to next) (d/r)?
Do you want to delete or replace Sys Free Mem (no response to next) (d/r)? r
Enter New Column Name: Client Connections
Column name= Client Connections
Enter New key for Client Connections Column in the table: client_connections
Do you want to delete or replace Cluster Visibility (no response to next) (d/r)?
Do you want to delete or replace Replicated Objects (no response to next) (d/r)?
Do you want to delete or replace Free Disk pct (no response to next) (d/r)?
Do you want to delete or replace Migrates (no response to next) (d/r)?
Do you want to delete or replace Free Mem pct (no response to next) (d/r)?
Do you want to delete or replace Build (no response to next) (d/r)?
Monitor> info Node
 
 === NODE ===
ip:port                 Build        Client   Cluster      Cluster   Free   Free   Migrates              Node   Replicated
                            .   Connections      Size   Visibility   Disk    Mem          .                id      Objects
                            .             .         .            .    pct    pct          .                 .            .
u11.aerospike.local    3.2.8            48        12         true     76     65      (0,0)   BB9500905CA0568    313.117 M
u9.aerospike.local      3.2.8            36        12         true     76     66      (0,0)   BB93AF106CA0568    312.120 M
u10.aerospike.local    3.2.8            26        12         true     76     66      (0,0)   BB931F106CA0568    312.609 M
No. of rows: 12
Monitor>
```



### Displaying Tables

In the asmonitor console, there are four standard (default) tables which you can display with the info command:

- Node
- Namespace
- SETS
- XDR

#### Displaying a List of Available Tables

To list all of the available tables available, use the autocomplete feature: type info followed by the spacebar followed by <tab> <tab>.  A list of tables appears.
Displaying the Node and Namespace Tables

If you enter the info command without parameters, the Nodes and Namespace tables are displayed.  


In the `asmonitor `console, the `sorttable` command allows you to display a table, sorted by any column that you prefer.

For example, the Namespace table is sorted by Available percent.  If you want to sort the Namespace table by IP/DNS, the `asmonitor` session would look like this:

```
Monitor> sorttable
Do you want to Sort a table by column (response n if you do not want to edit) (Y/n)?
Enter Table name : Namespace
table name= Namespace
Table sorted by Column:  Avail Pct
And it is sorted in Ascending order
Sort Evicted Objects (no response to next) (y/N)?
Sort Used Disk % (no response to next) (y/N)?
Sort Used Mem % (no response to next) (y/N)?
Sort ip/namespace (no response to next) (y/N)? y
Which order to sort ip/namespace Ascending or Descending ?(default is Descending) (a/D)?
Table sorted by Column:  ip/namespace
And it is sorted in Descending order
Monitor> info Namespace
Total(unique) objects in cluster for largecluster-320 : 1808.701 M
Total(unique) objects in cluster for largecluster-mem : 64.349 M
 
Sorting by IP, in Ascending order:
 
 === NAMESPACE ===
ip/namespace                        Avail       Evicted     Objects     Repl     Stop       Used   Used      Used   Used    hwm   hwm
                                      Pct       Objects           .   Factor   Writes       Disk   Disk       Mem    Mem   Disk   Mem
                                        .             .           .        .        .          .      %         .      %      .     .
192.168.120.109/largecluster-320       56   654,830,152   150.653 M        2    false   143.67 G     33   17.96 G     60     50    60
192.168.120.109/largecluster-mem       96             0     5.206 M        2    false     5.84 G      3    2.26 G      8     50    60
192.168.120.110/largecluster-320       56   600,998,189   150.623 M        2    false   143.65 G     33   17.96 G     60     50    60
192.168.120.110/largecluster-mem       96             0     5.372 M        2    false     5.95 G      3    2.29 G      8     50    60
192.168.120.111/largecluster-320       56   491,716,801   150.854 M        2    false   143.87 G     33   17.98 G     60     50    60
192.168.120.111/largecluster-mem       96             0     5.745 M        2    false     6.35 G      4    2.44 G      9     50    60

No. of rows: 6
Monitor>
```
