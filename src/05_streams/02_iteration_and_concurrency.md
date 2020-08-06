# 迭代与并发

与同步`Iterator`相似，有不少不同的迭代处理`Stream`中值的方法。有像`map`，`filter`和`fold`这种组合子风格(_combinator-style_)的方法，还有它们提供了出错时提早退出的(_early-exit-on-error_)的近亲`try_map`，`try_filter`和`try_fold`。

不幸的是，不能对`Stream`使用`for`循环，但是对于命令式风格(_imperative-style_)的代码，是可以使用`while let`和`next`/`try_next`这样的函数的：

```rust,edition2018,ignore
{{#include ../../examples/05_02_iteration_and_concurrency/src/lib.rs:nexts}}
```

但要是一次只处理一个元素，就失去了并发的机会，毕竟编写异步代码是第一位的。为了并发处理Stream中的多个内容，请使用`for_each_concurrent`和`try_for_each_concurrent`：

```rust,edition2018,ignore
{{#include ../../examples/05_02_iteration_and_concurrency/src/lib.rs:try_for_each_concurrent}}
```
