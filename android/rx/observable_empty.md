`Observable.empty()`
###情景###
mock数据的时候需要模拟服务器返回一个列表为空的response。下意识会使用`Observable.empty()`，其实是不对的，这个Observable什么也不会返回，换句话说不会触发`doOnNext()`方法。这样处理代码得不到任何响应。
###解决办法###
使用下面代码作为模拟响应

```
Observable.create(subscriber -> {
	List<T> emptyList = Collections.emptyList();
	subscriber.onNext(emptyList);
	subscriber.onCompleted();
})
```