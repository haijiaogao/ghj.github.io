---
title: Android Knowledge&Interview
tags:
---


# Android 基础知识复习

## Activity

### Activity生命周期

![image](https://camo.githubusercontent.com/908f0c391fd50377aebbb3c0fd1a0c4419007c0eecb7651a1908be8e273a46ab/687474703a2f2f6769747975616e2e636f6d2f696d616765732f6c6966656379636c652f61637469766974792e706e67)


Activity A 启动另一个Activity B，回调如下:
Activity A 的onPause() → Activity B的onCreate() → onStart() → onResume() → Activity A的onStop()；如果B是透明主题又或则是个DialogActivity，则不会回调A的onStop；


### Activity的启动模式和Android的任务栈分析

####　四种启动模式：Standard，SingleTop，SingleTask,SingleInstance
1. Standard: 默认启动模式当目标Activity在AndroidManifest中为Standard类型时，不管当前的任务栈中是否已经存在该Activity的实例，都重新生成一个Activity的实例，并推入任务栈栈顶。
2. SingleTop: 当目标Activity被声明为SingleTop时，如目标不存在，则新建一个activity放在栈顶；如果目标已经存在，并且非处于任务栈顶，则重新生成一个目标Activity放在栈顶；如果目标已经存在，并处于栈顶，则调用该Activity的onNewIntent方法。
3. SingleTask: 当目标Activity被声明为SingleTask的时候，如果说目标不存在，则新建一个Activity放在栈顶，并且新建一个目标Activity放在栈中；如果目标存在于当前栈中，则将当前栈中位于目标之上的Activity全部都出栈并销毁（onDestory方法会调用），并调用目标Activity的onNewIntent方法。
4. SingleInstance： 当目标Activity被声明为SingleInstance的时候，如果说目标不存在，则新建一个Task，并把新建的目标Activity压入栈中；singleInstance的Activity始终单独使用一个任务栈；如果目标存在，则调用该Activity的onNewIntent方法。
注意：
因为 singleInstance 的属性是禁止与其他 Activities 共享任务栈，所以启动模式为 SingleInstance 的 Activity 启动其他 Activity 时会默认带有 FLAG_ACTIVITY_NEW_TASK 属性。所以从一个launchMode为singleInstance的activity中新启动一个启动模式为Standard的 Activity F 后， Activity F会被推到SingleInstance之前的那个标准任务栈的栈顶。


#### taskAffinity
什么是affinity？
　　affinity是指Activity的归属，Activity与Task的吸附关系，也就是该Activity属于哪个Task。一般情况下在同一个应用中，启动的Activity都在同一个Task中，它们在该Task中度过自己的生命。每个Activity都有taskAffinity属性，这个属性指出了它希望进入的Task。如果一个Activity没有显式的指明taskAffinity，那么它的这个属性就等于Application指明的taskAffinity，如果Application也没有指明，那么该taskAffinity的值就等于应用的包名。我们可以通过在元素中增加taskAffinity属性来为某一个Activity指定单独的affinity。这个属性的值是一个字符串，可以指定为任意字符串，但是必须至少包含一个”.”，否则会报错。

affinity在什么场合应用呢？
1.根据affinity重新为Activity选择宿主task（与allowTaskReparenting属性配合使用）
　　allowTaskReparenting用来标记Activity能否从启动的Task移动到taskAffinity指定的Task，当把Activity的allowTaskReparenting属性设置成true时，Activity就拥有了一个转移所在Task的能力。具体点来说，就是一个Activity现在是处于某个Task当中的，但是它与另外一个Task具有相同的affinity值，那么当另外这个任务切换到前台的时候，该Activity就可以转移到现在的这个任务当中。allowTaskReparenting默认是继承至application中的allowTaskReparenting=false，如果为true，则表示可以更换；false表示不可以。
　　举一个形象点的例子，比如有一个天气预报程序，它有一个用于显示天气信息的Activity，allowTaskReparenting属性设置成true，这个Activity和天气预报程序的所有其它Activity具体相同的affinity值。这个时候，你自己的应用程序通过Intent去启动了这个用于显示天气信息的Activity，那么此时这个Activity应该是和你的应用程序是在同一个任务当中的。但是当把天气预报程序切换到前台的时候，这个Activity会被转移到天气预报程序的任务当中，并显示出来。如果将你自己的应用切换到前台，发现你自己应用Task里的那个Activity消失了。

2.启动一个Activity过程中Intent使用了FLAG_ACTIVITY_NEW_TASK标记，根据affinity查找或创建一个新的具有对应affinity的task。
　　当调用startActivity()方法来启动一个Activity时，默认是将它放入到当前的任务当中。但是，如果在Intent中加入了FLAG_ACTIVITY_NEW_TASK flag的话，情况就会变的复杂起来。首先，系统会去检查这个Activity的affinity是否与当前Task的affinity相同。如果相同的话就会把它放入到当前Task当中，如果不同则会先去检查是否已经有一个名字与该Activity的affinity相同的Task,如果有，这个Task将被调到前台，同时这个Activity将显示在这个Task的顶端；如果没有的话，系统将会尝试为这个Activity创建一个新的Task。需要注意的是，如果一个Activity在manifest文件中声明的启动模式是”singleTask”，那么他被启动的时候，行为模式会和前面提到的指定FLAG_ACTIVITY_NEW_TASK一样。
　　那么，有了上面的知识，我们应该可以实现开头提到的功能了。


 ####　IntentFlags

 FLAG_ACTIVITY_BROUGHT_TO_FRONT

        比方说我现在有Ａ，在Ａ中启动Ｂ，在Ａ中Intent中加上这个标记。此时B就是以FLAG_ACTIVITY_BROUGHT_TO_FRONT 这个启动的，在B中再启动C，D（正常启动C，D），如果这个时候在D中再启动B，这个时候最后的栈的情况是 A,C,D,B。

FLAG_ACTIVITY_REORDER_TO_FRONT

        如果在Intent中设置，并传递给Context.startActivity()，这个标志将引发已经运行的Activity移动到历史stack的顶端。 例如，假设一个Task由四个Activity组成：A，B，C，D。如果D调用startActivity()来启动Activity B，那么，B会移动到历史stack的顶端，现在的次序变成A，C，D，B。如果FLAG_ACTIVITY_CLEAR_TOP标志也设置的话，那么这个标志将被覆盖。

FLAG_ACTIVITY_CLEAR_TASK

        如果在调用startActivity时传递这个标记，该task栈中的其他activity会先被清空，然后该activity在该task中启动，也就是说，这个新启动的activity变为了这个空task的根activity。所有老的activity都结束掉。该标志必须和FLAG_ACTIVITY_NEW_TASK一起使用。

FLAG_ACTIVITY_CLEAR_TOP

        如果该activity已经在task中存在，并且设置了该task，系统不会启动新的 Activity 实例，会将task栈里该Activity之上的所有Activity一律结束掉，然后将Intent发给这个已存在的Activity。Activity收到 Intent之后，或者在onNewIntent()里做下一步的处理，或者自行结束然后重新创建。如 Activity 在 AndroidMainifest.xml 里将启动模式设置成默认standard模式，且 Intent 里也没有设置 FLAG_ACTIVITY_SINGLE_TOP，那么Activity将会结束并且重启；否则则会传递到onNewIntent方法。

已经启动了四个Activity：A，B，C和D。在D Activity里，我们要跳到B Activity，同时希望C finish掉，可以在startActivity(intent)里的intent里添加flags标记，这样启动B Activity，就会把D，C都finished掉，如果你的B Activity的启动模式是默认的（multiple） ，则B Activity会finished掉，再启动一个新的Activity B。  如果不想重新再创建一个新的B Activity，则可在启动Intent添加flag FLAG_ACTIVITY_SINGLE_TOP。

        可以利用此特性来退出程序，假设A为程序入口，将A的Manifest.xml配置成android:launchMode="singleTop"，通过此flag来启动A，同时在A的onNewIntent中判断来结束自己，可到退出的效果。FLAG_ACTIVITY_CLEAR_TOP 还可以和 FLAG_ACTIVITY_NEW_TASK 配合使用，用来启动一个task栈的根activity，他将会把该栈清空为根状态，比如从notification manager启动activity。

FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS

        设置完之后，新的activity将不会添加到当前activity列表中，当某些情况下我们不希望用户通过历史列表回到我们的Activity的时候这个标记比较有用。他等同于在XML中指定Activity的属性android:excludeFromRecents=”true”

FLAG_ACTIVITY_FORWARD_RESULT

        如果A需要onActivityResult中获取返回结果，startActivityForResult B，而B只是过渡页，启动C之后就finish掉了，需要在 C 中setResult返回给A就可以用到这个标志。

A -> B -> XXXXX(无论多少个过渡页) 设置 FLAG_ACTIVITY_FORWARD_RESULT 来启动 C ，之后该XXX过渡页finish - > C ，那么C的结果返回给A。

FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY

        如果设置，新的Activity将不再历史stack中保留。用户一离开它，这个Activity就关闭了。例如A启动B的时候，给B设置了FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY，那么：

A -> B -> C ，启动C 就算 B没有自行finish ，也会变为 AC

FLAG_ACTIVITY_MULTIPLE_TASK

        这个标识用来创建一个新的task栈，并且在里面启动新的activity（所有情况，不管系统中存在不存在该activity实例），经常和FLAG_ACTIVITY_NEW_DOCUMENT或者FLAG_ACTIVITY_NEW_TASK一起使用。这上面两种使用场景下，如果没有带上FLAG_ACTIVITY_MULTIPLE_TASK标识，他们都会使系统搜索存在的task栈，去寻找匹配intent的一个activity，如果没有找到就会去新建一个task栈；但是当和FLAG_ACTIVITY_MULTIPLE_TASK一起使用的时候，这两种场景都会跳过搜索这步操作无条件的创建一个新的task。和FLAG_ACTIVITY_NEW_TASK一起使用需要注意，尽量不要使用该组合除非你完成了自己的顶部应用启动器，他们的组合使用会禁用已经存在的task栈回到前台的功能。

FLAG_ACTIVITY_NEW_DOCUMENT

        api 21之后加入的一个标识，用来在intent启动的activity的task栈中打开一个document，和documentLaunchMode效果相等，有着不同的documents的activity的多个实例，将会出现在最近的task列表中。

documentLaunchMode可以设置4个值

intoExisting： activity 会为该document请求一个已经存在的task，这与设置FLAG_ACTIVITY_NEW_DOCUMENT且不设置FLAG_ACTIVITY_MULTIPLE_TASK 有相同的效果。

always： activity 会为该document创建一个新的task，即使该document已经被打开了，这与设置 FLAG_ACTIVITY_NEW_DOCUMENT且设置 FLAG_ACTIVITY_MULTIPLE_TASK 有相同的效果。

none：activity 不会为 document 创建新的task，该app被设置为 single task 的模式，它会重新调用用户唤醒的所有activity中的最近的一个。

never：activity 不会为document创建一个新的task，设置这个值复写了 FLAG_ACTIVITY_NEW_DOCUMENT 和 FLAG_ACTIVITY_MULTIPLE_TASK 标签。如果其中一个标签被设置，并且overview screen 显示该app为 single task 模式。则该activity会重新调用用户最近唤醒的activity。

注意： none 或 nerver 使用时，activity必须设置为 launchMode=”standard” ，如果该属性没有设置，documentLaunchMode=”none” 属性就会被使用。有

FLAG_ACTIVITY_RETAIN_IN_RECENTS

        api21加入。默认情况下通过FLAG_ACTIVITY_NEW_DOCUMENT启动的activity在关闭之后，task中的记录会相对应的删除。如果为了能够重新启动这个activity你想保留它，就可以使用者个flag，最近的记录将会保留在接口中以便用户去重新启动。接受该flag的activity可以使用autoRemoveFromRecents去复写这个request或者调用Activity.finishAndRemoveTask()方法。

FLAG_ACTIVITY_NEW_TASK

        设置此状态，记住以下原则，首先会查找是否存在和被启动的Activity具有相同的亲和性的任务栈（即taskAffinity，注意同一个应用程序中的activity的亲和性在没有修改的情况下是一样的，所以下面的a情况会在同一个栈中），如果有，刚直接把这个栈整体移动到前台，并保持栈中的状态不变，即栈中的activity顺序不变，如果没有，则新建一个栈来存放被启动的activity。

a. 前提: Activity A和Activity B在同一个应用中。

操作: Activity A启动开僻Task堆栈(堆栈状态：A)，在Activity A中启动Activity B， 启动Activity B的Intent的Flag设为FLAG_ACTIVITY_NEW_TASK，Activity B被压入Activity A所在堆栈(堆栈状态：AB)。

原因: 默认情况下同一个应用中的所有Activity拥有相同的关系(taskAffinity)。

b. 前提: Activity A在名称为”TaskOne应用”的应用中， Activity C和Activity D在名称为”TaskTwo应用”的应用中。

操作1:在Launcher中单击“TaskOne应用”图标，Activity A启动开僻Task堆栈，命名为TaskA(TaskA堆栈状态: A)，在Activity A中启动Activity C， 启动Activity C的Intent的Flag设为FLAG_ACTIVITY_NEW_TASK，Android系统会为Activity C开僻一个新的Task，命名为TaskB(TaskB堆栈状态: C), 长按Home键，选择TaskA，Activity A回到前台, 再次启动Activity C（两种情况：1.从桌面启动；2.从Activity A启动，两种情况一样）， 这时TaskB回到前台, Activity C显示，供用户使用, 即：包含FLAG_ACTIVITY_NEW_TASK的Intent启动Activity的Task正在运行，则不会为该Activity创建新的Task，而是将原有的Task返回到前台显示。

操作2:在Launcher中单击”TaskOne应用”图标，Activity A启动开僻Task堆栈，命名为TaskA(TaskA堆栈状态: A)，在Activity A中启动Activity C，启动Activity C的Intent的Flag设为FLAG_ACTIVITY_NEW_TASK，Android系统会为Activity C开僻一个新的Task，命名为TaskB(TaskB堆栈状态: C)， 在Activity C中启动Activity D(TaskB的状态: CD) 长按Home键， 选择TaskA，Activity A回到前台， 再次启动Activity C(从桌面或者ActivityA启动，也是一样的)，这时TaskB回到前台, Activity D显示，供用户使用。说明了在此种情况下设置FLAG_ACTIVITY_NEW_TASK后，会先查找是不是有Activity C存在的栈，根据亲和性(taskAffinity)，如果有，刚直接把这个栈整体移动到前台，并保持栈中的状态不变，即栈中的顺序不变。

FLAG_ACTIVITY_NO_ANIMATION

        启动的时候不执行动画。

FLAG_ACTIVITY_NO_USER_ACTION

        禁止activity调用onUserLeaveHint()函。onUserLeaveHint()作为activity周期的一部分，它在activity因为用户要跳转到别的activity而退到background时使用。比如，在用户按下Home键（用户的操作），它将被调用。比如有电话进来（不属于用户的操作），它就不会被调用。注意：通过调用finish()时该activity销毁时不会调用该函数。

FLAG_ACTIVITY_PREVIOUS_IS_TOP

        如果给Intent对象设置了这个标记，这个Intent对象被用于从一个存在的Activity中启动一个新的Activity，那么新的这个Activity不能用于接受发送给顶层activity的intent，这个新的activity的前一个activity被作为顶部activity。

FLAG_ACTIVITY_TASK_ON_HOME

        api11加入。把当前新启动的任务置于Home任务之上，也就是按back键从这个任务返回的时候会回到home，即使这个不是他们最后看见的activity，注意这个标记必须和FLAG_ACTIVITY_NEW_TASK一起使用。

FLAG_EXCLUDE_STOPPED_PACKAGES和FLAG_INCLUDE_STOPPED_PACKAGES

        从Android 3.1开始，给Intent定义了两个新的Flag，分别为FLAG_INCLUDE_STOPPED_PACKAGES和FLAG_EXCLUDE_STOPPED_PACKAGES，用来控制Intent是否要对处于停止状态的App起作用，顾名思义：

FLAG_INCLUDE_STOPPED_PACKAGES：表示包含未启动的App

FLAG_EXCLUDE_STOPPED_PACKAGES：表示不包含未启动的App

值得注意的是，Android 3.1开始，系统向所有Intent的广播添加了FLAG_EXCLUDE_STOPPED_PACKAGES标志。这样做是为了防止广播无意或不必要地开启未启动App的后台服务。如果要强制调起未启动的App，后台服务或应用程序可以通过向广播Intent添加FLAG_INCLUDE_STOPPED_PACKAGES标志来唤醒。




### 常用Android View的使用

1. Views（TextView,ImageView,EditText,Button,Switch,CheckBox,RadioGroup,RadioButton,ToggleButton,WebView,ProgressBar,SeekBar）
2. ViewGroup(FrameLayout,RelativeLayout,ConstraintLayout,LinearLayout,GridLayout,ScrollView,ListView,TabHost)

ConstraintLayout 优势：
	1). 所见即所得。易于在AndroidStudio中进行配置
	2). 和RelativeLayout一样，可以有效减少布局层数，从而降低绘制复杂度
	3). 和RelativeLayout相比，其布局方式更灵活，更容易适配多种机型。

