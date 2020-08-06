# `Stream` Trait

`Stream`trait和`Future`很相像，但`Stream`可以在其完成前返回多个值，与标准库中的`Iterator`trait类似：

```rust,ignore
{{#include ../../examples/05_01_streams/src/lib.rs:stream_trait}}
```

一个常见的`Stream`例子就是`futures`crate中的管道类型`Receiver`。每次`Sender`段有值发送时都会产生一个`Some(val)`，当`Sender`释放，且所有信息已接收完成时则产生`None`：

```rust,edition2018,ignore
{{#include ../../examples/05_01_streams/src/lib.rs:channels}}
```
