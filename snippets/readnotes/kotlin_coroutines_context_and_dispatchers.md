## Kotlin:Coroutines:Coroutines Context and Dispatchers

CoroutineContext = Job + Dispatcher

dispatcher => thread (confine / unconfine)

- `launch{}` `async{}` can accept optional param `CoroutineContext` 
- no param，inherit context

---

- `Dispatchers.Unconfined` thread changed by suspend function, this is special usage
- confined dispatcher thread not changed

Debug: `-Dkotlinx.coroutines.debug` JVM option can show coroutine info

`CoroutineContext.use{}` can release threads automatically

`witchContext(){}` can switch thread for coroutine execution

You can retrieve job with `coroutineContext[Job]` 

launch new coroutine in coroutine, new coroutine's job is child of outer coroutine's job. job can be cancelled   recursively

parent will wait for all children  completion automatically

`CoroutineName` for debug

combine multi coroutine contexts with `+`

`MainScope`  `Activity : CoroutineScope by CoroutineScope(Dispatchers.Default)`

`ThreadLocal.asContextElement(value = xx)`





