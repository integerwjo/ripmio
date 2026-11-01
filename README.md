# ripmio

A minimal I/O event notification system for TCP streams built on top of Linux `epoll`, implemented from scratch with raw FFI bindings to libc. Inspired by [`mio`](https://github.com/tokio-rs/mio).

## How it works

The crate exposes two small pieces:

- `Poll` — wraps an `epoll` file descriptor. `Poll::new()` creates the instance, and `poll()` waits for readiness events with an optional timeout.
- `Registry` — obtained via `Poll::registry()`, used to register TCP streams with an interest set and a token that identifies them when events fire.

The FFI layer in `src/ffi.rs` declares the necessary `epoll` syscalls and the `Event` struct used to receive notifications.

```rust
let mut poll = Poll::new()?;

let mut stream = TcpStream::connect("localhost:8080")?;
stream.set_nonblocking(true)?;

poll.registry()
    .register(&stream, 0, EPOLLIN | EPOLLET)?;

let mut events = Vec::with_capacity(10);
poll.poll(&mut events, None)?;

for event in &events {
    println!("ready: token={}", event.token());
}
```

## Example

`src/main.rs` connects to a server on `localhost:8080`, opens five non-blocking TCP connections, and registers each one with the poller using edge-triggered mode (`EPOLLET`). It then waits for readiness notifications and drains each stream until it would block or the connection closes.

## Building and running

```sh
cargo build
cargo run
```

The example expects an HTTP server listening on `localhost:8080` (the request paths are intentionally delayed, so any simple HTTP server works).

## Platform support

Linux only — the implementation relies on `epoll`.