ConstraintLayout的使用分析：
各种属性的使用：
https://www.jianshu.com/p/17ec9bd6ca8a
https://www.jianshu.com/p/38ee0aa654a8

ConstraintSet的使用：
ConstraintSet主要是方便在代码中修改ConstraintLayout的各种约束属性。
使用：
http://hulkyang.blogspot.com/2018/12/android-constraintlayout-constraintset.html


###　RecyclerView 使用
https://www.jianshu.com/p/4f9591291365

RecyclerView的特点：
1. 默认实现的多级缓存机制
2. 灵活的布局规则，有很强的可配置、可定制性，可实现多行多列，可多向滚动的List
3. 通过LayoutManager，Adapter，ViewHolder，实现了布局和逻辑的解耦
4. 可以局部刷新，并且可以通过ItemAnimation实现更丰富的动画效果


RecyclerView的4级缓存机制：
RecylerView的多级缓存机制，是有RecyclerView中的内部类Recycler实现的。
其中最重要的方法：
getViewForPosition(int position)->getViewForPosition(int position, boolean dryRun) -> tryGetViewHolderForPositionByDeadline(int position,
                boolean dryRun, long deadlineNs) 

核心逻辑实现于tryGetViewHolderForPositionByDeadline中，其中从上个方法传递过来的参数中的dealineNs为Int.MaxValue.

