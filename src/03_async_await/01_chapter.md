# `async`/`.await`

在 [第一章][the first chapter]中我们简单领略了下`async`/`await`并用它们建立了一个简单的服务器。这一章会更深入地讨论`async`/`.await`的细节，解释它们如何工作，而`async`代码和传统的Rust程序又有怎样的不同。

`async`/`.await`是使让出当前线程的控制权而非阻塞成为可能的语法，让程序可以在等待某个行为完成的同时允许其他代码运行。

有两个使用`async`的主要方法：`async fn`和`async`块(_block_)。它们都会返回一个实现了`Future`trait的值：

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:async_fn_and_block_examples}}
```

正如我们在第一章中看到的，`async`的主体和其他future都是惰性的：直到运行前都不会做任何事。运行一个`Future`最简单的方法就是`.await`它。当对`Future`调用`.await`时，会试图将Future运行到完成状态。如果`Future`阻塞了，就让出当前线程的控制权。而可以继续执行时，`Future`就会被executor取出并回复运行，让`.await`得以解析(_resolve_)。

## `async` 生命周期

与普通函数不同，获取引用或其他非`'static`参数的`async fn`会返回一个受参数声明周期约束的`Future`。

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:lifetimes_expanded}}
```

这意味着`async fn`返回的future必须 在其非`'static`参数还有效时 被`.await`。一般情况下，在调用函数后立即`.await`其返回的future(比如`foo(&x).await`)没有问题。但是如果 保存了此future 或 将其发送给了另一个task或线程，就可能出问题。

一个将含引用参数的`async fn`转化为`'static`future的常见方案是，将参数及其调用打包在`async fn`内部的一个`async`块中：

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:static_future_with_borrow}}
```

通过将参数移入`async`块中，我们延长了它的生命周期，使得它匹配了调用`call`时返回的`Future`(的生命周期)。

## `async move`

可以对`async`块和闭包使用`move`关键字，就跟普通的闭包一样。`async move`块会获取其引用变量的所有权，使得该变量能够在当前作用域外留存，但是失去了与其他部分代码共享自身的能力：

```rust,edition2018,ignore
{{#include ../../examples/03_01_async_await/src/lib.rs:async_move_examples}}
```

## 多线程Executor的`.await`

需要注意的是，当使用一个多线程`Future`executor时，一个`Future`可能会在线程间转移，因此`async`代码体重的任何变量同样有在线程间转移的可能，因为`.await`有可能导致切换至新的线程。

也就是说，使用`Rc`，`&RefCell`或者其他没有实现`Send`trait的类型，以及 没有实现`Sync`的类型 的引用，都是不安全的。

(另：如果不是在调用`.await`时使用还是可以的。)

类似地，不应该在`.await`过程中使用一个传统的，非future感知的锁(_non-futures-aware lock_)，因为其可能导致线程池的锁定：一个task除去锁，进行`await`并向executor让出，让另一个task尝试除去锁，因而造成死锁。为了避免这个现象，请使用`futures::lock`中的`Mutex`，而不是`std::sync`。



[the first chapter]: ../01_getting_started/04_async_await_primer.md
