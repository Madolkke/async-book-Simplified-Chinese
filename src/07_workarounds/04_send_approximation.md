# `Send` 估计

有的`async fn`状态机在线程间传递时是安全的，有的不是。一个`async fn`的`future`是否具有`Send`trait是根据 是否有 跨越带`.await`的语句的 非`Send`类型来判断的。编译器会尽可能估计这些值可能跨越`.await`的时点，但是在今天看来，这种分析显得太过保守了。
举个例子，考虑一个简单的非`Send`类型，比如包含了一个`Rc`的类型：

```rust
use std::rc::Rc;

#[derive(Default)]
struct NotSend(Rc<()>);
```

`NotSend`类型的变量在`async fn`中可以作为临时变量短时间出现，即便`async fn`返回的最终的`Future`类型必须是`Send`：

```rust,edition2018
# use std::rc::Rc;
# #[derive(Default)]
# struct NotSend(Rc<()>);
async fn bar() {}
async fn foo() {
    NotSend::default();
    bar().await;
}

fn require_send(_: impl Send) {}

fn main() {
    require_send(foo());
}
```

但是，如果改变`foo`来将`NotSend`存储在一个变量中，这个例子就不能编译了：

However, if we change `foo` to store `NotSend` in a variable, this example no
longer compiles:

```rust,edition2018
# use std::rc::Rc;
# #[derive(Default)]
# struct NotSend(Rc<()>);
# async fn bar() {}
async fn foo() {
    let x = NotSend::default();
    bar().await;
}
# fn require_send(_: impl Send) {}
# fn main() {
#    require_send(foo());
# }
```

```
error[E0277]: `std::rc::Rc<()>` cannot be sent between threads safely
  --> src/main.rs:15:5
   |
15 |     require_send(foo());
   |     ^^^^^^^^^^^^ `std::rc::Rc<()>` cannot be sent between threads safely
   |
   = help: within `impl std::future::Future`, the trait `std::marker::Send` is not implemented for `std::rc::Rc<()>`
   = note: required because it appears within the type `NotSend`
   = note: required because it appears within the type `{NotSend, impl std::future::Future, ()}`
   = note: required because it appears within the type `[static generator@src/main.rs:7:16: 10:2 {NotSend, impl std::future::Future, ()}]`
   = note: required because it appears within the type `std::future::GenFuture<[static generator@src/main.rs:7:16: 10:2 {NotSend, impl std::future::Future, ()}]>`
   = note: required because it appears within the type `impl std::future::Future`
   = note: required because it appears within the type `impl std::future::Future`
note: required by `require_send`
  --> src/main.rs:12:1
   |
12 | fn require_send(_: impl Send) {}
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
```

这个报错没问题。如果我们将`x`存储到变量中，直到`.await`结束后它才应被释放，到那时`async fn`可能在另一个线程上运行了。既然`Rc`不是`Send`，允许其在线程间传递就是不安全的。一个简单的解决方案就是在`.await`前就把`Rc`给`drop`掉，不过这个目前不行。

为了解决这个问题，可以通过一个块作用域来把非`Send`变量包裹起来，这样编译器就能知道有哪些变量不能越过`.await`语句了。

```rust,edition2018
# use std::rc::Rc;
# #[derive(Default)]
# struct NotSend(Rc<()>);
# async fn bar() {}
async fn foo() {
    {
        let x = NotSend::default();
    }
    bar().await;
}
# fn require_send(_: impl Send) {}
# fn main() {
#    require_send(foo());
# }
```
