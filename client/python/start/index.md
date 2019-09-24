---
title: Getting Started
description: Use the Aerospike Python client to build applications to store and retrieve data from the Aerospike database.
styles:
  - /assets/styles/ui/steps.css
---

Use the Aerospike Python client to build applications to store and retrieve data from the Aerospike database.

{{#steps}}

{{!---------------------------------------------------------------------------
  - STEP - INSTALL
  ----------------------------------------------------------------------------}}
{{#steps-step 1 "Install Python Client" markdown=true}}

Install the Aerospike Python client, which includes the source, libraries, examples, and other resources.


  {{steps-action "Install Python Client" "/docs/client/python/install"}}

{{/steps-step}}


{{!---------------------------------------------------------------------------
  - STEP - SETUP SERVER
  ----------------------------------------------------------------------------}}
{{#steps-step 2 "Install Aerospike Server" markdown=true}}

Install the Aerospike server.

  {{steps-action "Install Aerospike Server" "/download/server"}}

{{/steps-step}}

{{!---------------------------------------------------------------------------
  - STEP - Python Client Library
  ----------------------------------------------------------------------------}}
{{#steps-step 3 "Python Client Library" markdown=true}}

Create a client object, create a record, and write, retrieve and delete a record.

{{steps-action "Python Client Library" "/docs/client/python/usage"}}

{{/steps-step}}

{{!---------------------------------------------------------------------------
  - STEP - PHP API Reference
  ----------------------------------------------------------------------------}}

{{#steps-step 4 "Python API Reference" markdown=true}}

The Python API reference is available online.

{{steps-action "Python API Reference" "/apidocs/python"}}

{{/steps-step}}

{{/steps}}

