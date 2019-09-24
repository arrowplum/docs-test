---
title: Getting Started
description: Design and implement a Twitter-like application written with Aerospike as the only database.
styles:
  - /assets/styles/ui/steps.css
---

{{#steps}}

{{!---------------------------------------------------------------------------
  - STEP - INSTALL
  ----------------------------------------------------------------------------}}
{{#steps-step 1 "Install Client" markdown=true}}

Before you begin, install the Aerospike Java client library, which includes source code, examples, and the benchmark tool.

  {{steps-action "Install Java Client Library" "/docs/client/java/install"}}

{{#if book.vars.github-url}}

---

You can also start from **<a id="github" href="{{book.vars.github-url}}">GitHub <span class="fa fa-github" style="font-size: 1.5em"></span></a>**
{{/if}}

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - SETUP SERVER
  ----------------------------------------------------------------------------}}
{{#steps-step 2 "Setup Aerospike Server" markdown=true}}

Download and install the Aerospike Server.

  {{steps-action "Install Aerospike Server" "/download/server" }}

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - RUN EXAMPLES
  ----------------------------------------------------------------------------}}

{{#steps-step 3 "Run Examples" markdown=true}}

Run a few of the provided examples.
  {{steps-action "Examples" "https://github.com/aerospike/aerospike-client-java/tree/master/examples" }}

{{/steps-step}}

{{!---------------------------------------------------------------------------
  - STEP - RUN BENCHMARK
  ----------------------------------------------------------------------------}}

{{#steps-step 4 "Run Benchmarks" markdown=true}}

Use the Benchmarks tool to run a default load against the cluster.

 {{steps-action "Benchmarks" "/docs/client/java/benchmarks.html" }}

{{/steps-step}}

{{!---------------------------------------------------------------------------
  - STEP – BEGIN DEVELOPING
  ----------------------------------------------------------------------------}}
{{#steps-step 5 "Begin Developing" markdown=true}}

Read the following sections to understand how to:
- [Connect](/docs/client/java/usage/connect) to an Aerospike cluster. 
- [Read](/docs/client/java/usage/kvs/read.html) and [write](/docs/client/java/usage/kvs/write.html) a user record.
- Create a [secondary index](/docs/client/java/usage/query/sindex.html).
- Run [user-defined functions](/docs/client/java/usage/udf).


  {{steps-action "Java Client Library" "/docs/client/java/usage/connect"}}

---

Read [Best Practices](/docs/client/java/usage/best_practices.html) to best utilize the client library.

  {{steps-action "Best Practices" "/docs/client/java/usage/best_practices.html"}}

---

See the [API reference]({{book.vars.api-url}}).

  {{steps-action "API Reference" book.vars.api-url}}

{{/steps-step}}
{{/steps}}
