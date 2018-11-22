android 获取`?attr/`类属性的代码字段：

```
public static int getPrimaryColor(Activity activity) {
        TypedValue typedValue = new TypedValue();
        activity.getTheme().resolveAttribute(R.attr.colorPrimary, typedValue, true);
        return typedValue.data;
}
```