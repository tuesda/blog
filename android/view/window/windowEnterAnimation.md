情境：

```
需要同时执行window的Enter动画和window内View的动画
window动画通过styles里的android:windowEnterAnimation属性设置
View的动画通过代码来动态设置
```
> 例如：window的Alpha入场动画和window内View的Translation动画

问题：

```
sdk_int >= 21时，通过android:windowEnterAnimation设置的window动画会和View里面设置的动画冲突
```

解决办法：

```
sdk_int >= 21时，通过android:windowEnterTransition设置window动画;
sdk_int < 21时，用android:windowEnterAnimation设置
```