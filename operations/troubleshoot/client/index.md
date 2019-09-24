---
title: Client Errors
---
 
### Client Error Codes

Individual client error codes are provided in each client's API references:

Example:
- Java Client Error Codes: [Result Codes](/apidocs/java/constant-values.html#com.aerospike.client.ResultCode.NO_XDS)
- C# Client Error Codes: [Result Codes](/apidocs/csharp/html/Fields_T_Aerospike_Client_ResultCode.htm)

### Client receiving Server memory errors

When the node reaches a stop write or the node’s device is not able to catch up with writing to disk, the client will receive server memory errors. Errors will vary depending on your client. Example, for Java client, it is seen as Error Code 8 SERVER_MEM_ERROR.

- Verify that your cluster is running the latest OS firmware and there’s no general disk failure.
- Run iostat to get your server’s CPU and I/O stats.
- Check the cluster’s disk and memory usage using the monitoring tool (asadm) to see if it’s higher than the stop-writes percentage. If it is, your namespace may be running out of space on that node.
- Check the `queue` delay values on every node and watch if it’s increasing:

```
$ asinfo -v "statistics" -l | grep queue
```
or
```
$ asadm
$ asadm > watch 2 show statistics like queue
```

- Check the log file `/var/log/aerospike/aerospike.log` for "write-q" and check if any objects are stuck in the queue waiting.

In addition, you can address the server errors by setting the Stop Writes level or by deleting or evicting the objects faster.

Once you have adjusted the settings, continue monitoring the cluster and the write latency statistics. You also need to verify that the used memory bytes is decreasing as more objects are getting evicted:

```
$ asadm > watch 2 show statistics like memory
```

#### Client is logging KEY_BUSY result code messages
- This error is most likely caused by a ‘hotkey’ and will affect the clients doing reads or writes.
- The server node received too many concurrent operation requests for the same key. When this happens, the server rejects the request and returns KEY_BUSY.
- The server also increments the “fail_key_busy” count statistic. You can check for this on your server node by:

```
$ asadm
$ asadm > watch 2 show statistics like fail_key_busy
```

- If you want to reduce the number of errors for now, you can increase the pending limit configuration on all nodes or whatever limit you want to set for the hotkey. We don’t recommend increasing this past 9 because that could indicate a real problem. There is no performance effect for increasing the value of transaction-pending-limit.
- This command sets limit to 9 from current limit of 3.

```
$ asadm
$ asadm > asinfo -v "set-config:context=service;transaction-pending-limit=9;"
```

#### Client Request
If a network glitch occurs during a client request to the Aerospike server, the info request from the client will fail. This network issue can be caused by congestion or packet loss. The client will retry every second. If the message “DUNNED NODE” appears in the client log (happiness value becomes 0), this means that the client cannot reach a node in the cluster and will need to reconnect. This is a normal part of the client behavior when there is some interruption talking to the cluster nodes. You will see error messages similar to the following in the Aerospike log (`/var/log/aerospike/aerospike.log`). While alarming, these are generally temporary issues.

```
2012-09-25 09:01:41 EDT AEROSPIKE [143] info: could not connect to abc1.def.domain.com : connect timed out 
2012-09-25 09:01:41 EDT AEROSPIKE [143] node BB9738A8E9B2100 info get failed, decrease happiness abc1.def.domain.com/10.0.0.53:3000
2012-09-25 09:34:56 EDT AEROSPIKE [243] node BB9738A8E9B2100 has valid network activity but not fully happy : 250
```
