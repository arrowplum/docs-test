---
title: API
assets: /docs/guide/assets
---

The downloaded package contains a Java implementation of a parser for the
binary message payload sent by the Change Notification framework. The parser is
distributed under the permissive Apache License, Version 2.0. The parsing
library consists of two main components:
1. `RecordParser` class : reads and processes a message from an InputStream.
1. `ContentHandler` interface : used to transform or process the message read by the parser.

The RecordParser is initialized with an input stream, which contains a single
message in the binary wire format used by the Change Notification framework, as
well as a ContentHandler instance. The parser is designed to process messages
efficiently and does not require the whole message to be loaded into memory at
once. It’s event-driven API is comparable to that of Java’s Simple API for XML
(SAX). As it processes each part of the message, the parser makes callbacks to
the ContentHandler interface, passing the logical contents of the message, such
as record metadata, a single bin value, or a scalar value as part of a bin
containing a complex data type (e.g. List or Map). RecordParser supports a
single, public parse() method, which starts the parsing process and which
returns once the whole message has been processed. A new instance must be
created for every message processed.

## Content Handler Interface
The ContentHandler interface must be implemented to receive notifications from
the parser about the logical contents of the message sent by the Change
Notification framework. This is the interface you are likely to use when
interfacing with a new message system. The interface contains a number of
methods that correspond to the underlying database operations represented by
the messages, e.g. recordStart() for an insert/update operation or
recordDelete() for a delete operation. Additional methods are used to pass the
values of individual record bins, including integerBin(), stringBin(),
blobBin(), etc. Aerospike supports two complex data types, Lists and Maps,
which can contain collections of scalar values as well as nested lists and
maps. The ContentHandler interface contains a separate set of methods through
which it is notified for each individual element with a list or map bin.

Sample code:
```
public class DemoContentHandler implements ContentHandler {
	private final List<String> binNames = new ArrayList<>();
	private Key key;
	private Metadata meta;

	@Override
	public void recordStart(Key key, Metadata meta, int numBins) {
		this.key = key;
		this.meta = meta;
	}

	@Override
	public void recordEnd() {
		logRecord(key, meta, binNames);
	}

	@Override
	public void integerBin(String name, long value) {
		binNames.add(name);
		if ((name.equals(“Balance”) && (value < 100)) {
			// send email alert
	}
}
...
...
```

The SDK includes two ContentHandler implementations, including source, which
can be used as-is, e.g. as part of an ETL pipeline. But they can also serve as
examples for how to implement a new content handler. The two content handlers
are:
1. JSON Handler
1. Record Object Handler

### JSON Handler
The JsonHandler implementation of the ContentHandler interface is an example of
how to perform a streaming transformation of the Change Notification
framework’s binary message format into a different format. As the parser
processes the input stream and notifies the JsonHandler of the logical message
contents, the handler transforms the contents into JavaScript Object
Notification (JSON), and writes it into a separate output stream.

The JSON format used by the handler is documented in the
./docs/json-serialization-format.md document, included in the SDK. A formal
JSON Schema is provided as well.

### Record Object Handler
A second ContentHandler implementation is provided in the RecordObjectHandler
class. Instead of performing a streaming transformation, it builds up an
in-memory representation of the Change Notification framework message. Once the
entire message has been parsed, the handler returns a Message object, which
includes details about the message, such as the operation performed, the record
key and a Record object, which includes the record’s metadata and bin values.
The Message and Record objects provide a simple interface to perform further
processing on the data. They are also serializable so that they could e.g. be
passed to a remote system for further processing.

Since the RecordObjectHandler builds up an in-memory representation of the
entire message, it may not be as efficient, especially for processing messages
containing large records.

## Integrating with Web Server
As the Aerospike server will publish the records as HTTP POST requests, we need
to setup the webserver in such a way that the the parsing library will be
invoked on a POST request. The content of the POST request should be sent to
the parsing library for parsing. Most webserver implementations will provide
some kind of callback for each type of HTTP request. We just need define the
callback for POST requests. Similarly, we also need to responding with HTTP-OK
on a GET request as that is used for health check. In the example below we are
demonstrating a HTTP servlet by extending the HttpServlet java class and
overriding its doPost() and doGet() methods. 

```
@WebServlet(urlPatterns = "/connect", loadOnStartup = 1)
public class AerospikeConnectServlet extends HttpServlet {

 @Override
 protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException
 {
  ContentHandler handler = new DemoContentHandler();
  AerospikeRecordParser parser = new AerospikeRecordParser(request.getInputStream(), handler);
  try {
   parser.parse();
  } catch (IOException | ParseException e) {
   log("Error parsing Aerospike record", e);
  }
  response.setStatus(HttpServletResponse.SC_OK);
 }

 @Override
 protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException
 {
  response.setStatus(HttpServletResponse.SC_OK);
 }
}
```


