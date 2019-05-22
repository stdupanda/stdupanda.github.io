---
layout: post
title: Fragment中使用ButterKnife初始化view失败
categories: 归档
description: Fragment中使用ButterKnife初始化view失败，查找问题并解决
keywords: android, Android, ButterKnife
---

之前文章里写的 **[Android基本界面模型](https://stdupanda.github.io/2016/10/19/viewpager+fragment%E5%BA%95%E9%83%A8%E5%AF%BC%E8%88%AA%E5%B7%A6%E5%8F%B3%E6%BB%91%E5%8A%A8/)** 里提过，在Fragment中使用ButterKnife初始化view会提示空指针异常的问题，经过哥们们一起分析，是我的一个缺乏经验的低级操作失误。

# 代码中调用正常

在代码中按照作者的调用方法初始化绑定view的操作是正常的，例如：

- In Activity

```java
class ExampleActivity extends Activity {
  @BindView(R.id.title) TextView title;
  @BindView(R.id.subtitle) TextView subtitle;
  @BindView(R.id.footer) TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
    // TODO Use fields...
  }
}
```

- RESOURCE BINDING

```java
class ExampleActivity extends Activity {
  @BindString(R.string.title) String title;
  @BindDrawable(R.drawable.graphic) Drawable graphic;
  @BindColor(R.color.red) int red; // int or ColorStateList field
  @BindDimen(R.dimen.spacer) Float spacer; // int (for pixel size) or float (for exact value) field
  // ...
}
```

- NON-ACTIVITY BINDING,like in Fragment

```java
public class FancyFragment extends Fragment {
  @BindView(R.id.button1) Button button1;
  @BindView(R.id.button2) Button button2;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }
}
```

# Gradle 依赖库和插件配置
不再列举，详情见[butterknife官网](http://jakewharton.github.io/butterknife/ "Go to butterknife！")

重点在这里！

## 在 Gradle 配置中添加 ButterKnife 的依赖库

`compile 'com.jakewharton:butterknife:8.4.0'`

`apt 'com.jakewharton:butterknife-compiler:8.4.0'`

或者在 `Project Structure` -> `Dependency` 中添加 `Library dependency` ,输入

`om.jakewharton:butterknife:8.4.0`，需要选择两个：

`com.jakewharton:butterknife:8.4.0`

`com.jakewharton:butterknife-compiler:8.4.0`

按照作者的介绍，是在 Gradle 的配置文件中增加如下两行：

```
compile 'com.jakewharton:butterknife:8.4.0'
apt 'com.jakewharton:butterknife-compiler:8.4.0'
```

和通过IDE添加的结果不太一致，相差在 `compile`&`apt` 手动查了下,最后确定如下流程：

-  In your **project-level** `build.gradle` file:

```
buildscript {
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
  }
}
```

-  In your **module-level** `build.gradle` file:

```
apply plugin: 'android-apt'

android {
  ...
}

dependencies {
   compile 'com.jakewharton:butterknife:8.4.0'
   apt 'com.jakewharton:butterknife-compiler:8.4.0'
}
```

-  modify **module-level** `build.gradle` file

```
apply plugin: 'com.android.application'
apply plugin: 'android-apt'
```

最后整体sync一下就可以了


# 附记 #

compile与apt区别：是compile会编译到最后的APK或library，apt不会；

1. apt允许配置只在编译时作为注解处理器的依赖，而不添加到最后的APK或library
2. 设置源路径，使注解处理器生成的代码能被Android Studio正确的引用

参考[android-apt](http://www.jianshu.com/p/2494825183c5 "android-apt") 、[官网](https://bitbucket.org/hvisser/ "官网")

## 牢记！！！
**完全对照官网说明写！省的浪费时间！**
2016-11-05附记：版本更新后，直接按照作者github的readme文件内提示进行操作即可