第一步：判断是否处于isPreLayout状态，这个状态值只有在执行item动画的时候才会为true。如果是的话，则调用getChangedScrapViewForPosition（）方法获取viewHolder并返回

第二歩(核心步骤)：在isPreLayout=false的情况下，正常获取一个ViewHolder，需要先按照位置从scrap或者是hiden的或者是cachedView中找。如果说没有找到的话，就按照id和itemType在scrap和缓存中找；如果说没有找到合适的VH，那就判定是否有mViewCacheExtension的定义，有就从mViewCacheExtension中根据position和type找；如果还是没有找到合适的VH，下一个步骤就是在recyclerPool中根据type找

第三歩：如果都没有找到缓存的VH，则需要通过createViewHolder回调Adapter中的方法，进行创建ViewHolder。如果说缓存的VH的话判断是否需要绑定，需要的话就调用onBindViewHolder方法



### View 触摸事件传递机制


在应用中传递顺序：

>1. Activity  dispatchTouchEvent

				| 
>2. PhoneWindow的superDispatchTouchEvent()方法

				|
>3. DocerView的superDispatchTouchEvent()方法 调用super.dispatchTouchEvent

				|
>4. ViewGroup dispatchTouchEvent DocerView继承自ViewGroup，so调用ViewGroup的dispatch方法(划重点：该方法完成主要的传递逻辑)

				|
