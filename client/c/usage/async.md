---
title: Asynchronous Tutorial
description: Asynchronous C client tutorial.
---

The Aerospike C client supports asynchronous event driven interfaces in addition to the existing synchronous interfaces. This document explains how to get started writing programs using async interfaces and event loops.

### Install Event Framework

The Aerospike C client supports the following event frameworks:

- [libev](http://dist.schmorp.de/libev) 4.20 and above.
- [libuv](http://docs.libuv.org) 1.7.5 and above.
- [libevent](http://libevent.org) 2.0.22 and above.

The event framework must be installed independently of the C client.  For this tutorial, we will use libev 4.20.

```bash
tar xvf libev-4.20.tar.gz
cd libev-4.20
./configure
make
sudo make install
```

### Install Aerospike C Client

Install the [C client](https://www.aerospike.com/download/client/c) package built for use with libev. For this document, we will use the Redhat 6 Aerospike C client 4.0.2 package and assume that prerequisite C development tools have been installed.

```bash
tar xvf aerospike-client-c-4.0.2.el6.x86_64.tgz
cd aerospike-client-c-4.0.2.el6.x86_64
sudo rpm -i aerospike-client-c-libev-devel-4.0.2-1.el6.x86_64.rpm
```

## Tutorial

This tutorial demonstrates how to read/write large batches using async programming interfaces.  The source code is located on [GitHub](https://github.com/aerospike/async-c-tutorial).

### Initialize Event Loops

Async event loops are separate threads deadicated to registering and notifying the caller of nextwork events.  This includes socket writeable, socket readable and timer events.  Event loop(s) must be defined before connecting to the cluster.  If your program does not utilize its own event loop(s), they can be created with `as_event_create_loops()`.

```cpp
// Create one event loop
if (! as_event_create_loops(1)) {
    printf("Failed to create event loop\n");
    return -1;
}
```

Alternatively, the app can create its own event loops and share them with the C client.  If your program already uses its own event loop(s), then it is beneficial to share them with the C client.

```cpp
typedef struct {
    pthread_t thread;
    struct ev_loop* ev_loop;
    as_event_loop* as_loop;
} loop;

static bool
share_event_loops(loop* loops, uint32_t loop_count)
{
    // Tell C client the maximum number of event loops that will be shared.
    if (! as_event_set_external_loop_capacity(loop_count)) {
        return false;
    }
    
    // Initialize monitor.
    as_monitor_init(&share_loops_monitor);
    bool status = true;
    
    for (uint32_t i = 0; i < loop_count; i++) {
        loop* loop = &loops[i];
        
        // Start monitor.
        as_monitor_begin(&share_loops_monitor);
        
        // Create event loop thread that will be shared.
        if (pthread_create(&loop->thread, NULL, loop_thread, loop) != 0) {
            status = false;
            break;
        }
        
        // Wait till event loop has been initialized.
        as_monitor_wait(&share_loops_monitor);
    }
    as_monitor_destroy(&share_loops_monitor);
    return status;
}

static void*
loop_thread(void* udata)
{
    // Create external loop.
    loop* loop = udata;
    loop->ev_loop = ev_loop_new(EVFLAG_AUTO);

    // Share event loop with C client.
    // This must be done in event loop thread.
    loop->as_loop = as_event_set_external_loop(loop->ev_loop);

    // Notify parent thread that external loop has been initialized.
    as_monitor_notify(&share_loops_monitor);

    ev_loop(loop->ev_loop, 0);
    ev_loop_destroy(loop->ev_loop);
    return NULL;
}
```

### Connect To Cluster

When connecting to the cluster, it's important to set a limit on the number of async connections allowed for each node.  Each async command consumes resources including memory and a socket connection.  Since the async framework does not block, it's possible to place commands on the queue at a significantly higher rate than the rate the commands can be processed.  Left unchecked, memory and sockets can be exhausted.  

```cpp
as_config cfg;
as_config_init(&cfg);
as_config_add_host(&cfg, host, port);
cfg.async_max_conns_per_node = 200;
aerospike_init(&as, &cfg);

as_error err;
if (aerospike_connect(&as, &err) != AEROSPIKE_OK) {
    printf("Failed to connect to cluster\n");
    aerospike_destroy(&as);
    as_event_close_loops();
    return -1;
}
```

These connection limits are distributed across the number of event loops.  If there are 200 max connections per node with 2 event loops, then the max connections per node per event loop is 100.  If there are no more connections available, the command will fail and an error will be returned.  It's up to user to back-off placing new async commands on the queue if these connection errors occur.  Code to handle resource exhaustion errors is difficult, so the tutorial app avoids such errors by limiting the number of in-flight commands.

### Write Records

An async command goes through several steps:

    app ➔ client ➔ network ➔ node ➔ network ➔ client ➔ app

You can think of these steps as stages in a pipeline, and as with any pipeline, you will get the most throughput when all the components of the pipeline are
kept busy; some of the components can be overloaded if you apply too much pressure.

The tutorial app writes 5,000 records. If it were to wait for each write to complete before starting the next one, the app would not be keeping more than
one component of the pipeline busy at a time. The app can get a lot more throughput by continually feeding overlapping commands into the pipeline so that all components always have something to do.

#### Write Records Using Regular Async

Records can be written by regular async or pipeline commands.  Regular async commands use an exclusive connection pulled from an async connection pool.  To prevent the async queue from being overloaded, a concurrent max command limit (`counter->queue_size`) is enforced.  `queue_size`records are first written to the same event loop.

```cpp
static void
write_records_async(counter* counter)
{
    // Use same event loop for all records.
    as_event_loop* event_loop = as_event_loop_get();
    
    // Write queue_size commands on the async queue.
    for (uint32_t i = 0; i < counter->queue_size; i++) {
        if (! write_record(event_loop, counter)) {
            break;
        }
    }
}
```

The regular async write listener then issues a new command for every command response until all records are written.  This effectively limits the number of async connections used to `queue_size`.

```cpp
static void
write_listener(as_error* err, void* udata, as_event_loop* event_loop)
{
    ...
    // Atomic increment is not necessary since only one event loop is used.
    if (++counter->count == counter->max) {
        // We have reached total records.
        printf("Wrote %u records\n", counter->count);
        
        // Records can now be read in a batch.
        batch_read(event_loop, counter->max);
        return;
    }

    // Check if we need to write another record.
    if (counter->next_id < counter->max) {
        write_record(event_loop, counter);
    }
}
```

All async listener callbacks include the event loop the command was run on as an argument.  This makes it easy for the user to issue new commands on the same event loop thread.  Single threaded callback code can then be optimized with standard non-atomic operators.

We arrived at a queue size of 100 by starting with a lower value, testing with increasing values until there were timeout errors, then backing off. If you have a different network latency, more nodes, more flash devices per node, or other differences from our test setup, you could arrive at a different number.

#### Write Records Using Pipeline

Pipeline commands share connections pulled from a pipeline connection pool.  Pipeline commands:

- Allow single record commands (record put, get, operate...) to be batched together on shared connections.
- Can increase overall performance.
- Are slightly more complicated to use.
- Only applicable for single record commmands, since batch read/scan/query commands already handle large record blocks.

Pipeline writes are initiated with a single write.

```cpp
static void
write_records_pipeline(counter* counter)
{
    // Write a single record to start pipeline.
    // More records will be written in pipeline_listener to fill pipeline queue.
    // A NULL event_loop indicates that an event loop will be chosen round-robin.
    counter->pipe_count++;
    write_record(NULL, counter);
}
```

Pipeline commands are distinguished by an additional `as_pipe_listener` function callback passed to async methods.  Regular async commands pass in NULL for the pipe listener.

```cpp
static bool
write_record(as_event_loop* event_loop, counter* counter)
{
    int64_t id = counter->next_id++;
    
    as_key key;
    as_key_init_int64(&key, g_namespace, g_set, id);
    ...
    aerospike_key_put_async(&as, &err, NULL, &key, &rec, write_listener, counter, event_loop, counter->pipe_listener);
}
```

The pipeline listener is called when the command's socket send has completed.  Additional pipeline commands can be nested in this callback.  Even though connections are shared, a pipeline should still have limits because this pipeline is recursive and the memory/stack could be exhausted in extreme cases.  Therefore, a pipeline `queue_size` (1000) is defined that is larger than the regular async `queue_size` (100).  Also, a separate `pipe_count` is used to track how many commands are in the current pipeline.

```cpp
static void
pipeline_listener(void* udata, as_event_loop* event_loop)
{
    counter* counter = udata;
    
    // Check if pipeline has space.
    if (counter->pipe_count < counter->queue_size && counter->next_id < counter->max) {
        // Issue another write.
        counter->pipe_count++;
        write_record(event_loop, counter);
    }
}
```

The pipeline write listener also issues a new command for every write response until all records are written.  The separate `pipe_count` is decremented when no more records need to be written. 

```cpp
static void
write_listener(as_error* err, void* udata, as_event_loop* event_loop)
{
    ...
    if (++counter->count == counter->max) {
        // We have reached total records.
        printf("Wrote %u records\n", counter->count);

        // Records can now be read in a batch.
        batch_read(event_loop, counter->max);
        return;
    }

    // Check if we need to write another record.
    if (counter->next_id < counter->max) {
        write_record(event_loop, counter);
    }
    else {
        if (counter->pipe_listener) {
            // There's one fewer command in the pipeline.
            counter->pipe_count--;
        }
    }
}
```

### Read Records With Async Batch

Async batch read is used to read the records just written in one call.

```cpp
static void
batch_read(as_event_loop* event_loop, uint32_t max_records)
{
    // Make a batch of all the keys we inserted.
    as_batch_read_records* records = as_batch_read_create(max_records);

    for (uint32_t i = 0; i < max_records; i++) {
        as_batch_read_record* record = as_batch_read_reserve(records);
        as_key_init_int64(&record->key, g_namespace, g_set, (int64_t)i);
        record->read_all_bins = true;
    }

    // Read these keys.
    as_error err;
    if (aerospike_batch_read_async(&as, &err, NULL, records, batch_listener, NULL, event_loop) != AEROSPIKE_OK) {
        batch_listener(&err, records, NULL, event_loop);
    }
}

static void
batch_listener(as_error* err, as_batch_read_records* records, void* udata, as_event_loop* event_loop)
{
    if (err) {
        printf("aerospike_batch_read_async() returned %d - %s\n", err->code, err->message);
        as_batch_read_destroy(records);
        as_monitor_notify(&app_complete_monitor);
        return;
    }

    as_vector* list = &records->list;

    uint32_t n_found = 0;

    for (uint32_t i = 0; i < list->size; i++) {
        as_batch_read_record* record = as_vector_get(list, i);

        if (record->result == AEROSPIKE_OK) {
            n_found++;
        }
        else if (record->result == AEROSPIKE_ERR_RECORD_NOT_FOUND) {
            // The transaction succeeded but the record doesn't exist.
            printf("AEROSPIKE_ERR_RECORD_NOT_FOUND\n");
        }
        else {
            // The transaction failed.
            printf("Error %d\n", record->result);
        }
    }

    printf("Found %u/%u records\n", n_found, list->size);
    as_batch_read_destroy(records);
    as_monitor_notify(&app_complete_monitor);
}
```

### Shutdown Event Loops

Event loop(s) should be shutdown after closing the client’s connection to the cluster.

```cpp
aerospike_close(&as, &err);
aerospike_destroy(&as);
as_event_close_loops();
```

`aerospike_close()` waits for all async commands issued on that cluster to finish.  `as_event_close_loops()` releases resources associated with the event loop(s).

### Async Functionality

Async programming interfaces are available for the following commands:

- Single record (put, get, operate, apply, …)
- Batch
- Scan
- Query (Except for lua aggregation queries)
