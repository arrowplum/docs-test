---
title: Getting Started
description: Design and implement a Rust application with Aerospike as the database.
styles:
  - /assets/styles/ui/steps.css
---

{{#steps}}

{{!---------------------------------------------------------------------------
  - STEP - INSTALL
  ----------------------------------------------------------------------------}}
{{#steps-step 1 "Install Client" markdown=true}}

Before you begin, install the Aerospike Rust client library, which includes the source code.

  {{steps-action "Install Rust Client Library" "/docs/client/rust/install"}}

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
  - STEP – BEGIN DEVELOPING
  ----------------------------------------------------------------------------}}
{{#steps-step 3 "Begin Developing" markdown=true}}

Read the following sections to understand how to:
- [Connect](/docs/client/rust/usage/connect) to an Aerospike cluster. 
- [Read](/docs/client/rust/usage/kvs/read.html) and [write](/docs/client/rust/usage/kvs/write.html) a user record.
- Create a [secondary index](/docs/client/rust/usage/query/sindex.html).
- Run [user-defined functions](/docs/client/rust/usage/udf).

  {{steps-action "Rust Client Library" "/docs/client/rust/usage/connect"}}

{{/steps-step}}
{{/steps}}
