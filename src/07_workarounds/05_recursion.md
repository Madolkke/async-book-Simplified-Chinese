# 递归

在内部，`async fn`会创建一个包含了所有被`.await`的子`Future`的状态机类型。由于最终的状态机类型要包含自身，要实现递归的`async fn`就有些难办了：

```rust,edition2018
# async fn step_one() { /* ... */ }
# async fn step_two() { /* ... */ }
# struct StepOne;
# struct StepTwo;
// 这个函数：
async fn foo() {
    step_one().await;
    step_two().await;
}
// 会产生一个这样的类型：
enum Foo {
    First(StepOne),
    Second(StepTwo),
}

// 因此这个函数：
async fn recursive() {
    recursive().await;
    recursive().await;
}

// 会产生这样一个类型：
enum Recursive {
    First(Recursive),
    Second(Recursive),
}
```

此时还不能工作——我们创建了一个无限大小的类型！
编译器会这样报错：

```
error[E0733]: recursion in an `async fn` requires boxing
 --> src/lib.rs:1:22
  |
1 | async fn recursive() {
  |                      ^ an `async fn` cannot invoke itself directly
  |
  = note: a recursive `async fn` must be rewritten to return a boxed future.
```

为了允许这种情况的发生，就要在中间加一层`Box`。然而编译器的限制意味着，仅仅是将对`recursive()`的调用包装在`Box::pin`还不够。我们还得将`recursive`转化为一个非`async`函数并返回一个经过`.boxed()`的`async`块：

```rust,edition2018
{{#include ../../examples/07_05_recursion/src/lib.rs:example}}
```
