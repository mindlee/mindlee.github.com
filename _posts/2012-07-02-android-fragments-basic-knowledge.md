---
title: Android Fragments基础
author: Wei Li
excerpt: 别看代码短，庙小妖风大，池浅王八多。不知道明察秋毫的你有没有注意，正宗的Activity 竟然没有用 setContentView() 设置根内容视图，这是要闹哪样，难道 setContentView() 被请去喝咖啡了？ 踏破铁鞋无觅处，那人却在灯火阑珊处……
layout: post
permalink: /2012/07/02/android-fragments-basic-knowledge/
views:
  - 15691
categories:
  - 技术启蒙
tags:
  - Android
  - Fragment
  - JAVA
---
###1）Fragments基础   

到 Android3.0+ 以后，Android新增了[Fragments](http://developer.android.com/guide/components/fragments.html)，在没有 Fragment 之前，一个屏幕只能放一个 Activity，Fragment 从功能上讲相当于一个子活动（Activity），它可以让多个活动放到同一个屏幕上，也就是对用户界面和功能的重用，因为对于大屏设备来说，纯粹的 Activity 有些力不从心，比如下边这张截图：
![Image][1]
选择的时候，左边的列表是重用的，如果只用 Activity，那么每次跳转，目标 Activity 的 XML 文件要重新布置一遍左边的 ListView，无疑很费劲，Fragment 就是用来解决这类问题，上图中，左边列表是Fragment_A，右边详细内容是 Fragment_B，把 Fragment_A 和 Fragment_B 放到同一个 Activity 中，点击选项时，不必更改 Fragment_A，每次只更改 Fragment_B 的布局，如此这般，就可以实现界面重用了；但是，上边的屏幕是横着的，万一竖过来呢，不用担心，利用 Fragment 可以使一个布局适用于多种情况，比如下图，详细见后边的示例。
![Image][2]
Fragment 像是一个子活动，但是 Fragment 不是 Activity 的扩展，因为 Fragment 扩展自 android.app 中的 Object，而 Activity 是 Context 的子类。Fragment 有自己的视图层级结构，有自己的活动周期，还可以像活动一样响应后退按钮，Fragment 还有一个用作其初始化参数的包（Bundle），类似 Activity，Fragment 也可由系统自动保存并在以后还原。当系统还原 Fragment 时，它调用默认的构造函数（没有参数），然后将此Bundle还原到新创建的 Fragment 中，所以无论新建还是还原 Fragment，都要经过两个步骤：

1. 调用默认构造函数
2. 传入新的或者保存起来的Bundle

一个Activity可以运行多个 Fragment，Fragment 切换时，由 FragmentTransaction 执行，切换时，上一个 Fragment 可以保存在后退栈中（Back Stack），这里的后退栈由 FragmentManager 来管理，注意 Fragment 和 Activity 的后退栈是有区别的：Activity 的后退栈由系统管理，而 Fragment 的后退栈由所在的Activity 管理。

###2）Fragments活动周期

实现 Fragment，应该先了解一下它的[活动周期](http://developer.android.com/reference/android/app/Fragment.html#Lifecycle)，如图：
![Image][3]

所以创建一个 Fragment 的顺序是：

####1）用静态方法实例化一个 Fragment：

（①调用构造函数；②传入 Bundle）

{% highlight java %}
public static DetailsFragment newInstance(int index) {
    DetailsFragment df = new DetailsFragment();
    Bundle args = new Bundle();
    args.putInt("index", index);
    df.setArguments(args);
    return df;
}
{% endhighlight %}

####2）然后按照需求实现上图中的12个回调函数:
	1. onAttach(Activity) called once the fragment is associated with its activity.

	2. onCreate(Bundle) called to do initial creation of the fragment.

	3. onCreateView(LayoutInflater, ViewGroup, Bundle) creates and returns the view hierarchy associated with the fragment.

	4. onActivityCreated(Bundle) tells the fragment that its activity has completed its own Activity.onCreate().

	5. onStart() makes the fragment visible to the user (based on its containing activity being started).

	6. onResume() makes the fragment interacting with the user (based on its containing activity being resumed).

	As a fragment is no longer being used, it goes through a reverse series of callbacks:

	1. onPause() fragment is no longer interacting with the user either because its activity is being paused or a fragment operation is modifying it in the activity.

	2. onStop() fragment is no longer visible to the user either because its activity is being stopped or a fragment operation is modifying it in the activity.

	3. onDestroyView() allows the fragment to clean up resources associated with its View.

	4. onDestroy() called to do final cleanup of the fragment’s state.

	5. onDetach() called immediately prior to the fragment no longer being associated with its activity.

具体每个方法的参数功能建议去看官方文档吧，对于理解 Fragment 的实现过程有帮助；一般使用时，只需实现上边的几个就可以了，比如：

1）onCreat(Bundle)，传入已经保存的 Bundle；

2）onCreateView(LayoutInflater, ViewGroup, Bundle) ，返回此 Fragment 的一个 View，LayoutInflater 用于扩充此碎片的布局，ViewGroup 指出它的要附属的视图，Bundles 保存状态包，示例：

{% highlight java %}
public View onCreateView(LayoutInflater inflater, ViewGroup container,
        Bundle savedInstanceState) {
    View v = inflater.inflate(R.layout.details, container, false);
    TextView text1 = (TextView) v.findViewById(R.id.text1);
    text1.setText(Shakespeare.DIALOGUE[mIndex]);
    return v;
}
{% endhighlight %}
这里要注意的是 false 参数值，它指的是，不将此 Fragment 的视图附加到回调中的 ViewGroup，因为系统默认已经执行过此操作。

3）[onActivityCreated(Bundle)](http://developer.android.com/reference/android/app/Fragment.html#onActivityCreated(android.os.Bundle))，告诉 Fragment，Activity 的视图层级结构已经准备好，还可以实例化 Fragment 的视图层次，这里是用户看到界面之前，可以对界面最后调整的地方。（It can be used to do final initialization once these pieces are in place, such as retrieving views or restoring state）。

###3）Fragments实例

一个完整例子，如果你看过官方文档，一定不陌生，完整源文件[GitHub](https://github.com/welon/FragmentBasics)了，主要的几个文件如下：Java文件，MainActivity.java，TitlesFragment.java，DeailsFragment.java，DetailsActivity.java；布局文件：main.xml，details.xml，上图：

它的横向模式的 Activity 布局：layout-land/main.xml：

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<!-- This file is res/layout-land/main.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#555555"
    android:orientation="horizontal" >
 
    <fragment
        android:id="@+id/titles"
        android:layout_width="0px"
        android:layout_height="match_parent"
        android:layout_weight="1"
        class="net.mindlee.android.fragment.basics.TitlesFragment"
        android:background="#00550033" />
 
    <FrameLayout
        android:id="@+id/details"
        android:layout_width="0px"
        android:layout_height="match_parent"
        android:layout_weight="2" />
 
</LinearLayout>
{% endhighlight %}
需要注意的地方有：

1. 新标记<fragment>，因为Fragment 不是视图，所以这里<fragment>标记只是一个占位符，不能将子标记放在布局 XML 文件中的<fragment>下；

2. Fragment 标记的 class 特性指定应用程序的标题的扩展类；

3. 下一个标记是 FrameLayout，而不是<fragment>，因为 Fragment 本身并不是视图，所以 Fragment 必须位于视图容器内，回头看本文第一张截图，右边的界面每次都可能有不同的布局，所以需要把 Fragment 放到 FrameLayout中，对于不同的选项，只需更换右边框架布局里的 Fragment 即可。


MainActivity.java
{% highlight java %}
package net.mindlee.android.fragment.basics;
 
import android.app.Activity;
import android.app.FragmentTransaction;
import android.content.Intent;
import android.content.res.Configuration;
import android.os.Bundle;
 
public class MainActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }
 
    // 是否是多窗口
    public boolean isMultiPane() {
        return getResources().getConfiguration().orientation 
                == Configuration.ORIENTATION_LANDSCAPE;
    }
 
    public void showDetails(int index) {
        if (isMultiPane()) {// 如果是横屏(多窗口), 使用DetailsFragment
            DetailsFragment details = (DetailsFragment) getFragmentManager()
                    .findFragmentById(R.id.details);
 
            if (details == null || details.getShownIndex() != index) {
                details = DetailsFragment.newInstance(index);
 
                FragmentTransaction ft = getFragmentManager()
                        .beginTransaction();
                ft.setTransition(FragmentTransaction.TRANSIT_FRAGMENT_FADE);
                ft.replace(R.id.details, details);
                ft.addToBackStack("details");
                ft.commit();
            }
        } else {// 如果是竖屏，调用DetailsActivity，将当前索引值存入intent
            Intent intent = new Intent();
            intent.setClass(this, DetailsActivity.class);
            intent.putExtra("index", index);
            startActivity(intent);
        }
    }
}
{% endhighlight %}
这个代码里最妙的地方在 showDetails()，无论如何，资源 R.id.details 都要用于 FrameLayout，上边的 main.xml 中，如果在一个布局中有一个 DetailsFragment，由于没有为它分配任何其他 ID，所以它将拥有FrameLayout 的 ID。如果布局中有 DetailsFragment，可以使用 findFragmentById() 询问FragmentManager，如果框架布局是空的，则返回 null，否则将提供当前的 DetailsFragment。然后对Fragment进行过渡操作，比如hide()，show()，remove()，add()，replace()等，官方文档有说明：

	At any time while your activity is running, you can add fragments to your activity layout. You simply need to specify a ViewGroup in which to place the fragment.

	To make fragment transactions in your activity (such as add, remove, or replace a fragment), you must use APIs from FragmentTransaction. You can get an instance of FragmentTransaction from your Activitylike this:

	FragmentManager fragmentManager = getFragmentManager()
	FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
	You can then add a fragment using the add() method, specifying the fragment to add and the view in which to insert it. For example:

	ExampleFragment fragment = new ExampleFragment();
	fragmentTransaction.add(R.id.fragment_container, fragment);
	fragmentTransaction.commit();
	The first argument passed to add() is the ViewGroup in which the fragment should be placed, specified by resource ID, and the second parameter is the fragment to add.

	Once you’ve made your changes with FragmentTransaction, you must call commit() for the changes to take effect.

TitlesFragment.java代码：

{% highlight java %}
package net.mindlee.android.fragment.basics;
 
import android.app.Activity;
import android.app.ListFragment;
import android.os.Bundle;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.ListView;
 
public class TitlesFragment extends ListFragment {
    private MainActivity myActivity = null;
    int mCurCheckPosition = 0;
 
    public void onAttach(Activity myActivity) {
        super.onAttach(myActivity);
        this.myActivity = (MainActivity) myActivity;
    }
 
    @Override
    public void onActivityCreated(Bundle icicle) {
        super.onActivityCreated(icicle);
        if (icicle != null) {
            mCurCheckPosition = icicle.getInt("curChoice", 0);
        }
 
        setListAdapter(new ArrayAdapter<String>(getActivity(),
                android.R.layout.simple_list_item_1, Shakespeare.TITLES));
 
        ListView lv = getListView();
        lv.setChoiceMode(ListView.CHOICE_MODE_SINGLE);
        lv.setSelection(mCurCheckPosition);
 
        myActivity.showDetails(mCurCheckPosition);
    }
 
    @Override
    public void onSaveInstanceState(Bundle icicle) {
        super.onSaveInstanceState(icicle);
        icicle.putInt("curChoice", mCurCheckPosition);
    }
 
    @Override
    public void onListItemClick(ListView l, View v, int pos, long id) {
        myActivity.showDetails(pos);
        mCurCheckPosition = pos;
    }
}
{% endhighlight %}

这段代码有两个地方需要注意。首先，对于 DetailsFragment 没有实现 onCreatView()，这怎么能行，不创建 Fragment 的视图哪来的 ListView ? 这是因为 DetailsFragment 扩展自 TitlesFragment ，而它已包含了一个 ListView 视图。ListFragment 的默认 onCreateView() 会创建此 ListView 并返回它。（思考一下如果手动实现 onCreateView()，可以把哪些代码转移入其中），ListView 的资源 ID 为 android.R.id.list1，如果需要获取它的引用，可以调用 getListView()，可在 onActivityCreated() 中完成。但是由于 ListFragment 不同于 ListView，所以没有直接将适配器附加到 ListView。必须用 ListFragment 的 SetListAdapter()方法（不知道你有没有回想起典型的ListView 是怎么操作）。

其次，[onSaveInstanceState()](http://developer.android.com/reference/android/app/Fragment.html#onSaveInstanceState(android.os.Bundle)) 方法是必不可少的，为了防止 Activity 的进程被终止，需要在 onSaveInstanceState() 中存储 Fragment 的状态，以备进程重启时使用，这个方法有可能在 onDestroy() 之前的任何时候被调用，比较奇葩，可以点击上边看官方文档解释。

DetailsFragment.java代码：
{% highlight java %}
package net.mindlee.android.fragment.basics;
 
import android.app.Fragment;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
 
public class DetailsFragment extends Fragment {
    private int mIndex = 0;
 
    public static DetailsFragment newInstance(int index) {
        DetailsFragment df = new DetailsFragment();
        Bundle args = new Bundle();
        args.putInt("index", index);
        df.setArguments(args);
        return df;
    }
 
    public static DetailsFragment newInstance(Bundle bundle) {
        int index = bundle.getInt("index", 0);
        return newInstance(index);
    }
 
    public void onCreate(Bundle myBundle) {
        super.onCreate(myBundle);
        mIndex = getArguments().getInt("index", 0);
    }
 
    public int getShownIndex() {
        return mIndex;
    }
 
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        View v = inflater.inflate(R.layout.details, container, false);
        TextView text1 = (TextView) v.findViewById(R.id.text1);
        text1.setText(Shakespeare.DIALOGUE[mIndex]);
        return v;
    }
}
{% endhighlight %}
DetailsFragment 是典型的 Fragment。使用 newInstance() 先实例化一个 Fragment，然后在 onCreateView 中返回 Fragment 的视图层次结构。要使这里的 Fragment 有不同布局，传入不同的 XML 文件即可，本例中资源 R.id.details 指定 DetailsFragment 的布局为 details.xml。

details.xml的代码：

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<!-- This file is res/layout/details.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
 
    <ScrollView
        android:id="@+id/scroller"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
 
        <TextView
            android:id="@+id/text1"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
    </ScrollView>
 
</LinearLayout>
{% endhighlight %}

无论横向模式还是纵向模式，都可以为 DetailsFragment 使用完全相同的布局文件，此布局不是用于 Activity，仅仅供 Fragment 显示文本。

纵向模式时的界面：
![Image][4]

DetailsActivity代码：
{% highlight java %}
package net.mindlee.android.fragment.basics;
 
import android.app.Activity;
import android.app.FragmentTransaction;
import android.content.res.Configuration;
import android.os.Bundle;
 
public class DetailsActivity extends Activity {
 
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
 
        if (getResources().getConfiguration().orientation 
                == Configuration.ORIENTATION_LANDSCAPE) {
            finish();
            return;
        }
 
        if (getIntent() != null) {
            //从intent取出索引值，利用此值实例化DetailsFragment
            DetailsFragment details = DetailsFragment.newInstance(getIntent()
                    .getExtras());
             
            FragmentTransaction ft = getFragmentManager()
                    .beginTransaction();
            ft.add(android.R.id.content, details).commit();
        }
    }
}
{% endhighlight %}

竖屏时的 Activity，别看代码短，庙小妖风大，池浅王八多。不知道明察秋毫的你有没有注意，正宗的 Activity 竟然没有用 setContentView() 设置根内容视图，这是闹哪样？ 难道 setContentView() 被请去喝咖啡了？ 踏破铁鞋无觅处，那人却在灯火阑珊处。原因在 FragmentTransactions 上的 add() 里，Fragment 添加到的视图容器被指定为资源 android.R.id.content ，这是 Activity 的顶级视图容器，因此当将 Fragment 的视图层次结构附加到此容器时，Fragment 的视图层次结构成为了 Activity 的唯一视图层次结构，这里 newInstance() 使用了与之前完全相同的 DetailsFragment 类来创建 Fragment（接受一个Bundle），然后将它附加到 Activity 视图层次结构的顶部，这样 Fragment 就会在这个新 Activity 中显示。

对比一下 MainActivity.java 中实例化 details 的代码，如下：

{% highlight java %}
//MainActivity.java:
DetailsFragment details = (DetailsFragment) getFragmentManager()
                    .findFragmentById(R.id.details);
                     
1)details = DetailsFragment.newInstance(index);
 
//DetailsActivity.java
2)DetailsFragment details = DetailsFragment.newInstance(getIntent()
                    .getExtras());
{% endhighlight %}

这是实例化一个 Fragment 的两种方法，但无论如何，前提是布局中有此 DetailsFragnment，所以findFragmentById 也是很关键的。   

此工程中的其他文件没特殊之处，比如AndroidManifext.xml文件。

完整工程@Github，代码打包：《[FragmentBasics](https://github.com/welon/FragmentBasics/downloads)》

参考资料：

[Android官方文档：Design，API Guides，Fragments；](http://developer.android.com/guide/components/fragments.html)

[Android官方文档：Develop，Reference，Fragment；](http://developer.android.com/reference/android/app/Fragment.html)

[Develops Blog：《The Android 3.0 Fragments API 》;](http://android-developers.blogspot.com/2011/02/android-30-fragments-api.html)

[《Pro Android 3》& 《Pro Android 4》;](http://book.douban.com/subject/10538023/)

[1]: /uploads/2012/07/fragment_screenshot.jpg
[2]: /uploads/2012/07/fragments_diff_modes.png
[3]: /uploads/2012/07/fragments_lifecycle.png
[4]: /uploads/2012/07/fragment_basics_landscape.png