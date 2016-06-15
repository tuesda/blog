###情境
文本输入框需要点击回车触发某个操作，但复制粘贴的文本中包含回车不希望出发这个操作
###解决办法
设置EditText的KeyListener的监听事件来触发需要的操作，而不是设置`TextChangeListener`监听最后一个字符是否为`\n`
###其它
如果输入文本不需要换行，可以使用EditText下面两个属性来实现:

```
android:inputType="text"
android:imeOptions="actionDone"
```
在代码里通过监听`EditText.OnEditorActionListener`来实现点击回车触发某个操作