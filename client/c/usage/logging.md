---
title: Logging
description: Configure internal logging for the Aerospike C client and Aerospike database.
---

The Aerospike C client log facility generates log messages for cluster state changes and other useful debugging purposes.

{{#note}}
Logging is disabled by default. 
{{/note}}

### Log Callback

To enable logging, provide a callback that will be invoked for each log message.  The signature of the log callback is:

```cpp
typedef bool (*as_log_callback)(as_log_level level, const char *func, const char *file, uint32_t line, const char *fmt, ...);
```

Where,

- `level`  &mdash; The level of the log message.
- `func` &mdash; The function in which the log message was recorded.
- `file` &mdash; The name of the file where the log message was recorded.
- `line` &mdash; The line number in the file where the log message was recorded.
- `fmt` &mdash; The format string for the log message.
- `...` &mdash; The format parameters for `fmt`.

Example log callback:

```cpp
bool my_log_callback(as_log_level level, const char *func, const char *file,
    uint32_t line, const char *fmt, ...)
{
    char msg[1024] = {0};
    va_list ap;
  
    va_start(ap, fmt);
    vsnprintf(msg, 1024, fmt, ap);
    msg[1023] = '\0';
    va_end(ap);
 
    fprintf(stderr, "[%s:%d][%s] %d - %s\n", file, line, func, level, msg);
  
    return true;
}
```

Use the following to set the callback:

```cpp
as_log_set_level(AS_LOG_LEVEL_INFO);
as_log_set_callback(my_log_callback);
```

### Log Level

To modify the verbosity of the log messages, set a different log level using `as_log_set_level()`:

```cpp
as_log_set_level(AS_LOG_LEVEL_DEBUG);
```

Valid log levels are:

- `AS_LOG_LEVEL_ERROR`
- `AS_LOG_LEVEL_WARN`
- `AS_LOG_LEVEL_INFO`
- `AS_LOG_LEVEL_DEBUG`
- `AS_LOG_LEVEL_TRACE`
