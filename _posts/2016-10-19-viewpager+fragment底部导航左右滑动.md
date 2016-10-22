---
layout: post
title: Android基本界面模型
categories: android
description: Android底部导航，左右滑动开发程序
keywords: android, java
---

由于工作需要和兴趣爱好，java web开发的同时自己也喜欢写一些Android小app，最近整理了这个Viewpager+Fragment实现的底部导航、左右滑动的小基本框架，方便以后再开发类似app的时候直接拿来就用

# 页面布局

```xml

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <RelativeLayout
        android:id="@+id/ly_top_bar"
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:background="@color/my_blue">

        <TextView
            android:id="@+id/tv_top"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_centerInParent="true"
            android:gravity="center"
            android:text="@string/title_bar"
            android:textColor="@color/white"
            android:textSize="18sp" />

    </RelativeLayout>

    <RadioGroup
        android:id="@+id/radio_group"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_marginBottom="4dp"
        android:background="@color/white"
        android:orientation="horizontal">

        <RadioButton
            android:id="@+id/rb_clock"
            style="@style/tab_menu_item"
            android:drawableTop="@drawable/selector_tab_clock"
            android:text="@string/clock" />

        <RadioButton
            android:id="@+id/rb_todo"
            style="@style/tab_menu_item"
            android:drawableTop="@drawable/selector_tab_todo"
            android:text="@string/todo_tab" />

        <RadioButton
            android:id="@+id/rb_more"
            style="@style/tab_menu_item"
            android:drawableTop="@drawable/selector_tab_more"
            android:text="@string/more" />
    </RadioGroup>

    <View
        android:id="@+id/div_tab_bar"
        android:layout_width="match_parent"
        android:layout_height="2px"
        android:layout_above="@id/radio_group"
        android:background="@color/white" />

    <android.support.v4.view.ViewPager
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_above="@id/div_tab_bar"
        android:layout_below="@id/ly_top_bar">

    </android.support.v4.view.ViewPager>


</RelativeLayout>


```

基本就是顶部一个layout显示工具栏，中间一个android.support.v4.view.ViewPager用于加载管理Fragment，底部是一个RadioGroup，直接实现单选并联动进行ViewPager的滑动实现；


# 自定义FragmentPagerAdapter

```java

package cn.xz.mytodo.fragment;

import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentPagerAdapter;
import android.view.ViewGroup;

import cn.xz.mytodo.MainActivity;
import cn.xz.mytodo.util.MLog;

public class MyFragmentPagerAdapter extends FragmentPagerAdapter {

    private final int PAGE_COUNT = 3;
    private ClockFragment clockFragment = null;
    private TodoFragment todoFragment = null;
    private MoreFragment moreFragment = null;

    public MyFragmentPagerAdapter(FragmentManager fm) {
        super(fm);
        clockFragment = new ClockFragment();
        todoFragment = new TodoFragment();
        moreFragment = new MoreFragment();
    }

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        MLog.log("position:" + position);
//        MLog.log("container:" + container);
        return super.instantiateItem(container, position);
    }

    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        MLog.log("destroyItem:" + position);
        super.destroyItem(container, position, object);
    }

    @Override
    public Fragment getItem(int position) {
        Fragment fragment = null;
        switch (position) {
            case MainActivity.PAGE_CLOCK: {
                fragment = clockFragment;
                break;
            }
            case MainActivity.PAGE_TODO: {
                fragment = todoFragment;
                break;
            }
            case MainActivity.PAGE_MORE:
                fragment = moreFragment;
                break;
        }
        return fragment;
    }

    @Override
    public int getCount() {
        return PAGE_COUNT;
    }
}


```

底部导航栏和中间ViewPager对应几个Fragment就在自定义FragmentPagerAdapter中定义相对应的Fragment类的实例对象，并在getItem中进行对应判断，这里强调下重写`public Fragment getItem(int position)`方法时需要注意`position`参数对应的值是从0开始的。


# 主activity内容

