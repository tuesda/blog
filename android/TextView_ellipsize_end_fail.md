TextView ellipsize:end 设置失效

TextView字符串中如果有回车字符，会导致设置ellipsize:end失效。  
具体失效现象是：本应该显示在最后面的三个省略号，省略号后面会多出字符。

解决办法：

* 用`android:singleLine="true"`代替`android:lines="1"`或者`android:maxLines="1"`，不过singleLine已经deprecated所以不建议使用。
* `String.replaceAll("\n", "")`过滤掉内容中的所有回车，这个问题也可以解决，如果行数超过一行不能使用第一种方法，就只能用这种办法。