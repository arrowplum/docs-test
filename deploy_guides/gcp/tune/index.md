---
title: Performance Tuning on Google Compute Engine
description: |
  Tuning and tweaking parameters for performance on Google Compute Engine with Aerospike database
scripts:
  - /assets/scripts/utils/target_blank.js

persistence-name: "Persistent Disks"
cloud-provider: "Google Compute Engine"

---

#### Pinning Aerospike Server on Certain vCPUs (taskset)

Aerospike is extremely fast in its in-process operations of getting and setting data items. Indeed, the CPU work done by the Operating System to deal with the TCP packet is comparable to the work done by the Aerospike database engines. To this end, it is beneficial to dedicate vCPUs (virtual CPUs) to handle just the network IRQs. One technique we have found helpful in this case is to leave some vCPUs of the virtual machine totally free. This may take away some CPU power from the application, but it helps improve the network latencies. To leave a vCPU free, use the following command to utilize vCPU (0-6) for Aerospike on an 8-core box:

```bash
sudo taskset -c 0-6 /usr/bin/asd --config-file /etc/aerospike/aerospike.conf
```

{{#note}}
You may have to change the `init` script to use the `taskset` command.
{{/note}}

{{!-- Both taskset and RPS not required cannot be true. Ask Sunil. IMO, note about RPC should be removed

#### Receive Packet Steering

On some environments, network interrupts are handled by a few CPU cores of a machine. But, Google Compute Engine uses kernel-base Virtual Machine (KVM) for virtualization which uses [**virtio**](http://wiki.libvirt.org/page/Virtio) for IO Virtualization including the network. We have observed that the network interrupts are distributed evenly across all the CPU cores, thanks to virtio. This can be confirmed by observing the number of interrupts handled by each CPU core during a period of high network activity.
```bash
cat /proc/interrupts | grep virtio
```
So, the techniques used to mitigate the CPU core bottleneck on network handling, like Receive Packet Steering (RPS), are not necessary when running in the Google Compute Engine.

--}}
