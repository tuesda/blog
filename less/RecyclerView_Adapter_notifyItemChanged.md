如果没有禁用RecyclerView的ItemChanged的动画，调用RecyclerView.Adapter.notifyItemChanged会导致这个位置的item创建一个新的view，这样的话，之前在View中以setTag方式保存的方式都会丢失。

禁用ItemChanged动画的方法：

```
((SimpleItemAnimator) RecyclerView.getItemAnimator())
.setSupportsChangeAnimations(false);
```

> reference:<http://stackoverflow.com/questions/29873859/how-to-implement-itemanimator-of-recyclerview-to-disable-the-animation-of-notify>