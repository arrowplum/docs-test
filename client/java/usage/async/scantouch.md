---
title: Scan with Async Touch
description: Scan with Async Touch
---

This example demonstrates how to perform async commands resulting from a full scan of the cluster.

Scans return a tremendous amount of data to the client at a very high rate of speed.  It's important to throttle down scan throughput to give the resulting async commands a chance to complete before the async queue is overloaded.  Throttling async scans is generally a bad idea because it usually involves blocking the event loop the scan is running on.  Synchronous scans are a better choice in this case because the throttled scan thread is not shared by any other commands.

The following example performs an async touch for every record returned from a scan.  A throttle is added that sets a max concurrent commands for each event loop.  The async touch is distributed across event loops.  The sync scan thread will block if the event loop's max concurrent commands is reached.  The scan thread will resume when a slot becomes available.

```java
import io.netty.channel.EventLoopGroup;
import io.netty.channel.epoll.EpollEventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;

import java.util.concurrent.atomic.AtomicLong;

import com.aerospike.client.AerospikeClient;
import com.aerospike.client.AerospikeException;
import com.aerospike.client.Key;
import com.aerospike.client.Record;
import com.aerospike.client.ScanCallback;
import com.aerospike.client.async.EventLoop;
import com.aerospike.client.async.EventLoops;
import com.aerospike.client.async.EventPolicy;
import com.aerospike.client.async.NettyEventLoops;
import com.aerospike.client.async.NioEventLoops;
import com.aerospike.client.async.Throttles;
import com.aerospike.client.listener.WriteListener;
import com.aerospike.client.policy.ClientPolicy;
import com.aerospike.client.policy.ScanPolicy;
import com.aerospike.client.policy.WritePolicy;

public class SyncScanAsyncTouch implements ScanCallback {
    private static final String HostName = "localhost";
    private static final String Namespace = "test";
    private static final String SetName = "test";
    private static final int EventLoopSize = 4;  // Adjust to your machine's capability.
    private static final int CommandsPerEventLoop = 40;

    public static void main(String[] args) {
        try {
            SyncScanAsyncTouch test = new SyncScanAsyncTouch();
            test.run();
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }

    private AerospikeClient client;
    private EventLoops eventLoops;
    private Throttles throttle;
    private final AtomicLong touchCount;
    private long scanCount;

    public SyncScanAsyncTouch() {
        touchCount = new AtomicLong();
    }

    public void run() {
        EventPolicy eventPolicy = new EventPolicy();

        // Direct NIO
        eventLoops = new NioEventLoops(eventPolicy, EventLoopSize);
        
        // Netty NIO
        // EventLoopGroup group = new NioEventLoopGroup(EventLoopSize);             
        // eventLoops = new NettyEventLoops(eventPolicy, group);
        
        // Netty epoll (Linux only)
        // EventLoopGroup group = new EpollEventLoopGroup(EventLoopSize);               
        // eventLoops = new NettyEventLoops(eventPolicy, group);
        
        try {
            throttle = new Throttles(EventLoopSize, CommandsPerEventLoop);
            
            ClientPolicy clientPolicy = new ClientPolicy();
            clientPolicy.eventLoops = eventLoops;           
            client = new AerospikeClient(clientPolicy, HostName, 3000);
            
            try {
                ScanPolicy policy = new ScanPolicy();
                policy.concurrentNodes = false;
                policy.includeBinData = false;    
                client.scanAll(policy, Namespace, SetName, this);
                System.out.println("Scan count: " + scanCount);             
            }
            finally {
                // The latest client waits for pending async commands to finish
                // before performing the actual close, so there is no need
                // to externally track pending async commands.
                client.close();
                System.out.println("Touch count: " + touchCount.get());
            }
        }
        finally {
            eventLoops.close();
        }
    }

    @Override
    public void scanCallback(Key key, Record record) {
        // Since concurrentNodes is false, atomic increment is not necessary.
        scanCount++;
        
        WritePolicy policy = new WritePolicy();
        EventLoop eventLoop = eventLoops.next();
        int index = eventLoop.getIndex();
        
        if (throttle.waitForSlot(index, 1)) {
            try {
                client.touch(eventLoop, new WriteHandler(key, index), policy, key);
            }
            catch (Exception e) {
                throttle.addSlot(index, 1);
            }
        }
    }
 
    private final class WriteHandler implements WriteListener {
        private final Key key;
        private final int index;
        
        public WriteHandler(Key key, int index) {
            this.key = key;
            this.index = index;
        }
        
        @Override
        public void onSuccess(Key key) {
            throttle.addSlot(index, 1);
            touchCount.getAndIncrement();
        }

        @Override
        public void onFailure(AerospikeException e) {
            throttle.addSlot(index, 1);
            System.out.println("Touch failed for " + key + ": " + e.getMessage());
        }
    }
}
```
