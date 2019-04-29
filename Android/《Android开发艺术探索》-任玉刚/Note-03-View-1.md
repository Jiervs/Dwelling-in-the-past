# Review For Android
***
## View 的事件体系
***
### View 的基础知识  

**1**. **View** 是界面层控件的一种抽象，**ViewGroup** 也是继承于 **View** 的，**View** 的位置主要由它四个顶点来决定的：  
**top** : 左上角的纵坐标 .  
**left** : 左上角的横坐标 .  
**right** : 右下角的横坐标 .  
**bottom** : 右下角的纵坐标 .   

<img src = "resource/View.png" width = 300 />

这些参数都是相对于 **View** 的父容器而言 ，从 **Android** 3.0 开始 ，**View** 增加了额外的几个参数 ： **x** 、**y** 、 **translationX** 、**translationY**、这几个参数也是相对于父容器的，其中  **x** ， **y**  是  **View** 的左上角的坐标 , **translationX** 、**translationY** 是 **View** 左上角相对于父容器的偏移量.  

**2**. **MontionEvent** 典型的事件类型有如下几种 :   
<font size = 4 color = blue>`ACTION_DOWN` </font> : 手指刚接触屏幕 .   
<font size = 4 color = blue>`ACTION_MOVE` </font> : 手指在屏幕上移动 .   
<font size = 4 color = blue>`ACTION_UP` </font> : 手指从屏幕上松开的一瞬间.   
通过  **MontionEvent** 对象我们可以得到点击事件发生的 **x** 和 **y** 的坐标，**getX()** 和 **getY()** 返回的是相当于当前 **View** 左上角的 **x** 和 **y** 的坐标，而 **getRawX()** 和 **getRawY()** 返回的是相对于手机屏幕左上角的  **x** 和 **y** 的坐标.   

**3**.**TouchSlop** 是系统所能识别出的被认为是**滑动**的**最小距离**，跟设备有关，可以通过如下获得:

```java
ViewConfiguration.get(getContext()).getScaledTouchSlop();
```

**4**.**VelocityTracker** : 用于追踪手指在滑动过程中的速度 , 使用过程 :   
首先在 **View** 的 **onTouchEvent** 方法中追踪当前单击事件的速度 :  

```java
VelocityTracker tracker = VelocityTracker.obtain();
tracker.addMovement(event);
``` 
接着当我们想知道滑动速度时候，可以通过以下方式获得 :   

```java
tracker.computeCurrentVelocity(1000);
int xVelocity = (int) tracker.getXVelocity();
int yVelocity = (int) tracker.getYVelocity();
``` 
必须先调用  **VelocityTracker.obtain()** 方法，参数是时间间隔，这里是1000ms , 而这里的速度是指 一段时间内手指所滑过的像素值 .   

最后当不需要使用它的时候，需要调用 clear() 方法来重置并回收内存 :

```java
tracker.clear();
tracker.recycle();
```  
**5**. **GestureDetecor** : 手势检测，用于辅助检测用户的 **单击** ，**滑动**，**长按**，**双击** 等行为，使用过程如下 :  
首先 , 创建一个 **GestureDetecor** 对象并实现 **OnGestureListener** 接口 , 根据需要我们还可以实现 **OnDoubleTapListener** 从而能够监听双击行为 :

```java
GestureDetector detector = new GestureDetector(this);
detector.setIsLongpressEnabled(false);//解决长按屏幕后无法拖动的现象 
```  
接着 接管 目标 **View** 的 **onTouchEvent()** 方法，在监听 **View** 的 **onTouchEvent()** 方法中添加如下实现 :  

```java   
boolean  consume = detector.onTouchEvent(event);
return consume;
``` 

做完上面两步，就可以有选择地实现 **OnGestureListener** 和 **OnDoubleTapListener** 中的方法了，方法含义如下图所示： 

<img src = "resource/OnGestureListener%EF%BC%8COnDoubleTapListener(1).png" width = 700 />
<img src = "resource/OnGestureListener%EF%BC%8COnDoubleTapListener(2).png" width = 700 />   

**6**. **Scroller** : 弹性滑动对象 ， 使用 **View** 的 **scrollTo** / **scrollBy** 方法来进行滑动时，其过程是瞬间完成的，用户体验不好，这个时候可以用 **Scroller** 来实现有过度效果的滑动，**Scroller** 本身无法让 **View** 弹性滑动，它需要和 **View** 的 **computeScroll()** 配合使用才能共同完成这个功能 .   

***
### View 的滑动   

**1**. **View** 的滑动一般使用三种方法 :   
1.通过 **View** 本身提供的 **scrollTo()** 和 **scrollBy()** 方法实现滑动 .   
2.第二种是通过动画给 **View** 施加平移效果来实现滑动 .   
3.第三种是通过改变 **View** 的 **LayoutParam** 使得 **View** 重新布局从而实现滑动 .  

