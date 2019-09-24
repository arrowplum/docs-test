---
title: Getting Started
description: Use the Aerospike libevent client to write applications to store and retrieve data from an Aerospike database cluster.
breadcrumbs:
  - title: Libevent Client
    url: /docs/client/libvent
categories:
  - aerospike-client-libevent
tags:
  - aerospike-client-libevent
  - c
styles:
  - /assets/styles/ui/steps.css
---

{{#warn}}
Aerospike deprecated our C libevent Client Library.
<BR>
Please use the standard **[C Client](https://www.aerospike.com/download/client/c/)**, which supports asynchronous programming models.
{{/warn}}

{{#steps}}

{{!---------------------------------------------------------------------------
  - STEP - Build Client
  ----------------------------------------------------------------------------}}
{{#steps-step 1 "Build Client" markdown=true}}

Download and build the Aerospike libevent client library. The download package includes source code, header files, example files, and other resources.

  {{steps-action "Build libevent Client Library" "/docs/client/libevent/build"}}

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - SETUP SERVER
  ----------------------------------------------------------------------------}}
{{#steps-step 2 "Setup Aerospike Server" markdown=true}}

Install the Aerospike server.

  {{steps-action "Install Aerospike Server" "/download/server"}}
{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - RUN EXAMPLES
  ----------------------------------------------------------------------------}}
{{#steps-step 3 "Run Examples" markdown=true}}

Run the examples. In the various examples directories at the top of each main.c, there is a description of the test.
{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP – BEGIN DEVELOPING
  ----------------------------------------------------------------------------}}
{{#steps-step 4 "Begin Developing" markdown=true}}

Read the following topics to understand how to:
- Use the event base and threads to interact with the Aerospike libevent client.
- Connect to an Aerospike cluster.
- Read and write a single record.


  {{steps-action "libevent Client Library" "/docs/client/libevent/usage/eventbase.html"}}

{{/steps-step}}
{{/steps}}