```java

package cn.xz.mytodo;

import android.os.Bundle;
import android.support.v4.app.FragmentActivity;
import android.support.v4.app.FragmentManager;
import android.support.v4.view.ViewPager;
import android.view.KeyEvent;
import android.view.View;
import android.widget.RadioButton;
import android.widget.RadioGroup;
import android.widget.TextView;

import cn.xz.mytodo.fragment.MyFragmentPagerAdapter;
import cn.xz.mytodo.util.MLog;
import cn.xz.mytodo.util.MToast;

public class MainActivity extends FragmentActivity
        implements RadioGroup.OnCheckedChangeListener, ViewPager.OnPageChangeListener {

    //计数是从零开始！
    /**
     * 番茄钟
     */
    public static final int PAGE_CLOCK = 0;
    /**
     * 待办列表
     */
    public static final int PAGE_TODO = 1;
    /**
     * 更多
     */
    public static final int PAGE_MORE = 2;

    private long mExitTime = 0;
    private ViewPager viewPager;
    private MyFragmentPagerAdapter mAdapter;

    private TextView tvTop;

    private RadioGroup radioGroup;
    private RadioButton rbClock, rbTodo, rbMore;

    private <T extends View> T bindView(int viewId) {
        try {
            return (T) findViewById(viewId);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tvTop = bindView(R.id.tv_top);

        radioGroup = bindView(R.id.radio_group);
        radioGroup.setOnCheckedChangeListener(this);

        rbClock = bindView(R.id.rb_clock);
        rbTodo = bindView(R.id.rb_todo);
        rbMore = bindView(R.id.rb_more);

        FragmentManager supportFragmentManager = getSupportFragmentManager();
        MLog.log(supportFragmentManager);
        mAdapter = new MyFragmentPagerAdapter(supportFragmentManager);

        viewPager = bindView(R.id.view_pager);
        viewPager.setOffscreenPageLimit(5);
        viewPager.setAdapter(mAdapter);
        viewPager.addOnPageChangeListener(this);
        viewPager.setCurrentItem(0);

        //初始化显示实时同步
        rbClock.setChecked(true);
    }

    //---onCheckedChanged implement start
    @Override
    public void onCheckedChanged(RadioGroup group, int checkedId) {
        switch (checkedId) {
            case R.id.rb_clock:
                viewPager.setCurrentItem(PAGE_CLOCK);
                tvTop.setText(getText(R.string.clock));
                break;
            case R.id.rb_todo:
                viewPager.setCurrentItem(PAGE_TODO);
                tvTop.setText(getText(R.string.todo));
                break;
            case R.id.rb_more:
                viewPager.setCurrentItem(PAGE_MORE);
                tvTop.setText(getText(R.string.more));
                break;
        }
    }
    //---onCheckedChanged implement stop

    //---OnPageChangeListener implement start
    //重写ViewPager页面切换的处理方法
    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
        //整个page scroll的过程都会触发
        //MLog.log("onPageScrolled:" + position);
    }

    @Override
    public void onPageSelected(int position) {
        MLog.log("onPageSelected:" + position);
    }

    @Override
    public void onPageScrollStateChanged(int state) {
        //state的状态有三个，0表示什么都没做，1正在滑动，2滑动完毕
        if (state == 2) {
            switch (viewPager.getCurrentItem()) {
                case PAGE_CLOCK:
                    rbClock.setChecked(true);
                    break;
                case PAGE_TODO:
                    rbTodo.setChecked(true);
                    break;
                case PAGE_MORE:
                    rbMore.setChecked(true);
                    break;
            }
        }
    }
    //---OnPageChangeListener implement stop

    //连续按两次退出系统
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_BACK) {
            if ((System.currentTimeMillis() - mExitTime) > 2000) {
                MToast.Show(this, "再按一次退出程序！");
                mExitTime = System.currentTimeMillis();
            } else {
                finish();
                System.exit(0);
            }
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }
}


```

主要是在这个activity中进行ViewPager、MyFragmentPagerAdapter、RadioGroup和几个RadioButton的操作，activity需要继承`FragmentActivity`，同时还实现了`RadioGroup.OnCheckedChangeListener`，`ViewPager.OnPageChangeListener`两个接口，重写了对应的`onCheckedChanged`，`onPageScrolled`，`onPageSelected`，`onPageScrollStateChanged`方法进行自定义逻辑判断

# 后续需要整理

- 整体设计还不够mvp
- 尝试使用JakeWharton的butterknife，8.4.0，但是在Fragment中调用`ButterKnife.bind(this, view);`之后再使用界面控件时总是报null，即初始化控件失败
- 找时间commit到我的github上


