### 多列表ViewPager实现TabLayout吸顶效果 ###

这是怎样一种场景呢？如果你有用即刻App的话(没用的话就下载一个，哈哈)，可以打开最近热门页面。

[欠一张动图]

像上图，这个页面首先由上下两部分组成，上部是标题和指示当前选中页面的TabLayout，下部是由四个列表组成的可左右切换的ViewPager。所谓的吸顶效果就是不管滑动下部哪个列表，顶部的标题都会跟随滑动显示或者隐藏。但是指示页面选中的TabLayout是一直显示的，就好像吸在顶部一样。

我们使用Android官方的解决方案，代码如下：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.ruguoapp.jike.view.widget.JViewPager
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <android.support.design.widget.AppBarLayout
        android:id="@+id/lay_app_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/ThemeOverlay.AppCompat.ActionBar">

        <android.support.design.widget.CollapsingToolbarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:layout_scrollFlags="scroll|exitUntilCollapsed|enterAlways"
            app:titleEnabled="false">

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

            <android.support.design.widget.TabLayout
                android:id="@+id/tab"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:layout_marginTop="?attr/actionBarSize" />
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>
</android.support.design.widget.CoordinatorLayout>
```