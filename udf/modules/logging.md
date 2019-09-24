
 Lua: logging Module

    Usage
    Logging Functions
        trace()
        debug()
        info()
        warn()

Usage

Aerospike provides a set of logging functions that send log message to the Aerospike Server's logs.

The logging functions all accept a format string as the first argument with a variable argument list, much like printf in C.

While logging, you will need to ensure the value used as arguments for the format string are valid for the format string args. When in doubt, you can use the Lua tostring() function to convert a value to a string, whenever possible.

The following will ensure the value will be correctly logged, even if it is a nil or some other type that is not a String or Number.
info("Hello %s", tostring(name))
Logging Functions

The following are the logging functions available to UDFs.
trace()

Log a message as DETAIL in the Aerospike Server.  
trace(String msg, …)
debug()

Log a message as DEBUG in the Aerospike Server.  
debug(String msg, …)
info()

Log a message as INFO in the Aerospike Server.
info(String msg, …)
warn()

Log a message as a WARNING in the Aerospike Server.  
warn(String msg, …)

