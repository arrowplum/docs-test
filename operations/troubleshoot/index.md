---
title: Troubleshoot
---

{{#note}}
Make sure to consult and search over the [Knowledge-Base](https://discuss.aerospike.com/c/knowledge-base) topics as a wide variety of issues and remediations 
are covered there.
{{/note}}

 
### Diagnosing Problems

{{#todo markdown=true}}
- latency link is not the final resting place for latency... need better link
{{/todo}}

Whenever you need to diagnose system problems (and before contacting Aerospike Technical Support), we recommend that you review the following points:

- If you are having problems in [starting up a server](/docs/operations/troubleshoot/startup), check the server logs. The logs usually mention the reason for startup failure.
- If the servers are already [running](/docs/operations/troubleshoot/node), check system information by running [asadm](/docs/tools/asadm) and using the `info` command. Ensure none of the statistics are in red.
-  Check for [cluster problems](/docs/operations/troubleshoot/cluster):
	- Check the system [latency](/docs/operations/monitor/latency)
	- Review the [log files](/docs/operations/manage/log) on each node to see if there are warning or error messages.
- Check your [client](/docs/operations/troubleshoot/client) machines (application servers and application logs).
- Check the network (switches/routers/firewalls/load balancers) 
- Some problems could be [data](/docs/operations/troubleshoot/misc) related and need to be corrected on application side.
- Search through our Knowledge-Base reference guide if the issue observed has been previously reported - [Knowledge-Base Reference](https://discuss.aerospike.com/c/knowledge-base).

The troubleshooting pages in this section of the documentation provide details for how to address some common problems you may encounter when running the Aerospike software.

### Check for Aerospike Software Upgrades
After you figure out the problem, be sure to check the release notes to see if a newer version of Aerospike Database resolves your issue.  If you need to upgrade your server software, you can do a hot upgrade using rolling upgrade procedure described [here](/docs/operations/upgrade/aerospike).

In the troubleshooting pages, we assume that you know how to do the following standard tasks with Aerospike Database:

- How to look at the Aerospike log (by default it's in `/var/log/aerospike/aerospike.log` or you can [locate the log file location)]({{book.baseurl}}/operations/configure/log/#find-existing-log-location).
- How to [update parameters in the configuration file](/docs/operations/configure) and how to use asinfo to [read](/docs/tools/asinfo/index.html#get-config) and [write](/docs/tools/asinfo/index.html#set-config) parameters dynamically.
- How to use the [asadm tool](/docs/tools/asadm/)
- How to check [latency](/docs/tools/asloglatency)
- How to [start and stop asd](/docs/operations/manage/aerospike) (the main Aerospike process) 
- How to do a [cold](/docs/operations/manage/operations/cold_start) or a [fast](/docs/operations/manage/aerospike/fast_start) restart

