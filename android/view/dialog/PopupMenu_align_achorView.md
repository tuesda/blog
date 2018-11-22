通过`new PopupMenu(Context context, View achorView)`生成的PopupWindow只能处于anchorView下面不能覆盖在anchorView上面，像actionBar右边点更多那样。  
*解决办法*  
通过

```
new PopupMenu(context, anchor, GravityCompat.START,
 android.support.v7.appcompat.R.attr.actionOverflowMenuStyle, 0)
```
这样方式新建PopupMenu就可以实现系统actionBar那样的效果
