###code###

```
Spannable spannable = textView.getText();
return spannable.getSpans(0, spannable.length(), UnderlineSpan.class).length > 0;
```