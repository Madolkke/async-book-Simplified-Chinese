# 应用：简单的HTTP Server

来用`async`/`await`创建一个回显服务器吧！

首先，使用`rustup update stable`来确保你在使用Rust 1.39或者更新的版本。完成后，再输入`cargo new async-await-echo`来创建一个新项目，打开`async-await-echo`的文件夹。

在`Cargo.toml`中添加依赖：

```toml
{{#include ../../examples/01_05_http_server/Cargo.toml:9:18}}
```

解决了依赖问题后，就写点代码吧。先添加必要的import：

```rust,ignore
{{#include ../../examples/01_05_http_server/src/lib.rs:imports}}
```

之后，再加入用于处理请求的模板：

```rust,ignore
{{#include ../../examples/01_05_http_server/src/lib.rs:boilerplate}}
```

现在`cargo run`的话，终端里应该会显示 "Listening on http://127.0.0.1:3000" 的信息。若在浏览器中打开，就能看到其显示 "hello, world!"。恭喜！你刚刚写出了你的第一个异步的Rust webserver。

看一看请求的内容，就会发现它包含了请求的URI、HTTP版本号、请求头和其他的元数据一类的信息。可以这样直接输出请求的URI：

```rust,ignore
println!("Got request at {:?}", _req.uri());
```

可能你已经注意到了，在处理请求时我们并没有以异步的方式进行——我们只是立即进行响应，因此没能充分使用`async fn`的便利。比起仅仅返回一个静态的消息，来试试用Hyper的HTTP client来代理用户请求吧。

首先要解析请求的URL：

```rust,ignore
{{#include ../../examples/01_05_http_server/src/lib.rs:parse_url}}
```

然后创建一个新的`hyper::Client`，用它来发起一个`GET`请求，并将响应返回给用户：

```rust,ignore
{{#include ../../examples/01_05_http_server/src/lib.rs:get_request}}
```

`Client::get`会返回一个`hyper::client::ResponseFuture`，其实现为`Future<Output = Result<Response<Body>>>`(在future 0.1阶段为`Future<Item = Response<Body>, Error = Error>`)。当对该future使用`.await`时，就发送了一个HTTP请求，且当前任务被挂起到队列中，直到有可用的响应。

现在再`cargo run`并在浏览器中打开`http://127. 0.0.1:3000/foo`，就会看到Rust的主页，终端中有如下输出：

```
Listening on http://127.0.0.1:3000
Got request at /foo
making request to http://www.rust-lang.org/en-US/
request finished-- returning response
```

庆贺吧！这样就成功地代理了一个HTTP请求。