>5. ViewGroup中首先交由自己的onInterpretTouchEvent进行处理，如果说onInterpretTouchEvent返回true，那么事件被自身拦截，不会继续向下传递。如果不处理，则交由合适的子View进行处理，如果子view仍然是viewGroup则继续按照规则向下分发。如果没有子view则交由自身的touchEvent处理，如果都返回的false。touch事件将不再继续向下分发，而回到上一级的ViewGroup

 				|
>6. View 的dispatchTouchEvent中如果返回true，则交由该View进行处理，如果返回false，则继续由父View 进行处理。同理依次向上推导，直到回到Activity的dispatchTouchEvent方法。
https://blog.csdn.net/fengluoye2012/article/details/83782042

### View KeyEvent传递机制
和触摸事件的传递大同小异。相比较而言，KeyEvent事件会更加简单一些，因为触摸事件有ActionDown，ActionUp，中间还有若干次的ActionMove。

### View滚动事件分析，边际检测

### ViewPager使用和实现原理分析

### Android Jetpack
Room:官方数据库Orm框架
LifeCycle： 构建生命周期感知型组件，这些组件可以根据 Activity 或 Fragment 的当前生命周期状态调整行为
WorkManager：满足您的后台调度需求
Navigation：管理应用流程
ViewModels： 帮助应用实现更好的MVVM架构
DataBinding： 主要是基于依赖注入实现配置化UI的框架，对比框架有ViewBindings，GreenOrm
ConstraintLayout: 主要是可以减少布局层数，实现更灵活的页面布局




### 线程间通信方式有几种

### 为什么在Looper中while true 取消息不会阻塞主线程，Looper，MessageQueue，Handler的关系

### 为什么不能在工作线程直接刷新UI

