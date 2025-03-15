# Commit 1: Handling Connections

In this milestone, I implemented a simple TCP server that listens to incoming
requests. Whenever a request comes in, the server prints "Connection Established"
to the console. This module provides an introduction on how single-threaded web
servers work: a loop that listens for incoming requests and handles them accordingly.
While the implementation is far from production-ready, it serves as a solid foundation
for understanding the basics of concurrency using Rust.

One key insight from this milestone is Rust's approach to error handling The
use of `.unwrap` is unfamiliar the first time I used it. The method extracts the
contained value from a `Result`, assuming it is `Ok`, and panics if it is an `Err`.
For example, in line 17, there is a code like this:

```rust
        .map(|result| result.unwrap())
```

The `result` variable is of type `Result<String, Error>`. This means that `result`
is not directly a String. To access the underlying string value, I need to call
`.unwrap()`. While this approach is easy, it is not the best practice, as this
may lead to runtime panics, also known as server crashes.

Overall, this milestone provided a strong introduction to Rust's concurrency model
and its unique programming paradigms. While Rust's strict type system and ownership
model present an initial learning curve, they ultimately contribute to safer and more
efficient code.