**2**. 使用 **scrollTo()** 和 **scrollBy()** : **scrollBy()** 本质上调用了 **scrollTo()**，**scrollTo()**是实现了基于所传递参数的绝对滑动 ， **scrollBy()** 是相对滑动，**scrollTo()** 和 **scrollBy()** 只能改变 **View 内容**的位置而不能改变 **View** 在布局中的位置，关于 **View** 内部两个属性 **mScrollX** 和 **mScrollY** 的改变规则请看下图 ：    
<img src = "resource/mScrollX_mScrollY.png" width = 450 />  
 
**3**. **使用动画** : 主要是操作 View 的 **translationX** 和 **translationY**  属性，既可以采用传统的 **View** 动画，也可以采用属性动画，**View** 动画改变的只是一种影像， **View** 的位置参数，包括宽高都不会改变，而属性动画是真实改变 **View** 的位置，但是在3.0以上才能使用（有兼容库）.   

**4**.  **改变布局参数** : 比如使一个 **Button** 向右平移100px : 

```java
ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams)button.getLayoutParams();  
params.leftMargin += 100
button.requestLayout();
button.setLayoutParams(params);
```

**5**. 各种滑动方式的对比 : 1. **scrollTo(),scrollBy()** 操作简单，适合对 **View** 内容滑动 ;  2.动画 : 操作简单 , 主要适用于没有交互的 **View** 和实现复杂的动画效果 ;  3. 改变布局参数 : 操作稍微复杂，适用于有交互的 **View**.   

**6**.弹性滑动 : 实现弹性滑动都有一个共同的思想 : 将一次大的滑动分成若干次小的滑动并在一个时间段内完成，比如 : 通过 **Scroller** 、**Handler+postDelayed** 、**Thread+sleep**等：   
（1）. **Scroller** : Scroll 的典型用法 :      

```java
scroller = new Scroller(context); 

public void smoothScrollTo (int destX , int destY) {
    int scrollX = getScrollX();
    int deltaX = destX - scrollX;
    //1000 ms 内滑向 destX ，效果就是慢慢滑动
    scroller.startScroll(scrollX,0,deltaX,0,1000);
    invalidate();
}

@Override
public void computeScroll() {
    if (scroller.computeScrollOffset()) {
        scrollTo(scroller.getCurrX(),scroller.getCurrY());
        postInvalidate();
    }
}
```
    
**invalidate()** 会导致 **View** 重绘，在 **View** 的 **draw()** 中又回去调用 **computeScroll()** , **computeScroll()** 在 **View** 中是一个空实现，因此我们需要自己实现，而在上面的代码中，自己实现的  **computeScroll()** 中向 **Scroller** 获取当前的 **scrollX** , **scrollY** ，然后通过 **scrollTo** 方法实现滑动 , 接着又调用 **postInvalidate()** 来进行第二次重绘，还是会导致 **computeScroll()** 被调用 , 然后继续向 **Scroller** 获取当前的 **scrollX** 和 **scrollY** , 并通过 **scrollTo()** 滑动到新的位置 , 如此反复，直到整个滑动过程结束 .   

关于弹性滑动还有 **通过动画** 和 **使用延时策略** 思想 .

***
### View 的事件分发机制

**1**.当一个 **MotionEvent** 产生以后，系统需要把这个事件传递给一个具体的 **View**，期过程就是分发过程，点击事件的分发过程由3个很重要的方法共同完成 :    
(1).<font size = 4 color = blue>`public boolean dispatchTouchEvent(MotionEvent ev)` </font>   
用来进行事件的分发，如果事件能够传递给当前 **View** ，那么此方法一定会被调用，返回结果受当前 View 的 onTouchEvent() 和下级 View
 的 dispatchTouchEvent() 的影响，表示是否消耗当前事件.   
 
 (2).<font size = 4 color = blue>`public boolean onInterceptTouchEvent(MotionEvent ev)` </font>   
在上述方法内部调用 , 用来判断是否拦截某个事件 , 如果当前 **View** 拦截了某个事件 , 那么在同一个事件序列当中 , 此方法不会被再次调用 , 返回结果表示是否拦截当前事件 .   

 (3).<font size = 4 color = blue>`public boolean onTouchEvent(MotionEvent ev)` </font>   
在 **dispatchTouchEvent()** 中调用 ，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗 , 则在同一个事件序列中，当前 **View** 无法再次接收到事件 . 

以上3个方法的关系可以通过如下伪代码表示 : 
<img src = "resource/event_view.png" width = 450 /> 
详细分析及源码 见 **《Android开发艺术探索》- 任玉刚 - p142 - p154** 