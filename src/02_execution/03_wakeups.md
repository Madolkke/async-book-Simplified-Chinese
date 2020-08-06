# 通过`Waker`唤醒Task

future在第一次被`poll`时尚未完成是一件很平常的事。这件事发生时，future需要确保它在有新的进展时能够再次被轮询到。可以通过`Waker`类型做到这件事。

每次一个future被轮询时，其实是作为一个"task"的一部分来进行的。Task是被提交给executor的上层future们。

`Waker`提供了一个`wake()`方法，能够用于告知executor其关联的task需要被唤醒。当`wake()`被调用的时候，executor就知道与`Waker`关联的task已经有了进展，而future则会再次被轮询。

`Waker`也实现了`clone()`，因此它可以被复制再储存。

来试试通过`Waker`实现一个简单的定时器吧。

## 应用：创建定时器

由于只是个例子，我们将只在定时器创建时开辟一条新的线程，使其休眠所需的时间，然后在时间窗口结束时向计时器发出信号。

这是开始时要添加的import：

```rust
{{#include ../../examples/02_03_timer/src/lib.rs:imports}}
```

首先要定义future类型。future需要一条途径来告知线程定时器已经过指定的时间，该future应该被完成。

我们用一个共享的`Arc<Mutex<..>>`值来使线程和future进行交流。

```rust,ignore
{{#include ../../examples/02_03_timer/src/lib.rs:timer_decl}}
```

现在来编写`Future`的具体实现吧！

```rust,ignore
{{#include ../../examples/02_03_timer/src/lib.rs:future_for_timer}}
```

是不是很简单？当线程设置`shared_state.completed = true`时，就完成了！否则，我们就从当前的task把`Waker`进行clone，并传递至`shared_state.waker`，这样线程就能唤醒task了。

重要的是，每次future被轮询都要对`Waker`进行更新，因为future可能已经转移至另一个有不同`Waker`的task中了。这可能发生在future在被轮询后，在不同task间传递的过程中。

最后是用于构造计时器和启动线程的API：

```rust,ignore
{{#include ../../examples/02_03_timer/src/lib.rs:timer_new}}
```

这就是创建一个简单的计时器future的全过程了。现在，只差一个用于运行future的executor了