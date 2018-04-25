#### 当变量 a 非空时并且满足条件 conditionA 时执行代码块 blockA

```
a?.takeIf { conditionA }?.let { blockA }
```
这里的坑在于 .let 前的 ? 容易忘记，如果没加这个 ? 会导致 a 为空时但 conditionA 满足时 blockA 也会被执行。

#### 创建Runnable实例

创建Runnable实例有两种方式

```
val runnable1 = {
	println("runnable1")
}
val runnable2 = Runnable {
	println("runnable2")
}
println("runnable1 class name ${runnable1 is Runnable}")
println("runnable2 class name ${runnable2 is Runnable}")

view.post(runnable1)
view.post(runnable2)
```
runnable1 和 runnable2 都可以作为 view.post() 的参数。不同的地方在于 runnable1 每次被传入方法 view.post() 时会构建一个新的 Runnable 对象，而 runnable2 不会，因为 runnable2 本来就是Runnable类型。

按照 runnable1 方式的隐患在于这个 runnable 无法从 view 的消息队列中移除，也就是无法取消，而 runnable2 不存在这个问题。


