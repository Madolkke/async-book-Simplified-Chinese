# `join!`

`futures::join`宏可以在 并发执行多个future的同时 等待它们完成。

# `join!`

当进行多个异步行动时，很容易就只是简单地、连续对它们使用`.await`：

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:naiive}}
```

但是这样会比理论上要慢，因为`get_book`执行完后才会开始尝试执行`get_music`。在一些编程语言里，future会环境运行至完成(_ambiently run to completion_?译注：指有些语言会在调用异步函数时就开始执行future，而非像Rust一样到被`await`时才执行)，因此这两个行动会在`async fn`调用时就开始运行future，之后只要await它们就好了：

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:other_langs}}
```

但是，Rust中的Future在被`.await`时才开始工作。也就是说上面的两个代码片段都会顺序执行`book_future`和`music_future`，而非并发运行。为了使这两个future能正确地并发运行，要使用`futures::join!`：

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:join}}
```

`join!`的返回值是一个包含了 每个传入的`Future`的执行结果 的元组。

## `try_join!`

对于返回`Result`的future，可以考虑用`try_join!`取代`join!`。由于`join!`只在所有子future完成时才算完成，因此即便某个子future返回了`Err`，仍然会继续处理其他future。

与`join!`不同的是，`try_join!`会在任一子future返回`Err`时立即完成

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:try_join}}
```

需要注意的是，传递给`try_join!`的future参数返回的错误类型必须相同。请考虑使用`futures::future::TryFutureExt`中的`.map_err(|e| ...)`和`.err_into()`来合并错误类型：

```rust,edition2018,ignore
{{#include ../../examples/06_02_join/src/lib.rs:try_join_map_err}}
```
