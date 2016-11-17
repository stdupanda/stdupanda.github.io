---
layout: post
title: 常见Android开发问题整理
categories: Android
description: 常见Android开发问题整理
keywords: Android
---

记录常见Android开发问题并整理其解决方法。

# 界面控件相关

## `EditText` 的 `OnClick` 弹出键盘

有时候会在 `EditText` 上设置 `OnClick` 事件，实现点击弹出`DatePickerDialog` 或者进行其他后续操作，但是实际测试会出现点击后弹出键盘的问题，解决方法如下：

```java

editText.setFocusable(false);//如此设置即可实现点击控件不弹出键盘
...
@OnClick(R.id.et_todo_expire_date)
void showExpireDate() {//点击弹出日期选择对话框
    Calendar calendar = Calendar.getInstance();
    DatePickerDialog datePickerDialog = new DatePickerDialog(this, new DatePickerDialog.OnDateSetListener() {
        @Override
        public void onDateSet(DatePicker view, int year, int monthOfYear, int dayOfMonth) {
            //...
        }
    }, calendar.get(Calendar.YEAR), calendar.get(Calendar.MONTH), calendar.get(Calendar.DAY_OF_MONTH));
    datePickerDialog.show();
}

```

# 业务处理相关

## `getResources().getColor()` 过时

使用 `ContextCompat.getColor(this,R.color.white);` 即可从xml中获取颜色。

## `AlertDialog` 自定义布局

```java

View view = View.inflate(getActivity(), R.layout.view_dialog, null);
AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
builder.setCancelable(true)
        .setView(view);
final AlertDialog alertDialog = builder.create();
TextView tvDel = (TextView) view.findViewById(R.id.tv_dialog_del);

tvDel.setOnClickListener(new View.OnClickListener() {//删除
    @Override
    public void onClick(View v) {
        //...
        alertDialog.dismiss();
    }
});

alertDialog.show();
```