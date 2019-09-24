---
title: Node Problems
---
 
### Out of Memory
If a crash has occurred without a stack trace, run dmesg to find out if asd was killed due to out of memory (OOM). In a typical scenario, the clients are writing too fast for too long. This can eventually cause a memory crash if the eviction rate isn’t set high enough. If you were experiencing a short term flood of traffic, the problem should correct itself once the evictions catch up. However, if your traffic has simply been growing over a long period of time, you should do a capacity check to see if you should increase the amount of memory in your nodes or the number of nodes in your cluster.

Also inspect */var/log/messages* to see any relevant information.

In addition, get the machine’s uptime to confirm that the server did not go into an unexpected/undetected reboot. This has been known to happen, particularly in cloud environments.

