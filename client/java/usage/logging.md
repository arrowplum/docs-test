---
title: Logging
---

The Aerospike Java client contains a logging interface useful for debugging. 

{{#note}}
Logging is disabled by default. 
{{/note}}

### Log Callback

To enable logging, provide a callback to invoke for each log message. Ensure that the callback is an instance of `com.aerospike.client.Log.Callback`:

```java
	public interface Callback {
		public void log(Log.Level level, String message);
	}
```

To enable logging, create an instance of `Callback` and set it as the logging callback using `Log.setCallback()`:

```java
	class Log {
		public static void setCallback(Log.Callback callback);
	}
```

**Example**

```java
	public final class MyLogCallback implements Log.Callback {
		@Override
		public void log(Log.Level level, String message) {
			Date date = new Date();
			System.out.println(date.toString() + ' ' + level + ' ' + message);
		}
	}
```
This example creates an instance of `MyLogCallback` called `mycallback` and set it as the logging callback:

```java
	Log.Callback mycallback = new MyLogCallback();
	
	Log.setCallback(mycallback);
```

### Log Level

To control the verbosity of logs sent to the log callback, use `Log.setLevel()`. 

```java
	class Log {
		public static void setLevel(Log.Level level);
	}
```

Log levels are defined in `com.aerospike.client.Log.Level`.

To log all debug messages, set the level to `Log.Level.DEBUG`:

```java
	Log.setLevel(Log.Level.DEBUG);
```
