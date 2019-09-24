---
title: Error Handling
description: Aerospike Rust client error handling.
---

The Rust client provides the `aerospike::errors::Error` type which is used for
all errors returned by the client. The `Error` type is a struct containing an
`ErrorKind` (which defines the `description` and `display` methods for the
error), an opaque, optional, boxed `std::error::Error + Sync + Send + 'static`
object (which defines the `cause`, and establishes the links in the error
chain), and a `Backtrace`.

`ErrorKind` defines the following variants:

Exception | Description 
--- | ---
Base64             | Error deserializing a Base64 encoded value.
InvalidUtf8        | Error deserializing a sequence of `u8` into a UTF-8 encoded string.
Io                 | Error during an I/O operation.
MpscRecv           | Error returned from the `recv` function on an MPSC `Receiver`.
ParseAddr          | Error parsing an IP or socket address.
ParseInt           | Error parsing an integer.
PwHash             | Error returned while hashing a password for user authentication.
BadResponse        | The client received a server response that it was not able to process.
Connection         | The client was not able to communicate with the cluster due to some issue with the network connection.
InvalidArgument    | One or more of the arguments passed to the client are invalid.
ServerError        | The server responded with a response code indicating an error condition. The `ResultCode` enum lists all possible response codes and their meanings.
UdfBadResponse     | Error returned when executing a User-Defined Function (UDF) resulted in an error.

For more information on how to use the error chain please refer to the
documentation for external the `error_chain` crate, which is used to define the
`Error` type: https://docs.rs/error-chain/.

Here is example code that handles an AersopikeException.

```rust
#[macro_use] extern crate aerospike;
use aerospike::*;

fn main() {
    let hosts = std::env::var("AEROSPIKE_HOSTS").unwrap();
    let policy = ClientPolicy::default();
    let client = Client::new(&policy, &hosts).expect("Failed to connect to cluster");
    let key = as_key!("test", "test", "someKey");
    match client.get(&ReadPolicy::default(), &key, Bins::None) {
        Ok(record) => {
            match record.time_to_live() {
                None => println!("record never expires"),
                Some(duration) => println!("ttl: {} secs", duration.as_secs()),
            }
        },
        Err(Error(ErrorKind::ServerError(ResultCode::KeyNotFoundError), _)) => {
            println!("No such record: {}", key);
        },
        Err(err) => {
            println!("Error fetching record: {}", err);
            for err in err.iter().skip(1) {
                println!("Caused by: {}", err);
            }
            // The backtrace is not always generated. Try to run this example
            // with `RUST_BACKTRACE=1`.
            if let Some(backtrace) = err.backtrace() {
                println!("Backtrace: {:?}", backtrace);
            }
        }
    }
}
```
