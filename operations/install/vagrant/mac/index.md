---
title: Install on OS X
description: Get an Aerospike server up and running on OS X using a Vagrant box.
styles:
  - /assets/styles/ui/steps.css
amc_installed: true
steps:
  next_steps: 5
---

### Overview

Aerospike is supported on OS X in a virtual machine managed by Vagrant.
See **[Using Vagrant](/docs/operations/install/vagrant/mac/using-vagrant.html)** for an overview and
installation instructions.

{{#info}}
Aerospike's Community Edition is configured to **transmit anonymous usage statistics**.
We ask your help in making Aerospike better by leaving this feature enabled.
You can learn about our goals, how we use the data, and how to disable the feature [here](/aerospike-telemetry).
{{/info}}

{{#note}}
The rest of this document assumes that you have both Vagrant and a
virtualization system installed.
{{/note}}

{{md (path-resolve (path-dirname page.src) "download.part.md") }}

{{md (path-resolve (path-dirname page.src) "run.part.md") }}

{{#note}}
The Vagrant Aerospike install comes with three pre-configured clients: Java, Python, Node.js, and C.
{{/note}}

{{md (path-resolve (path-dirname page.src) "../../common/nextsteps.part.md") }}

