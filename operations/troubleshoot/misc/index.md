---
title: Misc Issues
---

This page describes a few miscellaneous issues/problems that may arise.

### How do I alter the write commit level?

For every write before responding to the client, Aerospike will wait for the prole reply (replica write) . You can enable the fire-forget feature, which will return to the client as soon as the master write is successful.

```
$ asinfo -v "set-config:context=namespace;id=<namespaceName>;write-commit-level-override=master" -h [host ip]
```
See documentation on [Per-Transaction Consistency Guarantees](/docs/architechture/consistency) for more details.

### Failures for the same key when accessed concurrently (transaction-pending-limit)

When many threads try to concurrently access (read/write operations) the same key, there will be some failures. This is due to the configuration [transaction-pending-limit](/docs/reference/configuration/#transaction-pending-limit). The server’s transaction queue will include reads/writes coming from all threads to that server.

e.g. If transaction-pending-limit is 9 and there are 5 reads and 5 writes for the same key in the queue, then transaction-pending-limit is considered exceeded and any other operations on that key will fail.

The default is 20. Increase this configuration to accommodate the behavior of your application. The stats variable that tracks these failures is err_rw_pending_limit. If you set transaction-pending-limit to zero, the transaction queue check is not performed.

See Knowledge-Base article on [Hot Key Error Code 14](https://discuss.aerospike.com/t/hot-key-error-code-14/986) for more details.

#### What to do when you get a Stack Trace
At times, you see a stack trace in the log that is not meaningful or helpful. The log might show a stack trace that looks like this:

```
Stack trace (non-dedicated):
 /usr/bin/asd() [0x403928]
 /lib64/libc.so.6() [0x3f8a432750]
 /usr/bin/asd() [0x4102fb]
 /usr/bin/asd() [0x412d35]
 /usr/bin/asd() [0x413a3c]
 /usr/bin/asd() [0x41755b]
 /lib64/libpthread.so.0() [0x3f8ac06a3a]
 /lib64/libc.so.6(clone+0x6d) [0x3f8a4de65d]
 End of stack trace.
 ```
Before Aerospike can help you diagnose the problem, you must run gdb.

If you don't have gdb installed, you must install it:
CentOS:

```
$ sudo yum install gdb
```
Then you must use gdb to analyze the stack trace:

```
$ gdb asd
From the gdb prompt:  info line * [ID numbers from the stack trace in the log]
```
 The output should look like this:

```
 (gdb) info line *0x403928
 Line 105 of "base/as.c" starts at address 0x403928  and ends at 0x40392b .
 (gdb) info line * 0x4102fb
 Line 1387 of "base/thr_rw.c" starts at address 0x4102fb  and ends at 0x41030a .
 (gdb) info line * 0x412d35
 Line 425 of "base/thr_rw.c" starts at address 0x412d35  and ends at 0x412d3b .
 (gdb) info line * 0x413a3c
 Line 693 of "base/thr_rw.c" starts at address 0x413a3c  and ends at 0x413a41 .
 ```

Send the output to Aerospike Tech Support for help resolving this issue.

#### "key field too big 1248. Is it for real?" in log

This occurs when the keys you are using are long. These messages are for informational purposes only and can be ignored. Writes will still continue. No loss of data happens. The same key will still be processed as normal.

```
Oct 11 2012 13:46:02 GMT: INFO (proto): (base/transaction.c:133) key field too big 1248. Is it for real?
Oct 11 2012 13:46:02 GMT: INFO (proto): (base/transaction.c:133) key field too big 1248. Is it for real?
```
