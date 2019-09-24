{{!-- This page is also used at /docs/operations/aws/--}}
{{!-- This page is also used at /docs/operations/gcp/--}}

### Cloud Considerations
1. **Lack of Multicast Support**:
Cloud providers (e.g. Amazon, Google Compute Engine) do not support multicast
networking. For these providers, we offer
[Mesh heartbeats](/docs/operations/configure/network/heartbeat/index.html#mesh-unicast-heartbeat),
which uses point-to-point TCP connections for heartbeats.

2. **Network Variability**:
Often, the network latency on cloud platforms is not consistent over time. This
can cause problems with heartbeat packet delivery times. For these providers,
we recommend setting the heartbeat `interval` to 150 and the heartbeat `timeout`
to 20.

3. **Instance Pauses**:
At times, your cloud instance could be paused by the cloud provider for short durations.
For example GCE employs
[live migration](http://googlecloudplatform.blogspot.in/2015/03/Google-Compute-Engine-uses-Live-Migration-technology-to-service-infrastructure-without-application-downtime.html)
which could pause your instance for short durations of time for maintenance or software updates.
These short pauses might cause the other instances in the cluster to consider this instance as "dead"
for the pause duration. We therefore recommend considering upgrades to server versions 3.8.1 and above which have the
[paxos-recovery-policy](/docs/reference/configuration/index.html#paxos-recovery-policy) default to `auto-reset-master`
to help your cluster recover quickly after any network disruption or cluster change.
This policy has been introduced with Aerospike server version 3.7.0.1 but needs explicit configuration to `auto-reset-master` until version 3.8.1.