### Android 自定义View和ViewGroup需要实现的方法 

###　Android draw， onDraw ，onLayout，方法的区别和联系


### Android 绘图相关 Paint 几种颜色混合模式


### Android Animation类型

### 如何优化图片加载

### 实现一个简单的LruCache


### Android中新增的几种数据类型 

SparseArray，IntSparseArray等

## service 
### Android Service相关知识
1. 生命周期。Service类型，两种方式 start&bind区别 。

2. Service里面弹出Dialog的方式



## 广播
1.广播的类型

2.Broadcast ANR


## ContentProvider
1. ContentProvider中使用的Context

2. ContentProvider的生命周期


## Android 常用的系统服务

### 1. AMS

### 2. PackageManagerService

### 3. WindowManagerService

### 4. InputManagerService

### 5. NotificationService

### 6. AccountManagerService

### 7. PowerManagerService


## Android 存储系统和存储优化相关内容

## AndroidStudio相关

###　AndroidStudio三方包管理工具maven

### AndroidStudio InstanceRun实现原理

## 简历上可能涉及到的内容

###　RN的原理

### Android应用 开发跨平台的几种流行方式（Native，Hybird，H5 +WebView单页 ）

### Kotlin相关知识

## Kotlin的一些优势分析

## Kotlin 跨线程方案分析

## Kotlin 一些语法特征

# Java基础知识复习

## Java5，Java6，Java7，Java8-11各版本feature

## Java 语法

### 数据类型，类接口和枚举

### java collection api相关 List,Map,Table,Tree,Queue,Stack

### Java HashMap，Arraylist分析，

### Java concorrent api 和多线程通信 线程池原理

### Java ThreadLocal原理



# Android Framework相关知识复习

## Android Binder 机制
### Binder 原理（上层，Linux层， Kernel层）

### Binder 的应用

## Android 跨进程通信的其他方式

## Android 系统启动流程分析

## Android 应用启动分析

## Android （JVM虚拟机）内存泄露的原理

## Android ANR的定义

1、发生原因
一句话总结：没有在规定的时间内，干完要干的事情，就会发生ANR。

2、ANR分类
从发生的场景分类：

Input事件超过5s没有被处理完
Service处理超时，前台20s，后台200s
BroadcastReceiver处理超时，前台10S，后台60s
ContentProvider执行超时，比较少见

3、从发生的原因分：

主线程有耗时操作，如有复杂的layout布局，IO操作等。
被Binder对端block
被子线程同步锁block
Binder被占满导致主线程无法和SystemServer通信
得不到系统资源（CPU/RAM/IO）

4、从进程的角度分：

问题出在当前进程:
主线程本身耗时, 或则主线程的消息队列存在耗时操作;
主线程被本进程的其他子线程所blocked;
问题出在远端进程(一般是binder call或socket等通信方式)


## Android 输入事件的产生和传递机制分析

##　Android apk打包流程分析和apk的签名机制

## Android 中View的底层绘制原理Chaerographer和SurfaceFlinger

## Android 丢帧 冻帧 和帧率优化相关知识分析

## Android 各个主要版本的Feature

## Android 编译系统相关知识 make .mk,.bp ，Aosp库各个目录的作用。以及主要版本的演进Android5.1，Android 6.0,Android 8.0,Android 9.0

## Android selinux 相关知识 sepolicy的定义，生成。

## Android 内存优化相关内容
内存优化，常常和卡顿联系在一起，在用户看来，应用使用起来常常卡顿，UI不流畅，是内存不足的最直观的体现，因此，对内存的优化，常常和UI卡顿的优化放在一起讨论。
内存问题包括内存泄露和内存溢出，还有内存耗尽，内存泄露比较难从用户直观的看出来，有的泄露是缓慢的积小成多的；内存溢出了之后，作为一种RuntimeException，如果没有被捕获，会导致程序被杀死；还有内存耗尽的问题，内存泄漏积少成多，最终达到应用分配的内存的最大值，会出现内存耗尽的问题，内存耗尽之后，系统会杀死该进程。

内存泄漏的原因（对象上存在不正确的引用，导致垃圾回收时无法被释放）：
1.使用非静态内部类，并持有Activity的引用，没有在Activity生命周期周期结束之前释放；
2.使用静态变量，持有Activity的强引用，没有在Activity生命周期结束前释放；
3.存在未关闭的Cursor，文件流等；
4.使用Bitmap的时候，没有在不需要的时候，把Bitmap占用的内存Recycle掉
5.在多线程的场景中，工作线程持有对主线程的引用（非ApplicationContext），不释放


内存溢出的原因：
1.太多的内存泄露
2.如在for循环中声明很多资源占用较大的对象实例。如Bitmap，造成生成很多的内存碎片
3.加载大的图片不基于显示区域对图片进行优化，造成资源浪费。
4.在滑动或者是其他频繁调用的方法中声明图片等实例对象（不使用缓存）或者是频繁的io操作


如何内存优化：
1. 使用一些三方的内存检测的工具，如LeakCanary （研发状态），Bugly（线上）
2. 静态代码检测工具，如StrictMode，可以检测出一些未关闭的流，cursor，之类的问题。
3. 使用AS自带的cpu 监控和 内存监控 网络监控等对于怀疑的场景进行监控，如果说出现内存抖动（太多小内存频繁申请，回收）或者是内存持续占用缓慢升高，主动gc也回收不到内存的情况（内存泄露），就需要对代码中逻辑进行检查
4. 如果说是对于系统中其他已经上线的应用进行检测，可以使用systrace。systrace可以看到系统当时的状态信息。对于分析问题，更具有全面意义。


