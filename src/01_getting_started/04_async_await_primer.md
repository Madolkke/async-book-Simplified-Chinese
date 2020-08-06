# `async`/`.await` 入门

`async`/`.await`是Rust中用来像同步代码一样编写异步代码的工具。`async`会将一个代码块转换为一个实现了`Future`trait的状态机。尽管在同步方法中调用阻塞函数(_blocking function_)会使整个线程阻塞，被阻塞的`Future`会让出线程的控制权，好让其他的`Future`能够运行。

先在`Cargo.toml`文件中添加一些依赖：

```toml
{{#include ../../examples/01_04_async_await_primer/Cargo.toml:9:10}}
```

使用`async fn`的语法就可以创建一个异步函数了：

```rust,edition2018
async fn do_something() { /* ... */ }
```

`async fn`返回值是一个`Future`，一个`Future`必须通过executor来运行。

```rust,edition2018
{{#include ../../examples/01_04_async_await_primer/src/lib.rs:hello_world}}
```

在一个`async fn`中，可以使用`.await`来等待另一个实现了`Future`trait的类型的完成，比如另一个`async fn`的输出。与`block_on`不同，`.await`不会阻塞当前的线程，而是异步地等待该future的完成，使得当这个future没有进展时其他任务仍然可以运行。

举个例子，假设有三个`async fn`：`learn_song`，`sing_song`，和`dance`：

```rust,ignore
async fn learn_song() -> Song { /* ... */ }
async fn sing_song(song: Song) { /* ... */ }
async fn dance() { /* ... */ }
```

一种方式是令learn，sing和dance分别被阻塞：

```rust,ignore
{{#include ../../examples/01_04_async_await_primer/src/lib.rs:block_on_each}}
```

这不是最好的方式——一次只能做一件事！显然要想sing，我们就必须先learn song，而dance是可以与learn和sing同时进行的。要这样做的话，可以创建两个可以并行运行的`async fn`：

```rust,ignore
{{#include ../../examples/01_04_async_await_primer/src/lib.rs:block_on_main}}
```

在这个例子中，learn song一定会发生在sing song之前，但是learn和sing都可以和dance同时进行。如果我们在`learn_and_sing`中使用`block_on(learn_song())`而非`learn_song().await`，线程就不能在`learn_song`的同时做其他的事情了。(后者)使得我们可以同时进行`dance`。通过`.await` `learn_song`的future，我们使得其他的任务可以在`learn_song`被阻塞时接管当前线程，这样就能在同一线程上并发运行多个future直至其完成了。

现在你已经学习到了`async`/`await`的基础知识了，来个例子试试吧。