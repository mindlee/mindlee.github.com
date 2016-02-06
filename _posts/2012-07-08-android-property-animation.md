---
title: Android Property Animation
author: Wei Li
excerpt: Property Animation的本质是，使用TimeInterpolator和TypeEvaluator改变对象的属性，之后每一帧都调用getAnimatedValue()获取实时属性值，并生成每一帧的画面，连续显示构成动画。
meta_description: roperty Animation的本质是，使用TimeInterpolator和TypeEvaluator改变对象的属性，之后每一帧都调用getAnimatedValue()获取实时属性值，并生成每一帧的画面，连续显示构成动画。
layout: post
permalink: /2012/07/08/android-property-animation/
views:
  - 9162
categories:
  - 技术启蒙
tags:
  - Android
  - Animation
  - JAVA
---
### 1）Android中的动画系统  
Android 3.0之前，支持三种动画，逐帧动画（Frame-by-Frame Animation，aka，Drawable Animation），布局动画（Layout Animation），视图动画（View Animation），后两种又合称为补间动画（Tween Animation），Android3.0引入了一种新的动画系统：属性动画（Property Animation）。最新的sdk-docs中介绍动画时，介绍了[三种](http://developer.android.com/guide/topics/graphics/overview.html)：Property Animation，View Animation， Drawable Animation。

Drawable Animation的作用是连续展示一帧一帧的图片资源（存放在res/drawable），能设置的几乎只有间隔时间；View Animation的作用是改变整个View（必须是View）的绘制效果，比如缩放，旋转，平移，alpha透明度等等，但是它的实际属性值没有改变，比如你缩小了 Button 的大小，它的有效点击区域却不变，因为它的位置和大小属性没变动。

### 2）Property Animation原理

到 Protery Animation，它还可以对 non-View 对象生成动画，改变的是对象的实际属性，比如 Button 缩放，它的位置和大小属性值随之更改，不仅功能比前两种强大，生成动画也更加稳定。因此，官方文档推荐使用 Property Animation。Property中可以更改的属性值有：（文档有详细介绍）

1. Duration：动画持续时间；
2. TimeInterpolation：插值器告诉动画某个属性（比如颜色渐变）如何随时间变化；
3. Repeat count and behavior：动画的重复次数与方式；
4. Animator sets：动画集合，可以打包多个动画，并且控制它们的时序；
5. Frame refresh delay：多少时间刷新一次，即每隔多少时间计算一次属性值，默认为10ms，最终刷新时间还受系统进程调度与硬件的影响 。

Protery Animation的工作流程如图：
![Image](http://mindlee.net/wp-content/uploads/2012/07/valueanimator.png)

可以看到一个 ValueAnimator（Value 动画生成器）里包括，一个TimeInterpolator（插值器，稍后会有介绍），一个TypeEvaluator（类型求值器，稍后会介绍），duration（持续时间），startPropertyValue（动画开始时的属性值），endPropertyValue（结束时的属性值），start()（动画开始）；

如果要修改动画属性值，需要实现接口[ValueAnimator.AnimatorUpdateListener](http://developer.android.com/reference/android/animation/ValueAnimator.AnimatorUpdateListener.html)，这个接口中包含一个抽象方法：[onAnimationUpdate()](http://developer.android.com/reference/android/animation/ValueAnimator.AnimatorUpdateListener.html#onAnimationUpdate(android.animation.ValueAnimator))，这个方法在动画中的每一帧都会被调用，通过监听这个事件，在属性值更新时执行相应的操作，当然在获取变更的值时需要调用 [getAnimatedValue()](http://developer.android.com/reference/android/animation/ValueAnimator.html#getAnimatedValue() ) ，即 Property Animation的本质是，使用TimeInterpolator和TypeEvaluator改变对象的属性，之后每一帧都调用getAnimatedValue()获取实时属性值，并生成每一帧的画面，连续显示构成动画。

另外，也可以继承[AnimatorListenerAdapter](http://developer.android.com/reference/android/animation/AnimatorListenerAdapter.html)类，而不是实现[Animator.AnimatorListener](http://developer.android.com/reference/android/animation/Animator.AnimatorListener.html)接口，用来简化工作，因为接口需要实现里边包含的所有方法（包含onAnimationStart()，onAnimationEnd()，onAnimationRepeat()，onAnimationCancel()四个方法），而[AnimatorListenerAdapter](http://developer.android.com/reference/android/animation/AnimatorListenerAdapter.html)可以选择性的重写需要的方法，相关详细内容可以看[官方文档](http://developer.android.com/guide/topics/graphics/prop-animation.html#listeners)。

### 3）TimeInterpolator

需要先介绍一下插值器，插值器告诉动画某个属性如何随时间变化，它以线性方式变化，还是以指数方式变化，还是先快后慢等等。支持的插值器包括：

	AccelerateDecelerateInterpolator(先加速后减速)
	An interpolator whose rate of change starts and ends slowly but accelerates through the middle.

	AccelerateInterpolator(加速)
	An interpolator whose rate of change starts out slowly and then accelerates.

	AnticipateInterpolator(先反方向，后正方向)
	An interpolator whose change starts backward then flings forward.

	AnticipateOvershootInterpolator(先反向，后向前超越目的值，再移回目的值)
	An interpolator whose change starts backward, flings forward and overshoots the target value, then finally goes back to the final value.

	BounceInterpolator(跳跃，到目的值后有反弹效果)
	An interpolator whose change bounces at the end.

	CycleInterpolator(循环，循环指定次数)
	An interpolator whose animation repeats for a specified number of cycles

	DecelerateInterpolator(减速)
	An interpolator whose rate of change starts out quickly and and then decelerates.

	LinearInterpolator(线性，均匀改变)
	An interpolator whose rate of change is constant.

	OvershootInterpolator(超越，超过目的值然后返回)
	An interpolator whose change flings forward and overshoots the last value then comes back.

	TimeInterpolator(一个接口，允许自定义 interpolator)
	An interface that allows you to implement your own interpolator.

接下来用一个实例来介绍 Propery Animation的相关使用方法，视频演示：（密码是123，o(╯□╰)o，拍的方向反了），代码打包最后有链接。

<div align="center">
<embed src="http://player.youku.com/player.php/sid/XNDI0OTU5MDY4/v.swf" allowFullScreen="true" quality="high" width="600" height="500" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash" >
</div>
虽然很明显，但是上边视频中的XML布局文件为：（后边例子中会用到），需要提前说明的是，Android手机中坐标的原点是左上角（0, 0）， 右下角是（width， height）。

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical"
    tools:context=".PropertyAnimationActivity" >
 
    <Button
        android:id="@+id/btn_st_animation1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="testObjectAnimator"
        android:text="Object Animator" />
 
    <Button
        android:id="@+id/btn_st_animation2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="testAnimationSet"
        android:text="Animation Set" />
 
    <Button
        android:id="@+id/btn_st_animation3"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="testAnimationXML"
        android:text="AnimationXML" />
 
    <Button
        android:id="@+id/btn_st_animation4"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="testPropertyValuesHolder"
        android:text="PropertyValuesHolder" />
 
    <Button
        android:id="@+id/btn_st_animation5"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="testViewPropertyAnimator"
        android:text="ViewPropertyAnimator" />
 
    <Button
        android:id="@+id/btn_st_animation6"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="testTypeEvaluator"
        android:text="Type Evaluator" />
 
    <Button
        android:id="@+id/btn_st_animation7"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="testKeyFrames"
        android:text="Key Frames" />
 
    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content" >
 
 
        <TextView
            android:id="@+id/tv_id"
            android:layout_width="fill_parent"
            android:layout_height="178dp"
            android:background="@color/blue"
            android:text="@string/hello"
            android:textColor="@color/white" />
    </LinearLayout>
 
</LinearLayout>
{% endhighlight %}
###4）Objet Animator

ObjectAnimator 是 ValueAnimator的一个子类，一般使用较多的是 ObjectAnimator，因为它不必实现ValueAnimator.AnimatorUpdateListener，它的属性值可以自动更新。使用时，指定一个对象及该对象的属性值，当属性值计算完成时，会自动设置为该对象的相应属性。使用ObjectAnimator要满足的条件如下：

1）对象必须有一个setter方法：set<PropertyName>（驼峰命名法）
2）比如下边的代码中的ofFloat方法，第一个参数为对象名，第二个为属性名（PropertyName，String类型），后边为可变参数，如果后边一个值，则默认是属性目标值；如果是两个值，则是起始值和目标值；要获得当前属性值，该对象要有相应属性的getter方法：get<PropertyName>。getter方法和setter方法需有相同的
参数类型。

上边视频中的第一个按钮，对应的就是ObjectAnimator，基础的的动画操作，代码：

{% highlight java %}
/*
 * 单个动画
 */
public void testObjectAnimator(View btnView) {
    float width = m_tv.getWidth();
    if (m_tv.getX() == 0) {
        ObjectAnimator translationRight = ObjectAnimator.
                ofFloat(m_tv, "X", width);
        translationRight.setDuration(1500);
        translationRight.start();
    } else {
        ObjectAnimator translationLeft = 
                ObjectAnimator.ofFloat(m_tv, "X", 0f);
        translationLeft.setDuration(1500);
        translationLeft.start();
    }
}
{% endhighlight %}

每次点击，只实现一个动画，右移或者左移，持续时间为1500毫秒；

另外，根据应用动画的对象或属性的不同，可能需要在 onAnimationUpdate() 中调用 invalidate() 方法来强制刷新视图。

还有一个不得不提的是，在 Fragment（碎片）上实现自定义动画的机制也要依靠ObjectAnimator类，相关内容可以去Google下。

上边代码改变的是对象的X值，补充一下Property Animation中可以设置的属性值：

>translationX and translationY: These properties control where the View is located as a delta from its left and top coordinates which are set by its layout container. (平移的距离）

>rotation, rotationX, and rotationY: These properties control the rotation in 2D (rotation property) and 3D around the pivot point. （旋转，支点是（pivotX, pivotY），rotation用于2D旋转，3D用到后两个）。

>scaleX and scaleY: These properties control the 2D scaling of a View around its pivot point. （缩放）

>pivotX and pivotY: These properties control the location of the pivot point, around which the rotation and scaling transforms occur. By default, the pivot point is located at the center of the object. (设置支点，默认支点在对象的正中央)

>x and y: These are simple utility properties to describe the final location of the View in its container, as a sum of the left and top values and translationX and translationY values. （X轴Y轴坐标）

>alpha: Represents the alpha transparency on the View. This value is 1 (opaque) by default, with a value of 0 representing full transparency (not visible).（透明度，1代表完全可见，0代表不可见）

### 5）Animation Set

利用 Animation Set，可以播放多个动画，并且可以控制它们的时序关系，比如同时播放，顺序播放等。设置顺序有两种方法：

第一种是使用 playSequentially() 和 playTogether()，分别表示顺序执行和同时执行，比如下边代码中注释掉的内容；

第二种是使用play()方法，它返回一个叫做 AnimatorSet.Builder 的类，这个类中的方法包括 after(animator)，before(animator)，with(animator), 分别表示之后，之前和同时进行。

上边视频，第二个按钮实现的效果：左上角到右下角，再从右下角到左上角，就是上下左右四个放心利用Animation Set组合形成。

{% highlight java %}
//连续动画
public void testAnimationSet(View v) {
    float width = m_tv.getWidth();
    float height = m_tv.getHeight();
    ObjectAnimator translationRight = ObjectAnimator.ofFloat(m_tv, "X", width);
    ObjectAnimator translationLeft = ObjectAnimator.ofFloat(m_tv, "X", 0f);
    ObjectAnimator translationDown = ObjectAnimator.ofFloat(m_tv, "Y",
            height);
    ObjectAnimator translationUp = ObjectAnimator.ofFloat(m_tv, "Y", 0);
 
    AnimatorSet as = new AnimatorSet();
    as.play(translationRight).before(translationLeft);
    as.play(translationRight).with(translationDown);
    as.play(translationLeft).with(translationUp);
 
    // 和上边四句等效，另一种写法
    /*
    AnimatorSet as = new AnimatorSet();
    as.playTogether(translationRight, translationDown);
    as.playSequentially(translationRight, translationLeft);
    as.playTogether(translationLeft, translationUp);
    */
    as.setDuration(1500);
    as.start();
}
{% endhighlight %}
###6）使用XML加载动画

Property Animation可以使用XML声明动画，这样做的好处是动画代码的重用，符合 DRY 原则（Don’t Repeat Yourself），将定义好的动画文件放到/res/animator/目录下，注意不是/res/anim/目录，/res/anim/是non-Property Animation的动画目录，新增加的/res/animator/专为Property Animation预留。引用方式为：

	In Java: R.animator.filename
	In XML: @[package:]animator/filename

比如上边第三个按钮中的，淡入淡出效果的XML文件：

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<!-- /res/animator/fadein.xml -->
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:ordering="sequentially" >
 
    <objectAnimator
        android:duration="2000"
        android:interpolator="@android:interpolator/accelerate_cubic"
        android:propertyName="alpha"
        android:valueFrom="1"
        android:valueTo="0"
        android:valueType="floatType" />
    <objectAnimator
        android:duration="2000"
        android:interpolator="@android:interpolator/accelerate_cubic"
        android:propertyName="alpha"
        android:valueFrom="0"
        android:valueTo="1"
        android:valueType="floatType" />
 
</set>
{% endhighlight %}

里边会用到的标签不多，对应关系如下：

	ValueAnimator-<animator>
	ObjectAnimator-<objectAnimator>
	AnimatorSet-<set>

关于Property Animation中XML的更多资料，可以去[这里](http://developer.android.com/guide/topics/resources/animation-resource.html#Property)看。

加载XML动画文件的代码很简单，第三个按钮代码：

{% highlight xml %}
/*
 * XML，便于代码重用
 */
public void testAnimationXML(View bView) {
    m_tv.setAlpha(1f);
    AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(this,
            R.animator.fadein);
    set.setTarget(m_tv);
    set.start();
}
{% endhighlight %}

### 7）PropertyValuesHolder

前边提到的都是，如何给一个动画设置一个属性值，PropertyValuesHolder 类可以给一个动画设置多个属性值。上边第四个按钮的功能是从右下角移动到左上角，弹跳效果是由弹性插值器（Bounce Interpolator()）实现的，前边已经提到过。代码如下：

{% highlight java %}
/*
 * 一个动画改变多个属性值
 */
public void testPropertyValuesHolder(View v) {
    m_tv.setAlpha(1f);
    float h = m_tv.getHeight();
    float w = m_tv.getWidth();
    float x = m_tv.getX();
    float y = m_tv.getY();
 
    m_tv.setX(w);
    m_tv.setY(h);
    PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("x", x);
    PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("y", y);
 
    ObjectAnimator oa = ObjectAnimator.ofPropertyValuesHolder(m_tv, pvhX,
            pvhY);
    oa.setDuration(3000);
    // oa.setInterpolator(new AccelerateDecelerateInterpolator());
    oa.setInterpolator(new BounceInterpolator());
    oa.start();
}
{% endhighlight %}

### 8）ViewPropertyAnimator

ViewPropertyAnimator是在Android3.1中新增加的动画，这个类对多属性动画进行了优化，会合并一些 invalidate() 来减少刷新视图。上边第五个按钮，代码：

{% highlight java %}
/*
 * 一个View的多个属性进行动画，3.1中引入，对多属性动画进行了优化
 */
public void testViewPropertyAnimator(View v) {
    m_tv.setAlpha(1f);
    float h = m_tv.getHeight();
    float w = m_tv.getWidth();
    float x = m_tv.getX();
    float y = m_tv.getY();
 
    m_tv.setX(w);
    m_tv.setY(h);
 
    ViewPropertyAnimator vpa = m_tv.animate().x(x).y(y);
 
    vpa.setDuration(1500);
    vpa.setInterpolator(new BounceInterpolator());
}
{% endhighlight %}
它的实现顺序是：

1. 用animate()方法从m_tv中获得一个ViewPropertyAnimator；
2. 使用ViewPropertyAnimator设置不同属性，比如x, y, scale, alpha等。
3. 然后UI线程会自动开始生成动画，不用start()方法。

关于ViewPropertyAnimator，Chet Haase （Google 图形动画工程师，这个类估计是他写的吧）的这篇《[Introducing ViewPropertyAnimator](http://android-developers.blogspot.com/2011/05/introducing-viewpropertyanimator.html)》估计是最详细的。

### 9）TypeEvaluator

Android 支持4种求值器（Evaluator），需要注意的是，它支持Android的所有动画系统，不仅Propery Animation。它提供了以下几种Evalutor：

- IntEvaluator：属性的值类型为int；
- FloatEvaluator：属性的值类型为float；
- ArgbEvaluator：属性的值类型为十六进制颜色值；
- TypeEvaluator：一个接口，可以通过实现该接口自定义Evaluator。
   
上边第六个按钮中自定义了一个Evaluator，它实现了TypeEvaluator接口，然后重写 evaluate() 方法。这里是涉及坐标的两个值，利用 startPropertyValue、endPropertyValue、fraction 求当前坐标，代码：

{% highlight java %}
package net.mindlee.android.propertyanimation;
 
import android.animation.TypeEvaluator;
import android.graphics.PointF;
 
public class MyPointEvaluator implements TypeEvaluator<PointF> {
    public PointF evaluate(float fraction, PointF startValue, PointF endValue) {
        PointF startPoint = (PointF) startValue;
        PointF endPoint = (PointF) endValue;
        return new PointF(
                startPoint.x + fraction * (endPoint.x - startPoint.x),
                startPoint.y + fraction * (endPoint.y - startPoint.y));
    }
}
{% endhighlight %}
其中的 fraction 来自 Interplator，另外，可以把 Point的 setPoint()，getPoint() 等方法封装为一个类，因为我们每一帧都要调用它们，如下：

{% highlight java %}
package net.mindlee.android.propertyanimation;
 
import android.graphics.PointF;
import android.view.View;
 
public class MyAnimatableView {
    PointF curPoint = null;
    View m_v = null;
 
    public MyAnimatableView(View v) {
        curPoint = new PointF(v.getX(), v.getY());
        m_v = v;
    }
 
    public PointF getCurPointF() {
        return curPoint;
    }
 
    public void setPoint(PointF p) {
        curPoint = p;
        m_v.setX(p.x);
        m_v.setY(p.y);
    }
}
{% endhighlight %}
那么实现上边第六个按钮的代码就是：
{% highlight java %}
/*
 * 自定义Evaluator
 */
public void testTypeEvaluator(View v) {
    m_tv.setAlpha(1f);
 
    float h = m_tv.getHeight();
    float w = m_tv.getWidth();
    float x = m_tv.getX();
    float y = m_tv.getY();
 
    ObjectAnimator tea = ObjectAnimator.ofObject(m_atv, "point",
            new MyPointEvaluator(), new PointF(w, h), new PointF(x, y));
    tea.setDuration(2000);
    tea.setInterpolator(new OvershootInterpolator());
    tea.start();
 
}
{% endhighlight %}
### 10）KeyFrames

一个关键帧包含一个【时间/值】对，通过它可以定义一个在特定时间的特定状态，而且每个KeyFrame可以设置不同的Interpolator，作用范围是它的前一个keyFrame和它本身之间。可以通过ofInt()，ofFloat()，ofObject() 获得适当的KeyFrame，然后通过 PropertyValuesHolder.ofKeyframe 获得PropertyValuesHolder对象，比如第七个按钮中的旋转效果，代码：

{% highlight java %}
/*
 * 关键帧
 */
public void testKeyFrames(View v) {
    float h = m_tv.getHeight();
    float w = m_tv.getWidth();
    float x = m_tv.getX();
    float y = m_tv.getY();
 
    Keyframe kf0 = Keyframe.ofFloat(0.2f, 360);
    Keyframe kf1 = Keyframe.ofFloat(0.5f, 30);
    Keyframe kf2 = Keyframe.ofFloat(0.8f, 1080);
    Keyframe kf3 = Keyframe.ofFloat(1f, 0);
    PropertyValuesHolder pvhRotation = PropertyValuesHolder.ofKeyframe(
            "rotation", kf0, kf1, kf2, kf3);
 
    PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("x", w, x);
    PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("y", h, y);
 
    ObjectAnimator anim = ObjectAnimator.ofPropertyValuesHolder(m_tv,
            pvhRotation, pvhX, pvhY);
    anim.setDuration(2000);
    anim.start();
}
{% endhighlight %}

完整代码@GihHub：可以 [Fork](https://github.com/wliday/PropertyAnimation) 或者 [打包下载](https://github.com/wliday/PropertyAnimation/downloads)。

参考资料：

Android官方文档：Develop，API Guides，Property Animation

Android Developers Blog: Animation in Honeycomb

Android Developers Blog: Introducing ViewPropertyAnimator

*Android动画学习笔记*

*Pro Android 4*，Chapter 21，Exploring 2D Animation