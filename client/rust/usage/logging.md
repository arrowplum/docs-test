---
title: Logging
---

The Aerospike Rust client uses the light-weight loggig facade provided by the
[`log` crate](https://crates.io/crates/log). Application using the Aerospike
client can choose the logging implementation that is most suitable for its use
case.

{{#note}}
Logging is disabled by default. To enable logging, add one of the existing
logging implementations that support the `log` API to your crate's dependencies
consult the documenation provided by the logging information.
{{/note}}

### Using the `log` API with `env_logger`

The [`env_logger` crate](https://crates.io/crates/env_logger) provides an
implemenations of the logging facade defined by the [`log`
crate](https://crates.io/crates/log). To enable logging using `env_logger`, add
it to your applications dependencies:

```
[dependencies]
env_logger = "0.3"
```

#### Enabling logging for the `aerospike` module

_(Following is a small excerpt of the `env_logger` documentation; for more
information please refer to the [full
documentation](http://rust-lang-nursery.github.io/log/env_logger/).)_

Log levels are controlled on a per-module basis, and by default all logging is
disabled except for `error!`. Logging is controlled via the `RUST_LOG` environment
variable. The value of this environment variable is a comma-separated list of
logging directives. A logging directive is of the form:

    path::to::module=log_level

To turn on all logging for the Aerospike client you would use a value of

    RUST_LOG=aerospike

The actual log_level is optional to specify. If omitted, all logging will be
enabled. If specified, it must be one of the strings `debug`, `error`, `info`,
`warn`, or `trace`.
