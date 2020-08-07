# Trait中的`async` 

目前还不能在trait中使用`async fn`。原因有点复杂，但移除这种限制已在计划之中。

但与此同时，其实可以用[crates.io中的async-trait crate](https://github.com/dtolnay/async-trait)来实现。

需要注意的是，使用这些trait方法会导致，每次调用函数时都需要进行堆分配。尽管对于大部分应用来说都不会有什么显著的开销，但是若准备在那种，可能每秒都要调用几百万次的，底层函数的公共API里使用此功能，还请三思。