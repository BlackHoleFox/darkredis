# darkredis : A Redis client based on `std::future` and `async_await`
[![Documentation](https://docs.rs/darkredis/badge.svg)](https://docs.rs/darkredis) [![Build Status](https://travis-ci.org/Bunogi/darkredis.svg?branch=master)](https://travis-ci.org/Bunogi/darkredis) [![Crates.io Status](https://img.shields.io/crates/v/darkredis.svg)](https://crates.io/crates/darkredis)


`darkredis` is a Redis client for Rust written using the new `std::future` and `async_await`. It's designed to be easy to use, and lacks advanced features. Requires rust 1.39.0 or newer.

Currently not all Redis commands have convenience functions, and there may be ergonomic improvements to make.

## Why?
When `darkredis` was originally written, it was part of an async project. At the time, there was no real way to use `.await` with redis-rs, so I set out to change that. The result is `darkredis`, and it has worked very well for my own personal use, which it might for you too. It's designed to be as simple as possible, serving as an easy way to call redis commands.

# Cargo features
 - `runtime_tokio`(on by default): Use the tokio 0.2 runtime and Tcp primitives. Requires a running tokio 0.2 runtime.
 - `runtime_agonic`: Use `async-std` instead of tokio, allowing you to use other runtimes than tokio. Is mutually exclusive with the `runtime_tokio` feature.

# Getting started
- Add `darkredis` and to your `Cargo.toml`.
- Create an async context somehow, for instance by using `tokio` 0.2.
- Create a `ConnectionPool` and grab a connection!

```rust
use darkredis::ConnectionPool;

#[tokio::main]
async fn main() -> darkredis::Result<()> {
    let pool = ConnectionPool::create("127.0.0.1:6379".into(), None, num_cpus::get()).await?;
    let mut conn = pool.get().await;

    //And away!
    conn.set("secret_entrance", b"behind the bookshelf").await?;
    let secret_entrance = conn.get("secret_entrance").await?;
    assert_eq!(secret_entrance, Some("behind the bookshelf".into()));

    //Keep our secrets
    conn.del("secret_entrance").await?;

    Ok(())
}
```

# Changelog
## 0.4.1
- Updated to async-std 1.0.1 and futures 0.3.1
## 0.4.0
- (BREAKING) Add the `runtime_agnostic` and `runtime_tokio` features.
- Simple Pubsub support using the `MessageStream` and `PMessageStream` types.
- Ability to spawn a new connection using the settings from a ConnectionPool
- Use `tokio` in test mode.
- Add convenience function for `PUBLISH`
- Add pubsub example
- Add variants of builder functions for `Command` and `CommandList` which mutate the object instead of moving it.
## 0.3.1
- Expclicitly use traits from `async-std`, not `std`. This fixes compilation on async-std 0.99.5
## 0.3.0
- Add convenience functions for the `INCR` and `DECR` family of commands, as well as for `APPEND`, `MGET`, `MSET`, `EXISTS`.
- Improve the documentation of convenience functions
- Allow renaming of client connections in connection pools
## 0.2.3
- Use `async-std` instead of `runtime` for TcpStream, allowing using darkredis with any runtime.
## 0.2.2
- Update dependencies
## 0.2.1
- Fix compilation error on latest nightly
## 0.2.0
- Command and CommandList no longer perform any copies
- Added args method to Command and CommandList
- `lpush` and `rpush` now take multiple arguments
- Support password auth
- `lrange` no longer returns an Option, returns an empty vec instead.
- Add convenience functions for the following commands: `lset`, `ltrim`
## 0.1.3
- Remove unnecesarry generic parameter from lpop and rpop methods.
## 0.1.2
- Fix a couple documentation errors
## 0.1.1
- Initial release

# Testing
If you're hacking on `darkredis` and want to run the tests, make sure you have a Redis instance running on your local machine on port 6379. The tests clean up any keys set by themselves, unless the test fails. Please submit an issue if it does not.

Also, make sure to run the tests using both the `runtime_tokio` and `runtime_agnostic` features, to make sure that it works.
