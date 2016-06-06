###问题描述
需要去掉`Dialog`的背景shadow(就是灰色的背景)
###解决办法

```
dialog.getWindow().clearFlags(WindowManager.LayoutParams.FLAG_DIM_BEHIND);
```

###反思
之前Google了很久没找到正确答案，是因为一直在查
> how to disable window background
其实应该搜：
> how to remove Dialog shadow