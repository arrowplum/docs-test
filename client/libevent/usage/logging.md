---
title: Logging
description: Use the Aerospike libevent client to configure logging.
---

{{#warn}}
Aerospike deprecated our C libevent Client Library.
<BR>
Please use the standard **[C Client](https://www.aerospike.com/download/client/c/)**, which supports asynchronous programming models.
{{/warn}}

Use the Aerospike libevent client to configure logging.

By default, the Aerospike libevent client logs messages to stderr, which provides basic information about the client.

### Setting the Log Level

Applications can change the verbosity level of these log messages using `cf_set_log_level()`. The function is thread-safe and can be called any time.

```cpp
// Use internal client logging, but set a filter.
cf_set_log_level(CF_WARN);
```

Valid log levels are:

- `CF_NO_LOGGING`
- `CF_ERROR`
- `CF_WARN`
- `CF_INFO` (default)
- `CF_DEBUG`

### Setting a Custom Log Callback

To provide a custom log callback using `cf_set_log_callback()`:

``` cpp
cf_set_log_callback(my_log_cb);
``` 

This directs log messages to the callback instead of to stderr to allow the application to format and filter the messages.

The signature of the log callback is:

```cpp
typedef void (*cf_log_callback)(cf_log_level level, const char* fmt, ...);
```

Where,

- `level` &mdash; The level of the log message.
- `fmt` &mdash; The format string for the log message (cannot end with `\n`).
- `...` &mdash; The parameters for `fmt`.

This example custom log callback adds a prefix and newline:

```cpp
void
my_log_cb(cf_log_level level, const char* fmt_no_newline, ...)
{
	// Add a prefix to show it's a client log statement, and to show the level.
	const char* prefix;

	switch (level) {
	case CF_DEBUG: prefix = "Aerospike Client - DEBUG: "; break;
	case CF_INFO:  prefix = "Aerospike Client - INFO: ";  break;
	case CF_WARN:  prefix = "Aerospike Client - WARN: ";  break;
	default: // Should not get here - just fall-through to error case.
	case CF_ERROR: prefix = "Aerospike Client - ERROR: "; break;
	}

	// New format string length is:
	// prefix length + format parameter length + 1 (for newline).
	size_t prefix_len = strlen(prefix);
	size_t fmt_len = prefix_len + strlen(fmt_no_newline) + 1;

	// Generate the new format string.
	char fmt[fmt_len + 1];

	strcpy(fmt, prefix);
	strcpy(fmt + prefix_len, fmt_no_newline);
	fmt[fmt_len - 1] = '\n';
	fmt[fmt_len] = 0;

	// Using the new format string, write the log message to stdout.
	va_list ap;

	va_start(ap, fmt_no_newline);
	vfprintf(stdout, fmt, ap);
	va_end(ap);

	fflush(stdout);
}
```

