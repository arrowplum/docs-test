---
title: Guarantees
assets: /docs/guide/assets
---

## Delivery
As with the regular XDR, the framework will guarantee at-least once delivery.
At least once means that, in some cases, a change will be delivered and
acknowledged, and yet the same change may be notified again. This commonly
happens because master ownership changes, and the information about the write
was not transmitted to the replica.

But there is an important limitation especially with hot keys. The XDR’s
digestlog only stores information about what records changed but not the data.
We do an internal read of the record to ship it. The record might have changed
since the logging due to some other write. So, when we ship a record on behalf
of a digestlog entry, we always read the latest value of the record in the
system and not the version when the digestlog entry was written. This issue
happens not just with hot keys but also when XDR builds up a lag. To summarize,
we ship a monotonically increasing version of the record.

This has an important implication on the receiver side and hence the Change
Notification framework may not apply to all use cases. The receiver should not
expect that every version of the record will be delivered. It should be designed
in such a way that it can handle skipped versions of intermediate records.

For example, if the receiver is observing the number of credit card transactions
to send an e-mail alert if it reaches 100, it should not check for the value to
be exactly 100. Instead, it should check for >= 100.

## Bin-Shipping Optimization
One optimization in XDR is to attempt to ship only the changed bins. If you
update a record, and change only a subset of the bins, only the changed bins
will be delivered. However, in the case of a complex type like a Map, the entire
Map will be transmitted as they are part of single bin.  This change
notification is done through a bloom filter, so you will find in some cases that
both changed bins and some of the unchanged bins will be notified.

## Ordering
Ordering of updates between different records is not guaranteed. As Aerospike
provides single-record consistency, this is to be expected.

Out of order notification, even on a single record, may rarely seem to occur.
This can happen when a retransmit is required. If a second transmission of the
message has been acknowledged, but the initial notification was only delayed by
network issues, the initial notification may arrive at the HTTP server after a
subsequent notification.

## Known issue - missed updates
Testing has shown that an unusual case can lead to a missed update. In cases
where shipping is slower than intra-cluster data migration, a node may be taken
down permanently after data migration between nodes has completed, but before
all notifications are complete. If more than one node is permanently removed in
that manner it may lead to missed updates. Please be aware of this possibility,
and apply Aerospike's recommendations regarding how to guard against node
removal before shipping is complete, especially on cluster maintenance.

## Retransmission
As Aerospike is delivering messages through the HTTP protocol, the receiver must
emit an OK response when the message is successfully consumed. Aerospike waits
for the "OK" response. If the response is not made within a certain time, the
HTTP request TCP connection will be broken and the POST request will be made
again. 

In some cases, this may seem to result in duplicate delivery or out of order
delivery. Please be aware of the tradeoffs of responding with OK early, allowing
greater throughput, vs responding OK when the message has been reliably ingested
and committed to storage.

