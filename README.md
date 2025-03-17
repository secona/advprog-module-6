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

# Commit 2: Returning HTML

![Commit 2 screen capture](/assets/images/commit2.png)

In milestone 1, the server did not respond to incoming requests, causing the browser to display a "site cannot be reached" error when visiting `localhost:7878`. In this milestone, I implemented the ability to return an HTML response, allowing the web server to function more like a real-world web server that serves files.

```rust
{
    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
}
```

Constructing an HTTP response follows a structured format. It begins with a status line (e.g., `HTTP/1.1 200 OK`), followed by headers, and finally the response body containing the requested content. The response body in this case is an HTML file, `hello.html`, which is read using Rust's `std::fs::read_to_string` function. This method efficiently reads the entire file into a string, making it easy to send as part of the HTTP response.

```rust
{
    stream.write_all(response.as_bytes()).unwrap();
}
```

After the response is constructed, it gets written to the `stream` variable. This way, when the function has finished executing, the `stream` gets returned from the server to the client with the response we had just constructed.

```rust
use std::fs;
use std::io::{BufRead, BufReader, Write};
use std::net::{TcpListener, TcpStream};
```

One interesting detail I encountered was the need to explicitly import `std::io::Write` to use the `write_all` method on `stream`. In Rust, `Write` is a trait that provides the ability to write bytes to a stream, and `TcpStream` implements this trait. This highlighted Rust's explicit trait system, where methods from traits are not automatically available unless the corresponding trait is in scope.

Overall, this milestone deepened my understanding of HTTP response structure and Rust's approach to file I/O and stream handling. It also reinforced how Rust's strict module and trait system encourages explicit and readable code. After this milestone get implemented, when I visit `localhost:7878`, the server responds with the correct HTML file.

# Commit 3: Validating Request and Selectively Responding

![Commit 3 screen capture](/assets/images/commit3.png)

In this milestone, I further implement the web server by adding a feature to handle 404 errors. The logic is to check the endpoint in the request line and handle them accordingly.

```rust
{
    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
}
```

The original implementation contains a lot of duplication, which can become difficult to manage when adding multiple endpoints. This redundancy increases the risk of inconsistencies, such as forgetting to update all occurrences of a response string template.

```
{
    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

The refactored version addresses this issue by eliminating duplication and centralizing the response logic. Instead of writing separate conditional blocks for each endpoint, it uses a match statement to determine the appropriate status line and file name. This approach improves readability, maintainability, and scalability, making it easier to add new endpoints in the future without introducing redundant code.

# Commit 4: Simulation of Slow Requests

Here, we simulate slow requests through the `/sleep` endpoint by using `thread::sleep` where each requests to `/sleep` takes 10 seconds. This replicates real-world scenarios where certain endpoints take longer to process, such as those involving database queries or external API calls. We can then observe how the server handles long-running requests.

The current implementation of the web server is single-threaded. This means that the server can only handle one request at a time. Therefore, when the server is handling a slow endpoint, such as `/sleep`, all other requests are blocked until it completes. This is a problem because a web server should be able to handle multiple requests at once.

# Commit 5: Multithreaded Server

In this milestone, I converted the single-threaded implementation of the web server to a multi-threaded implementation. The approach that I used is `ThreadPool`. A `ThreadPool` is a data structure that manages threads to handle requests. This approach is preferred over creating multiple unbounded threads to handle requests. Without using `ThreadPool`, the system could infinitely create many threads, potentially crashing the server. `ThreadPool` exists to limit the number of threads created.

`ThreadPool` works by storing multiple `Worker`s in a vector. These `Worker`s can then be delegated to incoming requests. That way, when a `Worker` is busy with an endpoint like `/sleep`, another `Worker` can handle other incoming requests, preventing request blockage.

Each `Worker` is a thin wrapper around `thread::JoinHandle<()>`. The communication between threads is done using `mpsc::channel`. When a request comes in, the main thread sends a `Job` to an available thread by using the `sender` variable from `mpsc::channel`. A `Job` is simply a boxed closure containing the function to be executed, in this case, a call to `handle_connection`. And since `Job` implements `Send`, it is safe to be sent across threads.

Overall, the conversion from single-threaded to multi-threaded significantly improves the server's performance. Previously, it could handle only one request at a time, but now it can process multiple requests concurrently.

# Commit 6: Bonus!

Here, I created a new function, `build`, as a replacement for `new`. The main advantage of `build` is error handling. The `new` function will panic if the size is less than or equal to zero. In contrast, the `build` function will return an `Err` to indicate an error.

This approach of error handling follows Rust's best practices as mentioned in [chapter 9 of the book](https://doc.rust-lang.org/book/ch09-00-error-handling.html). This makes the code more robust and flexible, as callers can decide how to proceed. By adopting this pattern, the function adheres to Rust's philosophy to make errors explicit and predictable, improving security and maintainability.
