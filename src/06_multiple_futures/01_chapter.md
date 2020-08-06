# 单次执行多个Future

到目前为止，我们几乎都是用`.await`来执行Future的，其会阻塞当前的task，直到某个特定`Future`完成。但是，在实际的异步应用中常常需要并发执行多个不同的行动。

在这一章中，会涉及一些单次执行多个异步行动的方法：

- `join!`：等待future全部完成
- `select!`：等待几个future中的某个完成
- Spawning: creates a top-level task which ambiently runs a future to completion
- `FuturesUnordered`：一组能够返回子future结果的future