系统卡顿优化： 

帧率：一般来说普通应用，60帧/s基本上就算是比较流畅的状态，游戏类应用可能帧率到达30帧/s的状态是比较流畅的。
卡顿（丢帧）： 在60帧以下，一帧绘制超过16ms，就会出现丢帧。
冻帧：一帧绘制超过700ms 就表示冻帧

卡顿的原因：
1.频繁的GC，也就是内存抖动的情况，在一段时间内，有大量的对象的生成和回收，例如在for循环里声明对象，在滑动或者是绘制的方法中声明很多对象。
2.过度绘制： 一个对象被绘制很多遍，或者是不可见元素仍然被绘制了	
3.过度复杂的布局






## Android I/O优化相关内容

### Linux I/O 的基本组成
众所周知，Android 基于 Linux 系统，先介绍一些 Linux 上 I/O 的知识。

1. I/O 操作由应用程序、文件系统和磁盘共同完成，应用程序将 I/O 命令发送给文件系统，文件系统在合适的时间把 I/O 指令发送给磁盘。I/O 的流程如下图：

![io流程图](images/interview/image/io1.webp)

CPU 和内存的速度比磁盘快得多，I/O 操作的瓶颈在于磁盘的性能。为了降低磁盘对应用程序的影响，文件系统要进行各种各样的优化。

2. 文件系统
简单来说，文件系统就是存储和组织数据的方式。应用程序调用 read() 方法，系统会通过中断从用户空间进入内核空间，然后经过虚拟文件系统、具体文件系统、页缓存。
![文件系统](images/interview/image/io2.webp)

	* 虚拟文件系统（VFS）。主要用于屏蔽具体的文件系统，为应用程序的操作提供一个统一的接口。
	* 文件系统（File System）。ext4、F2FS 都是具体文件系统实现。每个文件系统都有适合自己的场景。
	* 页缓存（Page Cache）。文件系统对数据的缓存，读文件时先检查页缓存，如果命中就不去读磁盘。


3. 磁盘
磁盘指的是系统的存储设备，常见的有机械硬盘、固态硬盘等。如果发现应用程序要读的数据没有在页缓存中，这时候就需要真正向磁盘发起 I/O 请求。磁盘 I/O 的过程要先经过内核的通用块层、I/O 调度层、设备驱动层，最后才会交给具体的硬件设备处理。

![磁盘调度架构图](images/interview/image/io3.webp)

磁盘架构
* 通用块层。接收上层发出的磁盘请求，并最终发出 I/O 请求。它与 VPS 的作用类似。
* I/O 调度层。根据设置的调度算法对请求合并和排序。不能接收到磁盘请求就立刻交给驱动层处理。
* 块设备驱动层。根据具体的物理设备，选择对应的驱动程序，通过操控硬件设备完成最终的 I/O 请求。

### Android IO

1. 文件系统： Android普遍采用的是ext4的文件系统，以及f2fs（2012三星针对为闪存研发的文件系统，它针对闪存进行了大量优化，F2FS 文件系统在小文件的随机读写方面比 ext4 更快，**随机写性能提高**），目前还有只在华为上应用的，华为自研的EROFS（2018年6月，华为工程师在开源社区展示了基于Linux的全新只读文件系统EROFS（Extendable Read-Only File System），采用改进的压缩算法，致力于提高文件访问性能，特别是**随机读**性能。根据当时公布的测试数据，可以看到执行随机读取数据时，EROFS有着一边倒的优势，并且文件压缩率越小时优势越明显：当文件压缩率为4%（即100MB文件压缩为4MB）时，提升高达172%。在EROFS面世后的半年多里，华为工程师对其持续打磨，终于在P30上实现了规模商用）

2. 磁盘格式： Android 使用的闪存主流有2种类型，一种是emmc（Embedded Multi Media Card），另外一种是UFS（2.0,2.1），USF的读写速度更快，性能更好（最高速度在300MB/s以下的大几率都是eMMC，在500MB/s附近则可能是UFS 2.0，在700MB/s以上则较大可能是UFS 2.1了）

	* 查看磁盘类型方法：`ls /proc/fs/*` ；在ext4文件系统下如果看到很多个emmc-blkxxx，则是emmc的 ，如果说文件系统为f2fs的就继续查看 `ls /proc/fs/f2fs/`下，如果说以sd开头则是ufs居多。

	* 另外，如果说是ext4的文件系统，查看emmc-blk+数字对应的具体分区的方法：

	> `ls -l /dev/block/platform/ff0f0000.rksdmmc/by-name/`

3. 手机变卡
Android 手机用久了会变卡，除了系统升级、设备折旧等因素，还和 I/O 有密切关系。I/O 操作变慢的原因有下面几条：

	* 内存不足。系统回收 Page Cache 和 Buffer Cache 的内存，大部分的写操作会直接落盘，导致性能低下。
	* 写入放大
	闪存重复写入需要先进行擦除，一次写入会引起整个块数据的迁移，导致写入时间非常久。
	* 设备性能差 
	在高负载的情况下容易出现瓶颈。
	* 文件损坏
	文件损坏是令人头疼的问题，大多是由不正确的操作导致的。文件损坏的原因可以从应用程序、	文件系统和磁盘三个角度来分析：

		* 应用程序。
		大部分的 I/O 方法都不是原子操作，文件的跨进程或者多线程写入、使用一个已经关闭的文件描述符 fd 来操作文件，都有可能导致数据被覆盖或者删除。
		* 文件系统。虽说内核崩溃或者系统突然断电都有可能导致文件系统损坏，不过文件系统也做了很多的保护措施。例如 system 分区保证只读不可写，增加异常检查和恢复机制。
		* 磁盘。手机上使用的闪存是电子式的存储设备，所以在资料传输过程可能会发生电子遗失等现象导致数据错误。

