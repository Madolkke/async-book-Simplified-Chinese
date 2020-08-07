# 返回类型出错

在一个常见的Rust函数中，返回一个错误的类型会导致如下错误：

```
error[E0308]: mismatched types
 --> src/main.rs:2:12
  |
1 | fn foo() {
  |           - expected `()` because of default return type
2 |     return "foo"
  |            ^^^^^ expected (), found reference
  |
  = note: expected type `()`
             found type `&'static str`
```

但是当前的`async fn`是不知道如何“信任”函数签名中的返回类型的，因而可能引起类型不匹配，甚至是(_reversed-sounding_)错误。比如说，函数`async fn foo() { "foo" }`就会导致这个错误：

```
error[E0271]: type mismatch resolving `<impl std::future::Future as std::future::Future>::Output == ()`
 --> src/lib.rs:1:16
  |
1 | async fn foo() {
  |                ^ expected &str, found ()
  |
  = note: expected type `&str`
             found type `()`
  = note: the return type of a function must have a statically known size
```

错误信息说该函数应返回一个`&str`值，但只找到了一个`()`，跟我们想象中的情况正好相反。这是因为编译器错误地相信了“函数体会返回正确类型”这件事。

一个应对方法就是记住，当错误信息通过"expected `SomeType`, found `OtherType`"的信息指出了 函数签名 的问题时，通常表明实际上是 返回的位置 出了问题。

正在通过[这个bug报告页](https://github.com/rust-lang/rust/issues/54326)来追踪此问题的修复情况。

