# Commit 1: Handling Connections

In this milestone, I implemented a simple TCP server that listens for incoming requests. Whenever a request arrives, the server prints "Connection Established" to the console. In this reflection, I will analyze the code changes and their significance.

```rust
use std::io::{BufRead, BufReader};
use std::net::{TcpListener, TcpStream};
```

This part of the code imports various structs and traits needed to implement the web server. Both lines import modules from Rust's standard library: `std::io` and `std::net`.

```rust
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();
        handle_connection(stream);
    }
}
```

This is the `main` function, which runs when the program starts. It creates a `TcpListener` bound to port 7878. The unwrap method is used to extract the `TcpListener` instance from the `Result` returned by `TcpListener::bind`. This is acceptable because if binding fails, the program cannot function properly and should terminate.

The second part of the function is a loop that listens for incoming requests. Each request is unwrapped and passed to the `handle_connection` function for processing.

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("{:#?}", http_request);
}
```

The `handle_connection` function processes each request by reading and printing it to the console. The `http_request` line reads the incoming HTTP request, collects the headers into a vector, and stops when it encounters an empty line. This effectively extracts the request headers, which are then printed for debugging. The `#?` formatter enables pretty-printed debug output.

Overall, this milestone introduced me to the basics of web servers and how they function. It also reinforced my understanding of Rust’s fundamentals. The Rust compiler is very strict, often flagging even minor issues. While this can make initial development challenging, it helps catch errors early, preventing them from accumulating later. This makes debugging significantly easier in the long run.
