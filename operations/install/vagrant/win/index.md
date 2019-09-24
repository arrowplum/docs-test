---
title: Install on Windows
description: Get an Aerospike server up and running on Windows using a Vagrant box.
styles:
  - /assets/styles/ui/steps.css
steps:
  next_steps: 6
---

### Overview

Aerospike is supported on Windows in a virtual machine managed by Vagrant.
See **[Using Vagrant](/docs/operations/install/vagrant/win/using-vagrant.html)** for an overview and
installation instructions.

{{#note}}
Aerospike's Community Edition is configured to **transmit anonymous usage statistics**.
We ask your help in making Aerospike better by leaving this feature enabled.
You can learn about our goals, how we use the data, and how to disable the feature [here](/aerospike-telemetry).
{{/note}}

{{#note}}
The rest of this document assumes that you have both Vagrant and a
virtualization system installed.
{{/note}}

{{#steps}}
{{#steps-step 1 "Install Git for Windows" markdown=true}}

Git for Windows provides a simple UNIX shell emulator called **Git Bash**, which is needed for successfully using Vagrant.

{{steps-action "Install Git for Windows" "http://msysgit.github.io/" }}

{{/steps-step}}
{{/steps}}

{{#note markdown=true}}
The remaining portions of this document will request commands be entered into the **command prompt**, which simply requires you to run **Git Bash** from the directory specified.
{{/note}}

{{md (path-resolve (path-dirname page.src) "download.part.md") }}

{{md (path-resolve (path-dirname page.src) "run.part.md") }}

{{#note}}
The Vagrant Aerospike install comes with three pre-configured clients: Java, Python, Node.js, and C.
{{/note}}

{{md (path-resolve (path-dirname page.src) "../../common/nextsteps.part.md") }}

