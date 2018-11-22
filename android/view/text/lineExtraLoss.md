### 行间距失效问题

问题描述：

在 App 中正常浏览一段时间后，某些文本行间距失效。问题一旦出现，新打开的页面也会有这个问题，必须通过杀掉进程才能恢复。

问题分析：

初步分析：

根据现象大胆猜测是触发了特定条件导致某个静态对象状态改变，否则不会杀掉进程才能恢复。

具体分析：

要找行间距失效的原因，我选择 TextView 的 draw() 方法作为切入点，因为这里是将字符展示在屏幕上的最后一步，肯定是有问题的。看代码可知 Layout.draw() 方法负责字符绘制，进入 Layout 类具体绘制方法是 drawText()

```
public void drawText() {
	...
	for() { // 遍历每一行
		...
		int ltop = previousLineBottom;
		int lbottom = getLineTop(lineNum + 1);
		previousLineBottom = lbottom;
		int lbaseline = lbottom - getLineDescent(lineNum);
		...
	}
	...
}
```
由这块代码可以发现行间距是包含在 getLineDescent(lineNum) 中的，换句话说行间距属于上一行。这个结论也可以根据 TextView 的 getLineHeight() 方法得出：

```
public int getLineHeight() {
    return FastMath.round(mTextPaint.getFontMetricsInt(null) * mSpacingMult + mSpacingAdd);
}
```

所以行间距失效的 TextView 中，mLayout 的 getLineDescent() 方法肯定有问题。Layout 类是个抽象类，负责 TextView 的文字布局，
实现类有 StaticLayout 和 DynamicLayout，具体使用哪个取决于 text 是否为 Spannable，如果是 Spannable 使用 DynamicLayout 否则是 StaticLayout。通过添加调试代码发现出现问题的 TextView 都是用的 DynamicLayout 来布局，所以继续看看 DynamicLayout 的 getLineDescent() 是否正常。

DynamicLayout 中代码如下：

```
@Override
public int getLineDescent(int line) {
    return mInts.getValue(line, DESCENT);
}
```

可以看出 lineDescent 是存在 mInts 这个 int 数组中，继续找计算的代码如下：

```
int desc = reflowed.getLineDescent(i);
if (i == n - 1)
    desc += botpad;

ints[DESCENT] = desc;
```

上面代码最后一行将计算的 descent 数值存在 ints 这个 int 数组中，可以看到 DynamicLayout 的 lineDescent 计算是委托给 reflowed 这个对象。这个 reflowed 是 StaticLayout 对象，继续看它的 getLineDescent() 方法。

```
@Override
public int getLineDescent(int line) {
    return mLines[mColumns * line + DESCENT];
}
```

和 DynamicLayout 的 getLineDescent() 类似，lineDescent 值是存在一个 int 数组中，继续查找计算 lineDescent 的代码

```
...
boolean lastLine = mEllipsized || (end == bufEnd);

if (firstLine) {
    if (trackPad) {
        mTopPadding = top - above;
    }

    if (includePad) {
        above = top;
    }
}

int extra;

if (lastLine) {
    if (trackPad) {
        mBottomPadding = bottom - below;
    }

    if (includePad) {
        below = bottom;
    }
}

if (needMultiply && !lastLine) {
    double ex = (below - above) * (spacingmult - 1) + spacingadd;
    if (ex >= 0) {
        extra = (int)(ex + EXTRA_ROUNDING);
    } else {
        extra = -(int)(-ex + EXTRA_ROUNDING);
    }
} else {
    extra = 0;
}

lines[off + START] = start;
lines[off + TOP] = v;
lines[off + DESCENT] = below + extra;
...
```

上面代码中的 extra 应该就是行间距的值，通过调试发现问题出在 lastLine 上，即使不是最后一行 lastLine 也为 true。lastLine 的赋值如下：

```
boolean lastLine = mEllipsized || (end == bufEnd);
```
end == bufEnd 不总为 true, 而 mEllipsized 总是 true。通过在 StaticLayout.java 文件中查找 "mEllipsized =" 发现赋值代码只有一处在 calculateEllipsis() 方法中，这个方法负责 TextView 省略号的显示逻辑，当需要显示时 mEllipsized 被设置为 true，但是 StaticLayout 中却没有将 mEllipsized 设置为 false 的地方。这样的逻辑在单次布局计算时是没有问题的，但当 StaticLayout 对象被复用时就会出错，因为 mEllipsized 没有被恢复 false 的逻辑。再回到 DynamicLayout 类中可以看到用来计算 lineDescent 的 StaticLayout 对象正是复用的，代码如下：

```
// generate new layout for affected text

StaticLayout reflowed;
StaticLayout.Builder b;

synchronized (sLock) {
    reflowed = sStaticLayout;
    b = sBuilder;
    sStaticLayout = null;
    sBuilder = null;
}

if (reflowed == null) {
    reflowed = new StaticLayout(null);
    b = StaticLayout.Builder.obtain(text, where, where + after, getPaint(), getWidth());
}
```

综上所述，这个问题是 Android 官方代码 StaticLayout 的 bug。引入这个 bug 的版本是 sdk 26，在 sdk 27 的源码里增加了将 mEllipsized 重置为 false 的代码，这也正说明了 sdk 26 里的实现确实是个 bug。

找到问题后，规避这个 bug 很简单，只要将 DynamicLayout 中的静态变量 reflowed 设置为空，每次用到时候创建新的 StaticLayout 对象就可以了。代码如下：

```

@Nullable
private Pair<Integer, Reflect> layoutReflect;

@Override
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    super.onLayout(changed, left, top, right, bottom);
    Layout layout = getLayout();
    if (layout != null) {
        // Issue: 全局性行间距失效
        // Reason: Sdk 26 DynamicLayout 的 sStaticLayout 成员变量中的 mEllipsized 一旦设为 true 无法恢复
        if (SdkChecker.eq(26) && layout instanceof DynamicLayout) {
            nullRecycledStaticLayout((DynamicLayout) layout);
        }
    }
}

private void nullRecycledStaticLayout(DynamicLayout dl) {
    try {
        final int dlHash = dl.hashCode();
        Reflect dlReflect;
        if (layoutReflect != null && layoutReflect.first != null && layoutReflect.second != null
                && layoutReflect.first == dlHash) {
            dlReflect = layoutReflect.second;
        } else {
            dlReflect = Reflect.on(dl);
            layoutReflect = new Pair<>(dlHash, dlReflect);
        }
        dlReflect.set("sStaticLayout", null);
    } catch (Exception e) {
        JLog.e(e);
    }
}
```
这里用到了反射来将 sStaticLayout 设置为空，这里用到的反射类库是 [JOOR](https://github.com/jOOQ/jOOR)。因为用到反射，混淆文件也要加上下面这行，防止被混淆。

```
-keep public class android.text.DynamicLayout { <fields>; }
```

