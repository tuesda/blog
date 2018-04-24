#### 当变量 a 非空时并且满足条件 conditionA 时执行代码块 blockA

```
a?.takeIf { conditionA }?.let { blockA }
```
这里的坑在于 .let 前的 ? 容易忘记，如果没加这个 ? 会导致 a 为空时但 conditionA 满足时 blockA 也会被执行。