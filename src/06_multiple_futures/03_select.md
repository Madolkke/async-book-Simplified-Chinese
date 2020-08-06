# `select!`

`futures::select`宏让用户可同时运行多个future，并在在任意future完成时做出响应。

```rust,edition2018
{{#include ../../examples/06_03_select/src/lib.rs:example}}
```

上面的函数会并发运行`t1`和`t2`。只要其中一个完成了，对应的处理程序(_corresponding handler_)就会调用`println!`，并在不执行剩余task的情况下直接结束。

`select`的基本语法是`<pattern> = <expression> => <code>,`，对每一个要`select`的future重复即可。

## `default => ...` 和 `complete => ...`

`select`同样支持`default`和`complete`分支。

`default`分支会在每个被`select`的future都还没完成时执行。拥有`default`分支的`select`会因此总是立即返回，因为`default`会在任一future都未就绪时执行。

`complete`分支可用来处理所有被`select`的future均完成且不需再运行的情况，经常是在`select!`中循环时被用到。

```rust,edition2018
{{#include ../../examples/06_03_select/src/lib.rs:default_and_complete}}
```

## 与`Unpin` 和 `FusedFuture`的交互使用

在上面的第一个例子中你可能注意到，我们不得不对这两个`async fn`返回的future调用`.fuse()`，并使用`pin_mut`将其固定(_pin_)。之所以要这么做，是因为在`select`中使用的future必须实现`Unpin`和`FusedFuture`两个trait。

由于`select`接受的future参数并非是其值，而是其可变引用，因此需要`Unpin`。而也正是因为没有获取future的所有权，在调用`select`后未完成的future仍可以被重用。

类似地，若要让`select`不在某个future已完成后，再对其进行轮询，该future就需要`FusedFuture`trait。通过实现`FusedFuture`，可以追踪future是否完成，因而可以在循环中使用`loop`，且只要对尚未完成的future进行轮询就够了。通过上面的例子就能看出，`a_fut`或`b_fut`在第二次循环时应已完成。由于`future::ready`返回的future实现了`FusedFuture`，就能够告知`select`不要再次对其进行轮询。

需要注意的是，`Stream`也具有相应的`FusedStream`trait。实现了`FusedStream`或经`.fuse()`封装的Stream会通过`.next()`/`.try_next()`组合子产生具有`FusedFuture`trait的future。

```rust,edition2018
{{#include ../../examples/06_03_select/src/lib.rs:fused_stream}}
```

## 使用`Fuse`和`FuturesUnordered`在`select`循环中运行并发Task

有个存在感稍弱但是很好用的函数是`Fuse::terminated()`，通过它可以构造一个已经终止(_terminated_)的空future，且之后可在其中填充我们要运行的内容。如果一个task要在`select`循环中运行，而且在这个循环中才被创建，这个函数就有用了。

还有一个就是`.select_next_some()`函数。它可以与`select`共同使用，当Stream中返回的值为`Some(_)`时才运行该分支，而忽略`None`值。

```rust,edition2018
{{#include ../../examples/06_03_select/src/lib.rs:fuse_terminated}}
```

若同一future的多个拷贝需要同时运行，请使用`FuturesUnordered`类型。下面的例子和上面的差不多，但是会将`run_on_new_num_fut`的每个拷贝都运行至完成，而非在一份新的拷贝创建时就放弃执行。它会输出`run_on_new_num_fut`的返回的一个值。

```rust,edition2018
{{#include ../../examples/06_03_select/src/lib.rs:futures_unordered}}
```
