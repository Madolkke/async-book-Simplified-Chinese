# `async` 块中的`?`

就跟在`async fn`中一样，在`async`块中使用`?`的场景也很常见。但是，`async`块的返回类型并未显式声明出，因而可能导致编译器不能推断出`async`块的错误类型。

比如说这段代码：

```rust,edition2018
# struct MyError;
# async fn foo() -> Result<(), MyError> { Ok(()) }
# async fn bar() -> Result<(), MyError> { Ok(()) }
let fut = async {
    foo().await?;
    bar().await?;
    Ok(())
};
```

会导致这个错误：

```
error[E0282]: type annotations needed
 --> src/main.rs:5:9
  |
4 |     let fut = async {
  |         --- consider giving `fut` a type
5 |         foo().await?;
  |         ^^^^^^^^^^^^ cannot infer type
```

不幸的是，目前还没有“为`fut`注明类型”，或者是显式声明`async`块返回值的方法。应对方法是使用“涡轮鱼(_turbofish_，指Rust中用来绑定泛型参数的操作符`::<>`)”操作符来为`async`块提供成功和失败时的返回类型。

```rust,edition2018
# struct MyError;
# async fn foo() -> Result<(), MyError> { Ok(()) }
# async fn bar() -> Result<(), MyError> { Ok(()) }
let fut = async {
    foo().await?;
    bar().await?;
    Ok::<(), MyError>(()) // <- note the explicit type annotation here
};
```

