###情境
`ViewPager (background="@color/black")`  
`PhotoView (width="match_parent" height="match_parent")` 作为`ViewPager`item View的子View,当PV手势缩小时，ViewPager的黑色背景作为填充PV空白部分的颜色，防止用户看到下面Activity的内容。

###问题
当硬件加速打开时，一切如预期。但关闭硬件加速后，PV缩放后，可以看见下面Activity的内容。
###分析
硬件加速关闭时，ViewPager直接被充满屏幕的PV覆盖，缩放PV图片，空白部分直接透明
###解决方法
硬件加速关闭时
`<PhotoView background="@color/black" />`
