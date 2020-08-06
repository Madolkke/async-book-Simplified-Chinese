# 应用：创建Executor

Rust中的`Future`是惰性(_lazy_)的：除非受到主动驱动而被完成，不会做任何事。一种驱动方式是在`async`函数中使用`.await`，但是这样只是继续向上层抛出了问题：最终是由谁来运行最顶层的`async`函数返回的future呢？答案就是，我们需要一个`Future`的executor。

`Future`的executor会获取一系列的上层`Future`并将它们运行至完成，以在`Future`有进展时调用`poll`的方式来进行。一般来说，在一个future开始后，executor就会连续对其进行`poll`。当`Future`通过调用`wake()`来表明它们已经准备好继续进行，就会被放回一个队列中，并再次调用`poll`，如此重复进行直到`Future`完成。

在这节中，我们要编写一个简单的，负责并发运行大量上层future直到它们完成的executor。

此例子中，我们要用到`futures`crate中的`ArcWake`trait，这个trait能够提供一个更简便的方法来构造一个`Waker`。

```toml
[package]
name = "xyz"
version = "0.1.0"
authors = ["XYZ Author"]
edition = "2018"

[dependencies]
futures = "0.3"
```

接下来，在`src/main.rs`中添加如下import：

Next, we need the following imports at the top of `src/main.rs`:

```rust,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:imports}}
```

我们的executor会通过使task在通道(_channel_)中运行来工作。executor会从通道中取出事件(_event_)并运行之。当一个task准备好继续工作时(即被唤醒)，就可以通过将自身放回通道的方式来进行调度，使自身再次被轮询(_poll_)。

在这种设计中，executor只需要保存task通道的接收端。用户会获得发送端，这样他们就能创建新的future了。至于task，就是能够重新调度自身的future，所以我们将其以一个搭配了sender的future的形式存储，以使task能够通过sender来重新使自身进入(管道的)队列。

```rust,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:executor_decl}}
```

还得在(future的)生成器(_spawner_)中添加一个能让其创建新future的方法。这个方法会接受一个future类型的参数，把它装箱(_box it_)，并创建一个包含装箱结果的`Arc<Task>`，这样future就可以进入executor的(管道的)队列了。

```rust,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:spawn_fn}}
```

为了对future进行轮询，还需要创建一个`Waker`。如同在[task唤醒][task wakeups section]那一节中所讨论的，`Waker`负责在`wake`被调用时，再次调度task使之被轮询。`Waker`会告知executor哪一个task已经就绪，并允许它们再次轮询已经准备好继续运行的那些future。最简单的创建`Waker`的方法就是实现`ArcWake`trait再使用`waker_ref`或`.into_waker()`函数来将`Arc<impl ArcWake>`转换为`Waker`。下面为我们的task实现了`ArcWake`，使之能够转换为`Waker`和被唤醒：

```rust,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:arcwake_for_task}}
```

当从`Arc<Task>`创建一个`Waker`时，调用`wake()`会将一份`Arc`的靠背送入task的通道。我们的executor只需要取出它并对其进行轮询。实现如下：

```rust,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:executor_run}}
```

恭喜！现在我们就有了一个可工作的future的executor。我们甚至可以用它来运行`async/.await`的代码和自定义的future，比如我们之前编写的`TimerFuture`：

```rust,edition2018,ignore
{{#include ../../examples/02_04_executor/src/lib.rs:main}}
```

[task wakeups section]: ./03_wakeups.md