4. I/O 操作的类型

总体分为，标准IO，mmap,直接IO三种类型。

![IO操作类型](images/interview/image/io4.webp)

	1. 标准 I/O
	应用程序平时用到 read/write 操作都属于标准 I/O，也就是缓存 I/O（Buffered I/O）。它的关键特性有：
		- 对于读操作，当应用程序读取某块数据时，如果这块数据已经在页缓存中，那么就不需要经过物理读盘操作。
		- 对于写操作，应用程序会先将数据写到页缓存中去，不需要等全部数据被写回磁盘，系统会定期将页缓存中的数据刷到磁盘上。
		- 缓存 I/O 可以很大程度减少真正读写磁盘的次数，从而提升性能。但是延迟写机制可能会导致数据丢失。在实际应用中，如果某些数据非常重要，我们应该采用同步写机制。
		- 读操作时，数据会先从磁盘拷贝到 Page Cache 中，然后再从 Page Cache 拷贝到应用程序的用户空间，这样就会多一次内存拷贝。内存相对磁盘是高速设备，即使多拷贝一次，也比真正读一次硬盘要快。

	2. mmap
	mmap 把文件映射到进程的地址空间，提高了 I/O 的性能。
	mmap 的优点有：
		- 减少系统调用。只需要一次 mmap() 系统调用，后续所有的调用像操作内存一样。
		- 减少数据拷贝。mmap 只需要从磁盘拷贝一次，由于做过内存映射，不需要再拷贝回用户空间。
		- 可靠性高。mmap 把数据写入页缓存后，跟缓存 I/O 的延迟写机制一样。
	存在的缺点：
		- 虚拟内存增大。Apk、Dex、so 都是通过 mmap 读取。mmap 会导致虚拟内存增大，mmap 大文件容易出现 OOM。
		- 磁盘延迟。mmap 通过缺页中断向磁盘发起真正的磁盘 I/O，不能通过 mmap 消除磁盘 I/O 的延迟。在 Android 中可以将文件通过 MemoryFile 或者 MappedByteBuffer 映射到内存，然后进行读写，使用这种方式对于小文件和频繁读写操作的文件还是有一定优势的。

	mmap 比较适合对同一块区域频繁读写的情况，推荐使用 I/O 线程来操作。用户日志、数据上报都满足这种场景，另外需要跨进程同步的时候，mmap 也是一个不错的选择。Android 跨进程通信有自己独有的 Binder 机制，它内部也是使用 mmap 实现。

	3. Direct I/O
	一些数据库自己实现了数据和索引的缓存管理，对页缓存的依赖没那么强烈。它们想绕开页缓存机制，减少一次数据拷贝，它的数据也不会污染页缓存。
	直接 I/O 访问文件方式减少了一次数据拷贝和一些系统调用的耗时，很大程度降低了 CPU 的使用率以及内存的占用。负面影响就是读写操作都是同步执行，导致应用程序等待。

	4. 同步与异步 I/O
	多线程阻塞式在 I/O 操作上的并没有优势，I/O 操作的主要瓶颈在于磁盘带宽。所以 I/O 操作不能开大量的线程。
	NIO 是非阻塞 I/O，将 I/O 以事件的方式通知，可以减少线程切换的开销。NIO 的最大作用不是减少读取文件的耗时，而是最大化提升应用整体的 CPU 利用率。
	另外，非常推荐 Square 的 Okio，它支持同步和异步 I/O，也做了比较多的优化。

I/O 优化对提升应用的体验非常有用，希望上面所讲的内容对你有帮助。

作者：落英坠露
链接：https://www.jianshu.com/p/43af9c156674
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

5. IO优化

	1. emmc 擦写次数和判断 emmc 使用寿命是否到期方法

	2. 查看系统整体的 io 状态

		2.1 iostat
		
		比如说我们想查看当前系统的 io 状态，以 kb/s　作为单位:	
		
		
    ```shell
	1|root@rk3368:/ # busybox iostat -k -d                                
	Linux 3.10.0 (localhost) 	03/03/21 	_aarch64_	(8 CPU)
	
	Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
	mmcblk0          70.26      1981.90       269.29  225512879   30641156
	mmcblk0p7         0.00         0.18         0.00      20288          0
	mmcblk0p9         0.00         0.00         1.10        305     124868
	mmcblk0p11        3.70       228.62         0.00   26013646         16
	mmcblk0p12        0.00         0.00         0.11        474      11992
	mmcblk0p15       59.42      1753.10       268.08  199478166   30504280
    ```


		比如说我们想查看当前系统的 io 状态，以 kb/s　作为单位，并且没两分钟统计一次，共5次:

		```
		busybox iostat -d -k  2 5
		```

		2.2 iotop

		iostat　只能查看系统的整体 io　状态，当 io 状态异常高时，　不能定位是哪个进程引起的问题，　这时我们可以使用 iotop 这个工具来查看每个进程的 io 占用比例。但是iotop只在Android9.0以上才原生集成，低版本系统，需要push Github开源脚本 [iotop.sh]('https://github.com/laufersteppenwolf/iotop'). 但是该脚本我在使用过程中发现会有不能用的情况，提示`Your kernel does not support I/O accounting,
		which is required for this tool to work :(`
	
		2.3 vmstat
		vmstat是一个非常有用的脚本，用来查看虚拟机当前的一些状态数据。

		```
		vmstat 2
		```

		2.4 http://qiushao.net/2020/02/11/Android/iostat/
		iotop　只可以查看某个时间点的 io 速率，　我们需要统计某个进程总共的 io 总量时就得使用　cat /proc/$pid/io 这个方法了。
