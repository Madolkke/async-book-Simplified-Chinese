# `Future` Trait

`Future`trait是Rust异步编程的核心。一个`Future`就是一个能够产生值的异步计算过程(即便值可以为空，如`()`)。下面是一个简化版的future trait：

```rust
{{#include ../../examples/02_02_future_trait/src/lib.rs:simple_future}}
```

可通过调用`poll`函数来提前执行future，以让future尽早完成。在future完成后，会返回`Poll::Ready(result)`。若future尚未完成，则会返回`Poll::Pending`并设置`Future`有进展时要调用的`wake()`函数。当`wake()`被调用时，用于驱动`Future`的executor会再次调用`poll`以便使`Future`能够继续进行。

如果没有`wake()`，executor就无从得知是否有某个future取得进展，而不得不对future进行轮询。通过`wake()`，executor就能准确得知哪个future准备好被`poll`了。

考虑这样的情况，我们需要从一个可能有数据，也可能没有数据的socket中进行读取内容。如果有可用的数据，就可以读取它并返回`Poll::Ready(data)`，但若数据尚未准备好，future就会被阻塞，不再有进展。当这种情况发生时，我们就得注册(_register_)当数据准备好时要调用的`wake`，它会告知executor，该future已经准备好继续进行了。一个简单的`SocketRead`future如下：

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:socket_read}}
```

这种基于`Future`的模型使得我们不依赖中间的分配(_intermediate allocations_)即可组织多个异步的行为。运行多个future或chaining futures可通过免分配状态机实现：

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:join}}
```


这个例子展示了如何在不分别分配任务的情况下，使多个future共同运行以提高效率。类似地，多个顺序future也可以接连运行，就像这样：

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:and_then}}
```

这些例子展示了`Future`trait是如何用于表达异步控制流，而不需借助多个分配对象和多级嵌套回调。解决了基础的控制流问题，现在来说说和实际的`Future`trait的不同之处吧。

```rust,ignore
{{#include ../../examples/02_02_future_trait/src/lib.rs:real_future}}
```

第一处变动是`self`的类型不再是`&mut Self`，而是变成了`Pin<&mut Self>`。我们会在 [之后的章节][pinning]中讨论固定(_pinning_)的内容，现在只要知道，它能让我们创建不能变动内容(_immovable_)的future就行了。这种对象能够存储其各字段的指针，比如`struct MyFut { a: i32, ptr_to_a: *const i32 }`。固定在`async`/`.await`的实现中非常有用。

其次，`wake: fn()`变成了`&mut Context<'_>`。在`SimpleFuture`中，我们通过调用一个函数指针(`fn()`)来告知future的executor，这个future需要被轮询。但是因为`fn()`只是一个函数指针，它不能储存在`Future`中被称为`wake`的数据。

在现实中，像web服务器这样的复杂应用可能会有上千个不同连接，而这些连接的唤醒状况也需要分别管理。`Context`类型通过提供一个`Waker`类型值来解决这个问题，这个值能够用于唤醒一个具体的任务(_task_)。

[pinning]: ../04_pinning/01_chapter.md
