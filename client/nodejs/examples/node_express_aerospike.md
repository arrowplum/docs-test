---
title: Web Application using Node.js, Express, and Aerospike
description: This simple web application uses Node.js and Express to demonstrate using the Aerospike Node.js client with the Aerospike database.
styles:
  - /assets/styles/ui/steps.css
---

This simple web application uses Node.js and Express to demonstrate using the Aerospike Node.js client with the Aerospike database. This web application:

- Connects to the Aerospike database.
- Allows you to enter a name on a web page.
- Stores the name in the Aerospike database.
- Reads the name and displays it on the web page.

### Prerequisites 

- **Aerospike Server** &mdash; Install the Aerospike server from <a href="/docs/operations/install/" target="_blank">here</a>.
- **Node.js** &mdash; Install Node.js from <a href="http://nodejs.org" target="_blank">here</a>. 
    
{{#steps}}


{{!---------------------------------------------------------------------------
  - STEP - Application Setup
  ----------------------------------------------------------------------------}}
{{#steps-step 1 "Application Setup" markdown=true}}

1. **Create a new folder** (for example, _aerospike_express_demo_) for the application.
2. **Initialize a new application,** by running `npm init`. This command prompts for a number of things, such as the name and version of your application. You can accept most of the defaults for now. At the end of the process the command should have created a new _package.json_ file in the application directory.
3. **Install the `aerospike` and `express` packages,** saving the dependencies to the _package.json_ file:
  ```
$ npm install express --save
...
$ npm install aerospike --save
...
  ```
Afterwards, your application's _package.json_ file should look something like this:
  ```
{
    "name": "aerospike_express_demo",
    "version": "1.0.0",
    "description": "aerospike express demo app",
    "main": "index.js",
    "author": "your name here!",
    "license": "ISC",
    "dependencies": {
        "aerospike": "^2.0.0",
        "express": "^4.13.4"
    }
}
  ```
{{/steps-step}}

{{!---------------------------------------------------------------------------
  - STEP - Start Coding
  ----------------------------------------------------------------------------}}
{{#steps-step 2 "Start Coding" markdown=true}}

#### Config Module

1. **Create a new file,** _aerospike_config.js_, and add the following code. Save this file in the application folder.
  ```
var aerospikeClusterIP = '127.0.0.1'
var aerospikeClusterPort = 3000
exports.aerospikeConfig = function () {
  return {
    hosts: [ { addr: aerospikeClusterIP, port: aerospikeClusterPort } ]
  }
}
exports.aerospikeDBParams = function () {
  return {
    defaultNamespace: 'test',
    defaultSet: 'test'
  }
}
  ```
1. **Update the** `aerospikeClusterIP` **and** `aerospikeClusterPort` **variables** to point at the Aerospike server instance.

  This config module exports these functions: 
  - `aerospikeConfig()` encapsulates and exposes the Aerospike **cluster IP** and **port** settings.
  - `aerospikeDBParams()` contains attributes such as namespaces and sets that the API module (see below) uses.

#### API Module

1. **Create a new file,** _api.js_, and add the following code. Save this file in the application folder. 
  ```
const client = Aerospike.client(aerospikeConfig)
// Establish connection to the cluster
exports.connect = function (callback) {
  client.connect(callback)
}
// Write a record
exports.writeRecord = function (k, v, callback) {
  let key = new Aerospike.Key(aerospikeDBParams.defaultNamespace, aerospikeDBParams.defaultSet, k)
  client.put(key, { greet: v }, function (error) {
    // Check for errors
    if (error) {
      // An error occurred
      return callback(error)
    } else {
      return callback(null, 'ok')
    }
  })
}
// Read a record
exports.readRecord = function (k, callback) {
  let key = new Aerospike.Key(aerospikeDBParams.defaultNamespace, aerospikeDBParams.defaultSet, k)
  client.get(key, function (error, record) {
    // Check for errors
    if (error) {
      // An error occurred
      return callback(error)
    } else {
      let bins = record.bins
      return callback(null, k + ' ' + bins.greet)
    }
  })
}
  ```
  This API module exports these functions: 
  - `connect()` encapsulates code related to connecting to the Aerospike server. It accepts a callback function invoked with the connection status code.
  - `writeRecord()` contains the code to write a record to the database. It accepts `k`ey, `v`alue, and `callback` as arguments.
  - `readRecord()` contains the code for reading a record. It accepts `k`ey and `callback` as arguments.

#### App Module

1. **Create a new file,** _index.js_, and add the following code. Save this file in the application folder.
  ```
const express = require('express')
const http = require('http')
const api = require('./api')
const app = express()
let dbStatusCode = 0
// Establish connection to the cluster
api.connect(function (error) {
  if (error) {
    // handle failure
    dbStatusCode = error.code
    console.log('Connection to Aerospike cluster failed!')
  } else {
    // handle success
    console.log('Connection to Aerospike cluster succeeded!')
  }
})
// Setup default/home route
app.get('/', function (req, res) {
  res.send('<div><form action="/write"><label>Enter your name:</label><input type="text" name="name"/><input type="submit"></input></form></div>')
})
// Setup write route
app.get('/write', function (req, res) {
  if (dbStatusCode === 0) {
    api.writeRecord('Hello', req.query.name, function (error, result) {
      if (error) {
        // handle failure
        res.send(error.message)
      } else {
        // handle success
        api.readRecord('Hello', function (error, result) {
          if (error) {
            // handle failure
          } else {
            // handle success
          }
          res.send(result)
        })
      }
    })
  } else {
    res.send('Connection to Aerospike cluster failed!')
  }
})
// Start server
let server = http.Server(app)
server.listen('9000', 'localhost', function () {
  console.log('App is running on http://localhost:9000. Press Ctrl-C to exit...')
})
  ```
 This establishes the connection to the Aerospike server using `api.connect()`, and then sets the default/home route that displays the web form for entering a name. The action executes on form submition and is mapped to the `/write` route which invokes `api.writeRecord()`, and passes it 'Hello' as the _key_ and the name entered as _value_. On a successful write operation, `api.readRecord()` reads the record and displays it on the web page.

{{/steps-step}}
{{!---------------------------------------------------------------------------
  - STEP - Run The Application
  ----------------------------------------------------------------------------}}
{{#steps-step 3 "Run The Application" markdown=true}}

1. Open a terminal window and `cd` **to the application folder and run** `node ./`.

  These messages appear:
  ```
Connection to Aerospike cluster succeeded!
App is running on http://localhost:9000. Press Ctrl-C to exit...
   ```
  If the message **“Connection to Aerospike cluster failed!”** appears:
  - Ensure that the Aerospike server is running and available. 
  - Confirm that `aerospikeClusterIP` and `aerospikeClusterPort` are correctly set in *aerospike_config.js*.
1. In your browser, **enter http://localhost:9000**. 

  A web page displays a form.
1. **Type a name** and click **Submit**.

  `“Hello [name]` appears on the page.

{{/steps-step}}

{{/steps}}