## Android 性能优化相关内容

## Android 应用启动优化

1. 冷启动：系统中没有该应用的进程，这时系统会创建一个新的进程分配给该应用；
2. 热启动：在启动应用时，系统中已有该应用的进程（例：按back键、home键，应用虽然会退出，但是该应用的进程还是保留在后台）；
应用启动时间统计：
`adb shell am start -W componenetName/Action`
3. 温启动： 
温启动包含了在冷启动期间发生的部分操作；同时，它的开销要比热启动高。有许多潜在状态可视为温启动。例如：

用户在退出应用后又重新启动应用。进程可能已继续运行，但应用必须通过调用 onCreate() 从头开始重新创建 Activity。
系统将您的应用从内存中逐出，然后用户又重新启动它。进程和 Activity 需要重启，但传递到 onCreate() 的已保存的实例 state bundle 对于完成此任务有一定助益。

### 冷启动优化
冷启动是指应用从头开始启动：系统进程在`冷启动`后才`创建应用进程`。发生冷启动的情况包括`应用自设备启动后`或`系统终止应用后首次启动`。这种启动给最大限度地减少启动时间带来了最大的挑战，因为系统和应用要做的工作比在另外两种启动状态中更多。

在冷启动开始时，系统有三个任务，它们是：

1. 加载并启动应用。
2. 在启动后立即显示应用的空白启动窗口。
3. 创建应用进程。

系统一创建应用进程，应用进程就负责后续阶段：

1. 创建应用对象。
2. 启动主线程。
3. 创建主 Activity。
4. 扩充视图。
5. 布局屏幕。
6. 执行初始绘制。
一旦应用进程完成第一次绘制，系统进程就会换掉当前显示的后台窗口，替换为主 Activity。此时，用户可以开始使用应用。

参考：https://developer.android.com/topic/performance/vitals/launch-time?hl=zh-cn
https://developer.aliyun.com/article/687808（分析的比较好）

冷启动优化：
1. 减少在Application的onCreate方法，以及主Activity的生命周期（onCreate，onStart）中进行IO或者是其他耗时操作，可以延时加载的就延时或者放在工作线程
2. 	绘制优化，检查view层级，以及view绘制的复杂度，尽量优化层级，减少绘制耗时，在layout中，使用ViewStub，在适当的时候再inflate真正的布局
3. 注意尽量少在Application，Activity启动时，调用很多Service以及初始化很多非常复杂的工具类，对于某些必要的的工具类，尽量做成单例
4. 使用系统主题，来预防系统白屏的问题，由于启动的时候，在应用尚未初始化绘制完成之前，会临时展示一个预览窗口（白屏），可以通过给Activity加上theme主题，增加颜色，这样就可以不显示白屏，然后再在MainActivity的onCreate方法调用super.onCreate之前把主题换回真正的主题。

### 热启动
由于热启动的时候，应用走的生命周期方法是
onRestart-> onStart-> onRestoreInstanceState -> onResume,所以重点关注Activity在这几个生命周期中的工作流程和逻辑。进行相应优化，同时应该考虑冷启动相关的一些界面绘制方面的优化点。








## Android onSaveInstanceState和onRestoreInstanceState调用时机
https://blog.csdn.net/Liu_yunzhao/article/details/79382897
onSaveInstanceState()肯定被执行时机

1、当用户按下HOME键时 
2、从最近应用中选择运行其他的程序时 
3、按下电源按键（关闭屏幕显示）时 
4、从当前activity启动一个新的activity时 
5、屏幕方向切换时(竖屏切横屏或横屏切竖屏)

onSaveInstanceState()执行顺序是在onPause()之后，onStop()之前即： 
onPause() –> onSaveInstanceState() –> onStop()


# HTTP相关知识

## HTTP三次握手和四次挥手

## HTTP协议演进（HTTP1.0,1.1,2.0）

## HTTPS 加密方式（对称加密，非对称加密）

## HTTP 协议详解，Head Body分析



# 虚拟机相关

## JVM虚拟机的内存模型	

## Android虚拟机（Dalvik,art）虚拟机内存模型和特点

## Android 应用 启动流程虚拟机流程分析

## Android 虚拟机的垃圾回收机制






# 设计模式

## 单例模式
## 观察者模式
## 工厂模式，简单工厂
## 装饰器模式	
## 建造者模式
## 策略模式
## 适配器模式
## 代理模式
## 模板模式
## 桥接模式
## 状态模式
## 组合模式
## 命令模式
## 责任链模式

## 设计模式的几大原则
### 开闭原则
### 迪米特法则



# 数据结构和算法


# 高频面试题

http://6a9396c1.wiz06.com/wapp/pages/view/share/s/1GAVr10WjQ002tX_1K3G9n4B3HlX-g2AmQ0_20ExS_27scZ8
https://github.com/BlackZhangJX/Android-Notes/blob/master/Docs/Android%E7%9F%A5%E8%AF%86%E7%82%B9%E6%B1%87%E6%80%BB.md#%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B-1

