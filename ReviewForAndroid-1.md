#Review For Android
***
##Activity
***
###生命周期

**1**.虽然 **onStart()** 和 **onResume()** 都表示 **Activity** 已经可见，但是 **onStart()** 的时候 **Activity** 还在后台， **onResume()** 的时候 **Activity** 才会显示到前台.  

**2**.前一个 **Activity** 的 **onPause()** 必须前执行完，后一个 **Activity** 的 **onResume()** 才会被执行（基于5.0 源码），在 **onPause()** 和 **onStop()** 不要做太耗时的操作， **onStop()** 中可进行一些稍微重量级的回收操作，在 **onDestory()** 中可以做最终的资源释放.  

**3**.当新打开的 **Activity** 采用透明主题，那么当前 **Activity** 不会回调 **onStop()** .  

**4**.当系统配置发生改变后， **Activity** 会被销毁，由于是在异常情况下终止的，系统会调用 **onSaveInstanceState()** 来保存当前状态，在 **onStop()** 前调用，当 **Activity** 被重新创建后，系统会调用 **onRestoreInstanceState()** ,并且把 **Activity** 销毁时 **onSaveInstanceState()** 所保存的 **Bundle** 对象作为参数传递给 **onRestoreInstanceState()** 和 **onCreate()** , **onRestoreInstanceState()** 在 **onStart()** 之后.  

**5**. **View** 和 **Activity** 同样具有  **onSaveInstanceState()** 和  **onRestoreInstanceState()** ,关于保存和恢复 **View** 的层次结构 ： 委托思想.  

**6**. **Activity** 优先级一般分为3种:  
	(1)前台 **Activity** ：正在和用户交互，优先级最高.  
	(2)可见但非前台 **Activity** .  
	(3)后台 **Activity** ：执行了 **onStop()** 优先级最低.  
	
当系统内存不足时，系统会按照上述优先级杀死目标 **Activity** 所在的 **进程**，脱离四大组件的进程很快被系统杀死，后台工作放入 **Service** 从而保证进程具有一定的优先级.
	
**7**.如果不想系统在系统配置发生改变后重新创建 **Activity** ，可以给 **Activity** 指定 **configChanges** 属性，例：  

<font size = 4 color = blue>`android:configChanges="orientation"`</font>  

之后系统不调用  **onSaveInstanceState()** 和  **onRestoreInstanceState()** ，而是调用 **onConfigurationChanged()** .  
***

###启动模式

**1**.4种 **LauchMode** ：(1) **standard**  (2) **singleTop**  (3) **singleTask**  (4) **singleInstance** .  

**2**.用 **ApplicationContext** 去启动 **standard** 模式的 **Activity** 时，会出现：  

<font size = 3 color = red>`android.util.AndroidRuntimeException: Calling startActivity() from outside of an Activity  context requires the FLAG_ACTIVITY_NEW_TASK flag. Is this really what you want?`</font>  

因为非 **Activity** 类型的 **Context** 并没有所谓的 任务栈 ， 解决方法 是为 待启动 的 **Activity** 指定  
<font size = 4 color = blue>`FLAG_ACTIVITY_NEW_TASK` </font> 标记位，这样启动 **Activity** 时会创建一个新的任务栈，本质以 **singleTask** 模式启动.

  